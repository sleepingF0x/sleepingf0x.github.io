---
title: "Skills：最近常用的几个skill"
date: 2026-04-19T02:20:00+08:00
draft: false
description: "这里记录最近常用的几个skill"
tags: ["Claude Code", "Skill", "Workflow"]
series: ["Claude-Code-Guide"]
---

## andrej-karpathy-skills

Andrej Karpathy 曾是 OpenAI 早期成员，也在 Tesla 负责过 AI 相关工作，对大模型和工程实践都有很深的理解。这个 skill 很像是把他对 AI 编码的思考，整理成一套更可复用的行为规则：先理解问题，再动手修改；优先选择简单方案，避免过度设计；只改真正需要改的部分，尽量减少对现有系统的扰动。

GitHub：`https://github.com/forrestchang/andrej-karpathy-skills`

## everything-claude-code

这个仓库把自己定义为一个面向 Claude Code、Codex、Cursor、OpenCode 等工具的“agent harness performance optimization system”，覆盖 skills、memory、security、hooks、rules 和 research-first development, 将这些能力整合成一个完整的系统。

作者本人长期在 Claude Code 和自动化系统方向实践，这套配置是作者真实开发中持续打磨出来的一套 AI coding workflow。

总结：非常全面，如果在做开发时不知道装什么，那就选择这个吧。个人会觉得有些heavy了，会选择Manual Installation，挑一些我自己用到的部份。

GitHub：`https://github.com/affaan-m/everything-claude-code`

## superpowers

Superpowers 不是单个 skill，更像是一套给 coding agent 用的软件开发方法论。它从 agent 启动时就开始接管流程：先确认你到底要做什么，再把需求拆成可读的设计说明；设计确认后，继续拆成足够细的执行计划；真正写代码时，用 TDD、代码评审、验证和分支收尾约束 agent，不让它凭感觉一路改下去。

它的核心价值不是多装几个命令，而是让 coding agent 形成固定工作流。比如新功能先 `brainstorming`，再 `writing-plans`；实现阶段走 `test-driven-development`；复杂任务交给 `subagent-driven-development` 或 `executing-plans`；做完后用 `requesting-code-review` 和 `verification-before-completion` 检查，最后用 `finishing-a-development-branch` 收尾。

GitHub：`https://github.com/obra/superpowers`

### 基础流程

- `brainstorming`：需求还不清楚时用，先确认目标、边界、取舍和备选方案。
- `using-git-worktrees`：方案确认后用，给任务开独立 worktree，避免污染当前工作区。
- `writing-plans`：把已确认的方案拆成具体任务，每一步要有文件路径、操作说明和验证方式。
- `subagent-driven-development`：按计划派发子 agent 执行任务，适合步骤多、上下文容易混乱的开发。
- `executing-plans`：按计划分批执行，适合需要人工检查点的任务。
- `test-driven-development`：实现功能或修 bug 前用，强制走 red、green、refactor。
- `requesting-code-review`：阶段完成后用，先 review 再继续推进。
- `finishing-a-development-branch`：整条开发分支完成后用，做最终验证、合并或 PR 决策、清理 worktree。

### Skills Library 分类

**Testing**

- `test-driven-development`：写实现前先写失败测试，避免 agent 先写一堆没有验证的代码。

**Debugging**

- `systematic-debugging`：遇到报错、失败测试、异常行为时用，按现象、模式、假设、验证的顺序查根因。
- `verification-before-completion`：准备说完成前用，要求拿证据证明问题真的修好，而不是只看起来修好。

**Collaboration**

- `brainstorming`：把模糊想法整理成可执行方案。
- `writing-plans`：把方案写成任务计划。
- `executing-plans`：按计划执行，保留人工检查点。
- `dispatching-parallel-agents`：多个独立任务可以并行时用，减少串行等待。
- `subagent-driven-development`：让子 agent 按任务推进，并做规格符合性和代码质量两轮检查。
- `requesting-code-review`：请求代码评审，按严重程度处理问题。
- `receiving-code-review`：收到 review 意见后用，先理解问题，再决定是否修改。
- `using-git-worktrees`：为并行开发创建隔离分支。
- `finishing-a-development-branch`：开发分支收尾，确认测试、合并、PR 或丢弃策略。

**Meta**

- `using-superpowers`：介绍 Superpowers 的技能系统，强调相关 skill 必须先调用。
- `writing-skills`：创建或修改 skill 时用，保证新 skill 可测试、可复用、能跨 coding agent 工作。

### 常用命令

- `/brainstorming 我想给博客加站内搜索，先把方案讲清楚`
- `/writing-plans 基于刚才确认的方案，拆成可执行步骤`
- `/using-git-worktrees 给这个功能开一个独立 worktree`
- `/test-driven-development 为这个功能先写失败测试`
- `/systematic-debugging 首页构建时报 RSS 错，先查根因`
- `/dispatching-parallel-agents 这几个互不依赖的检查并行跑`
- `/subagent-driven-development 按计划派发子任务实现`
- `/executing-plans 按计划执行，阶段之间停下来让我确认`
- `/requesting-code-review review 当前改动`
- `/receiving-code-review 处理刚才的 review 意见`
- `/verification-before-completion 确认这个问题是不是真的修好了`
- `/finishing-a-development-branch 当前分支做最终收尾`
- `/writing-skills 帮我写一个新的 skill`

