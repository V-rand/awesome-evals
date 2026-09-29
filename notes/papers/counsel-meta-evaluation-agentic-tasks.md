# Notes — “Counsel: A Meta-Evaluation Dataset for Agentic Tasks”

**作者：** Sashank Pisupati, Henry Broomfield, Eujeong Choi, Antonia Calvi, Charlie Wang, Roman Engeler, Max Bartolo, Patrick Lewis · **首次提交：** 2026-06-19 · **当前版本：** arXiv v1（preprint）· **URL：** https://arxiv.org/abs/2606.21627 · **数据：** https://huggingface.co/datasets/AtlaAI/counsel · **类型：** agent-judge meta-evaluation dataset paper · **Found:** true

## 论文问题与科学动机

Agent 评测正在从“最后答对没有”转向“哪一步出了错、为什么错”。但这带来第二层问题：**当 LLM judge 给出一段看似具体的轨迹批评时，谁来验证批评本身？** 一个 judge 可能找到了真正的失败步骤，却把原因说错；也可能在合理步骤上凭空挑错。若只比较最终 `error / no error` 标签，这两类失败会被压成同一个数字，无法判断其反馈能否用于调试、guardrail 或训练。

人工逐步审阅又很昂贵。论文以 SWE-bench 为例指出，专家完成一个任务约需一小时，而审阅对应 agent 轨迹约需两小时。Counsel 因此把研究对象从“agent 是否成功”提升为“judge 的过程级诊断是否可信”，并分别检查两个维度：错误位置是否正确，以及解释是否正确。

这项工作的科学动机不是再造一个 agent 排行榜，而是提供一层可审计的元评测数据：让研究者能区分“发现了真问题”“位置对但理由差”“根本不该报错”，进而校准或训练更可靠的 agent evaluator。

## 核心方法和具体机制

### 1. 在两个可程序化验收的 agent 环境中生成轨迹

作者选择两类常见任务：

- **τ-bench retail：** 多轮客服对话、业务规则和 API 工具调用，共 115 个可尝试任务；最终数据库状态和必要确认可程序化评分。
- **DA-Code：** 数据清洗、分析、建模和多文件执行。作者只保留 Data Insights 与 Data Manipulation 子集，并用 o3 和 Claude-3.5-Sonnet 做额外质量筛选：两者在某任务上的样本通过率都低于 50% 时丢弃，最终保留 50 个任务。这个筛选减少了 benchmark 自身欠说明或严格格式匹配造成的伪失败，但也把任务偏向较容易子集。

agent 使用开放权重模型以便数据开放：τ-bench 包含 GPT-OSS-20B（medium reasoning）和 Qwen3-235B-A22B-Instruct-2507；DA-Code 只使用 Qwen3。最终数据包含 225 条“至少被一个 judge 标出一处错误”的唯一轨迹，其中 τ-bench 185 条、DA-Code 40 条。

### 2. 对每个模型调用 span 做在线式独立判断

三个 judge 配置为 GPT-OSS-120B（low reasoning）、GPT-OSS-120B（high reasoning）和 Qwen3-235B-A22B-Instruct-2507（无显式 reasoning）。每次 judge 只看到 agent 当时能看到的历史、可用工具和当前输出，不看未来步骤，也不接收前面 span 的 judge 输出；各 span 独立判断是否存在错误并给出简短、具体的 critique。

这个约束保留了在线 guardrail / process reward model 的实际信息边界：judge 不能借用失败后的结果倒推前一步。然而，人类元标注者会看到完整轨迹和全部 judge 输出，因此其判断具有额外的事后信息。Counsel 测的是这种“事后人类标准下，在线 judge 的批评质量”。

### 3. 用三分类同时标注位置和解释

人类只审阅 judge 已标错的 span，并使用统一标签：

- **Spot On：** 错误位置正确，批评理由也正确；
- **Poor Reasoning but Correct Location：** 位置正确，但理由错误、遗漏关键点或不充分；
- **Should Not Have Flagged：** 该位置本不应被标错，位置和相应理由都不成立。

论文不设“位置错但理由对”，因为理由若解释的是一个并不存在于该位置的错误，就不能在上下文中算正确。这个标签设计把 location precision 与 reasoning quality 拆开，也明确牺牲了完整 recall：未被任何 judge 标出的步骤没有人类逐步标注。

