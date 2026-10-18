# Notes — “From Lossy to Verified: A Provenance-Aware Tiered Memory for Agents”

**作者：** Qiming Zhu, Shunian Chen, Rui Yu, Zhehao Wu, Benyou Wang · **首次提交：** 2026-02-20 · **版本：** arXiv v1 preprint · **论文：** https://arxiv.org/abs/2602.17913 · **官方实现：** https://github.com/FreedomIntelligence/Tiermem · **类型：** provenance-linked tiered agent memory · **Found:** true

> 状态说明：本文按 22 页 arXiv v1 全文审读。官方仓库此后把会议版题目更新为 *TierMem: Balancing Compressed Memory and Raw Evidence for Long-Horizon Agent Memory*，并标注 COLM 2026 接收；截至 2026-10-18，arXiv 仍只有 v1。下文数值来自 arXiv v1，不把后续 README 当作已复核的会议版实验。

## 论文问题与科学动机

长程 Agent 常把历史压成摘要，再丢弃或冷藏原始交互。这样省 token，却有一个根本矛盾：**系统写摘要时，还不知道未来问题会依赖哪个细节。** 一次合理压缩可能留下“用户有饮食限制”，却删掉具体过敏原；留下“配置改过”，却删掉精确版本和失败日志。未来问题一旦命中被删细节，问题不是“检索没找好”，而是证据已经不在可检索表示里。

论文称之为 `write-before-query barrier`。完整保留 raw log 可恢复可审计证据，但每个问题都读长上下文会放大 token 与时延。TierMem 因而问：**能否默认使用便宜摘要，只在当前问题的证据不足时，沿来源链接回到不可变原文？**

这也给昨天的 [MEMOREPAIR](memorepair-barrier-first-cascade-repair.md) 补上底层前提。MEMOREPAIR 研究来源失效后如何沿 influence provenance 撤回派生物；TierMem 研究来源仍有效、摘要却丢细节时，source link 能否把查询带回原始证据。两者都要求 provenance 是可执行基础设施，而非展示用引用。

## 核心方法与具体机制

### 1. 两层存储：摘要不是 source of truth

Tier-2 是不可变分页 raw log。交互以统一时间戳、speaker 和文本写入，约每 1,000 token 封成一页，并分配稳定 `raw_log_id/page_id`。Tier-1 是可检索 summary/fact index：每个条目有摘要、embedding 和指向 Tier-2 页的 provenance pointer。

新 raw page 封存时，系统生成页级摘要，并把来源 ID 确定性写入 Mem0 metadata。来源 ID 不是由模型随手生成，避免伪造 provenance。Tier-1 是便宜候选证据和导航层，Tier-2 才是作者定义的 authoritative source。

### 2. Sufficiency router 分配证据预算

查询先检索 top-k Tier-1 摘要。Qwen3-0.6B router 观察问题与摘要证据，只做二分类：`S/ANSWER` 表示摘要足够，走快路径；`R/ESCALATE` 表示存在信息缺口、歧义或精度不足，进入 raw path。它预测的不是问题“难不难”，而是**当前证据对当前问题是否充分**。

### 3. Linked warm start + 有界主动检索

升级后先读摘要直接链接的 raw pages；若仍不足，再做至多三轮 `retrieve → integrate → plan`，用 Mem0 semantic search 与 BM25 keyword search 补页，并抽取 `(fact, evidence quote)`。

quote 到 source page 的映射由确定性后处理完成：先 substring match，再 keyword overlap。生成模型可归纳事实，但不能自行决定来源页。最多三轮使 raw path 仍是受控预算，而非无界 research agent。

### 4. Verified write-back

从 raw log 恢复出的事实经过相关性、具体性、事实支撑和非重复检查后，可写回 Tier-1。manager 对旧记忆执行 `SKIP/UPDATE/ADD`；UPDATE 合并旧、新 provenance。

主 benchmark 为避免同一 run 内 memory distribution shift，关闭在线 write-back。consolidation 另用 epoch-wise replay：epoch 内冻结，epoch 间批量更新。它测到的是“同一批问题重复出现时能否摊薄成本”，不是新问题上的持续学习。

### 5. Router 训练

作者从 MemoryAgentBench LRU 的 2,000 个 summarization-style 样本分别运行 summary 和 raw/research path，由 LLM judge 得到 `c_S/c_R`：summary 正确标 S；summary 错、raw 对标 R；两条都错的样本丢弃。

