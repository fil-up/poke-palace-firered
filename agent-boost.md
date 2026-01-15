# QuizHack — Agent Issue Board

## EPIC 0 — Repo & Build Foundation

### Issue 0.1 — Verify FireRed decomp builds locally

**Type:** setup
**Goal:** Ensure baseline build works before any changes.

**Acceptance criteria**

* `pret/pokefirered` builds successfully
* ROM boots in emulator
* Screenshot or note confirming success

**Files touched**

* none (local only)

---

### Issue 0.2 — Add project docs and folder skeleton

**Type:** docs / scaffolding

**Acceptance criteria**

* Add `/docs/ENGINEERING_BRIEF.md`
* Add `/docs/DATA_SCHEMA.md`
* Add `/docs/DESIGN.md`
* Create empty folders:

  * `/src/quiz`
  * `/tools`
  * `/data/questions/routes`
  * `/generated`

**Files touched**

* docs/*
* folder structure only

---

## EPIC 1 — Question Data Pipeline

### Issue 1.1 — Implement question schema validator

**Type:** tooling

**Description**
Create `tools/validate_questions.py` to enforce schema + constraints.

**Acceptance criteria**

* Fails on:

  * missing required fields
  * duplicate IDs
  * wrong types
  * text length overflow
* CLI usage: `python tools/validate_questions.py`

**Files**

* tools/validate_questions.py

---

### Issue 1.2 — Implement question code generator

**Type:** tooling

**Description**
Generate static C tables from validated JSON.

**Acceptance criteria**

* Reads all JSON in `/data/questions/routes`
* Outputs:

  * `/generated/questions_gen.h`
  * `/generated/questions_gen.c`
* Tables indexed by:

  * scope
  * species
  * question index
* No dynamic allocation

**Files**

* tools/build_questions.py
* generated/questions_gen.{h,c}

---

### Issue 1.3 — Add Make targets for validation + generation

**Type:** build

**Acceptance criteria**

* `make validate` runs validator
* `make build-questions` runs generator
* Normal `make` depends on generation step

**Files**

* Makefile

---

## EPIC 2 — Quiz Core Logic (Engine-agnostic)

### Issue 2.1 — Create quiz module scaffolding

**Type:** core

**Acceptance criteria**

* Add headers + source files:

  * quiz.h / quiz.c
  * quiz_bank.h / quiz_bank.c
  * quiz_save.h / quiz_save.c
  * quiz_ui.h / quiz_ui.c
  * quiz_hooks.h / quiz_hooks.c
* All compile cleanly with stub functions

**Files**

* src/quiz/*

---

### Issue 2.2 — Implement quiz_bank lookup + selection

**Type:** logic

**Description**
Given `(scope, species)`, select next question.

**Acceptance criteria**

* Returns:

  * unanswered question index, OR
  * CAPTURE_PENDING flag if mastered
* Selection uniform among unmastered indices

**Files**

* src/quiz/quiz_bank.{c,h}

---

## EPIC 3 — Save Data (Persistent Progress)

### Issue 3.1 — Define QuizSave struct + versioning

**Type:** save system

**Acceptance criteria**

* Save struct includes:

  * version
  * mastery bitmask(s)
  * capturePending
  * cleared flag
* Version mismatch wipes quiz progress safely

**Files**

* src/quiz/quiz_save.{c,h}

---

### Issue 3.2 — Implement mastery bitmask read/write

**Type:** save system

**Acceptance criteria**

* Functions:

  * Get mastery mask
  * Set mastery bit
  * Check all-mastered
* Survives save/load cycle

**Files**

* src/quiz/quiz_save.{c,h}

---

## EPIC 4 — Quiz UI (MVP)

### Issue 4.1 — Implement question prompt UI

**Type:** UI

**Acceptance criteria**

* Displays:

  * prompt
  * 4 choices
* Returns selected index
* Uses standard FireRed text/menu UI

**Files**

* src/quiz/quiz_ui.{c,h}

---

### Issue 4.2 — Implement feedback messages

**Type:** UI

**Acceptance criteria**

* Shows “Correct!” or “Incorrect.”
* Optional explanation text
* Returns result enum

**Files**

* src/quiz/quiz_ui.{c,h}

---

## EPIC 5 — Wild Encounter Integration (Route 1 MVP)

### Issue 5.1 — Hook wild encounter start (Route 1 only)

**Type:** engine hook

**Acceptance criteria**

* On wild encounter start:

  * Call Quiz_OnWildEncounter()
  * Pause normal battle flow
* Limited to Route 1 for MVP

**Files**

* src/quiz/quiz_hooks.{c,h}
* minimal edits to battle init logic

---

### Issue 5.2 — Apply wrong-answer damage rule

**Type:** battle logic

**Acceptance criteria**

* Wrong answer:

  * Player HP = floor(currentHP / 2)
  * Clamp to minimum 1 HP
* Encounter ends immediately

**Files**

* src/quiz/quiz_hooks.c

---

### Issue 5.3 — Apply correct-answer KO

**Type:** battle logic

**Acceptance criteria**

* Correct answer:

  * Wild Pokémon HP → 0
  * Battle ends as fainted
* EXP granted normally

**Files**

* src/quiz/quiz_hooks.c

---

## EPIC 6 — Capture Quiz Flow

### Issue 6.1 — Detect mastery → capture pending

**Type:** logic

**Acceptance criteria**

* After last unanswered question is answered correctly:

  * Mark species CAPTURE_PENDING
* No capture UI yet

**Files**

* src/quiz/quiz.c
* src/quiz/quiz_save.c

---

### Issue 6.2 — Implement capture quiz sequence

**Type:** core gameplay

**Acceptance criteria**

* On next encounter:

  * Ask all N questions sequentially
* Any wrong:

  * Apply half-HP penalty
  * End encounter
* All correct:

  * Capture Pokémon
  * Mark species CLEARED

**Files**

* src/quiz/quiz.c
* src/quiz/quiz_hooks.c

---

### Issue 6.3 — Grant Pokémon without inventory

**Type:** capture

**Acceptance criteria**

* On capture success:

  * Add Pokémon to party or PC
  * Show success message
* No Poké Balls used

**Files**

* src/quiz/quiz_hooks.c

---

## EPIC 7 — Remove Cleared Species from Wild Encounters

### Issue 7.1 — Wild encounter reroll logic

**Type:** encounter system

**Acceptance criteria**

* If rolled species is CLEARED:

  * Reroll up to MAX_REROLLS (10)
* If exceeded:

  * Return “no encounter”

**Files**

* src/quiz/quiz_hooks.c

---

## EPIC 8 — Trainer Reinforcement (Optional / Later)

### Issue 8.1 — Hook enemy Pokémon send-out

**Type:** battle hook

**Acceptance criteria**

* On enemy Pokémon send-out:

  * Roll 15% chance
  * Trigger review question

**Files**

* src/quiz/quiz_hooks.c

---

### Issue 8.2 — Reinforcement question selection

**Type:** logic

**Acceptance criteria**

* Prefer mastered questions
* Apply same wrong penalty
* No capture logic involved

**Files**

* src/quiz/quiz_bank.c
* src/quiz/quiz.c

---

## Labels to create (important for agents)

* `good-first-issue`
* `engine-touch`
* `data-only`
* `ui`
* `save-system`
* `tooling`
* `blocked`
* `needs-design-review`

---

## How you should use this

1. Create all issues up front.
2. Mark EPIC 0–1 as **blocked=false**, everything else blocked.
3. Let agents take **one issue max at a time**.
4. Require screenshots or short clips for engine-touch PRs.
