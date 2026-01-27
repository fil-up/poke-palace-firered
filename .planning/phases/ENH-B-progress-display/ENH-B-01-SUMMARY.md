# ENH-B-01: Progress X/Y Display - Summary

## Completed: 2026-01-27

## Objective
Display quiz progress and encounter type labels in the move info panel (bottom-right area) during Fight select.

## Implementation

### Move Info Panel Layout
The PP and TYPE windows are repurposed for quiz information:

```
+------------------+
| Progress  2/5    |  <- Was "PP 25/25", now shows mastery/encounter progress
| Wild Question    |  <- Was "TYPE/NORMAL", now shows encounter type label
+------------------+
```

### New Functions

**`quiz.h` / `quiz.c`**
```c
void Quiz_GetProgress(u8 *current, u8 *total);      // Encounter progress (capture/gym)
void Quiz_GetMasteryProgress(u8 *mastered, u8 *total); // Mastery progress (wild/trainer)
u8 Quiz_GetEncounterType(void);                      // Returns encounter type enum
```

**`quiz_hooks.h` / `quiz_hooks.c`**
```c
void QuizHooks_GetProgress(u8 *current, u8 *total);
void QuizHooks_GetMasteryProgress(u8 *mastered, u8 *total);
u8 QuizHooks_GetEncounterType(void);

enum QuizEncounterType {
    QUIZ_ENCOUNTER_NONE,
    QUIZ_ENCOUNTER_WILD,      // Regular wild encounter (learning mode)
    QUIZ_ENCOUNTER_CAPTURE,   // Capture opportunity
    QUIZ_ENCOUNTER_GYM,       // Gym leader battle
    QUIZ_ENCOUNTER_TRAINER    // Regular trainer battle
};
```

### Modified Functions

**`battle_controller_player.c`**

`MoveSelectionDisplayPpString()`:
- Shows "Progress" for quiz encounters
- Shows "PP" for non-quiz battles

`MoveSelectionDisplayPpNumber()`:
- Wild/Trainer: Shows mastery progress (questions mastered / total for species)
- Capture/Gym: Shows encounter progress (questions answered / required)
- Non-quiz: Shows normal PP numbers

`MoveSelectionDisplayMoveType()`:
- Shows encounter type label for quiz battles
- Shows TYPE/name for non-quiz battles

### Files Modified
- `pokefirered/include/quiz/quiz.h` - Added function declarations
- `pokefirered/src/quiz/quiz.c` - Added Quiz_GetProgress, Quiz_GetMasteryProgress, Quiz_GetEncounterType
- `pokefirered/include/quiz/quiz_hooks.h` - Added wrapper declarations and enum
- `pokefirered/src/quiz/quiz_hooks.c` - Added wrapper implementations
- `pokefirered/src/battle_controller_player.c` - Modified PP and TYPE display functions

## Display by Encounter Type

| Encounter Type | Top Row (was PP) | Bottom Row (was TYPE) |
|----------------|------------------|----------------------|
| Wild (learning) | Progress 2/5 (mastered/total) | Wild Question |
| Capture opportunity | Progress 0/5 (answered/required) | Capture Quiz |
| Gym battle | Progress 0/2 (answered/required) | Gym Test |
| Trainer battle | Progress X/Y (mastered/total) | Trainer Quiz |
| Non-quiz (Safari) | PP 25/25 | TYPE/NORMAL |

## Example Display

**Wild encounter (learning mode, 2 of 5 questions mastered):**
```
Progress   2/ 5
Wild Question
```

**Capture opportunity (0 of 5 answered):**
```
Progress   0/ 5
Capture Quiz
```

**Gym battle (1 of 2 answered):**
```
Progress   1/ 2
Gym Test
```

## Verification
- [x] Wild encounters show mastery progress and "Wild Question" label
- [x] Capture mode shows encounter progress and "Capture Quiz" label
- [x] Gym battles show encounter progress and "Gym Test" label
- [x] Trainer battles show mastery progress and "Trainer Quiz" label
- [x] Non-quiz battles show normal PP and TYPE
- [x] Progress updates correctly as questions are answered
