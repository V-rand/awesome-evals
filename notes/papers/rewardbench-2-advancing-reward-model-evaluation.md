# Notes — “RewardBench 2: Advancing Reward Model Evaluation”

**作者：** Saumya Malik, Valentina Pyatkin, Sander Land, Jacob Morrison, Noah A. Smith, Hannaneh Hajishirzi, Nathan Lambert · **首次提交：** 2025-06-02 · **发表：** ICLR 2026 · **版本：** arXiv v2 / conference paper · **URL：** https://arxiv.org/abs/2506.01937 · **会议页：** https://proceedings.iclr.cc/paper_files/paper/2026/hash/ea4fe0a56d02c93401902b5b4c6b12da-Abstract-Conference.html · **代码：** https://github.com/allenai/reward-bench · **数据：** https://huggingface.co/datasets/allenai/reward-bench-2 · **类型：** benchmark paper · **Found:** true

## 论文问题与科学动机

Reward model（RM）把“这个回答有多好”压缩成奖励信号，既可用于 best-of-N（BoN）重排，也可作为 PPO 等 RLHF 算法的优化目标。问题在于，**一个 RM 在静态偏好题上答得准，是否真的意味着它能在下游生成和训练过程中提供有用的优化方向？** 早期 RewardBench、RM-Bench 等通常比较一对 chosen/rejected 回答，随机基线高达 50%；许多题目又复用了常见下游 benchmark 的 prompt，使静态分数可能饱和、受污染，或只测到旧题型记忆。

RewardBench 2 因此试图同时解决两个测量问题：一是构造一个对强 RM 仍有区分度、且与常见下游评测 prompt 尽量独立的多技能 benchmark；二是直接检验 benchmark 分数与 RM 在两种真实用途——BoN 选择和 PPO 训练——之间的关系。论文最重要的科学问题不是“谁排第一”，而是：**什么样的离线 RM 分数可以预测哪一种下游用途；又有哪些训练上下文无法被通用排行榜捕获？**

## 核心方法和具体机制

### 1. 从二选一改为四选一，主动拉开强模型与随机基线

除 Ties 外，每个样本包含 1 个正确回答和 3 个错误回答，RM 必须把正确回答排在最高。随机准确率从常见二选一的 50% 降到 25%，增加了强模型之间的分辨空间，也更接近 BoN 中“从多个候选中选最好”的使用方式。六个领域先分别计分，最终总分是六个领域的非加权平均，避免样本量最大的领域主导总分。

Ties 不是普通准确率：它同时要求所有正确回答高于错误回答，并比较“正确与错误之间的 reward margin”是否大于“不同正确回答之间的内部跨度”。直观上，RM 不应因为多个同样有效的答案表述不同就给出过强偏好；用于 RLHF 时，推动“从错到对”的梯度应该大于把一个正确答案挤压成另一个正确答案的梯度。

### 2. 六个子集分别用不同的生成与验证管线

Benchmark 共 **1,865 个 prompt**，约 70% 来自经用户同意收集、此前未发布的 WildChat 人类查询；作者用 Tulu 3 去污染工具与 20 个常用下游评测比对并报告无重叠。补全回答来自 20 个模型或人工编写，六个领域的标签机制并不相同：

1. **Factuality（475）：** 自然回答与被系统提示诱导出细微事实错误的回答混合。GPT-4o 初判，Claude 3.7 Sonnet 复核；两者不一致的约 30% 候选被丢弃。
2. **Precise Instruction Following（160）：** 给真实查询追加可程序验证的精确约束，例如禁用某字母；每题的四个回答来自同一生成模型，以尽量控制回答总体质量，只改变约束满足情况。
3. **Math（183）：** 对真实开放式数学/科学问题从多个模型采样，用多数投票形成候选答案，再由 Llama 3.1 8B Instruct 辅助判定；由于单位、舍入和答案抽取脆弱，最终每题都人工复核。
4. **Safety（450）：** 基于 CoCoNot 的细粒度合规/拒答 taxonomy 和子类 rubric，用 GPT-4o 生成/判断后全部人工复核；删除主观事项、模态限制、欠明确请求和拟人化请求等争议类别。
5. **Focus（495）：** 参考 LLMBar 改写原 query 的细节，诱导回答偏题、答非所问或响应错误；一个自然回答与三个轻微错位回答组成四选一。
6. **Ties（102）：** 人工构造存在多个等价正确答案的问题，并配套正确与错误回答，检查 reward 的相对排序和 margin。

