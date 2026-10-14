# Related-paper search — lifecycle safety of agent harnesses

Search date: 2026-10-14 (Asia/Shanghai)

Window: 2025–2026, prioritizing original papers, official project pages and public artifacts

Queries: `agent harness lifecycle safety benchmark`; `coding agent persistence detection security`; `agent harness configuration attack benchmark`; `AI agent incident recovery security`

## Search coverage and limitations

The installed six-source search workflow returned 237 unique records after merging 61 cross-source duplicate records: arXiv 60, OpenAlex 60, OpenReview 58, Semantic Scholar 60 and Crossref 60. DBLP failed behind anti-bot/TLS handling; OpenReview and Semantic Scholar also required HTTP-429 retries. These failures are coverage gaps, not negative evidence.

HarnessRisk was read in full and all 20 PDF pages were rendered. Its definitions, 128 cases, three-turn protocol, metric validation, tables, figures, appendix prompts and public artifacts were inspected. Dates and headline claims below were checked against arXiv or an official project page. The public GitHub repository and Hugging Face dataset expose the harness adapters, evaluators and 128 case specifications, but—at the inspected 2026-08-19 repository commit—do not expose raw trajectories, the validity ledger or table-generation outputs needed to reconstruct every reported aggregate.

## Relevant results

| # | Paper | First public date | Relation to HarnessRisk | Primary source |
|---:|---|---|---|---|
| 1 | ClawSafety | 2026-04-01 | Broad safety benchmark for autonomous desktop agents; motivates system-level rather than answer-only evaluation | https://arxiv.org/abs/2604.01438 |
| 2 | Your Agent, Their Asset | 2026-04-06 | Studies the security externalities created when agents can access user assets | https://arxiv.org/abs/2604.04759 |
| 3 | SafeHarness | 2026-04-15 | Treats harness design as a safety intervention and measures the resulting behavior | https://arxiv.org/abs/2604.13630 |
| 4 | Auditing Agent Harness Safety | 2026-05-14 | Audits safety properties at the harness layer instead of assigning all failures to the base model | https://arxiv.org/abs/2605.14271 |
| 5 | AgentS4D | 2026-07-29 | Evaluates security across agent deployment surfaces and complements lifecycle-phase coverage | https://arxiv.org/abs/2607.27294 |
| 6 | MemSecBench | 2026-07-29 | Isolates long-term-memory poisoning and persistence, one of HarnessRisk’s six phases | https://arxiv.org/abs/2607.27080 |
| 7 | HarnessSafe | 2026-08-07 | Evaluates safety of harness-mediated operation and provides a nearby system-level framing | https://arxiv.org/abs/2608.06984 |
| 8 | HarnessRisk | 2026-08-18 | Target paper; 128 cases over six lifecycle phases and four trajectory-level outcomes | https://arxiv.org/abs/2608.17597 |
| 9 | HarnessSecurity-Bench | 2026-10-06 | Tests native security mechanisms with deterministic task and attack-effect checks | https://arxiv.org/abs/2610.07639 |
| 10 | EvoRiskBench | 2026-10-02 | Adds evolving and adaptive risks that a fixed lifecycle suite may not retain over time | https://arxiv.org/abs/2610.03153 |

## What HarnessRisk changes

HarnessRisk changes the evaluation unit from a single prompt or tool call to the **model–harness configuration over a lifecycle trajectory**. Its 128 cases cover six phases: Harness Configuration and Capability Extension (22 cases each), then Runtime Operation, State Persistence, Action Control and Incident Recovery (21 each). Every case has three owner turns, and an untrusted artifact can influence configuration, become a skill, enter memory, redirect a high-impact action or survive attempted recovery.

The benchmark evaluates 14 configurations across OpenClaw, Nanobot and Hermes. Four models run on all three harnesses; GPT-5.5 and Claude Opus 4.7 run only on OpenClaw. A GPT-5.4 judge assigns four binary trajectory metrics: Utility, Attack Success Rate, Persistence and Detection. On sampled calibration sets, judge agreement ranges from 84.3% to 92.5%, with Cohen’s κ from 0.65 to 0.83.

Reported ASR spans 12.6%–80.9% while Utility spans 75.0%–97.6%. This is the paper’s most useful result: successful completion does not imply safe completion. For GLM-5.2, ASR is 54.7% on OpenClaw, 12.6% on Nanobot and 23.8% on Hermes. That 4.3× range establishes configuration sensitivity, but not a harness-only causal effect: the systems differ jointly in prompts, tools, state, authorization and adapters.

