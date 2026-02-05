# 15-04 Summary: Quiz Logic Reversion

## Completed: 2026-02-05

## What Was Done

Reverted quiz logic from topic-pool selection to species-bank selection for wild encounters and single-question trainer battles.

### Changes Made

1. **Quiz_InitWildEncounter** — Species bank selection:
   - Wild encounters now use `Quiz_GetBank(species)` directly
   - Removed topic pool selection logic for wild battles
   - Questions randomly selected from species bank: `Random() % bank->count`

2. **Single-question trainers** — Species bank with fallback:
   - Prioritizes species bank via `Quiz_GetBank(species)`
   - Falls back to "full pool" if no species bank available
   - Maintains consistency with wild encounter logic

3. **Quiz_OnMoveConfirmed** — Global mastery updates:
   - Uses `Quiz_GetMasteryCount` / `Quiz_SetMasteryCount` directly
   - No section parameter needed (global tracking)
   - Learning mode (wild encounters) increments mastery on correct answer

4. **Quiz_GetMasteryProgress** — Global display:
   - Returns global mastery count for progress display
   - Shows X/5 for wild encounters (learning mode)

5. **Quiz_GetEncounterType** — Multi-question trainer fix:
   - Returns `QUIZ_ENCOUNTER_GYM` for any multi-question trainer (`isGymLeader == TRUE`)
   - This includes gym trainers, gym leaders, rivals, bosses
   - Ensures UI displays turn progress (X/N) not mastery progress (X/5)
   - Single-question trainers return `QUIZ_ENCOUNTER_TRAINER`

6. **Deprecated functions**:
   - `Quiz_SelectUnmasteredQuestion` marked as `UNUSED static` with deprecation comment
   - Topic pool functions no longer called for wild/trainer encounters

### Bug Fixes

- **Trainer progress display**: Fixed `Quiz_GetEncounterType()` to check `isGymLeader` flag (multi-question indicator) rather than specific trainer tier
- This ensures all multi-question encounters show turn-based progress

## Files Modified

- `pokefirered/src/quiz/quiz.c`

## How Questions Work Now

**Wild Encounters:**
1. Player encounters wild Pokémon
2. Question selected randomly from species' bank via `Quiz_GetBank(species)`
3. Correct answer → mastery count incremented globally
4. 5 correct answers → capture mode triggered
5. Progress displayed as X/5

**Single-Question Trainers:**
1. Player battles trainer
2. Question selected from opposing Pokémon's species bank
3. Fallback to full pool if no bank exists
4. No mastery tracking (just win/lose)
5. Progress displayed based on trainer tier

**Multi-Question Trainers (Gym trainers, leaders, rivals):**
1. Player battles trainer with `isGymLeader == TRUE`
2. Multiple questions per Pokémon
3. Progress displayed as X/N (turns completed)
4. Different selection logic (gym pool functions)

## Testing

- Build passes with `make -j4`
- Wild encounters select questions from species bank
- Mastery progress increments correctly (0/5 → 5/5 → capture)
- Trainer battles show correct progress type