### 4. 专业标注、校准与质控

三位数据科学 / NLP 从业者均有十年以上经验。最初 15 条轨迹由三人独立标注，Krippendorff's α 约为 0.78；其余轨迹单人标注，并由 senior annotator 定期复核。平均每条 τ-bench 轨迹约 20 分钟、DA-Code 约 30 分钟。标注者还可写自由文本 meta-comment，但论文明确说该字段不是核心目标，也未做同等质量控制。

## 实验设置与主要结果

- **数据规模与标签分布。** 1,131 条 critique 元标注中，555 条 Spot On、289 条位置正确但理由差、287 条不应标错。换言之，在这个“只看已报错”的样本中，约 49.1% 的批评位置和理由都对，约 25.6% 找对位置却没说对原因，约 25.4% 是误报；这些比例不能当作所有 agent step 上的准确率或召回率。
- **领域会改变 judge 排名。** Qwen3 judge 的 Spot-On precision 在 DA-Code 为 43%，在 τ-bench retail 为 63%（跨 agent 的 micro-average）；GPT-OSS-120B:high 则分别为 60% 和 58%。不存在一个模型在两个环境都按同样幅度领先。
- **更多推理算力改善批评质量。** GPT-OSS-120B:high 在两个 benchmark 上都比 low 版本产生更少的 Poor Reasoning 和 Should Not Have Flagged。摘要把最强 judge 概括为错误位置约 88% 与人类一致、理由约 65% 一致；这是特定最佳配置的结果，不是三种 judge 的总体平均。
- **“严格”不等于“准确”。** GPT-OSS judge 比 Qwen3 更频繁标错；更高 flag rate 既可能带来更多真阳性，也可能增加误报。因此论文主要报告 critique precision，只在固定标注池内用真阳性数量比较相对 recall。例如 DA-Code 中，GPT-OSS-120B:high 的 33 个 Spot On 相对 Qwen3 的 19 个多约 74%，但仍不能得到绝对 recall。
- **未发现模型家族自偏好。** 作者比较同家族与跨家族 judge-agent 配对，并用单侧比例检验；三种 judge 都没有显著表现出“对自家模型更少报错”。这是当前模型、任务和样本量下的零结果，不应外推为自偏好普遍不存在。
- **few-shot 回路实验只有初步正信号。** 附录在 τ-bench retail 中把 20 个元标注示例放进 GPT-5-mini agent 或 guardrail judge 的系统提示，比较无示例、20 个 Spot On、20 个差/错反馈和 10/10 混合。十次完整运行中，只有“向 judge 提供 Spot On 示例”相对无反馈达到显著改善（p=0.0078）；其余效应小且不显著。示例与测试来自同一环境，虽避免同一用户任务泄漏，仍不能证明跨环境泛化。

## 已验证结论、合理推断与待验证假设

### 论文已验证

- 在 τ-bench retail 与经筛选的 DA-Code、三种开放权重 judge 配置上，judge critique 的“位置正确”与“理由正确”明显不是同一能力；1,131 条已报错批评中有 289 条位置对但理由不充分或错误。
- 更强 judge 和更多 reasoning effort 在作者的设置中提高了与人类元标注的一致性，但不同领域会改变相对表现。
- 15 条三人重叠轨迹上的 Krippendorff's α 约 0.78，说明三分类任务在受训专业标注者间具有较高但非完美一致性。
- 当前实验没有发现显著的同家族自偏好；Spot-On 示例用于 in-loop judge 时出现一次显著的下游收益。

### 基于证据的合理推断

- 只报告 judge 的最终二元准确率会高估其诊断价值。对于调试和训练，位置对但理由错的 feedback 可能把 agent 推向错误修复方向，因此应把 localization 与 explanation 分开计量。
- judge 的“批评得多”不能自动解释为覆盖更好。实际部署至少应同时记录 flag rate、已标错误的 precision、跨 judge 的相对 recall，并对未标步骤抽样做人类审计。
- 高质量元标注可以用于训练 meta-judge，先过滤虚假或理由薄弱的 critique，再把剩余反馈交给 agent；但 Counsel 本身主要证明数据可构建，尚未证明这种训练能稳定改善跨域 agent 成功率。

