# Notes — “Benchmarking the Benchmarks: A Validity Audit of Tool-Calling Evaluation”

**作者：** Vishvesh Bhat, Jay Vaghasiya, Muhammad Ahmed Mohsin, Asad Aali（按 arXiv 元数据；PDF 首页把前两位作者顺序对调）· **首次提交：** 2026-06-30 · **当前版本：** arXiv v1（preprint）· **URL：** https://arxiv.org/abs/2607.02577 · **类型：** benchmark validity audit + evaluation infrastructure paper · **Found:** true

## 论文问题与科学动机

工具调用 benchmark 通常把自动 evaluator 输出的 pass/fail 当作模型能力真值，但真正需要成立的是：**同一条 agent 轨迹交给 evaluator 与懂任务的人类审查时，二者是否对“用户目标是否完成”作出相同判断？** 如果不成立，排行榜比较的可能不是 agent capability，而是 state matcher、字符串断言、rubric 生成器和 judge 随机性的混合产物。

论文把 benchmark 写成三元组 `B=(T,E,G)`：任务集合 `T`、执行与评分 harness `E`、成功标准 `G`。agent 运行得到包含工具调用和环境状态变化的轨迹 `τ`，官方 evaluator 给出 `E(τ)`，专家审查给出 `H(τ)`。论文的核心审计对象就是 `E(τ) != H(τ)` 的案例，而不是重新比较哪个模型分数更高。

这个问题有两类相反风险：

- **假阴性：** agent 已完成用户目标，却因为无关状态差异、标点、固定轨迹或错误 reference 被判失败；
- **假阳性：** agent 没有执行必要动作，却因目标状态原本不变、rubric 漏项或 judge 被流畅回答说服而通过。

前者会低估可行策略并锁死 agent 的实现路径，后者会把“声称完成”误当成“真实完成”。因此论文主张，benchmark 不能只发布一个 aggregate score；还应让 evaluator 本身接受人工对齐、重复运行和 trace-level 审计。

## 核心方法和具体机制

### 1. 四类 benchmark 的统一轨迹审计

作者选择 BFCL v4、τ²-Bench Retail、LiveMCPBench 和 MCP-Atlas，覆盖两种主要评分范式：

- **确定性 evaluator：** BFCL 以 AST 与 simulator state matching 为主；τ²-Bench 结合数据库状态、动作、通信和自然语言断言；
- **LLM evaluator：** LiveMCPBench 由动态生成的 key points 和 LLM judge 评分；MCP-Atlas 根据 reference claims 判断覆盖情况。

每个实例保留用户请求、完整轨迹、工具调用/返回、最终回答、官方 verdict、evaluator 日志和可用环境状态。三名专家检查这些证据，以用户目标是否真实完成给出 PASS/FAIL，分歧再 adjudicate。论文报告共投入 89 annotator-hours，平均每题约 10.8 分钟。

发现分歧后，作者把原因分为：

- 确定性侧：exact-match constraint、state over-specification、trajectory lock-in、annotation/ground-truth error、reward-basis mismatch；
- LLM judge 侧：rubric drift、judge variance、hallucinated completion、answer-only scoring、论文规格与实际实现不一致。

### 2. 将单一 pass/fail 拆成三个可定位分量

论文提出把评价结果从一个二值标签拆成：

- `C_tool`：工具是否被正确选择与调用；
- `C_task`：任务要求是否完成；
- `C_outcome`：最终环境状态与用户可见结果是否得到可靠验证。

这个分解的价值不是增加三个分数，而是把失败定位到“调用错了”“任务没完成”或“验证器没看对”。否则 aggregate fail 无法区分 agent failure 与 evaluator failure，aggregate pass 也无法证明实际副作用发生过。

### 3. Tool-Veritas：deterministic-first，LLM 只能补充不能翻案

Tool-Veritas 把每题写成 `t=(q,s0,A,G,J)`：用户请求、初始 sandbox、工具集合、确定性状态谓词 `G`、可选定性 rubric `J`。每一轮后都检查可观察状态，例如文件是否创建、数据库记录是否改变、日历事件是否存在、必要的 tool-mediated action 是否完成。

设计关键点有三项：

1. 任一必需 deterministic gate 未通过，任务不能靠 LLM judge 的流畅解释获得事实完成分；
2. LLM 只评价难以写成状态谓词的沟通质量、policy adherence 和用户确认充分性；
3. 工具调用失败后允许一个有界 repair window，但 first-attempt completion 与 completion-after-repair 分开记录，避免把“最终修好”与“首次可靠”混成一项能力。

