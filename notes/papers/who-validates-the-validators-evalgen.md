# Notes — “Who Validates the Validators?” (EvalGen)

**作者：** Shreya Shankar, J. D. Zamfirescu-Pereira, Björn Hartmann, Aditya G. Parameswaran, Ian Arawjo · **发表：** UIST 2024（arXiv 首次提交：2024-04-18） · **URL：** https://arxiv.org/abs/2404.12272 · **会议版：** https://doi.org/10.1145/3654777.3676450 · **类型：** paper · **Found:** true

## 论文问题与科学动机

这篇论文追问的是自动评测中常被跳过的一层：**当 LLM 或 LLM 生成的代码充当 evaluator 时，谁来验证 evaluator 是否真的表达了人的标准？** 传统字符串规则难以检查“简洁”“有帮助”等模糊属性，人工逐条评分又昂贵；LLM judge 看似填补了空缺，却继承了提示敏感、偏置、幻觉和不稳定性。仅让模型自动生成 rubric 或 grader prompt，并没有消除评测成本，只是把“判断输出好坏”转移成“判断评测器是否可信”。

作者进一步指出，评测标准不是总能在看见输出前一次性写定。人需要标准才能给输出打分，但看见具体成功和失败样例后，才会发现遗漏的标准、修改阈值，甚至重写旧标签。论文把这种相互依赖称为 **criteria drift**。因此科学问题不只是“怎样生成更好的 rubric”，而是：在有限人工标注预算下，怎样让人、自然语言标准、可执行 assertion 和实际模型输出共同迭代，并让这种对齐过程可检查？

## 核心方法和具体机制

EvalGen 嵌入开源 prompt-engineering 工具 ChainForge，把评测器构建拆成一个 mixed-initiative 循环：

1. **标准提出。** GPT-4 根据待测 prompt 推荐若干二元 criteria；用户可以删改、自己填写，或先给若干输出打分再决定标准。每条 criterion 由用户指定为代码型或 LLM 型。
2. **候选实现生成。** 对每条 criterion，GPT-4 一次生成多个候选 assertion。确定性格式、计数和关键词规则可生成 Python；语义性条件则生成输出 true/false 的 grader prompt。候选以流式 JSON 解析，并异步跑在已有模型输出上。
3. **有限人工评分。** 用户对系统挑出的输出给“good/bad”二元标签。候选 assertion 的 pass rate 被称为 selectivity；若一个高-selectivity assertion 仍然判某输出失败，该输出更值得优先检查。系统以失败 assertion 的 selectivity 加权形成“可能是坏输出”的置信分数。
4. **非随机采样。** 系统在高分与低分输出间交替取样，希望同时覆盖疑似坏例和表面正常例。附录还比较 random、highest、lowest、alternating；每种策略对两个数据集各做 10 次、每次抽 16 条。三种非随机策略在该离线设置下都比随机抽样稳定，但 alternating 并非始终最优。
5. **按人工标签选 assertion。** 对 assertion 集合，coverage 是人工标为 bad 的输出中被至少一条 assertion 抓住的比例；false failure rate（FFR）是人工标为 good 的输出中被错误拦截的比例。alignment 是 `coverage` 与 `1 - FFR` 的调和平均。每条 criterion 选 alignment 最高且满足 FFR 阈值的实现，最后用 report card、混淆矩阵和全量输出表让用户检查结果。

与 SPADE 的关键差别不是换了一个更强 LLM，而是把人工反馈放进两个位置：人先确认“要测什么”，再用少量标签决定“哪种实现最像我的判断”。SPADE 则从 prompt 修改历史自动生成 criteria/assertions，并用整数规划挑满足 coverage/FFR 约束的最小集合。

## 实验设置与主要结果

### 离线算法核验

- **任务：** 医疗流水线从 84 段医患通话中抽取症状、药物等信息并避免 PII；商品流水线为 100 个 Amazon 商品及评论生成 SEO 描述。输出由 GPT-3.5-Turbo 生成，两位论文作者为全部输出打标签；两任务中 good 输出分别占 68% 和 51%。
- **人工预算模拟：** EvalGen 每个任务只使用 16 条已评分输出选 assertion，而 SPADE 使用全部标签。医疗任务 FFR 阈值为 10%；商品任务在 10% 下无可行 assertion 集，放宽到 40%。
- **结果：** 医疗任务中 EvalGen 用 3 条 assertions、SPADE 用 5 条，coverage 都为 0.33、FFR 都为 0.10、alignment 都为 48.29%。商品任务中 EvalGen 用 4 条、SPADE 用 9 条；FFR 都约为 0.39，但 coverage 从 0.49 提到 0.73，alignment 从 54.35% 提到 66.46%。SPADE 的多余或过窄规则包括“neutral tone”和仅匹配 “never order”“disappointed”等具体短语；EvalGen 在商品正面描述标准上选择了更宽的 LLM assertion。
- **解释边界：** 结果支持“人工确认标准 + 16 个标签能在两个固定流水线上选出更小且不差的 assertion 集”。它不是统计充分的通用算法胜出证明：只有两个任务，标签由作者给出，候选实现仍由 GPT-4 生成，而且同一批全量输出既参与候选执行又用于最终报告指标。

