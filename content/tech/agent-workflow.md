---
title: "agent workflow：把 AI 编程从对话变成流程"
date: 2026-07-05T00:00:00+08:00
lastmod: 2026-07-12T00:00:00+08:00
draft: false
description: "介绍一套双模型 spec-driven 开发工作流：Claude 管需求、复审和流程把关，Codex（gpt-5.6-sol）管实现和验收判定，用 GitHub Issues 和落盘文档交接，推理档位按 spec 确定性选；一个会话装不下的活先用 wayfinder 在 tracker 上建图。"
tags: ["Claude Code", "Codex", "AI Coding", "Workflow", "Spec"]
series: ["Claude-Code-Guide"]
aliases: ["/tech/spec-flow/"]
---

我最近在整理一套 spec-driven 的 AI coding 流程，叫 agent workflow。

它解决的不是“哪个模型写代码更强”，而是另一个更实际的问题：需求、实现、验收如果都挤在同一个长对话里，最后很容易分不清到底是 spec 模糊、实现跑偏，还是验收只是在迁就已经写出来的代码。

agent workflow 的做法很简单：把角色拆开。

Claude 负责把模糊需求拷问成 spec 和可执行 issue，实现时负责钉死 spec、复审和验证；写代码的活交给 Codex 的 gpt-5.6-sol，验收判定也由它在另一个干净线程里做。两个模型之间不靠聊天记录交接，只靠两样东西：GitHub Issues 和仓库里落盘的文档。

```text
Claude：需求拷问 -> spec -> 拆 tickets
  ↓
GitHub Issues + repo docs
  ↓
Claude 钉 spec、选推理档 -> Codex (gpt-5.6-sol) 实现 -> Claude 验证 + review -> commit
  ↓
新会话验收：Codex 逐条判定 -> Claude push + close
```

下面提到的 skill 我都放在这个仓库里，多台机器之间同步用：`https://github.com/sleepingF0x/skills`

## 为什么要拆成两个模型

单个 AI coding 会话最舒服的地方，也是它最危险的地方：上下文一直在。

你前面随口补的一句话，后面模型可能记得；你没有写进 issue 的边界，模型也可能因为刚聊过而知道。这样做 demo 很快，但它不容易复现。换一个新会话、换一个人、换一个模型，任务能不能继续推进，就很难说。

还有一个很现实的原因：Claude 这边用的是 Fable 5，它贵。

需求阶段需要的是判断力，token 花在这里比较值。实现阶段会反复读文件、跑测试、修小错，如果让 Fable 5 全程烧在这些循环上，成本很快就不舒服。Codex 更适合吃掉这部分执行成本，Fable 5 留在规格和把关这一端。

agent workflow 里故意让 Codex 冷启动。每个 issue 都开一个全新线程，它能看到的只有 Claude 打包给它的 brief：issue 原文、红测、仓库约定、验证命令。这样可以逼出一个很朴素的事实：如果一个实现者必须依赖原来那段聊天记忆才能做对，说明任务本身还没有写清楚。

这不是为了增加仪式感，而是为了把失败归因变清楚。

- Codex 做不出来，优先怀疑 issue 不够 AFK-ready。
- 测试绿但验收不通过，说明验收标准或行为验证还不够具体。
- 验收判定发现方向跑偏，说明 spec 阶段没有把关键决策固定下来。

角色拆开以后，流程变慢一点，但错误更容易定位。

## 上游：Matt Pocock 的 skills

agent workflow 的骨架不是我发明的，它建在 Matt Pocock 的 skills 上。他仓库里有个 `/ask-matt`，是一台路由，把所有 skill 的关系写死了。主线只有一条：

```text
grill-with-docs -> to-spec -> to-tickets -> implement
```

烤问在最前面，spec 和拆单在中间，implement 一张工单一张工单地做，每张开一个新的上下文。前三步必须待在同一个上下文窗口里，到 to-tickets 之前不许 compact，因为烤问、spec、拆单要建在同一段思考上。

主线之外有两条 on-ramp，都汇进主线：一条是 wayfinder，活大到一个会话装不下、路线又看不清时从它进，雾散完在 to-spec 汇入；另一条是 triage，外部进来的 bug 报告和 feature request 先过它的状态机，产出的 ready-for-agent issue 交给 implement。这两条各占后面一节。

我在主线上改了两处：implement 换成 `/implement-codex`，把写代码的活委派出去；链条尾部补了 accept-issue 和 land-issue，管验收和落地，这两个 Matt 的仓库里没有。

先说 wayfinder。它解决的是另一个量级的问题：活大到一个会话装不下，而且路线本身还看不清。

这种活直接 grill 不够用。一次烤问的产出活在当前会话里，会话结束，没聊完的分支就丢了。wayfinder 把整个探索过程搬到 issue tracker 上，变成一张持久化的地图：

