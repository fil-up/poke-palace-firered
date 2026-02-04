# Codebase Structure

**Analysis Date:** 2026-01-30

## Directory Layout

```
poke-palace-firered/
├── agbcc/                      # GBA C compiler toolchain (submodule)
│   ├── gcc/                    # Main GCC compiler
│   ├── gcc_arm/                # ARM cross-compiler
│   ├── libc/                   # Standard C library
│   └── libgcc/                 # GCC support library
├── pokefirered/                # FireRed decompilation (modified submodule)
│   ├── src/                    # C source files (~200 files)
│   │   ├── quiz/               # ★ Custom quiz system
│   │   ├── battle_*.c          # Battle engine files
│   │   ├── wild_encounter.c    # Wild encounter generation
│   │   ├── save.c              # Save/load system
│   │   └── ...
│   ├── include/                # Header files
│   │   ├── quiz/               # ★ Custom quiz headers
│   │   ├── battle.h            # Battle structures
│   │   ├── global.h            # Save block definitions
│   │   └── constants/          # Game constants
│   ├── data/                   # Game data (maps, Pokemon, etc.)
│   ├── graphics/               # Sprites, tilesets, UI
│   ├── sound/                  # Music and sound effects
│   ├── asm/                    # Assembly source files
│   ├── tools/                  # Build utilities
│   ├── build/                  # Build output (gitignored)
│   └── Makefile                # Main build configuration
├── data/                       # ★ Custom question data
│   └── questions/              # Question CSV files (if used)
├── tools/                      # ★ Custom build tools
│   └── quiz/                   # Question validation/generation
├── .planning/                  # GSD planning documents
│   ├── codebase/               # Codebase analysis docs
│   ├── phases/                 # Phase implementation plans
│   ├── research/               # Architecture research
│   ├── ROADMAP.md              # Project phases
│   └── STATE.md                # Current progress
├── DESIGN.md                   # ★ Gameplay behavior spec
├── BATTLE_SPEC.md              # ★ Battle integration spec
└── README.md                   # Project overview
```

## Directory Purposes

**`pokefirered/src/quiz/` (Custom Quiz System):**
- Purpose: All quiz-specific game logic
- Contains: State machine, hooks, question data, save functions
- Key files:
  - `quiz.c` - Main state machine, question selection, UI display
  - `quiz_hooks.c` - Battle engine integration hooks
  - `questions_gen.c` - Generated question banks
  - `trainer_scaling.c` - Trainer tier and question count logic
  - `route_completion.c` - Terrain completion tracking

**`pokefirered/include/quiz/` (Quiz Headers):**
- Purpose: Public interfaces for quiz subsystem
- Contains: Function declarations, structs, enums
- Key files:
  - `quiz.h` - Core quiz API
  - `quiz_hooks.h` - Hook function declarations
  - `questions_gen.h` - Generated question bank declarations
  - `trainer_scaling.h` - Trainer tier enums
  - `route_completion.h` - Completion API

**`pokefirered/src/` (FireRed Engine):**
- Purpose: Decompiled FireRed source code
- Contains: Battle engine, overworld, menus, Pokemon data
- Key modified files:
  - `battle_main.c` - Battle flow (calls `QuizHooks_OnBattleInit`)
  - `battle_controller_player.c` - Player input (calls quiz menu hooks)
  - `battle_script_commands.c` - Damage calculation (calls damage override)
  - `wild_encounter.c` - Encounter generation (filters cleared species)

**`pokefirered/include/` (Engine Headers):**
- Purpose: Type definitions, constants, function declarations
- Contains: Core game structures
- Key files:
  - `global.h` - SaveBlock1/2 definitions (extended with QuizSaveData)
  - `battle.h` - Battle structures and macros
  - `constants/battle.h` - Battle window IDs, flags

**`pokefirered/data/` (Game Data):**
- Purpose: Static game data compiled into ROM
- Contains: Maps, Pokemon stats, trainer data
- Key files:
  - `wild_encounters.h` - Per-route wild Pokemon tables

**`.planning/` (Project Management):**
- Purpose: GSD planning and tracking
- Contains: Phase plans, summaries, requirements
- Key files:
  - `ROADMAP.md` - Phase overview and status
  - `STATE.md` - Current task focus
  - `codebase/*.md` - Architecture analysis docs

## Key File Locations

**Entry Points:**
- `pokefirered/src/main.c`: Main entry point
- `pokefirered/src/battle_main.c`: Battle system entry
- `pokefirered/src/wild_encounter.c`: Wild encounter triggers

**Configuration:**
- `pokefirered/Makefile`: Build configuration
- `pokefirered/include/config.h`: Compile-time feature flags

**Quiz Core Logic:**
- `pokefirered/src/quiz/quiz.c`: State machine, question selection
- `pokefirered/src/quiz/quiz_hooks.c`: Battle integration