这种“按技能选择 verifier”的设计比统一用一个 LLM judge 更具体，但也意味着六个分数的标签噪声来源不同：程序规则、双模型共识、多数投票、人工判断和合成扰动并不是同一种 ground truth。

### 3. 不只评已有模型，还训练受控 RM 来分离训练因素

作者评测了 100 多个已有 RM/生成式 judge，并用 Open Instruct 训练约 **120 个 Bradley–Terry RM**。受控变量包括：

- 基座与 post-training 阶段：Llama 3.1 Base/Instruct、Tulu 3 SFT/DPO/RL、Qwen 2.5 Base/Instruct，以及部分 70B 模型；
- 训练数据：270K Tulu preference mix、80K Skywork preference mix 或两者合并；
- 学习率与训练轮数：1、2、3 epoch 以及三个学习率。

这使论文能区分“benchmark 上哪个 checkpoint 更强”和“什么训练配方带来哪些能力”。结果显示，post-training 阶段已经获得的能力会传入 RM；例如 Qwen Instruct 基座在 Math 上明显更强。两套偏好数据的优势也不同：Skywork 更利于 Focus/Safety，Tulu 更利于 Factuality，合并后平均最好。传统上担心过拟合而只训一轮，但 18 个最优配置中有 8 个来自两轮训练，且多轮 RM 并未在作者的下游实验中系统性变差。

### 4. 用 BoN 与 PPO 分别验证“静态分数是否可用”

**BoN 实验：** 作者用 Tulu 3 8B SFT 为 GSM8K、MATH、IFEval、AlpacaEval 2、BBH、PopQA 和 HumanEval+ 生成每题 16 个候选，再让 **113 个 RM** 各自选最高分回答。RewardBench 2 平均分与七项任务的 BoN 平均分 Pearson 相关为 **0.87**；Factuality 与总体下游最相关，Math 子集对数学与代码任务也有强域内信号。

**PPO 实验：** 以 Tulu 3 8B SFT 为统一初始 policy，用 Tulu preference mix prompt、学习率 3×10⁻⁷、线性衰减和 KL 系数 β=0.05，对 **17 个 RM** 分别跑 PPO；结果在 Tulu 3 evaluation suite 的九项任务上汇总，并在多组超参数中取最佳中间 checkpoint。

PPO 呈现与 BoN 不同的结构：低质量 RM 的确对应较差 policy，但当同源、同分布 RM 的 RewardBench 2 分数进入约 49.8–68.5 区间后，PPO 下游分数很快饱和在约 59.5–60.7，静态榜单难以继续排序。更关键的是，一些 RewardBench 2 高分 RM 若使用与 policy 不同的基座谱系，或只在不同 prompt 分布上训练，PPO 结果反而明显下降。例如 off-policy RM 的 RB2 分数可达 72.9，但 PPO 只有 54.5；同源、同分布的 RB2 68.0 模型可达到 60.4。

## 实验设置与主要结果

- 公开榜单中，Skywork-Reward-V2-Llama-3.1-8B 总分 **84.1**，为论文 Table 3 最高；最难的共同瓶颈是 Precise IF、Math 和 Factuality，而 Safety/Focus 已接近饱和。
- 强模型在 RewardBench 2 上平均比原 RewardBench 低约 **20 个百分点**。对作者受控训练的 17 个 RM，RewardBench 2 与 RewardBench、RM-Bench、PPE-Correctness 高度相关（Pearson 0.95–0.97），说明它更难，但没有完全换一个能力轴。
- 对 113 个 RM，RewardBench 2 与 BoN 七任务平均分相关 **0.87**；各下游任务彼此也高度相关，因此不能把相关性全部解释成 benchmark 的独特预测力。
- 对 PPO，benchmark 只清晰区分很差 RM；中高分、同源同分布 RM 的结果饱和。跨基座/跨数据分布时，高静态分数不再保证好 policy。
- RM 存在轻微但统计显著的“基座自偏好”：Tulu、Llama、Qwen 基座训练出的 RM 会相对更高地排序同一谱系生成的回答，即使再按训练数据来源分组，趋势仍存在。这支持 benchmark 使用多模型回答池，但也提示任何固定 completion pool 都可能偏向某些 RM 家族。
- 论文报告评测约 160 个模型消耗约 30 GPU 小时；训练约 120 个 8B RM、5 个 70B RM和 17 次 PPO，总计约 **55,000 GPU 小时**。

## 已验证结论、合理推断与待验证假设

### 论文已验证

