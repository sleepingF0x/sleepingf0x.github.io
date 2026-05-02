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

Matt Pocock 是 Total TypeScript 的作者，这个仓库整理的是他自己日常用的 Claude skills。整体风格很工程化，不追求让 agent 更会“发挥”，而是给它加流程：先把问题问清楚，再设计接口、拆 issue、写测试、做重构计划，尽量避免一上来就改代码。

GitHub：`https://github.com/mattpocock/skills`

### Planning & Design

- `to-prd`：把当前对话和代码库理解整理成 PRD，并创建 GitHub issue。适合需求已经聊得差不多，但还没有沉淀成正式规格的时候用。
- `to-issues`：把 PRD、计划或规格拆成多个可独立领取的 GitHub issues。它强调纵向切片，每个 issue 都应该能独立验证，而不是按数据库、API、UI 这种横向层拆。
- `grill-me`：让 Claude 反复追问你的方案，直到关键边界和取舍都讲清楚。适合方案还不稳、担心漏掉分支条件的时候用。
- `design-an-interface`：为一个模块生成多种明显不同的接口设计，再比较取舍。适合 API、模块边界、service 接口还没定型的时候用。
- `request-refactor-plan`：通过访谈生成一个重构计划，并拆成很小的提交步骤。适合不想让 Claude 直接大改，而是先把重构范围和验证方式讲清楚。
- `domain-model`：围绕领域模型拷问方案，并把确定下来的术语和决策写进 `CONTEXT.md` 或 ADR。适合业务概念复杂、术语容易混乱的项目。

### Development

- `tdd`：按 red、green、refactor 的节奏开发功能或修 bug。它不鼓励一次写完所有测试，而是一次一个行为，一条测试推动一小段实现。
- `triage-issue`：调查 bug 根因，并创建带 TDD 修复计划的 GitHub issue。适合先查清楚问题，再把修复交给人或 agent 执行。
- `improve-codebase-architecture`：找代码库里可以“加深”的模块，也就是通过收窄接口、隐藏复杂度来改善模块边界。适合做架构梳理、降低耦合、改善可测试性。
- `migrate-to-shoehorn`：把 TypeScript 测试里的 `as` 类型断言迁移到 `@total-typescript/shoehorn`。它只适合测试代码，主要解决测试数据很大但只关心少数字段的问题。
- `scaffold-exercises`：为课程练习创建目录结构、题目、答案和讲解文件。这个比较贴作者自己的课程项目，通用性不如前面几个。

### Tooling & Workflow

- `github-triage`：用 label 状态机管理 GitHub issues，比如 `needs-triage`、`needs-info`、`ready-for-agent`、`ready-for-human`。适合开源项目或 issue 流量比较大的项目。
- `qa`：开启一个 QA 会话，用户边测试边描述问题，Claude 负责轻量追问、整理上下文、创建 GitHub issue。适合把口头反馈快速沉淀成可执行任务。
- `setup-pre-commit`：给 JS/TS 项目配置 Husky、lint-staged、Prettier、typecheck 和 test。适合项目还没有提交前检查机制的时候用。
- `git-guardrails-claude-code`：给 Claude Code 配 PreToolUse hook，阻止危险 git 命令，比如 `git push`、`git reset --hard`、`git clean -f`、`git branch -D`。适合想给 agent 加本地安全护栏的时候用。

### Writing & Knowledge

- `edit-article`：编辑文章，先按标题拆分结构，再逐段改清晰度和表达。它关注信息依赖顺序，避免文章前面用了后面才解释的概念。
- `ubiquitous-language`：从当前对话里提取 DDD 风格的统一语言表，并保存成 `UBIQUITOUS_LANGUAGE.md`。适合把业务术语、别名和概念关系固定下来。
- `write-a-skill`：创建新的 Claude skill。它会要求写清楚触发场景、目录结构、是否需要脚本、哪些内容应该拆到 reference 文件里。
- `obsidian-vault`：管理作者自己的 Obsidian vault，包括搜索笔记、创建笔记、维护 wikilinks 和 index notes。默认路径很个人化，真要用需要改成自己的 vault。

### Communication & Navigation

- `caveman`：极简沟通模式。触发后 Claude 会去掉客套、铺垫和冗余解释，只保留技术信息，适合想减少上下文消耗或只要结论的时候用。
- `zoom-out`：让 Claude 从当前代码细节里退出来，先画模块地图、调用关系和上下文。适合不熟某块代码，或者 Claude 钻得太细时用。

### 我会优先装的几个

日常开发里，最值得先装的是 `grill-me`、`design-an-interface`、`tdd`、`to-prd`、`to-issues`、`triage-issue` 和 `zoom-out`。这几个覆盖了需求澄清、接口设计、测试驱动、任务拆分和陌生代码理解，使用频率会比较高。

如果在维护开源项目，再考虑 `github-triage` 和 `qa`。如果项目需要更强的本地约束，可以加 `setup-pre-commit` 和 `git-guardrails-claude-code`。剩下几个更偏作者自己的工作流，按场景安装就行。
