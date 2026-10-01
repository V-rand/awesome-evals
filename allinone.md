# Related-paper search — tool-calling evaluator validity and harness variance

Search date: 2026-10-01 (Asia/Shanghai)

Window: 2024–2026

Queries: `tool calling benchmark validity audit`; `agent benchmark evaluator human disagreement`; `tool use benchmark harness variance`; `benchmark audit agent evaluation`

## Search coverage and limitations

The installed `paper_search` CLI was invoked across arXiv, DBLP, OpenAlex, OpenReview, Semantic Scholar and Crossref. DBLP returned anti-bot, proxy and HTTP 429 failures; the remaining multi-source run did not complete after several minutes and was stopped safely, without producing a JSON result. This is missing coverage, not evidence that related work is absent.

The search was therefore supplemented with primary-page web search and direct inspection of official arXiv/OpenReview records. The retained set focuses on evaluator–human disagreement, trace evidence, harness-induced variance, deterministic versus LLM scoring, and protocol-level benchmark validity. Dates and central claims were checked against primary pages; citation counts are omitted because bibliographic coverage was incomplete.

The target paper itself requires extra bibliographic caution: its v1 reference list contains placeholder arXiv IDs and misidentifies LiveMCPBench as `2506.07982`, which is actually τ²-Bench. The corrected official records are included below.

## Relevant results

| # | Paper | First public date / venue | Relation to the target paper | Primary source |
|---:|---|---|---|---|
| 1 | Benchmarking the Benchmarks: A Validity Audit of Tool-Calling Evaluation | 2026-06-30; arXiv v1 | Target paper; audits evaluator–human mismatch across four tool-calling benchmarks | https://arxiv.org/abs/2607.02577 |
| 2 | τ²-Bench: Evaluating Conversational Agents in a Dual-Control Environment | 2025-06; arXiv | One audited deterministic/stateful benchmark; official ID corrects the target paper's placeholder | https://arxiv.org/abs/2506.07982 |
| 3 | LiveMCPBench: Can Agents Navigate an Ocean of MCP Tools? | 2025-08-03; arXiv / ICLR 2026 submission | One audited LLM-judge benchmark; official ID corrects the target paper's miscitation | https://arxiv.org/abs/2508.01780 |
| 4 | MCP-Atlas: A Large-Scale Benchmark for Tool-Use Competency with Real MCP Servers | 2026-01-31; arXiv | One audited claims-based benchmark; reports 78% human agreement in its own setting | https://arxiv.org/abs/2602.00933 |
| 5 | Claw-Eval: Towards Trustworthy Evaluation of Autonomous Agents | 2026-04-07; arXiv v3 | Uses traces, audit logs and environment snapshots to expose failures hidden by outcome-only grading | https://arxiv.org/abs/2604.06132 |
| 6 | Auditing Automated Evaluation, Error Propagation, and Runtime Mitigation in Tool-Using Language Agents | 2026-04-17; arXiv | Human-calibrates substring and LLM judges; closely parallels the target's brittle-assertion finding | https://arxiv.org/abs/2604.16706 |
| 7 | Do Androids Dream of Breaking the Game? / BenchJack | 2026-05-12; arXiv | Actively exploits benchmark flaws rather than only auditing label disagreement | https://arxiv.org/abs/2605.12673 |
| 8 | Harness-Bench: Measuring Harness Effects across Models in Realistic Agent Workflows | 2026-05-27; arXiv | Makes model–harness configuration, not the base model alone, the evaluation unit | https://arxiv.org/abs/2605.27922 |
| 9 | The Verification Horizon: No Silver Bullet for Coding Agent Rewards | 2026-06-24; arXiv v2 | Frames evaluator maintenance as generator–verifier co-evolution | https://arxiv.org/abs/2606.26300 |
| 10 | Counsel: A Meta-Evaluation Dataset for Agentic Tasks | 2026-06-19; arXiv | Human meta-evaluates judge critiques, separating correct localization from reasoning quality | https://arxiv.org/abs/2606.21627 |
| 11 | Do Agent Benchmarks Measure Capability? Protocol Validity in the Age of Agentic AI | 2026-07-24; arXiv | Extends audit from label mismatch to `Expose → Exploit → Mislead` and quantified score inflation | https://arxiv.org/abs/2607.22368 |

