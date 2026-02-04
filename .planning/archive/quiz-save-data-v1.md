# Quiz Save Data V1 - Historical Reference

**Archived**: Phase 14 migration to section-based system  
**Date**: 2026-02-04

## Overview

V1 used a simple species-indexed mastery system with modulo-31 slots. This was replaced in Phase 14 with section-aware tracking.

## Structures (from global.h)

```c
// Quiz save data - stored in SaveBlock1 padding
#define QUIZ_SAVE_SPECIES_COUNT 31

struct QuizSpeciesSave
{
    u8 masteryMask;  // Bitmask of mastered questions (up to 8)
    u8 state;        // QUIZ_STATE_* constant
};

struct QuizSaveData
{
    u8 version;      // Save format version for migrations
    u8 padding;      // Alignment padding
    struct QuizSpeciesSave species[QUIZ_SAVE_SPECIES_COUNT];  // 31 * 2 = 62 bytes
}; // Total: 64 bytes
```

## SaveBlock1 Location

```c
/*0x3A94*/ struct QuizSaveData quizData;  // Quiz mastery/state data
```

## How It Worked

1. **Species indexing**: `slot = species % 31` (modulo-31 to fit 31 slots)
2. **Mastery tracking**: 8-bit mask for up to 8 questions per species
3. **State tracking**: QUIZ_STATE_NONE, QUIZ_STATE_CLEARED, QUIZ_STATE_CAPTURE_PENDING
4. **Global cleared**: Species cleared once = cleared everywhere

## Limitations (Why V2 Was Needed)

1. **No section awareness**: Couldn't track same species in different areas
2. **Collision issues**: Modulo-31 caused species collision (Bulbasaur=1 and Diglett=32 shared slot)
3. **No re-capture**: Once a species was cleared, it stayed cleared globally
4. **Limited capacity**: Only 31 slots for 151+ species

## Migration Path

V1 saves are migrated to V2 by:
1. Detecting `version == 0 || version == 1` in V2 location
2. Initializing fresh V2 structure
3. Setting `version = 2`

Note: V1 mastery data is NOT migrated (fresh start on section system).

## Related Files

- `pokefirered/include/global.h` - Struct definitions
- `pokefirered/src/quiz/quiz.c` - Save data accessors
- `pokefirered/include/quiz/quiz.h` - Function declarations
