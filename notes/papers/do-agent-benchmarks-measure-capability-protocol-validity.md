# Notes — “Do Agent Benchmarks Measure Capability? Protocol Validity in the Age of Agentic AI”

**作者：** Jiaqi Shao, Hanck Chen, Wei Zhang, Maxm Pan, Bing Luo · **首次提交：** 2026-07-24 · **当前版本：** arXiv v1（preprint）· **URL：** https://arxiv.org/abs/2607.22368 · **类型：** agent benchmark protocol audit + trace-level attribution framework · **Found:** true

## 论文问题与科学动机

agent benchmark 的高分通常被解释成“模型拥有某种能力”，例如独立推导科学答案、修复软件缺陷或在隐藏环境中完成任务。但一个更基础的问题是：**在当前 benchmark 协议里，目标能力真的是拿到分数的必要条件吗？** 如果答案、隐藏状态、生成规律或 grader loophole 已经暴露给 agent，那么分数仍可能完全可复现，却不再支持原来的能力解释。

论文把 benchmark protocol 写成 `P={E,I,S,V}`：环境 `E`、信息流 `I`、评分 `S` 和验证机制 `V`。作者关心的 validity 不是任务看起来是否合理，而是 intended capability 是否仍然对高分“不可绕过”。这比传统 contamination 更宽：训练时见过题目只是一个入口，运行时检索答案、读取隐藏文件、推断伪随机种子、利用反馈或提交无效 artifact 也都可能让目标能力失去必要性。

全文最重要的概念链是：

> **Expose → Exploit → Mislead**

- **Exposure：** 协议暴露了与得分相关的替代路径；
- **Exploit：** agent 实际使用了这条路径；这里的 “exploit” 是描述性词，不等于恶意、违规或 prompt injection；
- **Mislead：** 评分仍把替代路径产出的结果记成目标能力成功，从而让分数支持了过强的能力结论。

只发现 benchmark 文件可读，不能证明 agent 用过；只发现 agent 读过文件，不能证明读取改变了答案；只发现 grader 有漏洞，也不能证明某次分数受其影响。论文试图把泛化的“benchmark 可能被污染”变成可定位、可复核的 trace-level 归因。

## 核心方法和具体机制

### 1. 审计对象是一次运行的证据包

每次运行整理成 `D_r=(B_r,T_r,A_r,R_r,C_r)`：benchmark 规格、完整轨迹、最终 artifact、评分记录，以及可选的对照证据。作者先用高召回规则筛出候选片段，再交给固定 prompt 的 LLM judge；judge 默认使用 GPT-5.5，只能在 sandbox 中调用 `read_file`，不能联网、执行命令、修改文件或重新打分。

judge 输出结构化字段：exposure 类型、agent engagement、是否 Mislead、置信度、capability drift、建议修复位置和精确证据指针。指针必须回到 trace、artifact 或 grader record；schema 不合法、指针不存在或证据无法支持判断时，该条输出被拒绝。这个限制很关键：LLM 的任务是做证据归因，而不是凭常识猜“看起来像作弊”。

### 2. 五类 exposure 与七个具体失效面

| 暴露来源 | 具体标签 | 替代了什么 |
|---|---|---|
| Answer source | `benchmark_overexposure` | 从独立求解退化为检索、转录已知答案 |
| Hidden state | `held_out_readable`, `setting_exposure` | 从未知条件下决策退化为读取本应不可见状态 |
| Generation mechanism | `generator_regularity` | 从解决实例退化为逆向生成规则或伪随机性 |
| Interaction feedback | `feedback_inference` | 从完成任务退化为通过逐步分数或反馈探测答案 |
| Evaluation pipeline | `harness_loophole`, `invalid_scoring_path` | 从产生有效结果退化为利用 runner、缓存或 grader 缺口 |

agent engagement 又分为 ignored、passive、active、engineered，避免把“环境中存在答案”和“agent 主动工程化地搜索答案”混成一类。论文还强调一种反直觉情况：当 scorer 直接给无效 artifact 记分时，即使 agent 没有利用策略，`engagement=none` 也可能出现 Mislead，因为失效发生在评分端。

### 3. Capability drift 与 Mislead gap

除二值标签外，论文要求说明能力解释怎样漂移，例如 `independent scientific derivation → retrieval and transcription`。若存在防泄漏或修复后的配对条件，作者在 judge 之外计算：

`G = S_exploit - S_intended`

其中 `S_exploit` 是原协议得分，`S_intended` 是切断替代路径后、较接近目标能力的得分。这样 Mislead 不只是“看起来不公平”，而有一个可审计的分数差。不过作者只在五个案例中构造了足够可信的配对，因此 `G` 不能被外推成所有 benchmark 的平均膨胀率。

