# Related-paper search — provenance-aware tiered memory for agents

Search date: 2026-10-18 (Asia/Shanghai)

Window: 2025–2026, with original papers and official pages preferred

Queries: `provenance aware agent memory evidence sufficiency`; `tiered memory raw logs summaries agents`; `write before query memory compression agents`

## Search coverage and reading status

The six-source process was launched against arXiv, DBLP, OpenAlex, OpenReview, Semantic Scholar and Crossref, but did not return merged JSON before the timeout and was stopped. No source counts or negative novelty claim are inferred from this incomplete pass. Identities, dates and abstract-level claims below were separately checked against official arXiv records.

The 22-page target arXiv v1 PDF was read completely, extracted, rendered page by page, and visually checked at the architecture, main tables, router/provenance ablations, consolidation, error analysis and prompts. The official repository was audited at commit `7a9b71f6389a9f8caa83c47aff5191921c6a82ca`. Adjacent papers were checked at primary pages but not all read in full.

GitHub now labels TierMem as accepted at COLM 2026 under the updated title *Balancing Compressed Memory and Raw Evidence for Long-Horizon Agent Memory*. This numerical audit remains tied to the earlier arXiv v1: arXiv has not been revised and OpenReview was blocked by browser verification during this run.

## Closest primary papers

| Paper | First public date | Distinct role | Primary source |
|---|---|---|---|
| ACON | 2025-10-01 | Optimize query-agnostic compression from failure analysis | https://arxiv.org/abs/2510.00615 |
| Goal-Directed Search / SUMER | 2025-11-20 | Learn to search uncompressed memory for a known goal | https://arxiv.org/abs/2511.21726 |
| Recursive Language Models | 2025-12-31 | Treat long input as an external environment | https://arxiv.org/abs/2512.24601 |
| TierMem | 2026-02-20 | Target: route between source-linked summaries and immutable raw evidence | https://arxiv.org/abs/2602.17913 |
| MEMOREPAIR | 2026-05-08 | Withdraw descendants when a source becomes invalid | https://arxiv.org/abs/2605.07242 |
| Dependency-Guided Rollback Repair | 2026-08-11 | Invalidate failed derivations and selectively replay | https://arxiv.org/abs/2608.10502 |

## The problem shift

The future query is unknown when an agent compresses its history. A summary may omit the allergy, version, date, exception or exact tool output later needed. Once absent from the summary representation, better semantic retrieval cannot recover it.

TierMem reframes memory access as inference-time evidence allocation:

```text
immutable raw pages
      ↓ deterministic source pointers
retrievable summaries / facts
      ↓ query + retrieved evidence
sufficiency router ── enough ──→ answer cheaply
      │
      └─ insufficient → linked raw pages → bounded extra search → answer
                                              ↓
                                   verified fact write-back
```

The summary is a cache/navigation layer; the raw log remains the authoritative evidence store.

## Mechanism audit

Tier-2 stores canonical interaction pages of roughly 1,000 tokens with stable IDs. Tier-1 stores summaries/embeddings plus pointers. A Qwen3-0.6B router selects summary answer or raw escalation. Escalation first opens linked pages, then performs at most three semantic-plus-BM25 retrieve/integrate/plan rounds.

The model extracts facts and evidence quotes, but deterministic substring/keyword matching assigns page provenance. Verified findings can be added or update existing summary units; UPDATE inherits old and new pointers.

Router labels come from paired summary/raw executions. Summary-correct cases are S; summary-wrong/raw-correct cases are R; both-fail and raw-fail cases are removed. SFT uses GPT-5 teacher traces, followed by GRPO with correctness, cost, unnecessary-escalation and format rewards. This trains a recoverability-conditioned router, not a detector of every memory failure.

## What the numbers establish

| Benchmark / path | Accuracy | QA input | Router input | Latency |
|---|---:|---:|---:|---:|
| LoCoMo summary | 0.755 | 260 | — | 1.24 s |
| LoCoMo raw | 0.873 | 7,398 | — | 17.18 s |
| LoCoMo routed | 0.851 | 3,396 | 584 | 6.76 s |
| LongMemEval summary | 0.678 | 380 | — | 5.37 s |
| LongMemEval raw | 0.808 | 7,298 | — | 13.56 s |
| LongMemEval routed | 0.752 | 4,235 | 660 | 10.04 s |

The LoCoMo abstract's 54.1% token reduction compares QA input only. Adding router input gives 3,980 vs 7,398, about 46.2%. LongMemEval drops about 32.9% rather than 42.0% when router input is included. Routed TierMem is a cost–accuracy compromise: it improves over summary-only but remains below raw-only on both datasets.

