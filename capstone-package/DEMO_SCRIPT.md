# Demo video script (3–5 minutes) — cell-exact recording guide

*Updated after mapping the exact executable cells in `work/notebooks/capstone.ipynb`. Skip markdown cells (0, 1, 3, 5, 7, 9) — leave them visible for context, but you only need to run cells **2, 4, 6, 8** in order. Everything past cell 9 (chart-building, ranked queue export) is not needed live.*

**Overall voice note:** talk slightly slower and lower than feels natural on camera — the instinct under recording pressure is to speed up, which makes the leakage moment land as a throwaway instead of a discovery.

**Total runtime target: ~4:00** (comfortably inside 3–5 minutes)

---

**0:00–0:20 — Portfolio site.** *(normal pace, warm, conversational)*
Screen: `flyrank.abdurrehman.online`.
> "This is my FlyRank ML internship capstone — a model that ranks a content team's declining pages by which one to review first."
Click into the case-study section. Keep energy up — this is the hook.

**0:20–0:45 — Switch to the notebook, state the decision.** *(normal pace, still warm)*
Screen: `capstone.ipynb` in Colab, Section 1 (Question) visible.
> "A content team has more declining pages than reviewer-hours. This ranks them, so someone opens the highest-value page first instead of guessing."

**0:45–1:15 — Run cell 2.** *(normal pace, matter-of-fact — scale-setting, not the emotional beat yet)*
Output: `Total pages: 30000`, `Pages showing a downward trend: 16262`, `Declining pages with >500 impressions: 9956`.
> "Thirty thousand pages, anonymized, from a 79-million-row production warehouse. About sixteen thousand of them are declining — that's the pool this has to rank."

**1:15–1:40 — Run cell 4.** *(normal pace)*
Output: `32 clients`, 3 content types, the `avg_position == 0` fix note.
> "Thirty-two clients, three content types. This line is a small real-world mess — about four percent of rows had a placeholder zero instead of missing data, which I had to catch and fix before it quietly lied to the model."

**1:40–1:55 — Design decision, spoken before cell 6.** *(slow down slightly; even pitch, not dramatic yet)*
> "Before training, one decision mattered more than the algorithm: I split by `client_id`, not randomly, so no client's pages appear in both the training set and the test set. It's the only split that actually tells you whether this generalizes to a client the model has never seen."

**1:55–2:45 — Run cell 6. THE moment.** *(slow down noticeably; pitch drops — measured, almost quiet, like telling someone a secret, not presenting a slide)*
Output is one table showing "before" and "after" side by side:
```
   Metric  WITH suspects (impressions_*_30d)  WITHOUT suspects
Precision                              1.000             0.716
   Recall                              1.000             0.783
 Accuracy                              1.000             0.669
Base Rate                              0.628             0.628
```
Pause a full second after the table appears before speaking.
> *(low, measured)* "Look at the left column. Precision, recall, accuracy — all 1.000. Perfect. That's not a win. That's a red flag — real-world behavioral data doesn't score perfectly."
*(slight pause)*
> "Two features — `impressions_last_30d` and `impressions_prev_30d` — had coefficients that nearly canceled each other out, negative 7.92 and positive 7.91. That's the fingerprint of a feature mathematically tied to the label instead of genuinely predicting it. So I ran the test you're looking at right now: retrain once with them, once without."
*(point at the right column; slow down further on the numbers)*
> "Without them — accuracy collapses to 0.669. Right at the base rate, plus a little real signal. That's the number I actually trust."

**2:45–3:15 — Run cell 8.** *(pitch lifts back up — this is the resolution, deserves warmth and a little pride)*
Output:
```
Global -- Precision: 0.713 | Recall: 0.801 | Accuracy: 0.673 | Base rate: 0.628
Top-of-queue -- precision@100: 0.890 | precision@250: 0.868
```
> *(warmer, a bit faster)* "Global precision lands at 71.3% — modest, close to the base rate, which is honest. But this queue is read top-down, not row by row. And in the top 100 candidates, precision jumps to 89 percent. That's the number a content team would actually act on."

**3:15–3:35 — AI attribution.** *(normal pace, plain, no dramatics — say it like a fact, not a confession)*
> "I used Claude throughout as a coding partner — including to apply the leakage checklist that first flagged those canceling coefficients. But the confession test you just watched, and the decision to trust the fix, I ran and checked myself."

**3:35–4:00 — Close.** *(normal pace, warm, wind down)*
> "That's the project — a suspicious perfect score, an honest audit, and a queue that's genuinely useful at the top. Links to the repo and the write-up are below."
Hold the repo or site URL on screen for 2–3 seconds before cutting.

---

**Practical note:** cells 6 and 8 run almost instantly (logistic regression on 30k rows, sub-second) — there's no dead air to fill while it "loads." The pause built in at 1:55 is for effect, not because you're waiting on compute. Don't let the fast execution rush your delivery of the table.

---

**Recording checklist:**
- [ ] Screen recording only, live cells actually executing — no slides, no pre-rendered screenshots standing in for a run
- [ ] Both the client-split decision and the leakage limitation are spoken out loud, not just shown as text
- [ ] The Claude-attribution line is said explicitly, once, clearly
- [ ] Runtime lands between 3 and 5 minutes
- [ ] Upload (YouTube unlisted or portal upload) and paste the link into `capstone-package/INDEX.md` and the README's "Related work" section
