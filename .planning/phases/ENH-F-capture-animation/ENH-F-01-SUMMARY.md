# ENH-F-01: Capture Animation on Final Correct Answer - Summary

## Completed: 2026-01-28

## Objective
Play a ball throw animation when the player successfully completes a capture quiz, providing visual feedback that the Pokemon was caught.

## Final Implementation

Uses the game's existing ball throw system with Master Ball - thematically fitting since the player "mastered" the question bank.

### Capture Trigger in Battle Main

**File: `pokefirered/src/battle_main.c`** (in `HandleAction_ActionFinished`)

```c
else if (QuizHooks_ShouldTriggerCapture())
{
    // All capture quiz questions answered - trigger ball throw!
    // Use Master Ball since player "mastered" the questions (guaranteed catch)
    
    // Clear turn result FIRST to prevent re-triggering during capture flow
    Quiz_ClearTurnResult();
    
    gBattlerAttacker = GetBattlerAtPosition(B_POSITION_PLAYER_LEFT);
    gBattlerTarget = gBattlerAttacker;  // UseItem sets both to same initially
    gBattle_BG0_X = 0;
    gBattle_BG0_Y = 0;
    ClearFuryCutterDestinyBondGrudge(gBattlerAttacker);
    gLastUsedItem = ITEM_MASTER_BALL;
    
    // Use the normal ball throw script - handleballthrow will do the rest
    gBattlescriptCurrInstr = gBattlescriptsForBallThrow[ITEM_MASTER_BALL];
    gCurrentActionFuncId = B_ACTION_EXEC_SCRIPT;
    return;
}
```

### Supporting Changes

**File: `pokefirered/include/quiz/quiz_hooks.h`**
- Added `QuizHooks_ShouldTriggerCapture()` declaration

**File: `pokefirered/src/quiz/quiz_hooks.c`**
- Added `QuizHooks_ShouldTriggerCapture()` - returns TRUE when `QUIZ_TURN_CAPTURE_SUCCESS`
- Modified damage override: `QUIZ_TURN_CAPTURE_SUCCESS` returns damage = 0 (keeps enemy visible)

**File: `pokefirered/include/quiz/quiz.h`**
- Added `Quiz_ClearTurnResult()` declaration

**File: `pokefirered/src/quiz/quiz.c`**
- Added `Quiz_ClearTurnResult()` - prevents double-triggering during capture flow

## Files Modified

| File | Change |
|------|--------|
| `pokefirered/include/quiz/quiz_hooks.h` | Added `QuizHooks_ShouldTriggerCapture()` |
| `pokefirered/src/quiz/quiz_hooks.c` | Added function, changed capture damage to 0 |
| `pokefirered/include/quiz/quiz.h` | Added `Quiz_ClearTurnResult()` |
| `pokefirered/src/quiz/quiz.c` | Added `Quiz_ClearTurnResult()` |
| `pokefirered/src/battle_main.c` | Added capture trigger using normal ball throw flow |

## Flow Diagram

```
Player answers final capture question correctly
  → QUIZ_TURN_CAPTURE_SUCCESS set
  → damage = 0 (enemy stays visible)
  → HandleAction_ActionFinished detects capture success
  → Quiz_ClearTurnResult() prevents re-trigger
  → Sets up state like HandleAction_UseItem does
  → gLastUsedItem = ITEM_MASTER_BALL
  → Redirects to gBattlescriptsForBallThrow[ITEM_MASTER_BALL]
  → Normal ball throw script runs:
    → "RED used MASTER BALL!"
    → handleballthrow (guaranteed catch)
    → Ball animation (3 wiggles + click)
    → "Gotcha! PIDGEY was caught!"
    → Pokedex registration
    → Nickname prompt
    → Pokemon added to party/box
    → Battle ends with B_OUTCOME_CAUGHT
```

## Verification

- [x] Wild capture quiz - final correct answer triggers Master Ball throw
- [x] Ball animation plays (3 wiggles + success click)
- [x] "Gotcha! [Pokemon] was caught!" message displays
- [x] Pokedex data added message
- [x] Nickname prompt works (Yes and No both work)
- [x] Pokemon added to party/box correctly
- [x] Battle ends normally after capture
- [x] No freeze during or after capture sequence

## Design Notes

- **Master Ball choice**: Thematically fitting - player "mastered" the questions, so they get the "Master" Ball
- **Reuses existing code**: No custom battle script commands needed - leverages the game's proven ball throw system
- **Critical fix**: `Quiz_ClearTurnResult()` must be called before triggering capture to prevent the check from firing again during the multi-step capture sequence
- **Damage = 0**: Keeps enemy Pokemon visible on screen for the ball throw animation

## Unused Code (can be cleaned up later)

The initial implementation attempt added custom battle script infrastructure that ended up not being used:
- `quizcaptureball` macro (0xF8) in `battle_script.inc`
- `Cmd_quizcaptureball` in `battle_script_commands.c`
- `BattleScript_QuizCapture` in `battle_scripts_2.s`
- `BattleScript_QuizCapture` declaration in `battle_scripts.h`

These can be removed in a future cleanup, but don't cause any issues.
