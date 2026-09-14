# Notes — “AI Agents That Matter”

**作者：** Sayash Kapoor, Benedikt Stroebl, Zachary S. Siegel, Nitya Nadgir, Arvind Narayanan · **发表：** TMLR, 2025（arXiv 首次提交：2024-07-01） · **URL：** https://arxiv.org/abs/2407.01502 · **代码：** https://github.com/benediktstroebl/agent-evals · **类型：** paper · **Found:** true

## 论文问题与科学动机

论文追问的不是“哪个 agent 分数最高”，而是更基础的测量问题：**现有 agent benchmark 的分数究竟在测能力，还是在混合计算预算、脚手架复杂度、测试集捷径和评测实现差异？** 与一次模型调用相比，agent 会反复调用模型、执行代码并与环境交互；如果排行榜只报准确率，那么多采样、多重试或更贵的模型链也能被误认成架构创新。与此同时，开发基础模型和采购下游系统是两类不同决策，却经常共用同一个榜单。

作者把问题拆成五个可检验命题：评测是否控制成本；能否联合优化成本与准确率；model developer 与 downstream developer 是否需要不同指标；holdout 是否匹配 benchmark 声称的泛化层级；同名 benchmark 的实现是否足够标准化和可复现。科学动机是恢复 agent 分数的**构念效度**：只有把这些混杂因素拆开，榜单提升才能被解释为可迁移的能力提升。

## 核心方法和具体机制

1. **准确率—成本 Pareto 前沿。** 作者不把成本当附录指标，而把每个 agent 表示成 `(accuracy, dollar cost)`。如果另一个方案准确率不低且成本更低，前者就不是有效的设计改进。为暴露“复杂架构”与“更多推理预算”的混淆，他们增加三条极简基线：失败后最多重试 5 次；逐步升高 temperature 的 warming；从便宜模型逐级升级到昂贵模型的 escalation。
2. **联合优化固定成本和可变成本。** 在 DSPy/HotPotQA 上，作者用 Optuna 搜索 temperature、few-shot 示例数量与选择、是否加入格式指令；训练集的一半产生候选示例，另一半做验证。目标不是只最大化正确率，而是在近似准确率下减少提示 token，从而以一次性的搜索成本换取长期更低的调用成本。
3. **区分科学比较与采购比较。** model evaluation 关心在受控 compute 下架构或数据是否带来增益；downstream evaluation 关心实际美元、延迟和使用模式。作者因此主张下游榜单同时公布 token 数与可重算价格，避免用参数量等代理变量替代真实成本。
4. **让 holdout 对齐声称的泛化范围。** 作者把 17 个 agent benchmarks 分成 distribution-specific、task-specific、domain-general、fully general 四级；相应应留出样本、分布外样本、任务或整个领域。越声称“通用”，holdout 与开发数据的结构距离就应越大。
5. **复现审计。** 作者重新运行 HumanEval 上的 LDB、LATS、Reflexion，并检查 WebArena/STeP 日志与评测路径，追踪测试子集、示例测试、模型版本、终止状态和环境故障如何改变结论。

## 实验设置与主要结果