### 4. 协议保真度越高，验证面不一定越小

论文把环境粗分为静态、模拟、sandbox、containerized、live/adaptive 五个阶段。更真实的环境能减少静态题库的某些污染，却会增加网络、文件权限、缓存、外部状态、反馈和恢复策略等新攻击面。作者的判断不是“越 live 越无效”，而是：**每增加一个 agent 可观察或可操作的通道，就需要对应的 protocol-level 验证。**

## 实验设置与主要结果

### 覆盖范围

作者保留 2,385 条轨迹，覆盖 15 个 benchmark family、表中 16 个 cohort，包括 Frontier Science、AutoLab、SWE-bench Verified、SWE-bench Pro、FrontierSWE、SWE-bench Multilingual、nl2repo、Terminal-Bench、WildClawBench、DeepSWE、MLS-Bench Lite，以及若干内部浏览、后端、企业和软件工程 benchmark。默认轨迹由 Claude Opus 4.8 生成，默认审计 judge 为 GPT-5.5。

样本不是统一随机抽样：SWE-bench Verified、SWE-bench Pro、SWE-bench Multilingual、Terminal-Bench 和 WildClawBench 明确是预先筛出的 suspicious subset；它们的阳性率不能解释成 benchmark-wide prevalence。

### 主要发现

| 设置 | 被审计轨迹 | Mislead-positive | 正确解释 |
|---|---:|---:|---|
| Frontier Science | 494 条通过轨迹 | 331 / 67.0% | 分母是 960 次 rollout 中的 494 条 passing trace，不是全部 rollout |
| AutoLab | 36 | 24 / 66.7% | 样本很小，不能代表所有自动实验任务 |
| 其余 cohort | 各 42–300 | 0–21.7% | 五个 cohort 为可疑子集；五个 cohort 未发现阳性 |

Frontier Science 还做了跨模型检查：GPT-5.5 在 480 次运行中通过 283 次，通过轨迹的 Mislead rate 为 65.0%（95% CI 59.4–70.6）；Kimi-k2.6 通过 211 次，对应 69.7%（95% CI 63.5–75.9）。人工复核的阳性案例中，88% 的 capability-drift 描述一致指向“独立推导变成答案检索/转录”。这支持问题不只绑定某一个 generator model，但仍只验证了一个 benchmark 和两种模型。

### detector 校准

- **Frontier Science 53 条人工标签：** 38 条 source-transcription 阳性中检出 29 条、漏掉 9 条；12 条真实独立推导中出现 2 个假阳性、10 个真阴性；另有 3 条无工具的记忆式回答被判定为审计范围外。对范围内样本，precision 0.94、recall 0.76、F1 0.84。
- **MLS-Bench 21 条 held-out：** 与人工标签全部一致，其中包括“读取隐藏文件但没有用于决策”的负对照。

这个校准说明严格证据指针有助于控制假阳性，但 0.76 recall 同时意味着“未检出”绝不能自动升级为“协议有效”。论文没有报告详细 annotator 数量、inter-rater agreement 或 adjudication 流程。

### 五个可配对的 Mislead gap

| 案例 | 替代路径 | `G` |
|---|---|---:|
| WildClawBench | 空或无效 artifact 被 scorer 接受 | 1.000 |
| EdgeBench | harness loophole | 0.859 |
| EdgeBench | feedback inference | 0.835 |
| Causal topology | 利用 generator 的拓扑顺序规律 | 0.621 |
| Frontier Science | 访问答案源 | 0.447 |

Frontier Science 中，移除答案访问后总体分数下降 0.447；题目与公开 source/rubric 越接近，得分越高（重叠与分数相关 `r=0.625`），close-match 题得分 0.850，其余题为 0.403。它是全文最有力的 paired evidence，但仍是特定数据构造下的结果，不代表所有科学推理 benchmark 都有相同幅度。

作者还展示三个机制性案例：读取可见随机种子并搜索约 20 万个 seed 以预测最优臂；利用变量按拓扑序生成的规律，把因果结构得分从 0.018 提高到 0.639；复用 warmup 状态或缓存，把本应测算法的查询变成约 0.009ms 的缓存命中。这些例子说明 exposure 不必是明文答案，也可能是环境机制留下的可计算侧信道。

### 失效来源分布

在 352 条证据完整的阳性轨迹中，answer source 占 303 条（86.1%），hidden state 36 条，generation mechanism 7 条，evaluation pipeline 6 条，interaction feedback 0 条。这个分布只描述当前样本和 detection coverage；不能据此推断现实 agent benchmark 中 feedback inference 不存在。

