# Capstone submission package — AI Fluency track (Assignments 8.1 / 8.2)

One place, per Assignment 8.2: every deliverable for the whole track, linked below. The technical substance (model, notebooks, eval results) lives in the repo root; this folder is the track-specific paperwork — retrospective, demo script, build-in-public post — kept out of the main project structure so the repo still reads as an ML project first.

## Deliverables

| Deliverable | Status | Link |
|---|---|---|
| README (what it does, setup, usage, architecture, eval results, limitations, AI transparency) | ✅ Done | [`../README.md`](../README.md) |
| Capstone paper (deployed) | ✅ Live | https://muhammadabdurrehmanmaqsood.github.io/flyrank-ml-internship-abdurrehman/ |
| Portfolio site | ✅ Live | https://flyrank.abdurrehman.online |
| Demo video (3–5 min, live run, one decision, one limitation) | ⬜ Script ready, needs recording | [`DEMO_SCRIPT.md`](DEMO_SCRIPT.md) → link once recorded |
| Retrospective (500–800 words) | ✅ Done | [`RETROSPECTIVE.md`](RETROSPECTIVE.md) |
| Build-in-public post | ⬜ Draft ready, needs publishing | [`BUILD_IN_PUBLIC_POST.md`](BUILD_IN_PUBLIC_POST.md) → link once posted |
| Hours log | ⬜ Complete in portal, cross-check against commit history | internship.flyrank.ai |
| Final review checkpoint | ⬜ Submit once the above are live | internship.flyrank.ai |

## The story in one line

An early model scored a suspicious 100% accuracy; a leakage audit traced it to two label-sibling features, and the honest model that replaced it — 71.5% global precision, 89% precision in the top 100 of the ranked queue — is the one actually shipped. Every deliverable above tells some version of that same, true story, at a different length: the README tells it technically, the retrospective tells it reflectively, the demo shows it live, and the post tells it in 250 words.

## AI transparency, stated once, referenced everywhere

Claude was used throughout as a coding and research partner — most notably to apply the leakage-hunting checklist that flagged the canceling coefficients (`impressions_last_30d`, `impressions_prev_30d`) during the Week 6 validation audit. The confession test that confirmed the leak, and the decision to trust the fix, were run and checked by hand. Full detail in the README's "Built with Claude" section.
