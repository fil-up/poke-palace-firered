# ENH-F-01: Capture Animation on Final Correct Answer

## Objective
When the player answers the final capture quiz question correctly, play a ball throw animation before completing the capture. Currently, the Pokemon just faints instantly - we want the visual satisfaction of seeing the ball capture.

## Current Flow (Problem)
1. Final correct answer → `QUIZ_TURN_CAPTURE_SUCCESS`
2. Damage override returns 1000 → enemy faints instantly
3. `gBattleOutcome = B_OUTCOME_WON`
4. `HandleEndTurn_BattleWon` → `QuizHooks_OnBattleWon()` adds Pokemon to party
5. Battle ends with standard "wild battle won" script

**Issue**: No capture animation - Pokemon just disappears from fainting.

## Target Flow (Solution)
1. Final correct answer → `QUIZ_TURN_CAPTURE_SUCCESS`
2. Damage override returns **0** → enemy stays alive and visible
3. In `HandleAction_ActionFinished`, detect capture success
4. Redirect to `BattleScript_QuizCapture`:
   - Play ball throw animation (guaranteed 3 shakes + click)
   - Pokemon shrinks into ball
   - "Gotcha! [Pokemon] was caught!"
   - Add to party
5. `gBattleOutcome = B_OUTCOME_CAUGHT`
6. Battle ends

## Implementation Steps

### Step 1: Add Quiz Hook for Capture Trigger

**File: `pokefirered/include/quiz/quiz_hooks.h`**
```c
// Add declaration
bool8 QuizHooks_ShouldTriggerCapture(void);
```

**File: `pokefirered/src/quiz/quiz_hooks.c`**
```c
bool8 QuizHooks_ShouldTriggerCapture(void)
{
    return Quiz_GetTurnResult() == QUIZ_TURN_CAPTURE_SUCCESS;
}
```

### Step 2: Modify Damage Override

**File: `pokefirered/src/quiz/quiz_hooks.c`**

In `QuizHooks_ApplyDamageOverride`, change the `QUIZ_TURN_CAPTURE_SUCCESS` case:
```c
case QUIZ_TURN_CAPTURE_SUCCESS:
    // Don't faint - keep alive for capture animation
    *damage = 0;
    return TRUE;
```

### Step 3: Add Capture Script Trigger

**File: `pokefirered/src/battle_main.c`**

In `HandleAction_ActionFinished`, after the existing gym quiz check:
```c
else if (QuizHooks_ShouldTriggerCapture())
{
    // Final capture question correct - trigger capture animation
    QuizHooks_OnBattleWon();  // Add Pokemon to party
    gBattleOutcome = B_OUTCOME_CAUGHT;
    gCurrentActionFuncId = B_ACTION_EXEC_SCRIPT;
    gBattlescriptCurrInstr = BattleScript_QuizCapture;
    return;
}
```

### Step 4: Create Battle Script Command for Ball Animation

The existing `handleballthrow` command does catch calculations we don't want. Need a simpler command that just plays the success animation.

**File: `pokefirered/asm/macros/battle_script.inc`**

Add new macro (find next available command slot, likely around 0x100+):
```asm
.macro quizcaptureball
.byte 0xXX  @ next available command ID
.endm
```

**File: `pokefirered/src/battle_script_commands.c`**

Add command implementation:
```c
static void Cmd_quizcaptureball(void)
{
    gActiveBattler = GetBattlerAtPosition(B_POSITION_PLAYER_LEFT);
    gBattleSpritesDataPtr->animationData->ballThrowCaseId = BALL_3_SHAKES_SUCCESS;
    gLastUsedItem = ITEM_POKE_BALL;  // Use standard Poke Ball visual
    BtlController_EmitBallThrowAnim(BUFFER_A, BALL_3_SHAKES_SUCCESS);
    MarkBattlerForControllerExec(gActiveBattler);
    gBattlescriptCurrInstr++;
}
```

Add to command table at appropriate index.

### Step 5: Create Quiz Capture Battle Script

**File: `pokefirered/data/battle_scripts_2.s`**

Add after `BattleScript_SuccessBallThrow`:
```asm
BattleScript_QuizCapture::
    quizcaptureball
    waitstate
    printstring STRINGID_GOTCHAPKMNCAUGHT
    waitmessage B_WAIT_TIME_LONG
    setbyte gBattleOutcome, B_OUTCOME_CAUGHT
    finishturn
```

**File: `pokefirered/include/battle_scripts.h`**

Add declaration:
```c
extern const u8 BattleScript_QuizCapture[];
```

### Step 6: Include Quiz Hooks Header

**File: `pokefirered/src/battle_main.c`**

Ensure `#include "quiz/quiz_hooks.h"` is present (should already be there).

## Files Modified

| File | Change |
|------|--------|
| `pokefirered/include/quiz/quiz_hooks.h` | Add `QuizHooks_ShouldTriggerCapture()` declaration |
| `pokefirered/src/quiz/quiz_hooks.c` | Add function, modify damage override |
| `pokefirered/src/battle_main.c` | Add capture trigger in `HandleAction_ActionFinished` |
| `pokefirered/asm/macros/battle_script.inc` | Add `quizcaptureball` macro |
| `pokefirered/src/battle_script_commands.c` | Add `Cmd_quizcaptureball` command |
| `pokefirered/data/battle_scripts_2.s` | Add `BattleScript_QuizCapture` |
| `pokefirered/include/battle_scripts.h` | Add script declaration |

## Verification

1. **Wild encounter - capture quiz success**:
   - Answer all questions correctly
   - Ball throw animation plays (3 shakes + click)
   - "Gotcha! [Pokemon] was caught!" message
   - Pokemon added to party
   - Battle ends normally

2. **Wild encounter - capture quiz fail**:
   - Answer wrong during capture
   - No animation (existing flee behavior)
   - Pokemon flees

3. **Learning mode**:
   - Normal quiz encounters unchanged
   - Still instant KO on correct answer
   - No capture animation (not in capture mode)

4. **Gym battles**:
   - Unchanged behavior
   - No capture animation

## Risks

1. **Command slot collision**: Need to verify next available battle script command ID
2. **Animation state**: Ball animation expects certain battle state - need to ensure sprite is visible
3. **Double add**: `QuizHooks_OnBattleWon()` called before script, but script might try to add again via `givecaughtmon` - simplified script avoids this

## Notes

- The ball animation uses `BALL_3_SHAKES_SUCCESS` for guaranteed capture visual
- `gLastUsedItem = ITEM_POKE_BALL` ensures standard red/white ball appearance
- Pokemon stays visible (HP > 0) until ball animation shrinks it
- Simplified script skips nickname prompt and Pokedex for quiz capture (can add later if desired)
