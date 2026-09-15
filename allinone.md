# Related-paper search — EvalGen / validating LLM evaluators

Search date: 2026-09-15 (Asia/Shanghai)

Window: 2024–2026

Queries: `human aligned evaluation criteria LLM outputs`; `interactive evaluation rubric refinement LLM`; `meta-evaluation LLM judge criteria`; `LLM evaluator rubric human preferences`

## Search coverage and limitations

The repository's installed `paper_search` CLI was run against arXiv, Semantic Scholar, OpenAlex, Crossref and DBLP. It returned 25 unique records after merging 7 duplicates (`semantic_scholar=8`, `open_alex=24`); arXiv repeatedly returned HTTP 429, Crossref returned TLS EOF errors, and DBLP returned non-JSON responses. A structured follow-up query returned 15 OpenAlex records but Semantic Scholar again returned HTTP 429 and OpenReview returned no matches. The installed CLI still lacks the skill-documented `--json` flag, so the programmatic API was used to preserve the follow-up abstracts in `tmp/paper-search-evalgen.json`.

The table below retains the 12 records judged directly relevant from title plus available abstract or a verified primary page. Thirteen records about unrelated application domains or broad LLM studies were filtered. Zero results from a failed or rate-limited source are not treated as evidence of absence. Citation counts are API snapshots and may drift.

## Relevant results

| # | Paper | Date / venue | Citations in search snapshot | Why it is relevant | Primary source |
|---|---|---|---:|---|---|
| 1 | Who Validates the Validators? Aligning LLM-Assisted Evaluation of LLM Outputs with Human Preferences | 2024-04-18; UIST 2024 | 141 | Target paper; mixed-initiative criteria and assertion validation | https://arxiv.org/abs/2404.12272 |
| 2 | LLM-Rubric: A Multidimensional, Calibrated Approach to Automated Evaluation of Natural Language Texts | ACL 2024 | 44 | Calibrates multidimensional rubric responses to individual human judges | https://aclanthology.org/2024.acl-long.745/ |
| 3 | EvaluLLM: LLM Assisted Evaluation of Generative Outputs | IUI 2024 | 33 | Interactive comparison and human oversight predecessor | https://doi.org/10.1145/3640544.3645216 |
| 4 | Human-Centered Design Recommendations for LLM-as-a-Judge | 2024-07-03 | 1 | Independent eight-expert evidence on control, trust and criterion design | https://arxiv.org/abs/2407.03479 |
| 5 | Constructing Domain-Specific Evaluation Sets for LLM-as-a-Judge | CustomNLP4U 2024 | 10 | Builds local evaluation data rather than assuming generic judge validity | https://aclanthology.org/2024.customnlp4u-1.14/ |
| 6 | Re-evaluating Automatic LLM System Ranking for Alignment with Human Preference | Findings of NAACL 2025 | 5 | Shows automatic benchers degrade on similarly capable systems | https://aclanthology.org/2025.findings-naacl.260/ |
| 7 | The Progress Illusion: Revisiting Meta-evaluation Standards of LLM Evaluators | Findings of EMNLP 2025 | 0 | Tests evaluator validity at realistic, small model-quality gaps | https://aclanthology.org/2025.findings-emnlp.1036/ |
| 8 | Approximating Human Preferences Using a Multi-Judge Learned System | 2025-10-29 | 0 | Learns aggregation over rubric-conditioned judges and preference personas | https://arxiv.org/abs/2510.25884 |
| 9 | Rethinking Rubric Generation for Improving LLM Judge and Reward Modeling for Open-ended Tasks | 2026-02-04 | 45 | Recursive decomposition/filtering for coverage, direction and redundancy | https://arxiv.org/abs/2602.05125 |
| 10 | From Rubrics to Reliable Scores: Evidence-Grounded Text Evaluation with LLM Judges | 2026-01-13 | 1 | Criteria transfer through evidence-grounded execution and calibration | https://arxiv.org/abs/2601.08654 |
| 11 | iRULER: Intelligible Rubric-Based User-Defined LLM Evaluation for Revision | CHI 2026 | 2 | User-defined rubric interpretation and revision workflow | https://arxiv.org/abs/2602.12779 |
| 12 | Evaluative Fingerprints: Stable and Systematic Differences in LLM Evaluator Behavior | 2026-01-08 | 0 | Treats judges as distinct measurement devices with stable dispositions | https://openalex.org/W7120272790 |

