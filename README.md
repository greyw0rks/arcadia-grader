# Arcadia Arcade — on-chain skill XP for GenLayer

A GenLayer **Intelligent Contract** that turns quiz-style skill into portable,
wallet-tied XP — graded by AI validator consensus, not by a trusted backend.

Think of it as the GenLayer-native equivalent of a Discord trivia bot. Anyone
can **host** a game room (a set of questions + grading rubrics), anyone can
**join and play**, answers are graded on-chain by GenLayer's non-deterministic
LLM consensus, and XP accrues to the player's wallet.

## Positioning — native XP vs. activity XP

GenLayer's community levels today (Molecule → Synapse → Singularity) are earned
through Discord *activity*: messages, voice time, POAPs. There's no way to earn
standing by demonstrating actual knowledge, and no way for a member to host
their own game the way a server owner spins up a trivia channel.

Arcadia Arcade is that missing layer: XP earned by **demonstrated skill**,
graded by **judgement** (LLM validator consensus) rather than message count, and
**portable** — it lives on the player's wallet, provable and not locked inside
one Discord server. Hosting is permissionless: anyone opens a room, anyone plays.

## Why GenLayer

Grading a free-text answer is a *judgement call*, not a deterministic
computation. The usual fix is a trusted off-chain service that decides pass/fail
and signs the result — a single point of trust (this is exactly how the original
Arcadia works today, via an EIP-712 `trustedSigner`). GenLayer removes it: an LLM
call runs inside the **Equivalence Principle**, so validators each grade
independently and reach consensus on an *equivalent* verdict. The judgement lives
on-chain — no oracle, no backend, no single signer.

```python
# subjective grading -> comparative equivalence, NOT strict_eq
principle = (
    "The 'pass' boolean must be identical. The 'score' integers must be within "
    "15 points of each other. The 'reasoning' may differ in wording but must "
    "support the same pass/fail verdict."
)
verdict = gl.eq_principle.prompt_comparative(grade, principle)
```

## The Arcadia inspiration

> Arcadia Arcade grows out of Arcadia, a skill-based quiz arcade on Celo whose
> answer grading runs through a centralized backend that signs off on
> correctness (an EIP-712 trusted signer). This project reimagines that grading
> step as a GenLayer Intelligent Contract — AI validators reach on-chain
> consensus on whether an answer passes a rubric — and wraps it in a
> community-XP game layer. It's a standalone testnet MVP, not a port of Arcadia.

## Contract surface

`contracts/arcadia_arcade.py` — contract class `ArcadiaArcade`.

| Method | Kind | What it does |
|---|---|---|
| `create_room(room_id, title, is_private, access_code="")` | write | Host opens a room. Private rooms store a **sha256 hash** of the access code, never the raw code. |
| `add_question(room_id, question_id, question, rubric)` | write | Host-only, open-room-only. Adds a rubric-graded question. |
| `join_room(room_id, access_code="")` | write | Join a room. Private rooms verify the hashed code first. Required before playing. |
| `submit_answer(room_id, question_id, answer)` | write | Grade via LLM + Equivalence Principle, store the verdict, credit room + global XP on a pass. Returns the verdict. |
| `close_room(room_id)` | write | Host-only. Locks further joins and submissions. |
| `get_room` / `get_room_questions` / `get_room_leaderboard` | view | Room state, its questions, and per-room scores. |
| `get_global_xp(address)` | view | Cumulative XP across all rooms for a wallet. |
| `list_public_rooms()` | view | Open, non-private rooms with host + question/player counts. |
| `get_submission(submission_id)` | view | A stored submission and its verdict. |

State (all `TreeMap`s): `rooms`, `questions`, `room_question_ids`,
`submissions`, `room_leaderboard` (nested per room), `global_xp`, `room_players`
(join set), `answered` (rate-limit set), and a `submission_count` counter. XP is
a points tally — no real token movement.

### Design decisions & guard rails

- **Join required for every room** (public included) so player tracking and
  counts are consistent — resolves the spec's open design question.
- **Single submission per (question, player).** A graded answer consumes the
  attempt, so the leaderboard can't be gamed by resubmitting until a pass. A
  malformed/ungraded verdict does *not* consume the attempt (fair retry).
- **Access codes are hashed** (`hashlib.sha256`) — the raw code never touches
  contract state. Private rooms are excluded from `list_public_rooms`.
- **Host-only** enforcement on `add_question` / `close_room`; join-gating and
  closed-room checks on `submit_answer`.
- **Malformed verdict never breaks the room.** The LLM is asked for strict JSON
  (`response_format="json"`); parsing is wrapped so a bad/missing field records
  `graded=False` instead of reverting. `score` is clamped `0–100`.
- **Prompt-injection resistant** grading prompt (judge the answer against the
  rubric; ignore instructions embedded in the answer).

## Consensus wrapper (resolved open item)

The spec flagged `strict_eq` for the grading call. **Don't use it** —
`strict_eq` demands byte-identical validator output, which never holds for
free-form LLM text and breaks consensus. Grading is a *subjective assessment*, so
the correct primitive is `gl.eq_principle.prompt_comparative(fn, principle)`.
LLM call syntax: `gl.nondet.exec_prompt(prompt, response_format="json")`. Both
verified against GenLayer's current docs and reference contracts.

## Testing (GenLayer Studio simulator)

`tests/test_arcadia_arcade.py` covers: public-room join + pass/fail, private-room
access control, multi-question / multi-player XP accumulation, borderline
convergence, adversarial-input safety, and the host-only / join-gating /
closed-room / single-submission guard rails.

```bash
# 1. start GenLayer Studio (local, or use https://studio.genlayer.com)
genlayer network        # studionet / localnet / testnet
# 2. run the suite from the repo root
gltest
```

Grading is non-deterministic, so `submit_answer` calls run with extra
finalization time (`wait_interval` / `wait_retries`). Tests use the `accounts`
fixture to exercise multiple players against one deployed contract.

## Deploy

**Bradbury** is the current active testnet for Intelligent Contracts (chain id
`4221`, RPC `https://rpc-bradbury.genlayer.com`); Asimov is infra/stress testing.
Deploy from the Studio UI or the `genlayer` CLI against the selected network,
then record the deployment transaction hash.

> Deployment tx hash: _`<fill in after deploying>`_

## Layout

```
arcadia-grader/                 # (repo name kept; project is "Arcadia Arcade")
├── contracts/arcadia_arcade.py     # the Intelligent Contract
├── tests/test_arcadia_arcade.py    # Studio simulator test cases
├── docs/demo-script.md             # host → 2 players → grade → leaderboard demo
└── README.md
```

## Scope & roadmap

Testnet MVP for the GenLayer Portal Builders track. **In scope now:** the full
room-based grading contract + Studio tests. **Next:** a minimal web frontend
(wallet connect + host/play/leaderboard/profile screens via genlayer-js) pointed
at the deployed contract. **Not in scope:** token rewards / real staking,
matchmaking, mainnet.

