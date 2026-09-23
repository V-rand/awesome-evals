# Related-paper search — RewardBench 2 / reward model evaluation

Search date: 2026-09-23 (Asia/Shanghai)

Window: 2024–2026

Queries: `reward model evaluation preference benchmark`; `LLM judge reward model robustness`; `verifier evaluation reasoning reward models`

## Search coverage and limitations

The repository's installed `paper_search` CLI queried arXiv, DBLP, OpenAlex, OpenReview, Semantic Scholar and Crossref. It returned 40 unique records after merging 8 cross-source duplicates (`open_alex=24`, `crossref=24`; `arxiv=0`, `dblp=0`, `openreview=0`, `semantic_scholar=0`). arXiv returned HTTP 406 for all three queries, DBLP returned non-JSON responses, Semantic Scholar returned HTTP 429 after bounded retries, and OpenAlex had a transient HTTP 504 before succeeding. Failed or rate-limited sources are not treated as evidence of absence.

The installed CLI still lacks the skill-documented `--json` option, so it could not provide the complete abstract-bearing recovery file. Abstract-level relevance and dates for the retained set were verified on official arXiv, ICLR proceedings, GitHub or Hugging Face pages. The table retains 12 records directly relevant to general reward-model accuracy, downstream validity, robustness, inference-time scaling or agent/verifier extensions; unrelated medical, aviation, generic preference-learning and non-LLM reward papers were excluded. Citation counts are the incomplete API snapshot returned on 2026-09-23 and should not be interpreted as current totals.

## Relevant results

| # | Paper | Date / venue | Citations in search snapshot | Relation to RewardBench 2 | Primary source |
|---|---|---|---:|---|---|
| 1 | RewardBench: Evaluating Reward Models for Language Modeling | 2024-03-20; NeurIPS 2024 D&B | 11 | Pairwise predecessor and shared benchmark/code base | https://arxiv.org/abs/2403.13787 |
| 2 | RewardBench 2: Advancing Reward Model Evaluation | 2025-06-02; ICLR 2026 | 0 | Target paper; best-of-4, six-domain benchmark with BoN/PPO validation | https://arxiv.org/abs/2506.01937 |
| 3 | How to Evaluate Reward Models for RLHF / Preference Proxy Evaluations | 2024-10-18; arXiv | not returned | Separates correctness and human-preference proxies and evaluates downstream relation | https://arxiv.org/abs/2410.14872 |
| 4 | RM-Bench: Benchmarking Reward Models of Language Models with Subtlety and Style | 2024-10-21; arXiv | not returned | Pairwise benchmark emphasizing subtle content/style differences | https://arxiv.org/abs/2410.16184 |
| 5 | Evaluating Robustness of Reward Models for Mathematical Reasoning | 2024-10-02; arXiv | not returned | Domain-specific RM robustness predecessor for mathematical reasoning | https://arxiv.org/abs/2410.01729 |
| 6 | Inference-Time Scaling for Generalist Reward Modeling | 2025-04-03; arXiv | not returned | DeepSeek-GRM scales generative judging with principles, critiques and parallel sampling | https://arxiv.org/abs/2504.02495 |
| 7 | Rethinking Reward Model Evaluation Through the Lens of Reward Overoptimization | 2025-05-19; arXiv | 0 | Tests benchmark design against policy overoptimization, not only static accuracy | https://arxiv.org/abs/2505.12763 |
| 8 | VerifyBench: Benchmarking Reference-based Reward Systems for Large Language Models | 2025-05-21; arXiv | 0 | Extends meta-evaluation to reference-based reasoning verifiers | https://arxiv.org/abs/2505.15801 |
| 9 | One Token to Fool LLM-as-a-Judge | 2025-07-11; arXiv | 1 | Adversarially tests false-positive rewards from superficial master-key tokens | https://arxiv.org/abs/2507.08794 |
| 10 | Agent-RewardBench: Towards a Unified Benchmark for Reward Modeling across Perception, Planning, and Safety in Real-World Multimodal Agents | 2025-06-26; arXiv | not returned | Moves RM evaluation from single text answers to multimodal agent steps and trajectories | https://arxiv.org/abs/2506.21252 |
| 11 | An Empirical Study of LLM-as-a-Judge for LLM Evaluation: Fine-tuned Judge Model is not a General Substitute for GPT-4 | ACL Findings 2025 | 34 | Cross-dataset generalization warning for specialized/fine-tuned judges | https://aclanthology.org/2025.findings-acl.306/ |
| 12 | An Empirical Investigation of Practical LLM-as-a-Judge Improvement Techniques on RewardBench 2 | 2026-04-15; arXiv | 0 | Direct follow-up using RB2 to compare criteria injection and judge ensembling | https://arxiv.org/abs/2604.13717 |

