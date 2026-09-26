# Related-paper search — LLM evaluator self-preference

Search date: 2026-09-26 (Asia/Shanghai)

Window: 2024–2026

Queries: `LLM evaluator self preference bias`; `LLM judge own generations bias`; `evaluator generator identity bias`

## Search coverage and limitations

The repository's installed `paper_search` CLI queried arXiv, DBLP, OpenAlex, OpenReview, Semantic Scholar and Crossref. It returned 46 unique records after merging 10 cross-source duplicates (`open_alex=24`, `semantic_scholar=8`, `crossref=24`; `arxiv=0`, `dblp=0`, `openreview=0`). arXiv returned HTTP 406 for all three queries, DBLP returned non-JSON responses, and Semantic Scholar returned HTTP 429 for part of the query set. Failed or rate-limited sources are not treated as evidence of absence.

The installed CLI still lacks the skill-documented `--json` option, so it could not emit a complete abstract-bearing recovery file. Relevance, dates and claims for the retained set were checked against official arXiv, ACL Anthology or NeurIPS pages. Clearly unrelated generic evaluation and non-LLM bias records were excluded; uncertain search omissions remain possible. Citation counts are the incomplete API snapshot returned on 2026-09-26 and should not be interpreted as current totals.

## Relevant results

| # | Paper | Date / venue | Citations in search snapshot | Relation to target paper | Primary source |
|---|---|---|---:|---|---|
| 1 | LLM Evaluators Recognize and Favor Their Own Generations | 2024-04-15; NeurIPS 2024 | 807 | Target paper; intervenes on self-recognition and measures self-preference | https://arxiv.org/abs/2404.13076 |
| 2 | Self-Preference Bias in LLM-as-a-Judge | 2024-10-29; arXiv | 5 | Tests familiarity/perplexity as a competing explanation | https://arxiv.org/abs/2410.21819 |
| 3 | Replacing Judges with Juries: Evaluating LLM Generations with a Panel of Diverse Models | 2024-04-29; arXiv | 352 | Multi-model jury mitigation for single-judge family effects | https://arxiv.org/abs/2404.18796 |
| 4 | Benchmarking Cognitive Biases in Large Language Models as Evaluators | ACL Findings 2024 | 43 | Places egocentric preference alongside broader evaluator biases | https://aclanthology.org/2024.findings-acl.29/ |
| 5 | Humans or LLMs as the Judge? A Study on Judgement Bias | EMNLP 2024 | 64 | Compares human and LLM judgment biases | https://aclanthology.org/2024.emnlp-main.474/ |
| 6 | Do LLM Evaluators Prefer Themselves for a Reason? | 2025-04-04; arXiv | 0 | Separates legitimate quality preference from harmful self-preference on verifiable tasks | https://arxiv.org/abs/2504.03846 |
| 7 | Play Favorites: A Statistical Method to Measure Self-Bias in LLM-as-a-Judge | 2025-08-08; arXiv | 48 | Adjusts for underlying response quality with an independent evaluator | https://arxiv.org/abs/2508.06709 |
| 8 | Beyond the Surface: Measuring Self-Preference in LLM Judgments | EMNLP 2025 | 3 | Uses gold judgments/DBG to reduce response-quality confounding | https://aclanthology.org/2025.emnlp-main.86/ |
| 9 | Assistant-Guided Mitigation of Teacher Preference Bias in LLM-as-a-Judge | EMNLP Findings 2025 | 0 | Studies mitigation when a teacher judge favors aligned outputs | https://aclanthology.org/2025.findings-emnlp.510/ |
| 10 | Are LLM Evaluators Really Narcissists? Sanity Checking Self-Preference Evaluations | 2026-01-30; arXiv | 0 | Shows shared errors can create false self-preference and proposes a quality baseline | https://arxiv.org/abs/2601.22548 |
| 11 | Quantifying and Mitigating Self-Preference Bias of LLM Judges | 2026-04-24; arXiv | 0 | Automated equal-quality pairing across 20 models plus structured mitigation | https://arxiv.org/abs/2604.22891 |
| 12 | Self- and Other-Labels Induce Bidirectional Bias in LLM Judges | 2026-06-06; arXiv | not returned | Separates source labels from generated-text style fingerprints | https://arxiv.org/abs/2608.18091 |

## Overview

