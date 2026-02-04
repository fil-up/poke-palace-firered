# Coding Conventions

**Analysis Date:** 2026-01-30

## Naming Patterns

### C Language Conventions

| Element | Convention | Example |
|---------|------------|---------|
| Functions | `Module_PascalCase` | `Quiz_OnMoveConfirmed()`, `Quiz_GetMasteryMask()` |
| Local variables | `camelCase` | `moveSlot`, `mapGroup`, `questionIndex` |
| Global variables | `gPascalCase` | `gBattleMons`, `gBattleTypeFlags`, `gSaveBlock1Ptr` |
| Static variables | `sPascalCase` | `sQuizState`, `sFillerMoves`, `sCompletionCache` |
| Constants (defines) | `SCREAMING_SNAKE` | `QUIZ_ANSWER_COUNT`, `QUIZ_FRAME_BASE_TILE` |
| Macros | `SCREAMING_SNAKE` | `GET_BATTLER_POSITION()`, `ARRAY_COUNT()` |
| Enums | `PascalCase` type, `SCREAMING_SNAKE` values | `enum QuizTurnResult { QUIZ_TURN_CORRECT }` |
| Structs | `PascalCase` | `struct QuizState`, `struct QuizQuestion` |
| Header guards | `GUARD_PATH_H` | `#ifndef GUARD_QUIZ_H` |

### Files

| Type | Convention | Example |
|------|------------|---------|
| Source files | `lowercase_snake.c` | `quiz.c`, `quiz_hooks.c`, `trainer_scaling.c` |
| Header files | `lowercase_snake.h` | `quiz.h`, `questions_gen.h` |
| Constants headers | `constants/name.h` | `constants/trainers.h`, `constants/maps.h` |
| Generated files | `*_gen.{h,c}` | `questions_gen.h`, `questions_gen.c` |

## Code Style

### Include Organization

Order includes in this sequence:
1. `"global.h"` (always first)
2. Library headers (`"gflib.h"`, etc.)
3. Core engine headers (`"battle.h"`, `"pokemon.h"`)
4. Module headers (`"quiz/quiz.h"`)
5. Constants headers (`"constants/maps.h"`)

```c
#include "global.h"
#include "gflib.h"
#include "battle.h"
#include "battle_message.h"
#include "pokemon.h"
#include "quiz/quiz.h"
#include "quiz/trainer_scaling.h"
#include "constants/maps.h"
#include "constants/moves.h"
```

### Function Organization Within Files

1. Includes and defines at top
2. Static const data tables
3. Static EWRAM_DATA state structures
4. Forward declarations for static functions
5. Static helper functions (internal)
6. Public API functions

### GBA-Specific Types

| Type | Size | Usage |
|------|------|-------|
| `u8` | 8-bit unsigned | Loop counters, small values, indices |
| `u16` | 16-bit unsigned | Species IDs, move IDs, trainer IDs |
| `u32` | 32-bit unsigned | Flags, large values, pointers in some contexts |
| `s8`, `s16`, `s32` | Signed variants | When negative values possible |
| `bool8` | 8-bit boolean | `TRUE`/`FALSE` values |

### Memory Sections

| Macro | Memory Region | Usage |
|-------|---------------|-------|
| `EWRAM_DATA` | External Work RAM | Persistent runtime state |
| `IWRAM_DATA` | Internal Work RAM | Fast-access data |
| `EWRAM_BSS` | EWRAM (uninitialized) | Zero-initialized buffers |
| (none/const) | ROM | Read-only data tables |

```c
// Runtime state in EWRAM
static EWRAM_DATA struct QuizState sQuizState = {0};

// Read-only data in ROM
static const u16 sLegendarySpecies[] = { SPECIES_ARTICUNO, ... };
```

## Error Handling

### Validation Pattern

Use early returns with guard clauses:

```c
static bool8 Quiz_IsValidSpecies(u16 species)
{
    return species > SPECIES_NONE && species < NUM_SPECIES;
}

void Quiz_SomeFunction(u16 species)
{
    // Early return for invalid input
    if (!Quiz_IsValidSpecies(species))
        return;
    
    // Main logic follows...
}
```

### Null Checks

Always validate pointers before use:

```c
if (bank == NULL || bank->count == 0)
    return 0xFF;
```

### Unused Parameter Suppression

Cast to void for intentionally unused parameters:

```c
void SomeFunction(u8 param1, u8 param2)
{
    (void)param2;  // Suppress unused warning
    // Use only param1...
}
```

## Decomp-Specific Patterns

### String Handling

```c
// Use _() macro for all user-visible strings (charmap translation)
static const u8 sQuizNoQuestionText[] = _("No questions available.");

// String constants
#define EOS 0xFF        // End of string marker
#define CHAR_NEWLINE 0xFE  // Line break

// String buffer for dynamic text
static u8 sQuizTextBuffer[256];

// String manipulation
cursor = StringCopy(buffer, sourceText);
cursor = StringAppend(cursor, moreText);
```

### External References

