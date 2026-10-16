# Related-paper search — post-failure memory recovery for agents

Search date: 2026-10-16 (Asia/Shanghai)

Window: 2025–2026, with original papers and official arXiv records preferred

Queries: `agent memory rollback repair provenance dependency graph`; `post failure recovery memory augmented agents`; `selective memory repair derived state replay`; `agent memory fault correction rollback`

## Search coverage and limitations

The installed multi-source search was launched against arXiv, DBLP, OpenAlex, OpenReview, Semantic Scholar and Crossref, but did not produce its merged result file before the run timeout; no source counts or negative novelty claim are inferred from that incomplete pass. Paper identity, first public date and abstract-level claims below were instead checked against official arXiv records. This is a coverage limitation, not evidence that related work is absent.

The target paper was read in full: its 19-page PDF was extracted and all pages rendered; the formal setup, algorithm, primary tables, ablations, per-domain/fault results, backbone transfer, LongMemEval-V2 adaptation and appendices were inspected. Adjacent papers were checked at their primary pages, not all read in full. The target arXiv record provides no direct author-linked official code/data/replay repository, so the reported tables were not independently reproduced.

## Closest primary papers

| Paper | First public date | Distinct recovery question | Primary source |
|---|---|---|---|
| MemoRepair | 2026-05-08 | Can invalidated descendants be hidden before cascade repair and only validated successors republished? | https://arxiv.org/abs/2605.07242 |
| MemSecBench | 2026-07-29 | Can a poisoned memory be followed through external consequence, then removed without deleting specified benign memory? | https://arxiv.org/abs/2607.27080 |
| Dependency-Guided Rollback Repair | 2026-08-11 | Target: which claims, actions and derived memories lose support, and which computation should be replayed? | https://arxiv.org/abs/2608.10502 |
| HarnessRisk | 2026-08-18 | How do persistence, detection and incident recovery behave across the harness lifecycle? | https://arxiv.org/abs/2608.17597 |
| StateWise | 2026-10-04 | Can invalid persistent operational state be diagnosed and corrected before a coding-agent action? | https://arxiv.org/abs/2610.05241 |
| DREAM | 2026-10-09 | Can poisoned memory be functionally forgotten through reactivation, anomaly diagnosis and attenuation? | https://arxiv.org/abs/2610.13267 |

## Recovery is not one metric

```text
detect fault → block further exposure → invalidate unsupported descendants
             → repair active memory → replay necessary computation
             → regenerate answer → retest future triggers
             → compensate reversible external effects
```

Different papers intervene at different edges. MemoRepair focuses on visibility during cascade repair; StateWise repairs before action; Dependency-Guided Rollback starts after a failed execution; DREAM suppresses poisoned memory; MemSecBench audits persistence, consequence and selective deletion. Their headline percentages are not comparable without the same fault oracle, trace, tool semantics, persistence store and re-trigger protocol.

## Target mechanism

The target assumes exact faulty-memory IDs and an instrumented runtime. It builds a typed graph over memory, claims, plans, actions, observations, answers and mutations, using explicit provenance rather than temporal adjacency. Forward tracing creates an over-approximate affected set. An independent-support pass preserves candidates whose complete content still has trusted support outside the affected set. Faulty memories are deleted, unsupported derived memories quarantined, unsupported trace nodes invalidated, and answer-relevant executable closure replayed in original order. The final answer is always regenerated.

The graph contracts the rollback surface but does not undo irreversible actions. Replay needs resettable, idempotent or compensating tools. Missing provenance and imperfect fault diagnosis are outside the primary evaluation.

## Experimental audit

The controlled set has 150 conditioned cases across shopping, travel and customer support, with poisoned, stale, wrong-user and summary-drift faults. Cases are retained only if injection changes the answer and has a downstream effect. All methods receive exact fault IDs; clean traces and labels remain evaluation-only. Primary metrics are answer recovery, recurrence after rerunning the final input, fault removal, benign preservation, claim-invalidation F1, replay ratio and LLM calls.