论文把 generator regularity 评为最难检测、benchmark overexposure 最易检测，但该表是作者的 ordinal judgment，不是用盲测实验得到的 detection probability。

## 已验证结论、合理推断与待验证假设

### 论文已验证

- 在作者保留的 2,385 条轨迹中，多个 benchmark 存在“协议暴露替代路径、agent 使用该路径、评分仍误导能力解释”的完整案例链。
- Frontier Science 的高阳性结果在 GPT-5.5 和 Kimi-k2.6 两个 generator 上方向一致；移除答案源会产生 0.447 的 paired score gap。
- 有效替代路径不局限于答案泄漏：隐藏状态、generator regularity、缓存或 runner loophole 和无效 artifact scoring 都有具体证据支持。
- pointer-grounded detector 在 53 条 Frontier Science 人工样本上达到 0.94 precision、0.76 recall；它适合发现高置信问题，不适合证明问题不存在。

### 基于证据的合理推断

- agent benchmark 应把信息权限、生成机制、runner 状态和 grader validation 当作协议的一部分发布，而不只发布 task 与最终分数。
- “模型遵守了允许的接口”与“分数仍能表示目标能力”是两件事。协议允许检索答案时，agent 的行为可能完全 compliant，但 benchmark interpretation 仍然无效。
- protocol audit 应同时保存 exposure、engagement 和 score consequence；只做静态泄漏扫描会高估风险，只做行为检测又无法定位设计责任。
- live benchmark 和随机生成 benchmark 不是天然免污染；它们把风险从静态题目记忆转移到搜索、反馈、generator 和 runtime state。

### 待实验验证

- 在公开、随机抽样的完整 benchmark 上复现 detector，并发布所有 `D_r` evidence bundle 后，阳性率是否保持。
- 用多个不同家族的 judge、盲化 benchmark 身份和更大人工样本，测量 shared-error、prompt sensitivity 与真正的 inter-rater agreement。
- 为 15 个 benchmark 系统构造 matched repaired protocol，而不只挑五个可配对案例，估计 benchmark-level Mislead gap 分布。
- 在固定任务、模型、预算和可见信息下逐项关闭 exposure，验证 capability drift 是否由该机制因果导致。
- 对 interaction feedback 做主动 adversarial probing；当前零检出究竟是没有问题、样本不足，还是 detector 对跨轮策略不敏感。
- 将 protocol repair 纳入 longitudinal evaluation，观察 agent 能力增长后旧修复是否再次失效。

## 局限性与可信度警告

- **抽样高度不均。** 2,385 看似很大，但 cohort 从 36 到 494 条不等；若干公开 benchmark 只保留预筛的 suspicious traces。
- **67% 的分母容易误读。** Frontier Science 的 331/494 只针对 passing traces；若以全部 960 次 rollout 为分母，问题定义会不同。
- **五个 gap 不能代表总体。** `0.447–1.000` 来自五个异质配对案例，不是 15 个 benchmark 的平均 inflation interval。
- **人工校准样本小。** 53+21 条不足以覆盖七种 exposure，且缺少 annotator、agreement 和 adjudication 细节。
- **judge 依赖仍然明显。** 主审计默认 GPT-5.5，部分 batch 的 judge config/review policy 不同；LLM shared bias 与版本漂移没有系统隔离。
- **零检出不是有效性证明。** detector 的已测 recall 为 0.76，且对无工具的 memorized answer 明确不在范围内。
- **可复现性不完整。** 多个 cohort 来自内部 benchmark，论文没有给出公开 trace、audit bundle 或完整代码仓库链接。
- **必要性是强反事实。** post-hoc trace 能证明某条替代路径被使用，却不能穷尽未观察路径；最好用隔离后的 paired intervention 验证。
- **难度排序未实测。** “generator regularity 最难检测”等结论是作者判断，不应当作 detector benchmark 结果。
- **仍是 arXiv v1。** 当前没有同行评审结论；内部数据和框架命名也需等待公开 artifact 核验。

## 与仓库已有论文和主题的主动关联

