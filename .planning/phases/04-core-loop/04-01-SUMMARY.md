# Phase 4 Plan 01: Core Loop Enhancement Summary

## One-Liner
Added mastery filtering so answered questions don't repeat, with automatic state transition to CAPTURE_PENDING when all questions mastered.

## Tasks Completed

| Task | Description | Status |
|------|-------------|--------|
| 1 | Add Quiz_CountUnmasteredQuestions helper | ✓ Complete |
| 2 | Add Quiz_SelectUnmasteredQuestion helper | ✓ Complete |
| 3 | Update Quiz_InitWildEncounter with mastery filtering | ✓ Complete |
| 4 | Check for all-mastered after correct answer | ✓ Complete |

## Files Modified

- `src/quiz/quiz.c` - Added helpers and updated core functions

## Key Functions Added

```c
// Count unmastered questions for a species
static u8 Quiz_CountUnmasteredQuestions(u16 species, const struct QuizBank *bank);

// Select random question from unmastered pool
static u8 Quiz_SelectUnmasteredQuestion(u16 species, const struct QuizBank *bank);
```

## State Transitions

```
NONE ─────────────► MASTERING ─────────────► CAPTURE_PENDING
     (first encounter)      (all questions correct)
```

## Logic Summary

**Quiz_InitWildEncounter:**
- Skip if CLEARED
- Skip if CAPTURE_PENDING (Phase 5 handles)
- Select from unmastered questions only
- Transition to CAPTURE_PENDING if none left

**Quiz_OnMoveConfirmed:**
- Set mastery bit on correct answer
- Check if all mastered → set CAPTURE_PENDING

## Deviations from Plan

None - executed as planned.

## Next Phase Readiness

- [x] Questions don't repeat once mastered
- [x] State transitions work correctly
- [x] CAPTURE_PENDING detection complete
- [ ] Ready for Phase 5: Capture Flow
