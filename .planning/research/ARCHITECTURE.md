# Architecture Research

## SaveBlock Pattern (Discovered)

FireRed uses `SaveBlock1` and `SaveBlock2` structures that persist to flash memory.

**Location:** `include/global.h` lines 759-830

**Key Insight:** SaveBlock1 contains:
- Player position, party, items
- **Unused bytes** at various offsets (can repurpose)
- Static size verified at compile time

**Recommended Approach for Quiz Data:**

```c
// Option A: Add to SaveBlock1 (if space available)
struct SaveBlock1
{
    // ... existing fields ...
    
    /*0x????*/ struct QuizSaveData quizData; // Add at end
};

// Option B: Repurpose unused fields
// Search for "unused" or padding in SaveBlock1

// Option C: Use existing variable/flag system
// gSaveBlock1Ptr->vars[VAR_QUIZ_*]
// Less structured but no struct changes
```

**Save Constraints:**
- SaveBlock1 spans 4 flash sectors (0x3D00 bytes max)
- SaveBlock2 is 1 sector (0xF80 bytes max)
- STATIC_ASSERT checks prevent overflow

## Battle Window System (Discovered)

**Window IDs:** `include/constants/battle.h`
- `B_WIN_QUIZ` already defined as ID 25 ✓
- Uses `BattlePutTextOnWindow()` for text display

**Text Box Pattern:**
```c
// Display text in quiz window
BattlePutTextOnWindow(textBuffer, B_WIN_QUIZ);

// Multi-page would use battle message system
// with wait-for-button callbacks
```

## Question Bank Architecture

**Recommended Structure:**
```c
// questions_gen.h (auto-generated)
struct QuizQuestion
{
    const u8 *prompt;      // Question text
    const u8 *choices[4];  // A, B, C, D
    u8 correctIndex;       // 0-3
    u8 difficulty;         // 1-5
    const u8 *explanation; // Why correct
};

struct QuizBank
{
    u16 speciesId;         // Which Pokémon
    u8 scopeId;            // Which route/area
    u8 questionCount;      // 5-10 typically
    const struct QuizQuestion *questions;
};

// Lookup function
const struct QuizBank *Quiz_GetBank(u16 species, u8 mapGroup, u8 mapNum);
```

## Data Flow Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     BUILD TIME                              │
├─────────────────────────────────────────────────────────────┤
│  CSV Files → validate_questions.py → build_questions.py    │
│                                              ↓              │
│                                    questions_gen.c/.h      │
└─────────────────────────────────────────────────────────────┘
                           ↓ compiled into ROM
┌─────────────────────────────────────────────────────────────┐
│                     RUNTIME                                 │
├─────────────────────────────────────────────────────────────┤
│  Wild Encounter                                             │
│       ↓                                                     │
│  Quiz_GetBank(species, mapGroup, mapNum)                   │
│       ↓                                                     │
│  Quiz_SelectQuestion(bank, masteryMask)                    │
│       ↓                                                     │
│  Display → Answer → Grade → Update Mastery                 │
│       ↓                                                     │
│  Save to QuizSaveData on save game                         │
└─────────────────────────────────────────────────────────────┘
```

## Scope-Species Mapping

**MVP Scope Definition:**
```c
// Simple: Use mapGroup + mapNum directly
#define SCOPE_ROUTE1          (MAP_GROUP(ROUTE1) << 8 | MAP_NUM(ROUTE1))
#define SCOPE_VIRIDIAN_FOREST (MAP_GROUP(VIRIDIAN_FOREST) << 8 | MAP_NUM(VIRIDIAN_FOREST))

// Save data: array indexed by scope + species
// Compact: bitmask for each (scope, species) pair
```

## Component Boundaries

| Component | Responsibility | Files |
|-----------|----------------|-------|
| quiz_bank | Question lookup/selection | quiz_bank.c/.h |
| quiz_save | Mastery persistence | quiz_save.c/.h |
| quiz | State machine, grading | quiz.c/.h |
| quiz_hooks | Battle engine integration | quiz_hooks.c/.h |
| quiz_ui | Text display, pagination | quiz_ui.c/.h (new) |

## Build Order Recommendation

1. **Foundation:** CSV → C pipeline, question bank structure
2. **Persistence:** SaveBlock integration, mastery bits
3. **Core Loop:** Question selection, grading flow
4. **Capture:** State machine, capture quiz
5. **Polish:** Multi-page text, explanations
6. **Content:** Populate question banks for MVP areas