## Overview

The retained corpus tracks a shift from pairwise static RM accuracy toward three additional validity questions: whether rankings survive best-of-N use, whether they predict policy optimization, and whether reward systems remain reliable under distribution shift or adversarial inputs. RewardBench 2 is the bridge: it improves static measurement and then demonstrates both a successful transfer case (BoN) and a boundary case (PPO).

## Trends

- **2024 — benchmark formation:** RewardBench, RM-Bench, PPE and domain-specific math robustness work establish pairwise accuracy, preference agreement and proxy evaluation as competing RM metrics.
- **Early 2025 — evaluation follows the optimization loop:** reward-overoptimization work asks whether benchmark design predicts what happens after the policy is optimized; DeepSeek-GRM asks how judge performance scales with inference compute.
- **Mid-2025 — specialization and attacks:** VerifyBench isolates reference-based reasoning verification; One Token shows that high standard accuracy can coexist with exploitable false-positive reward channels; Agent-RewardBench moves toward trajectory-level multimodal feedback.
- **2026 — RewardBench 2 becomes a development target:** criteria injection and ensembling are tuned directly against RB2, illustrating both practical usefulness and the eventual risk of benchmark overfitting.

## Key themes

1. **Harder static discrimination** — best-of-4, subtle negatives and domain-specific verifiers increase headroom beyond pairwise tests (#1, #2, #4, #5).
2. **Downstream validity** — static scores must be checked separately for candidate selection and policy optimization (#2, #3, #7).
3. **Reference-based verification** — reasoning correctness requires dedicated verifier benchmarks rather than broad preference proxies (#5, #8).
4. **Generative judge scaling and robustness** — more inference compute can improve judges, but superficial triggers can still hack them (#6, #9, #12).
5. **Distribution and modality expansion** — judge generalization across datasets, policy lineages and agent trajectories remains unresolved (#2, #10, #11).

## Keyword frequency in retained titles

| Keyword | Count |
|---|---:|
| reward model / reward modeling | 8 |
| evaluation / evaluating | 7 |
| benchmark / benchmarking | 6 |
| judge / judging | 3 |
| robustness / overoptimization | 3 |

## Most cited accepted papers in the retained set

| Rank | Title | Year | Citations |
|---:|---|---:|---:|
| 1 | An Empirical Study of LLM-as-a-Judge for LLM Evaluation | 2025 | 34 |
| 2 | RewardBench | 2024 | 11 |
| 3 | One Token to Fool LLM-as-a-Judge | 2025 | 1 |
| 4 | RewardBench 2 | 2026 | 0 in API snapshot |

Only four retained records had both clearly verified acceptance/publication status and a citation value in the API snapshot. Missing or zero counts for recent papers are not evidence of low impact.

## Most cited first authors in the retained set

| Rank | Author | Papers in set | Total citations |
|---:|---|---:|---:|
| 1 | Hui Huang | 1 | 34 |
| 2 | Nathan Lambert | 1 | 11 |
| 3 | Yulai Zhao | 1 | 1 |
| 4 | Saumya Malik | 1 | 0 |
| 5 | Sunghwan Kim | 1 | 0 |

The table uses only counts returned in the 2026-09-23 search snapshot; it does not infer citations for official-page-only records.

## Recommended reading path

1. **RewardBench** (#1) — establish the original pairwise RM evaluation problem and open tooling.
2. **RewardBench 2** (#2) — see how best-of-4, unseen prompts and downstream BoN/PPO validation change the benchmark's claims.
3. **Rethinking Reward Model Evaluation Through the Lens of Reward Overoptimization** (#7) — move from static correctness to behavior after optimizing against the reward.
4. **VerifyBench** (#8) — isolate reference-based reasoning verification as a separate construct.
5. **One Token to Fool LLM-as-a-Judge** (#9) — finish with the adversarial robustness gap that ordinary accuracy leaves unmeasured.

## Synthesis

A useful RM evaluation must state which downstream operation it predicts. RewardBench 2 gives strong evidence for candidate ranking under one BoN setup and a useful floor for PPO, but it also shows that policy lineage and prompt distribution dominate fine-grained PPO ranking among competent RMs. The surrounding literature adds two missing axes: optimization-induced overfitting and adversarial reward hacking. The defensible interpretation is therefore not “RB2 score selects the best verifier,” but “RB2 is a broad static gate that must be followed by target-policy, target-distribution and attack-aware validation.”
