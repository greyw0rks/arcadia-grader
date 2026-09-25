# Demo script — Arcadia Grader (2–3 min)

Goal: show a subjective quiz answer being graded **by validator consensus**, on
chain, with no trusted backend — and the leaderboard updating as a result.

## Setup (before recording)
- GenLayer Studio open (local or https://studio.genlayer.com), a network
  selected, and the account funded on that network.
- `contracts/arcadia_grader.py` deployed; have the contract address / tx hash on
  screen or in the clipboard.

## Beat 1 — Framing (~20s)
One line: "Arcadia grades quiz answers today with a centralized signer deciding
what's correct. Here that judgement is an on-chain GenLayer Intelligent Contract
— AI validators reach consensus on the verdict, no backend."

## Beat 2 — Create a question (~20s)
Call `create_question`:
- `question_id`: `capital-of-france`
- `question`: `What is the capital of France, and what river runs through it?`
- `rubric`: `A passing answer must name Paris as the capital AND the Seine as
  the river. Both are required.`

Show the tx succeed; call `get_question` to show it stored.

## Beat 3 — Submit a passing answer (~30s)
Call `submit_answer(capital-of-france, "The capital is Paris and the Seine flows
through it.")`.

**This is the key moment:** open the transaction's consensus view and show the
multiple validators independently grading and agreeing on the verdict
(`pass: true`, high score). Call `get_submission("sub_1")` to show the stored
verdict + reasoning.

## Beat 4 — Submit a failing answer (~20s)
Call `submit_answer(capital-of-france, "It's Berlin, on the Nile.")`.
Show `pass: false`. Note the leaderboard did **not** move for this one.

## Beat 5 — Borderline case (optional, great to show) (~25s)
Call `submit_answer(capital-of-france, "It's Paris.")` — only half the rubric is
met. Open the consensus view: the interesting part is that despite the ambiguity
the validators still **converge on one verdict** rather than diverging. This is
the screenshot worth keeping.

## Beat 6 — Leaderboard (~15s)
Call `get_leaderboard_entry(<your address>)` (or `get_leaderboard`) and show the
score reflects only the passing submission(s). Close on: "The grade, the
consensus, and the score all live on chain — no signer, no oracle."

## Shot list to capture
- [ ] `create_question` tx success
- [ ] `submit_answer` (pass) — multi-validator consensus view
- [ ] `submit_answer` (fail) — verdict false
- [ ] borderline consensus view (the convergence screenshot)
- [ ] leaderboard reflecting the pass
- [ ] deployment tx hash on screen at some point
