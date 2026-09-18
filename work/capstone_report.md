# Capstone Report — Refresh / Content Opportunity Scoring

**Author:** Mohammed Ibrahim SH  
**Lane:** Refresh / Content Opportunity Scoring  
**Repository:** https://github.com/moham882/flyrank_ml_internship  
**Date:** September 2026

## 0. Abstract

This study asks whether observed search and content-performance signals can be used to rank pages for refresh or review. Using the FlyRank ML Internship warehouse, March 2026 search-performance data was used as the feature window and April 2026 data was held out for time-aware validation across 116,114 eligible content items. A transparent opportunity score combined CTR opportunity relative to position-band benchmarks with ranking-position opportunity, and was compared with a simple CTR-only baseline using Spearman rank correlations and score-quintile analysis. The combined score showed a moderate negative association with subsequent April CTR (ρ = −0.468, p < 0.001) and a stronger association with April average position (ρ = +0.544, p < 0.001), but the CTR-only baseline had a stronger association with April CTR (ρ = −0.512, p < 0.001); the lowest-score quintile also had 3.36× the mean April CTR of the highest quintile. The resulting score is therefore best used as a directional editorial review signal that helps prioritize pages for human inspection, not as a causal prediction that refreshing a page will improve performance.

## 1. Problem framing

**Decision supported:**  
Which content pages should an editor prioritize for refresh or review?

**Unit of analysis:**  
Individual content/page identified by `content_hash_id`, using March 2026 as the feature window.

**Output:**  
A continuous opportunity score that ranks eligible pages from lower to higher review opportunity, together with a reason code and recommended action.

**Human action:**  
An editor reviews high-scoring pages and determines whether the page requires a refresh, title/snippet improvement, deeper content coverage, or no change.

**Cost of a wrong call:**
- False positive: editorial time may be spent reviewing a page that does not need changes.
- False negative: a potentially useful content opportunity may be missed.

**Why use data/ML:**  
The warehouse contains search-performance signals such as impressions, clicks, CTR, and average position. A repeatable scoring method can combine these signals and create a consistent review queue.

**Research question:**  
Can observed search and content-performance signals be used to rank pages for refresh or review?

**Framing:**  
This is a decision-support system. The March score is compared with observed April performance for directional validation. The analysis does not establish that refreshing a page causes its performance to improve.

## 2. Data safety

**Feature window**
- March 2026

**Validation window**
- April 2026

The data was aggregated at the `content_hash_id` level.

**Primary scoring signals included:**
- `gsc_impressions`
- `gsc_clicks`
- CTR
- `gsc_avg_position`

Pages with fewer than 50 March impressions were excluded from the scoring population.

**Eligible March content items:** 116,114.

April data was used only for validation and included:
- April impressions
- April clicks
- April CTR
- April average position
- April sessions
- April engaged sessions

April performance was not used to calculate the March opportunity score.

No future-derived label such as April trend direction or April percentage change was used as a March scoring feature.

`content_hash_id` was used for joining and output identification, not as a predictive feature.

The report and ranked outputs do not intentionally include client names, domains, private queries, credentials, or raw client-identifying exports.

An explicit leakage check confirmed that April validation fields were not present in the March opportunity-score inputs.

## 3. Baseline

The baseline ranks pages using March CTR only, with lower CTR receiving higher review priority.

The baseline was evaluated on the same 116,114 pages and against the same April validation period.

| Method | Spearman ρ with April CTR |
|---|---:|
| CTR-only baseline | −0.512 |
| Combined opportunity score | −0.468 |

Both associations were statistically significant at p < 0.001.

The combined score therefore did not outperform the simple CTR-only baseline for the April CTR metric. This negative result is retained rather than selecting a more favorable interpretation.

The baseline is directional and observational rather than causal.

## 4. Model / analysis

The analysis uses a transparent opportunity-scoring method rather than a trained predictive model.

**Signals**

Two March signals were combined:
1. CTR opportunity
2. Ranking-position opportunity

**Step 1 — CTR**

CTR was calculated as:

`clicks / impressions`

for pages with positive impressions.

**Step 2 — Position-band CTR benchmarks**

Pages were grouped into position bands:
- 1–3
- 4–5
- 6–10
- 11–20
- 21–50
- 51–100

For each band, the median March CTR was calculated.

**Step 3 — CTR opportunity**

For each page:

`CTR gap = position-band benchmark CTR − page CTR`

Negative gaps were clipped to zero.

**Step 4 — Ranking opportunity**

Average position was used as the ranking-opportunity signal and clipped to the range 0–50.

Higher position numbers represent weaker search ranking.

**Step 5 — Normalization**

Both signals were converted to percentile ranks.

**Step 6 — Combined opportunity score**

