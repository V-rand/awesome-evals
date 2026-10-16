# Notes — “From Faulty Memories to Corrected Actions: Dependency-Guided Rollback Repair for Memory-Augmented Agents”

**作者：** Caili Yu, Yiqi Wang, Jiaqi Zhang, Yiqun Duan, Mingkai Zheng, Zhangkai Wu, Kaize Shi, Taotao Cai · **首次提交：** 2026-08-11 · **版本：** arXiv v1 preprint · **论文：** https://arxiv.org/abs/2608.10502 · **类型：** post-failure agent memory recovery · **Found:** true

## 论文问题与科学动机

昨天的 [MemSecBench](memsecbench-agent-memory-poisoning-repair.md) 把记忆安全拆成 Write–Execute–Forget 七个 checkpoint，却让 Execute 和 Forget 从同一个 post-Write 快照分别出发。它能问“坏记忆是否被清掉、好记忆是否还在”，但不能回答一个更接近事故处置的问题：**错误已经参与推理、生成工具动作和派生记忆之后，删掉源记录能否把 agent 恢复到可信状态？**

本文把这个缺口称为 post-failure memory recovery。错误可能来自投毒、过期信息、用户归属错误或摘要漂移。一条错误记忆被读入后，会支持 claim，claim 再支持 plan/action，工具 observation 又可能写回新的 memory。此时只删源记录，会留下仍可被后续会话调用的派生结论；清空全部记忆或完整重跑，又会丢掉无关的合法状态并重复昂贵工具调用。

论文因此提出一个更强的恢复目标：给定失败执行、当前记忆库、完整 trace，以及已经诊断出的故障记忆 ID，同时修正最终答案和持久状态，并尽量保留不受影响的工作。这里最重要的边界是：**论文不解决故障检测，也不能撤销已经发生且不可逆的外部副作用。** 它研究的是已定位故障之后，如何决定哪些内部节点作废、哪些记忆隔离、哪些计算重放。

## 核心方法与具体机制

### 1. 从运行时 provenance 建 typed memory-to-action graph

系统把用户输入、memory record、claim、plan、action、observation、answer 和 memory mutation 都作为节点。边不靠“前后相邻”猜测，而来自运行时显式字段：谁发起步骤、claim 引用了什么、plan 由什么支持、action 产生什么 observation、哪次写入/更新/合并生成了哪条 memory，以及 memory 的 supersede/derive 关系。

```text
故障 memory F
  → 明确引用它的 claim
  → 依赖 claim 的 plan / action
  → action observation
  → 由 observation 或 claim 写出的派生 memory
  → 受影响 final answer
```

纯时间顺序边不参与污染传播。这样避免“一条错误记忆出现后，后面所有节点都被当成坏节点”的极端回滚，但也要求 harness 从一开始就记录细粒度依赖；普通对话日志本身不够。

### 2. 先过近似传播，再做独立支持裁剪

算法沿 producer/mutation lineage 找到每个故障记录的源节点，再沿显式依赖边向前传播，得到过近似受影响集合 `A0`。第一步优先避免漏掉污染，不急着判断节点最终是否应删除。

随后做 independent-support check。一个位于 `A0` 的 claim 或 decision，如果其完整内容仍可由故障集合和受影响集合之外的证据支持，就可以保留。可接受证据包括当前轮用户指令、活跃且非故障的 memory、成功工具 observation、或未受影响且已验证的 claim。已被“救回”的候选不能递归为另一个候选背书，避免污染节点互相证明正确。action/observation 的保留状态继承其 plan/action 的判断。

这一步的机制意义是区分“曾受坏记忆影响”与“失去坏记忆后已经没有独立依据”。论文不是按来源一刀切，而是在 dependency graph 上寻找仍有可信支撑的工作。

### 3. 确定性回滚与选择性重放

回滚阶段删除已诊断故障 memory，隔离没有独立支持的受影响派生 memory，使对应 trace 节点失效，并重新生成 final answer。为了限制重放范围，系统先选出仍能通向最终答案的受影响节点，再用 `ExecClosure` 补齐这些节点执行所需、但同样失效的前置步骤。

