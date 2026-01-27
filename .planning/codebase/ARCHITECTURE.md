# Architecture

## System Overview

The project is a **Pokémon FireRed ROM hack** that replaces traditional battle mechanics with a quiz-based system. It's built on top of the pret/pokefirered decompilation project.

```
┌─────────────────────────────────────────────────────────────┐
│                    Quiz System Layer                        │
│  ┌──────────┐  ┌───────────┐  ┌──────────┐  ┌────────────┐ │
│  │Quiz Bank │  │  Progress │  │   Hooks  │  │   Quiz UI  │ │
│  │  (Data)  │  │ (Save)    │  │  (Logic) │  │   (Text)   │ │
│  └──────────┘  └───────────┘  └──────────┘  └────────────┘ │
└────────────────────────┬────────────────────────────────────┘
                         │ Minimal integration points
┌────────────────────────▼────────────────────────────────────┐
│              FireRed Battle Engine (Existing)               │
│  battle_setup.c  battle_main.c  battle_controller_player.c  │
│  battle_script_commands.c  wild_encounter.c                 │
└─────────────────────────────────────────────────────────────┘
```

## Core Subsystems

### 1. Question Bank (Data Layer)
- **Purpose**: Store and retrieve quiz questions by scope/species
- **Location**: `/data/questions/` (JSON) → `/generated/` (C tables)
- **Status**: Schema designed, not yet populated

### 2. Progress Tracking (Save Layer)
- **Purpose**: Persist mastery bits, capture state, cleared species
- **Data Structure**: Bitmask per (scope, species) pair
- **Status**: Not yet implemented

### 3. Battle/Encounter Hooks (Logic Layer)
- **Purpose**: Intercept battle flow, grade answers, apply effects
- **Location**: `/pokefirered/src/quiz/quiz_hooks.c`
- **Status**: Partially implemented (hardcoded single question)

### 4. Quiz UI (Presentation Layer)
- **Purpose**: Display questions, answers, feedback
- **Location**: `/pokefirered/src/quiz/quiz.c`
- **Status**: Partially implemented (uses move menu)

## Data Flow

### Wild Encounter Flow
```
1. Wild encounter triggered
   └─> battle_setup.c::CreateBattleStartTask()
       └─> QuizHooks_OnWildBattleStart(species, mapGroup, mapNum)

2. Battle command menu shown
   └─> battle_controller_player.c (modified)
       └─> QuizHooks_OnCommandMenuShown()
           └─> Display question text in B_WIN_QUIZ window

3. Player hovers over move slots
   └─> QuizHooks_OnMoveCursorChanged(slot, moveId)
       └─> Display "Use [Move] to pick Answer [A-D]?"

4. Player confirms move
   └─> QuizHooks_OnMoveConfirmed(slot, moveId)
       └─> Grade answer (slot 0 = correct in current hardcode)

5. Damage calculation
   └─> QuizHooks_ApplyDamageOverride()
       └─> Correct: 1000 damage (KO)
       └─> Wrong: 1 damage, enemy does maxHP/2
```

## Integration Philosophy

**Minimal Engine Touches**:
- All custom logic lives in `/src/quiz/`
- Battle engine files only call into `quiz_hooks.h` functions
- No scattered modifications across battle files

## State Machine (Per Scope+Species)

```
S0: UNSEEN ──> S1: LEARNING ──> S2: CAPTURE_PENDING ──> S3: CLEARED
     │              │                   │
     └──────────────┴───────────────────┘
           (correct answers progress state)
```
