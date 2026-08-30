# Capstone Report — Content Refresh Prioritization

- **Author:** Muhammad Abdur Rehman Maqsood
- **Lane:** ML Foundations — Search Ranking / Content Performance
- **Repo:** https://github.com/muhammadabdurrehmanmaqsood/flyrank-ml-internship-abdurrehman
- **Live paper:** https://muhammadabdurrehmanmaqsood.github.io/flyrank-ml-internship-abdurrehman/
- **Date:** August 2026

> This document follows the eight-axis structure used to evaluate capstone work in the
> FlyRank ML internship. Each section states what was built, the reasoning behind the
> approach taken, and the evidence supporting the claims made. Source notebooks are cited
> throughout for full technical detail.

---

## 1. Problem framing

**Decision supported.** A content team managing pages across many clients needs to decide
which pages to review and refresh first. Without a systematic aid, this defaults to manual
triage — someone scanning a spreadsheet with no consistent criteria.

**Unit of analysis.** One content page (`content_id`), evaluated at a point in time.

**Output.** A ranked score per page, paired with human-readable reason codes
(e.g. `declining_with_demand`, `low_ctr_visible_page`) explaining why it was flagged.

**Action taken from the output.** A content strategist opens the highest-ranked candidate
first and runs a refresh review — updating, consolidating, or reworking the page.

**Cost of a wrong call.** A false positive costs a reviewer's time on a page that didn't need
attention. A false negative leaves a genuinely declining page unreviewed for another cycle.
Neither is catastrophic in isolation, which is why the project is framed as a prioritization
aid rather than an automated action — the human stays in the loop, and the tool's job is to
make their triage faster and more consistent, not to replace their judgment.

**Why ML helps here.** The warehouse spans ~79M rows across the full client base — well beyond
what manual review can cover systematically. A model that reliably separates likely-declining
pages from stable ones, even imperfectly, converts an unstructured scan into an ordered queue.

---

## 2. Data safety

**Data used.** A 30,000-row anonymized cross-section of FlyRank's content-performance
warehouse, spanning 32 clients and 3 content types. Columns cover observed search and
engagement metrics, content metadata, age/freshness fields, and derived comparison windows.

**What was deliberately excluded.**
- `trend_direction` and `trend_pct` — excluded from the feature set because the target label,
  `is_declining_label`, is derived directly from `trend_direction`. Including either would be
  a direct label leak, not a subtle one.
- `client_id` / `content_id` — hashed pseudonymous identifiers, used only for grouped
  train/test splitting and never as model features.
- FlyRank's own product decision flags (e.g. `health_score`, `needs_ctr_fix`) — not present in
  the starter data at all, so the model is built from observable evidence rather than learning
  to reproduce an existing internal score.

**Leakage risks considered beyond the excluded columns.** The Stage 5→6 validation audit
(Section 5) found a second, less obvious leakage source inside the *permitted* feature set:
two impressions-window features whose relationship to each other mechanically tracked the
label. This is discussed in full in Section 4 and Section 5, since it materially changed the
reported model.

**Confirmation.** No client names, domains, URLs, page titles, or search keywords appear
anywhere in `work/`. All public-facing claims in this report and in the deployed paper use
observed / measured / directional / decision-support language, consistent with the
repository's `DATA_USE.md`.

---

## 3. Baseline

**What it is.** A transparent, hand-authored rule that flags a page as a refresh candidate
based on directly interpretable thresholds (e.g. visible demand paired with weak engagement or
CTR), with no model training involved.

**Why it's a fair comparison.** It uses the same feature set available to the model, requires
no training data, and produces reason codes a reviewer can inspect line by line — it is the
floor any ML approach needs to clear to justify its added complexity.

**Its numbers.** Evaluated on the same data and metric as the model: a majority-class base
rate of **62.8%** — meaning a rule that always guessed "not declining" would be right 62.8% of
the time. This is the number every downstream metric is measured against; a model or rule that
doesn't clear it materially isn't adding value. Detail in `w04_baseline_score.ipynb`.

---

## 4. Model / analysis

**Method.** Logistic regression, compared against a decision tree and a random forest on the
same client-grouped split. Logistic regression is reported as the primary model because its
coefficients are directly interpretable — a property that turned out to be essential during
the validation audit (Section 5), where reading the coefficients was what exposed the leakage
mechanism.

**Target definition.** `is_declining_label = (trend_direction == "down")` — a binary label
marking whether a page's performance trend is downward, defined in `w02_ml_task_framing.ipynb`.

**Feature set.** Observable, non-label-derived search and engagement metrics (impressions,
CTR, engagement rate, scroll rate, AI-traffic percentage, content age/freshness fields, and
content-type metadata). `trend_direction` and `trend_pct` are excluded by design (Section 2).
The final feature list, after the Stage 6 audit removed `impressions_last_30d` and
`impressions_prev_30d`, is enumerated in `w06_validation_audit.ipynb` and
`w07_action_playbook.ipynb`.

**What was left out on purpose.** Beyond the label-adjacent columns, any field that encoded a
FlyRank-internal decision (rather than an observed metric) was excluded, per the data-safety
constraints in Section 2 — the model learns from evidence, not from an existing scoring system.

---

## 5. Evaluation

**Split.** `GroupShuffleSplit` on `client_id`, 80/20, seed 42 — chosen so that no client's
pages appear in both the training and test sets. A per-row random split would let
near-duplicate pages from the same client leak across the split boundary, which would overstate
performance on genuinely new clients. This is the standard applied throughout the project,
starting at `w05_model.ipynb`.

