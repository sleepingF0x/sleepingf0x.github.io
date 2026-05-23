---
title: "用 Cloudflare Worker 搭一个轻量 Webhook Gateway"
date: 2026-05-23T00:00:00+08:00
draft: false
description: "记录一个 Cloudflare-native webhook gateway 应该具备的完整能力：多入口、多通道、队列重试、事件历史、安全边界和 Telegram 管理入口。"
tags: ["Cloudflare Worker", "Telegram Bot", "Webhook", "Serverless", "ntfy"]
---

一开始我只是想给 Telegram bot 配一个 webhook。Cloudflare Worker 天然有 HTTPS endpoint，写一个 `POST /telegram`，再用 Telegram `setWebhook` 绑定过去，事情就结束了。

但 Telegram 只是其中一个入口。更完整的形态应该是一个 Cloudflare-native webhook gateway：外部系统把事件打进来，Worker 验签、标准化、入库、排队，再按规则发到 Telegram、ntfy、Discord、Slack、邮件或其他 webhook。

我不想为了这件事维护一套 n8n、Node-RED 或 Windmill。那些平台很强，但对个人博客、自动化和小型系统通知来说偏重。Cloudflare Worker 更适合做薄入口，D1、KV、Queues、Cron Triggers 补上状态、限流、重试和清理。

## 最终形态

这个 gateway 最后应该长这样：

```text
GitHub / Cloudflare / Stripe / Uptime Kuma / RSS / Custom Form / Telegram
  ↓
Ingress Adapter
  ↓
Signature Verification + Rate Limit + Dedup
  ↓
Normalized Event
  ↓
D1 Event Store
  ↓
Queue Fanout
  ↓
Telegram / ntfy / Discord / Slack / Teams / Email / Generic Webhook
```

核心模型只有四个：

```text
inbox：一个事件入口，比如 github-build、cloudflare-alert、contact-form
event：一次收到的 webhook，包含 raw payload 和标准化后的字段
channel：一个通知出口，比如 telegram-main、ntfy-phone、discord-ops
route：把 inbox、条件和 channel 连接起来
```

这几个模型够支撑大部分个人自动化。入口负责接收事件，事件负责留下记录，通道负责发消息，路由负责决定发到哪里。实现时尽量让每一层都薄，不要把验签、格式化、投递和存储揉在一个函数里。

## 输入通道

输入通道要覆盖两类：机器事件和人发命令。

机器事件走 `/hooks/:slug`：

```text
POST /hooks/github-build
POST /hooks/cloudflare-alert
POST /hooks/contact-form
POST /hooks/uptime
POST /hooks/payment
POST /hooks/custom
```

每个 `slug` 对应一个 inbox。inbox 里配置验签方式、payload 解析器、默认等级、路由规则和保留时间。

Telegram 走单独入口：

```text
POST /telegram
```

这个入口接 Telegram bot update。它不只是 echo 消息，而是 gateway 的管理入口。比如：

```text
/status                 查看 gateway 状态
/recent errors          看最近失败事件
/retry <event_id>       重发某个事件
/mute github-build 1h   临时静音某个 inbox
/test telegram-main     测试某个 channel
/routes                 查看当前路由
```

这样 Telegram 既是通知通道，也是轻量运维界面。个人项目里，这比再做一个 Web UI 更实用。

## 输出通道

输出通道不要写死在业务逻辑里，应该做成 adapter。

```text
channels/
  telegram.ts
  ntfy.ts
  discord.ts
  slack.ts
  teams.ts
  email.ts
  webhook.ts
```

每个 adapter 做同一件事：把内部事件转成目标通道需要的请求。

```text
NormalizedEvent + ChannelConfig
  → method / url / headers / body
```

Telegram 适合主通知和交互，ntfy 适合轻量 push，Discord/Slack/Teams 适合团队频道，email 适合需要留档的通知，generic webhook 适合接下游系统。