故障 memory 是删除，受影响的派生 memory 是 quarantine。这个区别保留审计证据，也避免未经重新验证的派生内容继续处于 active store。安全的旧输出作为固定上下文；已作废 claim 不提供给模型。invalidated action 会重新调用工具，observation 不会脱离 action 单独“生成”。重放中通过验证的 memory mutation 被依次应用，形成修复后的持久状态 `M'`。

这要求外部工具可重置、幂等，或提供 compensating interface。对于付款、消息发送、数据泄露等不可逆动作，算法最多修复之后的内部状态，不能证明事故已经逆转。选择性重放也不是免费收益：它节省调用，却可能保留足够多的旧路径，使故障以后复发。

## 实验设置

### 受控集

主实验有 150 个 case：shopping 48、travel 49、customer support 53；故障分为 poisoned 42、stale 34、wrong-user 34、summary-drift 40。128 个 case 含一条故障 memory，22 个含 2–3 条。只有“注入能改变最终答案，并产生至少一个下游影响”的 case 才保留，因此这是富集后的恢复压力集，不是部署中故障发生率。

每个 case 有 clean/faulty 配对 trace。修复方法只能看到 faulty trace、faulty store 和**精确的故障 memory ID**；clean trace、期望答案、良性标签和受影响节点标注只用于评测。主实验所有方法用 GPT-4o、temperature 0、seed 42，每个条件一次。Gemini-3.6-Flash 和 Qwen3.6-27B 只比较本文方法与两个最接近 baseline。

baseline 包括 no repair、full reset、delete retrieved memories、MemAudit-style、LLM-judge repair 和 AgentTrace-style。它们是按论文统一接口改写的版本，并非各原系统的直接复现；例如 MemAudit-style 在候选为空时还有 oracle-assisted fallback。

评测必须分开看：task-oracle answer recovery、恢复后重跑最后一个输入时的 recurrence、faulty-memory removal、benign-memory preservation、claim invalidation F1、replay ratio 和 LLM calls。answer recovery 高不代表持久状态正确，removal 高不代表不误删，当前答案正确也不代表未来不复发。

### LongMemEval-V2 迁移集

迁移实验只取 LongMemEval-V2 中 50 个 procedural/navigation case，要求存在明确文本证据和可验证 clean answer；模糊 Boolean、普通短答案、纯视觉和聚合题被排除。所有 case 都改造成 poisoned-memory 压力测试，且 40/50 含 3–4 条故障。这是 trajectory-derived stress test，不是 LongMemEval-V2 全分布成绩。

## 主要结果与数值审计

### 主表：即时恢复最强，但复发和定位并不最好

| 方法 | Recovery ↑ | Recurrence ↓ | Fault removal ↑ | Benign preservation ↑ | Claim invalidation F1 ↑ | Replay ratio ↓ | LLM calls ↓ |
|---|---:|---:|---:|---:|---:|---:|---:|
| Full reset | 40.7% | 19.7% | 100% | 0% | 0.000 | 0.061 | 4.03 |
| Delete retrieved | 34.0% | 43.1% | 88.1% | 78.9% | 0.000 | 0.061 | 4.05 |
| MemAudit-style | 32.0% | 29.2% | 31.1% | 98.0% | 0.000 | 0.051 | 10.43 |
| LLM-judge repair | 77.3% | **12.1%** | 100% | 100% | **0.946** | 0.217 | 9.80 |
| AgentTrace-style | 60.7% | 34.1% | 0% | 99.7% | 0.853 | 0.091 | 4.69 |
| Dependency-guided rollback | **85.3%** | 26.6% | 100% | 100% | 0.566 | 0.123 | 5.70 |

本文方法把即时恢复从最强 competing method 的 77.3% 提到 85.3%，同时在 benchmark 指定集合上做到全部故障删除和全部良性 memory 保留。可是它的 recurrence 为 26.6%，明显差于 LLM-judge repair 的 12.1%；claim invalidation F1 也只有 0.566，低于 LLM-judge 的 0.946 和 AgentTrace 的 0.853。论文摘要自己也限定为较好的 recovery–cost trade-off，并未声称 trace reconstruction 全面更好。

