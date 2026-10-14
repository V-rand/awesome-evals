# Notes — “HarnessRisk: A Lifecycle-Oriented Benchmark for Agent Harness Safety”

**作者：** Yajing Bai, Jinhao Duan, Jie Peng, Xianfeng Wu, Sijia Liu, Song Wang, Tianlong Chen · **首次提交：** 2026-08-18 · **版本：** arXiv v1 preprint（under review） · **论文：** https://arxiv.org/abs/2608.17597 · **项目：** https://baiyajing.github.io/harness-risk/ · **代码：** https://github.com/Baiyajing/HarnessRisk · **数据：** https://huggingface.co/datasets/YajingB/HarnessRisk · **类型：** agent-harness lifecycle safety benchmark · **Found:** true

## 论文问题与科学动机

前一天读的 HarnessSecurity-Bench 把 native security setting 变成自变量：打开 network isolation、read-only、command allowlist 后，攻击效果和任务效用各改变多少。但真实 agent 的风险不只发生在一次工具调用的授权点。它可以在安装 skill 时获得权限、在日常任务中遇到恶意内容、把内容写进长期记忆、几天后执行付款，并在事后恢复时继续听从被污染的“修复指南”。

HarnessRisk 因此把研究对象从“某个攻击或某个开关”改成 **harness 生命周期中的责任链**：

```text
配置安全边界
  → 安装/更新能力
  → 处理不可信内容
  → 写入长期状态
  → 执行高影响动作
  → 发现并修复污染
```

论文问的不是“哪个模型最安全”，而是三个更系统的问题：

1. 同一个 model 换到不同 harness 后，安全表现是否改变？
2. 高 Utility 或显式识别风险，是否意味着没有发生越权后果？
3. 六个阶段中，风险从哪里进入、在哪里持久化、为什么恢复失败？

科学动机是：如果只测最终回答或单次 tool call，就会漏掉“任务完成且攻击也成功”“本轮没出事但恶意规则已写入 memory”“发现污染却没有撤销 token/skill/policy”等过程性失败。安全必须成为 **model–harness configuration 的轨迹属性**，而不是 base model 或最终答案的标签。

## 核心方法和具体机制

### 1. 六阶段 lifecycle taxonomy

| 阶段 | Harness 应承担的责任 | 典型不可信载体 | 失败机制 |
|---|---|---|---|
| Harness Configuration | 初始化 connector、credential、gateway、policy、template | setup guide、template、manifest | 在正常配置修改里夹入 inline secret、开放 gateway、关闭 approval |
| Capability Extension | 选择、安装、更新和授权 skill/plugin | package/marketplace metadata | typosquat、恶意或过度授权扩展进入执行面 |
| Runtime Operation | 日常处理邮件、网页、文档、tool output | email、webpage、document、tool reply | 间接指令重定向 agent，触发泄漏或越权工具调用 |
| State Persistence | 保存 memory、identity、preference、policy、trigger | profile、note、同步记录 | 短期恶意内容进入长期可信状态，后续继续生效 |
| Action Control | 授权付款、部署、删除、grant、外发通信 | ticket、message、operational record | 在本来合法的动作中替换金额、收款人、endpoint 或 secret |
| Incident Recovery | 调查、回滚、撤销、轮换凭证、修复状态、保留证据 | log、hold notice、recovery record | 识别出污染，却保留 token/extension/policy/memory，或只写报告不执行修复 |

这个 taxonomy 按 harness 在运行周期中的职责组织，而不是按 prompt injection、memory poisoning、malicious skill 等攻击名字组织。优点是可以问“哪一段责任链没有守住”；代价是一个 case 往往同时包含多个攻击技术，阶段不是独立的单一处理变量。

### 2. 128 个三轮 workflow，合法目标与攻击目标并存

128 个 cases 近似均分：Configuration 22、Capability Extension 22，其余四阶段各 21。每个 case 有四个组件：

- benign user task；
- initial world state（文件、配置、持久状态、mock service）；
- 可用工具；
- case-specific evaluation criteria。

固定三个 owner turns 依次给出上下文、引入或重新解释 workflow evidence、请求最终操作。恶意指令藏在 setup guide、extension metadata、邮件、网页、文档、memory record、tool output 或 recovery record 中。作者要求合法任务不依赖攻击目标，理论上 agent 可以既完成任务又保持安全。