### 待实验验证

- 对未被任何 judge 标出的 span 做分层抽样或完整标注后，各 judge 的绝对 error-localization recall、漏错类型和 precision-recall 曲线如何。
- 用 Counsel 训练 meta-judge、judge 或 process reward model，是否在 held-out benchmark、held-out agent family 和真实在线 guardrail 中优于 matched-budget baseline。
- “更多 reasoning effort 提高批评质量”是否由更好的因果诊断带来，还是只是生成更长、更保守的解释；需要长度、预算和 prompt 都匹配的对照。
- 人类元标注者拥有未来信息，而在线 judge 没有。若把人类也限制在相同前缀信息，哪些标签会改变，在线可判定性边界是否更公平。
- few-shot Spot-On 示例的收益能否跨环境复现，并排除相同 benchmark 规则与表达方式带来的 in-domain 记忆效应。

## 局限性

- **没有绝对召回率。** 数据只覆盖至少被 judge 标出一处错误的轨迹和被标出的 span；“未报错”可能是真阴性，也可能是三种 judge 共同漏掉的错误。
- **范围较窄。** 只有客服与数据科学代码两个环境、两个 agent 家族和两个 judge 家族；DA-Code 又经过容易度 / 可靠性筛选，结论不能直接代表浏览器、GUI、科研或长周期 agent。
- **标注信息不对称。** 人类看完整轨迹和未来 judge 输出，judge 只看当前前缀。该设计适合事后质量审计，却可能把事后才能确认的问题算作在线 judge 错误。
- **一致性估计样本小。** α≈0.78 来自最初 15 条三人重叠轨迹；其余为单标加周期复核，论文未报告全程隐藏质检题或各类别一致性。
- **自由文本 meta-comment 未做核心质控。** 数据虽开放该字段，但不宜未经清洗直接当作高质量监督信号。
- **下游效用证据初步。** few-shot 实验只有一个配置显著，且同环境、prompt 工程很少；不能据此断言 Counsel 已能普遍提高 agent 完成率。
- **开放数据不等于覆盖开放问题。** MIT 许可和开放权重模型利于复现，但人类标准仍可能受 τ-bench / DA-Code 特定规则、标注指南和事后信息影响。

## 与仓库已有论文和主题的主动关联

- **AgentRewardBench：** AgentRewardBench 评估整条 web-agent 轨迹的 success、side effects 和 repetitiveness，揭示 rule-based grader 会漏掉有效成功；Counsel 把粒度推进到每个已报错 span，并把“找对位置”与“解释正确”分开。前者校准 outcome evaluator，后者校准 diagnostic critique，二者共同说明 agent evaluator 也需要独立 ground truth。
- **Agent-as-a-Judge：** 该工作用有工具和层级 rubric 的 agent evaluator 提供过程反馈，并报告接近人类；Counsel 提供了验证此类丰富反馈的元标注框架。评估者变得更 agentic 并不自动保证 critique soundness，仍需逐步位置与理由审计。
- **Who Validates the Validators? / EvalGen：** EvalGen 通过 coverage 与 false-failure rate 让用户校准标准；Counsel 的 Spot On / Poor Reasoning / Should Not Have Flagged 是 agent 轨迹版本的 validator validation。差别是 EvalGen 围绕用户定义 criteria，Counsel 围绕专业标注者对开放式错误批评的判断。
- **RewardBench 2：** RewardBench 2 测静态偏好 / reward model 的多技能排序与下游相关性；Counsel 测 agent 过程批评是否定位正确、理由是否成立。二者都反对用单一总体 accuracy 替代分能力元评测，但 Counsel 还暴露了“位置对、理由错”的结构性中间态。
- **LLM Evaluators Recognize and Favor Their Own Generations：** 仓库已有报告表明来源自偏好可系统出现；Counsel 在三种 agent-judge 家族配对上没有发现显著自偏好。两者不矛盾：任务、输出形式、家族定义、统计功效不同，Counsel 的结果应视为局部零结果而非反证。
- **BenchJack / One Token to Fool LLM-as-a-Judge：** 这两项工作从 benchmark scoring 和生成式 verifier 的可利用漏洞出发；Counsel 关注自然轨迹中的误报与错误解释。它们合起来给出两种互补压力测试：自然分布下 critique 是否可信，以及攻击者 / 优化器能否主动放大 judge 的薄弱边界。
- **AI Agents That Matter：** 该文强调 evaluator、scaffold、成本与 holdout 一起决定结论。Counsel 为其中 evaluator 层提供可操作拆分：不要只问最后打分是否一致，还要问失败点与诊断理由是否能被人类复核。