## Overview

The retained literature separates three validity questions. **Evaluator validity** asks whether the grader agrees with human task-success judgments. **Harness validity** asks whether the execution layer, retries, context and permissions change what is being measured. **Protocol validity** asks whether the intended capability remains necessary for obtaining the score, or whether an exposed shortcut can replace it. The target paper is strongest on the first question and supplies infrastructure for the second; later protocol-audit work makes the third question explicit.

## Trends

- **2024–2025 — realistic tool use expands faster than evaluator validation:** benchmarks add multi-turn state, live MCP servers and large tool ecosystems, often relying on exact state checks or scalable LLM judges.
- **Early 2026 — trajectory evidence becomes first-class:** Claw-Eval and related work retain traces, environment snapshots and audit logs so completion, safety and robustness can be checked separately.
- **Human calibration reveals both rigid and permissive failure:** substring/state matchers reject valid alternatives, while LLM judges and incomplete rubrics pass fluent but unexecuted outcomes.
- **Harness configuration becomes part of measured capability:** 5,194 Harness-Bench trajectories show that model rankings cannot be interpreted independently of execution configuration.
- **Single-run leaderboards lose epistemic weight:** the target paper's 23 LiveMCPBench reruns span 18.9pp end to end, motivating distributions, version pinning and repeated-run comparison.
- **Auditing moves toward causal attribution:** Protocol Validity asks not only whether a score is wrong, but what exposure was available, whether the agent used it and how much the score was inflated.

## Key themes

1. **Outcome truth versus evaluator verdict** — a benchmark label is a measurement output, not ground truth by definition.
2. **Fact gates versus qualitative judgment** — state changes should be checked deterministically where possible; LLM judges should not override failed factual requirements.
3. **Repair as a separate capability** — first-attempt success and bounded recovery should not be merged.
4. **Versioned evidence** — task, tool state, runner, rubric, judge, retry policy and artifacts all belong in the result record.
5. **Matched comparison** — evaluator designs must be compared on the same tasks, trajectories and evidence before claiming superiority.
6. **Meta-evaluation closure** — human calibration itself needs agreement statistics, adjudication rules and publicly auditable samples.

## Recommended reading path

1. **LiveMCPBench** (#3) and **MCP-Atlas** (#4) — understand why dynamic tool ecosystems turn to LLM or claims-based scoring.
2. **Claw-Eval** (#5) — see how independent evidence channels improve trajectory-aware evaluation.
3. **AgentProp-Bench** (#6) — compare substring heuristics and LLM judges against human labels.
4. **Harness-Bench** (#8) — treat the harness as part of the evaluated system.
5. **Benchmarking the Benchmarks** (#1) — inspect concrete false positives, false negatives and repeated-run instability.
6. **Protocol Validity** (#11) — move from disagreement diagnosis to evidence-backed shortcut attribution and score distortion.

## Synthesis

A trustworthy tool-agent result should be represented as a versioned evidence bundle rather than a scalar: task specification, allowed tools and permissions, initial/final state, complete trajectory, submitted artifact, deterministic predicates, qualitative rubric, judge/version, retry policy, repeated-run distribution and human calibration sample. The target paper points in this direction with Tool-Veritas and Harness Lab, but its next decisive test is matched and reproducible: freeze a shared trajectory set, compare the original evaluator, deterministic-only gates, unrestricted LLM judging and deterministic-first restricted judging; report false-positive/false-negative rates, judge-only variance, cost and inter-annotator agreement, then release the exact artifacts.