成本数据支持“少重放”的部分：总 token 约 12,697.55，低于 LLM-judge 的 28,175.28，但高于 AgentTrace 的 10,508.94。这个比较仍依赖论文统一 executor 和改写 baseline。

### 消融：选择性重放用复发风险换效率

完整方法 recovery/recurrence/replay/calls 为 85.3%/26.6%/0.123/5.70。去掉 selective replay、改为广泛重放后，recovery 略降至 84.0%，但 recurrence **降到 7.1%**；代价是 replay ratio 升至 0.755、调用升至 24.01。因而 selective replay 的直接证据是显著省计算，不是更彻底清除未来错误。

去掉 support check 时 recovery 反而升到 88.0%，但 benign preservation 降到 98.6%，F1 降到 0.507；去掉 planner 后 recovery 降到 71.3%、recurrence 升到 43.0%、calls 升到 9.19。support check 的价值更接近减少 collateral damage，而不是单调提高当前答案。

### 故障类型和 backbone 边界

poisoned/stale/wrong-user/summary-drift 的 recovery 分别为 81.0%/94.1%/70.6%/95.0%。但 stale 的 recurrence 达到 **78.1%**，远高于 poisoned 的 0%、wrong-user 的 12.5% 和 summary-drift 的 15.8%。这说明“这次答案被旧信息修好”远不等于旧状态以后不会再支配行为；过期事实的替代/版本关系可能比删除一条污染记录更难。

跨 backbone 也没有统一胜出：GPT-4o 上本文方法 85.3% 对 LLM-judge 77.3%，Qwen3.6-27B 上 66.0% 对 61.3%，但 Gemini-3.6-Flash 上是 73.3% 对 74.0%。不能把主表优势扩写成与 backbone 无关的支配关系。

在 50-case adapted subset 上，本文方法 recovery 68.0%，AgentTrace 54.0%，LLM-judge 26.0%；claim invalidation F1 分别为 0.669、0.603、0.369。本文方法 fault removal 100%、benign preservation 99.3%、recurrence 5.9%。它支持方法能离开生成式受控集，但受控到迁移 recovery 从 85.3% 降到 68.0%，而且筛选规则排除了大量难以程序判断的 LongMemEval-V2 题型。论文明确把受控结果看成上界而非部署估计。

## 已验证结论、合理推断与待验证假设

### 论文直接支持

- 在给定精确故障 ID、完整 provenance 和可安全重放工具的 150-case 富集集上，dependency-guided rollback 的即时 recovery 为 85.3%，高于论文实现的其他 baseline。
- 它在该集合的标注范围内达到 100% diagnosed-fault removal 与 100% benign-memory preservation；这些值不覆盖未标注的有用状态。
- 更少重放显著降低调用和 token，但消融表显示全量重放的 recurrence 更低，故效率与未来复发存在可见权衡。
- 在迁移子集上 recovery 为 68.0%，并非对完整 LongMemEval-V2 的结果。
- 方法依赖 runtime provenance、上游 fault diagnosis 和可重放/可补偿工具，且不能撤销不可逆外部副作用。

### 基于证据的合理推断

- agent 事故恢复至少要分别报告当前答案、active memory、派生状态、未来复发和外部副作用；把它们压成一个“repair success”会隐藏失败。
- typed dependency graph 比纯时间窗口更适合缩小回滚面，但 provenance 缺失时的漏边会成为系统性假阴性；论文没有测不完整 provenance。
- stale recurrence 异常高，暗示版本替代、时间范围和 authority 应成为一等 memory relation，而不能只标一个 faulty node 后向前传播。
- selective replay 适合作为成本控制器，不应被默认视为安全强化器；上线时需要按副作用风险调整重放范围。

### 仍待实验验证