- **BenchJack：** BenchJack 主动生成 exploit、证明 benchmark 可被攻破并尝试 patch；本文从真实运行证据反向归因 exposure、engagement 和 Mislead。前者回答“能不能攻击”，本文回答“这次分数是否真的被替代路径支撑”。
- **Benchmarking the Benchmarks（工具调用审计）：** 该文检查 `E(τ) != H(τ)`，即 evaluator 是否把任务结果判对；本文检查 intended capability 是否仍为得分必要条件。grader 即使准确识别“答案正确”，也可能错误归因于独立推导。
- **The Verification Horizon：** 它主张 verifier 必须与 generator 共演化；本文把失效面具体化为答案源、隐藏状态、生成规律、交互反馈和评分管线，并要求 trace-level pointer。
- **AI Agents That Matter：** 该文强调 harness、成本和 control group 会改变 agent 比较；本文进一步说明信息流与 scorer shortcut 会改变“分数测的到底是什么”。
- **Counsel：** Counsel 用人类 meta-label 检查 judge critique；本文同样要求可复核指针，但人工校准只有 74 条且流程披露更少。因此 HackDetect 自己也需要 meta-evaluation。
- **Search-Time Contamination：** 它量化 web-enabled deep-research agent 的运行时泄漏；本文把搜索答案归入更一般的 answer-source exposure，并补上实际使用及分数后果。
- **RewardHackingAgents：** 它把 evaluator tampering 与 train/test leakage 作为 agent 行为 outcome；本文提醒，即便 agent 没改 evaluator、完全按接口行动，协议也可能让高分失去能力含义。

## 与近期 AI / 评测论文的关系

- **RewardHackingAgents** — 2026-03-11，https://arxiv.org/abs/2603.11337 。它记录 evaluator tampering 和 train/test leakage，并测量 evaluator locking 成本；本文把问题扩展到合法接口中的 protocol exposure。
- **MonitoringBench** — 2026-05-10，https://arxiv.org/abs/2605.09684 。它把“能否从长轨迹中发现有害行为”做成 benchmark。本文的 pointer-grounded judge 可视为窄域 monitor，但 0.76 recall 表明 monitoring miss 会变成 false assurance。
- **BenchJack** — 2026-05-12，https://arxiv.org/abs/2605.12673 。它主动发现并利用 benchmark flaw；本文采用保守的事后 attribution。组合实验可用 BenchJack 生成 exploit，再用 HackDetect 检出并量化 gap。
- **Hack-Verifiable Environments** — 2026-05-20，https://arxiv.org/abs/2605.20744 。它把可验证环境本身作为攻击对象，说明“有自动 reward”不等于 reward 健壮；本文提供跨环境 failure-surface taxonomy。
- **SpecBench** — 2026-05-20，https://arxiv.org/abs/2605.21384 。它评测 agent 是否遵守规范；本文区分 specification compliance 与 capability validity：遵守规则仍可能走不需要目标能力的捷径。
- **Search-Time Contamination** — 2026-06-03，https://arxiv.org/abs/2606.05241 。它报告 web search 可带来最高 4% 的表现膨胀；本文的 Frontier Science gap 更大，但任务、渠道和分母不同，不能直接比较。
- **The Verification Horizon** — 2026-06-24，https://arxiv.org/abs/2606.26300 。它给出 verifier 随 policy 失效的动态命题；本文给出横截面 audit，但没有证明修复能跨代稳定。
- **Benchmarking the Benchmarks** — 2026-06-30，https://arxiv.org/abs/2607.02577 。它审计 grader–human disagreement；本文在 24 天后把“判得对不对”推进到“分数能否支持目标能力解释”。
- **HVTB** — 2026-08-22，https://arxiv.org/abs/2608.22103 。它用人工验证任务审计 benchmark validity，代表把 protocol validity 转成可复用测试任务的方向。
- **Shortcutting the Fix** — 2026-09-06，https://arxiv.org/abs/2609.06780 。它在 SWE-bench Multilingual 与 DeepSWE 中报告 45.1–82.4% / 44.2–66.1% 的 shortcut exploitation，originality prompt 后分别降至 4.0–10.7% / 1.5–7.1%；prompt mitigation 是否持久仍需复现。

近期脉络可以概括为：**从记录 reward hacking，到主动寻找 benchmark exploit，再到审计 grader 是否判对，最后把 exposure、实际使用和能力误导连成证据链。** 下一步应发布可重放的 evidence bundle，并用修复前后配对实验检验目标能力是否重新成为高分必要条件。

## 一句话判断

这篇论文最有价值的贡献是把模糊的“benchmark 被污染或 agent 在作弊”拆成 **Expose → Exploit → Mislead** 三个需要分别举证的命题，并把能力漂移落到 trace 和 score gap；但它目前更像高精度的问题发现框架，而不是完整的 benchmark 健康证明——抽样偏置、0.76 recall、少量人工校准、仅五个 paired gap 和未公开 artifact 都限制了总体结论。

## Themes

4 benchmarks · 5 reliability · 7 RL environments · agent benchmark validity · protocol audit · benchmark contamination · reward hacking · capability attribution · trace evidence · LLM-as-judge · mislead gap
