# FlyRank ML Internship — Capstone Workspace

**Project:** Content Refresh Prioritization
**Author:** Muhammad Abdur Rehman Maqsood
**Track:** ML Foundations (FL) + Portfolio (PF)
**Live paper:** https://muhammadabdurrehmanmaqsood.github.io/flyrank-ml-internship-abdurrehman/

This folder contains the complete build: every notebook, the trained models, the figures,
and the capstone report for the internship's core deliverable — a model that ranks a content
team's existing pages by how likely they are to be a declining page worth reviewing first.

The rest of the repository (`scripts/`, `notebooks/01-03`, `data/`) is the shared starter
kit provided to every intern. Everything in `work/` is the applied project built on top of
it: nine notebooks executed in sequence, a validation audit that changed the outcome, and a
final report that reflects the honest result rather than the first one produced.

---

## 1. Project overview

**Objective.** Given a content team managing pages across many clients, identify which pages
are declining and worth a review pass first — replacing "someone eyeballs a spreadsheet" with
a ranked, explainable queue.

**Why this matters.** A content strategist working through hundreds of pages needs a
defensible starting point. The output isn't a black-box score; each ranked page carries reason
codes (e.g. `declining_with_demand`, `low_ctr_visible_page`) so a reviewer can see *why* a page
was flagged, not just that it was.

**Data.** The pipeline is built against FlyRank's production search-performance warehouse
(~79M rows, daily content metrics across the client base), modeled and evaluated on a
30,000-row anonymized cross-section spanning 32 clients and 3 content types. No client names,
domains, URLs, titles, or keywords are present in the modeling data (see the repository's
`DATA_USE.md` for the full data-safety contract).

**Outcome.** A logistic regression model, evaluated on a client-grouped holdout, that ranks
content 71.5% precise globally and 89% precise at the top 100 rows of the queue — a result that
only exists because a suspiciously perfect earlier version of the model was audited and
rejected before it went into the report.

---

## 2. Methodology

The project follows the notebook sequence below. Each stage was a deliberate checkpoint, not
just a step toward a final notebook — several produced findings that changed the next stage's
approach.

| Stage | Notebook | Purpose | Key decision or finding |
|---|---|---|---|
| 1 | `w01_research_question.ipynb` | Frame the decision this project supports and who acts on the output | Defined the unit of analysis (one content page), the action (refresh review), and the cost of a wrong call |
| 2 | `w02_ml_task_framing.ipynb` | Define the label and the shape of the ML task | `is_declining_label = (trend_direction == "down")` — a binary classification framed to feed a ranked queue, not a standalone prediction |
| 3 | `w03_data_contract.ipynb` | Confirm the data contract: warehouse scale vs. the modeling slice | Validated the 79M-row production scale against the 30K-row anonymized sample used for modeling |
| 3 | `w03_feature_leakage_check.ipynb` | First leakage pass on the raw feature set | Because the label derives from `trend_direction`, both `trend_direction` and `trend_pct` were excluded from the feature set by design |
| 4 | `w04_signal_audit.ipynb` | Separate genuinely predictive signals from noise | Narrowed the candidate feature list to observable, non-label-derived metrics |
| 4 | `w04_baseline_score.ipynb` | Build a transparent, non-ML comparison point | Hand-rule baseline against a 62.8% majority-class base rate — the number every later model has to beat |
| 5 | `w05_model.ipynb` | Train the first model on a client-grouped split | Logistic regression returned **1.000** precision, recall, and accuracy — a result treated as a red flag, not a win |
| 6 | `w06_validation_audit.ipynb` | Investigate the suspicious result | Root-caused the leak (see Section 3), removed it, and re-evaluated |
| 7 | `w07_action_playbook.ipynb` | Turn the fixed model into a usable output | Ranked content-refresh queue with human-readable reason codes per row |
| 8 | `capstone.ipynb` | Final write-up | Consolidates results, limitations, and the figures published to the live paper |

**Modeling approach.** Three model classes were compared on the same client-grouped split:
logistic regression, a decision tree, and a random forest. Logistic regression was carried
forward as the reported model because its behavior on the leakage audit was the most legible —
its coefficients directly exposed the mechanism behind the inflated score (Section 3), which
made the fix verifiable rather than assumed.

**Evaluation design.** All models are evaluated with `GroupShuffleSplit` on `client_id`
(80/20, seed 42), so no client's pages appear in both the training and test sets. This matters
because a per-row random split would let the model see near-duplicate pages from the same
client in both sets, inflating scores in a way that would not hold up on a genuinely new
client.

---

## 3. The validation audit — the pivotal decision point

The project's central methodological event happened at Stage 5→6. The first trained model
scored a perfect 1.000 across precision, recall, and accuracy. On a real-world behavioral
label, a perfect score is a signal to investigate, not a result to report.

**Diagnosis.** The audit traced the score to two features, `impressions_last_30d` and
`impressions_prev_30d`, whose logistic regression coefficients nearly canceled
(-7.92 / +7.91) — the signature of a feature pair mechanically entangled with how the label
itself was computed, rather than genuinely predictive of it.

**Verification, not assumption.** The suspected leak was not taken on faith. A confession
test was run: the model was retrained once with the suspect features and once without, on the
identical split. Accuracy dropped from 1.000 to approximately 0.67 with the features removed —
confirming the leak before the fix was finalized, and giving a reproducible before/after
comparison for the report.

**Consequence.** Every downstream artifact — the action playbook queue, the capstone paper's
figures, and the results below — is built on the leakage-fixed model, not the original.

