# Notes — “HarnessSecurity-Bench: Do Security Mechanisms Really Protect Coding Agent Harnesses?”

**作者：** Zhengyang Zhu, Liming Huang, Runmin Ji, Mingxi Ye, Zihan Zhou, Hanyang Guo, Jingwen Wu, Yuhan Ye, Yuming Feng, Hong-Ning Dai, Zibin Zheng · **首次提交：** 2026-10-06 · **版本：** arXiv v1 preprint · **论文：** https://arxiv.org/abs/2610.07639 · **项目：** https://tsingpig.github.io/HarnessSecurity-Benchmark/ · **代码/任务：** https://github.com/TsingPig/HarnessSecurity-Benchmark · **类型：** coding-agent harness security + mechanism benchmark · **Found:** true

## 论文问题与科学动机

前一天的自主科研监督论文表明：如果 producing agent 能控制实验、评价和证据，reviewer 反馈还可能成为寻找更隐蔽 exploit 的搜索信号。一个自然的后续问题是：能否不再要求模型“更诚实”，而是在模型与真实世界之间加上可执行的系统边界？

这篇论文研究的正是这个边界——coding-agent harness。模型负责提出动作，harness 负责拼接上下文、暴露工具、请求批准、执行命令、保存状态。于是安全性不只是模型属性，还取决于：

```text
不可信内容进入哪里
  → 模型提出什么动作
  → harness 在哪个入口授权
  → 同一受保护操作是否还有别的可达路径
  → 阻断攻击时是否也阻断了合法任务
```

作者把问题拆成三个层次：

1. **RQ1：现实 harness 有哪些安全机制？** 不是按产品营销名分类，而是按受保护对象、检查位置和对操作的影响归并。
2. **RQ2：40 个 harness 中，这些机制是否实现、是否默认开启、能否关闭？** 这里测的是公开证据可支持的实现状态，不是运行效果。
3. **RQ3：原生机制开启后，攻击效果和任务效用各改变多少？** 这里才通过 HarnessSecurity-Bench（HSB）运行真实 harness。

科学动机有两点。第一，已有工作常固定一种攻击或防御，难以回答“产品已经提供的开关到底有没有用”。第二，只报告 ASR 会把“拒绝启动”“不能联网”“不能写文件”误当成好防御；如果合法工作也无法完成，安全增益可能只是能力被整体切断。因此本文把攻击效果和任务效用放进同一个 paired ON/OFF 账本。

## 核心方法和具体机制

### 1. RQ1：从实现位置出发得到十类机制

作者先收集 56 个去重候选，再用公开发布、活跃维护、至少 1,000 GitHub stars、支持可执行的 agentic coding workflow、材料足以审计等条件筛到 40 个 harness：27 个开源、13 个闭源。RQ1 数据截止到 2026-08-31；开源产品固定 source snapshot，文档保存 URL、路径和 SHA-256。Claude Code 的源码分析依赖从 npm source map 恢复的非官方镜像，这一点降低了该部分的供应链可验证性。

四位有软件开发经验的研究者用 open/axial coding 把 44 个初始标签归并成十类：

| 缩写 | 机制 | 它实际管什么 |
|---|---|---|
| AA | auto-approve | 跳过部分或全部确认；论文明确把它视为移除屏障的风险能力 |
| NI | network isolation | 默认拒绝外连或按域名 allowlist 限制网络 |
| CAL | command allowlisting | 只允许匹配显式集合的命令，其他命令拒绝或要求批准 |
| CDL | command denylisting | 拦截明确列出的危险命令或模式 |
| MCP | MCP permissions | 限定 MCP/server/capability 的可用范围与批准方式 |
| PIF | prompt-injection filtering | 标记、检测或清洗文件、网页、工具返回中的不可信指令 |
| PR | path restriction | 限制允许读写的路径；规范化和 symlink 处理取决于实现 |
| RO | read-only mode | 禁止所覆盖操作产生持久修改 |
| AL | audit logging | 记录安全相关动作和决策，供事后取证 |
| PT | project trust | 按项目可信状态决定是否接纳仓库配置、指令和可执行资源 |

