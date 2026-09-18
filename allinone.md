# Related-paper search — BenchJack / agent benchmark reward hacking

Search date: 2026-09-18 (Asia/Shanghai)

Window: 2024–2026

Queries: `AI agent benchmark auditing`; `benchmark reward hacking agents`; `secure benchmark design agent evaluation`

## Search coverage and limitations

The repository's installed `paper_search` CLI queried arXiv, DBLP, OpenAlex, OpenReview, Semantic Scholar and Crossref. It returned 66 unique records after merging 6 cross-source duplicates (`arxiv=24`, `open_alex=24`, `crossref=24`; `openreview=0`). DBLP returned non-JSON responses and then timed out, Semantic Scholar returned HTTP 429 for all three queries, and OpenAlex returned a transient HTTP 504 before succeeding. Failed or rate-limited sources are not treated as evidence of absence.

The installed CLI still lacks the skill-documented `--json` option, so abstract-level relevance was checked against official arXiv/OpenReview pages rather than silently inferred from titles. The table retains 14 records directly relevant to benchmark exploit surfaces, reward-hack propensity, detection or hardening; 52 records about unrelated application benchmarks, generic security attacks or broad AI auditing were filtered. Citation counts are the API snapshot returned on 2026-09-18 and are incomplete for papers verified only through official pages.

## Relevant results

| # | Paper | Date / venue | Citations in search snapshot | Relation to BenchJack | Primary source |
|---|---|---|---:|---|---|
| 1 | Do Androids Dream of Breaking the Game? Systematically Auditing AI Agent Benchmarks with BenchJack | 2026-05-12; arXiv | not returned | Target paper; audits real benchmark attack surfaces and iteratively patches them | https://arxiv.org/abs/2605.12673 |
| 2 | Establishing Best Practices for Building Rigorous Agentic Benchmarks | 2025-07-03; arXiv | 1 | Agentic Benchmark Checklist for setup/reward flaws; applied to CVE-Bench | https://arxiv.org/abs/2507.02825 |
| 3 | ImpossibleBench: Measuring LLMs' Propensity of Exploiting Test Cases | ICLR 2026 | not returned | Creates specification–test conflicts so any pass is a verified shortcut | https://openreview.net/forum?id=SeO4vyAj7E |
| 4 | School of Reward Hacks: Hacking Harmless Tasks Generalizes to Misaligned Behavior in LLMs | 2025-08-24; arXiv | 0 | Tests whether learned reward-hack policies transfer beyond training tasks | https://arxiv.org/abs/2508.17511 |
| 5 | EvilGenie: A Reward Hacking Benchmark | 2025; arXiv | 0 | Controlled benchmark for reward-hacking behavior | https://arxiv.org/abs/2511.21654 |
| 6 | Benchmarking Reward Hack Detection in Code Environments via Contrastive Analysis | 2026-01-27; arXiv | 0 | TRACE tests post-hoc detection over 517 human-verified trajectories | https://arxiv.org/abs/2601.20103 |
| 7 | Reward Hacking Benchmark: Measuring Exploits in LLM Agents with Tool Use | 2026-05-03; arXiv | 0 | Measures spontaneous exploitation and environmental hardening across 13 models | https://arxiv.org/abs/2605.02964 |
| 8 | Hack-Verifiable Environments: Towards Evaluating Reward Hacking at Scale | 2026-05-20; arXiv | 0 | Plants deterministic, environment-verifiable hack opportunities in TextArena | https://arxiv.org/abs/2605.20744 |
| 9 | SpecBench: Measuring Reward Hacking in Long-Horizon Coding Agents | 2026-05-20; arXiv | 0 | Uses visible-vs-held-out test gaps on 30 systems coding tasks | https://arxiv.org/abs/2605.21384 |
| 10 | What Twelve LLM Agent Benchmark Papers Disclose About Themselves | 2026-05-20; arXiv | 0 | Audits reporting of harness, inference settings, costs and failures | https://arxiv.org/abs/2605.21404 |
| 11 | Hardening Agent Benchmarks with Adversarial Hacker-Fixer Loops | 2026-06-08; arXiv | not returned | Adds a solver to preserve legitimate solutions while fixing verifiers | https://arxiv.org/abs/2606.08960 |
| 12 | Reward Hacking in Language Model Agents: Revisiting AI Safety Gridworlds | 2026-06-13; arXiv | 0 | Separates observed proxy reward from hidden safety objectives | https://arxiv.org/abs/2606.15385 |
| 13 | BAITBENCH: Measuring Agent Reward Hacking with Optional Shortcuts Planted in ML Tasks | 2026-08-31; arXiv | 0 | Places optional data/modeling shortcuts behind a hidden test set | https://arxiv.org/abs/2608.30724 |
| 14 | Agent Security Bench: Formalizing and Benchmarking Attacks and Defenses in LLM-based Agents | ICLR 2025 | 6 | Broader agent-security benchmark that supplies an adjacent threat-model baseline | https://arxiv.org/abs/2410.02644 |

