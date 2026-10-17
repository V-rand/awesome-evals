# Notes — “MEMOREPAIR: Barrier-First Cascade Repair in Agentic Memory”

**作者：** Yang Zhao, Chengxiao Dai, Mengying Kou, Yue Xiu · **首次提交：** 2026-05-08 · **版本：** arXiv v1 preprint · **论文：** https://arxiv.org/abs/2605.07242 · **类型：** agentic-memory cascade repair contract · **Found:** true

## 论文问题与科学动机

昨天的 [Dependency-Guided Rollback Repair](dependency-guided-rollback-repair-agent-memory.md) 研究失败之后如何沿 memory-to-action graph 作废 claim、action 和派生记忆，再选择性重放。但它把关注点放在“最终该重算什么”，没有单独回答一个更早的问题：**修复尚未完成时，那些已经失去来源支撑的 summary、cache、skill 和 tool procedure 还能否继续被检索和执行？**

MEMOREPAIR 把这类失败称为 cascade update problem。源 artifact 被删除、纠正，或因 API/tool migration 失效，并不意味着其后代自动消失。一条过期 API contract 仍可能藏在 chain procedure；已更正的 tool response 仍可能存在 cache；被删除的用户偏好仍可能保留在 summary 或 neural skill 中。系统若边修边继续服务，会出现一个窗口：根已经无效，派生状态仍可见。

论文的核心立场是：修复首先是一个**可见性合同**，其次才是一个生成问题。事件到来后，应先把整个 affected cascade 撤出服务，再构造 successor；只有通过验证、且所有必需前驱也已成功修复的 successor 才能重新发布。这个顺序把“修复过程中继续暴露 stale state”变成显式安全属性。

## 核心方法与具体机制

### 1. 两类边：influence 决定修复范围，semantic 只用于检索

持久记忆表示成图 `G=(V, E_inf, E_sem)`。节点是 record、cache、summary 或 skill；skill 再区分 neural、prompt、chain。`E_inf` 表示“后者生成时把前者当作因果支撑”，用来计算 cascade；`E_sem` 只表示检索相似性，不能扩大修复范围。

修复事件是 `e=(F, τ, Δ)`：`F` 是无效根集合，`τ` 为 delete/correct/migrate，`Δ` 提供更正内容或迁移信息。沿 influence edge 的可达闭包为 `C(F)`，去掉根后的 descendant 为 `D(F)`。

这个区分很重要：相似内容不一定依赖同一来源；反过来，如果生成时没有记录 influence edge，后续即使语义上明显受影响，也不会进入撤回范围。论文的保证因此完全取决于 provenance 的完整性。

### 2. Withdrawal barrier 先于任何修复

事件被确认后，系统立即令 `Bρ ← C(F)`，把根和所有可达 descendant 标为 non-servable。新的 successor 只有在对应 `Validate_i` 通过后才能发布。对不同 artifact，验证内容不同：重算项检查 replay consistency；再生成项检查 schema 和 task regression；chain skill 在 sandbox 中检查行为；parametric repair 检查 forget/reference 行为。

barrier 不提高 artifact 本身的正确性，它只是保证修复期间旧版本不再被服务。可用性代价是，在 successor 尚未验证前，相关能力会暂时缺失。论文消融显示取消 barrier 可把 repair time 降到 0.52×，但修复期间 Leak 和 Stale-use 都回到 100%。这是 availability–correctness 取舍，不是免费安全机制。

### 3. Artifact-aware candidate construction

每个 affected descendant 由确定性 mode map 分到四类：

- `recompute`：可重放的确定性 record/cache；
- `regen`：在保留支撑足够时重建 summary、prompt skill 或 chain procedure；
- `param`：有 forget/reference provenance partition 时修复 neural skill；
- `remove`：缺支撑或无法安全修复的 artifact。

每个非 remove 候选记作 `q_i=(μ_i, π_i, P_i, v_i, w_i, c_i)`：修复模式、operator、所需已修复前驱、可执行标志、价值和成本。如果它依赖一个不可修复的 affected predecessor，就直接不可执行。候选之间形成新的 repair-time dependency graph；这不是原始 provenance 图，而是“发布 successor 前必须先有哪些 repaired successor”的依赖。

