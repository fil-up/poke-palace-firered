# Conventions

## Code Style (C)

### Naming
| Element | Convention | Example |
|---------|------------|---------|
| Functions | `PascalCase` | `Quiz_OnMoveConfirmed()` |
| Local variables | `camelCase` | `moveSlot`, `mapGroup` |
| Global variables | `gPascalCase` | `gBattleMons`, `gBattleTypeFlags` |
| Static variables | `sPascalCase` | `sQuizState` |
| Constants | `SCREAMING_SNAKE` | `QUIZ_ANSWER_COUNT` |
| Macros | `SCREAMING_SNAKE` | `GET_BATTLER_POSITION()` |
| Enums | `SCREAMING_SNAKE` values | `QUIZ_TURN_CORRECT` |
| Structs | `PascalCase` | `struct QuizState` |
| Typedefs | `lowercase_t` (rare) | Standard GBA types |

### GBA-Specific Types
| Type | Size | Usage |
|------|------|-------|
| `u8` | 8-bit unsigned | Small integers, flags |
| `u16` | 16-bit unsigned | Species IDs, moves |
| `u32` | 32-bit unsigned | Larger values |
| `s8`, `s16`, `s32` | Signed variants | When negatives needed |
| `bool8` | 8-bit boolean | TRUE/FALSE |

### Memory Sections
| Macro | Memory | Usage |
|-------|--------|-------|
| `EWRAM_DATA` | External RAM | Large persistent data |
| `IWRAM_DATA` | Internal RAM | Fast access data |
| `EWRAM_BSS` | EWRAM (uninitialized) | Zero-init buffers |

### String Handling
```c
// Use the _() macro for string literals (charmap translation)
static const u8 sQuizQuestionText[] = _("What does IBNR stand for?");

// String terminator
#define EOS 0xFF  // End of string
#define CHAR_NEWLINE 0xFE
```

## File Organization

### Headers
- Include guards: `#ifndef GUARD_FILENAME_H`
- All headers in `include/` mirror `src/` structure
- Quiz headers: `include/quiz/*.h`

### Source Files
- One subsystem per file
- Static functions for internal use
- Prefix module functions: `Quiz_`, `QuizHooks_`

## Quiz Module Conventions

### Hook Functions
All battle integration uses hooks with this pattern:
```c
// In quiz_hooks.c
void QuizHooks_OnEventName(params);

// In battle_*.c (minimal touch)
#include "quiz/quiz_hooks.h"
QuizHooks_OnEventName(params);
```

### State Management
```c
// Private state in EWRAM
static EWRAM_DATA struct QuizState sQuizState = {0};

// Public accessors
bool8 Quiz_IsActive(void);
enum QuizTurnResult Quiz_GetTurnResult(void);
```

## Build System

### Make Targets
| Target | Purpose |
|--------|---------|
| `make rom` | Build the GBA ROM |
| `make clean` | Remove build artifacts |
| `make validate` | Run validators (planned) |

### Build Flags
| Flag | Purpose |
|------|---------|
| `MODERN=0` | Use agbcc (Nintendo-matching) |
| `MODERN=1` | Use arm-none-eabi-gcc |

## Documentation

- Gameplay specs in `DESIGN.md`
- Technical specs in `ENGINEERING_BRIEF.md`
- Battle details in `BATTLE_SPEC.md`
- Per-file comments for non-obvious logic
