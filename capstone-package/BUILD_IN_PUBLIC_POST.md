# Build-in-public post — draft

*Adapts the "Shareable Cut" already drafted in `work/notebooks/capstone.ipynb` (Section 9), expanded to name one real decision alongside the existing limitation, per the AI Fluency track's post requirement. Post this to LinkedIn (or X) once the demo video is live, and link both.*

---

**Draft (≈254 words):**

I spent the last few weeks building a model to rank a content team's declining pages by "fix this one first" — trained on an anonymized slice of real FlyRank search data across 32 clients.

One real decision: I split the data by `client_id`, not randomly, so no client's pages could appear in both training and test. It's the less convenient choice — it makes the split harder to set up — but it's the only one that tells you whether the model generalizes to a client it's never seen, which is the actual use case.

One real limitation: my first version scored 100% precision, recall, and accuracy. That's not a win, it's a red flag — real-world behavioral data doesn't produce perfect scores. A leakage audit found two features whose coefficients nearly canceled out, meaning they were mathematically tied to the label instead of genuinely predictive. I re-trained without them and watched accuracy drop to an honest 67%, right at the base rate plus a real signal.

That honest model is a smaller, less impressive number than 100%. It's also the only one I trust. Its ranked queue hits 89% precision in the top 100 candidates — the number that actually matters, since a content team reads a queue top-down, not the global average.

I used Claude throughout as a coding partner, including to apply the leakage checklist that caught the canceling coefficients — but the confession test that confirmed the leak, and the decision to trust the fix, were mine to run.

Write-up: [capstone paper link]
Repo: [github repo link]
Demo: [video link]

---

**To finalize before posting:** confirm the links above, and pick LinkedIn vs. X — send your preference and I'll tailor tone/length if you want a shorter X version too.