### 4. 从发布合同到 min-cut

系统在固定 trade-off `λ` 下选择候选：最大化 `Σw_i x_i - λΣc_i x_i`，同时要求不可执行项不能入选，并且选中一个 successor 时，其所有 repaired predecessor 也必须入选。这个 predecessor-closed 选择可转成 maximum-weight closure，再由一次 `s-t` min-cut 精确求解。

论文明确承认 min-cut 本身是经典结果；贡献在于 barrier、candidate、validation、predecessor closure 共同把 agent memory 的 successor publication 变成了这一优化问题。若改成硬预算上限，则一般成为 precedence-constrained knapsack；主实验没有求这个版本，而固定 `λ=0.3`。

### 5. 发布失败要向后传播

选中的候选按 repair dependency 的拓扑序执行。只有验证通过的 successor 重新进入服务；失败项以及依赖它的候选继续 withdrawn。这样“某个父节点修复失败”不会让下游以残缺支撑发布。

## 实验设置

论文在 ToolBench（经 StableToolBench 执行）和 MemoryArena 上构造 deletion、correction、migration 事件。artifact 覆盖 API call、cached output、executable procedure、偏好、episodic record 和 retrieval trace；neural skill 使用 ToolLLaMA-7B-LoRA-v1，并比较多种 machine-unlearning operator。

内部 baseline 包括：

- `No action`：什么都不改；
- `Remove all`：撤回 cascade，但不修复/重发；
- `Barrier + Greedy`：共享 barrier、候选、operator 和 validator，只把 min-cut 换成价值/成本贪心；
- `Repair all`：执行所有初始可执行候选，并只发布验证通过者，作为 publication ceiling。

外部系统 Mem0、Zep/Graphiti、MemR3、TierMem、A-MEM、O-Mem 按各自原生更新/检索语义运行。作者明确把它们称为 stale-resistance reference points，而不是同合同下的 repair baseline：它们没有 withdrawal、successor validation 或 predecessor-closed republication，`Rep.=0` 表示没有执行这类发布操作，不是修复尝试失败。

五个主指标是：affected cascade 中修复后仍可服务的 `Leak%`；post-event action 使用无效信息的 `Stale-use%`；在非 remove repair contract 上的 validated republication rate `Rep.%`；归一化 operator `Cost`；以及诊断性的 `ΔTask`。

必须特别说明：`ΔTask` **不是 ToolBench/MemoryArena 的原生任务成功率**。对每个 task，它从被使用 artifact 中“没有 validated successor 的比例”和“stale version 仍可见的比例”各扣一半，再对任务平均。它衡量该形式化合同下的状态缺失/污染，不是端到端真实任务正确率。

## 主要结果与数值审计

### 1. 在完整 provenance 下，0% Leak/Stale 是合同结果

对所有使用 withdrawal barrier 的内部方法，主表 Leak 和 Stale-use 都是 0。原因不是模型在实验中成功识别了所有 stale artifact，而是实验已给出 complete influence provenance，`C(F)` 被整体设为 non-servable。真正有经验差异的是：撤下之后能发布多少 validated successor、成本多大、诊断分数损失多少。

六个不具备 cascade contract 的外部系统在 ToolBench 仍留下 69.8%–93.1% affected cascade 可见，92.4%–99.7% post-event action 使用 stale 信息；MemoryArena 对应 73.9%–94.3% 和 93.2%–99.6%。这说明其原生更新/检索不足以模拟 cascade withdrawal，但不能解释为同等工程投入下 MEMOREPAIR 对六个系统的公平胜率。

### 2. Min-cut 以约六成成本逼近 Repair all ceiling

