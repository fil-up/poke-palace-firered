---
phase: 14-map-questions-to-encounters
plan: 01
subsystem: quiz-section-infrastructure
tags: [section-config, map-lookup, topic-pools, json-config]
dependency-graph:
  requires: []
  provides: 
    - Quiz_GetSectionForMap function
    - Quiz_GetCurrentSection function
    - QUIZ_NUM_SECTIONS constant
    - QUIZ_SECTION_* constants
    - map_topics.json topic configuration
    - section_config.json section definitions
  affects:
    - 14-02 (integrate topic pools with question selection)
    - 14-03 (section-based mastery tracking)
tech-stack:
  added: []
  patterns:
    - Map ID switch statement for section lookup
    - JSON configuration for build tool consumption
key-files:
  created:
    - pokefirered/include/quiz/section_config.h
    - pokefirered/src/quiz/section_config.c
    - pokefirered/data/questions/map_topics.json
    - pokefirered/data/questions/section_config.json
  modified: []
decisions:
  - id: SEC-001
    title: Section constants use 1-indexed values
    context: Sections map to gym numbers (1-8)
    choice: QUIZ_SECTION_BROCK=1 through QUIZ_SECTION_CHAMPION=8
    rationale: Aligns with gym badge numbering, more intuitive for game logic
  - id: SEC-002
    title: Default section fallback
    context: Unmapped areas need a default section
    choice: Return QUIZ_SECTION_BROCK (1) for unknown maps
    rationale: Ensures all encounters work, uses simplest topics for edge cases
metrics:
  duration: ~15 minutes
  completed: 2026-02-04
---

# Phase 14 Plan 01: Section Infrastructure Summary

Section constants, map-to-section lookup, and topic configuration files for quiz system.

## One-liner

8-section quiz infrastructure with Quiz_GetSectionForMap(), section constants, and JSON topic pools from 14-CONTEXT.md.

## What Changed

### Task 1: section_config.h
Created header file with:
- `QUIZ_NUM_SECTIONS` constant (8 sections total)
- Section constants: `QUIZ_SECTION_BROCK` through `QUIZ_SECTION_CHAMPION` (1-8)
- Function declarations: `Quiz_GetSectionForMap()` and `Quiz_GetCurrentSection()`

### Task 2: section_config.c
Implemented map-to-section lookup with comprehensive switch statement covering:
- **Section 1 (Pre-Brock):** Pallet Town, Route 1-2, Viridian, Viridian Forest, Pewter
- **Section 2 (Pre-Misty):** Route 3-4, Mt. Moon (all floors), Cerulean, Route 24-25
- **Section 3 (Pre-Surge):** Route 5-6, Vermilion, S.S. Anne (all rooms), Route 11, Diglett's Cave
- **Section 4 (Pre-Erika):** Route 7-10, Rock Tunnel, Lavender, Pokemon Tower (all floors), Celadon, Rocket Hideout
- **Section 5 (Pre-Koga):** Route 12-15, Fuchsia, Safari Zone (all areas)
- **Section 6 (Pre-Sabrina):** Saffron City (all buildings), Silph Co. (all floors)
- **Section 7 (Pre-Blaine):** Route 16-21, Seafoam Islands (all floors), Cinnabar, Pokemon Mansion (all floors)
- **Section 8 (Pre-Champion):** Route 22-23, Victory Road (all floors), Indigo Plateau, Viridian Gym, Cerulean Cave

Also implemented `Quiz_GetCurrentSection()` using `gSaveBlock1Ptr->location`.

### Task 3: JSON Configuration Files
Created two JSON files for build tool consumption:

**section_config.json:**
- Section boundaries with gym gates
- Maps array per section (with wildcard notation)

**map_topics.json:**
- Complete `question_pool_by_map` from 14-CONTEXT.md
- Complete `gyms_by_map` with mastery pools
- Elite Four configuration with category focus

## Technical Details

### Map ID Structure
Map IDs in pokefirered use format: `(mapNum | (mapGroup << 8))`
- Group 1: Dungeons (Viridian Forest, Mt. Moon, caves, etc.)
- Group 3: Towns, cities, and routes
- Groups 4-14: Town interiors organized by city

### Quiz_GetCurrentSection Implementation
```c
u8 Quiz_GetCurrentSection(void)
{
    u16 currentMapId = (gSaveBlock1Ptr->location.mapGroup << 8) | gSaveBlock1Ptr->location.mapNum;
    return Quiz_GetSectionForMap(currentMapId);
}
```

## Decisions Made

| ID | Decision | Rationale |
|----|----------|-----------|
| SEC-001 | 1-indexed section constants | Aligns with gym badge numbers |
| SEC-002 | Default to section 1 | Safe fallback using simplest topics |

## Deviations from Plan

None - plan executed exactly as written.

## Verification Status

- [x] section_config.h created with QUIZ_NUM_SECTIONS=8
- [x] section_config.c created with Quiz_GetSectionForMap implementation
- [x] map_topics.json valid JSON with complete topic pools
- [x] section_config.json valid JSON with section definitions
- [ ] Git commits (PowerShell/WSL integration issue - manual execution needed)
- [ ] Full build verification (manual execution needed)

**Note:** PowerShell/WSL command execution has an ENOENT error preventing automated git commits. Please run the following commands manually in WSL:

```bash
cd /home/filuptomis/poke-palace-firered/pokefirered

# Commit section_config.h and section_config.c
git add include/quiz/section_config.h src/quiz/section_config.c
git commit -m "feat(14-01): add section_config.h and section_config.c

- Define QUIZ_NUM_SECTIONS=8 for gym-gated progression
- Define QUIZ_SECTION_* constants (1-8) for each gym gate
- Implement Quiz_GetSectionForMap() with map ID switch statement
- Implement Quiz_GetCurrentSection() using gSaveBlock1Ptr location
- Cover all Kanto maps across 8 sections (pre-Brock to pre-Champion)
"

# Commit JSON configuration files
git add data/questions/map_topics.json data/questions/section_config.json
git commit -m "feat(14-01): add JSON topic configuration files

- map_topics.json: topic pools per map from 14-CONTEXT.md
- section_config.json: section boundaries with gym gates
- gyms_by_map: mastery pools for all 8 gym leaders
- elite_four: category focus for E4 and Champion
"

# Build verification
make -j4
```

## Next Phase Readiness

Phase 14-02 can proceed:
- Section lookup functions available for import
- JSON configuration ready for build tool integration
- All map IDs mapped to sections 1-8

## Files Created

| File | Purpose |
|------|---------|
| `include/quiz/section_config.h` | Section constants and function declarations |
| `src/quiz/section_config.c` | Map-to-section lookup implementation |
| `data/questions/map_topics.json` | Topic pools per map and gym |
| `data/questions/section_config.json` | Section boundary definitions |
