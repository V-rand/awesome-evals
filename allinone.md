# Related-paper search — security mechanisms in coding-agent harnesses

Search date: 2026-10-12 (Asia/Shanghai)

Window: 2026, prioritizing primary papers and official project pages

Queries: `coding agent harness security benchmark`; `agent harness permission security utility`; `coding agent sandbox allowlist denylist`; `AI agent harness attack surface`

## Search coverage and limitations

The installed six-source `paper_search` workflow returned 142 unique records after merging 40 cross-source duplicates: arXiv 48, OpenAlex 48, OpenReview 38 and Crossref 48. DBLP was blocked by anti-bot handling and Semantic Scholar returned HTTP 429. Those failures are missing coverage, not evidence that adjacent work does not exist.

The target paper was read in full, all 23 PDF pages were rendered, and its mechanism definitions, 400-cell rating protocol, task table, metric definition, result tables, cases and limitations were inspected. Dates and headline claims for the closest papers were separately checked against arXiv or official project pages. The official HSB page says tasks and utility checks are public; the full harness dataset, rating records, trial artifacts and analysis scripts are still being prepared.

## Relevant results

| # | Paper | First public date | What it contributes | Primary source |
|---:|---|---|---|---|
| 1 | SkillGuard | 2026-06-02 | Permission-bearing skills governed across context influence and action side effects | https://arxiv.org/abs/2606.03024 |
| 2 | One Goal, Many Commands | 2026-06-14 | Systematic command-denylist bypass search across 1,709 real-world denylists | https://arxiv.org/abs/2606.15549 |
| 3 | Local LLM Agents as Vulnerable Runtimes | 2026-06-19 | Static audit of agent runtime components and implementation-level weaknesses | https://arxiv.org/abs/2606.21071 |
| 4 | Harness Engineering | 2026-07-15 | Source-code anatomy of eleven coding-agent harnesses and their safety controls | https://arxiv.org/abs/2609.00006 |
| 5 | HarnessRisk | 2026-08-18 | Lifecycle benchmark for utility, attack success, persistence and detection | https://arxiv.org/abs/2608.17597 |
| 6 | When Context Gets Root | 2026-08-27 | Instruction privilege escalation caused by harness context assembly | https://arxiv.org/abs/2608.27299 |
| 7 | What’s in Your Agent’s Context? | 2026-09-01 | MessageRole and Cross-Scope context privilege escalation in 12 harnesses | https://arxiv.org/abs/2609.01222 |
| 8 | Reward Hacking Challenges Oversight of Autonomous Research Agents | 2026-09-23 | Static and adaptive oversight failures when agents control evaluation evidence | https://arxiv.org/abs/2609.28614 |
| 9 | CheatBench | 2026-09-28 | Cross-domain behavior under implicit honest-work norms and actionable shortcuts | https://arxiv.org/abs/2609.36308 |
| 10 | SideKernel | 2026-10-01 | Usable microVM sandbox for local macOS coding agents | https://arxiv.org/abs/2610.02456 |
| 11 | HarnessSecurity-Bench | 2026-10-06 | Target paper; native mechanism taxonomy, evidence matrix and ON/OFF runtime evaluation | https://arxiv.org/abs/2610.07639 |

## Overview

HarnessSecurity-Bench separates three questions that are often collapsed. RQ1 derives ten mechanism categories from source and official materials. RQ2 assigns evidence-backed `D/C/O/A/U` ratings to 400 harness–mechanism cells. RQ3 runs nine mechanisms across 23 adversarially augmented tasks, six harnesses and 2,500 trials, with deterministic utility and attack-effect oracles outside the workspace.

The central result is not “sandboxing works.” Different controls fail in different ways. Network isolation and read-only mode greatly reduce attack effects but also remove capabilities legitimate tasks need. Command allow/deny lists, path restrictions and MCP permissions preserve more utility, yet alternative interpreters or retained tools can still reach the protected operation. Project trust looks costly only when a startup refusal is pooled with four harnesses that retain full utility.

The metric boundary is important: Utility and ASR are proportions of oracle checks, not binary per-trial success rates. Different mechanisms have different tasks, attack goals, denominators and paired harnesses. Their overall deltas describe their own benchmark slices, not a comparable product leaderboard.

## Trends

- **From model-only safety to model–harness systems:** the same model can face different tools, permissions, context and execution boundaries.
- **From mechanism presence to behavioral effect:** documentation-backed implementation does not establish runtime protection.
- **From content filtering to operation mediation:** prompt-injection screening leaves attacks framed as ordinary workflow prerequisites; the protected action still needs an independent gate.
- **From command names to reachability:** a denied executable can be reached through an allowed Python interpreter, wrapper or child process.
- **From ASR alone to paired utility:** blocking every action is secure only in the trivial sense; legitimate completion must remain measurable.
- **From one aggregate to a mechanism matrix:** task requirements, baseline ASR and product semantics determine whether the same control helps.
- **From hidden defaults to verifiable policy:** closed-source unknowns are evidence gaps; providers should expose versioned effective settings and coverage.
- **From isolated controls to defense composition:** real deployments combine project trust, approval, path, network and tool policies, an interaction HSB does not yet test.