先用 GPT-5 teacher 生成 reasoning/action 做 SFT，只保留 teacher 决策与 oracle label 一致的样本；再用 GRPO 奖励正确、惩罚错误、raw 成本、不必要升级与格式错误。GRPO 数据也移除 raw-fail 样本并过采样 hard escalation。router 因而主要学习“raw path 可修复时何时升级”，没有覆盖原文缺失、检索失败或生成器仍答错的完整部署分布。

## 实验设置

主实验使用 LoCoMo 和 LongMemEval。TierMem 的 summary-only、raw-only 与 routed 版本共享 GPT-4.1-mini generator、embedding 和 reranker，并比较 Mem0、LightMem、O-Mem、MemOS、MemR3、GAM 等系统。

正确性主指标是 LLM-judge accuracy，另报 token、latency、F1 和 `UOR`：summary 因缺证据失败、raw 成功的比例。router 还报告 R-rate、hard-case recall、routing F1 与 counterfactual repair/regression。论文未给出 judge 与人工标注的一致性校准，小幅准确率差异需谨慎解释。

## 主要结果与数值审计

### 1. 是成本—正确性折中，不是 raw-only 的等价替代

| Benchmark / path | Accuracy | QA input tokens | Router input | Latency |
|---|---:|---:|---:|---:|
| LoCoMo summary-only | 0.755 | 260.1 | — | 1.24 s |
| LoCoMo raw-only | 0.873 | 7,398.4 | — | 17.18 s |
| LoCoMo routed | 0.851 | 3,396.0 | 584.0 | 6.76 s |
| LongMemEval summary-only | 0.678 | 380 | — | 5.37 s |
| LongMemEval raw-only | 0.808 | 7,298 | — | 13.56 s |
| LongMemEval routed | 0.752 | 4,235 | 660 | 10.04 s |

LoCoMo routed 比 summary-only 高 9.6 点，但仍比 raw-only 低 2.2 点；LongMemEval 高 7.4 点、仍低 5.6 点。证据支持“以较低成本逼近 raw evidence”，不支持“路由后没有质量损失”。

摘要宣称 LoCoMo 输入 token 下降 54.1%，对应 `3396/7398`，但没有把 router 的 584 input tokens 加入分子。计入后总输入 3,980，降幅约 46.2%。LongMemEval 只算 QA input 降 42.0%，计入 router 后约降 32.9%。引用 headline 必须说明口径。

GAM 在 LoCoMo 为 0.869，高于 routed 的 0.851，但输入约 15,774；LongMemEval GAM 为 0.764，也高于 0.752，但输入约 37,850。TierMem 的主张应是 cost-aware frontier，而非最高准确率。

### 2. 学习 router 没在准确率上击败大模型 router

LoCoMo 上 zero-shot、SFT、SFT+GRPO accuracy 依次为 0.806、0.831、0.851，hard recall 为 44.7%、65.8%、71.7%，训练确有作用。

但 GPT-4.1-mini router 同样得到 0.851，hard recall 76.4%、routing F1 46.5%，高于 learned router 的 71.7%/40.6%，总时延也略低（6.40 vs 6.76 秒）。learned router 的 counterfactual regression 较低（7.7% vs 12.1%），repair rate 略低（64.9% vs 67.8%）。所以“小 router 匹配任务准确率”成立，“全面优于强 router”不成立。

### 3. Provenance link 有增益，但不是主要性能解释

移除 linked warm start 后，LoCoMo routed accuracy 为 0.836；加入链接为 0.851，升级子集从 77.5% 到 81.7%。但两个设置在 15.2%（234/1,540）问题上路由不同，并非只替换 raw retrieval。

固定共同路由后，共同升级样本中 Linked 对/No-linked 错 31 个，反向 25 个，净增只有 6；共同走 summary 也是净增 6。Linked 搜索深度 1.38 vs 1.36，升级 token 8,875 vs 8,705。证据支持“小而稳定的 warm-start 增益”，尚不足以证明 provenance 解释了大部分准确率或成本提升。它更重要的审计价值也没有用 citation precision/recall 单独测量。

### 4. Consolidation 主要证明重复 workload 的摊薄