每轮 JSONL 记录模型动作、工具返回、状态变化、gate 结果、repair 状态和可选 judge 输出。当前稿称 benchmark 覆盖 16 个领域，包括文件、数据库、版本控制、银行、旅行、日历、智能家居和医疗工作流。

### 4. Harness Lab：把评分过程变成可检查实验对象

Harness Lab 不是新 evaluator，而是跨 benchmark 的运行与诊断层。论文称它支持 7 个 benchmark family、21 个 suite variant，并保留 raw harness output、官方 score、inference log、case summary、逐轮诊断和执行元数据。

它支持：定位第一处失败、对齐两次运行的 case、识别分叉 turn、只重试瞬时失败的样本、重新合并官方得分，以及把 human adjudication 与官方 label 分开存储。其科学意义在于：版本、patch、数据、endpoint 和 evaluator 配置都成为结果的一部分，而不是排行榜背后的隐变量。

## 实验设置与主要结果

### 四个 benchmark 的 evaluator–human 对齐

| Benchmark | Agent | 审计数 | 人类一致率 | 分歧数 / 错误率 |
|---|---|---:|---:|---:|
| τ²-Bench Retail | Kimi-K2.6 | 112 | 90.0% | 11 / 9.8% |
| BFCL v4 | MiniMax-M2.7 | 200 | 80.0% | 40 / 20.0% |
| MCP-Atlas | MiniMax-M2.7 | 89 | 87.0% | 12 / 13.5% |
| LiveMCPBench | MiniMax-M2.7 | 95 | 69.0% | 29 / 30.5% |
| **合计** | — | **496** | **81.5%** | **92 / 18.5%** |

这证明失配同时存在于 deterministic 与 LLM-judge evaluator，但不能据此断言“LiveMCPBench 天生比 BFCL 差 10.5pp”：四组任务、轨迹和部分 agent 不同，并不是控制其他变量后的 evaluator A/B test。

### 两个具体错误机制

- **BFCL state mismatch：** 在作者重点检查的 50 个 multi-turn base export 中，25 个官方失败里有 20 个（80%）被标为 `instance_state_mismatch`。人工检查发现其中既有真实遗漏，也有任务无关字段、标点差异、全对象比较以及未允许后续纠错造成的 artifact。
- **τ²-Bench 错误 reference：** 某题要求取消两笔预订后报告其余航班总价。agent 完成五个预期动作并报告剩余两笔 `$402+$306=$708`，但字符串断言要求回答包含 `1628`；reference 把本应取消的预订也计入，导致正确任务结果被判错。另一个反例中，agent 没执行退货、只转人工，但数据库恰好保持不变，反而获得 pass。

### LiveMCPBench 重跑方差

同一个 95-task 配置完整重跑 23 次，得分从 57.9%（55/95）到 76.8%（73/95），均值 69.4%，标准差 5.4pp，极差 18.9pp。默认 OpenBench scorer 每题先调用 `identify_key_points(task)` 重新生成 rubric，再由 `gpt-4.1-mini` 判断轨迹与回答；不同运行既可能生成不同严格度的 criteria，也可能在最终 verdict 上翻转。

但这个结果测到的是 **end-to-end pipeline instability**：agent trajectory sampling、rubric generation 和 judge sampling 同时变化。论文自己承认，若要测 judge-only variance，应固定轨迹并分别在固定/重生成 rubric 下重复 rescoring。因此 `18.9pp` 不能被简写成“同一轨迹仅因 judge 随机性波动 18.9pp”。

### Tool-Veritas 的初步结果

70 个任务由 6 个模型分别运行，共 420 个 model–task evaluation。Tool-Veritas 与专家判断一致 401 次，即 95.5%；19 个分歧全是假阴性，没有观察到 benchmark pass、专家 fail 的假阳性。模型级一致率为 91%–100%。

这个结果支持“确定性事实 gate + 受限 LLM 补充”值得继续测试，却不是对四个旧 benchmark 的公平胜负：Tool-Veritas 使用了不同任务分布和模型配置，论文也明确把它解释为 evaluator design evidence，而非 controlled benchmark ranking。

## 已验证结论、合理推断与待验证假设

### 论文已验证

- 在作者审计的 496 条轨迹上，官方 label 与专家 task-success 判断有 92 次分歧，四个 benchmark 都出现失配。
- 确定性 evaluator 不等于语义正确：无关 state field、错误 reference、字符串断言和“未变化即成功”都能系统性制造错判。
- LiveMCPBench 的默认端到端 pipeline 在 23 次完整重跑中有 18.9pp 极差；单次分数不足以支撑细微排行榜差异。
- 在作者新建的 420 个 Tool-Veritas model–task evaluation 上，deterministic-first protocol 与专家判断的一致率为 95.5%，观察到的 19 个错误全部偏严格。

