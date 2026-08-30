# Demo video script (3–5 minutes)

*Builds on the "Five-Minute Demo Outline" already drafted in `work/notebooks/capstone.ipynb` (Section 8), extended to cover both the AI Fluency track's requirements (show the live portfolio, name where AI did the heavy lifting) and Assignment 8.1's requirements (live run, one design decision, one limitation, no slides). Record this yourself, then drop the hosted link into `capstone-package/INDEX.md` and the README.*

**Total runtime target: 3:30–4:30**

---

**0:00–0:20 — Open on the live site, not a slide.**
Screen-share `flyrank.abdurrehman.online`. Say what it is in one sentence: "This is my FlyRank ML internship capstone — a model that ranks a content team's declining pages by which one to review first." Click through to the case-study section briefly.

**0:20–0:40 — State the decision the model supports.**
"A content team has more declining pages than reviewer-hours. This ranks them, so someone opens the highest-value page first instead of guessing." (Pulled directly from `capstone.ipynb`, Section 1.)

**0:40–1:30 — Live run, not a result screenshot.**
Open `work/notebooks/capstone.ipynb` in Colab (or terminal: `python scripts/run_all.py`) and run the evaluation cell live on screen. Narrate while it runs: "This is training on a 90-day anonymized slice — 32 clients, 30,000 rows drawn from a 79-million-row production warehouse."

**1:30–2:15 — One design decision, explained on camera.**
"I split this by `client_id`, not randomly — grouped 80/20, so no client's pages appear in both training and test. It's the less convenient split to set up, but it's the only one that actually tells you whether this generalizes to a client the model has never seen." Show the `GroupShuffleSplit` line in the notebook as you say it.

**2:15–3:15 — One limitation, explained on camera. (The strongest material in this whole video.)**
"The first version of this model scored 100% precision, recall, and accuracy. That's not a win — it's a red flag." Show the leaked coefficients (`impressions_last_30d` -7.92, `impressions_prev_30d` +7.91) in the notebook. "Those two nearly cancel out — that's the signature of a feature mathematically tied to the label. I ran a confession test — retrained once with them, once without — and accuracy collapsed from 1.0 to about 0.67. That's the number I actually trust." Show the before/after numbers on screen.

**3:15–3:45 — Where Claude did the heavy lifting, named specifically.**
"I used Claude to apply the leakage-hunting checklist that flagged the canceling coefficients in the first place — but I ran the confession test myself and checked the collapse before I trusted the fix. Claude also helped build the ranked-queue notebook and harden this portfolio site." (This single beat satisfies both the README's AI-transparency requirement and the AI Fluency track's "show where AI did the heavy lifting" criterion — say it once, clearly, here.)

**3:45–4:15 — The honest result and close.**
"Global precision lands at 71.5% — modest, close to the 62.8% base rate. But the top 100 of the ranked queue hits 89% precision, and since a content team reads this queue top-down, that's the number that matters." Close on the live site or repo URL on screen.

---

**Recording checklist:**
- [ ] Screen recording only, live cells actually executing — no slides, no pre-rendered screenshots standing in for a run
- [ ] Both the client-split decision and the leakage limitation are spoken out loud, not just shown as text
- [ ] The Claude-attribution line is said explicitly, once, clearly
- [ ] Runtime lands between 3 and 5 minutes
- [ ] Upload (YouTube unlisted or portal upload) and paste the link into `capstone-package/INDEX.md` and the README's "Related work" section