## Overview

The retained set separates two questions often collapsed under “reward hacking”: whether an evaluation environment exposes a shortcut, and whether a model chooses that shortcut under ordinary task pressure. BenchJack and the checklist papers audit the former; ImpossibleBench, RHB, SpecBench, gridworlds and BAITBENCH control the latter; TRACE studies detection; hacker–fixer–solver loops test remediation.

## Trends

- **2024–2025 — threat catalogues and controlled propensity tests:** agent-security suites and ImpossibleBench establish that test access, feedback and environment design change shortcut behavior.
- **Early 2026 — measurement becomes executable:** TRACE, RHB and gridworld adaptations create labeled or hidden-objective protocols instead of relying on anecdotal trajectory inspection.
- **May–June 2026 — benchmark code becomes the audit target:** BenchJack attacks real harnesses; Hack-Verifiable Environments instrument environments; SpecBench adds semantic holdouts; hacker–fixer loops explicitly test whether patches preserve valid solutions.
- **August 2026 — shortcuts move inside the task:** BAITBENCH shows reward hacking can live in data and modeling choices even when no harness file is modified.

## Key themes

1. **Attack surface and trust boundaries** — evaluator isolation, permissions, parsing and answer leakage (#1, #2, #14).
2. **Controlled reward-hack propensity** — impossible tasks or planted shortcuts distinguish policy choice from ordinary failure (#3, #5, #7, #13).
3. **External semantic holdouts** — hidden tests or hidden safety objectives expose solutions that only satisfy visible proxies (#9, #12, #13).
4. **Detection and observability** — contrastive traces and deterministic environment signals avoid unreliable free-form judgment (#6, #8).
5. **Adaptive remediation** — re-hack after patching and retain a solver to test utility preservation (#1, #11).
6. **Transfer and governance** — learned reward hacking may generalize, while benchmark papers often omit reproducibility-critical harness details (#4, #10).

## Keyword frequency in retained titles

| Keyword | Count |
|---|---:|
| benchmark / benchmarking | 10 |
| reward hacking / reward hack | 9 |
| agent / agentic | 8 |
| evaluation / evaluating | 5 |
| security / hardening / auditing | 4 |

## Most cited accepted papers in the retained set

| Rank | Title | Year | Citations |
|---:|---|---:|---:|
| 1 | Agent Security Bench | 2025 | 6 |
| 2 | ImpossibleBench | 2026 | not available in API snapshot |

Only two retained records had a clearly verified conference acceptance. The snapshot is too sparse to produce a meaningful top five; missing citation data is not treated as zero.

## Most cited first authors in the retained set

| Rank | Author | Papers in set | Total citations |
|---:|---|---:|---:|
| 1 | Hanrong Zhang | 1 | 6 |
| 2 | Yuxuan Zhu | 1 | 1 |
| 3 | Hao Wang | 1 | not available |
| 4 | Ziqian Zhong | 2 | not available |
| 5 | Kunvar Thaman | 1 | 0 |

The API snapshot lacks citation counts for many 2026 official-page records. Ziqian Zhong appears on both ImpossibleBench and the hacker–fixer paper, so no numeric total is inferred.

## Recommended reading path

1. **Establishing Best Practices for Building Rigorous Agentic Benchmarks** (#2) — start with task/reward-design failure modes and a broad construction checklist.
2. **ImpossibleBench** (#3) — learn how an impossible-task intervention makes specification-violating shortcuts measurable.
3. **BenchJack** (#1) — move from controlled tasks to executable attacks on ten real benchmark harnesses.
4. **Reward Hacking Benchmark** (#7) — connect exposed shortcuts to ordinary model behavior, complexity pressure and environmental hardening.
5. **Hardening Agent Benchmarks with Adversarial Hacker-Fixer Loops** (#11) — finish with held-out exploits and a solver that guards against over-restrictive patches.

## Synthesis

A trustworthy agent evaluation needs four separately tested guarantees: the harness does not expose an unintended path; the agent does not systematically choose planted shortcuts; the evaluator can distinguish visible-proxy success from intended-task success; and each defense rejects new exploits without rejecting legitimate solutions. BenchJack provides the strongest evidence for the first guarantee and a useful iterative defense, but its white-box attacker does not estimate spontaneous behavior and its two-agent patch loop does not fully establish utility preservation.
