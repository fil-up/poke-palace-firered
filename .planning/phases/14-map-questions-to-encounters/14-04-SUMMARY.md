---
phase: 14-map-questions-to-encounters
plan: 04
subsystem: quiz-logic
tags: [topic-pools, quiz-selection, section-aware, gym-battles]
dependency-graph:
  requires:
    - 14-02 (QuizSaveDataV2 with section-aware accessors)
    - 14-03 (Topic pools with Quiz_GetRandomQuestionForMap, Quiz_GetGymConfig)
  provides:
    - Topic-based question selection for wild encounters
    - Section-aware mastery counting
    - Gym-specific topic pools for gym battles
    - Map-based question selection for trainer battles
  affects:
    - 14-05 (testing and verification)
tech-stack:
  added: []
  patterns:
    - Wild encounters use map topic pools instead of species banks
    - Gym battles use Quiz_GetGymConfig for gym-specific pools
    - Single-question trainers use Quiz_GetRandomQuestionForMap
    - Fallback to species bank when no topic pool available
key-files:
  created: []
  modified:
    - pokefirered/src/quiz/quiz.c
    - pokefirered/include/quiz/quiz.h
decisions:
  - id: TOPIC-SELECT-001
    title: Topic pool primary, species bank fallback
    context: Wild encounters need question selection from topic pools
    choice: Try Quiz_GetRandomQuestionForMap first, fallback to species bank
    rationale: Enables section-based learning while maintaining capture mechanics
  - id: TOPIC-SELECT-002
    title: Gym ID lookup from map
    context: Quiz_GetGymConfig takes gymId but we have mapId
    choice: Added Quiz_GetGymIdFromMap helper function
    rationale: Simple switch mapping gym maps to gym IDs (1-8)
  - id: TOPIC-SELECT-003
    title: Track question source for mastery
    context: Topic pool questions have no species index
    choice: Use currentQuestionIndex == 0xFF to indicate topic pool source
    rationale: Skip species mastery update for topic pool questions
metrics:
  duration: ~15 minutes
  completed: 2026-02-04
---

# Phase 14 Plan 04: Topic-Based Question Selection Summary

Converted quiz logic from species-based to topic-based question selection while maintaining capture mechanics.

## One-liner

Quiz_InitWildEncounter uses Quiz_GetRandomQuestionForMap for topic-based selection; gym battles use Quiz_GetGymConfig for gym-specific pools.

## What Changed

### Task 1: Update Quiz_InitWildEncounter for Topic-Based Selection

Added topic-based question selection for wild encounters:
- Added `currentMapId` and `currentSection` fields to QuizState struct
- Wild encounters call `Quiz_GetRandomQuestionForMap(mapId)` for questions
- Fallback to species bank when no topic pool available for the map
- Updated mastery tracking to handle topic pool questions (currentQuestionIndex == 0xFF)
- Reset new state fields in Quiz_ResetBattleState()

### Task 2: Section-Aware Mastery Counting

Added explicit mastery counting function:
- Added `Quiz_CountMasteredQuestionsForSpecies()` declaration to quiz.h
- Implementation counts set bits in mastery mask using current section
- Existing section-aware infrastructure from 14-02 already handles other cases
- Quiz_GetMasteryMask/Quiz_SetMasteryMask delegate to section-aware variants

### Task 3: Gym Battle Logic for Topic Pools

Updated trainer battle question selection:
- Added `Quiz_GetGymIdFromMap()` to map gym maps to gym IDs (1-8)
- Added `Quiz_AddTopicPoolToPool()` to add topic pool questions to gymPool
- Added `Quiz_BuildGymPool(gymId)` using Quiz_GetGymConfig mastery pools
- Added `Quiz_BuildMapPool(mapId)` using Quiz_GetMapConfig topic pools
- Gym leaders use gym-specific topic pools from Quiz_GetGymConfig
- Other trainers use map topic pools via Quiz_GetRandomQuestionForMap
- Single-question trainers try map pools first, fallback to species bank

## Technical Details

### Question Source Tracking

```c
// In QuizState struct
u16 currentMapId;      // Map where encounter started
u8 currentSection;     // Section for this encounter (1-8)
u8 currentQuestionIndex; // 0xFF = from topic pool, else = species bank index
```

### Pool Building Functions

| Function | Purpose |
|----------|---------|
| Quiz_BuildGymPool(gymId) | Build from gym's masteryPools |
| Quiz_BuildMapPool(mapId) | Build from map's topicPools |
| Quiz_BuildAreaPool() | Legacy species-based (fallback) |
| Quiz_BuildFullPool() | All species banks (fallback) |

### Question Selection Flow

1. **Wild encounters**: Quiz_GetRandomQuestionForMap → species bank → no quiz
2. **Gym leaders**: Quiz_BuildGymPool → Quiz_SelectGymPoolQuestion
3. **Other trainers**: Quiz_GetRandomQuestionForMap → species bank → full pool

## Decisions Made

| ID | Decision | Rationale |
|----|----------|-----------|
| TOPIC-SELECT-001 | Topic pool primary | Enables section-based learning |
| TOPIC-SELECT-002 | Quiz_GetGymIdFromMap | Simple map → gymId mapping |
| TOPIC-SELECT-003 | currentQuestionIndex == 0xFF | Distinguishes topic pool questions |

## Deviations from Plan

None - plan executed exactly as written.

## Verification Status

- [x] Wild encounter questions come from current map's topic pool
- [x] Mastery is tracked per-section (via existing infrastructure)
- [x] Capture still requires 5 correct answers (species bank used)
- [x] Gym battles use gym mastery pools
- [x] Build compiles without errors

## Next Phase Readiness

Phase 14-05 can proceed:
- Topic-based question selection fully integrated
- Gym battles use gym-specific topic pools
- Section-aware mastery tracking in place
- Ready for testing and verification

## Files Modified

| File | Changes |
|------|---------|
| `src/quiz/quiz.c` | Topic pool selection, gym pool building, helper functions |
| `include/quiz/quiz.h` | Quiz_CountMasteredQuestionsForSpecies declaration |

## Commits

| Hash | Message |
|------|---------|
| 3f600f61d | feat(14-04): update Quiz_InitWildEncounter for topic-based selection |
| 35139ab63 | feat(14-04): add Quiz_CountMasteredQuestionsForSpecies for section-aware mastery |
| ddc67ef6c | feat(14-04): update gym battle logic for topic-based question pools |
