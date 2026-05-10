---
title: "Claude for Financial Services"
date: 2026-05-10T00:00:00+08:00
draft: false
description: "介绍 Anthropic 的 Claude for Financial Services 项目，适合谁、能做什么、怎么用。"
tags: ["Claude", "Agent", "Financial Services", "Workflow"]
series: ["Claude-Code-Guide"]
---

`Claude for Financial Services` 是 Anthropic 提供的一套金融行业 Claude Agent、Skill 和插件参考库。它面向的不是某一个单点任务，而是金融团队每天都会遇到的材料整理、模型搭建、分析初稿和检查流程。

它适合经常和 Excel、PDF、财报、电话会 transcript、PPT 模板、市场数据、内部资料打交道的人。比如投行、企业融资、PE、二级研究、资管、财富管理、基金运营、财务运营，以及 onboarding / KYC 支持团队。

这类工作的共同点是输入材料多、格式要求高、人工重复步骤多。Claude 在这里承担的是起草和整理角色：先把资料读进去，按金融工作流生成一版可审核的输出，再交给专业人员判断、修改和批准。

## 适用谁

如果你经常需要产出 pitch book、估值模型、研究笔记、IC memo、对账说明或 KYC 审查结果，这个项目就比较对口。

投行和 PE 团队可以用它处理 pitch、估值、交易材料和投资备忘录；二级研究和资管团队可以用它整理财报、电话会、filings 和模型更新；基金运营和财务团队可以用它辅助对账、月结和 LP statement 审核；KYC 或 onboarding 团队可以用它做文件检查和缺口整理。

它不要求所有人都写代码。个人试用时，可以把它当作 Claude Code 或 Cowork 插件来用；企业落地时，也可以把同一套 Agent 接到内部系统、数据源和审批流里。

## 能做什么

项目里有两类能力。

一类是完整 Agent，适合跑一段端到端工作流。比如从公司资料出发生成 pitch deck 草稿，或者从财报和电话会出发起草 earnings note。

另一类是垂直插件和 slash command，适合处理更具体的任务，比如 `/comps`、`/dcf`、`/lbo`、`/earnings`、`/ic-memo`。这些命令更像是金融分析里的单项工具，用的时候给它材料和目标，它按对应方法产出初稿。

可以重点看四个代表 Agent。

## Pitch Agent

Pitch Agent 适合投行、PE 和企业融资团队准备 pitch 材料。

它可以基于公司资料、财务数据、行业信息和模板，辅助完成可比公司分析、precedent transactions、LBO、估值区间和 pitch deck 草稿。对 deal team 来说，它更像一个能先把材料铺好的分析师助手，最终版本仍然需要 banker 或项目组审核。

## Earnings Reviewer

Earnings Reviewer 适合二级研究和资管团队。

它可以读取财报、filings 和 earnings call transcript，提取收入、利润率、guidance、管理层表述和关键 KPI 变化，辅助更新模型并起草 earnings note。它适合做第一版整理，把事实、变化和初步观点放到同一个框架里，但不应该直接替代投资结论。

## GL Reconciler

GL Reconciler 面向基金运营和财务运营团队。

它可以帮助查找总账对账差异，追踪 break 的可能原因，整理 reconciliation 结果和待处理项。它的价值在于减少人工查表和解释差异的时间，最后的入账、调整和审批还是要走原有财务流程。

## KYC Screener

KYC Screener 面向 onboarding、运营和合规支持团队。

它可以解析开户文件，根据规则检查缺失项、异常点和需要补充的信息，生成给人工复核的 gap list。它可以做前置筛查和材料整理，但不能直接批准客户准入。

## 边界在哪里

金融场景里边界很重要。这个项目不应该被当成自动交易、投资建议、风控审批、会计入账或 KYC 最终批准系统。

README 里也明确说，所有输出都应该进入 human sign-off 流程。Claude 可以帮你起草、整理、检查和建模，专业判断、合规责任和对外发布仍然在人手里。

## 个人怎么试用

个人或小团队试用时，可以先把它作为 Claude Code 插件安装。

```bash
claude plugin marketplace add anthropics/claude-for-financial-services
claude plugin install financial-analysis@claude-for-financial-services
```

然后按任务使用命令：

```text
/comps 帮我基于这些公司做可比公司分析
/dcf 基于这份财务预测做 DCF 估值框架
/earnings 分析这份财报电话会 transcript，起草 earnings note
```

如果你做投行材料，可以再装 `investment-banking`；如果做二级研究，可以装 `equity-research`；如果想跑完整流程，可以安装对应 Agent，比如 `pitch-agent`、`market-researcher`、`gl-reconciler`。

## 企业怎么落地

企业落地时，重点不是命令行，而是把这些 Agent 接进自己的工作流。

项目提供了 Managed Agents API 的模板，可以部署到内部投研平台、财务运营系统、KYC onboarding 系统，或者 Excel / PowerPoint 环境里。MCP connectors 则负责连接 Morningstar、FactSet、S&P Global、LSEG、PitchBook、Egnyte 等外部数据或文档系统，不过通常需要企业订阅和权限配置。

如果只是想体验，先从 `financial-analysis` 开始就够了；如果已经有明确业务场景，再安装对应 vertical plugin 或 Agent。这个项目最值得看的地方，是它把金融行业里重复、耗时、格式化强的工作，拆成了 Claude 可以执行、也便于人工审核的工作流。
