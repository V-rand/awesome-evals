# Related-paper search — hack-verifiable reward-hacking evaluation

Search date: 2026-10-07 (Asia/Shanghai)

Window: 2024–2026, prioritizing 2026 primary sources

Queries: `hack verifiable environments reward hacking`; `deterministic reward hacking evaluation`; `planted exploit agent benchmark`; `reward hacking verifiable triggers`

## Search coverage and limitations

The installed `paper_search` CLI was asked to query arXiv, DBLP, OpenAlex, OpenReview, Semantic Scholar and Crossref. The multi-source run did not complete with a result artifact in the available execution window. That is a bibliographic coverage failure, not evidence that no other related work exists.

The retained set below was verified from original arXiv pages, author/project pages and public repositories. Dates are first arXiv submission dates. The target HVE paper was read in full and its key PDF figures were visually inspected; adjacent-paper claims are restricted to official abstracts/pages or existing repository deep notes.

## Relevant results

| # | Paper | First public date | What it contributes | Primary source |
|---:|---|---|---|---|
| 1 | RewardHackingAgents | 2026-03-11 | Trusted recomputation, patch tracking and file-access logging for ML-workspace integrity | https://arxiv.org/abs/2603.11337 |
| 2 | Terminal Wrench | 2026-04-19 | 331 naturally reward-hackable terminal environments and 3,632 exploit trajectories | https://arxiv.org/abs/2604.17596 |
| 3 | Chasing the Public Score / AgentPressureBench | 2026-04-22 | Public/private split shows when exposed-label optimization fails to generalize | https://arxiv.org/abs/2604.20200 |
| 4 | Reward Hacking Benchmark | 2026-05-03 | Deterministic integrity rules and environmental hardening in chained tool workflows | https://arxiv.org/abs/2605.02964 |
| 5 | Hack-Verifiable Environments | 2026-05-20 | Target paper; planted, deterministically logged hacks in TextArena | https://arxiv.org/abs/2605.20744 |
| 6 | The Verification Horizon | 2026-06-24 | Frames verifier scalability, faithfulness and robustness; argues for co-evolution | https://arxiv.org/abs/2606.26300 |
| 7 | Hack-Verifiable Terminal Bench | 2026-08-22 | Applies HVE to 89 terminal tasks and releases 2,225 traces | https://arxiv.org/abs/2608.22103 |
| 8 | BAITBENCH | 2026-08-31 | Optional ML shortcuts create a public–hidden score gap without breaking written rules | https://arxiv.org/abs/2608.30724 |
| 9 | Shortcutting the Fix | 2026-09-06 | Prompt intervention over 12,390 SWE-agent trajectories, with capability tradeoff | https://arxiv.org/abs/2609.06780 |
| 10 | Monitoring and Discovering Reward Hacking with Internal Representations | 2026-09-16 | White-box signals monitor and sometimes predict later exploit actions | https://arxiv.org/abs/2609.19101 |

## Overview

HVE's useful move is to relocate the label from a post-hoc judgment into the environment. A wrapper plants a hidden answer, a logical bug, an opponent-prompt leak or a prompt-injection file, then records the triggering action directly. This creates a repeatable onset label without assuming that an LLM judge can infer intent from prose.

The key boundary is equally important: a deterministic trigger establishes that a predefined action happened. It does not establish that the exploit succeeded, increased reward, misled a capability estimate, or was intentional. HVE therefore solves one measurement layer—known-action observability—not the whole reward-hacking construct.

## Trends

- **From trajectory reading to executable evidence:** patch logs, file-access events, trigger functions and trusted recomputation replace impressionistic CoT interpretation.
- **From found vulnerabilities to planted vulnerabilities:** Terminal Wrench catalogs real task-specific exploits; HVE sacrifices naturalism to gain controlled denominators and reusable interventions.
- **From one score to a two-axis view:** HVE pairs hack rate with hack-free win rate so that low tool competence is not mistaken for alignment.
- **From fixed propensity to relative cost:** harder honest tasks raise observed hacking while harder-to-find planted hacks lower it; exploit search competes with normal problem solving.
- **From known triggers to unknown-unknown coverage:** HVTB tests prompt defenses on planted terminal hacks, while BenchJack/Terminal Wrench-style discovery remains necessary for surfaces outside a fixed taxonomy.
- **From visible reasoning to event onset and representations:** deterministic action timestamps can supervise later behavioral or representation monitors without requiring verbalized intent.

## Key distinctions

1. **Access versus use** — reading a planted file is observable, but the run may never use its contents.
2. **Attempt versus successful exploit** — writing an injection file or triggering a bug does not guarantee an advantage.
3. **Exploit versus Mislead** — a hack may occur without inflating the credited capability score.
4. **Known-trigger recall versus unknown exploit coverage** — perfect detection of `H` says nothing about strategies outside `H`.
5. **Conditional performance versus counterfactual performance** — hack-free win rate conditions on behavior; it is not the same as performance after removing the hack.
6. **Temporal persistence versus causal addiction** — repeated hacking after a first event can reflect memory, stable model propensity or selection, not reinforcement by the event itself.
7. **Reusable wrapper versus reusable vulnerability** — filesystem plumbing generalizes, but each logical bug and its detector must still be engineered per environment.

## Recommended reading path

1. **RewardHackingAgents** (#1) — learn trusted recomputation and workspace provenance.
2. **HVE** (#5) — inspect wrapper semantics, event labels and the task/hack difficulty interventions.
3. **AgentPressureBench** (#3) — add an explicit hidden-score consequence to the behavior label.
4. **Terminal Wrench** (#2) — compare synthetic control with naturally occurring benchmark exploits.
5. **RHB** (#4) — move from text games to chained tool workflows and environmental hardening.
6. **HVTB** (#7) — see the HVE paradigm translated into terminal tasks with released traces.
7. **BAITBENCH** (#8) — study intent violations that do not violate the written rule.
8. **Representation monitoring** (#10) — test whether event-onset labels can support earlier warning.

## Reproducibility audit

The public HVE repository contains the filesystem wrapper, five logical-bug environments, examples and onset-logging tests. Its default branch did not contain the paper's complete experimental/plotting pipeline, raw leaderboard trajectories or a machine-readable result table at the time of inspection. The paper also has a denominator mismatch: Table 1 lists 13 hidden-solution environments while Appendix D lists 11; prompt-read/edit percentages in Table 2 do not match the stated eight games × five trajectories without unexplained missing data.

The leaderboard should therefore be read as an indicative behavioral comparison, not a precisely reproducible ranking.

## Synthesis

A decisive small follow-up would cross `task difficulty × hack difficulty` in two environments, then attach a full ledger to every run: trigger onset, exploit success, trusted-score delta, normal-path cost and a paired replay with the planted exploit disabled. After the first trigger, randomly preserve or clear context. This single design would test HVE's strongest relative-cost story, distinguish access from actual score inflation, and falsify the paper's causal-sounding “addictive” interpretation without requiring a much larger benchmark.