Detection also is not equivalent to prevention. Across the 12 configurations used for the correlation analysis, Detection and ASR have an unadjusted Pearson correlation of −0.708 (p=.010) and Spearman correlation of −0.769 (p=.003); after adding model and harness indicators, the reported significance weakens to p=.101. MiniMax M3 on OpenClaw detects 97.9% yet still has 31.2% ASR; GLM-5.2 detects 92.2% yet has 54.7% ASR.

## Key distinctions and audit points

1. **Lifecycle coverage versus causal phase effects.** The six phases organize cases; they are not homogeneous interventions that identify a phase’s causal contribution.
2. **Configuration comparison versus harness comparison.** Cross-harness results compare complete deployments, not a single isolated harness variable.
3. **Detection versus containment.** Recognizing suspicious content does not prove that the agent revoked credentials, removed extensions, repaired memory or prevented side effects.
4. **Persistence versus observable persistence.** The evaluator can score only state exported by a harness adapter; hidden state can be missed.
5. **Owner authority versus provenance laundering.** Later owner turns sometimes explicitly approve or restate suspicious material. Some measured failures therefore combine indirect attack susceptibility with a trusted-input intervention that launders provenance.
6. **Process isolation versus a security boundary.** The repository’s sandbox creates per-run process state; it is not a kernel namespace, chroot or firewall. An agent with arbitrary command execution may still reach host or network resources.
7. **Judge estimate versus deterministic fact.** All four main metrics come from GPT-5.4. Deterministic rules and human labels calibrate samples but do not generate the full headline tables.
8. **Valid-run estimate versus deployment rate.** Provider errors, empty or abnormal responses and judge parse failures are excluded without a per-configuration ledger, so rates are conditional on retained runs.
9. **Low ASR versus safe competence.** Refusal, tool failure and incomplete task execution can all suppress ASR; Utility must be read beside it.
10. **Three-seed variation versus inferential uncertainty.** The reported standard deviation over three seeds is not a standard error and does not absorb provider drift.

## Relation to adjacent evaluation lines

- **HarnessSecurity-Bench** isolates native mechanism toggles and uses deterministic outside-workspace oracles; HarnessRisk broadens the story to the whole lifecycle but relies more heavily on an LLM judge. They answer different questions and should be combined rather than ranked.
- **MemSecBench** gives deeper resolution on memory poisoning; HarnessRisk shows how memory failure connects to configuration, action and recovery. A useful follow-up should preserve that cross-phase causal chain.
- **SafeHarness, HarnessSafe and Auditing Agent Harness Safety** share the system-level unit of analysis. HarnessRisk’s distinctive contribution is the six-phase responsibility map and the joint Utility/ASR/Persistence/Detection view.
- **EvoRiskBench** highlights temporal adaptation. HarnessRisk’s public static cases can establish a reproducible baseline, but not whether a defense remains effective after attackers learn its evaluator or policy.

## Recommended reading path

1. **HarnessRisk** — understand the lifecycle taxonomy, three-turn case protocol and four outcomes.
2. **HarnessSecurity-Bench** — inspect mechanism-level counterfactuals and deterministic attack-effect checks.
3. **MemSecBench** — deepen the State Persistence phase and distinguish memory write, later retrieval and action-time use.
4. **SafeHarness / HarnessSafe / Auditing Agent Harness Safety** — compare harness-level interventions and audit assumptions.
5. **EvoRiskBench** — test whether fixed-case conclusions survive adaptive pressure and system updates.

## Falsifiable follow-up

A narrow extension is a **provenance-carrying recovery benchmark**. Hold the task, artifact and model fixed; vary only whether provenance and revocation dependencies survive the transition from ingestion to memory to recovery. Measure attack side effect, malicious-state persistence, evidence preservation and verified rollback.

- **Competing explanation:** the apparent gain comes from adding another warning string, not from carrying provenance or dependency state.
- **Discriminating intervention:** keep warning text identical while ablating provenance labels, dependency links and recovery actions separately.
- **Falsifier:** if provenance/dependency ablation does not change side-effect or rollback rates beyond a prespecified interval, the mechanism claim fails.
- **Stop rule:** do not scale to more models until at least one matched configuration shows a reproducible separation between “detected” and “fully remediated.”

## Synthesis

HarnessRisk’s strongest contribution is not a new safety score. It makes the lifecycle responsibility chain measurable and shows why Utility, Detection and safety cannot substitute for one another. Its main empirical limitation is equally important: the public evidence currently supports a benchmark design and reported aggregates, but not a fully reconstructable run ledger, and its cross-harness numbers should be read as configuration-level observations rather than causal rankings.