generic webhook output 要谨慎。它允许系统向任意 URL 发请求，必须做 SSRF 防护，不能请求 localhost、内网 IP、link-local 地址和云厂商 metadata endpoint。

## 事件格式

内部事件格式要稳定。入口可以越来越多，但系统内部只认一种事件。

```ts
type NormalizedEvent = {
  id: string
  inbox: string
  source: string
  title: string
  body: string
  level: 'info' | 'warning' | 'error' | 'critical'
  status: 'received' | 'queued' | 'delivered' | 'failed'
  url?: string
  fingerprint?: string
  rawPayloadRef?: string
  receivedAt: string
}
```

`fingerprint` 用来去重。GitHub workflow、Cloudflare alert、Uptime Kuma outage 这类事件经常会重试，没有去重就容易刷屏。

`rawPayloadRef` 可以为空。如果 payload 很大，再把原始数据放到 R2，D1 只存引用。多数个人场景直接把裁剪后的 raw JSON 存 D1 就够了。

## 数据模型

D1 负责存配置和事件历史。

```text
inboxes
  id
  slug
  name
  source_type
  verification_type
  secret_ref
  retention_days
  enabled

channels
  id
  name
  type
  config_json
  enabled

routes
  id
  inbox_id
  channel_id
  level_filter
  keyword_filter
  enabled

events
  id
  inbox_id
  source
  title
  body
  level
  status
  fingerprint
  raw_payload
  received_at

delivery_attempts
  id
  event_id
  channel_id
  status
  status_code
  error_message
  attempted_at
```

这几张表够用了。事件出了问题，可以查它有没有进来、进来后有没有入队、发到了哪些 channel、失败原因是什么。

## 队列和重试

Webhook 入口不能被下游通知服务拖住。入口收到事件后应该完成这几件事：验签、限流、去重、标准化、入库、投递到 Queue，然后返回。

真正的发送放到 Queue consumer 里：

```text
Queue message
  → load event
  → match routes
  → dispatch channels
  → write delivery_attempts
  → retry or dead letter
```

重试要按通道记录。Telegram 失败不应该影响 ntfy，Slack 失败不应该让整个事件变成未知状态。

需要一个 dead-letter queue。重试多次仍失败的消息进 DLQ，再发一条高优先级 Telegram 通知：哪个 event、哪个 channel、失败了几次、最后一次错误是什么。

## 路由规则

route 不要做成复杂规则引擎，但要有几个基本条件：

```text
按 inbox 路由
按 level 路由
按关键字路由
按时间窗口静音
按 channel enabled 状态过滤
```

比如：

```text
github-build + error       → Telegram + ntfy
cloudflare-alert + warning → Telegram
contact-form               → Telegram + email
uptime + critical          → Telegram + ntfy + Discord
```

这里最容易过度设计。不要急着做表达式语言，也不要做可视化 workflow。D1 里的几列配置足够覆盖大部分个人自动化。

## 安全边界

这个系统是公开入口，安全边界比功能更重要。

每个输入通道都要有自己的校验方式：

```text
Telegram：X-Telegram-Bot-Api-Secret-Token
GitHub：X-Hub-Signature-256
Stripe：Stripe-Signature
Custom webhook：X-Webhook-Secret
Contact form：Turnstile + rate limit
Cloudflare notification：独立 secret
```

不要所有入口共用一个 secret。泄露一个，不应该拖垮全部入口。

敏感配置只能放 Cloudflare secrets 或 D1 加密字段里，API 返回时必须 redaction：

```text
123456789:AA...redacted
https://hooks.slack.com/services/...redacted
re_...redacted
```

Telegram 消息如果使用 `parse_mode: HTML`，用户输入必须 escape。表单字段、GitHub commit message、外部 webhook body 都不能直接拼进 HTML。

公开 inbox 要做 payload size limit。过大的 payload 直接拒绝，或者写入 R2 后只保留引用。否则一个公开 webhook 很容易被打成日志和存储垃圾桶。

