# ENH-E-01: Encounter Initiation Label - Summary

## Completed: 2026-01-27

## Objective
Display "Quiz Encounter!" or "Catch Opportunity!" label at the start of wild battles, after "Go! [Pokemon]!" message.

## Implementation

### New String IDs (`battle_string_ids.h`)
```c
#define STRINGID_QUIZ_TEST 386
#define STRINGID_QUIZ_CATCH_OPPORTUNITY 387
#define BATTLESTRINGS_COUNT 388
```

### New Strings (`battle_message.c`)
```c
static const u8 sText_QuizEncounter[] = _("Quiz Encounter!");
static const u8 sText_QuizCatchOpportunity[] = _("Catch Opportunity!");
```

### Quiz Label Logic (`quiz.c`)
Added `Quiz_GetEncounterLabelStringId()` function:
- Returns `STRINGID_QUIZ_TEST` for normal quiz encounters
- Returns `STRINGID_QUIZ_CATCH_OPPORTUNITY` for capture mode
- Returns 0 if quiz not active

### Battle Flow Integration (`battle_main.c`)
Modified battle intro sequence:
1. `BattleIntroPlayerSendsOutMonAnimation` → `BattleIntroPrintQuizTest` (was → `TryDoEventsBeforeFirstTurn`)
2. `BattleIntroPrintQuizTest` now:
   - Initializes quiz state for wild battles via `QuizHooks_OnWildBattleStart()`
   - Displays appropriate label via `PrepareStringBattle()`
   - Proceeds to `TryDoEventsBeforeFirstTurn`

### Files Modified
- `pokefirered/include/constants/battle_string_ids.h`
- `pokefirered/src/battle_message.c`
- `pokefirered/include/quiz/quiz.h`
- `pokefirered/src/quiz/quiz.c`
- `pokefirered/src/battle_main.c`

## Additional Work: Debug Quick Start Mode

Implemented a development feature to speed up testing iterations.

### Files Modified
- `pokefirered/src/new_game.c` - Player setup (name, Pokemon, flags, warp)
- `pokefirered/src/oak_speech.c` - Auto-advance through intro screens

### Features
1. Auto-advances through Controls Guide and Pikachu intro
2. Instant text speed for Oak's dialogue
3. Auto-selects BOY gender and default names
4. Sets player name to "RED" with level 5 Charmander
5. Sets flags to bypass Oak stopping player
6. Gives 10 Poke Balls for testing
7. Warps to Pallet Town near Route 1

### Activation
Set `#define DEBUG_QUICK_START 1` in both files (set to 0 for release).

## Verification
- [x] Wild battle shows "Quiz Encounter!" after "Go! Charmander!"
- [x] Capture mode shows "Catch Opportunity!" instead
- [x] Label appears before action menu
- [x] Debug quick start enables rapid testing

## Notes
- Wild battles are identified as `!(gBattleTypeFlags & BATTLE_TYPE_TRAINER)` (FireRed has no `BATTLE_TYPE_WILD` flag)
- Quiz initialization moved earlier in battle flow for wild battles
- Trainer battles still initialize quiz at action selection (for multi-Pokemon support)