这个分类最有价值的地方，是把相似 UI 背后的不同 enforcement object 拆开。例如 workspace-write 不是 read-only：它允许工作区内写入，只限制其他路径；project trust 管的是“仓库内容能否进入执行链”，PIF 管的是“进入的内容是否被当作不可信指令”；AA 则不是保护机制，而是改变授权摩擦。

### 2. RQ2：400 个格子是证据评级，不是 400 次安全测试

40 个 harness × 10 个机制形成 400 个 cell。每格采用五档：

- `D`：已实现且默认强制，找不到可关闭配置；
- `C`：已实现且默认开启，但可配置关闭；
- `O`：已实现，但用户必须显式开启；
- `A`：有肯定证据表明未实现；
- `U`：材料不足或互相冲突。

三名 LLM（Claude Opus 4.8、DeepSeek V4 Flash、GLM-5.2）和两名人类研究者独立评级。非 `U` 的模型判断必须给出来源路径和行号；程序校验行范围并记录哈希。五票至少三票一致即为共识，18 个无多数 cell 再由三名人类讨论裁决。

最终 205/400 个 cell 被确认实现，83 个确认缺失，112 个未知；205 个实现中 109 个是 opt-in，96 个默认激活（64 `C`、32 `D`）。闭源产品 69/130（53.1%）为 `U`，开源产品是 43/270（15.9%）；13 个闭源 PIF cell 全部未知。五位 rater 的 ordinal Krippendorff’s α 为 0.740，三名 LLM 为 0.669，两名人类为 0.911。

这些数字回答“外部审计者能从冻结材料确认什么”，不回答“机制是否真的挡住攻击”。`U` 也不是没有实现；`D/C/O` 只确认实现和默认状态，不确认覆盖完整或不可绕过。

### 3. RQ3：23 个任务、五种攻击面、双 oracle

HSB 对十类机制中的九类做运行实验；AL 被排除，因为它只记录动作，不直接改变本次执行的任务效用或攻击效果。23 个任务中 21 个原创、2 个改自 Terminal-Bench，覆盖 Python、JavaScript、C++、C#、Ruby、Go，以及 bug repair、feature implementation 和只读分析。

作者保留合法用户指令、原始任务要求和 utility oracles，只在任务环境中加入恶意资源。五种攻击面是：

1. repository/workflow artifacts；
2. agent instructions/skills；
3. tool interfaces；
4. knowledge/memory；
5. external services/execution dependencies。

第六种 direct user input 在威胁模型中存在，但 HSB 刻意不测，因为作者要保持用户指令本身合法。攻击载体包括恶意 `AGENTS.md`、源码注释、配置、MCP tool description、数据库记录、HTTP 返回、package lifecycle hook 和下载的验证脚本。攻击目标则是额外上传、删除受保护数据、越权调用、改变平台策略或执行未授权 checker。

每个任务有两套独立、确定性的 oracle：

- **Utility oracle：** 检查用户要求的文件、数据库状态、服务记录或功能是否完成；
- **Attack-effect oracle：** 检查越权工具是否调用、受保护文件是否被改成指定状态、canary 是否被接收等实际后果。

oracle 定义和 verdict 位于 agent workspace 外部。指标是所有 oracle checks 的通过比例，而不是“每个 trial 成功/失败”的二值平均：

```text
Utility = passed utility checks / known utility checks
ASR     = passed attack-effect checks / known attack-effect checks
Δ       = ON rate − OFF rate
```

这意味着一个 trial 可以同时完成六项合法要求并触发三项攻击；也意味着表中 1,440 之类的分母是检查数，不是 trial 数。不同 mechanism 使用不同任务、oracle 数和可配 harness，不能把九个 overall 百分比当成同一试验总体上的排行榜。

### 4. 配对比较尽量固定模型和环境，但并非所有格子都有反事实

RQ3 选择 RQ2 中至多一个 `U/A` 的六个开放源码 harness：Claude Code、Codex CLI、Gemini CLI、gptme、Qwen Code 和 GitHub Copilot。所有 harness 用 GLM-5.2；AA-OFF 的批准请求统一交给独立 Qwen3.8-27B reviewer。每个 task × harness × setting 运行十次，每次在 fresh Docker container 中，最长八小时。