Full detail, including the coefficient values and the confession-test code, is in
`work/notebooks/w06_validation_audit.ipynb`.

---

## 4. Key results

**On the 30K-row modeling sample (`w06_validation_audit.ipynb`):**

| Metric | Leaky model (pre-audit) | Honest model (post-audit) |
|---|---:|---:|
| Accuracy | 1.000 | ~0.67 |
| Precision | 1.000 | ~71–72% |
| Base rate (majority-class) | — | 62.8% |

**On the full capstone evaluation, read as a ranked queue (`capstone.ipynb`):**

| Metric | Value |
|---|---:|
| Global precision | 71.5% |
| Precision@250 | ~86.8% |
| Precision@100 | 89.0% |

The gap between global precision (71.5%) and precision@100 (89%) is the operational point of
the project: a ranked queue is read top-down, not row-by-row, so the model is materially more
trustworthy at the top of the list than a single global metric would suggest. This is the
number a content strategist actually relies on.

---

## 5. Limitations

Reported directly, because a decision-support tool is only useful if its boundaries are
explicit:

- **Single holdout, not cross-validated** — precision@K would shift somewhat under a different
  random seed.
- **Coverage gap** — the client-grouped holdout used for precision@100/250 landed entirely on
  one content type (`keyword article`); the model trained on all three types, but top-K
  precision is directly demonstrated for only one.
- **Not a calibrated probability** — the score ranks well but is not a literal percentage
  chance of decline.
- **No time-forward claim** — there is no calendar-date column supporting a genuine
  past-vs-future split; this describes resemblance to past declining content, not a forecast.
- **No causal claim** — this is observational data; the model cannot say that refreshing a
  page *causes* recovery.
- **Small sample vs. full warehouse** — the 30K-row modeling slice is drawn from a 79M-row
  warehouse; conclusions have not been re-verified at full scale.
- **No live validation yet** — no A/B test against actual content-team outcomes exists; the
  queue is decision support, not a decision.

---

## 6. Technologies used

| Category | Tools |
|---|---|
| Language | Python |
| Modeling | scikit-learn (logistic regression, decision tree, random forest) |
| Data handling | pandas, numpy |
| Large-scale querying | DuckDB (SQL over the 79M-row Hugging Face–hosted warehouse) |
| Visualization | matplotlib |
| Reporting | Markdown reports, ReportLab (PDF export), static charts as SVG |
| Environment | Google Colab (primary), local venv + `requirements.txt` (alternative) |
| Version control / CI | Git, GitHub Actions (pipeline smoke test, dataset-commit guard) |

---

## 7. File structure

```text
work/
  notebooks/
    w01_research_question.ipynb        Stage 1 — problem framing
    w02_ml_task_framing.ipynb          Stage 2 — label and task definition
    w03_data_contract.ipynb            Stage 3 — data scale and contract check
    w03_feature_leakage_check.ipynb    Stage 3 — first leakage pass
    w04_signal_audit.ipynb             Stage 4 — signal vs. noise audit
    w04_baseline_score.ipynb           Stage 4 — transparent baseline
    w05_model.ipynb                    Stage 5 — first trained model
    w06_validation_audit.ipynb         Stage 6 — leakage audit and fix
    w07_action_playbook.ipynb          Stage 7 — final ranked queue
    capstone.ipynb                     Stage 8 — consolidated write-up
  figures/                             Charts referenced in the capstone report and live paper
  capstone_report.md                   Formal capstone report (see Section 9)
  capstone_report_template.md          Report template / rubric reference
```

---

## 8. How to run this work

1. Clone the repository and install dependencies from the repo root:
   ```bash
   git clone https://github.com/muhammadabdurrehmanmaqsood/flyrank-ml-internship-abdurrehman.git
   cd flyrank-ml-internship-abdurrehman
   pip install -r requirements.txt
   ```
2. Run the reference pipeline once to generate the baseline artifacts referenced by the
   capstone notebooks:
   ```bash
   python scripts/run_all.py
   ```
3. Open any notebook in `work/notebooks/` in order (each is also runnable directly in Colab
   via the badges in the repository's root `README.md`). New notebooks in this project start
   with a setup cell installing `duckdb` and `huggingface_hub`, since the full-scale stages
   query the 79M-row warehouse hosted on Hugging Face rather than a local file.
4. Random seeds are fixed (seed 42) throughout for reproducibility; re-running the notebooks
   end to end on the same environment reproduces the reported numbers.

---

## 9. Capstone report

The formal write-up — problem framing, data safety, baseline, model, evaluation,
interpretation, recommendation, and reproducibility instructions — is `capstone_report.md`,
built from `capstone_report_template.md`. It is the single document that summarizes this
entire folder for a reader who has five minutes, not five notebooks.

---

## 10. Conclusions and learnings

- **A perfect score is a bug report, not a result.** The single highest-value decision in this
  project was refusing to accept a 1.000 accuracy at face value and instead treating it as a
  prompt to audit the pipeline.
- **Verification beats explanation.** Diagnosing the leak by inspecting coefficients was a
  hypothesis; the confession test (retrain with/without the suspect features) is what actually
  confirmed it. Both steps mattered — one located the problem, the other proved it.
- **Ranking is more useful than classifying** for this kind of decision-support tool. The gap
  between global precision and precision@100 shows that framing the output as a ranked queue,
  rather than a single global classifier, is what makes the model's uncertainty tolerable in
  practice.
- **Group-aware splitting is not optional** for client-partitioned data — a per-row split would
  have masked the leak for longer, and would have overstated performance on genuinely new
  clients regardless.