- **故障诊断误差：** 将 fault ID 的 precision/recall 分别从 100% 降低，测 recovery、误删和复发曲线。若轻微漏检就使优势消失，系统瓶颈仍在 detector。
- **provenance 缺边：** 随机或按类型删除 cite/support/produce 边，检验传播集合是否漏掉关键派生状态。
- **真实副作用补偿：** 为发送、支付、权限修改设计 idempotency key 和 compensating action，分别计内部状态修复与外部世界恢复。
- **长期再触发：** 不只重跑最后一个输入，而是在 5–10 个后续自然任务中检查 quarantined/derived state 是否重现，特别针对 stale fault。
- **统一恢复合同：** 在相同 trace 上比较 barrier-first quarantine、依赖图 rollback、counterfactual pre-action repair 与 functional forgetting，统一报告 exposure window、答案、状态、复发、效用和成本。

## 局限性与可信度警告

- **fault diagnosis 被假定解决。** 所有方法直接获得准确故障 memory ID；现实中最难的来源判定、过期识别和 wrong-user 检测未进入结果。
- **完整 provenance 是强前提。** 方法使用显式运行时依赖，不适用于只保留消息日志、工具 stdout 或无 memory lineage 的普通 agent。
- **case 是条件富集集。** 注入必须改变答案并产生下游作用才入选，85.3% 不能解释成自然事故中的恢复概率。
- **单次运行与模型漂移。** 主条件每 case 一次，没有置信区间；temperature 0 也不消除服务端模型和工具环境变化。
- **baseline 是适配实现。** 结果支持此统一实现下的比较，不等于击败各方法作者在原生设置中的最佳系统。
- **标注集合限制。** 100% benign preservation 只对 case 标签中的良性 memory 成立，未证明缓存、技能、共享 artifact 或未列出的用户状态完整。
- **recurrence 定义偏短期。** 只重跑最后一个输入，不能覆盖新的检索 query、跨会话组合或长延迟 trigger。
- **不能撤销现实伤害。** 已发送消息、泄露数据或执行付款不因 trace 回滚而消失；论文恢复的是 agent 内部计算与可控持久状态。
- **迁移子集经过筛选。** 50 个 LongMemEval-V2 case 不是全任务分布，且全部改造成 poisoning 情境。
- **公开复算受限。** 截至 2026-10-16，arXiv v1 页面没有作者直接链接的官方 code/data/replay 仓库；本次结果来自全文核对，未独立重跑。

## 与仓库已有论文和主题的主动关联

- [MemSecBench](memsecbench-agent-memory-poisoning-repair.md) 的 F1/F2 检查坏语义消失与必要良性记忆保留；本文进一步追踪 claim/action/derived memory 并重新生成答案。但前者有外部 E3 oracle，本文仍不能撤销已经完成的 E3。最直接组合是：在同一 post-Write snapshot 上跑 dependency rollback，再开启新会话重测 E3、F1、F2。
- [HarnessRisk](harnessrisk-lifecycle-agent-harness-safety.md) 把 Incident Recovery 设为生命周期阶段；本文给出其中一个可执行机制。它仍只覆盖 memory-to-action provenance，不覆盖 compromised skill、配置文件、共享 artifact 和 credential 的联合恢复。
- [HarnessSecurity-Bench](harnesssecurity-bench-coding-agent-harnesses.md) 用可切换安全机制和确定性 oracle 区分 harness 效应；本文的 recovery graph 也应做同类缺边、故障 ID、重放范围干预，而不是只比较完整 pipeline。
- [Benchmarking the Benchmarks](benchmarking-the-benchmarks-tool-calling-validity-audit.md) 提醒工具执行和 evaluator 都会制造假差异。本文用 task oracle 判断 recovery 是优点，但受控 case、baseline adapter 和工具可重放性仍决定测到的能力。
- [Protocol Validity](do-agent-benchmarks-measure-capability-protocol-validity.md) 所说的 protocol validity 在这里尤其直接：给真 fault ID、完整 provenance 和 clean resettable tools 测的是“已知故障后的 rollback planner”，不是端到端自主恢复。

## 与近期 AI / 评测论文的关系