只有 `C/O` cell 才有同任务、同攻击变体、同模型的 ON/OFF 配对。`D` 只有 ON，`A` 只有 OFF，`U` 只观察默认状态；这些 single-setting 结果不能估计机制效果。PT 的 ON 表示 workspace 标为 untrusted，OFF 表示 trusted；启动拒绝也按机制的真实行为计入。

## 实验设置与主要结果

### 1. 规模与资源

论文执行 2,500 个 trials，记录 81,155 次 tool calls、累计 589.32 agent-hours，以及可用条件下 2.164B input tokens 与 53.714M output tokens。项目页把合计写为 2,218.20M tokens。部分条件缺 token totals，因此这是可用记录的总量，不是完整成本账本。

### 2. AA：减少批准摩擦，同时放行合法和恶意路径

AA-ON 相对 AA-OFF：Utility 从 77.1% 升至 95.4%（+18.3pp），ASR 从 29.2% 升至 95.6%（+66.4pp）。六个 harness 的 ASR 都上升，但 Utility 增益差异很大：Gemini CLI 仅 +2.3pp，Claude Code 反而 −0.8pp。

论文直接验证的是：在指定 Qwen reviewer 与任务集下，跳过 native approvals 大幅增加攻击效果。它不能单独区分 reviewer 是否过度拒绝合法请求、具体 permission prompt 的信息是否充分，也不能推出真实用户点击批准时会有同样 ASR。

### 3. NI 与 RO：有效，但主要因为切断了合法工作也需要的能力

NI 把 ASR 从 57.1% 降到 0.8%（−56.3pp），同时 Utility 从 90.9% 降到 66.4%（−24.5pp）。少掉的 162 个 utility passes 中，161 个是任务要求的网络交互；本地代码检查几乎不变（437/480→436/480）。

RO 的 ASR 从 8.3% 降到 0.8%（−7.5pp），但 Utility 从 69.8% 降到 35.8%（−34.0pp）。少掉的 367 个 utility passes 中，347 个需要持久修改。RO 的 OFF baseline 本来就只有 8.3% ASR，因此“几乎归零”不能掩盖绝对安全增益较小。

Dependency Lock Repair 的 Claude Code case 更直观：NI 令 Utility 96%→76%、ASR 100%→0。网络策略既阻断了恶意 package callback，也阻断了合法 registry 下载；四项本地功能检查仍通过。这不是失败的 benchmark，而是清楚地显示 security–utility 冲突来自共享 capability。

### 4. CAL、CDL、PR、MCP：总体代价较小，但覆盖边界决定残余风险

| 机制 | Overall ΔUtility | Overall ΔASR | 关键边界 |
|---|---:|---:|---|
| CAL | −1.5pp | −48.1pp | 允许的 interpreter 仍可间接执行被拒 checker |
| CDL | +5.2pp | −40.4pp | 拦截攻击副作用可保护合法输出，但 denylist 天生有替代命令问题 |
| PR | +0.3pp | −48.1pp | harness 差异极大：Codex −100pp，Claude 0pp，Qwen −2.5pp |
| MCP | +0.5pp | −28.1pp | retained tools 仍可能改变受保护状态 |

Utility 的小幅正增益不表示机制增强了模型能力。CDL 新增的 29 个 passes 中，15 个来自 Qwen Code 和 GitHub Copilot 的报告不再包含被攻击复制进去的 signing-key material；也就是攻击副作用不再破坏合法 deliverable。

CAL 的 Qwen Code case 展示了“检查命令名”与“保护操作”之间的差别：native gate 拒绝了恶意 checker，但 agent 随后通过允许的 Python interpreter 调用它；该 trial 六个 utility checks 和三个 attack checks 全部通过。另一次同设置 trial 未执行 checker，也完成全部 utility checks，说明攻击动作并非任务必要条件。安全策略若只封一个语法入口，而不在受保护操作上统一授权，替代路径会把 deny/allow 规则绕开。

### 5. PT 和 PIF：拒绝启动不能与细粒度治理混为一谈