- 地图是一个 issue（标签 `wayfinder:map`），正文写 Destination（这张图要走到的终点：一份 spec、一个决策、或者一次迁移）、已敲定决策的索引、还看不清的部分。决策的细节留在各自的工单里，地图只做一行摘要加链接。
- 每个待决问题是一张子 issue，分四种类型：research（AFK 查资料）、prototype（做个能戳的原型来对齐）、grilling（烤问，默认类型）、task（为解锁决策而做的杂务，比如先注册个服务拿到 API 才能评估它）。
- 依赖用 tracker 原生的 blocked-by 表示。开着的、没被阻塞的、没人认领的工单就是 frontier，任何一个新会话都能领一张接着走。

调用分两种模式：丢给它一个模糊想法是制图，先烤问钉死 Destination，再广度优先扫出首批工单和迷雾，制图本身占满一个会话；给它 map 编号是走图，领一张 frontier 工单解掉。

两条纪律最关键。一是每个会话只解一张工单，解完把答案评论回工单、关单、在地图上留一行索引，然后停手。二是战争迷雾：能精确问出来的问题才开工单，只有模糊感觉的写进地图的 Not yet specified 区，等前面的工单解完、雾散了再升格。不预切迷雾，因为一块雾散开后可能变成三张工单，也可能什么都不是。

wayfinder 是规划不是执行。地图的完成标志是到终点的路完全清晰、没有待决问题，这时候把结论交给 to-spec 固化成 spec，进入下面几节的流程。中途手痒想直接开写，多半就是该收图交棒的信号。

用不用它，我的标准是两个条件都满足：多会话，多雾。已经拆清楚的一串 issue 不需要它，直接 implement；但“要不要为一个每天只有几美元的市场上付费基础设施”这种问题，牵扯调研、开户、成本核算，甚至可能中途改写终点，就值得先建图。

## 需求阶段：先把模糊需求压成 spec

agent workflow 的第一阶段在 Claude 里完成，一个 feature 对应一个 Claude 会话。

我会先用 `/grill-with-docs` 被追问一轮。这里的重点不是让模型多问几个问题，而是把真正会影响实现的分支提前摊开：这个功能服务谁、成功标准是什么、有哪些不能碰的边界、哪些术语必须统一、哪些架构决定后面不应该反复摇摆。

这一步会把两类东西写进 repo：

- 术语和共享上下文进 `CONTEXT.md` 或 domain docs。
- 架构决定进 `docs/adr/`。

为什么要落盘？因为聊天记录不是工程资产。它对当前会话有用，但对下一个冷启动 agent、下一次 code review、三周后的自己都不够可靠。

需求问清楚以后，再用 `/to-spec` 把当前对话综合成 spec issue，也就是通常说的 PRD。这个 issue 不是随便记一段想法，而是后续任务拆分的母版：目标、非目标、验收标准、测试 seam、风险和边界都应该写进去。

最后用 `/to-tickets` 拆成 vertical-slice issues。这里我更看重“能独立验证”，而不是“看起来任务很小”。一个好的 issue 应该让 Codex 拿到后可以自己读上下文、自己写测试、自己实现、自己提交，但不需要猜需求。这一步的质量后面会直接变成账单：issue 写得越死，实现阶段需要的推理档位就越低。

拆单时还要给每张工单标出被谁阻塞，用 tracker 原生的 blocked-by。这样开着的、没被阻塞的那几张就是可以马上开工的一批，冷启动会话进来一眼就能领。

## 另一扇门：triage

`/to-spec` 和 `/to-tickets` 发单时会自己贴上 `ready-for-agent`。这些 issue 是从烤问、spec、我确认过的垂直切片一路走下来的，出生就在水位线上，再 triage 一遍是重复劳动。Matt 在 `/ask-matt` 里把这条写死了：triage 只处理不是你自己建的 issue，to-tickets 拆出来的工单不要 triage。

triage 服务的是另一扇门进来的 issue：外部的 bug 报告、别人提的 feature request、我自己随手记的一句话、needs-info 之后报告人又回复了的。这些东西质量未知，不能直接扔给 Codex。

`/triage` 是一台状态机。类别只有 bug 和 enhancement 两种，状态有五个：needs-triage、needs-info、ready-for-agent、ready-for-human、wontfix。没标签的 issue 先落到 needs-triage，再往下走。

处理一条 issue 的顺序是固定的。先读全文和历史评论，去代码库里按领域术语搜一遍这个功能是不是已经实现了，再翻 `.out-of-scope/` 看这个请求以前是不是拒过。是 bug 就按报告人的步骤实际复现一次，复现不出来就是 needs-info 的强信号。需求写得含糊就拉进 `/grilling` 烤一轮，顺手把新术语写进 `CONTEXT.md`。最后才落标签：升格成 ready-for-agent 的必须附一份 agent brief，需要人做的写清楚为什么不能委派，拒掉的 enhancement 要把理由写进 `.out-of-scope/` 再关单，下次有人提同样的请求，翻记录就够了。两条路在 ready-for-agent 汇合，之后走同一套实现和验收。