The two normalized signals were combined with equal weighting:
- 50% CTR opportunity
- 50% ranking-position opportunity

**Eligibility**

Only pages with at least 50 March impressions were included.

**Reason codes**

Example:

`LOW_CTR_AND_WEAK_POSITION`

**Recommended action:**

Refresh / Improve CTR

The validation target was whether higher March scores were associated with weaker subsequent April search performance.

## 5. Evaluation

Validation used a time-aware design:

**March 2026 score → April 2026 observed performance**

All 116,114 eligible March content items matched an April content record.

### April CTR

The combined opportunity score had a moderate negative association with April CTR:

**Spearman ρ = −0.468, p < 0.001**

The CTR-only baseline showed:

**Spearman ρ = −0.512, p < 0.001**

Therefore, the baseline showed a stronger association with subsequent April CTR.

### April average position

Among the 114,993 items with observed April average position:

| Method | Spearman ρ with April position |
|---|---:|
| CTR-only baseline | +0.324 |
| Combined opportunity score | +0.544 |

Because a larger average-position number represents a weaker ranking, the positive association indicates that higher March opportunity scores tended to be associated with weaker subsequent ranking position.

### Score-quintile validation

The lowest March opportunity-score quintile had:

**Mean April CTR: 0.4017%**

The highest March opportunity-score quintile had:

**Mean April CTR: 0.1195%**

The lowest-score quintile therefore had approximately:

**3.36× the mean April CTR**

of the highest-score quintile.

Median April CTR declined from approximately 0.2683% in Q1 to 0% in Q5.

### Missing data

April average position was unavailable for 1,121 validation items. Those items were excluded only from the April-position correlation analysis.

### Leakage check

The explicit leakage check found no April validation fields among the March scoring features.

**Leakage check: PASSED.**

## 6. Interpretation

Higher March opportunity scores were generally associated with weaker subsequent April search performance.

The combined score showed a stronger association with April average position than the CTR-only baseline.

However, the combined score did not outperform the CTR-only baseline for April CTR. This indicates that adding the ranking-position signal did not improve the CTR validation metric uniformly.

The top 20 recommendations were relatively homogeneous. They primarily had:
- March CTR = 0
- Average position around 20
- Position band = 11–20
- Positive CTR opportunity gap

They therefore received the same reason code:

`LOW_CTR_AND_WEAK_POSITION`

and the same recommended action:

**Refresh / Improve CTR**

This concentration is a property of the scoring method and data distribution; the ranking should not be artificially diversified simply to produce different reason codes.

The score should be treated as a prioritization signal for editorial review rather than an automatic refresh decision.

The validation is observational and directional. It does not establish that refreshing a page will improve its performance.

## 7. Recommendation

Use the opportunity score as a review-queue signal.

Higher-scoring pages can be inspected earlier by an editor, but the score should not automatically trigger a refresh.

**Suggested review workflow**
1. Start with high-scoring pages.
2. Inspect search intent and page relevance.
3. Review title and search snippet opportunities.
4. Investigate impressions without clicks.
5. Review ranking position and content depth.
6. Decide whether the appropriate action is:
   - Refresh content
   - Improve title/snippet
   - Improve topical coverage
   - Monitor
   - Make no change
7. Monitor subsequent performance after any editorial change.

The top 20 ranked pages should be treated as a prioritized review queue, not a definitive list of pages that must be refreshed.

Final editorial decisions remain with the human reviewer.

## 8. Reproducibility

**Repository:**  
https://github.com/moham882/flyrank_ml_internship

**Main capstone notebook:**  
`work/notebooks/capstone.ipynb`

**Validation notebook:**  
`work/notebooks/w06_validation_audit.ipynb`

**Action-playbook notebook:**  
`work/notebooks/w07_action_playbook.ipynb`

**Derived outputs:**
- `work/outputs/ranked_refresh_recommendations.csv`
- `work/outputs/quintile_validation.csv`
- `work/outputs/validation_chart_data.csv`
- `work/outputs/validation_results_summary.csv`
- `work/outputs/april_ctr_by_score_quintile.png`

The analysis used DuckDB, pandas, and SciPy.

The March 2026 dataset was used for scoring and April 2026 was reserved for validation.

The underlying FlyRank warehouse requires authorized access. The repository contains the analysis workflow and derived outputs rather than the private warehouse itself.

An authorized user with the required dataset access can rerun the analysis.

## 9. Acknowledgments & data credit

This work was completed as part of the FlyRank ML Internship 2026 capstone.

**Built on the FlyRank ML Internship dataset**  
https://flyrank.ai

The analysis uses pseudonymous content identifiers and aggregated search-performance signals.

No client names, domains, private search queries, credentials, or raw client-identifying exports are included in the public research paper.

Thanks to FlyRank for providing the dataset and internship framework used for this analysis.