### 9 人定性用户研究

- **参与者与流程：** 通过 Twitter 招募前 9 名有编码和生产 LLM 流水线经验的从业者，包括软件工程师、ML 科学家、创业公司管理者和独立顾问。参与者在 Zoom 中远程操作，以 100 条 tweet 的 NER 流水线为共同任务，最多自由探索 40 分钟，再做约 10 分钟访谈；每场共 45–75 分钟，并通过 IRB。
- **使用行为：** 6 人直接让系统生成 criteria，1 人先手写一条再自动生成，2 人先评分；后两人先评 5–10 条、其中给出 2–4 个 bad 标签。参与者通常删掉一些建议并新增 1–2 条，3 人实际开启第二轮迭代，超过一半表示若有更多时间愿继续。
- **可用性证据：** 8/9 认为自动生成 criteria 缓解了“从空白开始”的困难；8/9 认为等待候选生成时顺便评分是合理利用时间。除 1 人外都理解 coverage 和 FFR，全员对逐输出、逐 assertion 的表格结果感兴趣。
- **对齐并不稳定：** 对“assertions 与我的评分一致”的 1–7 分，九人的分数为 6、5、3、4、5、3、1、2、5，均值约 3.78，不能概括为高满意。参与者更愿用代码检查格式、计数、特定短语，用 LLM 检查模糊语义和外部知识；代码规则更容易直接审计，LLM assertion 更难信任和维护。
- **criteria drift：** 5 名参与者在看到新失败类型后想新增标准；也有 5 人重新解释已有标准。例如“实体必须是 proper noun”从“全部满足”改成“大多数满足”；“不要抽取 hashtag”对不同人分别意味着“不应抽取 Nike”或“可以抽取 Nike 但去掉 #”。这说明同一句自然语言 criterion 可能掩盖不同的决策边界。

## 已验证结论、合理推断与待验证假设

### 论文已验证

- 在两个指定流水线和给定候选实现中，加入人工 criteria 选择与 16 条标签后，EvalGen 选出的 assertion 集比 SPADE 更小，alignment 相同或更高。
- 在 9 名有 LLM 生产经验的参与者中，自动建议 criteria 普遍被视为有用起点，但最终对齐感受差异很大；参与者希望继续查看、修改和部署前验证 assertion。
- 本次短时任务中确实观察到两类 criteria drift：发现新错误后新增维度，以及在实例刺激下改变既有 criterion 的含义或阈值。

### 基于证据的合理推断

- 评测数据集不只是用于估计一个固定 rubric 的性能，也会参与形成 rubric；因此应记录 criterion 版本、触发修改的样例和重标历史，否则“gold label”会掩盖标准形成过程。
- 代码 grader 与 LLM grader 需要不同的验证界面：前者强调可读实现、单元测试和边界例；后者强调带证据的解释、标注样例、稳定性与持续重校准。
- 单一总体 agreement 会混淆两个失败源：标准遗漏（没有 criterion 抓住某类错误）与实现偏差（criterion 正确但 assertion 写错）。EvalGen 的分层界面有助于定位二者，但论文尚未把这种诊断收益量化。

### 待实验验证

- criteria drift 是否随着标注量增加而收敛，还是在模型、prompt、输入分布变化后持续重现；需要纵向实验和 criterion 版本轨迹，而不是一次 40 分钟探索。
- EvalGen 的 alternating sampling 是否优于不确定性采样、分层抽样或覆盖新错误簇的采样；附录只显示“非随机通常更稳定”，没有证明当前策略最优。
- 用同一批输出形成标准和检验 assertion 是否过拟合；应保留时间外或分布外 holdout，并测量 criteria、实现和阈值分别迁移多少。
- 二元 good/bad 标签是否足以处理多主体冲突偏好；需要比较单一负责人、多人分歧建模和按用户/场景条件化的 evaluator。

## 局限性

- 用户研究只有 9 人，且为社交媒体自选、有经验从业者；任务主要是 100 条 tweet 的 NER，不能直接外推到长文本、agent 轨迹或高风险领域。
- 研究是定性可用性探索，不是随机对照实验；没有 no-EvalGen 条件，也没有测量节省的标注时间、上线后的缺陷率或未来数据上的 judge accuracy。
- 离线实验只有两个流水线，医疗数据虽来自真实场景，但 ground truth 由两名作者建立；文中未给出标注者间一致性。
- alignment 把 coverage 和 `1-FFR` 等权合并，但真实产品中漏掉坏输出与误杀好输出的代价可能不对称；商品任务需把 FFR 阈值放宽到 40% 才有可行解，本身暴露了指标可用性的限制。
- GPT-4 负责生成 criteria 和候选实现，GPT-3.5-Turbo 负责目标输出；结论依赖 2024 年模型行为、提示和价格。系统也没有解决 assertion 的版本维护、漂移告警和生产执行成本。