同一固定 query set 重放三轮时，No-Recall accuracy 为 0.851 → 0.847 → 0.845，输入 3,958 → 3,035 → 2,419；Retrieve-and-Edit 为 0.851 → 0.852 → 0.850，输入 3,958 → 2,719 → 2,447。成本显著下降，准确率大体稳定但非严格不降。

E2/E3 重复 E1 问题，write-back 可以记住评测问题刚要求的细节。这证明 repeated-workload amortization，不能证明向未见问题迁移，也可能形成 query-specific evaluation cache。长期 ADD/UPDATE 后的存储增长、冲突和错误累积未充分报告。

### 5. False cache hit 仍是关键失败

summary-only 在 1,540 个 LoCoMo 问题上错 378 个，其中 missing information 156、over-generalization 109。router 升级 272 个；未升级的 106 个里只有 10 个后来被判断为可由摘要合理回答，留下 96 个 false cache hit。

附录另一组 diagnostic 又报告 431 个错误案例与 148 个 false cache hit，却未解释和 378 样本的关系；两者不能混用分母。机制结论一致：不必要升级主要浪费成本，错误直答则切断 raw 修复路径。

## 已验证结论、合理推断与待验证假设

### 论文直接支持

- raw evidence 在 LoCoMo/LongMemEval 可恢复大量被摘要遗漏的细节。
- sufficiency router 把 LoCoMo 从 0.755 提到 0.851，并显著低于 raw-only 查询成本，但仍损失 2.2 点准确率。
- SFT+GRPO 改善 zero-shot router；最终 accuracy 与 GPT-4.1-mini router 相同。
- Linked pages 相比无链接检索带来 1.5 点总体增益，但固定共同路由后的净优势较小。
- 相同问题的 epoch-wise replay 中，write-back 降低后续 token，accuracy 大体稳定。

### 基于证据的合理推断

- Agent memory 更适合定义成“便宜索引 + 可验证原始证据”，摘要应是 cache，而非唯一事实源。
- 路由错误成本非对称：false escalation 浪费预算，false cache hit 直接切断恢复路径。
- 来源指针的主要价值可能是审计、修复和冲突消解，而不只是当前 QA 增益。
- TierMem 可为 MEMOREPAIR/rollback 提供 source anchor，但 page pointer 不等于完整 influence graph。

### 仍待实验验证

- **新问题迁移：** 同一历史改问未见问题，禁止重复原问题，检查 write-back 是否保存通用事实。
- **provenance correctness：** 单独测 citation precision/recall、unsupported fact、缺指针和错链。
- **结构性缺链：** 删除高出度 link、跨页融合 link 与 UPDATE inherited link，测覆盖是否非线性崩溃。
- **真实 Agent 轨迹：** 加入 tool output、失败重试、代码 patch、环境状态和外部副作用。
- **不可恢复证据：** 保留 raw-fail、retrieval-fail、generator-fail，训练三态或带不确定度 router。
- **总成本：** 纳入 summary/embedding 建库、raw storage、router 训练/服务、Qdrant 和尾延迟。

## 局限性与可信度警告

- **headline 漏计 router input。** 54.1% 是 QA input 降幅，计入表中 router input 后约 46.2%。
- **正确性依赖 LLM judge。** UOR、router label 与 accuracy 都受模型判断影响，没有人工校准。
- **router 数据有可恢复性选择偏差。** both-fail/raw-fail 被剔除。
- **learned router 未超过 GPT router。** 它匹配 accuracy 并减少 regression，但 F1、hard recall、latency 不占优。
- **raw-only 仍是 accuracy ceiling。** 两个 benchmark 上 routed 均更低。
- **provenance ablation 不完全受控。** linked/no-linked 改变 15.2% 路由，+1.5 点不能全归于同一路由的 warm start。
- **write-back 测重复问题。** 没有证明未见问题、分布漂移或长期多次更新。
- **缺独立 provenance 指标。** 没有系统报告每个 fact 的 citation precision/recall 与 inheritance 错误。
- **代码公开但一键复算仍不完整。** 官方仓库已有主实现、三个 benchmark runner、router 数据准备/奖励/评测脚本与 Hugging Face 权重入口；但主表 result ledger、训练数据未随 git 提交，HF model card 为空，训练/服务也依赖外部配置。
- **版本需分开。** GitHub 标注 COLM 2026 和新题目，OpenReview 当前被浏览器验证阻挡；本笔记未把会议版全文当作已读证据。

## 与仓库已有论文和主题的主动关联