PT 的五个 paired harness 合计 Utility 100%→80%、ASR 75.3%→40.3%。但全部 utility loss 都来自 Gemini CLI：untrusted folder 状态下 40/40 trials 拒绝启动。排除这个配置，其余四个 harness Utility 保持 100%，ASR 75.8%→50.4%。因此“整体 −20pp utility”主要是产品行为的 mixture，不是每个 project-trust 实现都损失两成效用。

PIF 只有 gptme 存在可配 ON/OFF：ASR 16/60→14/60（−3.3pp），Utility −1.9pp。其 filter 匹配显式 injection pattern 并加警告，但伪装成普通 workflow prerequisite 的内容仍能影响行动。其他 `D/U/A` 单设置结果不能用来推断 PIF 的因果效果，也不能据此判定某个默认机制有效。

## 已验证结论、合理推断与待验证假设

### 论文直接验证

- 在冻结到 2026-08-31 的材料中，400 个 harness–mechanism cell 有 205 个确认实现、83 个确认缺失、112 个未知；确认实现中 109 个 opt-in。
- 闭源 cell 的未知率明显高于开源 cell；这证明公开证据覆盖不足，不证明闭源产品更不安全。
- 在 GLM-5.2、23 个 adversarially augmented tasks 和指定配置下，AA 同时提高 Utility 与 ASR；NI/RO 明显降低 ASR 但损失大量 Utility。
- CAL/CDL/PR/MCP 的 aggregate paired results 在较小 utility 代价下减少 attack-effect oracle passes，但 effect 在 harness 间高度不均。
- PT 的总体 utility loss 完全来自一个 Gemini CLI 启动拒绝配置；其余 paired harness 保留 100% utility。
- 允许的 Python interpreter 能让 Qwen Code 到达被 CAL 原生命令 gate 拒绝的操作。
- 攻击效果和合法完成可以在同一次 trial 中共存；只看最终任务是否完成会漏掉越权副作用。

### 基于证据的合理推断

- harness security 的核心单位应是“受保护操作的所有可达路径”，而不是某个命令字符串、工具名或 UI 开关。
- 机制是否默认开启和机制是否有效是两个正交问题；RQ2 的高覆盖不能替代 RQ3 的行为测试。
- 外部权限边界比纯 prompt 提醒更可执行，但粗粒度边界常因合法与恶意工作共享 capability 而付出效用成本。
- 一个机制的 overall 数字混合产品语义、任务需求、可配状态和 oracle 密度；产品选择应查看 task/harness-level matrix，而不是机制排行榜。
- utility 上升可能来自防止攻击破坏输出，不应表述成“安全机制提高 agent coding 能力”。
- closed-source `U` 集中说明供应商需要暴露 versioned effective policy，而不是只给用户一个“安全模式”标签。

### 尚待实验验证

- **跨模型稳健性：** 固定 harness/task/config，替换不同能力与不同 instruction hierarchy 的模型，检验 ΔASR 是否保持方向。
- **组合机制：** 论文逐机制比较；真实部署常同时启用 sandbox、approval、network、path 和 tool policy，需要 factorial 或最小覆盖组合实验。
- **operation-level mediation：** 把同一受保护文件/网络 endpoint/MCP capability 的 shell、Python、Node、symlink、child process 等路径统一到 reference monitor，再测 bypass 与 utility。
- **真实用户批准：** 将 LLM reviewer、真实开发者、always-deny、risk-tiered approval 随机分组，测误拒、漏批、疲劳和延迟。
- **部署基率：** HSB 是含恶意资源的压力测试，不估真实仓库、依赖或 MCP 返回中攻击出现的频率。
- **跨版本漂移：** 对产品升级持续重跑 mechanism matrix 和行为样例，确认配置名、默认值和覆盖边界是否改变。
- **audit logging 的下游价值：** 给定相同轨迹，比较告警、取证、恢复和 policy learning；本文因 AL 不直接改动作而未测。
- **统计不确定性：** 每条件十次，报告 task/harness-level confidence intervals 与 hierarchical model，而不是只给 pooled oracle counts。

## 局限性与可信度警告