## Overview

The retrieved line of work moves from generating evaluators to validating the entire measurement chain. EvalGen and EvaluLLM place humans inside criterion formation; LLM-Rubric and later Rulers calibrate criteria to human scoring behavior; 2025 meta-evaluation papers ask whether judge scores can distinguish the small differences encountered in model development; 2026 work refines rubrics at scale or measures stable judge-specific distortions.

## Trends

- **2024 — interface and calibration:** systems make rubric construction interactive and treat human disagreement as information rather than simple label noise.
- **2025 — decision-valid meta-evaluation:** studies stop relying only on broad leaderboard correlation and test close model pairs, reference choice and domain-specific data.
- **2026 — rubric systems become pipelines:** recursive rubric refinement, evidence-grounded execution, score calibration and judge-disposition audits are separated into inspectable stages.
- The dominant venues span HCI (UIST/CHI) and NLP (ACL/NAACL/EMNLP), reflecting that validator quality is jointly an interaction-design and measurement problem.

## Key themes

1. **Criterion elicitation and drift** — standards emerge while users inspect concrete outputs (#1, #3, #4, #11).
2. **Criteria transfer and calibration** — a written rubric must be converted into a stable scoring protocol and mapped to human scales (#2, #8, #10).
3. **Meta-evaluation at the real decision boundary** — high global correlation can hide failure on near-tied systems (#6, #7).
4. **Rubric structure and redundancy** — criteria need coverage, correct preference direction and non-redundant weighting (#9, #10).
5. **Judge-specific measurement behavior** — evaluators may consistently implement different theories of quality (#8, #12).

## Keyword frequency in retained titles

| Keyword | Count |
|---|---:|
| evaluation / evaluator | 8 |
| LLM | 8 |
| rubric | 5 |
| human preference | 4 |
| judge | 4 |

## Most cited accepted papers in the retained set

| Rank | Title | Year | Citations |
|---:|---|---:|---:|
| 1 | Who Validates the Validators? | 2024 | 141 |
| 2 | LLM-Rubric | 2024 | 44 |
| 3 | EvaluLLM | 2024 | 33 |
| 4 | Constructing Domain-Specific Evaluation Sets for LLM-as-a-Judge | 2024 | 10 |
| 5 | Re-evaluating Automatic LLM System Ranking for Alignment with Human Preference | 2025 | 5 |

## Most cited first authors in the retained set

| Rank | Author | Papers in set | Total citations |
|---:|---|---:|---:|
| 1 | Shreya Shankar | 1 | 141 |
| 2 | William F. Shen | 1 | 45 |
| 3 | Helia Hashemi | 1 | 44 |
| 4 | Michael Desmond | 1 | 33 |
| 5 | Ravi Raju | 1 | 10 |

Counts reflect the search snapshot rather than authoritative citation indexing; William F. Shen's item is a 2026 preprint and is excluded from the accepted-paper ranking above.

## Recommended reading path

1. **EvalGen** (#1) — start with the criterion-formation and criteria-drift problem.
2. **LLM-Rubric** (#2) — see how fixed multidimensional criteria can be calibrated to different human judges.
3. **The Progress Illusion** (#7) — learn why evaluator validity must be tested on close, decision-relevant model comparisons.
4. **Rulers** (#10) — connect rubric intent to evidence-grounded, perturbation-tested scoring.
5. **RRD** (#9) — finish with scalable rubric decomposition and filtering, while retaining EvalGen's warning that automatic refinement still needs human validation.

## Synthesis

EvalGen's lasting contribution is the boundary it exposes: criteria, implementations, labels and output distributions are not independent. Later methods improve one edge of this chain—calibration, fine-grained meta-evaluation, rubric decomposition or evidence grounding—but none makes the chain self-validating. A robust evaluator therefore needs versioned criteria, examples that motivated each revision, separate tests for criterion coverage and implementation fidelity, a future/distributional holdout, and an explicit owner for resolving conflicting human preferences.
