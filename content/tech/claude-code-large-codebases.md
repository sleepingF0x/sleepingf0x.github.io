---
title: "让 Claude Code 读懂大代码库"
date: 2026-05-16T00:00:00+08:00
draft: false
description: "基于 Anthropic 的大代码库实践文章，整理 Claude Code 在复杂仓库里的配置方法。"
tags: ["Claude Code", "AI Coding", "Workflow", "Agent"]
series: ["Claude-Code-Guide"]
---

Anthropic 最近发了一篇文章，讲 Claude Code 怎么在大代码库里工作。它里面最有用的点不是某个 prompt 技巧，而是一个更工程化的判断：Claude Code 在大仓库里好不好用，很大程度取决于仓库本身有没有被整理成 agent 能导航的环境。

Claude Code 不是先把整个仓库做成一个中心化索引，再从索引里召回答案。它更像一个坐在你电脑前的开发者，会读文件、搜关键词、看目录、跟引用、跑命令。这个模式的好处是它看到的是本地最新代码，不太会被过期索引误导；坏处是你不能只丢一句“帮我改一下支付逻辑”，然后指望它在几十万行代码里自己精准落点。

在大代码库里用 Claude Code，重点是减少它的无效探索。

## 从相关目录启动

如果是 monorepo，不要每次都从仓库根目录启动。改某个服务，就先进入那个服务目录；改某个 package，就从 package 目录启动。

Claude Code 仍然可以向上读取父级 `CLAUDE.md`，但它的第一视角会更接近当前任务。搜索范围变小，读到的文件更相关，后面跑测试、lint、build 也更容易收窄。

可以把日常入口做成 alias：

```bash
alias c-api='cd ~/repo/apps/api && claude'
alias c-web='cd ~/repo/apps/web && claude'
alias c-worker='cd ~/repo/services/worker && claude'
```

这种小动作比写一大段 prompt 更稳定。

## 写分层 CLAUDE.md

根目录的 `CLAUDE.md` 只放全局信息：系统大概怎么分层、关键目录负责什么、通用代码规范、绝对不能踩的坑。

不要把它写成百科全书。根文件每次都会进上下文，越长越容易把真正有用的信息挤掉。

更适合的结构是分层：

```text
repo/
  CLAUDE.md                    # 全局架构、通用约定、关键禁忌
  apps/web/CLAUDE.md            # 前端启动、测试、路由、组件约定
  apps/api/CLAUDE.md            # API 测试、数据库迁移、错误处理约定
  services/worker/CLAUDE.md     # 队列、重试、部署前检查
```

子目录里的 `CLAUDE.md` 要写具体命令，比如：

```markdown
## 常用命令

- 单测：`pnpm test --filter @repo/api`
- 类型检查：`pnpm typecheck --filter @repo/api`
- 本地服务：`pnpm dev --filter @repo/api`

## 修改数据库相关代码时

- 先看 `migrations/README.md`
- 新 migration 必须有 rollback
- 不要在请求路径里做全表扫描
```

这类信息比“请认真修改代码”有用得多。Claude Code 不缺认真，缺的是当前仓库的入口、边界和验证方式。

## 让测试命令可以局部运行

大仓库里最容易浪费时间的是全量验证。小改动如果每次都跑完整 CI，本地反馈会慢到不可用。

可以给每个子项目写清楚最小验证命令：

