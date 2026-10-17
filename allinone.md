# Related-paper search — barrier-first cascade repair for agent memory

Search date: 2026-10-17 (Asia/Shanghai)

Window: 2025–2026, with original papers and official arXiv records preferred

Queries: `agentic memory cascade repair provenance`; `memory invalidation derived artifacts agents`; `barrier first repair agent memory`; `memory lineage repair LLM agents`

## Search coverage and reading status

The six-source paper-search process was launched against arXiv, DBLP, OpenAlex, OpenReview, Semantic Scholar and Crossref, but did not produce its merged JSON before the run timeout. No source counts or negative novelty claim are inferred from that incomplete pass. Identities, first public dates and abstract-level claims below were separately checked against official arXiv records.

The 12-page target PDF was read completely, extracted, rendered page by page, and visually checked at the formal definition, algorithm, main tables, parametric-repair result, selector frontier, robustness ablation and limitations. Adjacent papers were checked at primary pages but not all read in full. No author-linked code, event corpus or result ledger is present on the target arXiv page.

## Closest primary papers

| Paper | First public date | Distinct role | Primary source |
|---|---|---|---|
| Agentic Unlearning | 2026-02-06 | Joint parameter–memory unlearning and backflow control | https://arxiv.org/abs/2602.17692 |
| TierMem / From Lossy to Verified | 2026-02-20 | Provenance-linked summary/raw hierarchy and evidence escalation | https://arxiv.org/abs/2602.17913 |
| MEMOREPAIR | 2026-05-08 | Target: withdraw affected descendants before repair and republish a validated closure | https://arxiv.org/abs/2605.07242 |
| MemSecBench | 2026-07-29 | Persistence, external consequence and selective memory repair checkpoints | https://arxiv.org/abs/2607.27080 |
| Dependency-Guided Rollback Repair | 2026-08-11 | Failed-trace invalidation and selective replay | https://arxiv.org/abs/2608.10502 |
| StateWise | 2026-10-04 | Pre-action diagnosis and correction of persistent operational state | https://arxiv.org/abs/2610.05241 |
| DREAM | 2026-10-09 | Detection and functional forgetting of poisoned memory | https://arxiv.org/abs/2610.13267 |

## The problem shift

Deleting or correcting a source does not automatically invalidate summaries, cached outputs, prompt skills, executable procedures or neural skills derived from it. MEMOREPAIR treats this as a cascade-update problem. Its strongest conceptual move is temporal: a stale descendant should not remain available while its replacement is being built.

```text
root delete / correction / migration
        ↓ influence provenance closure
withdraw root + descendants from service
        ↓ artifact-specific recompute / regenerate / parameter repair
validate candidates in dependency order
        ↓ predecessor-closed publication
republish only a consistent successor set
```

This is a commit protocol for visible memory state, not an end-to-end detector or external-action rollback mechanism.

## Mechanism audit

The graph separates influence edges, which define repair scope, from semantic edges, which only support retrieval. A repair event supplies invalid roots, event type and replacement/migration information. Every provenance-reachable artifact is withdrawn. Descendants are assigned `recompute`, `regen`, `param` or `remove`; each candidate carries a repair operator, repaired-predecessor set, executability, value and cost.

For fixed trade-off `λ`, publication maximizes total value minus `λ` times cost, under executability and predecessor-closure constraints. This is maximum-weight closure and reduces to one `s-t` min-cut. The graph-cut algorithm is classical; the paper’s contribution is the barrier/candidate/validation contract that creates this optimization problem.

Failures remain withdrawn, as do candidates depending on them. The contract covers artifact visibility and publication. It does not reverse an already sent message, payment, permission change or data disclosure.

## What the numbers actually establish

On ToolBench and MemoryArena, all barrier-based internal policies have 0% Leak and 0% Stale-use under complete provenance. This is mainly the contract operating as specified, not a learned system discovering the cascade. The empirical selector question is how much validated publication is recovered at what operator cost.

| Benchmark / event | Repair-all publication ceiling | MEMOREPAIR publication | Cost | Difference in diagnostic ΔTask |
|---|---:|---:|---:|---:|
| ToolBench delete | 24.6% | 22.4% | 0.61 | -0.06 pt |
| ToolBench correct | 48.8% | 45.1% | 0.57 | -0.06 pt |
| ToolBench migrate | 38.7% | 36.5% | 0.66 | -0.08 pt |
| MemoryArena delete | 5.5% | 5.1% | 0.62 | -0.05 pt |
| MemoryArena correct | 35.6% | 33.1% | 0.76 | -0.07 pt |
| MemoryArena migrate | 18.4% | 17.0% | 0.64 | -0.05 pt |

