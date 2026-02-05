# 15-03 Summary: Save System Simplification

## Completed: 2026-02-05

## What Was Done

Refactored the quiz save system from section-aware tracking to global species tracking.

### Changes Made

1. **global.h** — QuizSaveDataV2 simplified to V3 format:
   - Added `QUIZ_CLEARED_BYTES 20` and `QUIZ_MAX_SPECIES 152` constants
   - Changed `speciesCleared[8][20]` to `speciesCleared[20]` (global bit array for 160 species)
   - Changed `masteryMask[8][28]` to `masteryCount[152]` (per-species counter, not per-section)
   - Struct name kept as `QuizSaveDataV2` for compatibility

2. **quiz.h** — Updated accessor declarations:
   - Removed `section_config.h` include
   - Removed section-aware function declarations
   - Added global accessors: `Quiz_GetMasteryCount`, `Quiz_SetMasteryCount`, `Quiz_SetSpeciesCleared`
   - Added legacy macros: `Quiz_GetMasteryMask` → `Quiz_GetMasteryCount`, `Quiz_SetMasteryMask` → `Quiz_SetMasteryCount`

3. **quiz.c** — Implemented global mastery functions:
   - `Quiz_IsSpeciesCleared_Internal()` — checks bit in speciesCleared array
   - `Quiz_SetSpeciesCleared()` — sets bit in speciesCleared array
   - `Quiz_GetMasteryCount()` — returns masteryCount for species
   - `Quiz_SetMasteryCount()` — sets masteryCount for species
   - `Quiz_GetSpeciesState()` — derives state from mastery count
   - `Quiz_InitSaveDataV3()` — initializes V3 format save data
   - Added forward declarations for all global mastery functions

### Bug Fixes

- **Symbol duplication error**: Removed redundant wrapper function definitions for `Quiz_GetMasteryMask` and `Quiz_SetMasteryMask` since macros in quiz.h handle aliasing
- **Implicit declaration error**: Added forward declarations for `Quiz_GetMasteryCount`, `Quiz_SetMasteryCount`, `Quiz_GetSpeciesState`, `Quiz_SetSpeciesState`, `Quiz_SetSpeciesCleared`

## Files Modified

- `pokefirered/include/global.h`
- `pokefirered/include/quiz/quiz.h`
- `pokefirered/src/quiz/quiz.c`

## Testing

- Build passes with `make -j4`
- Mastery tracking works correctly in wild encounters
- Species cleared state persists across sessions
