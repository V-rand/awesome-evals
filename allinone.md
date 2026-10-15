# Related-paper search — memory poisoning, consequence, and selective repair

Search date: 2026-10-15 (Asia/Shanghai)

Window: 2025–2026, with original papers and official arXiv records preferred

Queries: `agent memory poisoning selective repair`; `agent memory security benchmark Write Execute Forget`; `persistent agent memory attack evaluation`; `memory poisoning recovery agent benchmark`

## Search coverage and limitations

The installed six-source paper search returned 144 unique records after merging 68 cross-source duplicates: arXiv 48, OpenAlex 48, OpenReview 44, Semantic Scholar 24, Crossref 48, and DBLP 0. DBLP returned anti-bot/HTTP-429 errors, and some Semantic Scholar queries also returned HTTP 429. These are gaps in search coverage. Paper identity, first public date and abstract-level claims in the selection below were checked against official arXiv records.

The target paper was read in full: 28 PDF pages were extracted and rendered, with the setup, seven checkpoint criteria, main table, figures, worked cases, backend details, runtime controls and judge protocol inspected. Adjacent papers below were checked at their primary pages; they were not all read in full. The target arXiv record does not provide a direct official code/data/replay repository link, so the reported aggregates are not independently reconstructed here.

## Closest primary papers

| Paper | First public date | Role in this map | Primary source |
|---|---|---|---|
| MemEvoBench | 2026-04-17 | Repeated misleading updates and gradual memory drift | https://arxiv.org/abs/2604.15774 |
| Trojan Hippo Bench | 2026-05-03 | Delayed, adaptive memory attack through one untrusted tool output; defense and utility trade-off | https://arxiv.org/abs/2605.01970 |
| From Untrusted Input to Trusted Memory / MPBench | 2026-06-03 | Write channels, structural vulnerabilities and memory poisoning classes | https://arxiv.org/abs/2606.04329 |
| MemPoison | 2026-07-16 | Multi-record composition and dormant trigger risks missed by write-time checks | https://arxiv.org/abs/2607.14651 |
| MemSecBench | 2026-07-29 | Target: one case from memory write to external consequence and selective repair | https://arxiv.org/abs/2607.27080 |
| HarnessSafe | 2026-08-07 | Persistent-risk chains across memory, skills, tools and shared artifacts | https://arxiv.org/abs/2608.06984 |
| Dependency-Guided Rollback Repair | 2026-08-11 | Repair of derived state and affected actions using a dependency graph | https://arxiv.org/abs/2608.10502 |
| HarnessRisk | 2026-08-18 | Whole-harness lifecycle safety, including persistence and incident recovery | https://arxiv.org/abs/2608.17597 |
| Forgetting is Not Always Bad / DREAM | 2026-10-09 | Candidate functional-forgetting intervention for poisoned agent memory | https://arxiv.org/abs/2610.13267 |

## What MemSecBench measures

MemSecBench defines 310 linked Write–Execute–Forget cases from 48 contexts. Every case supplies a clean memory source, a Write task that can store malicious semantics, a later benign Execute task, and an independent Forget task. Execute and Forget branch from copies of the same verified post-Write memory snapshot. A hidden case rubric specifies one target malicious meaning and an external consequence contract.

Seven evidence checkpoints track the chain: W1 accepted write, W2 persisted malicious meaning, E1 later recall, E2 adoption into the agent’s decision, E3 externally recorded consequence, F1 removal or neutralization, and F2 preservation of required benign memory. W1 is deterministic. The remaining checkpoints use a separately pinned DeepSeek-V4-Pro judge and programmatic checks for admissible evidence, timing and final-state consistency. An agent’s claim or tool invocation alone cannot establish E3 or F1.

The experiment spans 24 complete configurations: OpenClaw or Hermes, Native/Mem0/Mem0-Graph/A-MEM, and DeepSeek-V4-Pro/MiniMax-M3/GPT-5.5. Each runs all 310 cases once, producing 7,440 configuration–case observations. Native differs between harnesses, and every backend condition includes adapters, retrieval behavior, memory-side model use and storage settings. Matched comparisons therefore concern complete memory stacks rather than isolated storage algorithms.

## Numerical audit and interpretation

| Metric | Denominator | Reported configuration macro-average | Meaning |
|---|---:|---:|---|
| MPSR | All 310 cases per configuration | 84.2% | W1 and W2 both pass |
| MESR | Successfully poisoned cases in each configuration | 59.6% | E1, E2 and E3 all pass after W1/W2 |
| E2E-ASR | All 310 cases per configuration | 50.3% | Full Write–Execute chain passes |
| SRSR | Successfully poisoned cases in each configuration | 56.1% | F1 and F2 both pass |