```c
// Declare extern for global data defined elsewhere
extern u16 gTrainerBattleOpponent_A;
extern const struct Trainer gTrainers[];
extern struct Pokemon gEnemyParty[PARTY_SIZE];
```

### Hook Pattern for Engine Integration

Minimize touches to core engine files by routing through hook functions:

```c
// In quiz/quiz_hooks.h - public interface
void QuizHooks_OnBattleInit(void);
bool8 QuizHooks_ShouldForcePlayerFirst(void);
bool8 QuizHooks_ApplyDamageOverride(s32 *damage, u8 attacker, u8 target);

// In battle_script_commands.c - minimal change
#include "quiz/quiz_hooks.h"
if (QuizHooks_ApplyDamageOverride(&damage, attacker, target))
    return;
```

### State Machine Pattern

```c
// Define states as enum
enum QuizTurnResult
{
    QUIZ_TURN_NONE,
    QUIZ_TURN_CORRECT,
    QUIZ_TURN_WRONG,
    QUIZ_TURN_CAPTURE_SUCCESS,
    // ...
};

// Store in static state struct
static EWRAM_DATA struct QuizState sQuizState = {0};

// Provide accessor functions
enum QuizTurnResult Quiz_GetTurnResult(void)
{
    return sQuizState.turnResult;
}
```

## Module Design

### Quiz Module Structure

| File | Purpose |
|------|---------|
| `src/quiz/quiz.c` | Core quiz logic, state management, UI |
| `src/quiz/quiz_hooks.c` | Battle engine integration hooks |
| `src/quiz/trainer_scaling.c` | Trainer tier calculation, evolution helpers |
| `src/quiz/route_completion.c` | Terrain completion tracking with caching |
| `src/quiz/questions_gen.c` | Generated question data (DO NOT EDIT) |
| `include/quiz/quiz.h` | Public quiz API |
| `include/quiz/quiz_hooks.h` | Hook function declarations |
| `include/quiz/trainer_scaling.h` | Trainer tier enums and functions |
| `include/quiz/questions_gen.h` | Generated question structures |

### Function Prefixes

| Prefix | Usage |
|--------|-------|
| `Quiz_` | Core quiz logic functions |
| `QuizHooks_` | Battle engine integration points |
| (module)_ | Standard decomp module functions |

## Constants and Configuration

### Define Placement

- File-local constants: at top of .c file
- Shared constants: in dedicated header under `include/constants/`
- Generated constants: in `*_gen.h` files

```c
// File-local
#define QUIZ_FRAME_BASE_TILE 0x090
#define QUIZ_FRAME_PALETTE   7
#define MAX_CACHED_HEADERS 8

// From questions_gen.h
#define QUIZ_TOTAL_QUESTIONS 155
```

### Static Const Arrays

```c
// ROM-resident lookup tables
static const u16 sGymTrainerIds[] =
{
    TRAINER_CAMPER_LIAM,
    TRAINER_PICNICKER_DIANA,
    // ...
};

// Use ARRAY_COUNT for iteration
for (i = 0; i < ARRAY_COUNT(sGymTrainerIds); i++)
```

## Comments

### When to Comment

- Non-obvious algorithms or GBA hardware quirks
- Section separators for long files
- External dependencies and why they exist

```c
// ============================================
// Mastery Helpers
// ============================================

// Ensure BG1 has highest priority so quiz window appears in front of
// healthbox sprites. Some battle operations (level-up box, animations)
// change BG1's priority, and it may not be reset before we show the quiz.
SetBgAttribute(1, BG_ATTR_PRIORITY, 0);
```

### Documentation in Headers

```c
// Save data functions
void Quiz_InitSaveData(void);
u8 Quiz_GetMasteryMask(u16 species);

// Trainer multi-question functions (trainer battles with multiple questions)
bool8 Quiz_IsGymLeaderMode(void);
```

## Import/Export Organization

### Headers

All public functions declared in corresponding header:

```c
// quiz.h
#ifndef GUARD_QUIZ_H
#define GUARD_QUIZ_H

#include "global.h"

// Enum declarations
enum QuizTurnResult { ... };

// Function declarations
void Quiz_InitWildEncounter(u16 species, u8 mapGroup, u8 mapNum);
bool8 Quiz_IsActive(void);

#endif // GUARD_QUIZ_H
```

### Static vs Public

- Use `static` for all internal functions
- Only expose what's needed in headers
- Prefix static data with `s`, public with `g`

## Build Considerations

### agbcc vs Modern GCC

| Flag | Compiler | Notes |
|------|----------|-------|
| `MODERN=0` | agbcc | Matches original ROM binary (default) |
| `MODERN=1` | arm-none-eabi-gcc | More warnings, faster builds |

### File-Specific Compiler Flags

Defined in Makefile for specific optimization needs:

```makefile
$(C_BUILDDIR)/m4a.o: CC1 := $(TOOLS_DIR)/agbcc/bin/old_agbcc$(EXE)
$(C_BUILDDIR)/trainer_tower.o: CFLAGS += -ffreestanding
```

---

*Convention analysis: 2026-01-30*