### 基于证据的合理推断

- agent benchmark 应同时发布 point estimate、重复运行分布、evaluator–human 校准样本和 trace/artifact，而不是只发布一个 pass rate。
- deterministic 与 LLM judge 不是二选一：事实副作用适合状态谓词，开放表达质量才交给受限 judge；关键是限制每类 evaluator 的权限边界。
- “正确工具调用”“任务完成”“结果被证实”应分开记录。三者合并后，研究者无法判断改模型、改 harness 还是修 grader。
- benchmark 的可复现性不仅取决于随机种子，还取决于 rubric 是否每次重生成、judge 版本、工具/API 状态、retry policy 和环境快照。

### 待实验验证

- 把完全相同的固定轨迹交给原 evaluator 与 Tool-Veritas，做 matched-task、matched-model、matched-evidence 比较后，95.5% 优势是否仍成立。
- 固定 LiveMCPBench 轨迹，分别控制 rubric 与 judge 随机性，量化三类方差各占多少，而不是只看整条 pipeline 极差。
- Tool-Veritas 在状态难以观测、结果延迟出现、跨系统最终一致性和安全约束冲突的任务上，deterministic gate 是否仍可维护。
- 受限 LLM judge 是否会在 qualitative criteria 上保留 self-preference、rubric drift 或同源模型 shared-error；论文没有独立做 judge-family 对照。
- bounded repair window 的长度怎样影响能力解释：它究竟测可靠执行、恢复能力，还是额外推理预算。
- 不同模型和不同错误分布下，四个原 benchmark 的 misalignment rate 是否稳定；当前每个 benchmark 只审计一种 agent 配置。

## 局限性与可信度警告

- **不是受控的跨 benchmark 比较。** 四个 error rate 来自不同任务集合、evaluator 和部分不同 agent；只能证明每个被审计设置存在问题，不能直接排序 benchmark 质量。
- **LiveMCP 方差没有隔离 evaluator。** 23 次是完整重跑，agent 轨迹也变化；`18.9pp` 不能单独归因给 rubric/judge。
- **人工审查报告不完整。** 论文称三名专家独立检查并 adjudicate，但没有给 inter-annotator agreement、每题是否三人全量标注、盲评设置和按 benchmark 的置信区间。
- **Tool-Veritas 对照不匹配。** 95.5% 来自新的 70 题 × 6 模型，而被审计 benchmark 使用另一组任务/模型；更高一致率可能部分来自题目更容易写成确定性谓词。
- **样本和选择机制交代不足。** 论文未充分解释 496 条如何从各 benchmark 全集抽取、是否先由规则“flagged”再人工审查，以及这种流程是否低估未触发诊断的错判。
- **当前不可复核。** v1 的 Availability 写的是“will release / being prepared for release”，没有给 Tool-Veritas、Harness Lab、修复 evaluator、原始轨迹或 human label 的仓库链接；摘要却使用“release/open-source”的完成时态。
- **参考文献存在明显草稿错误。** PDF 把 MCP-Atlas 写成 `arXiv:2601.XXXX`（官方为 https://arxiv.org/abs/2602.00933 ），把 τ²-Bench 写成 `2503.XXXX`（官方为 https://arxiv.org/abs/2506.07982 ），并把 LiveMCPBench 错标为 `2506.07982`（该编号其实是 τ²-Bench；LiveMCPBench 为 https://arxiv.org/abs/2508.01780 ）。正文末尾还残留 LaTeX/引号碎片。
- **元数据也未完全一致。** arXiv 页面作者顺序为 Vishvesh Bhat、Jay Vaghasiya，PDF 首页相反。它不改变实验结论，但提示当前 v1 尚未完成出版级校对。
- **仍是 12 页 v1 preprint。** 没有同行评审结论；尤其是“release”“open-source”和跨 benchmark 优势必须等公开 artifact 后再确认。

## 与仓库已有论文和主题的主动关联

