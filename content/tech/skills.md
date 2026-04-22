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

Superpowers 不是单个 skill，更像是一套给 coding agent 用的软件开发流程。它把需求澄清、方案确认、任务拆分、TDD、代码评审和收尾这些环节串起来，让 agent 少一点“想到什么就改什么”，多一点按阶段推进的纪律。

GitHub：`https://github.com/obra/superpowers`

### 流程介绍

- `brainstorming`：刚提出需求，还在确认目标、边界和取舍
- `writing-plans`：方案已经定了，开始拆执行步骤
- `test-driven-development`：准备动手实现，需要先写失败测试
- `requesting-code-review`：某个阶段做完，准备继续往下走
- `verification-before-completion`：准备宣告完成前，先确认是不是真的完成了
- `finishing-a-development-branch`：整条开发分支收尾，统一做测试和清理

比如可以这样直接用：

- `/brainstorming 我想给博客加站内搜索，先把方案讲清楚`
- `/writing-plans 基于刚才确认的方案，拆成可执行步骤`
- `/test-driven-development 为这个功能先写失败测试`
- `/systematic-debugging 首页构建时报 RSS 错，先查根因`
- `/requesting-code-review review 当前改动`
- `/verification-before-completion 确认这个问题是不是真的修好了`