## 与近期 AI / 评测论文的关系

- **AgentRewardBench: Evaluating Automatic Evaluations of Web Agent Trajectories** — 2025-04-11，https://arxiv.org/abs/2504.08942 。它收集 1,302 条、来自五个 benchmark 和四种 LLM 的 web-agent 轨迹，由专家标 success、side effects 与 repetitiveness，并发现没有一个 LLM judge 在所有 benchmark 上都最好。Counsel 延续“先评 evaluator”的路线，但从整轨迹结果进入 span-level critique soundness。
- **TRAIL: Trace Reasoning and Agentic Issue Localization** — 2025-05-13，https://arxiv.org/abs/2505.08638 。TRAIL 构建 148 条人工标注的单 / 多 agent 轨迹和错误分类，最佳长上下文模型在 trace debugging 上仅得 11%。它直接测模型能否定位问题；Counsel 则收集多个 judge 已给出的候选批评，再由人类判断位置与理由，为训练和筛选 critique 提供更密集信号。
- **Agent-as-a-Judge: Evaluate Agents with Agents** — 2024-10-14，https://arxiv.org/abs/2410.10934 。它在 55 个 AI 开发任务、365 条层级需求上展示 agent evaluator 的过程反馈能力。Counsel 提醒，对“丰富反馈”的验证不能停在最终分数相关性，还应检查每条过程批评的可定位性和因果解释。
- **From Confident Closing to Silent Failure: Characterizing False Success in LLM Agents** — 2026-06-01，https://arxiv.org/abs/2606.09863 。该工作用可验证环境状态检查 agent 的虚假成功，发现五种 judge、五种提示在 τ2-bench 上都未超过 0.65 AUROC，在 AppWorld API trace 上最高约 0.54。它证明 outcome monitor 会依赖表面完成线索；Counsel 则证明 process critique 即使报到正确步骤，也可能给出错误理由。两者共同要求 evaluator 的输出必须对环境证据可追溯。
- **Benchmarking LLM-as-a-Judge for Long-Form Output Evaluation / LongJudgeBench** — 2026-06-01 首次提交、2026-08-28 更新，https://arxiv.org/abs/2606.01629 。LongJudgeBench 发现长文 judge 在场景间不稳定，rubric 与 reference 有帮助但不足。Counsel 面对的是长轨迹而非长文；共同难点是跨段一致性和任务特定标准，但 Counsel 进一步引入在线信息边界与工具状态。
- **Reward Reasoning Model** — 2025-05-20，https://arxiv.org/abs/2505.14674 。该工作用 RL 让 reward model 在给分前主动推理，并报告 test-time compute 改善 reward accuracy；Counsel 对应地发现 high-reasoning judge 比 low 版本更接近人类。不过 Counsel 的 Poor Reasoning 类说明“生成了推理”不等于“理由正确”，更强的 reward reasoning 仍需独立元评测。

近期脉络可以概括为：**从最终成功标签的 evaluator calibration（AgentRewardBench）→ 轨迹错误定位（TRAIL）→ 过程批评的位置与理由分解（Counsel）→ 用环境真值检查 silent failure**。Counsel 的独特位置不是提出更强 judge，而是提供一层能训练和审计 judge 的人类 critique-quality 数据。

## 一句话判断

Counsel 最有价值的贡献，是把“judge 找到错误”拆成可复核的两步：位置找对了吗，理由说对了吗；1,131 条元标注证明两者经常分离。但它只精标已被 judge 报出的错误，因此当前数据更适合研究 critique precision、过滤与校准，不能单独支撑“judge 已覆盖大多数真实错误”的结论。

## Themes

8 LLM-as-judge · agent trajectory evaluation · meta-evaluation · critique quality · error localization · process supervision · guardrail judges · human annotation
