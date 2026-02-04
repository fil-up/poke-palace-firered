# Architecture

**Analysis Date:** 2026-01-30

## Pattern Overview

**Overall:** Layered Plugin Architecture with Minimal Engine Intrusion

**Key Characteristics:**
- Quiz system implemented as a separate layer that hooks into the existing FireRed battle engine
- All custom logic isolated in `pokefirered/src/quiz/` directory
- Battle engine files only make function calls to `quiz_hooks.h` — no scattered modifications
- State machine pattern for quiz progression (UNSEEN → MASTERING → CAPTURE_PENDING → CLEARED)
- Build-time code generation for question data (CSV → Python → C)

## Layers

**Quiz System Layer (Custom):**
- Purpose: Implements quiz-based battle mechanics replacing traditional combat
- Location: `pokefirered/src/quiz/`
- Contains: State machine, question selection, damage overrides, save persistence
- Depends on: FireRed battle engine, save system
- Used by: Battle controller hooks, wild encounter hooks

**FireRed Battle Engine (Modified):**
- Purpose: Core battle flow, animations, UI windows
- Location: `pokefirered/src/battle_*.c`
- Contains: Turn order, damage calculation, move execution, controller dispatch
- Depends on: Pokemon data, map system
- Used by: Quiz hooks (via hook points)

**Save System (Extended):**
- Purpose: Persist quiz progress across sessions
- Location: `pokefirered/include/global.h`, `pokefirered/src/quiz/quiz.c`
- Contains: QuizSaveData structure in SaveBlock1
- Depends on: Flash memory subsystem
- Used by: Quiz mastery tracking, species state

**Question Data Layer (Generated):**
- Purpose: Store and retrieve quiz content
- Location: `pokefirered/src/quiz/questions_gen.c`, `pokefirered/include/quiz/questions_gen.h`
- Contains: Per-species question banks with prompts, choices, explanations
- Depends on: Build-time Python scripts
- Used by: Quiz state machine for question lookup

## Data Flow

**Wild Encounter Quiz Flow:**

1. Wild encounter triggered in `wild_encounter.c`
   └─> `TryGenerateWildMon()` filters cleared species (reroll up to 10 times)

2. Battle initialization in `battle_main.c`
   └─> `QuizHooks_OnBattleInit()` resets quiz state, fills party moves

3. Species sent into battle
   └─> `QuizHooks_OnWildBattleStart(species, mapGroup, mapNum)`
       └─> `Quiz_InitWildEncounter()` looks up bank, selects question based on mastery

4. Command menu shown
   └─> `QuizHooks_OnCommandMenuShown()`
       └─> Display question in `B_WIN_QUIZ` window (BG1 with transparency)

5. Player selects move (A/B/C/D mapped to move slots 0-3)
   └─> `QuizHooks_OnMoveConfirmed(slot, moveId)`
       └─> Grade answer, set `QuizTurnResult`, update mastery mask

6. Damage calculation phase
   └─> `QuizHooks_ApplyDamageOverride(&damage, attacker, target)`
       └─> Correct: 1000 damage (instant KO)
       └─> Wrong: 0 player damage, enemy deals half player HP

7. Post-turn handling
   └─> Check for capture success, flee conditions, state transitions

**Capture Quiz Flow:**

1. All questions mastered → state = CAPTURE_PENDING
2. Next encounter of same species triggers capture mode
3. Player must answer ALL questions correctly in sequence
4. Perfect run: species state = CLEARED, Pokemon added to party
5. Any wrong answer: wild Pokemon flees, state stays CAPTURE_PENDING

**Trainer Battle Flow:**

1. Trainer tier determined by `Quiz_GetTrainerTier()`
   - REGULAR: 1 question per Pokemon
   - GYM_TRAINER: 2 questions + evolution bonuses
   - GYM_LEADER: 5 base + evolution + area pool
   - RIVAL_BOSS: 5 base + full question pool

2. Question source determined by tier
   - SPECIES source: questions from current enemy species bank
   - POOL source: aggregate pool from all area species (leaders) or all species (rivals)

3. Questions answered in sequence within same turn
   └─> `QUIZ_TURN_GYM_CORRECT`: more questions remain
   └─> `QUIZ_TURN_GYM_ALL_CORRECT`: attack succeeds, instant KO

**State Management:**

```
Per-Species State (in SaveBlock1):
┌─────────────────────────────────────────────────────────┐
│ gSaveBlock1Ptr->quizData.species[index]                │
│   - masteryMask: u8 bitmask (up to 8 questions)        │
│   - state: u8 enum (NONE/MASTERING/CAPTURE_PENDING/CLEARED) │
└─────────────────────────────────────────────────────────┘

Species Index Mapping:
  index = species % QUIZ_SAVE_MAX_SPECIES (64 slots)
```

