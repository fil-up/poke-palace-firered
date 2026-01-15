# Engineering Brief — FireRed “Quiz Encounters” Hack

## Goal

Build a Pokémon FireRed decomp-based ROM hack where **encounters and battles are driven by quiz questions**, with persistent mastery tracking so that:

* Wild encounters introduce *new* questions until mastered
* Captures require a *perfect* mini-quiz
* Cleared Pokémon/routes stop appearing as wild encounters
* Trainers and gyms reintroduce questions for reinforcement / section exams

## Recommended foundation and tooling

### Base repo

* **pret/pokefirered** (FireRed/LeafGreen decomp). ([GitHub][1])

### Content tools

* **Porymap** for maps/encounters editing (supports pokefirered). ([GitHub][2])
* **Poryscript** optional, but great for scaling event scripting + agent edits. ([GitHub][3])

### Reference-only (do not switch bases yet)

* **pokeemerald-expansion** is a *pattern library* for big systems (data-driven features, configs). We’ll borrow ideas, not base FireRed on it. ([GitHub][4])

---

## Architecture overview

Think of this as 4 subsystems:

1. **Question Bank (Data)**
2. **Progress Tracking (Save Data)**
3. **Battle/Encounter Hooks (Logic)**
4. **UI + Input (Presentation)**

Keep them separated so Cursor/agents can work in parallel with minimal merge conflicts.

---

## Repo layout (proposed)

```
/docs
  ENGINEERING_BRIEF.md
  DESIGN.md
  DATA_SCHEMA.md

/tools
  validate_questions.py
  build_questions.py   (JSON/CSV -> .h/.c outputs)

/data/questions
  routes/
    ROUTE_1.json
    VIRIDIAN_FOREST.json
  global/
    gym_topics.json

/src/quiz
  quiz.h
  quiz.c              (orchestrates “ask question”, grading, result)
  quiz_bank.h
  quiz_bank.c         (lookup + selection)
  quiz_save.h
  quiz_save.c         (read/write progress)
  quiz_ui.h
  quiz_ui.c           (text box menus)
  quiz_hooks.h
  quiz_hooks.c        (wild/trainer/gym hook glue)

/include/quiz
  quiz_config.h       (constants, tuning knobs)

/generated
  questions_gen.h
  questions_gen.c
```

### Principle

* **All agent edits should target either `/data/questions` or `/src/quiz/*`**, not random battle engine files.

---

## Data design

### Minimal question record (JSON)

Store questions as simple structured objects; enforce strict limits (GBA UI constraints).

Example:

```json
{
  "scope": "ROUTE_1",
  "species": "RATTATA",
  "questions": [
    {
      "id": "R1_RATTATA_001",
      "prompt": "What does IBNR stand for?",
      "choices": ["Incurred But Not Reported", "Issued But Not Rated", "Incurred By Net Revenue", "Internal Benefit Notation Rule"],
      "answer_index": 0
    }
  ]
}
```

### Constraints (define in `DATA_SCHEMA.md`)

* `prompt` max length (e.g., 120 chars)
* each choice max length (e.g., 40 chars)
* 4 choices fixed (simplifies UI)
* stable IDs (never change once shipped; only deprecate)

### Build step

* `tools/validate_questions.py` (schema + length + duplicate IDs)
* `tools/build_questions.py` generates `generated/questions_gen.{h,c}`
* Game code includes generated tables; no runtime parsing needed.

---

## Save data design (progress tracking)

### Core gameplay requirements

For each **(route, species)**:

* which question IDs were answered correctly
* whether capture-quiz is unlocked
* whether species is “cleared” (no more wild spawns)
  Optionally:
* a spaced-repetition counter for trainer reminders

### Implementation approach

Use a compact bitset system:

* Give each species bank **N questions** (e.g., 5)
* Track mastery as `uint8 masteryMask` (bits 0..N-1)
* Cleared when `masteryMask == (1<<N)-1` AND capture quiz passed

Persist in SaveBlock (FireRed has established save structures; we’ll add a “QuizSave” block or append to an existing unused region if available). **Plan for versioning**:

