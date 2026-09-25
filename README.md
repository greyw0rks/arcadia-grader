# Arcadia Grader

A GenLayer **Intelligent Contract** that grades free-text quiz answers against a
plain-language rubric — and reaches on-chain consensus on the verdict through AI
validators, with no trusted backend in the loop.

Register a question and a rubric, submit an answer, and the network itself
decides whether the answer passes. Correct answers accrue points on an on-chain
leaderboard.

## Why GenLayer

Grading a subjective answer is a *judgement call*, not a deterministic
computation. Traditionally you solve that with a trusted off-chain service: a
backend reads the answer, decides pass/fail, and signs the result so a contract
will accept it. That signer is a single point of trust — whoever holds its key
can grade however they like.

GenLayer removes that. Its Intelligent Contracts can run a non-deterministic LLM
call inside the **Equivalence Principle**, so multiple validators each grade the
answer independently and consensus is reached on an *equivalent* verdict. The
judgement lives on-chain. No oracle, no backend, no single signer.

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

> Arcadia Grader is a small proof-of-concept inspired by Arcadia, a quiz
> platform on Celo. Arcadia's answer grading today runs through a centralized
> backend that signs off on correctness (an EIP-712 trusted signer). This
> project explores what that grading step looks like as a GenLayer Intelligent
> Contract instead — where AI validators reach on-chain consensus on whether an
> answer passes a rubric, removing the need for a trusted backend signer. It's a
> standalone experiment, not a port of Arcadia itself.

## Contract surface

`contracts/arcadia_grader.py`

| Method | Kind | What it does |
|---|---|---|
| `create_question(question_id, question, rubric)` | write | Register a question + grading rubric. Rejects duplicate ids and empty fields. |
| `submit_answer(question_id, answer)` | write | Grade `answer` against the rubric via LLM + Equivalence Principle, store the verdict, credit the leaderboard on a pass. Returns the verdict. |
| `get_question(question_id)` | view | The question + rubric + creator (the rubric is deliberately public — grading is transparent). |
| `get_submission(submission_id)` | view | A stored submission and its verdict. |
| `get_leaderboard_entry(address)` | view | Cumulative score for an address. |
| `get_leaderboard()` | view | The full address → score map. |

State: `questions`, `submissions`, `leaderboard` (all `TreeMap`s) and a
`submission_count` counter. The leaderboard is a mocked, points-style tally — no
real token movement.

### Defensive design

- **Malformed verdict never breaks consensus.** The LLM is asked for strict JSON
  (`response_format="json"`); parsing the verdict is wrapped so any bad/missing
  field records `graded=False` on the submission instead of reverting the
  validator round. `score` is clamped to `0–100`.
- **Prompt-injection resistant.** The grading prompt instructs the model to
  judge the answer against the rubric and ignore instructions embedded in the
  answer text.
- **Guard rails.** Duplicate `question_id` is rejected; submitting to a
  nonexistent question is rejected; empty inputs are rejected.

## Consensus wrapper (resolved open item)

The build spec flagged whether to use `strict_eq` for the grading call. **Do
not.** `strict_eq` demands byte-for-byte identical validator output, which never
holds for free-form LLM text and breaks consensus. Grading is a *subjective
assessment*, so the correct primitive is
`gl.eq_principle.prompt_comparative(fn, principle)` — validators converge on an
equivalent verdict (same pass/fail, similar score). This matches GenLayer's own
guidance and reference contracts. Current LLM call syntax:
`gl.nondet.exec_prompt(prompt, response_format="json")`.

## Testing (GenLayer Studio simulator)

`tests/test_arcadia_grader.py` covers the four judgement cases — clear pass,
clear fail, borderline/partial (validators converging on ambiguity, the case to
screenshot), and adversarial/junk input — plus the duplicate-id and
missing-question rejections.

```bash
# 1. start GenLayer Studio (local, or use https://studio.genlayer.com)
# 2. select a network
genlayer network        # studionet / localnet / testnet
# 3. run the suite from the repo root
gltest
```

The grading path is non-deterministic, so `submit_answer` calls run with extra
finalization time (`wait_interval` / `wait_retries`).

## Deploy

GenLayer's **Bradbury** testnet is the current active network for deploying
Intelligent Contracts (chain id `4221`, RPC `https://rpc-bradbury.genlayer.com`);
Asimov is reserved for infrastructure/stress testing. Deploy from the Studio UI
or the `genlayer` CLI against the selected network, then record the deployment
transaction hash.

> Deployment tx hash: _`<fill in after deploying>`_

## Layout

```
arcadia-grader/
├── contracts/arcadia_grader.py     # the Intelligent Contract
├── tests/test_arcadia_grader.py    # Studio simulator test cases
├── docs/demo-script.md             # 2–3 min demo walkthrough
└── README.md
```

## Scope

A GenLayer Portal Builders-track proof of concept. No frontend, no real token
mechanics, no mainnet — just the grading mechanic done correctly and
demonstrably on-chain.