## 限流和去重

KV 很适合做短期状态。

```text
rate_limit:{inbox}:{ip}
dedup:{inbox}:{fingerprint}
mute:{inbox}
```

限流按 inbox 和 IP 做，contact form 这种公开入口还要叠 Turnstile。去重按 fingerprint 做，TTL 可以设成几分钟到几小时。

Telegram 管理命令也要限制 user id。只有白名单用户可以执行 `/retry`、`/mute`、`/routes` 这类命令。

```ts
const allowedUserIds = new Set([123456789])
```

不要只依赖“没人知道 bot 名字”。

## 观测和运维

最终版本至少要有这些接口：

```text
GET  /health
GET  /v1/events?inbox=&status=&level=
GET  /v1/events/:id
POST /v1/events/:id/retry
GET  /v1/channels
POST /v1/channels/:id/test
GET  /v1/routes
PATCH /v1/routes/:id
```

这些接口用 `X-API-Key` 保护。最终版本里，API key 应该带 scopes，比如 `events:read`、`events:retry`、`channels:write`、`routes:write` 和 `admin`。

Telegram bot 里也要有对应的轻量命令。平时我不会打开后台看 dashboard，但 Telegram 收到失败通知后，可以直接 `/retry <event_id>`，这才是个人工具的舒服形态。

## Cloudflare 组件怎么用

最终版本会用到这些 Cloudflare 组件：

| 组件 | 用途 |
| --- | --- |
| Workers | HTTP 入口、Telegram bot、API、Queue consumer |
| D1 | inbox、channel、route、event、delivery attempt |
| KV | 限流、去重、静音状态、短期缓存 |
| Queues | 多通道 fanout、失败重试、DLQ |
| Cron Triggers | 清理过期事件、发送每日摘要、检查死信队列 |
| R2 | 大 payload、附件、长期原始事件，可选 |
| Turnstile | 公开表单入口防 bot，可选 |

这套东西仍然是 Cloudflare-native，不需要 VPS，也不用维护常驻进程。

## 配置方式

我更倾向把敏感值放 secret，把动态配置放 D1。

Cloudflare secrets：

```text
ADMIN_API_KEY
TELEGRAM_BOT_TOKEN
TELEGRAM_WEBHOOK_SECRET
GITHUB_WEBHOOK_SECRET
STRIPE_WEBHOOK_SECRET
NTFY_TOKEN
RESEND_API_KEY
```

D1 里存 channel config，但 API 读取时做脱敏。

```json
{
  "name": "telegram-main",
  "type": "telegram",
  "config": {
    "chat_id": "123456789",
    "parse_mode": "plain"
  }
}
```

Telegram bot token 不应该出现在 D1 config 里。所有 Telegram channel 共用 Worker secret 里的 `TELEGRAM_BOT_TOKEN`，channel 只存 chat id 和格式化偏好。

## 最后

这不是一个 Telegram bot 教程，也不是一个 n8n 替代品。它更像一个放在 Cloudflare 边缘上的个人事件总线。

最后版本应该具备这些能力：

```text
多入口：Telegram、GitHub、Cloudflare、Stripe、Uptime、表单、自定义 webhook
多出口：Telegram、ntfy、Discord、Slack、Teams、email、generic webhook
事件历史：D1 存 event 和 delivery attempts
可靠投递：Queue fanout、按通道重试、dead-letter queue
安全边界：独立验签、限流、去重、SSRF 防护、secret redaction
管理入口：HTTP API + Telegram bot commands
运维能力：health check、失败查询、重发、静音、每日摘要
```

这才是我想要的形态：不用维护服务器，不依赖重型 workflow 平台，又能把个人系统里的事件统一。Telegram 是入口和控制台，ntfy 是备用推送，D1 留下历史，Queue 保证投递，Cron 负责清理和摘要。复杂度没有消失，但都落在 Cloudflare 已经提供的组件里。
