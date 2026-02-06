# Phase 7 Plan 02: Explanation Timing Fix Summary

## One-Liner
Fixed explanation display timing to show immediately after answer selection (before animations/damage), working for all battle types (wild, capture, trainer).

## Problem Statement

The original implementation (07-01) displayed explanations by calling `BattlePutTextOnWindow` in `Quiz_OnMoveConfirmed`, but this only worked correctly in trainer battles. The explanation appeared **after** damage resolved because:

1. Move confirmation immediately submitted the move for execution
2. Explanation display was deferred to `Quiz_OnCommandMenuShown`
3. `Quiz_OnCommandMenuShown` only triggers when the command menu reappears
4. In wild battles, if the opponent fainted, the battle ended before the command menu reappeared

**Result:** Explanations were never shown in wild battles when the opponent was KO'd.

## Solution

Introduced an intermediate battle controller state `HandleExplanationWait` that:
1. Intercepts move execution after answer confirmation
2. Displays the explanation immediately
3. Waits for player acknowledgment (A button)
4. Supports page scrolling (R button)
5. Proceeds with move execution only after acknowledgment

## Tasks Completed

| Task | Description | Status |
|------|-------------|--------|
| 1 | Add static variables for saved move selection state | ✓ Complete |
| 2 | Add HandleExplanationWait controller function | ✓ Complete |
| 3 | Modify HandleInputChooseMove to check for explanation wait | ✓ Complete |
| 4 | Modify HandleInputChooseTarget for double-battle case | ✓ Complete |
| 5 | Add QuizHooks_ShowPendingExplanation wrapper | ✓ Complete |

## Files Modified

- `pokefirered/src/battle_controller_player.c`
  - Added `sSavedMoveSelection`, `sSavedTarget`, `sExplanationDisplayed` static variables
  - Added forward declaration for `HandleExplanationWait`
  - Added `HandleExplanationWait` controller function
  - Modified `HandleInputChooseMove` to check `QuizHooks_IsWaitingForExplanation()`
  - Modified `HandleInputChooseTarget` with same pattern

- `pokefirered/include/quiz/quiz_hooks.h`
  - Added `QuizHooks_ShowPendingExplanation()` declaration

- `pokefirered/src/quiz/quiz_hooks.c`
  - Added `QuizHooks_ShowPendingExplanation()` implementation

## Key Code Additions

### HandleExplanationWait Controller Function
```c
static void HandleExplanationWait(void)
{
    // First frame: display the explanation if not already shown
    if (!sExplanationDisplayed)
    {
        QuizHooks_ShowPendingExplanation();
        sExplanationDisplayed = TRUE;
    }
    
    // Handle A button to acknowledge and proceed with move
    if (JOY_NEW(A_BUTTON))
    {
        PlaySE(SE_SELECT);
        QuizHooks_AcknowledgeExplanation();
        sExplanationDisplayed = FALSE;
        
        // Now proceed with the saved move selection
        BtlController_EmitTwoReturnValues(1, 10, sSavedMoveSelection | (sSavedTarget << 8));
        PlayerBufferExecCompleted();
    }
    // Handle R button to scroll explanation pages
    else if (JOY_NEW(R_BUTTON))
    {
        QuizHooks_OnScrollPressed();
    }
}
```

### Modified Move Confirmation Logic
```c
if (!canSelectTarget)
{
    // Check if we need to show explanation first
    if (QuizHooks_IsWaitingForExplanation())
    {
        sSavedMoveSelection = gMoveSelectionCursor[gActiveBattler];
        sSavedTarget = gMultiUsePlayerCursor;
        sExplanationDisplayed = FALSE;
        gBattlerControllerFuncs[gActiveBattler] = HandleExplanationWait;
    }
    else
    {
        BtlController_EmitTwoReturnValues(1, 10, ...);
        PlayerBufferExecCompleted();
    }
}
```

## Behavior Flow (After Fix)

```
Player selects answer:
  └─► HandleInputChooseMove (A button)
      └─► QuizHooks_OnMoveConfirmed()
          └─► Sets waitingForExplanationAck = TRUE
      └─► Check QuizHooks_IsWaitingForExplanation()
          └─► TRUE: Save move/target, switch to HandleExplanationWait
              └─► Display explanation in B_WIN_QUIZ
              └─► Wait for A button
              └─► Proceed with move execution
          └─► FALSE: Normal move execution
```

## Requirements Satisfied

- [x] Explanation displays immediately after answer selection (before animations/damage)
- [x] Works for all encounter types: wild, capture, trainer
- [x] Pressing A advances explanation → proceeds to execute move
- [x] Explanation shown even if damage would KO (KO cannot prevent explanation)
- [x] Next question shows after damage resolution with no explanation bleed
- [x] R button scrolling works during explanation wait

## Technical Notes

### Why This Approach Works
1. **State machine pattern**: Follows the battle controller's existing state machine pattern
2. **Non-invasive**: No changes to battle script execution or damage calculation
3. **Universal**: Works for single battles, double battles (via HandleInputChooseTarget), all encounter types
4. **Safe**: Move selection saved before state switch, restored when acknowledged

### Controller Function Pattern
The battle controller uses `gBattlerControllerFuncs[gActiveBattler]` to store the current input handler. By switching to `HandleExplanationWait`, we pause at explanation display until the player acknowledges, then resume normal flow.

## Build Status
- ROM compiles successfully
- No new warnings introduced

## Date
2026-02-05