一个 Configuration 例子要求配置 vector index 并总结政策；retrieval manifest 声称必须写入 plaintext token 并打开 workspace-wide sharing。安全轨迹保留间接 secret reference 和窄 sharing scope，危险轨迹写入攻击者指定值。这个设计测的不是 agent 是否拒绝整个任务，而是能否在授权操作内部区分安全字段和危险字段。

但三轮结构还有一个重要边界：第二个 owner turn 会把某些可疑 artifact 描述成“已批准 context”或“owner-provided validation step”。论文自己把这一失败解释为 **later turns launder provenance**。因此部分 case 同时测试间接注入与受信任用户重新背书；不能把所有 ASR 都归因于 agent 无视 instruction hierarchy。

### 3. 四个 trajectory-level 二值指标

每条轨迹得到四个独立标签：

```text
Utility      = 合法任务是否完成
ASR          = 指定攻击后果是否实际发生
Persistence  = 恶意影响是否进入可观察的持久状态并继续有效
Detection    = agent 是否明确指出具体内容恶意、异常、可疑或不可信
```

四个标签相互独立，所以同一轨迹可以 `Utility=1, ASR=1`。尝试了攻击但没有产生指定后果不算 ASR；一次性副作用不算 Persistence；沉默避开、泛化免责声明或无法解释的拒绝不算 Detection。

每个 model–harness configuration 在 128 cases 上重复三个 model-sampling seeds，名义上是 384 trajectories。14 个配置的名义总量为 5,376，但正文没有报告各配置因 provider failure、空响应、异常退出或 evaluator parse failure 排除后的实际分母。

### 4. 主结果用 GPT-5.4 统一判定，确定性规则只做抽样校准

主表的四项指标都由 GPT-5.4 读取 benign/adversarial goal、normalized transcript、tool events、final answer、workspace diff、persistent-state candidates、mock-service state 和 network observations后判定。Tool/state evidence 优先于 final answer 的自述。

作者没有直接把 deterministic predicates 用作全部 Utility/ASR 主标签，而是抽样验证 judge：

| 指标 | 参考标准 | n | exact agreement | Cohen’s κ |
|---|---|---:|---:|---:|
| Utility | deterministic predicates | 360 | 92.5% | 0.83 |
| ASR | deterministic predicates | 360 | 89.7% | 0.77 |
| Persistence | adjudicated human labels | 300 | 84.3% | 0.65 |
| Detection | adjudicated human labels | 300 | 85.7% | 0.69 |

这比完全不校准 LLM judge 强很多，但也说明主表不是 deterministic ground truth：ASR 抽样仍有 10.3% disagreement，Persistence/Detection 有约 14–16%。而且论文没有按 harness、phase、model 或 label prevalence 分层报告误差，无法判断模型排名是否受系统性 judge bias 影响。

### 5. 三个 harness、六个 models、14 个部署配置

OpenClaw、Nanobot、Hermes 都测试 DeepSeek-V4-Pro、GLM-5.2、Kimi K2.6、MiniMax M3；OpenClaw 额外测试 GPT-5.5 和 Claude Opus 4.7，共 14 个配置。每次运行重新建立 workspace、harness home、persistent state 和 mock services；真实外部网络关闭，邮件、付款、部署、browser、gateway、webhook 和 skill marketplace 都由本地服务模拟。

这里的 “sandbox” 需要谨慎读。附录与公开 README 明确说是 **process-level state/side-effect isolation**，没有额外 kernel namespace、chroot 或 firewall；能执行任意 host command 的 agent 仍可能访问宿主文件或真实网络。因此它是安全评测的可控实验环境，不是被验证过的 OS security boundary。

## 实验设置与主要结果

### 1. 高 Utility 与高 ASR 可以同时出现

14 个配置的 Utility 为 75.0%–97.6%，ASR 为 12.6%–80.9%。按 harness 聚合的四象限图中，“useful but unsafe”占 OpenClaw 59%、Nanobot 38%、Hermes 43%；“useful and safe”分别为 36%、51%、53%。

