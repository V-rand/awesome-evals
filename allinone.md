# Related-paper search — meta-evaluation of agent judges

Search date: 2026-09-29 (Asia/Shanghai)

Window: 2024–2026

Queries: `agent trajectory judge meta evaluation`; `LLM judge critique quality agent trajectories`; `agent evaluator error localization reasoning`

## Search coverage and limitations

The repository's `paper_search` CLI queried arXiv, DBLP, OpenAlex, OpenReview, Semantic Scholar and Crossref, returning 88 unique records after merging 16 cross-source duplicates. Per-source hits were `arxiv=24`, `open_alex=24`, `openreview=24`, `crossref=24`, `semantic_scholar=8`; DBLP returned no usable records because its anti-bot challenge blocked the API. Semantic Scholar returned HTTP 429 during later queries and OpenReview also rate-limited one request, so failed sources are not treated as evidence of absence. The complete machine-readable result was saved during review and the retained papers below were checked against official arXiv, OpenReview, ACL or dataset pages.

The broad query also surfaced many unrelated multi-agent orchestration, domain application and generic survey papers. This report retains work that directly evaluates agent trajectories, localizes agent errors, meta-evaluates judges, or uses critique/reward reasoning. Citation counts are intentionally omitted because source failures made them incomplete and non-comparable.

## Relevant results

| # | Paper | First public date / venue | Direct relation to Counsel | Primary source |
|---:|---|---|---|---|
| 1 | Counsel: A Meta-Evaluation Dataset for Agentic Tasks | 2026-06-19; arXiv v1 | Target paper; 1,131 human meta-annotations separate error location from critique reasoning | https://arxiv.org/abs/2606.21627 |
| 2 | AgentRewardBench: Evaluating Automatic Evaluations of Web Agent Trajectories | 2025-04-11; arXiv | Meta-evaluates whole-trajectory success, side effects and repetitiveness across web-agent benchmarks | https://arxiv.org/abs/2504.08942 |
| 3 | TRAIL: Trace Reasoning and Agentic Issue Localization | 2025-05-13; arXiv | Direct error-localization benchmark on 148 human-annotated agent traces | https://arxiv.org/abs/2505.08638 |
| 4 | Agent-as-a-Judge: Evaluate Agents with Agents | 2024-10-14; arXiv | Uses agentic evaluators and hierarchical requirements to provide process-level feedback | https://arxiv.org/abs/2410.10934 |
| 5 | AgentDiagnose: An Open Toolkit for Diagnosing LLM Agent Trajectories | EMNLP 2025 System Demonstrations | Operational tooling for trajectory diagnosis; complements Counsel's labeled critique-quality data | https://aclanthology.org/2025.emnlp-demos.15/ |
| 6 | Where LLM Agents Fail and How They can Learn From Failures | 2025-09-29; arXiv | AgentErrorBench and AgentDebug target root-cause localization and corrective feedback | https://arxiv.org/abs/2509.25370 |
| 7 | Reward Reasoning Model | 2025-05-20; arXiv | Uses test-time reasoning to improve reward judgments; Counsel tests whether added reasoning is actually sound | https://arxiv.org/abs/2505.14674 |
| 8 | JudgeBench: A Benchmark for Evaluating LLM-Based Judges | ICLR 2025 | General judge meta-evaluation benchmark; mostly static response comparison rather than agent trajectories | https://openreview.net/forum?id=G0dksFayVq |
| 9 | Benchmarking LLM-as-a-Judge for Long-Form Output Evaluation | 2026-06-01; EMNLP 2026 | LongJudgeBench finds cross-scenario instability on long outputs, neighboring long-context evidence | https://arxiv.org/abs/2606.01629 |
| 10 | From Confident Closing to Silent Failure: Characterizing False Success in LLM Agents | 2026-06-01; arXiv | Uses environment state to show outcome judges miss false success; complements process-critique audit | https://arxiv.org/abs/2606.09863 |

## Overview

The literature is converging on a layered view of agent evaluation. Outcome benchmarks ask whether the task succeeded; trajectory benchmarks ask where the process failed; critique meta-evaluation asks whether an evaluator's diagnosis is itself correct; deployment monitoring asks whether the diagnosis remains reliable under online information constraints. Counsel occupies the third layer and contributes supervision that can be used to audit or train the evaluator rather than the task agent directly.

## Trends

- **2024 — richer evaluators:** Agent-as-a-Judge extends final-score LLM judging with tools, hierarchical requirements and intermediate feedback.
- **2025 — trajectory ground truth and localization:** AgentRewardBench, TRAIL, AgentDiagnose and AgentErrorBench move from answer pairs to complete agent runs, error taxonomies and root-cause diagnosis.
- **2025 — reasoning inside the evaluator:** Reward Reasoning Model treats extra inference compute as a way to improve reward accuracy, creating a need to verify whether longer critiques are causally correct.
- **2026 — meta-evaluation becomes explicit:** Counsel directly labels judge critiques; LongJudgeBench measures scenario instability; False Success checks evaluator predictions against environment state rather than surface claims.
- **The key metric shifts from agreement to utility:** research increasingly distinguishes final label accuracy, localization, explanatory soundness, calibration, and whether feedback improves future task success.

## Key themes

1. **Outcome versus process** — task success does not identify the step or mechanism of failure.
2. **Localization versus explanation** — finding the right span and explaining its error are separable abilities.
3. **Precision versus recall** — reviewing only flagged errors makes critique precision scalable but leaves shared judge blind spots unmeasured.
4. **Online versus retrospective information** — guardrail judges see only the current prefix; human auditors often see the completed trajectory.
5. **Meta-judge training** — human critique-quality labels can filter, rank or improve judge outputs, but cross-domain benefits remain to be established.
6. **Environment-grounded audit** — database and tool state can expose confident but false success that text-only judges miss.

## Recommended reading path

1. **Agent-as-a-Judge** (#4) — understand why agent evaluation needs intermediate, requirement-aware feedback.
2. **AgentRewardBench** (#2) — see how whole-trajectory automatic evaluators diverge from expert success labels.
3. **TRAIL** (#3) — examine the difficulty of localizing and classifying errors in long traces.
4. **Counsel** (#1) — separate critique location from critique reasoning and inspect the precision-only annotation boundary.
5. **From Confident Closing to Silent Failure** (#10) — close the loop with environment-grounded evidence of outcome-monitor failure.

## Synthesis

A trustworthy agent evaluator needs at least four independently tested properties: it must detect failures, localize them, explain them correctly, and ground its conclusions in observable environment state. Counsel supplies unusually useful labels for the middle two properties, but its sampling design intentionally does not establish the first. The next decisive experiment is therefore a matched evaluation that annotates a stratified sample of both flagged and unflagged spans, holds online information constant between human and model judges, and tests whether a Counsel-trained meta-judge improves held-out agent success rather than only critique classification.
