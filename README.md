# FlyRank Search Ranking Capstone — Content Refresh Prioritization

**What it does, and for whom:** this project ranks a content team's existing pages by how likely they are to be a declining page worth reviewing first — so a content strategist opens the highest-value candidate instead of guessing. It's built on FlyRank's production search-performance warehouse (79M rows, daily content metrics across the full client base) and modeled on a 30,000-row anonymized cross-section spanning 32 clients and 3 content types.

**Built with Claude — here's what I checked myself.** I used Claude as a coding and research partner throughout this project. The most consequential moment: during the Week 6 validation audit, Claude helped me run a leakage-hunting checklist against a model that had scored a suspicious 1.0 accuracy — precision, recall, and accuracy all perfect, which is a red flag on a real-world behavioral label, not a result to celebrate. That audit traced the score to two features, `impressions_last_30d` and `impressions_prev_30d`, whose coefficients nearly canceled (-7.92 / +7.91) — the signature of features mechanically tied to how the label was computed. I didn't take that conclusion on faith: I ran the confession test myself (retraining once with the suspect features, once without) and watched accuracy collapse from 1.0 to ~0.67, which is what confirmed the leak before I trusted the fix. Claude also helped build the Week 7 action-playbook notebook, draft the capstone write-up, and harden the [portfolio site](https://flyrank.abdurrehman.online) (metadata, analytics, accessibility) that showcases this project to non-technical reviewers.

> New here? Two reads: **[SETUP.md](SETUP.md)** (GitHub, Colab, and data access), then **[GUIDE.md](GUIDE.md)** (every file explained). The capstone package for the AI Fluency track — retrospective, demo script, build-in-public post, submission index — lives in **[capstone-package/](capstone-package/INDEX.md)**.

---

## Quickstart — first win in 2 minutes

The fastest path is Google Colab (one click, zero install). Open Notebook 1 and run all cells:

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/muhammadabdurrehmanmaqsood/flyrank-ml-internship-abdurrehman/blob/main/notebooks/01_first_look_and_discovery.ipynb?flush_cache=true)
 **Week 1 — Run it, then discover a real truth yourself**

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/muhammadabdurrehmanmaqsood/flyrank-ml-internship-abdurrehman/blob/main/notebooks/02_your_first_readable_model.ipynb?flush_cache=true)
 **Week 2 — The model is just a rule you can read**

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/muhammadabdurrehmanmaqsood/flyrank-ml-internship-abdurrehman/blob/main/notebooks/03_working_with_the_full_release.ipynb?flush_cache=true)
 **Weeks 3+ — The full release (~79M rows) via DuckDB, no download needed** — hosted at
 [`FlyRank/internship-warehouse`](https://huggingface.co/datasets/FlyRank/internship-warehouse) (gated: request access + accept the data-use terms, approval is instant)

### Prefer local?

```bash
git clone https://github.com/muhammadabdurrehmanmaqsood/flyrank-ml-internship-abdurrehman.git
cd flyrank-ml-internship-abdurrehman
pip install -r requirements.txt          # or: uv pip install -r requirements.txt
python scripts/run_all.py
```

That runs the whole reference pipeline on the bundled sample and writes results to `outputs/`.

---

## The capstone notebooks — the actual build, in order

Every assignment is one notebook in `work/notebooks/`. This is the real sequence the project was built in:

| Week | Card | Notebook | What happens there | Open |
|---|---|---|---|---|
| 1 | ML-02 | `w01_research_question` | Frames the decision this project supports and who acts on it | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/muhammadabdurrehmanmaqsood/flyrank-ml-internship-abdurrehman/blob/main/work/notebooks/w01_research_question.ipynb?flush_cache=true) |
| 2 | ML-03 | `w02_ml_task_framing` | Defines the label (`is_declining_label`) and the ML task shape | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/muhammadabdurrehmanmaqsood/flyrank-ml-internship-abdurrehman/blob/main/work/notebooks/w02_ml_task_framing.ipynb?flush_cache=true) |
| 3 | ML-04 | `w03_data_contract` | Confirms the 79M-row warehouse scale and the anonymized modeling slice | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/muhammadabdurrehmanmaqsood/flyrank-ml-internship-abdurrehman/blob/main/work/notebooks/w03_data_contract.ipynb?flush_cache=true) |
| 3 | ML-05 | `w03_feature_leakage_check` | First leakage pass on the raw feature set | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/muhammadabdurrehmanmaqsood/flyrank-ml-internship-abdurrehman/blob/main/work/notebooks/w03_feature_leakage_check.ipynb?flush_cache=true) |
| 4 | ML-06 | `w04_signal_audit` | Audits which signals are genuinely predictive vs. noise | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/muhammadabdurrehmanmaqsood/flyrank-ml-internship-abdurrehman/blob/main/work/notebooks/w04_signal_audit.ipynb?flush_cache=true) |
| 4 | ML-07 | `w04_baseline_score` | Builds the transparent hand-rule baseline (62.8% base rate) | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/muhammadabdurrehmanmaqsood/flyrank-ml-internship-abdurrehman/blob/main/work/notebooks/w04_baseline_score.ipynb?flush_cache=true) |
| 5 | ML-08 | `w05_model` | First model, grouped `client_id` split — this is where the 1.0 accuracy first appeared | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/muhammadabdurrehmanmaqsood/flyrank-ml-internship-abdurrehman/blob/main/work/notebooks/w05_model.ipynb?flush_cache=true) |
| 6 | ML-09 | `w06_validation_audit` | **The leakage audit.** Confession test, fix, honest re-evaluation | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/muhammadabdurrehmanmaqsood/flyrank-ml-internship-abdurrehman/blob/main/work/notebooks/w06_validation_audit.ipynb?flush_cache=true) |
| 7 | ML-10 | `w07_action_playbook` | Ranked content-refresh queue with reason codes | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/muhammadabdurrehmanmaqsood/flyrank-ml-internship-abdurrehman/blob/main/work/notebooks/w07_action_playbook.ipynb?flush_cache=true) |
| 8 | ML-11 | `capstone` | Final write-up: results, limitations, recommendations, the deployed paper's charts | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/muhammadabdurrehmanmaqsood/flyrank-ml-internship-abdurrehman/blob/main/work/notebooks/capstone.ipynb?flush_cache=true) |