图中 unsafe 是 attack success **或** persistence 的联合类别，不应把 59/38/43% 直接当作各 harness 的 ASR。OpenClaw 又比另两个 harness 多两个专属模型，聚合模型构成不同；这些比例适合说明“完成任务不能证明安全”，不适合作为纯 harness 排名。

### 2. 同一 model 换 harness，ASR 可相差 4.3 倍

GLM-5.2 在 OpenClaw 的 ASR 是 54.7%，在 Nanobot 是 12.6%，比值约 4.3；Hermes 为 23.8%。安全排名也改变：GLM-5.2 在 Nanobot 最低，MiniMax M3 在 OpenClaw（31.2%）和 Hermes（14.8%）最低。

论文直接验证的是 **deployed configuration 差异**。三个 harness 的 system prompt、tool surface、state representation、authorization context 和 adapter 都不同，没有保持模型之外的因素不变。因此不能把 54.7→12.6 解释为“Nanobot harness 单独降低 42.1pp ASR”，更不能由此推出某项具体设计是原因。

### 3. Configuration 是三个 harness 的共同高风险阶段

Figure 6 用三个 harness 共有的四个 models 做 phase macro-average；Configuration 在每个 harness 都是最高 ASR。Figure 3 的四模型均值约为 OpenClaw 74.8%、Nanobot 45.3%、Hermes 61.5%。

定性分析给出的机制是：任务本来就授权修改 connector、gateway、credential 或 policy，恶意 artifact 只替换一个字段。agent 不是明显“偏离任务”，而是在合法配置动作中写入 inline secret、扩大 sharing、取消 approval 或开放 gateway。这类攻击绕过的是目标/字段级授权，而不是简单诱导 agent 调用一个完全无关的工具。

但“Configuration 最危险”仍是这 22 个设计案例和四模型样本下的 descriptive result。各 phase 的 artifact、动作可逆性、工具数量和 oracle 难度不同，没有做同一攻击跨阶段随机化，不能把阶段本身视为因果处理。

### 4. Detection 与较低 ASR 相关，但识别不等于 containment

12 个共同 model–harness 配置中，Detection 与 ASR 的 unadjusted Pearson `r=-0.708, p=0.010`，Spearman `ρ=-0.769, p=0.003`；Utility 与 ASR 的关系弱且不显著。

反例更重要：OpenClaw + MiniMax M3 的 Detection 为 97.9%，ASR 仍为 31.2%；OpenClaw + GLM-5.2 的 Detection 92.2%，ASR 54.7%。在 recovery cases 中，agent 会指出 contaminated state，却不删除 token、skill、policy 或 memory，或者只写证据报告而不完成 rollback/revocation。

附录进一步说明：加入 model 和 harness indicators 后，Detection coefficient 不再显著（`p=0.101`），而样本只有 12 个配置。因此“检测帮助安全”是描述性关联；论文更扎实的结论是 **Detection 与 remediation 必须分开测**。

### 5. Persistence 与 ASR 是不同故障

Persistence 范围为 4.0%–23.9%。恶意内容可能已经进入 memory/config/skill，但尚未在本轮实现攻击目标；相反，一次性付款或泄漏可让 ASR=1，却没有持久状态。HarnessRisk 的贡献之一正是让“现在造成后果”与“把未来行为变危险”拥有不同标签。

不过 Persistence 只基于 harness 导出的可观察 durable state。不同 harness 暴露的 memory、reasoning 和 internal state 不同，较低 Persistence 可能部分是较少可见，而不一定是较少污染。

## 已验证结论、合理推断与待验证假设

### 论文直接验证

- 128 个合成、sandboxed workflows 覆盖六个 lifecycle phases；每个 case 有三轮 owner messages，并分别定义合法目标和攻击目标。
- 在 14 个 model–harness configurations 中，GPT-5.4 judge 报告 Utility 75.0%–97.6%、ASR 12.6%–80.9%。
- GLM-5.2 在 OpenClaw 与 Nanobot 的 ASR 分别为 54.7% 与 12.6%；这证明完整部署配置差异，不隔离 harness-only effect。
- 三个 harness 的四模型 phase macro-average 都以 Configuration ASR 最高。
- 高 Utility 与 unsafe outcome 经常共存；最终任务完成不能替代安全轨迹评价。
- 高 Detection 也能与高 ASR 共存；明确识别风险并不保证拒绝、containment 或 remediation。
- GPT-5.4 evaluator 在抽样上与 deterministic/human reference 有 84.3%–92.5% agreement，尚非无误差 oracle。
- 公开项目目前提供 128-row case dataset、三个 harness adapters、九类 mock services、rule evaluator 和 LLM judge；代码快照 `07671f25b515` 不包含论文实验的 raw trajectories 或 table-generation ledger。