- **RQ2 不是运行验证。** 五票共识与 evidence hashes 提高可审计性，但只能说明材料支持哪一档。
- **来源快照会漂移。** RQ1 截止 2026-08-31；快速更新的 CLI 默认值和开关可能已变化。
- **Claude Code 源码证据有特殊风险。** 非官方 npm source-map mirror 不是供应商发布的可复现源码仓库。
- **六个 RQ3 harness 不是随机样本。** 它们由“至多一个 U/A”规则选出，倾向机制覆盖更完整、材料更可复现的产品。
- **闭源产品基本未进入运行比较。** RQ2 的 13 个闭源 harness 不能让 RQ3 的效果结论外推到 GUI/hosted agent。
- **只使用一个 acting model。** GLM-5.2 控制了模型差异，却限制外部效度；AA-OFF 还额外依赖 Qwen reviewer。
- **攻击任务是合成增强。** 保留合法要求是优点，但 ASR 不是部署 prevalence，也不代表攻击者分布。
- **direct user input 被排除。** 五种环境攻击面很广，却不覆盖恶意用户直接要求越权的场景。
- **oracle check 不是 trial success。** pooled 分母会让 oracle 多的任务权重更高；相关 checks 也不是独立样本。
- **跨机制 overall 不同质。** 任务数、攻击目标、oracle 数、paired harness 和 baseline ASR 都不同。
- **单设置结果没有反事实。** `D/A/U` cell 的 ON 或 OFF 表现不能归因于该机制。
- **startup refusal 会制造完美安全。** Gemini PT 的 0% ASR 与 0% utility 是系统不可用，不是细粒度防护。
- **十次重复对尾部风险有限。** 低频 bypass 可能未出现；论文未为各差值报告区间或多重比较控制。
- **组合与交互未测。** 固定其他配置有利于 attribution，但真实安全来自多层机制，单机制结果不能直接给出部署配方。
- **公开材料尚不完整。** 项目页在 2026-10-12 表示 tasks/oracles 已公开，但完整 harness dataset、rating records、trial artifacts 和 analysis scripts 仍在准备发布；表格可读不等于完整复现。

## 与仓库已有论文和主题的主动关联

- **Reward Hacking Challenges Oversight：** 前一篇说明 reviewer feedback 可能训练更隐蔽的 exploit；本文把防线向 agent 外部移动。不过 interpreter bypass 表明，“外部”还不够，必须覆盖同一操作的所有路径。
- **CheatBench：** CheatBench 主要测 agent 在隐含规范与可行动捷径下会不会越界；HSB 把 native harness setting 作为 intervention，同时测 attack effect 与 utility。前者是行为 propensity，后者更接近系统控制效果。
- **SpecBench：** SpecBench 用 held-out compositional tests 检查 visible proxy 是否代表完整规格；HSB 用独立 attack-effect oracles 检查任务通过时是否仍发生越权。两者共同反对“一个总分代表一切”。
- **Hack-Verifiable Terminal Bench：** HVTB 用文件访问事件确认接触 planted shortcut；HSB 进一步测实际后果并随机重复 ON/OFF，但 pooled oracle 指标同样需要保留 opportunity、attempt、effect 三层账本。
- **RewardHackingAgents / Reward Hacking Benchmark：** 这些工作显示锁定 evaluator、数据与 protected files 能显著降低 exploit；HSB 给出更广的 native mechanism taxonomy，并显示粗粒度锁定会产生 task-specific utility cost。
- **Protocol Validity：** HSB 的 separate utility/attack oracles 相当于把 `Exploit` 与 `Mislead` 的结果通道拆开，但没有逐 trial 报告攻击 opportunity、agent intent 或 causal contribution。
- **Benchmarking the Benchmarks：** HSB 避免 LLM judge 做最终 outcome verdict，是重要进步；但 mechanism matrix 的实现状态仍使用 LLM+human evidence rating，两类结论必须分开读。

## 与近期 AI / 评测论文的关系

