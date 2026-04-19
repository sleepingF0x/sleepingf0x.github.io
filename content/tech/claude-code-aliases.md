---
title: "Claude Code：我在用的几个 alias"
date: 2026-04-19T03:10:00+08:00
draft: false
description: "记录我给 Claude Code 和 Codex 配的几个常用命令别名。"
tags: ["Claude Code", "CLI", "Shell", "Workflow"]
series: ["Claude-Code-Guide"]
---

日常用 Claude Code / Codex时，我会先配几条 alias，跳过频繁的权限确认。

## 配置

```bash
alias ccd="claude --dangerously-skip-permissions"
alias codex="codex --ask-for-approval never --sandbox danger-full-access"
```