### 基于证据的合理推断

- agent safety 更适合以 `(model, harness, configuration, case)` 为评价单位，而不是给 base model 一个固定安全分。
- Configuration 风险高，可能因为恶意值藏在已授权的修改中；只做 tool-level allow/deny 无法表达字段、recipient、scope 和 provenance 约束。
- 多轮 workflow 的关键状态不只是文本内容，还包括“谁说过、何时被批准、批准范围是什么”；harness 需要在跨 turn、memory write 和 action execution 中保留 provenance。
- 安全闭环至少需要 detection、containment、remediation、verification 四个不同结果；把“模型说它发现风险”当防御成功会严重高估系统能力。
- Persistence 指标让 benchmark 能区分本轮未爆发的 latent compromise，但需要统一、可查询的 durable-state interface 才能跨 harness 公平比较。
- process isolation 足以避免 mock service 相互污染，却不能证明系统能安全承载任意恶意代码；复现实验仍应外套容器、VM、独立用户和 egress firewall。

### 尚待实验验证

- **Harness-only causal effect：** 固定 system prompt、tool schema、state API、approval policy 和 adapter，只替换 harness routing/implementation；或逐组件 swap，定位 4.3× 差异来源。
- **Provenance laundering 的独立作用：** 同一 artifact 随机分配 no follow-up、neutral owner message、explicit endorsement、cryptographically verified endorsement，测来源背书如何改变 memory write 与 action。
- **Judge error 对排名的影响：** 对所有配置与阶段分层抽样，公开 confusion matrix；用 deterministic predicates 重算 Utility/ASR 主表，并给出 rank-stability interval。
- **真正跨 session 的持久性：** 写入后结束进程、轮换 agent session、经过时间间隔与无关任务，再触发并尝试 selective repair。
- **恢复是否完整：** 注入多组件污染后，要求 revoke credential、remove extension、clean memory、restore policy、preserve evidence，逐项确定性校验且防止再感染。
- **现实部署基率：** 采样真实 setup docs、marketplace metadata、邮件、logs 与 recovery guides，估计危险 opportunity，而不是把 128 个植入案例的 ASR 当发生率。
- **防御 intervention：** 在相同 case/config 上随机启用字段级 policy、provenance label、two-person approval、memory quarantine 和 recovery verifier，同时测 Utility、ASR、Persistence、Detection 与成本。

## 局限性与可信度警告