| Method | Recovery | Recurrence | Fault removal | Benign preservation | Claim F1 | Replay | Calls |
|---|---:|---:|---:|---:|---:|---:|---:|
| LLM-judge repair | 77.3% | **12.1%** | 100% | 100% | **0.946** | 0.217 | 9.80 |
| AgentTrace-style | 60.7% | 34.1% | 0% | 99.7% | 0.853 | 0.091 | 4.69 |
| Dependency-guided | **85.3%** | 26.6% | 100% | 100% | 0.566 | 0.123 | 5.70 |

The target leads immediate recovery, but not recurrence or claim localization. Its clearest trade-off appears in the ablation: replacing selective replay with broad replay changes recovery only from 85.3% to 84.0%, while recurrence improves from 26.6% to 7.1%; replay ratio rises from 0.123 to 0.755 and calls from 5.70 to 24.01. Selective replay is therefore an efficiency mechanism with a visible recurrence cost.

Stale faults expose the main weakness: 94.1% immediate recovery but 78.1% recurrence. Across model backbones, the target beats the closest LLM-judge baseline on GPT-4o and Qwen3.6-27B, but not on Gemini-3.6-Flash (73.3% versus 74.0%). The 50-case selected LongMemEval-V2 stress subset gives 68.0% recovery; this is not a full-benchmark result.

## Evidence levels

### Directly supported by the paper

- Under exact fault IDs, complete runtime provenance and controlled tools, dependency-guided rollback reaches 85.3% immediate recovery on the conditioned 150-case set.
- It removes all diagnosed faulty memories and preserves all labelled benign memories in that set, but recurrence remains 26.6%.
- Selective replay reduces cost sharply; full replay produces a much lower recurrence rate.
- The method cannot undo irreversible external side effects and does not evaluate fault detection.

### Reasonable inferences

- Recovery evaluation should separately report current answer, persistent state, descendant invalidation, future recurrence and external-world compensation.
- Typed provenance can reduce unnecessary rollback, but incomplete logging is likely a safety-critical failure mode.
- Version, temporal scope and authority should be first-class memory relations, because stale records recur even after strong immediate recovery.

### Hypotheses requiring experiments

- The advantage may collapse under modest fault-ID false negatives or missing graph edges.
- Barrier-first quarantine plus dependency-guided replay may reduce exposure without requiring full replay.
- A longer natural re-trigger suite may reverse method rankings obtained from rerunning only the final input.

## Literature boundary

- **MemoRepair** supplies a repair contract: withdraw affected descendants first, then publish only validated predecessor-closed successors. It addresses exposure during repair more directly than the target; the target gives a concrete failed-trace replay planner.
- **MemSecBench** gives evidence checkpoints from memory persistence to external consequence and F1/F2 selective repair. The target provides the missing descendant/action repair mechanism, but needs MemSecBench-style post-repair E3 retesting.
- **StateWise** performs record-level counterfactual diagnosis, read-only verification and intent clarification before action. Its 93.3% coding-case headline is not comparable to post-failure recovery; together they define prevention and residual recovery.
- **DREAM** combines detection and functional forgetting, whereas the target assumes diagnosed IDs and repairs derived computation. A shared poisoned snapshot and multi-session recurrence test is needed.
- **HarnessRisk/HarnessSafe** broaden the carrier set beyond memory. The target graph should eventually span skills, configuration, tools and shared artifacts before it can support whole-harness recovery claims.

## Small falsifiable follow-up

Use twelve MemSecBench cases with deterministic E3 service oracles, at least two benign memories and observable derived state. From the same verified W2 snapshot compare original Forget, dependency rollback, and dependency rollback plus barrier-first quarantine. End the repair session and use three new natural tasks to re-trigger the fault. Report removal, benign preservation, answer recovery, future E3, exposure window and cost separately.

Then remove 10% and 30% of provenance edges and hide one faulty memory ID. If the method no longer separates from broad replay on future E3 and legitimate-task completion, stop scaling the model matrix and narrow the claim to cost optimization under complete provenance.

## Synthesis

Dependency-guided rollback turns memory repair from deleting a record into repairing a computation and its persistent descendants. That is the conceptual advance. The numerical record is deliberately mixed: best immediate recovery and exact labelled-state preservation, but weaker recurrence and claim localization than the best LLM-judge baseline, with particularly severe stale-state recurrence. A trustworthy recovery benchmark must keep these outcomes separate and add imperfect diagnosis, incomplete provenance and external-effect compensation before making deployment claims.