```markdown
## 验证范围

改 `src/billing/**` 时：

1. `pnpm test billing`
2. `pnpm typecheck --filter @repo/api`
3. 如果改了 invoice 生成逻辑，再跑 `pnpm test:e2e invoice`
```

这不是为了偷懒，而是为了让 Claude Code 有明确的反馈循环。它改完代码后能马上跑一组小检查，发现问题再修；最后需要合并前，再跑完整检查。

如果仓库没有局部测试命令，可以先补这个。对大代码库来说，这往往比多写十条 prompt 更有价值。

## 把噪音排出去

Claude Code 会按需读文件，但它也可能被生成物、构建产物、第三方代码和大日志带偏。

建议团队统一排除这些内容：

```text
public/
dist/
build/
coverage/
*.log
generated/
vendor/
```

如果某些生成文件必须保留在仓库里，也可以在相关 `CLAUDE.md` 里写清楚：

```markdown
`src/generated/**` 是自动生成代码，不要手改。需要修改类型时，改 `schema/` 后重新生成。
```

这条规则很适合放进 hook 或权限配置里。能用自动化约束的地方，不要只靠自然语言提醒。

## 用 hooks 处理确定性的事

Hooks 适合做格式化、lint、阻止危险命令、动态加载上下文这类确定性工作。

比如每次编辑 TypeScript 文件后自动跑 formatter，或者在提交前检查是否改了不该改的生成目录。Claude Code 可以遵守规则，但稳定性最高的方式仍然是把规则变成脚本。

适合放 hook 的事情有几类：

- 保存后自动 format
- 修改数据库 migration 后提示跑对应检查
- 阻止直接编辑生成文件
- 阻止提交 `.env`、密钥、临时日志
- 任务结束时提醒是否需要更新局部 `CLAUDE.md`

Hook 的价值不是“让 Claude 更聪明”，而是把团队已经确定的规则固定下来。

## 把专门流程放进 skills

`CLAUDE.md` 适合放长期有效的项目知识，skills 适合放按任务触发的流程。

比如安全审查、发布检查、文档更新、数据库 migration review，这些流程不应该常驻在根 `CLAUDE.md`。它们平时用不到，只有相关任务出现时才需要加载。

一个简单判断是：

- 每次会话都该知道的，放 `CLAUDE.md`
- 只有某类任务才需要的，做成 skill
- 可以自动执行的，不写进 prompt，交给 hook
- 需要连接外部系统的，再考虑 MCP

这样上下文不会越堆越厚，后面维护起来也更清楚。

## LSP 比纯 grep 更适合大仓库

在小项目里，grep 通常够用。到了大仓库，`handler`、`Service`、`getConfig` 这种名字可能出现几百次，纯文本搜索很容易噪音爆炸。

LSP 能提供更接近 IDE 的导航能力，比如 go to definition、find references、区分同名符号。对 C、C++、Java、C# 这类大型工程尤其有用。

但 LSP 不是默认就能工作。你需要确认语言服务器装好了，项目能被它正确识别，必要时还要配置编译数据库或 workspace。这里值得花时间，因为它会直接影响 Claude Code 找代码的质量。

## 用 subagents 做探索，不要一边探索一边改

大改动前，可以让 subagent 先做只读探索：

```text
请只读分析 billing 模块：
1. 找到 invoice 生成入口
2. 梳理主要调用链
3. 标出测试文件
4. 不要修改代码，只返回文件路径和结论
```

主会话拿到结果后，再决定改哪里。这样能把“摸地图”和“动手术”分开，主上下文也不会被大量无关搜索结果塞满。

这个习惯在遗留系统里很有用。先让 subagent 画出局部地图，再让主 agent 基于地图做小范围修改。

## MCP 不要太早上

MCP 很适合接内部文档、工单系统、监控平台、代码搜索服务和内部 API。但如果仓库本身还没有清晰的 `CLAUDE.md`、局部测试命令、ignore 规则和基础导航，上 MCP 只会让可访问的信息更多，问题不一定更少。

更稳的顺序是：

1. 先让 Claude Code 能在本地仓库里找对文件
2. 再让它能跑对局部验证命令
3. 然后把重复流程沉淀成 hooks 和 skills
4. 最后再接 MCP，把外部系统引进来

先把本地闭环打通，再扩展外部能力。

## 配置也要定期删

Claude Code 的配置不是写完就结束。模型能力会变，工具能力也会变，以前为了补短板写下的规则，过几个月可能变成束缚。

比较好的节奏是三到六个月回看一次：

- 根 `CLAUDE.md` 有没有变成杂物间
- 子目录命令是否还准确
- hooks 是否还在解决真实问题
- skills 有没有重复或过时
- 权限规则是不是太宽或太窄
- 以前的 workaround 是否已经不需要了

大代码库里的 Claude Code 配置，最后会变成一种 DevEx 基础设施。有人维护，它会越来越顺；没人维护，它就会慢慢变成另一套过期文档。

## 我的实践顺序

如果要从零开始整理一个大仓库，我会按这个顺序做：

1. 在根目录写一份短的 `CLAUDE.md`，只放架构、目录说明、全局禁忌。
2. 给最常改的两个子目录补局部 `CLAUDE.md`，写清测试、lint、build 命令。
3. 把生成物、构建产物、日志和第三方代码排除掉。
4. 确认局部测试可以独立跑，不行就先补命令。
5. 给高频任务做 skills，比如 code review、release check、migration check。
6. 把格式化、安全检查、禁止编辑生成物这些规则交给 hooks。
7. 给主要语言配好 LSP。
8. 遇到大范围陌生模块时，先派 subagent 做只读探索。
9. 等本地流程稳定后，再接 MCP。
10. 每隔几个月删掉过时配置。

Claude Code 在大代码库里的关键不是一次性给它更多上下文，而是让它每一步都能拿到刚好够用、并且足够准确的上下文。仓库越大，这件事越重要。