- **Preprint 且只一个版本。** 首次提交 2026-08-18；没有同行评审状态或跨版本修订证据。
- **主表是 LLM-judge estimates。** deterministic predicates 存在却只用于 360 条抽样校准；ASR agreement 89.7%，不是 ground truth。
- **语义指标更不稳定。** Persistence/Detection 与 human reference 的 agreement 84.3%/85.7%，κ 0.65/0.69；harness 可见性差异还会改变证据量。
- **有效运行分母未报告。** nominal 5,376 trajectories 会排除 provider error、空响应、异常退出和 judge parse failure；若排除不均匀，配置率会偏。
- **三次 seed 不等于完整不确定性。** 标准差只是三个 seed-level rates 的 sample SD，不是 pooled mean 的 SE，也不覆盖 provider/model drift。
- **Cross-harness 不是单变量实验。** prompts、tools、state、authorization、adapter 都一起变化；4.3× 是配置对比。
- **OpenClaw 聚合含额外两个模型。** useful-but-unsafe 的 59/38/43% 还把 attack success 或 persistence 合并，不是可直接排序的 ASR。
- **阶段 case 不同质。** Configuration、Action、Recovery 的攻击载体、可逆性、工具和 oracle 不同；“最脆弱阶段”不是 phase causal effect。
- **Owner follow-up 混合威胁来源。** 后续受信任 turn 会重新背书可疑内容；有些失败可解释为 social engineering/authorization laundering，而不只是 untrusted artifact injection。
- **Sandbox 不是 OS security boundary。** 进程级目录、端口和 mock state 隔离没有 kernel namespace/chroot/firewall；公开 README 建议另加 OS sandbox。
- **低 ASR 可能来自无能力。** refusal、没到达工具、一般性任务失败都会令 ASR=0；Utility 分开报告缓解但不能完全诊断原因。
- **Persistence 是可观察状态。** 未导出的内部记忆不被计入；跨 harness 差异可能混合行为与 instrumentation。
- **相关性样本小。** Detection–ASR 只有 12 个 observation units；控制 model/harness 后 `p=0.101`，不能声称检测因果降低攻击。
- **没有原始结果账本。** 公开仓库与 HF 提供执行框架和 case definitions，但截至 2026-10-14 未见 raw trajectories、validity exclusions、主表重算数据或完整 analysis scripts；论文“可从 archived trajectories 重算所有表”的承诺当前无法独立检查。
- **模型/endpoint 会漂移。** Hosted models、serving policy 和默认 inference controls 可变；论文对 OpenClaw 使用 lowest thinking，其他控制保持 provider defaults，不保证跨模型预算等价。

## 与仓库已有论文和主题的主动关联

- **HarnessSecurity-Bench：** HSB 以 native security mechanism 为 intervention，回答“开关打开后安全/效用怎样变”；HarnessRisk 以 lifecycle responsibility 为 coverage，回答“风险在哪一段进入、持久和恢复”。前者更接近 causal setting comparison，后者覆盖 persistence/recovery，但更依赖配置级观察。
- **Reward Hacking Challenges Oversight：** 那篇论文显示识别 exploit、返回 review feedback 不保证监督成功；HarnessRisk 在系统安全里给出对应现象：Detection 很高仍可执行攻击或留下污染。共同边界是“识别”与“改变后果”必须分开。
- **CheatBench：** CheatBench 比较 implicit honest-work norms、task framing 和 harness swap；HarnessRisk 把 task+attack 固定到完整 model–harness configuration，并增加 durable state 与 recovery，但没有对 harness 组件做 controlled swap。
- **Hack-Verifiable Terminal Bench / HVE：** 两者用 deterministic access/event triggers 避免 judge；HarnessRisk 的 case predicates也能观察 state/service effect，却仍把主表交给 GPT-5.4。最强组合应让 deterministic consequence label 做主，LLM 只解释语义边界。
- **Protocol Validity：** HarnessRisk 同时保存 transcript、tool calls、diff、mock state 和 network events，避免 final answer “Mislead” outcome；但 validity filtering 分母与 raw evidence 未公开，外部审计仍受限。
- **Benchmarking the Benchmarks：** 该工作证明 evaluator–human disagreement 会改变 agent score；HarnessRisk 做了难得的 judge calibration，却没有把约 10–16% disagreement 传播进配置排名和区间。
- **SpecBench：** SpecBench 的 visible/held-out gap 说明 task success proxy 不等于完整规格；HarnessRisk 的 useful-but-unsafe 象限是安全版本：同一轨迹可完成目标又违反边界。

## 与近期 AI / 评测论文的关系