**Question Data:**
- `pokefirered/src/quiz/questions_gen.c`: All question content (generated)
- `pokefirered/include/quiz/questions_gen.h`: Question bank declarations

**Save System:**
- `pokefirered/include/global.h`: SaveBlock1 structure (lines 759-822)
- `pokefirered/src/quiz/quiz.c`: Quiz save accessors (lines 903-964)

**Battle Integration:**
- `pokefirered/src/battle_controller_player.c`: Player input handling
- `pokefirered/src/battle_script_commands.c`: Damage/effect execution

**Testing:**
- Manual playtesting only (GBA ROM)

## Naming Conventions

**Files:**
- `quiz_*.c/.h`: Quiz subsystem files
- `battle_*.c`: Battle engine components
- `*_gen.c/.h`: Auto-generated files (do not edit)
- `constants/*.h`: Constant definitions

**Functions:**
- `Quiz_*`: Core quiz API (e.g., `Quiz_InitWildEncounter`)
- `QuizHooks_*`: Battle hook functions (e.g., `QuizHooks_OnMoveConfirmed`)

**Variables:**
- `s*`: Static variables (e.g., `sQuizState`)
- `g*`: Global variables (e.g., `gBattleMons`)
- `EWRAM_DATA`: Marks data in extended working RAM

**Types:**
- `struct Quiz*`: Quiz-specific structures
- `enum Quiz*`: Quiz enumerations
- `u8/u16/u32`: Unsigned integers
- `s8/s16/s32`: Signed integers
- `bool8`: Boolean (8-bit)

## Where to Add New Code

**New Quiz Feature:**
- Primary code: `pokefirered/src/quiz/quiz.c` or new file in `quiz/`
- Headers: `pokefirered/include/quiz/`
- Hook integration: Add hook to `quiz_hooks.c`, call from battle file

**New Question Species:**
1. Add CSV data to `data/questions/` (if using CSV pipeline)
2. Run `python tools/quiz/build_questions.py` to regenerate
3. Add `extern const struct QuizBank gQuizBank_SPECIES_*` to header
4. Add lookup case to `Quiz_GetBank()` in `questions_gen.c`

**New Trainer Tier:**
- Definition: `pokefirered/include/quiz/trainer_scaling.h`
- Logic: `pokefirered/src/quiz/trainer_scaling.c`
- ID list: Add to `sGymTrainerIds` array if applicable

**New Battle Hook:**
1. Declare in `pokefirered/include/quiz/quiz_hooks.h`
2. Implement in `pokefirered/src/quiz/quiz_hooks.c`
3. Call from appropriate battle file (e.g., `battle_main.c`)

**Modifying Save Data:**
- Structure: Modify `QuizSaveData` in `pokefirered/include/global.h`
- Increment version number in `Quiz_InitSaveData()`
- Ensure size fits within SaveBlock1 bounds

## Special Directories

**`pokefirered/build/`:**
- Purpose: Compiled object files and ROM output
- Generated: Yes (by make)
- Committed: No (gitignored)

**`pokefirered/tools/`:**
- Purpose: Build-time utilities (graphics, map processing)
- Generated: No
- Committed: Yes

**`agbcc/`:**
- Purpose: GBA C compiler (required for building)
- Generated: No (submodule)
- Committed: Submodule reference only

**`.planning/`:**
- Purpose: GSD workflow documents
- Generated: By GSD commands
- Committed: Yes

## Import Patterns

**Standard Include Order:**
```c
#include "global.h"           // Always first
#include "gflib.h"            // Graphics library
#include "battle.h"           // Battle structures
#include "quiz/quiz.h"        // Quiz core API
#include "quiz/quiz_hooks.h"  // Hook declarations
#include "constants/maps.h"   // Map constants
#include "constants/moves.h"  // Move IDs
```

**Quiz System Includes:**
```c
#include "quiz/quiz.h"           // Core state machine
#include "quiz/quiz_hooks.h"     // Battle hooks
#include "quiz/questions_gen.h"  // Question data
#include "quiz/trainer_scaling.h"// Trainer difficulty
#include "quiz/route_completion.h"// Terrain tracking
```

## Build System

**Main Makefile:** `pokefirered/Makefile`

**Key Targets:**
- `make` - Build ROM (pokefirered.gba)
- `make clean` - Remove build artifacts
- `make questions` - Regenerate question data (if configured)

**Quiz File Compilation:**
All `.c` files in `pokefirered/src/quiz/` are automatically included via wildcard in Makefile.

**Question Generation Pipeline:**
```
data/questions/*.csv
    ↓ (validate_questions.py)
    ↓ (build_questions.py)
pokefirered/src/quiz/questions_gen.c
pokefirered/include/quiz/questions_gen.h
    ↓ (make)
pokefirered.gba
```

---

*Structure analysis: 2026-01-30*
