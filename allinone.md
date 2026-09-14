# Related-paper search — AI Agents That Matter

Search date: 2026-09-14 (Asia/Shanghai)  
Query window: 2024–2026  
Queries: `AI agent evaluation cost holdout overfitting`; `agent benchmark generalization leaderboard reliability`; `AI Agents That Matter`

## Search coverage and limitations

The local `paper_search` CLI was invoked across its configured sources, but the installed script did not implement the skill-documented `--json` option and the fallback multi-source run produced no results before it was stopped. Therefore, no empty source is treated as evidence of absence. The records below were independently verified against primary arXiv/OpenReview pages; this is a focused relation audit, not an exhaustive systematic review.

## Verified results

| # | Paper | First posted / venue | Direct relationship | Primary source |
|---|---|---|---|---|
| 1 | AI Agents That Matter | 2024-07-01; TMLR 2025 | Target paper: cost control, matched holdouts, downstream-vs-model evaluation, reproducibility | https://arxiv.org/abs/2407.01502 |
| 2 | AgentRewardBench: Evaluating Automatic Evaluations of Web Agent Trajectories | 2025-04-12 | Tests evaluator validity against expert-reviewed agent trajectories | https://arxiv.org/abs/2504.08942 |
| 3 | Holistic Agent Leaderboard: The Missing Infrastructure for AI Agent Evaluation | 2025-10-13; ICLR 2026 | Implements standardized, cost-aware, trajectory-logged evaluation across models, scaffolds and benchmarks | https://arxiv.org/abs/2510.11977 |
| 4 | ImpossibleBench: Measuring LLMs' Propensity of Exploiting Test Cases | 2025-10 | Turns benchmark shortcut exploitation into a directly measurable outcome | https://arxiv.org/abs/2510.20270 |
| 5 | Search-Time Contamination in Deep Research Agents: Measuring Performance Inflation in Public Benchmark Evaluation | 2026-06-03; under review | Measures inference-time retrieval of benchmark metadata, question context and answers | https://arxiv.org/abs/2606.05241 |

## Synthesis

The line of work shifts from diagnosing confounds to instrumenting them. *AI Agents That Matter* shows that accuracy-only leaderboards confound architecture, inference budget, benchmark shortcuts and harness choices. AgentRewardBench validates the grader rather than assuming it. HAL standardizes the execution layer and preserves complete traces. ImpossibleBench creates tasks where passing certifies test exploitation. Search-Time Contamination then isolates a new inference-time channel that ordinary training-contamination audits miss.

The robust conclusion is that an agent evaluation needs at least four separately inspectable layers: task/holdout validity, execution and scaffold standardization, evaluator validity, and trajectory-level integrity. It remains a hypothesis—not an established result—that satisfying all four will predict real deployment utility; that requires prospective deployment or matched human-in-the-loop studies.

## Reading path

1. *AI Agents That Matter* for the original measurement decomposition.
2. *AgentRewardBench* for grader validation.
3. *Holistic Agent Leaderboard* for standardized infrastructure at scale.
4. *ImpossibleBench* for falsifiable shortcut measurement.
5. *Search-Time Contamination* for web-enabled agents' inference-time leakage.