The learned SFT+GRPO router reaches 0.851 on LoCoMo, matching GPT-4.1-mini routing. GPT has higher hard-case recall/F1 and slightly lower latency; the learned router has fewer counterfactual regressions. The evidence supports a small router matching final accuracy, not dominating the larger router.

## Provenance-specific evidence

Linked raw-page warm starts improve routed LoCoMo accuracy from 0.836 to 0.851 and escalated-subset accuracy from 77.5% to 81.7%. However, the settings change routing on 234/1,540 questions. Where both escalate, Linked-correct/No-linked-wrong occurs 31 times and the reverse 25 times: a net six-case advantage. Search depth and token use are similar.

The pointer has a modest measured QA effect. Its claimed audit value is plausible but not directly established by citation precision/recall, unsupported-fact rate or provenance-corruption tests. Those are the missing measurements if provenance is the scientific claim.

## Consolidation boundary

Three epochs replay the same fixed questions, freezing memory within each epoch and updating between epochs. Input tokens fall while accuracy stays near 0.85. This shows repeated-workload amortization, not transfer to new questions; repeated queries can turn write-back into evaluation-specific cache. Future work should keep the history but replace later epochs with unseen questions and report store growth, conflict and provenance inheritance.

## Evidence boundary

### Directly supported

- Raw evidence recovers details lost by write-time summaries on both benchmarks.
- A learned sufficiency router improves summary-only accuracy at lower query cost than raw-only.
- Source-linked warm starts produce a small gain over unlinked search.
- Repeated-question consolidation reduces later token use with roughly stable accuracy.

### Reasonable inference

- Compact memory should be treated as an index/cache rather than the only fact store.
- False cache hits are more harmful than unnecessary escalation because they remove the recovery path.
- Page provenance can anchor later audit/repair only if downstream consumption dependencies are also logged.

### Untested hypotheses

- Write-back improves unseen future questions rather than only repeated questions.
- Pointers remain correct through multi-source synthesis and repeated UPDATE.
- TierMem improves real tool-using trajectories with state changes and side effects.
- Routing remains safe when raw evidence is absent or retrieval/generation cannot repair the answer.

## Reproducibility audit

The repository contains the linked-view implementation, modified Mem0, three benchmark runners, router offline-data/SFT/GRPO preparation, reward code and online evaluation. It links a Hugging Face Safetensors router and documents Qdrant/API/vLLM dependencies.

It does not include the main result ledger or router training records in git; the Hugging Face model card is empty, and training/serving commands are environment-dependent. The release improves inspectability and supports a reproduction attempt, but a checkout alone cannot regenerate the paper tables.

## Relationship to recent work

- **ACON** improves the compressor through natural-language optimization. TierMem instead retains a recovery path for details any query-agnostic compressor misses.
- **SUMER** searches uncompressed memory after the goal is known. TierMem uses summaries first and searches raw data after a sufficiency decision.
- **RLM** treats long input as an external environment for programmatic inspection, but does not itself define persistent source-linked memory or verified cross-query write-back.
- **MEMOREPAIR** asks what to withdraw when a source becomes invalid. TierMem asks how to reopen a still-valid source when a summary is insufficient: invalidation recall versus evidence recall.
- **Dependency-Guided Rollback Repair** needs evidence-to-action consumption edges in addition to TierMem's summary-to-page pointers.

## Small falsifiable follow-up

Use 120 real coding-agent histories. Generate summaries before revealing questions, then ask held-out questions answerable from summaries, linked pages, or non-linked pages. Compare summary-only, raw-search-only, TierMem-linked and TierMem with shuffled links under the same generator and total token budget.

Report accuracy, false cache hits, escalation precision/recall, citation precision/recall, unsupported facts, P95 latency and total input including router overhead. In round two, ask different questions about the same history after write-back. If linked pointers do not beat shuffled pointers under fixed routing/retrieval budgets, provenance is not the performance mechanism. If write-back helps only repeated questions, classify it as query caching rather than memory learning.

## Synthesis

TierMem's durable contribution is the evidence hierarchy: fast lossy summaries linked to an immutable source, plus a query-time decision about whether cheap evidence is sufficient. It demonstrates a useful cost–accuracy frontier, but the headline needs denominator care, raw-only remains more accurate, and provenance's controlled QA gain is modest. The decisive next evaluation is whether links are correct, false cache hits are controlled, and recovered facts help unseen questions without contaminating long-term memory.