- [MEMOREPAIR](memorepair-barrier-first-cascade-repair.md) 沿 influence graph 撤回失效来源的派生物。TierMem 能找回摘要原页，但未覆盖 action、cache、skill 与外部副作用的完整依赖闭包。
- [Dependency-Guided Rollback Repair](dependency-guided-rollback-repair-agent-memory.md) 还需要 evidence 到 reasoning/action 的消费边，才能选择性重放。
- [MemSecBench](memsecbench-agent-memory-poisoning-repair.md) 提醒 source availability 不等于 source validity：若 raw log 被投毒，升级可能稳定重放攻击证据。
- [HarnessRisk](harnessrisk-lifecycle-agent-harness-safety.md) 覆盖 whole harness；TierMem 没有统一 credential、environment、tool side effect 与并发 cache。
- [Protocol Validity](do-agent-benchmarks-measure-capability-protocol-validity.md) 解释了为什么问题是否重复、raw 是否含答案、judge 和 token 口径会改变“长期记忆能力”结论。

## 与近期 AI / 评测论文的关系

- **ACON** — 首发 2025-10-01，v3 2026-06-01，https://arxiv.org/abs/2510.00615 。ACON 用 failure analysis 迭代压缩规则；TierMem 接受任何 query-agnostic 压缩都可能丢信息，保留 raw fallback。组合后应测更好压缩是否降低 escalation，而非让遗漏更难发现。
- **Goal-Directed Search / SUMER** — 首发 2025-11-20，https://arxiv.org/abs/2511.21726 。SUMER 直接搜索未压缩信息，代表 raw-first；TierMem 是 summary-first。应在同预算下比较 false cache hit、搜索遗漏和不可恢复问题，而不只比平均 QA。
- **Recursive Language Models** — 首发 2025-12-31，v3 2026-05-11，https://arxiv.org/abs/2512.24601 。RLM 把长 prompt 当外部环境，程序化检查和递归调用；TierMem 额外要求持久 raw store、summary provenance 与跨查询写回。
- **MEMOREPAIR** — 首发 2026-05-08，https://arxiv.org/abs/2605.07242 。它测试来源失效时的 invalidation recall；TierMem 测摘要不足时的 evidence recall。
- **Dependency-Guided Rollback Repair** — 首发 2026-08-11，https://arxiv.org/abs/2608.10502 。TierMem page provenance 可作 source anchor，但不能替代运行期 dependency capture。

本次六源 paper-search 对 arXiv、DBLP、OpenAlex、OpenReview、Semantic Scholar、Crossref 发起合并检索，但持续无输出，已停止；没有据此作“没有更多相关工作”的判断。相邻论文均以官方 arXiv 核对题目、日期和摘要级结论，除目标外未逐篇全文审读。官方 GitHub checkout 审计到 commit `7a9b71f6389a9f8caa83c47aff5191921c6a82ca`。

## 一个小而可证伪的后续实验

从真实 coding-agent 轨迹构造 120 个“同历史、后发问题”case，每条含 tool failure、约束更改和可核对原始输出。先在不知道问题时生成摘要，再随机问：摘要充分、需 linked page、需非 linked page 二跳搜索。禁止训练/写回阶段看测试问题。

比较 summary-only、raw-search-only、TierMem linked、TierMem shuffled-link，统一 generator、retriever 和总 token budget。报告 accuracy、false cache hit、escalation precision/recall、source citation precision/recall、unsupported fact、P95 latency 和**含 router 的总输入**。第二轮给相同历史换新问题，检查第一次 write-back 是否迁移。

竞争解释是收益主要来自更多 raw retrieval/token，source pointer 贡献很小。如果固定路由和候选页预算后，linked 不显著优于 shuffled-link，就停止宣称 provenance 是性能核心，只保留“分层存储 + sufficiency routing”。若 write-back 只改善重复问题，也应称为 query cache，而非长期记忆学习。

## 一句话总结

TierMem 最有价值的思想是把摘要降级为可丢失索引，把 raw log 保留为可回溯证据，并按“证据是否充分”分配查询预算；它在 LoCoMo 上以总输入约下降 46.2% 的代价把 raw-only 0.873 折中到 0.851。现有证据也划出边界：false cache hit 仍危险，source link 的受控净增益不大，write-back 只在重复问题上验证；下一步应直接评测 provenance correctness 和未见问题迁移。
