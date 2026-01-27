# Phase 5 Plan 01: Capture Flow Summary

## One-Liner
Implemented complete capture quiz state machine where CAPTURE_PENDING species trigger a sequential all-questions quiz, with perfect runs adding the Pokémon to party and wrong answers causing flee.

## Tasks Completed

| Task | Description | Status |
|------|-------------|--------|
| 1 | Add capture mode state tracking to QuizState | ✓ Complete |
| 2 | Implement capture quiz initialization | ✓ Complete |
| 3 | Implement capture quiz progression and completion | ✓ Complete |
| 4 | Implement battle outcome hooks for capture mode | ✓ Complete |
| 5 | Add Pokémon to party on capture success | ✓ Complete |
| 6 | Verify compilation | ✓ Complete |

## Files Modified

- `include/quiz/quiz.h` - Added new enum values and function declarations
- `src/quiz/quiz.c` - Capture mode state, init, progression, and party add
- `include/quiz/quiz_hooks.h` - New hook declarations
- `src/quiz/quiz_hooks.c` - Damage override and flow control hooks
- `src/battle_main.c` - Hooks for flee/continue and capture on win

## Key Additions

### New QuizState Fields
```c
bool8 captureMode;        // TRUE during capture quiz
u8 captureProgress;       // 0-based index of current question
u8 captureTotal;          // Total questions for capture quiz
const struct QuizBank *currentBank;  // Bank reference for progression
```

### New Turn Results
```c
QUIZ_TURN_CAPTURE_SUCCESS  // Perfect capture run complete
QUIZ_TURN_CAPTURE_FAIL     // Wrong answer during capture quiz
```

### New Functions
- `Quiz_IsCaptureMode()` - Check if in capture mode
- `Quiz_GetCaptureProgress()` - Get current/total for UI
- `Quiz_InitCaptureQuiz()` - Initialize capture quiz with all questions
- `Quiz_AdvanceCaptureQuestion()` - Move to next question in sequence
- `Quiz_AddCapturedPokemon()` - Add to party on capture success
- `QuizHooks_ShouldForceFlee()` - Check if battle should end with flee
- `QuizHooks_ShouldContinueCaptureQuiz()` - Check if more questions remain
- `QuizHooks_OnBattleWon()` - Handle capture on victory

## State Machine

```
CAPTURE_PENDING encounter:
  └─► Init capture quiz (all questions in sequence)
      └─► Question 1
          ├─► Wrong → CAPTURE_FAIL → flee, stay CAPTURE_PENDING
          └─► Correct → Question 2
              ├─► Wrong → CAPTURE_FAIL → flee
              └─► Correct → ... → Question N
                  ├─► Wrong → CAPTURE_FAIL → flee
                  └─► Correct → CAPTURE_SUCCESS → add to party, set CLEARED
```

## Battle Flow Changes

1. **Damage Override**: 
   - Capture success: 1000 damage (KO)
   - Capture fail: 0 damage (then flee)
   - Correct during capture: 0 damage (no damage, continue quiz)

2. **Action Finished Hook**:
   - Check `ShouldForceFlee()` → set `B_OUTCOME_MON_FLED`
   - Check `ShouldContinueCaptureQuiz()` → skip enemy turn, loop back

3. **Battle Won Hook**:
   - Check for `CAPTURE_SUCCESS` → call `Quiz_AddCapturedPokemon()`

## Deviations from Plan

None - executed as planned.

## Next Phase Readiness

- [x] CAPTURE_PENDING triggers capture quiz
- [x] All questions presented in sequence
- [x] Perfect run adds Pokémon to party
- [x] Wrong answer causes flee
- [x] State correctly set to CLEARED on success
- [ ] Ready for Phase 6: Encounter Filter (skip CLEARED species)