- 在论文给定的数据与模型池上，四选一、多领域 RewardBench 2 明显比原 RewardBench 更难，并对强 RM 保留更大分辨空间。
- 在 Tulu 3 8B SFT 生成的固定 16 候选池和七个指定任务上，RewardBench 2 分数与 BoN 平均结果高度相关（r=0.87）。
- 在作者给定的 Tulu 3 8B PPO 配方中，静态分数对低质量 RM 有筛除价值，但对中高质量、同源 RM 的下游排序迅速饱和；基座谱系或 prompt 分布失配会让高分 RM 的 PPO 效果下降。
- 在作者训练的 RM 集合上，基座 post-training 能力、偏好数据组成和训练轮数都会改变领域分数；“RM 必须只训练一轮”不是该实验中的普遍规律。
- 所测试 RM 对自身基座谱系生成的文本存在轻微自偏好，完成池多样性因此是公平评测的一部分。

### 基于证据的合理推断

- RM benchmark 更适合做“最低质量门槛 + 能力诊断”，而不是把总分当作所有下游算法共享的全序排名。BoN 是直接重排固定候选，静态排序能力自然更可迁移；PPO 会主动改变 policy 分布，因此依赖 RM 与 policy 的联合动力学。
- 选择 PPO 奖励模型时，应复制合适的训练配方并在目标 policy/目标 prompt 分布上重训或复核，而不是直接下载排行榜第一的 checkpoint。
- 每个领域都应同时报告平均准确率、reward margin/校准和生成模型覆盖；只增加难题而不控制候选来源，可能把“偏爱某种写作谱系”误记为能力。
- “与下游相关”必须声明生成器、候选数、任务组合和优化算法。r=0.87 不能外推为对任意 BoN 生成器、任意 N 或任意 RL 算法同样有效。

### 待实验验证

- 换成不同规模、不同家族或更强生成器后，BoN 相关性是否保持。论文附录已观察到更强生成器可能压缩候选质量差异，使相关性下降。
- PPO 中真正的因果因素是基座表示、tokenizer、输出风格、训练数据覆盖还是 reward scale；论文把它们统称为 on/off-policy 或 in/out-of-distribution，但未完全解耦。
- 六个子集的标签准确率与人类复标一致性，尤其是 Factuality 的双 LLM 共识和 Safety 的规范选择；“两模型同意”仍可能是相关错误。
- RewardBench 2 是否能预测 GRPO、online DPO、RLAIF、过程奖励模型或 agent trajectory reward；论文只系统验证了 BoN 与一种 PPO 配方。
- 榜单长期被公开使用后是否会出现训练集泄漏、针对性优化和 metric capture；论文使用未发布 prompt 降低发布前污染，但发布后独立性会随时间衰减。

## 局限性

- 数据规模只有 1,865 题，六域又被非加权平均；Precise IF 仅 160 题、Ties 仅 102 题，少量标注变化可能显著影响领域排名。
- “客观准确率”并不等于没有价值判断：Safety 的类别删选、合规 rubric、Ties 中哪些答案等价，都包含规范选择；只是比自由偏好更显式。
- Factuality 的拒绝样本部分由“故意犯细微错误”的系统提示生成，可能留下可被 RM 学会的生成痕迹。作者发现自然错误更难，也侧面说明合成错误并不完全代表部署中的幻觉。
- 生成式 judge 取 rankings 与 ratings 两种提示中较高的成绩，相当于为这一模型类型做小规模提示选择；不同 judge 的提示敏感性可能影响横向公平。
- PPO 只围绕 Tulu 3 8B、相同 tokenizer 和固定 preference mix 展开，且汇报多组超参数中最佳中间 checkpoint；这对实际训练预算和泛化的估计偏乐观，也不足以证明一般 RLHF 规律。
- 相关分析使用许多共享任务和同一个生成器，样本并非完全独立；高 Pearson 相关不能证明优化 benchmark 分数会因果提升下游。
- 论文没有提供统一的人类 inter-annotator agreement，也没有给每个子集的置信区间或 leaderboard 排名稳定性。

## 与仓库已有论文和主题的主动关联