| Benchmark / event | Repair all Rep. | MEMOREPAIR Rep. | Ceiling recovered | MEMOREPAIR Cost | ΔTask gap vs Repair all |
|---|---:|---:|---:|---:|---:|
| ToolBench delete | 24.6% | 22.4% | 91.1% | 0.61 | -0.06 pt |
| ToolBench correct | 48.8% | 45.1% | 92.4% | 0.57 | -0.06 pt |
| ToolBench migrate | 38.7% | 36.5% | 94.3% | 0.66 | -0.08 pt |
| MemoryArena delete | 5.5% | 5.1% | 92.7% | 0.62 | -0.05 pt |
| MemoryArena correct | 35.6% | 33.1% | 93.0% | 0.76 | -0.07 pt |
| MemoryArena migrate | 18.4% | 17.0% | 92.4% | 0.64 | -0.05 pt |

`λ=0.3` 在 ToolBench 三种 event 的三 seed frontier 上位于作者定义的 knee；越过该点后 `ΔRep/ΔCost` 边际斜率下降 4–7 倍。与 Greedy 相比，min-cut 的主要增益是利用 predecessor closure 做全局选择，而不是逐个按局部价值/成本排序。

但 Rep. 的分母只包含 non-remove contract，且 `Repair all` 自己也只发布验证通过者；因此“恢复 91.1%–94.3% successor”准确含义是**接近论文 validator 所允许的 publication ceiling**，不是恢复了原系统 91% 的全部能力。MemoryArena deletion 的绝对 Rep. 只有 5.1%，就是这个边界最直观的例子。

### 3. Store repair 与 parameter unlearning 修不同层

在 ToolBench neural-skill 子集上，无修复时 Stale=100、forget-set suppression (FSP)=8.7、`ΔTask=-7.42`。只用 LUNE 把 FSP 提到 85.7，却仍有 Stale=83.4；只做 cascade repair 把 Stale 降到 0，FSP 仅 12.5。组合后 Stale=0、FSP=86.4、`ΔTask=-1.86`，Cost 0.74，低于两个独立运行成本之和 0.80。

这个结果支持“可见派生物”和“参数内影响”是两条不同路径。它不证明 exact erasure：论文明确说 neural skill 继承具体 unlearning operator 的保证与失败模式。

### 4. 1% provenance 缺边会放大成 17.7% 泄漏

随机丢弃 influence edge 比例 0.5%/1%/2%/5% 时，Leak 分别为 8.6%/17.7%/34.2%/61.8%，Stale-use 为 9.4%/19.7%/38.0%/68.5%。小比例缺边出现约 18× 放大，直接说明 0% 保证不是对 provenance noise 鲁棒的性质。

validation 也不是装饰：schema-only 的 stale false-pass 为 88.6%，task-regression-only 为 29.8%，二者合并才在该测试中为 0。可是“通过已实现测试”仍不保证未来所有上下文语义正确。

## 已验证结论、合理推断与待验证假设

### 论文直接支持

- 给定准确 invalidated root、完整 influence provenance 和同步 barrier，affected cascade 可按合同立即变为 non-servable；主表的 0% Leak/Stale 是这个条件下的结果。
- 固定 `λ=0.3` 时，min-cut selector 以 0.57–0.76 的归一化 repair-operator cost 找回 91.1%–94.3% 的 `Repair all` validated-publication ceiling。
- Greedy 与 min-cut 共享 barrier、candidate、operator、validator，因此两者差异可较干净地归于 selector。
- store-level cascade repair 和 parameter-level unlearning 解决不同残留路径；组合在论文 neural-skill 子集上最好。
- provenance 缺边显著破坏保证；1% edge drop 对应 17.7% Leak 和 19.7% Stale-use。

### 基于证据的合理推断

- 安全修复需要把“撤出服务”和“重新构造”分成两个状态转换；直接原位改写会留下 repair-time exposure window。
- provenance 完整度不是普通 metadata 质量指标，而是 barrier 能否覆盖真实 cascade 的安全边界。
- predecessor-closed publication 可作为 dependency-guided rollback 的外层提交协议：先隔离，再选择/重放，再原子发布通过验证的 closure。
- `Remove all` 的 0% Leak 和 0% Stale 表明这两个指标单独会奖励彻底失能；必须与 publication、真实任务效用和 availability 一起报告。

### 仍待实验验证