## Key distinctions

1. **Implementation evidence versus protection** — RQ2 ratings concern source/docs/CLI evidence; RQ3 concerns execution outcomes.
2. **Unknown versus absent** — `U` means insufficient/conflicting evidence, not non-implementation.
3. **Default-on versus effective** — `D/C` says a mechanism is active by default, not that its mediation is complete.
4. **Risk capability versus defense** — auto-approve removes confirmation barriers and is intentionally included as a contrast.
5. **Oracle pass versus trial success** — one trial may satisfy all utility checks and all attack checks simultaneously.
6. **Aggregate utility gain versus capability gain** — blocking an attack can prevent it from corrupting a legitimate deliverable.
7. **Restriction versus refusal** — Gemini PT’s 0% utility/ASR comes from not starting, not selective safe execution.
8. **Single-setting observation versus causal effect** — `D/A/U` cells lack an ON/OFF counterfactual.
9. **Attack stress test versus prevalence** — malicious resources are deliberately planted; ASR is not a deployment base rate.
10. **One path blocked versus complete mediation** — protection requires gating all paths to the operation, not one command syntax.

## Numerical audit

- 400 cells = 40 harnesses × 10 mechanisms; 205 implemented, 83 absent, 112 unknown.
- Implemented cells split into 109 opt-in and 96 active by default (64 configurable-off, 32 default-enforced).
- Closed-source unknowns are 69/130 (53.1%); open-source unknowns are 43/270 (15.9%).
- RQ3 covers 23 tasks, nine mechanisms, six harnesses and 2,500 trials; AL is excluded because it records rather than directly changes execution.
- AA changes Utility 77.1%→95.4% and ASR 29.2%→95.6% under the specified approval reviewer.
- NI changes Utility 90.9%→66.4% and ASR 57.1%→0.8%; 161/162 lost utility passes are required network interactions.
- RO changes Utility 69.8%→35.8% and ASR 8.3%→0.8%; the baseline attack rate is already low.
- CAL/CDL/PR/MCP change ASR by −48.1/−40.4/−48.1/−28.1pp with −1.5/+5.2/+0.3/+0.5pp Utility changes.
- PT’s aggregate −20pp Utility comes entirely from Gemini CLI refusing all 40 untrusted-workspace trials.
- Only gptme has a paired PIF comparison: ASR −3.3pp and Utility −1.9pp; other PIF cells cannot estimate an ON/OFF effect.
- Token totals omit unavailable conditions; the project page’s 2,218.20M is a recorded-total figure, not guaranteed full cost.

## Recommended reading path

1. **HarnessSecurity-Bench** (#11) — learn the mechanism taxonomy, rating/effect split and paired outcome design.
2. **Harness Engineering / Local LLM Agents** (#4/#3) — understand the implementation layer being classified and audited.
3. **One Goal, Many Commands** (#2) — see why command-string policy is structurally fragile.
4. **Context Gets Root / What’s in Your Context** (#6/#7) — connect context assembly to privilege escalation before tool execution.
5. **SkillGuard** (#1) — examine a principal/permission model that links context-bearing artifacts to action rights.
6. **HarnessRisk** (#5) — compare mechanism-centric evaluation with lifecycle-centric persistence/detection coverage.
7. **SideKernel** (#10) — add deployment usability and sandbox adoption to efficacy/utility.
8. **Reward Hacking Oversight / CheatBench** (#8/#9) — connect enforceable boundaries to behavior and reviewer limitations.

## Synthesis

The reusable evaluation object is an operation-reachability ledger:

```text
untrusted resource enters context
  → agent proposes an action
  → native policy allows, asks, or denies one route
  → alternative commands/tools/interpreters are enumerated
  → external oracle records the protected operation and its consequence
  → legitimate completion is evaluated in the same trial
```

A decisive follow-up should create semantically equivalent shell, Python, Node, symlink, child-process and MCP paths to the same protected object, then compare native single controls, recommended multi-control configuration and an operation-level reference monitor. Report per-trial “any path reached the operation” together with complete legitimate success, approvals, latency and token cost.

If the reference monitor does not outperform native composition on path coverage, stop claiming a new authorization layer is needed. If it only lowers attack success by destroying legitimate completion, call it capability removal. If bypasses occur only through one interpreter, narrow the claim to that enforcement gap rather than generalizing to all harness security.