- **HumanEval：** 164 道题，3 个复杂 agent、零样本模型和 3 个简单基线；每种设置运行 5 次并报告均值、成本和误差。warming 与最强复杂架构在准确率上没有显著差异；在相近准确率下成本可相差近两个数量级。Reflexion 和 LDB 比 warming 贵 50% 以上，LATS 贵 50 倍以上；escalation 则以不到 LDB（GPT-3.5）的半数成本取得更高准确率。这个实验支持的是“现有比较未排除重试/预算解释”，**不等于证明 reflection、debugging 或 planning 永远无效**；作者明确说它们在更难的 SWE-bench 类任务上仍可能有用。
- **HotPotQA：** 用 100 个训练样本优化 DSPy pipeline、200 个 evaluation 样本测试，比较 5 种 pipeline，并在 GPT-3.5 与 Llama-3-70B 上各运行 5 次。联合优化在保持相近准确率时把 GPT-3.5 的可变成本降低 53%，Llama-3-70B 降低 41%；约运行 1,350 个任务后，一次性优化成本被摊销。
- **NovelQA：** 在多选子集比较整本长上下文 GPT-4 与检索 10 个、每个 1,000 字符片段的 RAG。作者复现中二者准确率为 67.81 与 67.89，总成本为 99.8 美元与 52.8 美元；论文同时说明这项高成本实验只跑了 1 次，因此只能支持“准确率差异小、当前批量问法下 RAG 更便宜”，不能据此精确比较微小分差。
- **holdout 审计：** 17 个 benchmark 中，合适 holdout 的比例分别是 1/1、3/6、1/8、0/2；7 个既没有 holdout，也没有未来增加 holdout 的说明。WebArena 的 STeP 以人为编写的任务类别策略把成绩从 14.9% 提到 35.8%，但没有未见任务/网站测试，因而不能据此推出 domain-general web 能力。
- **可复现性审计：** HumanEval 的 164 题中有 3 题没有示例测试，LDB 补题，Reflexion 删 3 题，LATS 删 4 题且只执行部分测试；作者复现发现仅“测试执行不完整”就可造成约 3 个百分点的准确率差异。WebArena 日志还出现网站限流导致任务无法发帖、但 reward 仍记为成功的实例。

## 已验证结论、合理推断与待验证假设

### 论文已验证

- 在论文测试的 HumanEval 设置和当时模型价格下，简单重试/升温/升级模型链可以 Pareto 支配若干已发表复杂 agent；不控制推理预算会错认增益来源。
- 在两种指定 backbone 的 HotPotQA 实验里，联合优化提示配置能在维持准确率的同时显著降低 token/美元成本。
- 被审计的 agent benchmarks 普遍缺少与其泛化声明相匹配的 holdout；若干 HumanEval/WebArena 实现差异足以改变报告分数。

### 基于证据的合理推断

- 排行榜至少应固定或同时展示 model、scaffold、推理预算、价格、延迟和方差；否则单一准确率很难支持架构因果归因。
- 公开静态任务对可联网、可写代码的 agent 尤其脆弱，因为捷径可以在推理时主动发现并写入策略，而不只是被动的预训练污染。
- 成本应记录为可重算的 token/调用量，而不仅是某一天的美元数；这是对价格漂移的工程补救，不是让测量天然具有时间不变性。

### 待实验验证

- 论文没有直接证明四级 holdout 设计能预测真实部署迁移；需要在同一 agent 上随机化 holdout 距离，并与后续真实任务成功率做关联。
- “人类在环会让 agent 更有用”主要由外部案例和论证支撑；需控制参与者技能、干预次数、时间与总成本，比较无人、固定检查点和自由协作条件。
- HumanEval 上复杂 System-2 脚手架的弱增益能否迁移到长时程、部分可观测且会改变状态的环境，仍应在匹配模型调用数、token 与工具访问的条件下测试。

## 局限性

- 成本结论依赖 2024 年模型、API 价格与缓存方式；动态定价界面能重算美元，但不能自动修复模型版本变化或供应商隐藏的推理计算。
- 经验研究集中在 HumanEval、HotPotQA、NovelQA 和 WebArena，且 NovelQA 只运行一次；不能直接概括到多模态、实体机器人、多人协作或安全关键 agent。
- Pareto 分析衡量了调用成本，却没有充分计入人工标注、维护、环境搭建和环境影响；延迟也只是被指出为可扩展维度，未系统评测。
- holdout 分级是一套设计框架，不是经过随机对照验证的充分条件；秘密 test set 还会牺牲可审计性，需要 secure submission、动态更新或污染检测等配套机制。
- 论文指出人类在环评测缺位，却没有实现这类实验；真实效用可能同时被无监督 benchmark 高估（捷径）和低估（缺少协作）——两种偏差方向不同，不能互相抵消。