## Key Abstractions

**QuizBank:**
- Purpose: Container for all questions about a single species
- Examples: `gQuizBank_SPECIES_RATTATA`, `gQuizBank_SPECIES_PIDGEY`
- Pattern: Const struct with array of question pointers + count

**QuizQuestion:**
- Purpose: Single quiz question with prompt, 4 choices, correct index, explanation
- Examples: Questions in `questions_gen.c`
- Pattern: Const struct with string pointers

**QuizState:**
- Purpose: Runtime state for current battle's quiz session
- Location: `static struct QuizState sQuizState` in `quiz.c`
- Pattern: EWRAM_DATA static (persists during battle, reset on new battle)

**QuizTurnResult:**
- Purpose: Outcome of player's answer for current turn
- Values: NONE, CORRECT, WRONG, CAPTURE_SUCCESS, CAPTURE_FAIL, GYM_CORRECT, GYM_ALL_CORRECT, GYM_WRONG
- Pattern: Enum checked by damage/effect override hooks

**QuizTrainerTier:**
- Purpose: Classify trainer difficulty for question scaling
- Values: REGULAR, GYM_TRAINER, GYM_LEADER, RIVAL_BOSS, SPECIAL
- Pattern: Determined by trainer class + ID lookup tables

## Entry Points

**Battle Initialization:**
- Location: `pokefirered/src/battle_main.c`
- Triggers: `QuizHooks_OnBattleInit()`
- Responsibilities: Reset quiz state, fill party moves to ensure 4 moves for A/B/C/D

**Wild Battle Start:**
- Location: Hook in battle setup
- Triggers: `QuizHooks_OnWildBattleStart(species, mapGroup, mapNum)`
- Responsibilities: Initialize quiz for species, load question bank

**Command Menu:**
- Location: `pokefirered/src/battle_controller_player.c`
- Triggers: `QuizHooks_OnCommandMenuShown()`
- Responsibilities: Display question in quiz window

**Move Selection:**
- Location: `pokefirered/src/battle_controller_player.c`
- Triggers: `QuizHooks_OnMoveCursorChanged()`, `QuizHooks_OnMoveConfirmed()`
- Responsibilities: Show answer preview, grade answer

**Damage Calculation:**
- Location: `pokefirered/src/battle_script_commands.c`
- Triggers: `QuizHooks_ApplyDamageOverride()`
- Responsibilities: Override damage based on quiz result

**New Game:**
- Location: `pokefirered/src/new_game.c` (likely)
- Triggers: `Quiz_InitSaveData()`
- Responsibilities: Zero out all mastery masks and states

## Error Handling

**Strategy:** Graceful degradation with fallback to vanilla behavior

**Patterns:**
- NULL bank check: If `Quiz_GetBank()` returns NULL, quiz doesn't activate
- Reroll safety: Cleared species reroll capped at 10 attempts, then "no encounter"
- Save index collision: Species mapped via modulo 64, potential collisions accepted
- Missing questions: `sQuizNoQuestionText` displayed if no question available

## Cross-Cutting Concerns

**Logging:** None (GBA has no console)

**Validation:** Build-time validation via `validate_questions.py`
- Checks CSV format, correct index bounds, duplicate detection

**Save Compatibility:**
- Quiz data stored in SaveBlock1 padding bytes
- Version field in QuizSaveData for future migrations
- New saves initialize with `Quiz_InitSaveData()`

**Battle Type Exclusions:**
- Safari Zone battles skip quiz entirely
- Link battles not supported with quiz (would desync)

## Quiz Window System

**Window ID:** `B_WIN_QUIZ` (defined in battle constants)

**Rendering:**
- BG1 layer with priority 0 (appears above healthbox sprites)
- Semi-transparent blend with BG2/BG3 (87% opacity)
- Frame tiles loaded from user window graphics

**Text Display:**
- `BattlePutTextOnWindow()` for question and answer prompts
- Text buffer: 256 bytes (`sQuizTextBuffer`)

## Move Normalization (Phase 13)

All moves normalized during quiz battles:
- Damage: Controlled entirely by quiz result (1000 for correct, 0 for wrong)
- Accuracy: Always hits (`QuizHooks_ShouldAlwaysHit()`)
- PP: Infinite (`QuizHooks_ShouldInfinitePp()`)
- Effects: Secondary effects disabled (`QuizHooks_ShouldDisableSecondaryEffects()`)
- Type immunity: Ignored (`QuizHooks_ShouldIgnoreTypeImmunity()`)
- Multi-hit: Clamped to 1 hit (`QuizHooks_ClampMultiHitCounter()`)
- Targeting: Forced to opponent (`QuizHooks_ForceOpponentTarget()`)

---

*Architecture analysis: 2026-01-30*
