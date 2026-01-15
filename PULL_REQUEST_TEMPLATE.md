## Summary
<!-- 1–3 sentences: what you changed and why -->

## Issue
Closes #

## Files changed
<!-- List exact files edited -->
- 

## Hook point verification (REQUIRED for engine-touch PRs)
If this PR touches battle/engine internals:
- Hook location (file + function): 
- How verified (search term / call site / brief explanation): 

## Behavior / Rules checklist (QuizHack)
- [ ] I did NOT invent FireRed internals (functions/structs/globals). I verified identifiers exist in this repo.
- [ ] All new feature logic is under `src/quiz/*` (except minimal hook glue).
- [ ] Engine/battle edits are limited to `src/quiz/quiz_hooks.c` (or clearly justified if not).
- [ ] Wild encounter behavior matches spec:
  - [ ] Correct answer: wild Pokémon instantly KO’d, encounter ends.
  - [ ] Wrong answer: player loses HALF of CURRENT HP (floor), clamped to ≥ 1, encounter ends.
- [ ] Mastery → Capture quiz:
  - [ ] After mastering all N: next encounter triggers capture quiz.
  - [ ] Capture quiz asks all N sequentially.
  - [ ] Any wrong: apply wrong penalty, end encounter, capture remains pending.
  - [ ] All correct: grant Pokémon (party/PC) WITHOUT inventory, mark species cleared.
- [ ] Cleared species reroll:
  - [ ] Cleared species do not appear as wild encounters.
  - [ ] Reroll capped at MAX_REROLLS=10; if exceeded, no encounter.

## How to test (REQUIRED)
Provide step-by-step instructions someone else can follow.

1.
2.
3.

## Evidence (RECOMMENDED)
- [ ] Screenshot / short clip (especially for engine-touch)
- [ ] Logs or notes

## Risk / Notes
What could break? Any edge cases?

- 
