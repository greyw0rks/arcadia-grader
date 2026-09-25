# Demo script — Arcadia Arcade (2–3 min)

Goal: show a host spinning up a private game room, two players joining and
answering, answers graded **by validator consensus** on chain, and XP landing on
the players' wallets — no trusted backend.

## Setup (before recording)
- GenLayer Studio open (local or https://studio.genlayer.com), a network
  selected, `contracts/arcadia_arcade.py` deployed. Have the contract address /
  deploy tx hash on screen.
- Three accounts ready: a **host** and two **players** (P1, P2).

## Beat 1 — Framing (~20s)
"GenLayer community XP today comes from Discord activity — messages, voice time.
This is an on-chain XP layer earned by *skill*: anyone hosts a game, answers are
graded by AI validator consensus, and XP lands on your wallet. No backend signer."

## Beat 2 — Host opens a private room (~30s)
As the host:
- `create_room("genlayer-101", "GenLayer 101", true, "sesame-42")` — private.
- `add_question("genlayer-101", "q1", "What is the capital of France, and what
  river runs through it?", "Must name Paris AND the Seine. Both required.")`
Show the txs succeed; call `get_room` to show status `open`, `question_count 1`.
Note the room does **not** appear in `list_public_rooms` (it's private).

## Beat 3 — Players join with the code (~20s)
- P1 `join_room("genlayer-101", "sesame-42")` → succeeds.
- Show a wrong code rejected: `join_room("genlayer-101", "nope")` → fails.
- P2 `join_room("genlayer-101", "sesame-42")` → succeeds.

## Beat 4 — Play + live grading (the key moment) (~40s)
- P1 `submit_answer("genlayer-101", "q1", "Paris, and the Seine runs through it.")`.
  Open the transaction's consensus view: show multiple validators independently
  grading and **agreeing** on `pass: true`, high score. This is the screenshot.
- P2 `submit_answer("genlayer-101", "q1", "Berlin, on the Nile.")` → `pass: false`.
- (Optional borderline) a third answer `"It's Paris."` — show validators still
  **converging** despite the answer being only half-right.

## Beat 5 — Leaderboard + portable XP (~25s)
- `get_room_leaderboard("genlayer-101")` — P1 scored, P2 at zero.
- `get_global_xp(P1)` — the same points on P1's wallet, portable across rooms.
- Try P1 re-answering q1 → rejected (one submission per question; leaderboard
  can't be farmed).

## Beat 6 — Close (~15s)
Host `close_room("genlayer-101")`. Show a further `submit_answer` now rejected.
Close on: "The game, the grade, the consensus, and the XP all live on chain."

## Shot list to capture
- [ ] `create_room` (private) + `add_question` succeed
- [ ] wrong access code rejected / correct code joins
- [ ] `submit_answer` pass — multi-validator consensus view
- [ ] `submit_answer` fail — verdict false
- [ ] borderline convergence view (optional)
- [ ] room leaderboard + `get_global_xp` reflecting only passes
- [ ] re-submission and post-close submission both rejected
- [ ] deployment tx hash on screen