Badges not opening *your* copy? Colab's built-in opener always works: **File → Open notebook → GitHub tab** → paste the repo URL → pick the notebook.

---

## Architecture

```text
Reference pipeline (scripts/, on the bundled 30k-row sample):
  01_prepare_features.py    clean + build the feature vector, define the label
  02_baseline_score.py      transparent hand-rule "fix this first" score
  03_train_model.py         logistic regression, decision tree, random forest (client-holdout split)
  04_evaluate_and_export.py ranked queue + charts + Markdown report
  05_build_pdf_report.py    a shareable PDF summary

Capstone workflow (work/notebooks/, the real build):
  w01 research question  →  w02 ML task framing  →  w03 data contract + leakage check
  →  w04 signal audit + baseline  →  w05 first model (client-grouped split, 1.0 accuracy — red flag)
  →  w06 validation audit (confession test, leak found + fixed, honest re-score)
  →  w07 action playbook (final ranked queue + reason codes)
  →  capstone (write-up, charts, deployed paper)
```

The `w06` step is the hinge of the whole project: a near-perfect score triggered a leakage audit instead of a celebration, and everything downstream (the queue, the paper, the numbers below) is built on the model that came out the other side of that audit.

---

## Usage example

Running the reference pipeline against the bundled anonymized sample produces a ranked queue like this (real rows from `outputs/model_report.md`):