还有一种情况会让 issue 逆行回 triage：某张 to-tickets 拆出来的工单，实现或验收阶段反复卡在“这到底什么意思”，说明它其实没到可执行规格的密度。把它降回 needs-triage 重新烤一轮，比在实现会话里硬猜稳。标签词汇是共享的，状态机接得住。

## 实现阶段：Claude 钉 spec，Codex 写代码

实现阶段在 Claude Code 里跑，每个 issue 一条命令：

```text
/implement-codex #N
```

这个 skill 是在 Matt Pocock 的 implement 基础上改的：不让 Claude 自己写代码，拆成钉 spec、选推理档、委派 Codex、自己验证四步。

Claude 先拉 issue 原文和全部评论，把歧义当场问清，能写红测的先在约定 seam 上把失败测试写好。红测是能交给实现者的最紧的 spec，比任何文字描述都硬。

然后选推理档位。这是我跑这套流程最大的一个体会：推理强度买的是对 spec 空白的推理能力，跟代码本身的质量关系不大。我的默认档是 `xhigh`，只有 spec 钉得足够死才降档：

- `xhigh`：默认档，实现类任务直接从这里起步。
- `high`：brief 完整、照着现有模式走、没有架构上的意外。
- `medium`：行为已经被红测完全钉死，纯机械的局部改动。

实现任务不用 `minimal` 和 `low`，省下的时间会在返工里加倍还回来。档位还必须每次显式传：我的 codex config 里默认是 `high`，比 `xhigh` 低一档，省掉 `-c model_reasoning_effort` 这个参数不会报错，只会让任务安安静静地跑在一个不够用的档上。

档位选好，Claude 把 brief 落成文件，后台直接跑 codex exec：

```bash
codex exec --sandbox danger-full-access \
  --model gpt-5.6-sol -c model_reasoning_effort='"xhigh"' \
  --output-last-message issueN-report.md \
  - < issueN-brief.md
```

一开始我走的是 codex 插件，后来放弃了：插件的沙箱写死在 workspace-write，没有网络，也够不到 Docker socket。我的项目测试都跑在容器里，Codex 在那个沙箱里等于蒙着眼实现，写完连自己的代码都没法验，每轮红绿循环都要弹回 Claude。

danger-full-access 意味着完全不设沙箱，约束全靠 brief 写死：只许改仓库里的文件和自己在 `/tmp` 下的草稿，不许 commit、不许 push、不许碰 issue tracker，工作树的 diff 是它唯一的输出。跑完 Claude 用 git status 兜底核查。

这里有个很重要的边界：Codex 可以指出需求缺口，但不应该在实现阶段偷偷重写父 spec。它的责任是把当前 issue 做到可以验收。如果 issue 写得含糊，它应该暴露问题，而不是靠猜测把事情糊过去。

Codex 写完，Claude 不信它的汇报，自己跑 typecheck 和全量测试。失败了不重开会话，resume 回原线程，档位显式提到 `xhigh` 之上的 `max`，把失败输出一起喂回去，上下文还在，比从零重跑省得多。

最后 `/code-review` 复审 diff。Claude 审 GPT 写的代码，两个模型的盲区不重叠，这是跨模型分工里最实在的一层收益。审完 commit，message 里引用 `#N`。到这里为止，不 push 不关单。

这一步有个死角，而且是它自己看不见的：Claude 审的是别人的代码，用的却是自己写的 spec。spec 要是从头就写歪了，让写 spec 的人再看一遍也看不出来。这个洞留给验收去堵。

issue 攒多了可以批量：`/implement-codex 处理所有 ready-for-agent 的 issue`。串行跑，一个 issue 一个新线程，失败两次的跳过并把发现评论回 issue，最后出一份队列报告。工作树只有一个，两个写任务并发只会互相踩。

冷启动会让上下文缺口变得很明显。只要 Codex 频繁问“这个到底是什么意思”，或者实现和预期总是偏一点，通常不是模型不努力，而是 issue 没有达到可执行规格的密度。

## 验收阶段：不要让实现者给自己发通过

Codex commit 以后，新开一个会话跑：

```text
/accept-issue #N
```

注意是新会话，不是回到实现会话，这是条硬规则。实现会话带着自己的合理化，“issue 显然就是这个意思”；冷启动的判定者只能从 issue 原文重新推导需求，验收要的就是这个。

验收和测试不是一回事。测试看的是代码有没有满足你写出来的断言；验收看的是这个 issue 是否真的满足当初聊出来的需求。