### workflow

- `/requesting-code-review 描述你的 GOAL，请仔细审核这份代码`
- `/systematic-debugging 找到问题的根因，尝试定位到具体代码。思考解决方案，再列出改进plan，写完后交给subagent再次检查，直到合格为止`


## mattpocock/skills

Matt Pocock 是 Total TypeScript 的作者，这个仓库整理的是他自己每天在用的 Claude skills。现在 README 的定位更明确了：这些 skills 不是为了 vibe coding，而是把真实工程里的基本功拆成小而可组合的流程，方便你按项目需要改造。

它和 GSD、BMAD、Spec-Kit 这类更重的流程不太一样。Matt 的思路不是让一套框架接管全部开发过程，而是把常见失败模式拆开处理：需求没对齐，就 grilling；术语混乱，就沉淀共享语言；代码跑不通，就补反馈环；代码变成泥球，就重新看模块边界。

GitHub：`https://github.com/mattpocock/skills`

### Engineering

- `setup-matt-pocock-skills`：每个项目先跑一次，用来配置 issue tracker、triage labels、文档保存位置和领域文档布局。现在不少工程类 skills 都会依赖这份配置。
- `grill-with-docs`：带文档沉淀的需求拷问。它会基于现有 domain model 追问方案，同时更新 `CONTEXT.md` 和 ADR。适合业务术语多、决策容易散在对话里的项目。
- `to-prd`：把当前对话上下文整理成 PRD，并提交成 GitHub issue。它不再做长访谈，更像是把已经聊清楚的内容正式落文档。
- `to-issues`：把 PRD、计划或规格拆成能独立领取的 issues。重点还是纵向切片，每个 issue 都应该能单独验证。
- `triage`：用一套 triage role 状态机处理 issues。新版不再叫 `triage-issue`，更偏 issue 流转和分工管理。
- `tdd`：按 red、green、refactor 节奏开发功能或修 bug。它强调一次一个 vertical slice，不鼓励一次性写一大堆测试。
- `diagnose`：诊断 bug 和性能问题的流程，顺序是复现、最小化、假设、插桩、修复、补回归测试。这个比直接让 agent 猜原因稳很多。
- `zoom-out`：让 agent 从局部代码里退出来，先解释模块关系、调用路径和系统背景。适合接手陌生代码，或者发现 agent 钻进局部细节出不来时用。
- `improve-codebase-architecture`：寻找可以加深的模块边界，结合 `CONTEXT.md` 和 ADR 判断哪些接口该收窄。适合定期处理 agent 写代码带来的复杂度膨胀。
- `prototype`：做一次性原型，用来验证状态流、业务逻辑或 UI 方向。它明确把原型当成临时设计工具，不是直接拿来进主线。

### Productivity

- `grill-me`：不涉及代码时用的需求拷问。它会持续追问你的计划，直到关键分支和取舍都讲清楚。
- `caveman`：极简沟通模式，去掉客套和解释，只保留技术信息。适合上下文吃紧，或者你只想要结论的时候用。
- `write-a-skill`：创建新的 Claude skill。它会要求写清楚触发场景、目录结构、脚本需求，以及哪些内容该拆到 reference 文件里。

### Misc / Personal

- `git-guardrails-claude-code`：给 Claude Code 配 hooks，阻止 `git push`、`git reset --hard`、`git clean` 这类危险命令。适合想给 agent 加本地安全护栏的时候用。
- `setup-pre-commit`：配置 Husky、lint-staged、Prettier、typecheck 和 tests。适合项目还没有提交前检查机制的时候用。
- `migrate-to-shoehorn`：把测试文件里的 `as` 类型断言迁移到 `@total-typescript/shoehorn`。它只适合 TypeScript 测试代码。
- `scaffold-exercises`：创建课程练习目录、题目、答案和讲解文件，更贴 Matt 自己的课程工作流。
- `edit-article` 和 `obsidian-vault`：放在 personal 目录下，明显偏作者个人使用习惯。除非你的写作流程和 Obsidian vault 结构接近，否则不需要优先装。

### 已经 deprecated 的几个

`design-an-interface`、`request-refactor-plan`、`qa`、`ubiquitous-language` 已经移到 deprecated 目录。旧文章里我原本把它们当成主力技能，现在看应该降级处理。接口设计和领域语言这两块并没有消失，而是被并进了 `grill-with-docs`、`tdd`、`improve-codebase-architecture` 这些更完整的流程里。

### 我会优先装的几个

日常开发里，最值得先装的是 `setup-matt-pocock-skills`、`grill-with-docs`、`tdd`、`diagnose`、`to-prd`、`to-issues`、`zoom-out` 和 `improve-codebase-architecture`。这几个覆盖了需求澄清、文档沉淀、测试驱动、问题诊断、任务拆分、陌生代码理解和架构收敛。

如果项目里 issue 流量比较大，再加 `triage`。如果经常需要验证设计方向，可以加 `prototype`。本地安全和提交质量这块，可以按需装 `git-guardrails-claude-code` 和 `setup-pre-commit`。
