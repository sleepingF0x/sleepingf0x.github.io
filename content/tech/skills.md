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