## 与仓库已有论文和主题的主动关联

- **SPADE / DocETL：** SPADE 把 prompt 修改历史转成 assertions，EvalGen 在它上面加入人确认 criteria 与少量标签选实现。SPADE 更像离线合成和集合优化，EvalGen 更像在线的标准—实例共同澄清；两者组合说明“能自动生成检查器”与“检查器符合人的意图”是不同问题。
- **Judging LLM-as-a-Judge / G-Eval：** MT-Bench 和 G-Eval 证明 LLM judge 可以规模化且与人有一定相关，但 EvalGen 关注的是任务所有者如何验证本地 judge，而不是再报告一个跨任务平均相关系数。
- **Evaluating the Effectiveness of LLM-Evaluators：** 仓库实践条目强调用专家标签、precision/recall 和 failure examples 校准 evaluator；EvalGen 把这套建议实现成界面，并用 coverage/FFR 分别显示漏检和误杀。
- **AI Agents That Matter：** 前者审计 benchmark、成本、holdout 与 harness；EvalGen 审计 scoring specification 本身。即使执行层完全标准化，如果 criterion 或 assertion 与人的构念错位，排行榜仍然无效。
- **AgentRewardBench / Counsel：** EvalGen 处理面向文本输出的本地 binary assertions；后续 agent meta-evaluation 将相同问题推进到长轨迹，要求 judge 不只判最终成功，还要正确定位错误步骤并给出可信理由。

## 与近期 AI / 评测论文的关系

- **Human-Centered Design Recommendations for LLM-as-a-Judge** — arXiv 2024-07-03，https://arxiv.org/abs/2407.03479 。EvaluLLM 对 8 名领域专家的访谈同样发现，用户需要帮助定义标准，同时担心 judge 的透明度、控制权与可靠性。它与 EvalGen 形成独立的 HCI 证据：human-in-the-loop 不是最后抽查几个分数，而要参与 criterion 形成和校准。
- **LLM-Rubric** — ACL 2024，https://aclanthology.org/2024.acl-long.745/ 。它固定 9 个手工 rubric 问题，再用带 judge-specific 与共享参数的小网络校准 LLM 分布，使 1–4 分总体满意度预测 RMSE 低于 0.5、约为未校准基线的一半。它解决“已给定多维标准后怎样拟合不同人”，而 EvalGen 解决“标准和实现怎样在看样例过程中形成”；两者的前提互补而非替代。
- **The Progress Illusion** — Findings of EMNLP 2025，https://aclanthology.org/2025.findings-emnlp.1036/ 。该文把 validator validation 从实例级推进到系统排名：AlpacaEval 在常规模型集合上的 Kendall’s τ 为 0.86，但在分差小于 2 分、接近真实迭代幅度的模型对上降到 0.19。它支持 EvalGen 的核心警告：总体相关性不能代替在实际决策边界附近的局部验证。
- **Rethinking Rubric Generation for Improving LLM Judge and Reward Modeling for Open-ended Tasks** — arXiv 2026-02-04，https://arxiv.org/abs/2602.05125 。RRD 用递归分解—过滤扩大 coverage、移除方向错误和冗余标准，并按相关性加权；在 JudgeBench 上最高提升 17.7 个点，还把 rubric 用作 RFT reward。它把 rubric 工程规模化，但其自动 refinement 并未消除 EvalGen 的问题：由谁确认分解后的维度真是人的意图，以及标准是否被观察到的输出分布塑造。
- **From Rubrics to Reliable Scores (Rulers)** — arXiv 2026-01-13，https://arxiv.org/abs/2601.08654 。Rulers 把问题称为 criteria transfer：锁定 task-level rubric，以结构化、证据支撑的判断执行，再校准到人的分数边界，并测试语义等价 rubric 扰动的稳定性。它补强 EvalGen 未覆盖的 inference-time 可审计性与尺度校准，但“何时锁定 rubric”仍受 criteria drift 挑战。

这条研究线可分成三个层级：EvalGen/EvaluLLM 研究**人怎样形成和确认标准**；LLM-Rubric/Rulers 研究**怎样把标准稳定映射为人的分数**；Progress Illusion 研究**这些分数能否支撑接近真实研发幅度的系统比较**。RRD 再把 rubric 扩展到训练 reward。尚未解决的共同问题是：如何在持续变化的数据和模型上同时保持标准可修订、版本可审计、holdout 不泄漏，并对不同利益相关者的冲突偏好负责。

## 一句话判断

EvalGen 最重要的贡献不是又一个 rubric 生成器，而是把“评测标准会在看见输出时改变”变成可观察的系统与研究对象：自动 evaluator 的有效性必须沿着 **criterion → implementation → sampled labels → future distribution** 逐层验证；论文给出了前两层的可用原型和小规模证据，长期迁移与生产效度仍待验证。

## Themes

4 observability & eval surfaces · 5 evaluation infrastructure · 8 LLM-as-judge & verifiers · human-in-the-loop evaluator alignment · criteria drift