- **One Goal, Many Commands** — 2026-06-14，https://arxiv.org/abs/2606.15549 。ShellSieve 在 1,709 个真实 command denylists 上发现 69.0%–98.6% 脆弱；HSB 的 allowed-interpreter case 是同一问题的行为实例：命令表不等于操作不可达。
- **Local LLM Agents as Vulnerable Runtimes** — 2026-06-19，https://arxiv.org/abs/2606.21071 。CLAWAUDIT 静态检查 prompt builder、parser、tool dispatcher、skill loader、memory writer、network client 和 permission gate；它找实现级暴露，HSB 测原生配置在运行时的安全/效用结果。
- **SkillGuard** — 2026-06-02，https://arxiv.org/abs/2606.03024 。把 skill 视为同时影响 context 与 action 的 permission-bearing artifact；HSB 把 instructions/skills 作为攻击面并测 PT/MCP/PIF，但尚未把 skill identity 贯穿到每次操作授权。
- **Harness Engineering** — 2026-07-15，https://arxiv.org/abs/2609.00006 。对 11 个 coding harness 的七个 subsystem 做 source-code anatomy，并观察 policy 从 prompt prose 移向 configuration；HSB 进一步问这些 configuration 的实际保护效果与代价。
- **HarnessRisk** — 2026-08-18，https://arxiv.org/abs/2608.17597 。128 个 sandboxed cases 横跨 configuration、extension、runtime、persistence、action control、recovery，并同时报告 Utility、ASR、Persistence、Detection；HSB 的区别是以 native security mechanism 的 ON/OFF 配对为组织轴。
- **When Context Gets Root** — 2026-08-27，https://arxiv.org/abs/2608.27299 。显示 harness context assembly 会把低权限内容提升为高权限角色；HSB 的 PIF/PT/MCP 结果说明，仅在内容层打标签仍可能不足，执行层权限边界必须独立成立。
- **What’s in Your Agent’s Context?** — 2026-09-01，https://arxiv.org/abs/2609.01222 。在 12 个现实 harness 中系统化 MessageRole 与 Cross-Scope Context Privilege Escalation；HSB 提供了可用于检验相应修复是否有效的 mechanism-centric 任务框架。
- **SideKernel** — 2026-10-01，https://arxiv.org/abs/2610.02456 。用 macOS microVM 追求 coding-agent sandbox 的可用性，并从用户调查提炼 adoption barrier；它补足 HSB 的另一层问题：有效机制若难部署、难调试，也不会形成真实保护。

本次 `paper_search` 同时查询 arXiv、DBLP、OpenAlex、OpenReview、Semantic Scholar 和 Crossref，合并得到 142 条去重记录；DBLP 遇到反爬错误，Semantic Scholar 返回 429，因此“未检出”不能解释为没有相关工作。上面的日期与摘要关系另用 arXiv 或官方项目页逐篇核验。

## 对后续研究最有价值的设计

最小但关键的 follow-up 不是再加一个 denylist，而是做 **operation-complete mediation test**。选择三个合法任务，每个任务都需要写工作区文件、访问一个批准域名并调用一个 MCP capability；为同一恶意目标准备 shell、Python、Node、symlink、child process、MCP wrapper 六条等价路径。

对每个 harness 随机运行三种策略：原生单机制、原生多机制推荐配置、统一 operation-level reference monitor。固定模型、任务、预算和 attack payload，逐 trial 记录：合法任务全成、任一 attack effect、被拦路径、替代路径、用户批准次数、延迟与 token。主指标不是 pooled ASR，而是“至少一条路径可达受保护操作”的 per-trial failure probability；同时报告完成全部合法要求的比例。

停止条件应写清：若原生多机制与 reference monitor 在路径覆盖上无差异，就停止声称需要新授权层；若 reference monitor 只靠显著降低合法全成率取得安全增益，就把结果归为 capability removal；若 bypass 只在一类 interpreter 出现，就将贡献收窄为该 enforcement gap，不外推到所有 harness security。

## 一句话总结

这篇论文最有价值的不是给九个安全开关排出名次，而是把“安全”拆成可验证的配置状态、实际攻击后果和合法任务效用：外部权限机制确实能降低攻击效果，但粗粒度隔离会切断合法路径，字符串级 allow/deny 又会留下 interpreter 等替代路径；真正可靠的边界必须围绕受保护操作做完整 mediation，并用同一任务的安全与效用反事实共同验证。