| Rank | Score | Action | Reasons | Impressions | Trend |
|---:|---:|---|---|---:|---|
| 1 | 81.6 | `refresh_and_review_ctr` | declining_with_demand, low_ctr_visible_page, low_engagement_visible_page | 12,834 | down |
| 10 | 80.3 | `refresh` | declining_with_demand, model_decline_risk, visible_model_opportunity | 3,867 | down |

Each reason code names *why* a page ranked where it did — a reviewer can see the model's reasoning, not just a bare score. The full queue, model comparison, and top feature importances are in [`outputs/model_report.md`](outputs/model_report.md).

---

## v2 eval results — the honest numbers

**On the bundled 30k-row sample (`w06_validation_audit`):**
- Base rate (majority-class guess): **62.8%**
- Leaky model (pre-audit, all numeric features): precision, recall, and accuracy all **1.000** — the red flag, not the win.
- Honest model (after removing `impressions_last_30d` / `impressions_prev_30d`): accuracy collapses to **~0.67**, precision to **~71–72%** — modest, real signal above the base rate, not near-certain prediction.
- Split: `GroupShuffleSplit` on `client_id`, 80/20, seed 42 — no client appears in both train and test.

**On the full capstone evaluation (`capstone.ipynb`, same leakage-fixed model, read as a ranked queue):**
- Global precision: **71.5%** (close to base rate — trustworthy at the top of the queue, not row-by-row)
- Precision@250: **~86.8%**
- Precision@100: **89.0%** — the number that matters, since the queue is read top-down

The gap between global precision (71.5%) and precision@100 (89%) is the actual point of building a *ranked* queue instead of a single global classifier: the model is far more trustworthy at the top than the global number alone would suggest.

---

## Limitations

- **Single holdout, not cross-validated** — precision@K would move somewhat under a different random seed.
- **Coverage gap** — the client-grouped holdout used for precision@100/250 happened to land entirely on one content type (`keyword article`); the model trained on all three types, but top-K precision is directly demonstrated for only one.
- **Not a calibrated probability** — the model's score ranks well but isn't a literal percentage chance of decline.
- **No time-forward claim** — there's no calendar-date column to support a genuine past-vs-future split; this describes resemblance to past declining content, not a forecast.
- **No causal claim** — this is observational data; the model cannot say that refreshing a page *causes* recovery.
- **Small anonymized sample vs. full release** — the 30k-row modeling slice is drawn from a 79M-row warehouse; conclusions haven't been re-verified at full scale.
- **No live validation yet** — no A/B test against actual content-team outcomes; the queue is a decision-support aid, not a decision.
- Full detail in `work/notebooks/capstone.ipynb`, Section 5.

---

## Data safety (read `DATA_USE.md`)

- Only the small **anonymized** CSV ships here — no client names, domains, URLs, titles, or keywords.
- **Never** add raw private client data to this repo or your fork. Need more data? Request an approved release from your mentor — never export it yourself.
- Don't paste client data into third-party AI tools.
- Frame every result as **observed / measured / directional / decision-support** — never "I predicted Google's algorithm."

The `.gitignore` blocks datasets by default, and CI fails any commit that includes a dataset.

---

## Related work

- **Live capstone paper (deployed):** https://muhammadabdurrehmanmaqsood.github.io/flyrank-ml-internship-abdurrehman/
- **Portfolio site (built and hardened with Claude's help):** https://flyrank.abdurrehman.online
- **AI Fluency track submission package** (retrospective, demo script, build-in-public post, index): [`capstone-package/`](capstone-package/INDEX.md)

---

## Assignments & schedule

Weekly assignments, live events, and the capstone live on the internship portal. This repo is the technical foundation they all build on — the `skills/` folder is the instruction library for AI coding assistants (start at [skills/README.md](skills/README.md)).

*Track leads: Mirza Ašćerić (ML) · Hole (data engineering). Code under MIT (see `LICENSE`); data under `DATA_USE.md`.*
