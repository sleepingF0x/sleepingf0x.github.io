---
title: "spec-flow：把 AI 编程从对话变成流程"
date: 2026-07-05T00:00:00+08:00
draft: false
description: "介绍一套双模型 spec-driven 开发工作流：Claude 负责需求和验收，Codex 负责冷启动实现，中间用 GitHub Issues 和落盘文档交接。"
tags: ["Claude Code", "Codex", "AI Coding", "Workflow", "Spec"]
series: ["Claude-Code-Guide"]
---

我最近在整理一套 spec-driven 的 AI coding workflow，叫 spec-flow。

它解决的不是“哪个模型写代码更强”，而是另一个更实际的问题：需求、实现、验收如果都挤在同一个长对话里，最后很容易分不清到底是 spec 模糊、实现跑偏，还是验收只是在迁就已经写出来的代码。

spec-flow 的做法很简单：把角色拆开。

Claude 负责把模糊需求拷问成 PRD 和可执行 issue，也负责最后验收；Codex 负责从一个干净会话里读 issue 和 repo 文档，按一个 issue 一个 vertical slice 去实现。两个模型之间不靠聊天记录交接，只靠两样东西：GitHub Issues 和仓库里落盘的文档。

```text
Claude：需求拷问 -> PRD -> issue 拆分
  ↓
GitHub Issues + repo docs
  ↓
Codex：冷启动实现 -> 测试 -> commit
  ↓
Claude：验收 -> push -> close
```

GitHub：`https://github.com/sleepingF0x/spec-flow`

## 为什么要拆成两个模型

单个 AI coding 会话最舒服的地方，也是它最危险的地方：上下文一直在。

你前面随口补的一句话，后面模型可能记得；你没有写进 issue 的边界，模型也可能因为刚聊过而知道。这样做 demo 很快，但它不容易复现。换一个新会话、换一个人、换一个模型，任务能不能继续推进，就很难说。

还有一个很现实的原因：Claude 这边用的是 Fable 5，它贵。

需求阶段和验收阶段需要的是判断力，token 花在这里比较值。实现阶段会反复读文件、跑测试、修小错，如果让 Fable 5 全程烧在这些循环上，成本很快就不舒服。Codex 更适合吃掉这部分执行成本，Fable 5 留在规格和裁决这两端。

spec-flow 里故意让 Codex 冷启动。每个 issue 都开一个新会话，让它只读 issue、`AGENTS.md`、`docs/agents/`、ADR、glossary 和当前代码。这样可以逼出一个很朴素的事实：如果一个实现者必须依赖原来那段聊天记忆才能做对，说明任务本身还没有写清楚。

这不是为了增加仪式感，而是为了把失败归因变清楚。

- Codex 做不出来，优先怀疑 issue 不够 AFK-ready。
- 测试绿但验收不通过，说明验收标准或行为验证还不够具体。
- Claude 验收时发现方向跑偏，说明 PRD 阶段没有把关键决策固定下来。

角色拆开以后，流程变慢一点，但错误更容易定位。

## 需求阶段：先把模糊需求压成 spec

spec-flow 的第一阶段在 Claude 里完成，一个 feature 对应一个 Claude 会话。

我会先用 `/grill-with-docs` 被追问一轮。这里的重点不是让模型多问几个问题，而是把真正会影响实现的分支提前摊开：这个功能服务谁、成功标准是什么、有哪些不能碰的边界、哪些术语必须统一、哪些架构决定后面不应该反复摇摆。

这一步会把两类东西写进 repo：

- 术语和共享上下文进 `CONTEXT.md` 或 domain docs。
- 架构决定进 `docs/adr/`。

为什么要落盘？因为聊天记录不是工程资产。它对当前会话有用，但对下一个冷启动 agent、下一次 code review、三周后的自己都不够可靠。

需求问清楚以后，再用 `/to-prd` 把当前对话综合成 PRD issue。这个 issue 不是随便记一段想法，而是后续任务拆分的母版：目标、非目标、验收标准、测试 seam、风险和边界都应该写进去。

最后用 `/to-issues` 拆成 vertical-slice issues。这里我更看重“能独立验证”，而不是“看起来任务很小”。一个好的 issue 应该让 Codex 拿到后可以自己读上下文、自己写测试、自己实现、自己提交，但不需要猜需求。