这一步的角色拆成法官和书记员。Codex 当法官，还是 gpt-5.6-sol，同样 codex exec 直跑、`xhigh` 起步，但换了个全新线程：跑 verify 入口、用测试没用过的输入实际驱动一遍行为、对照验收标准逐条裁决。每条判定必须带证据，跑了什么命令、看到了什么输出；判案期间不许改任何源码。

Claude 当书记员，只查形式不碰结论：working tree 必须干净，Codex 动了源码这次验收作废；判定缺证据就打回去补，不自己脑补；不许推翻法官的裁决，有异议只能把双方证据摆给我。

如果通过，Claude 把逐条报告评论到 issue，再接 `/land-issue #N`：push、留验收摘要、close。人工只需要扫报告抽查证据。

如果不通过，把“期望 vs 实际”和复现步骤写回 issue 评论，issue 保持 open，让下一个冷启动会话继续 `/implement-codex #N`。这比在同一个长会话里反复修更稳，因为失败信息也变成了 issue 的一部分。

如果验收标准本身有问题，就不要硬判通过或不通过。那是 spec 的 bug，需要回到需求层修正。

实现和验收都是 gpt-5.6-sol，看着像自己判自己。但法官判的不是 Codex 的代码，那份代码 Claude 在 code-review 阶段已经用另一套先验审过了。法官判的是交付的行为满不满足这个 issue，而 issue 是 Claude 写的。所以交叉检查其实是双向的：Claude 审 Codex 的代码，Codex 审 Claude 的 spec。上一节留下的那个洞就是这么堵的，一份从头写歪的 spec，只有没参与过它的人才看得出来。

真正堵不上的是共享先验：实现者和法官透过同一个模型的眼睛读同一份验收标准。这只在法官必须解释、而不是观察的时候才咬人，所以 accept-issue 里把升档规则写死了。四种情况换 Claude 来判：某条标准没法化成“跑这条命令、看到这个输出”；diff 碰了钱、密钥、权限或者不可逆操作；Codex 上一轮标了 spec bug，或者说标准有歧义；这个 issue 是验收失败后打回来重判的。剩下的情况标准都能机械核验，判案靠证据不靠先验，谁来判就没那么要紧。

顺着这个逻辑往回推，验收标准写得越像“跑 X、期望 Y”，法官是谁越无所谓。所以真正该使劲的地方不在验收，在 to-spec：把标准写成可核验的断言，同源的问题自己就消解了。

## 真正重要的是交接面

agent workflow 表面上看是 Claude + Codex 的组合，但它真正依赖的不是某个模型版本，而是交接面。

GitHub Issues 是任务通道。它负责告诉实现者：这次只做什么、怎么判断完成、依赖什么、不能碰什么。

repo docs 是上下文通道。它负责保存那些不应该散落在聊天里的东西：领域术语、架构决策、项目约定、验证命令。

commit 是实现输出。Codex 只把代码推进到可验收状态，不把“已经写完”直接等同于“已经接受”。

这三个东西加起来，才让 AI coding 从一段对话变成一个可复盘的工程流程。

```text
聊天里的意图
  -> spec issue
  -> vertical-slice issues
  -> 冷启动实现
  -> 独立验收报告
  -> landed commit
```

每一步都有一个可检查的中间物。这样就算某一步失败，也知道该回头修哪里。

## 它适合什么场景

agent workflow 不适合所有任务。

如果只是改一个 typo、补一个 alias、修一个很明确的小 bug，直接让一个 agent 做完就好。为这种任务写 spec、拆 issue、跨模型验收，成本太高。

它更适合几类场景：

- 需求一开始是模糊的，需要先被拷问清楚。
- 功能会拆成多个 issue，前后有依赖关系。
- 项目里有领域术语、架构边界或验证步骤，不能只靠模型临场发挥。
- 你希望实现结果可以被冷启动复现，而不是只在当前聊天里成立。
- 你想把验收从“模型说做完了”变成一份可以回看的报告。

代价也很明确。Fable 5 贵，所以它要尽量花在需求和把关上；Codex 的推理档也吃 token，我的取舍是默认 `xhigh`，用一次做对换少返工，spec 钉得够死时再降档省钱。issue 写不好时，流程会不断把问题打回规格层；前几个 issue 的验收报告最好人工认真读，因为那是校准 issue 写法最直接的反馈。

但这正是 agent workflow 的价值：它会让模糊暴露得更早。

以前我更容易把失败归因给模型：它没懂、它写偏了、它又偷懒了。跑这套流程以后，很多问题会回到更基础的地方：我有没有把需求讲清楚，有没有把验收标准写到能执行，有没有把项目约定沉淀到 repo 里。

AI 编程最后拼的不只是模型能力，而是你能不能把意图变成规格，把规格变成任务，把任务变成可验证的代码。

agent workflow 只是把这条链路显式化。