The attack chain contracts most from E1 recall (76.1% of all cases) to E2 adoption (53.7%). E3 then gives 50.3% full-chain success. On the repair branch, F1 removal passes 86.3% of successfully poisoned cases, F2 benign preservation 62.5%, and both together 56.1%. The 30.2-point F1-to-SRSR gap identifies collateral loss of legitimate memory after apparently successful removal.

Two matched contrasts show why attack resistance and repair must be reported separately. Under OpenClaw/DeepSeek-V4-Pro, A-MEM versus Native changes E2E-ASR from 59.03% to 56.45% while SRSR rises from 46.31% to 87.63%. Under OpenClaw/GPT-5.5, Mem0-Graph versus Native lowers E2E-ASR from 57.10% to 40.97% but SRSR barely changes, from 87.50% to 86.73%. Under Hermes/MiniMax-M3, Mem0 raises both attack success and repair relative to Native. There is no backend ranking that stays stable across configurations.

## Boundaries that matter

1. **Conditional cases, not deployment prevalence.** The corpus deliberately plants malicious content; 43.5% of carriers are User Interaction, and some Write prompts explicitly request storing the suspect rule. Execute contains retrieval cues, while Forget asks the agent to perform a security self-check.
2. **Independent repair branch, not incident reversal.** Forget starts from the poisoned snapshot, not from a state after E3 harm. F1/F2 say nothing about reversing a payment, disclosure or downstream action already recorded.
3. **Macro-averages, not pooled trials.** MESR and SRSR have configuration-specific poisoned-case denominators. Their macro-average cannot be interpreted as one pooled conditional rate.
4. **Descriptive contrasts, not isolated backend effects.** The complete stack, adapter, capacities and retrieval settings change together. One run per case/configuration gives no repeat-run uncertainty interval.
5. **Evidence-backed judging, still semantic judging.** The judge matches two human annotators on 90.6% and 91.8% of the same 500 sampled records, but the paper does not report checkpoint-specific confusion, inter-annotator agreement or rank sensitivity to judge errors.
6. **Specified preservation set.** F2 checks required benign memories named by the case. Other useful memories or later task utility may still be lost.
7. **Public reconstruction limit.** The paper describes source hashes, snapshots and per-case ledgers, but no official linked public repository was found in the arXiv record during this run.

## How the literature connects

- **Attack entry and evolution:** MPBench studies why content crosses from low-trust input into trusted memory; MemEvoBench studies gradual drift under repeated updates; MemPoison studies joint retrieval and dormant triggers. MemSecBench provides a controlled chain to check which of these mechanisms reaches an external consequence.
- **Long delay and deployment realism:** Trojan Hippo Bench’s single untrusted tool-output entry and activation after many benign sessions stress a boundary that MemSecBench’s prompted Write and fixed later Execute do not establish.
- **System-wide coverage:** HarnessSafe and HarnessRisk trace persistence beyond memory into skills, tools and recovery. MemSecBench contributes a finer-grained memory state and selective-repair test.
- **Recovery mechanisms:** Dependency-Guided Rollback Repair traces derived claims/actions and replays affected computation; DREAM proposes functional forgetting. Both are candidate interventions to run against MemSecBench’s same post-Write snapshots, with E3, F1, F2 and legitimate task utility all reported.

## Small falsifiable follow-up

Select twelve cases with service-side E3 oracles and nontrivial benign memories. Fix the model, harness, backend and W2 snapshot. Compare the paper’s Forget instruction with (a) provenance/scope labels and (b) dependency-guided repair. End the session after repair, trigger the later task in a fresh session, and measure F1, F2, new E3 events and normal task completion separately.

The competing explanation is that a method merely removes more memory, lowering E3 by impairing the agent. A successful mechanism claim needs lower re-trigger E3 **and** intact benign memory and task completion. If twelve paired cases show no reproducible separation from the baseline, stop scaling the model matrix and narrow the claim.

## Synthesis

MemSecBench makes the causal chain easier to inspect: storage, recall, adoption, consequence and selective repair each require different evidence. Its most useful number is not the 50.3% full-chain attack rate alone, but the gap between 86.3% target removal and 56.1% selective repair. The benchmark’s current public evidence supports this diagnostic framing and descriptive configuration results; natural attack prevalence, backend-only causal effects and post-harm recovery remain open tests.