**The leakage finding.** The first trained model (`w05_model.ipynb`) returned precision,
recall, and accuracy all equal to **1.000** — a result treated as a red flag rather than a
success, since a perfect score on a real-world behavioral label almost always indicates a
methodological problem rather than a genuinely solved task.

The `w06_validation_audit.ipynb` investigation traced the result to two features,
`impressions_last_30d` and `impressions_prev_30d`, whose coefficients nearly canceled
(-7.92 / +7.91) — the signature of a feature pair mechanically entangled with how the label
was computed rather than genuinely predictive of it.

The diagnosis was confirmed with a confession test: retraining once with the suspect features
included and once with them removed, on the identical split. Accuracy collapsed from 1.000 to
approximately **0.67** once the features were removed, confirming the leak before it was
reported as fixed.

**Metrics, model vs. baseline, same split (post-fix):**

| Metric | Baseline | Model (honest) |
|---|---:|---:|
| Base rate / accuracy | 62.8% | ~67% |
| Precision | — | ~71–72% |

**Full capstone evaluation, read as a ranked queue (`capstone.ipynb`):**

| Metric | Value |
|---|---:|
| Global precision | 71.5% |
| Precision@250 | ~86.8% |
| Precision@100 | 89.0% |

**Error analysis.** Global precision (71.5%) sits close to the base rate, which is expected
and appropriate for a model read as a full classifier — it is *not* the number that matters for
this use case. Precision@100 (89.0%) is substantially higher, which is the actual point of
building a ranked queue rather than a single global classifier: the model is far more
trustworthy at the top of the list, which is exactly where a reviewer starts. The precision@K
figures were demonstrated on a holdout that, by chance of the client-grouped split, landed
entirely within one content type (`keyword article`) — noted as a coverage limitation in
Section 8 of `work/README.md` rather than glossed over.

---

## 6. Interpretation

**What the model found.** After removing the leaking impressions-window pair, the retained
predictive signal centers on observable engagement quality (CTR, engagement rate, scroll rate)
relative to visible search demand (impressions) — pages with strong demand but weak on-page
engagement are the pages the model consistently ranks highest as refresh candidates. This
matches the intuition behind the original hand-rule baseline, which is a reassuring sign: the
model is not finding an exotic pattern, it is finding a sharper, better-calibrated version of
what a human reviewer would already suspect.

**Surprises and negative results.** The most consequential finding of the project was negative:
the first model's apparent skill was not real. Reporting that finding, and the process used to
confirm it, is treated as being as valuable as the final metric — a well-understood "the first
result was wrong, and here's why" is a legitimate and important outcome of a validation process,
not a failure to hide.

---

## 7. Recommendation

**Ranked actions.** The output of `w07_action_playbook.ipynb` is a ranked content-refresh
queue: each row is a candidate page, a numeric priority score, an action label (e.g.
`refresh_and_review_ctr`, `refresh`), and the reason codes driving that score.

**How a FlyRank editor would use it.** Start at the top of the queue, open the highest-ranked
page, and use the attached reason codes to understand what's driving the flag (declining trend,
low CTR relative to visible demand, low engagement) before deciding on a refresh approach.
Given the precision@100 figure, the top 100 rows are where confidence is highest; rows further
down the queue still outperform the base rate but warrant more of the reviewer's own judgment.

**Confidence and limits, stated explicitly.** This is a decision-support ranking, not an
automated decision and not a forecast. It has not been validated against real-world refresh
outcomes (no A/B test exists yet), it does not support causal claims about what refreshing a
page would achieve, and its precision@K evidence is strongest for one content type. The full
limitations list is in Section 5 of `work/README.md`.

---

## 8. Reproducibility

**Environment.** Python with pandas, numpy, scikit-learn, matplotlib, duckdb, and
huggingface_hub — pinned in the repository's `requirements.txt`.

**Commands to reproduce from a fresh clone:**

```bash
git clone https://github.com/muhammadabdurrehmanmaqsood/flyrank-ml-internship-abdurrehman.git
cd flyrank-ml-internship-abdurrehman
pip install -r requirements.txt
python scripts/run_all.py
```

Then run the notebooks in `work/notebooks/` in numeric order (`w01` through `w07`, then
`capstone.ipynb`), or open each via the Colab badges in the repository's root `README.md`.
Notebooks that query the full warehouse begin with a setup cell installing `duckdb` and
`huggingface_hub`, since a fresh kernel does not have these by default.

**Random seed.** 42, fixed throughout for the client-grouped split (`GroupShuffleSplit`) and
model training.

**Environment notes.** Metrics reproduce closely on the pinned `requirements.txt` stack.
Small variation (roughly a few percentage points) is possible across different scikit-learn or
numpy versions, since model coefficients and holdout composition are sensitive to library
version at the margins; the stable, load-bearing claim is the lift over the 62.8% base rate and
the precision@100 figure, not any single metric's third decimal.

---

> **Claims checklist:** observed / measured / directional / decision-support language used
> throughout · metrics reported against the 62.8% base rate, not in isolation · no causal
> claims made · no "predicted Google's algorithm" framing · no client-identifying details
> anywhere in this report · figures in this report match a fresh re-run of the pipeline and
> notebooks on the pinned `requirements.txt` environment.