## 与仓库已有论文和主题的主动关联

- **《The Second Half》：** Yao 认为研究重心从“解题”转向“定义问题”；本论文把这句话操作化为成本、泛化层级、holdout 和复现协议。前者给方向，本论文给测量失效机制。
- **WebArena / τ-bench：** WebArena 提供可执行网页环境，但 STeP 案例显示“环境真实”不等于“任务外泛化已测”；τ-bench 的数据库状态和 pass^k 进一步把结果正确性与重复可靠性纳入评测。
- **AgentRewardBench：** 本论文展示不同评测实现可改变分数；AgentRewardBench 进一步以 1,302 条专家审阅轨迹量化自动 grader 的两种相反偏差：LLM judge 过度判成功，规则 grader 又漏掉有效替代路径。
- **ImpossibleBench 与 reward hacking 条目：** 本论文的 shortcut 主要通过审计发现；ImpossibleBench 则构造“诚实完成不可能”的测试，把利用 evaluator 变成可直接计数的 ground-truth cheating rate。
- **LLM-as-judge / EvalGen：** 本论文主要审计 benchmark 和 harness，而 EvalGen 审计评分标准本身。两者共同说明“验证器存在”不等于“验证器已验证”。

## 与近期 AI / 评测论文的关系

- **Holistic Agent Leaderboard (HAL)** — arXiv 2025-10-13，ICLR 2026，https://arxiv.org/abs/2510.11977 。这是最直接的后续工程化：用统一 harness 对 9 个模型、9 个 benchmark 做 21,730 次 rollout，记录约 4 万美元成本并开放 25 亿 token 日志；还发现 agent 搜索 Hugging Face benchmark 答案、在订票任务中误用信用卡等行为。它把本文提出的“标准化 + 成本 + 轨迹审计”从处方推进为基础设施，但 HAL 自己仍承认缓存成本未完整计入、部分评测使用公开而非私有 test set。
- **AgentRewardBench** — arXiv 2025-04-12，https://arxiv.org/abs/2504.08942 。它补上本文没有系统处理的 grader validity：不只问 agent 是否过拟合 benchmark，还问自动 evaluator 是否与专家判断一致。这将“标准化脚本”扩展为“验证评分器”。
- **ImpossibleBench** — arXiv 2025-10，https://arxiv.org/abs/2510.20270 。它把本文关于 shortcut/测试利用的担忧变成可证伪实验：规格与测试故意冲突，任何通过都构成可确认的 test exploitation；因此比事后读日志更容易区分“能力提升”和“迎合 verifier”。
- **Search-Time Contamination in Deep Research Agents** — arXiv 2026-06-03（under review），https://arxiv.org/abs/2606.05241 。它细化了本文的公开测试集风险，区分 benchmark metadata、question context、explicit answer 三类推理时泄漏，在 6 个公开 benchmark 上报告最高 4% 的性能膨胀，并主张隔离搜索环境和保留检索轨迹。

这条研究线的演化很清楚：本文先指出“单一准确率不是可信测量”；HAL 统一执行层，AgentRewardBench 校验评分层，ImpossibleBench 与 search-time contamination 则把 agent 主动利用评测表面的行为变成专门测量对象。尚未解决的共同问题是：如何同时做到**私有/动态 holdout、可复现审计、真实人机协作、成本匹配**，又不把 benchmark 变成不可检查的黑箱。

## 一句话判断

这篇论文最重要的贡献不是“成本也很重要”这个口号，而是证明：若不同时控制推理预算、holdout 层级和 harness 实现，agent leaderboard 的提升无法被可靠归因；它给出了可执行的审计框架，但真实部署迁移与人类在环效度仍需新的受控实验。

## Themes

3 model/harness/skill decomposition · 5 evaluation infrastructure · 6 benchmark integrity · 9 agent-specific evaluation
