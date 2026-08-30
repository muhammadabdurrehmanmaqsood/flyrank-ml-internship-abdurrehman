# Build-in-public post — LinkedIn (final)

*Adapts the "Shareable Cut" originally drafted in `work/notebooks/capstone.ipynb` (Section 9). Finalized for LinkedIn with real links, ready to post.*

---

I spent the last few weeks building a model to rank a content team's declining pages by "fix this one first," trained on an anonymized slice of real search performance data across 32 clients.

One real decision: I split the data by client_id, not randomly, so no client's pages could appear in both training and test. It's the less convenient choice, since it makes the split harder to set up, but it's the only one that tells you whether the model generalizes to a client it has never seen, which is the actual use case.

One real limitation: my first version scored 100% precision, recall, and accuracy. That's not a win, it's a red flag. Real world behavioral data does not produce perfect scores. A leakage audit found two features whose coefficients nearly canceled each other out, meaning they were mathematically tied to the label instead of genuinely predictive. I retrained without them and watched accuracy drop to an honest 67%, right at the base rate plus a real signal.

That honest model is a smaller, less impressive number than 100%. It is also the only one I trust. Its ranked queue hits 89% precision in the top 100 candidates, the number that actually matters, since a content team reads a queue top down, not the global average.

I used Claude throughout as a coding partner, including to apply the leakage checklist that caught the canceling coefficients. The confession test that confirmed the leak, and the decision to trust the fix, were mine to run.

Demo (live run, leakage story explained): https://youtu.be/e-FmiwsWgjM
Write-up: https://muhammadabdurrehmanmaqsood.github.io/flyrank-ml-internship-abdurrehman/
Repo: https://github.com/muhammadabdurrehmanmaqsood/flyrank-ml-internship-abdurrehman

---

**Status:** ready to copy-paste to LinkedIn. Once posted, send the post URL back so it can be linked from `INDEX.md` and the README.