* `quizSaveVersion`
* on load: if version mismatch, wipe quiz progress cleanly (safe default)

**Important:** Don’t over-engineer “perfect compression” early. Start with 1–2 routes and ensure it survives save/load.

---

## Hook points (where engine changes happen)

### Wild encounter flow

Trigger “ask question” before battle actions resolve.

* On encounter start:

  1. determine `(currentRoute, encounteredSpecies)`
  2. select an **unused** question
  3. show UI (prompt + choices)
  4. grade answer
  5. apply battle effect:

     * correct: force KO (“one-shot”)
     * incorrect: deal damage to player mon (or halve your damage; pick one consistent rule)

**Where to implement:** keep engine touches minimal by routing to `quiz_hooks.c` and using existing battle scripting commands where possible (FireRed’s battle script command system lives in files like `src/battle_script_commands.c`). ([GitHub][5])

### Trainer battles

On each enemy Pokémon send-out (or per turn—recommend send-out):

* roll chance to ask a “review” question tied to that species (or topic)
* if wrong: player takes a hit / debuff

### Gyms

Define “topic gym” in config (e.g., Gym 1 = Reserving)
Gym battle runs a **quiz sequence**:

* X questions total
* higher difficulty pool allowed
* fail condition: simple (e.g., 2 misses = lose battle immediately) for early MVP

---

## UI plan (MVP first)

### MVP UI

Reuse standard text boxes + a 4-choice menu.

* show prompt
* show A/B/C/D
* player selects
* feedback: “Correct!” / “Incorrect…”

Don’t build a fancy UI until the full loop works end-to-end.

---

## Core logic modules

### `quiz_bank`

Responsibilities:

* map `(route, species)` -> question bank
* select next question:

  * prefer unanswered
  * if all answered: return “CAPTURE_QUIZ” mode

### `quiz_save`

Responsibilities:

* get/set mastery bits
* mark capture success
* clear species/route
* versioning

### `quiz`

Responsibilities:

* orchestration:

  * call bank -> get question
  * call UI -> get answer
  * grade
  * update save
  * return result enum:

    * `QUIZ_RESULT_CORRECT`
    * `QUIZ_RESULT_WRONG`
    * `QUIZ_RESULT_CAPTURE_UNLOCKED`
    * `QUIZ_RESULT_CAPTURE_SUCCESS`
    * `QUIZ_RESULT_CAPTURE_FAIL`

### `quiz_hooks`

Responsibilities:

* minimal integration surfaces:

  * `Quiz_OnWildEncounter(species)`
  * `Quiz_OnTrainerMonSendOut(species)`
  * `Quiz_OnGymBattleStart(gymId)`

---

## Build + CI (so agents don’t break things)

### Required scripts

* `make validate` runs validators
* `make build-questions` generates C tables

### CI checks (GitHub Actions)

* run validate
* build ROM
* optionally run a headless test harness if available (nice-to-have)

---

## Milestones (fast path to “not an idea thread”)

### Milestone 0 — “Hello World”

* Build pokefirered locally
* Add a message on game start (“QuizHack build ok”)

### Milestone 1 — Hardcoded question on Route 1 wild encounter

* 1 route, 1 species, 1 question
* Correct → force KO
* Wrong → take damage

### Milestone 2 — Save persistence

* Mastery bit survives save/load
* Once answered, the next encounter uses next question

### Milestone 3 — Capture quiz

* After N correct, trigger capture quiz
* Perfect = caught + species cleared

### Milestone 4 — Remove cleared species from wild tables

* After species cleared, no longer appears in wild encounters for that route

### Milestone 5 — Trainer reinforcement

* Ask 10–20% of the time on enemy send-out
* Use already-mastered pool

---

## FireRed-first vs Emerald patterns

**Recommendation:** stay on FireRed base. Use Emerald Expansion only as a reference for:

* data-driven feature patterns
* clean config toggles
* keeping changes modular ([GitHub][4])

If later you decide “this belongs as a broader hack base,” we can re-evaluate porting patterns or migrating, but right now your success metric is **a working FireRed prototype**.
