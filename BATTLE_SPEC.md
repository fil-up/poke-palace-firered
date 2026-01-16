## The exact order of operations (authoritative spec)

### A) Battle starts normally

1. Wild/trainer battle begins as usual.
2. Intro text runs as usual:

   * “A wild X appeared!” OR “Trainer Y sent out X!”
3. The game reaches the **main command menu**:

   * `FIGHT / BAG / POKEMON / RUN`

**Only here** do we “arm” the quiz UI.

---

### B) Quiz question appears at the command menu

4. When the command menu is active, show the **question textbox** (your screenshot “What does IBNR stand for?”).

   * This should be “passive UI” (not asking for input yet).
5. If the player selects **BAG / POKEMON / RUN**, behave normally (for MVP I’d recommend: still allow, but quiz remains armed next time the menu returns).

---

### C) Answer selection happens in the move menu, by hovering moves

6. If the player selects **FIGHT**, the normal 4-move menu appears.
7. While the player moves the cursor across moves:

   * Slot 1 → Answer A
   * Slot 2 → Answer B
   * Slot 3 → Answer C
   * Slot 4 → Answer D
8. When the cursor highlights a move slot, show a textbox:

   * “Use Waterfall to pick Answer A?”
   * And below it, show **the answer text for that slot**
9. When the player confirms a move:

   * That locks their answer (A–D).
   * Quiz grading happens immediately.
   * Then the turn executes.

---

### D) Turn resolution (your damage rules)

**Player always acts first** (quiz mode only):

* Simplest: set player battler’s speed to “effectively infinite” each turn, or force move order.

**If answer is correct:**

* The selected move executes as normal (animation etc.)
* But **damage is forced to 1000** (fixed damage; ignore normal calc)
* Message can show “It’s super effective!” (optional)

**If answer is wrong:**

* The selected move executes as normal
* But **damage is forced to 1**
* Message can show “It’s not very effective…” (optional)

**Then foe attacks:**

* Whatever move the foe uses, its damage is forced to:

  * `ceil(userMaxHP / 2)`
  * i.e. `(maxHP + 1) / 2` using integer math
* This guarantees 2 wrong answers = faint.

That’s the whole “battle contract.”

---

## Implementation approach that won’t explode later

### Key idea

**Do NOT start this at encounter start anymore.**
You want it at the **battle controller stage** where the command menu is already up.

So instead of hooking `StartWildBattle()` and trying to do UI there, we do:

* `StartWildBattle()` → only sets `gQuizMode = TRUE` for wild battles (or for all battles if you want)
* UI + behavior happens inside the **player battle controller**:

  * when the main command menu is shown
  * when the move cursor changes
  * when a move is confirmed

This matches your screenshots perfectly.

---

## What files to adjust (high level)

### 1) Keep engine touches minimal

**Only**:

* `firered/src/battle_setup.c`

  * set a flag like `gQuizMode = TRUE` for wild battles (and maybe trainer battles later)

Everything else should be under:

* `firered/src/quiz/*`

### 2) Implement the quiz runtime state machine in `src/quiz`

Add (or refactor toward) something like:

* `firered/src/quiz/quiz_runtime.c/.h`

  * stores current question prompt + 4 answers + correct index
  * stores whether question is displayed
  * stores current “hovered slot” answer preview
  * stores last graded result for this turn (correct/wrong)

### 3) Hook into player menu + move selection (the big work)

Wherever FireRed handles:

* showing the main command menu
* handling “Fight” selection
* handling move cursor movement
* confirming move

…we add calls like:

* `Quiz_OnCommandMenuShown()` → show question textbox
* `Quiz_OnMoveCursorChanged(slotIndex, moveId)` → show “Use {move} to pick Answer {A-D}” + answer text
* `Quiz_OnMoveConfirmed(slotIndex, moveId)` → grade answer and store `gQuizTurnResult`

### 4) Force damage during damage calculation (cleanest)

Instead of creating fake moves, the simplest stable approach is:

* During damage calculation:

  * If attacker is player and `gQuizTurnResult == CORRECT` → force damage = 1000
  * If attacker is player and `WRONG` → force damage = 1
  * If attacker is foe and quiz mode is on → force damage = `(playerMaxHP + 1) / 2`

This preserves:

* the selected move animation
* the normal battle flow
* without fighting the move database

