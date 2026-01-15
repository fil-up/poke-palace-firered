# QuizHack Design Spec (FireRed Decomp)

## Purpose

Define the exact gameplay behavior and state machines for:

* Wild encounters → question loop → mastery → capture quiz
* “Cleared” behavior (no repeat wild encounters for mastered species)
* Damage rules (wrong = half current HP, clamped to ≥ 1)
* Minimal trainer reinforcement (optional phase)

This doc is meant to be **precise enough for agents** to implement without interpretation.

---

## Core definitions

### Entities

* **Scope**: a location bucket for wild encounters, initially by `(mapGroup, mapNum)`; later may be “route name”.
* **Bank**: a set of `N` questions associated with a `(scope, species)` pair.
* **Mastery**: answering a question correctly marks that question index as mastered in persistent save.
* **Cleared species**: species is “done” in that scope (no longer appears as wild encounter).
* **Capture quiz**: a perfect run through all `N` questions, unlocked after mastery, triggered on the **next encounter**.

### Constants (MVP defaults)

* `QUESTIONS_PER_SPECIES = 5` (cap 8 max for bitmask)
* `WRONG_DAMAGE_MODE = HALF_CURRENT_HP` (fallback: set HP to 1)
* `REINFORCEMENT_CHANCE = 15%` (later)

---

## Player experience summary (wild encounters)

### Normal phase (learning)

When you encounter a wild Pokémon species that is not cleared:

1. Game pauses battle flow.
2. A question is shown (selected from unanswered questions for that species in this scope).
3. Player answers.
4. Result:

   * **Correct**: wild Pokémon is instantly KO’d.
   * **Wrong**: player’s active Pokémon takes damage equal to **half of its current HP**, clamped so it never drops below **1 HP**.

Repeat encounters cycle through remaining unanswered questions.

### Mastery → Capture quiz trigger

After the player has correctly answered all questions in the bank (all bits set):

* The species is now in **CAPTURE_PENDING** state for that scope.
* The **next time** this species is encountered in that scope, the encounter enters **capture quiz mode**.

### Capture quiz mode

In capture quiz mode:

* Player must answer **all N questions in sequence** (can be fixed order or deterministic shuffle; MVP: fixed order by index).
* If the player answers **any question wrong**, capture quiz fails immediately:

  * Apply wrong penalty (half current HP, clamp ≥ 1)
  * Wild Pokémon may either:

    * (MVP A) flee immediately (recommended: faster loop, less messy)
    * (MVP B) continue battle normally (more complex)
  * Capture remains pending (they can try again on a future encounter).
* If player answers **all N correctly**, capture succeeds:

  * Mark species as **CLEARED** for that scope.
  * Give the Pokémon to the player:

    * (MVP) place directly into party if space else PC.
  * End encounter cleanly (no traditional capture UI / no inventory).

**Important:** Capture quiz does not use Poké Balls or inventory.

---

## State machine (per scope + species)

Represented as persistent state + runtime state.

### Persistent state fields (per scope+species)

* `masteryMask` (bitmask of N questions)
* `capturePending` (bool)
* `cleared` (bool)

#### State definitions

* **S0: UNSEEN**

  * `masteryMask = 0`
  * `capturePending = false`
  * `cleared = false`
* **S1: LEARNING**

  * `masteryMask != allOnes`
  * `capturePending = false`
  * `cleared = false`
* **S2: CAPTURE_PENDING**

  * `masteryMask == allOnes`
  * `capturePending = true`
  * `cleared = false`
* **S3: CLEARED**

  * `cleared = true` (implies no encounters)
  * `capturePending = false` (optional)
  * `masteryMask` can remain allOnes

#### Transitions

* S0 → S1: first correct answer sets first bit
* S1 → S1: correct answer sets additional bits
* S1 → S2: after correct answer that completes the mask
* S2 → S2: capture quiz failed
* S2 → S3: capture quiz perfect success

---

## Encounter selection rules (no repeats)

### Rule: cleared species should not appear in wild encounters

When generating a wild encounter for a scope:

* If rolled species is CLEARED, re-roll encounter.

### Reroll safety

To avoid infinite loops late-game:

* Cap rerolls at `MAX_REROLLS = 10`
* If still CLEARED after cap:

  * Option A (recommended): return “no encounter”
  * Option B: allow encounter but immediately skip quiz and end (feels weird)

MVP recommendation: **Option A**.

---

## Question selection rules (learning phase)

When in LEARNING phase:

* Select uniformly among **unmastered** question indices.
* If none exist, transition to CAPTURE_PENDING and next encounter uses capture quiz.

Determinism note:

* For debugging and agent work, it’s helpful if selection is stable:

  * MVP: random among remaining
  * Add optional flag `DETERMINISTIC_SELECTION` later.

---

## Damage rules

### Wrong answer penalty (primary)

* Let `curHP` be player active Pokémon current HP (integer).
* Damage = `floor(curHP / 2)`
* New HP = `curHP - damage`
* Clamp: `newHP = max(newHP, 1)`

Examples:

* curHP 100 → damage 50 → newHP 50
* curHP 1 → damage 0 → newHP 1
* curHP 3 → damage 1 → newHP 2

### Fallback mode (prototype)

* Set player HP to 1 on wrong answer (only if implementing half HP is annoying initially)

---

## Wild battle outcome rules (MVP)

### Correct answer

* Immediately KO wild Pokémon and end the battle as if it fainted.
* Player gains standard EXP or not:

  * MVP recommendation: **Yes, normal EXP** (it keeps leveling consistent)
  * Optional later: EXP tied to correct answers instead.

### Wrong answer

* Apply half HP penalty.
* Continue battle?

  * MVP recommendation: **end encounter** immediately (like “you got hit and it escaped”)

    * This reduces edge cases and keeps the loop “quiz-first”.
  * Later: allow battle continuation if you want.

---

## Capture success behavior (no inventory)

On capture quiz perfect:

* Add Pokémon to party if space else PC.
* Use a simple message sequence:

  * “Perfect! You caught [SPECIES]!”
* Immediately mark `cleared = true` for that scope+species.

---

## Gym “section exam” mode (later milestone)

Not required for MVP, but design direction:

* Each gym is mapped to a “topic scope”.
* On gym battle start:

  * Run an exam quiz of `EXAM_Q_COUNT` questions.
  * Allow `EXAM_MISSES_ALLOWED` (e.g., 1).
  * On fail: immediate loss.
  * On pass: proceed with normal gym battle OR skip battle and award badge (choose later).

---

## Trainer reinforcement (later milestone)

Goal: spaced repetition.

On enemy Pokémon send-out:

* Roll `REINFORCEMENT_CHANCE`.
* If triggered:

  * ask a question from that species bank, but prefer already-mastered questions.
  * wrong penalty: half HP
  * correct: optional small benefit (tiny heal/buff) later; MVP can be “no benefit”.

---

## Implementation notes for FireRed decomp

* All engine hooks should call into `/src/quiz/quiz_hooks.c`.
* Avoid scattering logic across battle engine files.
* Keep generated data in `/generated` with a Make target.

---

## MVP acceptance criteria (what “working” means)

1. Route 1 has banks for Rattata and Pidgey.
2. Wild encounter triggers question UI before battle actions.
3. Correct = wild Pokémon KO and encounter ends.
4. Wrong = player loses half current HP (clamp ≥ 1) and encounter ends.
5. Mastery persists through save/load.
6. After mastering all N, the **next encounter** triggers capture quiz.
7. Perfect capture quiz adds Pokémon to party/PC and marks species cleared.
8. Cleared species no longer appears in wild encounters (reroll + safety cap).