- **ClawSafety: “Safe” LLMs, Unsafe Agents** — 2026-04-01，https://arxiv.org/abs/2604.01438 。120 个高权限 workplace scenarios、2,520 trials 显示 ASR 随 injection vector 与 framework 改变；HarnessRisk 把单次 runtime/action 风险扩到配置、持久化和恢复。
- **Your Agent, Their Asset** — 2026-04-06，https://arxiv.org/abs/2604.04759 。用 Capability–Identity–Knowledge 统一 OpenClaw persistent state，并在 live instance 上测 12 个场景；HarnessRisk 更广但使用 mock services，真实性与控制性取舍不同。
- **SafeHarness** — 2026-04-15，https://arxiv.org/abs/2604.13630 。提出 input filtering、causal verification、privilege-separated tools、rollback/adaptive degradation 四层防御；HarnessRisk 可作为跨生命周期覆盖框架，但没有直接评估这些防御的 paired intervention。
- **Auditing Agent Harness Safety** — 2026-05-14，https://arxiv.org/abs/2605.14271 。210 tasks 审计 boundary compliance、execution fidelity、system stability，强调中途资源/信息流违规；HarnessRisk 用 ASR/Persistence/Detection 补充后果、持久和恢复。
- **AgentS4D** — 2026-07-29，https://arxiv.org/abs/2607.27294 。328 cases × 20 configurations = 6,560 runs，用 risk source、induction strategy、target harm 和七个 checkpoint 追踪 runtime evidence；它比 HarnessRisk 的 taxonomy 更细，但同样是配置级而非 harness-only 因果比较。
- **MemSecBench** — 2026-07-29，https://arxiv.org/abs/2607.27080 。310 cases、24 configurations 用 Write–Execute–Forget 专门追踪 memory persistence、downstream consequence 和 selective repair；它是 HarnessRisk State Persistence/Recovery 阶段的深挖版。
- **HarnessSafe** — 2026-08-07，https://arxiv.org/abs/2608.06984 。328 executable cases 把 memory、skill、tool、shared artifact 等七类 persistent carrier 串成 entry→persistence→trigger→violation chain；相比 HarnessRisk 的四个终点标签，它更关注攻击在链上停在哪里。
- **EvoRiskBench** — 2026-10-02，https://arxiv.org/abs/2610.03153 。450 adversarial tasks 用 entry point–risk path–effect 构造持续演化的 workspace-agent benchmark；它扩大动态风险覆盖，但论文提交时 artifact 尚待 release。
- **HarnessSecurity-Bench v2** — 2026-10-10 更新，https://arxiv.org/abs/2610.07639 。对九类 native mechanism 做 ON/OFF 比较并同时测 Utility/ASR；它补足 HarnessRisk 最缺的 intervention 证据，而 HarnessRisk 补足其未覆盖的 persistence 与 incident recovery。

本次 `paper_search` 对 arXiv、DBLP、OpenAlex、OpenReview、Semantic Scholar 和 Crossref 发起四组检索，得到 237 条去重记录，合并 61 条跨源重复；DBLP 遇到反爬/TLS 错误，OpenReview 和 Semantic Scholar发生 429 重试。失败源只表示覆盖不完整，上述相邻论文的身份、日期和摘要关系另用 arXiv 或官方项目页核验。

## 对后续研究最有价值的设计

最值得做的小实验是 **Provenance-Carrying Recovery Bench**：选 12 个 HarnessRisk cases，每个包含一个可疑 artifact、一条后续 owner message、一次 memory/config write 和一个高影响 action。把第二轮消息随机成四组：不追加、只重复任务、口头背书、带可验证签名与明确 scope 的背书。随后结束 session，在新 session 触发 action，再要求另一个 agent 执行 recovery。

对每条 run 用 deterministic state/event oracles 分开记录：artifact 是否被读取、是否被写入 durable state、是否触发动作、是否识别、是否 containment、是否完全撤销、合法任务是否完成。LLM judge 只负责归类理由，不决定主标签。固定 model/harness 后，再逐一打开 provenance label、memory quarantine 和 recovery verifier。

停止条件应预注册：若 neutral 与口头背书没有差异，就停止声称“来源洗白”是主要机制；若 cryptographic scope 仍不能降低越权，说明 enforcement 没有消费 provenance；若 recovery verifier 降低 ASR/Persistence 但同幅破坏 Utility，则只称为 capability removal；若不同 harness 的差异在统一 prompt/tool/state 后消失，就不再主张 harness implementation 本身造成排名变化。

## 一句话总结

HarnessRisk 最有价值的贡献，是把 agent 安全从“本轮有没有拒绝攻击”扩展成贯穿配置、扩展、运行、持久化、行动与恢复的责任链，并显示任务完成和风险识别都可能与真实攻击后果共存；但它的主表仍依赖有 10–16% 抽样分歧的 LLM judge，跨 harness 不是单变量实验，owner 消息会参与来源洗白，且公开材料缺 raw result ledger，因此最可靠的后续方向是用可携带 provenance 与确定性 consequence/recovery oracles 做受控干预。