- **真实任务成功率：** 用 ToolBench/MemoryArena 原生 end-to-end 指标重跑，而不是只用由 artifact 状态构造的 `ΔTask`。
- **现实 provenance：** 从真实 harness 日志估计 influence edge precision/recall，再按结构缺边而不是均匀随机 drop 测 cascade 漏检。
- **事件检测延迟：** 从根失效到 barrier 生效之间仍可能有动作；需要测 detection latency、in-flight action 和已经外部化副作用。
- **validator 泛化：** 用 held-out trigger、对抗性输入和跨会话任务检验 validated successor 是否真的不再依赖旧语义。
- **可用性成本：** 记录 withdrawal duration、被阻塞请求、fallback 行为和用户任务损失，而不只报 operator cost。
- **跨载体 cascade：** 将 memory、skill、config、credential 和 shared artifact 放进同一 influence graph，检查 whole-harness recovery。

## 局限性与可信度警告

- **缺少实验规模细节。** 论文没有报告主实验包含多少 repair event、每类 event 数量、cascade size 分布明细或构造/筛选流程；只有“从少量后代到几十个 artifact”的描述。读者无法从论文判断点估计的不确定性。
- **主表重复性不清楚。** repair–cost frontier 报三 seed 均值与标准差，但 Table 1/2 没有给 run 数、区间或显著性。
- **`ΔTask` 是自定义诊断分数。** 它由 missing/stale artifact 比例定义，不能替代 benchmark 原生任务完成率，也与 Leak/Stale 共享构造信号。
- **0% 是强前提下的合同保证。** complete provenance、已知 invalidated root、同步撤回和正确服务层拦截都先被假定；Table 4 已显示轻微缺边即可破坏结论。
- **外部系统不是等合同 baseline。** 六个系统没有被赋予 barrier、validator 或 successor publication，`Rep.=0` 也只是指标不适用；不能据此声称其修复算法全面落后。
- **价值和成本来源不透明。** `w_i/c_i` 如何从实际 workload 确定、对结果是否敏感，正文缺少足够实现细节；固定 `λ=0.3` 的 knee 只在所构造 frontier 上成立。
- **验证 oracle 决定 publication ceiling。** schema、regression、sandbox 与 forget/reference 检查不完备，既可能漏放 stale successor，也可能误拒可用 successor。
- **外部副作用不在合同内。** barrier 管的是 memory store 中 artifact 的 servability；已发送消息、执行交易或泄露信息不会被撤销。
- **公开复算受限。** 截至 2026-10-17，arXiv v1 页面没有作者直接链接的官方 code、event corpus 或结果 ledger；精确复现 min-cut 输入、validator 和六个外部系统适配仍不可行。

## 与仓库已有论文和主题的主动关联

- [Dependency-Guided Rollback Repair](dependency-guided-rollback-repair-agent-memory.md) 解决“失败后哪些 computation/state 要作废和重放”，MEMOREPAIR 解决“修复完成前哪些 artifact 不得继续服务”。最自然的组合是 barrier 先冻结 cascade，rollback 生成候选，predecessor closure 再原子发布。两者都依赖 provenance，但 MEMOREPAIR 的 1% 缺边消融把这个风险量化得更直接。
- [MemSecBench](memsecbench-agent-memory-poisoning-repair.md) 的 F1/F2 要求清除恶意语义并保留规定的良性项；MEMOREPAIR 的 Leak/Stale/Rep 更适合审计修复过程和派生 artifact。前者有服务端 E3 consequence，后者没有真实外部后果撤销，二者可互补。
- [HarnessRisk](harnessrisk-lifecycle-agent-harness-safety.md) 把 Incident Recovery 放在 whole-harness 生命周期；MEMOREPAIR 给出 memory store 内的提交合同，但对配置、extension、credential 和 action side effect 没有统一 cascade。
- [HarnessSecurity-Bench](harnesssecurity-bench-coding-agent-harnesses.md) 强调机制必须用可切换干预和确定性后果 oracle 测量。MEMOREPAIR 的 barrier/validator/provenance 消融符合这一方向，但主 `ΔTask` 仍不是端到端任务 oracle。
- [Protocol Validity](do-agent-benchmarks-measure-capability-protocol-validity.md) 提醒协议会决定“能力”定义；这里若预先提供完整 graph 和 invalid root，评测的是 repair selection/commit contract，而非自主检测和端到端事故恢复。