The literature moves from observing that an evaluator favors its own generations to identifying what the measurement actually contains. The target paper supplies an intervention on recognition ability; later work adds independent quality adjustment, verifiable correctness, gold judgments, shared-error controls and label-only manipulations. The central trend is methodological tightening: “picked its own output” is no longer accepted as sufficient evidence of identity-based favoritism.

## Trends

- **2024 — phenomenon and mechanism candidates:** the target paper links self-recognition to preference; contemporaneous work studies perplexity/familiarity, broader cognitive biases and multi-judge aggregation.
- **Early 2025 — legitimate versus harmful preference:** verifiable math, factual and code tasks make it possible to ask whether the judge's own answer is actually better before labeling its vote biased.
- **Late 2025 — explicit quality correction:** statistical and gold-judgment methods compare self/family effects conditional on independently estimated quality rather than raw win rate.
- **2026 — sanity checks and interventions:** shared mistakes, evaluator quality, equal-quality pair construction and source-label experiments test whether earlier measurements isolate the intended construct; mitigation shifts toward decomposed rubrics and assistant/jury designs.

## Key themes

1. **Recognition versus familiarity** — an evaluator may identify a family style without representing literal authorship (#1, #2).
2. **Quality confounding** — the model's output may genuinely be better, or judge and generator may share the same mistake (#6, #7, #8, #10).
3. **Self-bias versus family-bias** — shared post-training and model-family signatures can matter even when checkpoints differ (#7, #12).
4. **Protocol sensitivity** — order, labels, pairwise versus absolute grading and reference availability change measured bias (#1, #8, #12).
5. **Mitigation by independence and decomposition** — diverse juries, assistant critiques and multidimensional rubrics reduce reliance on one model's latent preferences (#3, #9, #11).

## Keyword frequency in retained titles

| Keyword | Count |
|---|---:|
| preference / prefer / favorites | 8 |
| judge / evaluator | 8 |
| bias | 7 |
| self / own | 7 |
| measure / quantify / benchmark | 4 |

## Most cited accepted papers in the retained set

| Rank | Title | Year | Citations |
|---:|---|---:|---:|
| 1 | LLM Evaluators Recognize and Favor Their Own Generations | 2024 | 807 |
| 2 | Replacing Judges with Juries | 2024 | 352 |
| 3 | Humans or LLMs as the Judge? | 2024 | 64 |
| 4 | Play Favorites | 2025 | 48 |
| 5 | Benchmarking Cognitive Biases in LLMs as Evaluators | 2024 | 43 |

Acceptance status was verified for the NeurIPS/ACL/EMNLP papers. Search-snapshot counts are uneven across providers and recent papers, so ranks are descriptive only.

## Most cited first authors in the retained set

| Rank | Author | Papers in set | Total citations |
|---:|---|---:|---:|
| 1 | Arjun Panickssery | 1 | 807 |
| 2 | Pat Verga | 1 | 352 |
| 3 | Guiming Hardy Chen | 1 | 64 |
| 4 | Evangelia Spiliopoulou | 1 | 48 |
| 5 | Ryan Koo | 1 | 43 |

The table uses only counts returned in the 2026-09-26 search snapshot and does not infer missing citations.

## Recommended reading path

1. **LLM Evaluators Recognize and Favor Their Own Generations** (#1) — understand the original intervention and its evidence boundary.
2. **Do LLM Evaluators Prefer Themselves for a Reason?** (#6) — introduce verifiable correctness and legitimate versus harmful preference.
3. **Play Favorites** (#7) — see explicit statistical adjustment for candidate quality and family effects.
4. **Beyond the Surface** (#8) — replace raw self-win rates with comparison to gold judgments.
5. **Are LLM Evaluators Really Narcissists?** (#10) — finish with the strongest sanity check on shared-error confounding.

## Synthesis

Self-preference is best treated as a measurement-identification problem, not a single bias score. Panickssery et al. show that recognition capability and preference co-move under interventions, which is stronger than a raw observational win rate. Yet later work demonstrates that familiarity, true quality, shared errors, family lineage and source labels can produce overlapping patterns. A defensible eval therefore needs blinded sources, order randomization, independent or verifiable quality controls, cross-family judges and stratified reporting. Without those controls, “the judge preferred itself” is an observation; it is not yet a causal explanation.