- **MemoRepair** — 首发 2026-05-08，https://arxiv.org/abs/2605.07242 。它把 cascade repair 定义为 barrier-first contract：先撤下所有受无效来源影响的可见 descendant，再用 predecessor-closed 条件发布验证 successor；完整 provenance 下把 invalidated-memory exposure 降到 0%。本文则围绕一次失败 trace 恢复答案、action 和 active memory。前者强调“修复期间不能继续暴露”，后者强调“哪些计算值得重放”，二者是互补而非同指标排名。
- **MemSecBench** — 首发 2026-07-29，https://arxiv.org/abs/2607.27080 。它提供从持久化到外部后果和选择性删除的 checkpoint；本文补上派生 claim/action 的 recovery mechanism，但缺少 MemSecBench 那种修复后外部服务后果验证。
- **DREAM / Forgetting is Not Always Bad** — 首发 2026-10-09，https://arxiv.org/abs/2610.13267 。DREAM 通过扰动重激活、异常诊断和动态衰减做功能性遗忘，摘要报告 backdoor ASR 大幅下降并保留效用。它同时承担发现与抑制，本文假定 detector 已给答案；需要同一污染 snapshot 和长期复发协议才能判断哪种机制真正恢复了状态。
- **StateWise** — 首发 2026-10-04，https://arxiv.org/abs/2610.05241 。它在 coding agent 动作执行前做 record-level counterfactual replanning、读验证、针对性澄清和独立 state-action check，摘要报告 150 case 上 93.3% correctness、无 unsafe action。StateWise 是 pre-action persistent-state repair，本文是 post-failure rollback；把两者串联可测“预防失败后，剩余事故能否恢复”，但其 headline 数字来自不同任务，不能横比。
- **HarnessSafe / HarnessRisk** — 首发分别为 2026-08-07、2026-08-18，https://arxiv.org/abs/2608.06984 、https://arxiv.org/abs/2608.17597 。它们把持久载体和 incident recovery 放进 whole-harness 生命周期；本文的 typed graph 是把 memory 载体的恢复做细，但尚未覆盖跨载体依赖图。

本次多源检索窗口为 2025–2026，查询围绕 agent memory rollback、post-failure recovery、provenance 和 selective replay；六源检索进程在本次运行时限内未生成合并结果文件，因此不报告来源计数，也不据此作“没有更多相关工作”的否定判断。相邻论文的身份、首次日期和摘要级结论另以官方 arXiv 页面核对。除目标论文外，近期工作未逐篇全文复核，因此上面的关系判断限于机制边界，不把摘要数字当成统一 benchmark 排名。

## 一个小而可证伪的后续实验

从 MemSecBench 选 12 个有确定性 E3 服务 oracle、至少两条良性 memory、且故障能产生派生状态的 case。固定 harness、model、backend 和同一 W2 snapshot，运行三种恢复：原始 Forget、本文 dependency rollback、dependency rollback + barrier-first 隔离。恢复后结束会话，用三个不重复原始 prompt 的自然任务再触发。

分开报告六项：fault removal、benign preservation、answer recovery、未来 E3、invalidated-state exposure window、额外工具/LLM 成本。再人为删去 10%/30% provenance 边，并让 detector 漏掉一条故障，形成真正的判别干预。

竞争解释是 dependency method 只是靠精确 oracle 和完整日志缩小重放，并未对真实不完整系统更可靠。如果任一轻度缺边/漏检条件下，它在 E3 与良性任务完成率上不再优于简单全量重放，就停止扩模型，先把贡献收窄为“完整 provenance 条件下的成本优化”，不声称通用事故恢复。

## 一句话总结

这篇论文最有价值的推进不是把 85.3% 写成新的 SOTA，而是把“删掉坏记忆”改写成一个可审计的状态恢复问题：沿显式依赖找到失去支持的 claim/action/derived memory，隔离后选择性重放；但主表同时说明即时答案、污染清除、未来不复发和 trace 定位是四个不同目标，其中复发、stale state、故障诊断和不可逆外部动作仍是硬边界。