- **RewardBench：** 第一版建立 pairwise chosen/rejected 的开放 RM 评测；RewardBench 2 把它扩展为 best-of-4、未见过的人类 prompt 和新领域，并用下游 BoN/PPO 明确测试外部效度。两者高相关说明 v2 是难度和覆盖升级，不是完全独立的构念。
- **Training Verifiers to Solve Math Word Problems：** verifier 在 best-of-N 中把候选选择变成可扩展推理；RewardBench 2 给这一范式补上跨领域 meta-evaluation，并显示 Math/Factuality 的静态识别能力确实能预测相应候选选择。
- **Who Validates the Validators? / EvalGen：** EvalGen 让应用开发者用错误分析、coverage 与 false-failure rate 校准 criterion；RewardBench 2 则为通用 RM 提供固定多技能试卷。前者强调本地任务/人的 specification alignment，后者强调跨模型可比性；真正部署应先过通用门槛，再在本地数据上做 EvalGen 式验证。
- **LLM Evaluators Recognize and Favor Their Own Generations：** 该工作在生成式 judge 中建立自偏好证据；RewardBench 2 把现象扩展到判别式 RM 的基座谱系，并据此要求 completion pool 多样化。两者共同说明“评估器与被评模型的关系”本身是实验变量。
- **AI Agents That Matter：** 该文要求把模型、scaffold、成本与 holdout 拆开比较；RewardBench 2 在 RM 场景给出相同警告：reward checkpoint、policy lineage、训练 prompt 分布和下游算法必须一起声明，单一 leaderboard 分数不是系统性能。
- **BenchJack：** RewardBench 2 主要验证 reward label 的语义质量和下游效度；BenchJack 审计 evaluator 实现是否可被绕过。前者回答“评分目标是否有用”，后者回答“评分链是否可信”，二者是 verifier 可靠性的不同层。

## 与近期 AI / 评测论文的关系

- **Rethinking Reward Model Evaluation Through the Lens of Reward Overoptimization** — 2025-05-19，https://arxiv.org/abs/2505.12763 。它主张 benchmark 应使用多个 chosen/rejected 比较、控制正确性之外的差异并扩大生成模型来源；与 RewardBench 2 的 best-of-4 和多模型池高度一致，但进一步要求从 policy 优化后的 reward overoptimization 观察 RM，而不只看静态准确率。
- **VerifyBench** — 2025-05-21，https://arxiv.org/abs/2505.15801 。它专门评估“给定参考答案的 reasoning verifier”，覆盖 RewardBench 2 没有细分的 reference-based reward system。前者更聚焦可验证推理，后者更广泛覆盖事实、指令、安全、焦点和等价答案。
- **Inference-Time Scaling for Generalist Reward Modeling / DeepSeek-GRM** — 2025-04-03，https://arxiv.org/abs/2504.02495 。它让生成式 RM 通过自生成原则、critique 和并行采样随推理算力扩展；RewardBench 2 的静态 ratings/rankings 测试可作为结果指标，但没有测“增加 judge 计算量后的性能-成本曲线”。
- **One Token to Fool LLM-as-a-Judge** — 2025-07-11，https://arxiv.org/abs/2507.08794 。它证明参考型生成式 reward model 也会被冒号、句点或通用推理开头等 master-key token 诱发假阳性。RewardBench 2 测自然与合成错误的识别准确率，却没有对抗性地测试 reward hacking；高 RB2 分数不等于 verifier 对优化攻击鲁棒。
- **Agent-RewardBench** — 2025-06-26，https://arxiv.org/abs/2506.21252 。它把 reward evaluation 扩展到多模态 agent 的 perception、planning、safety 和 step-level reward。RewardBench 2 的单位是单轮文本回答，不能直接衡量长轨迹中的局部信用分配与状态感知。
- **An Empirical Investigation of Practical LLM-as-a-Judge Improvement Techniques on RewardBench 2** — 2026-04-15，https://arxiv.org/abs/2604.13717 。这篇直接后续工作把 RB2 当作 judge 开发集，报告 task-specific criteria 与 ensemble 能把 GPT-5.4 judge 从 71.7% 提到 83.6%。它证明 RB2 可驱动无训练的 judge 改进，也同时提醒公开 benchmark 可能逐渐从独立测试集变成调参目标。

这些工作把 RM 评测拆成至少四层：**静态多技能排序是否正确（RewardBench 2）→ 对参考推理的验证是否可靠（VerifyBench）→ 优化 policy 后是否过拟合奖励（reward-overoptimization）→ verifier 是否抵抗主动攻击与轨迹分布变化（One Token / Agent-RewardBench）**。一个排行榜无法替代这四层证据。

## 一句话判断

RewardBench 2 的价值不只是“让分数低 20 点”，而是用未见人类 prompt、四选一、多技能标签和真实下游实验说明：静态 RM 评测对 BoN 有强预测力、对筛掉劣质 PPO 奖励有用，但无法脱离 policy 谱系与训练分布选择最优 RLHF checkpoint；它把 leaderboard 从终点改成了进入目标训练系统前的一道诊断门槛。

## Themes

7 verifiers & reward design · 8 LLM-as-judge · reward models · RLHF · best-of-N · benchmark validity · distribution shift · evaluator self-preference