## 与近期 AI / 评测论文的关系

- **Agentic Unlearning** — 首发 2026-02-06，v2 2026-02-28，https://arxiv.org/abs/2602.17692 。SBU 同时处理 parameter 和 persistent-memory pathway，并用 dependency closure 防止跨路径回流。MEMOREPAIR 把 store-side visible successor publication 形式化得更细，并可把 SBU 当 param operator；两者共享的难点是 provenance/forget-set 是否真实完整。
- **TierMem / From Lossy to Verified** — 首发 2026-02-20，https://arxiv.org/abs/2602.17913 。TierMem 用 summary 默认回答、证据不足时回到 immutable raw log，并把 verified finding 写回有 provenance 的 summary。它主要解决写时压缩丢证据，MEMOREPAIR 解决来源后来失效；前者的 source link 正是后者 cascade repair 所需基础设施。
- **Dependency-Guided Rollback Repair** — 首发 2026-08-11，https://arxiv.org/abs/2608.10502 。后者在 150+50 个 case 上恢复答案、trace 和 active memory，并暴露 selective replay 的 recurrence 权衡；MEMOREPAIR 更早关注 repair-time exposure 和 atomic republication。两篇论文的指标不可横比。
- **StateWise** — 首发 2026-10-04，https://arxiv.org/abs/2610.05241 。StateWise 在 coding-agent 动作前用 counterfactual replanning、read-only verification 和 targeted clarification 修正 operational record；它补 MEMOREPAIR“失效根已知”的检测缺口，但需要测试其 repair lineage 能否支持完整 cascade withdrawal。
- **DREAM** — 首发 2026-10-09，https://arxiv.org/abs/2610.13267 。DREAM 通过 reactivation、anomaly diagnosis 和 attenuation 做 poisoned-memory functional forgetting。它报告攻击和效用，MEMOREPAIR 报可见 artifact 与 successor 发布；共享 snapshot、held-out trigger 和 availability 指标才能比较。

本次六源 paper-search 进程在运行时限内仍未生成合并 JSON，已停止；这只表示检索覆盖不完整，不能据此作“没有更多相关工作”的判断。上述论文的身份、首发日期和摘要级结论另以官方 arXiv 页面核对；除目标外未逐篇阅读全文。

## 一个小而可证伪的后续实验

在 12 个 MemSecBench case 上显式记录 source→summary/cache/skill/action influence edges。先让同一 poisoned snapshot 产生至少两层 descendant，再触发 root deletion。随机分配三种修复：原地删除、barrier + Repair all、barrier + min-cut + dependency rollback。修复期间持续发起 benign/trigger request，修复后再开新会话，并用服务端 E3 oracle 检查外部动作。

必须分开报：barrier 生效前 stale action 数、repair-time exposure、withdrawal duration、原生任务完成率、F1/F2、未来 E3、operator/token cost。再分别丢弃 1% 的高出度 edge 与随机 edge，判断论文的均匀 drop 曲线是否低估结构性 provenance 风险。

竞争解释是 0% Leak/Stale 仅来自“知道完整 closure 后全部下线”，min-cut 只在自定义 `w/c` 下压低成本，并未改善现实任务。如果加入原生任务 oracle 和结构性缺边后，min-cut 在可用性–安全性前沿不优于 Repair all，就停止扩模型，把贡献收窄为 barrier contract，而不声称已有部署级最优修复器。

## 一句话总结

MEMOREPAIR 最有价值的贡献不是 0% Leak 这个强前提下的数字，而是把修复定义成一次受控提交：先撤下整个受影响 cascade，再只发布通过验证且 predecessor-closed 的 successor；min-cut 则在这个合同内减少 operator cost。它也用 1% provenance 缺边→17.7% Leak 清楚表明，真正的系统瓶颈可能不是 selector，而是能否持续获得可信、完整的 influence graph 和端到端 validator。