Thus the reported 91.1%–94.3% refers to the fraction of the paper’s validated `Repair all` ceiling recovered, not the fraction of all lost system capability restored. `Rep.%` is over non-remove contracts. MemoryArena deletion, for example, republishes only 5.1% in absolute terms.

`ΔTask` is not native ToolBench or MemoryArena task success. It is constructed from the fraction of task-used artifacts lacking validated successors and the fraction whose stale versions remain visible. It shares state signals with Leak/Stale and cannot independently prove end-to-end task correctness.

Six cascade-unaware memory systems leave 69.8%–94.3% of affected artifacts leakable and 92.4%–99.7% of post-event actions stale across the two benchmarks. They are reference points under native semantics, not equal-contract repair baselines: they are not given the target barrier, validator or successor publication mechanism.

## The most important counter-result

Dropping only 1% of influence edges produces 17.7% Leak and 19.7% Stale-use; 5% edge drop produces 61.8% and 68.5%. The contract is therefore highly sensitive to provenance recall. Its 0% headline should always be stated with the complete-provenance premise.

The validation oracle is similarly decisive. Schema-only checking false-passes 88.6% stale use; task-regression-only false-passes 29.8%; the composed checker reaches zero on the constructed test. Passing these checks still does not prove correctness in all future contexts.

## Evidence boundary

### Directly supported

- Barrier-first withdrawal eliminates repair-interval/post-repair serving of the known cascade when roots and influence closure are complete.
- The exact selector recovers close to the exhaustive validated-publication ceiling with 0.57–0.76 normalized operator cost at `λ=0.3`.
- Store-level cascade repair and parameter unlearning address different persistence paths and combine better than either alone in the neural-skill subset.
- Small provenance omissions strongly amplify into uncovered stale descendants.

### Reasonable inference

- Visibility withdrawal and successor construction should be separate states in a safe memory lifecycle.
- A provenance graph is a safety-critical control structure, not optional observability metadata.
- Dependency-guided rollback can operate inside a barrier-first transaction, with predecessor-closed publication as the commit step.

### Untested hypotheses

- The selector remains advantageous under native benchmark task success, realistic event distributions and structural provenance errors.
- A production harness can enforce the barrier atomically across replicas, caches, tools and in-flight actions.
- Held-out future triggers agree with the paper’s validation oracle.

## Major reporting gaps

The paper does not state the number of repair events, per-event-type counts, detailed cascade-size distribution, event-generation/filtering process, or uncertainty for Tables 1–2. Only the selector frontier explicitly reports three seeds. It does not publish code or the min-cut inputs/validators. Value and cost assignment are insufficiently specified for independent reconstruction.

The diagnostic task metric, complete-provenance premise and unequal external baselines mean the evidence supports a clean repair contract and a selector result under that contract. It does not yet establish deployment prevalence, autonomous invalid-root detection, native end-to-end task recovery or reversal of external harm.

## Relationship to recent work

- **Agentic Unlearning** jointly suppresses parameter and memory pathways; MEMOREPAIR refines the store-side publication contract and can host a parameter operator.
- **TierMem** preserves links from verified summaries to immutable raw evidence; those links are infrastructure for later cascade correction when a source changes.
- **MemSecBench** supplies write/execution/consequence/repair checkpoints and a service-side E3 oracle. It can test whether zero store exposure actually prevents future external consequences.
- **Dependency-Guided Rollback Repair** decides which claims, actions and derived memories to invalidate/replay after failure. MEMOREPAIR defines what must be hidden before that computation finishes and how successors commit.
- **StateWise** attacks the missing diagnosis stage before a coding action; its repair lineage could be tested as the influence graph feeding a barrier.
- **DREAM** detects and attenuates poisoned memory. A shared snapshot, held-out triggers, availability and external-action oracle are required for comparison.

## Small falsifiable follow-up

Instrument twelve MemSecBench cases with at least two descendant layers. Compare in-place deletion, barrier plus exhaustive repair, and barrier plus min-cut plus dependency rollback. During repair, issue concurrent benign and trigger requests; after repair, start a fresh session and inspect service-side E3.

Report barrier latency, in-flight stale actions, withdrawal duration, native task completion, F1/F2, future E3 and total operator/token cost. Drop 1% of high-out-degree edges separately from 1% random edges. If min-cut does not improve the safety–availability frontier over exhaustive repair once native task success and structural missing edges are included, stop scaling models and retain only the barrier-first contract claim.

## Synthesis

MEMOREPAIR’s durable contribution is the transaction boundary: invalidate visibility first, then reconstruct and atomically republish a validated dependency closure. The min-cut selector is useful inside that boundary. The paper’s own robustness table also points to the harder research problem: complete and trustworthy influence provenance. Without it, a small logging omission can leave a large part of the stale cascade outside the barrier.