## 实现阶段：Codex 只拿 issue 干活

实现阶段每个 issue 开一个新的 Codex 会话：

```bash
codex "/implement #N"
```

或者非交互跑：

```bash
codex exec --sandbox danger-full-access "/implement #N"
```

Codex 会读 issue、项目文档和代码，然后走 TDD、typecheck、code review，最后只 commit，不直接 push。

这里有个很重要的边界：Codex 可以指出需求缺口，但不应该在实现阶段偷偷重写父 PRD。它的责任是把当前 issue 做到可以验收。如果 issue 写得含糊，它应该暴露问题，而不是靠猜测把事情糊过去。

这也是为什么我倾向于一个 issue 一个新会话。冷启动会让上下文缺口变得很明显。只要 Codex 频繁问“这个到底是什么意思”，或者实现和预期总是偏一点，通常不是模型不努力，而是 issue 没有达到可执行规格的密度。

## 验收阶段：不要让实现者给自己发通过

Codex commit 以后，回到原来的 Claude feature 会话跑：

```text
/accept-issue #N
```

验收和测试不是一回事。

测试看的是代码有没有满足你写出来的断言；验收看的是这个 issue 是否真的满足当初聊出来的需求。Claude 负责验收，是因为它保留了 feature 级别的意图上下文，也没有参与刚才那轮实现。

`/accept-issue` 会做三件事：

- 跑必要的 verify。
- 实际驱动一遍行为，不只看测试绿。
- 对照验收标准逐条裁决。

如果通过，再接 `/land-issue #N`，push、在 issue 里留下验收摘要并 close。人工只需要扫报告，确认方向没有跑偏。

如果不通过，Claude 会把“期望 vs 实际”和复现步骤写回 issue 评论，让下一个 Codex 会话继续 `/implement #N`。这比在同一个长会话里反复修更稳，因为失败信息也变成了 issue 的一部分。

如果验收标准本身有问题，就不要硬判通过或不通过。那是 spec 的 bug，需要回到需求层修正。

## 真正重要的是交接面

spec-flow 表面上看是 Claude + Codex 的组合，但它真正依赖的不是某个模型版本，而是交接面。

GitHub Issues 是任务通道。它负责告诉实现者：这次只做什么、怎么判断完成、依赖什么、不能碰什么。

repo docs 是上下文通道。它负责保存那些不应该散落在聊天里的东西：领域术语、架构决策、项目约定、验证命令。

commit 是实现输出。Codex 只把代码推进到可验收状态，不把“已经写完”直接等同于“已经接受”。

这三个东西加起来，才让 AI coding 从一段对话变成一个可复盘的工程流程。

```text
聊天里的意图
  -> PRD issue
  -> vertical-slice issues
  -> 冷启动实现
  -> 独立验收报告
  -> landed commit
```

每一步都有一个可检查的中间物。这样就算某一步失败，也知道该回头修哪里。

## 它适合什么场景

spec-flow 不适合所有任务。

如果只是改一个 typo、补一个 alias、修一个很明确的小 bug，直接让一个 agent 做完就好。为这种任务开 PRD、拆 issue、跨模型验收，成本太高。

它更适合几类场景：

- 需求一开始是模糊的，需要先被拷问清楚。
- 功能会拆成多个 issue，前后有依赖关系。
- 项目里有领域术语、架构边界或验证步骤，不能只靠模型临场发挥。
- 你希望实现结果可以被冷启动复现，而不是只在当前聊天里成立。
- 你想把验收从“模型说做完了”变成一份可以回看的报告。

代价也很明确。Fable 5 贵，所以它要尽量花在需求和验收上；Codex 高推理档也吃 token，但用来烧实现循环更合理。issue 写不好时，流程会不断把问题打回规格层；前几个 issue 的验收报告最好人工认真读，因为那是校准 issue 写法最直接的反馈。

但这正是 spec-flow 的价值：它会让模糊暴露得更早。

以前我更容易把失败归因给模型：它没懂、它写偏了、它又偷懒了。跑这套流程以后，很多问题会回到更基础的地方：我有没有把需求讲清楚，有没有把验收标准写到能执行，有没有把项目约定沉淀到 repo 里。

AI 编程最后拼的不只是模型能力，而是你能不能把意图变成规格，把规格变成任务，把任务变成可验证的代码。

spec-flow 只是把这条链路显式化。