- **BenchJack：** BenchJack 主动寻找可被 agent 利用的 benchmark loophole；本文则从已经运行的轨迹出发，判断 evaluator 是否把真实成功标错。前者审计“agent 能否骗分”，后者审计“grader 是否理解结果”，两者共同覆盖 protocol validity。
- **The Verification Horizon：** 该文把 verifier 定义成会随 policy 失效的训练基础设施；本文给出工具调用场景的具体失败：固定 state matcher 过窄，开放 judge 又会漂移。Tool-Veritas 是一种分层 verifier，但尚未验证长期随 policy 共演化。
- **Counsel：** Counsel 用人类 meta-label 检查 judge critique 的定位与推理；本文把 human reference 改成 task-success verdict。二者都强调“judge 输出不是 ground truth”，但本文没有像 Counsel 那样细分 human disagreement 或公开 annotation artifact。
- **Who Validates the Validators? / EvalGen：** EvalGen 让用户参与 criteria coverage 与 false-failure alignment；本文的错误 reference 案例说明，criteria 即使可执行也可能语义错误。确定性并不能代替标准本身的人工确认。
- **AI Agents That Matter：** 该文要求统一 harness、成本和 holdout 才能比较 agent；本文进一步说明，即使 harness 固定，evaluator artifact 仍会改变结果。模型能力应绑定 model + harness + evaluator + version 报告。
- **LLM Evaluators Recognize and Favor Their Own Generations：** 本文没有系统控制 generator/judge 同源性。LiveMCPBench 使用 LLM 生成 rubric 再用 LLM 判分时，除了随机性，还可能存在风格熟悉与 shared-error，需独立模型和固定 criteria 对照。

## 与近期 AI / 评测论文的关系

- **Claw-Eval: Towards Trustworthy Evaluation of Autonomous Agents** — 2026-04-07 首次提交、2026-05-07 更新，https://arxiv.org/abs/2604.06132 。它用 execution trace、audit log、environment snapshot 三条证据通道评价 300 个任务，并显示只看终局会漏掉 44% safety violation 与 13% robustness failure。本文更聚焦 evaluator–human label mismatch；两者共同支持 trace-aware grading。
- **Auditing Automated Evaluation, Error Propagation, and Runtime Mitigation in Tool-Using Language Agents / AgentProp-Bench** — 2026-04-17，https://arxiv.org/abs/2604.16706 。它在 14,750 条轨迹上发现 substring heuristic 与人工判断的 κ=0.049，并以 100 条分层人工标签校准 judge。它与本文的 τ² 字符串反例高度一致，但给出了 agreement statistic；本文则跨四个 benchmark 做更直接的 case audit。
- **Harness-Bench** — 2026-05-27，https://arxiv.org/abs/2605.27922 。它在 106 个任务、5,194 条轨迹上把 harness configuration 当作测量单位，主张结果应归因于 model–harness pairing。本文再加一层：同一 pairing 的分数还取决于 evaluator 与 rubric 版本。
- **The Verification Horizon** — 2026-06-24，https://arxiv.org/abs/2606.26300 。它主张 verifier 必须随 generator 共演化；本文在 6 天后给出工具调用 evaluator 的横截面审计和 deterministic-first 修复方向，但没有进行多代 policy–verifier 纵向实验。
- **Do Agent Benchmarks Measure Capability? Protocol Validity in the Age of Agentic AI** — 2026-07-24，https://arxiv.org/abs/2607.22368 。这篇后续工作把问题扩展为 `Expose → Exploit → Mislead`，审计 15 个 benchmark、2,385 条轨迹，并用 Mislead gap 量化 0.45–1.00 的分数膨胀。本文测 evaluator 与人工是否一致；它进一步追问错误分数是否真的由 shortcut 使用造成。
- **LiveMCPBench 与 MCP-Atlas 原论文** — 2025-08-03，https://arxiv.org/abs/2508.01780 ；2026-01-31，https://arxiv.org/abs/2602.00933 。两者分别报告 LiveMCPEval 与 claims-based scoring 的人工一致性证据，而本文在具体实现和新 agent 轨迹上发现更高失配。最合理的解读不是谁“推翻”谁，而是 evaluator agreement 必须绑定版本、任务分布、模型和实际 runner 复测。

近期脉络可以概括为：**先记录完整执行证据（Claw-Eval），再把 harness 当作能力组成部分（Harness-Bench），随后直接审计 evaluator–human mismatch（本文），最后把 exposure、agent 使用和分数误导串成因果链（Protocol Validity）。**

## 一句话判断

这篇论文最有价值的结论是：工具调用 benchmark 的 grader 也必须被当作待评测系统，单次 pass rate 不能自动升级为能力真值；但当前 v1 的未公开 artifact、非 matched 对照、未分解的方差，以及多处错误/占位参考文献，使它更适合作为重要审计线索，而不是已经完成复现闭环的定论。

## Themes

4 benchmarks · 5 reliability · 8 LLM-as-judge · tool calling · evaluator validity · human adjudication · harness variance · deterministic state verification · trace-level audit · protocol validity
