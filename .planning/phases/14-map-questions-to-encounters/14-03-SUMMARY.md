---
phase: 14-map-questions-to-encounters
plan: 03
subsystem: quiz-build-tools
tags: [topic-pools, build-script, code-generation, map-config]
dependency-graph:
  requires:
    - 14-01 (section_config.h, map_topics.json)
  provides:
    - QuizTopicPool struct and topic pool data
    - Quiz_GetMapConfig() lookup function
    - Quiz_GetGymConfig() lookup function
    - Quiz_GetRandomQuestionForMap() helper
    - Quiz_GetQuestionCountForMap() helper
  affects:
    - 14-04 (quiz logic uses topic pools)
tech-stack:
  added: []
  patterns:
    - Topic pools generated from CSV category+topic grouping
    - Map configs link maps to their topic pools
    - Gym configs provide mastery pools for gym battles
key-files:
  created: []
  modified:
    - pokefirered/tools/quiz/build_questions.py
    - pokefirered/include/quiz/questions_gen.h
    - pokefirered/src/quiz/questions_gen.c
decisions:
  - id: TOPIC-001
    title: Generate topic pools in questions_gen.c
    context: Topic pools reference sQuestion_* structs which are static
    choice: Generate all topic pool data in questions_gen.c alongside question structs
    rationale: Avoids cross-file references and extern complexity
  - id: TOPIC-002
    title: Category/topic normalization mapping
    context: CSV uses "Plan and Product Provisions", JSON uses "Plan Provisions"
    choice: Add CATEGORY_MAPPING and TOPIC_MAPPING dicts in build script
    rationale: Aligns CSV column values with JSON config keys
  - id: TOPIC-003
    title: Map name correction
    context: JSON used MAP_S_S_ANNE_1F and MAP_PEWTER_MUSEUM_1F
    choice: Fixed to MAP_SSANNE_1F_CORRIDOR and MAP_PEWTER_CITY_MUSEUM_1F
    rationale: Must match actual constants in map_groups.h
metrics:
  duration: ~20 minutes
  completed: 2026-02-04
---

# Phase 14 Plan 03: Topic Pool Build Tools Summary

Updated build_questions.py to generate topic-indexed question pools from map_topics.json.

## One-liner

Topic pool generation in build_questions.py with Quiz_GetMapConfig(), Quiz_GetGymConfig(), and Quiz_GetRandomQuestionForMap() helpers.

## What Changed

### Task 1: Updated build_questions.py

Added topic pool generation capability:
- `--topics` argument for map_topics.json path
- `load_topic_config()` to load JSON configuration
- `group_by_topic()` with category/topic normalization
- `CATEGORY_MAPPING` and `TOPIC_MAPPING` for CSV/JSON alignment
- Topic pool structs and data generation in `generate_source()`
- Map config and gym config generation from JSON

### Task 2: Generated Topic Pools

Updated questions_gen.h/c with:
- `struct QuizTopicPool` - questions grouped by category+topic
- `struct QuizMapConfig` - links maps to their topic pools
- `struct QuizGymConfig` - gym mastery pools for gym battles
- Topic pool declarations and data for all category+topic combinations
- `Quiz_GetMapConfig(mapId)` lookup function
- `Quiz_GetGymConfig(gymId)` lookup function

### Task 3: Helper Functions

Added convenience functions:
- `Quiz_GetRandomQuestionForMap(mapId)` - selects random question from map's topic pools
- `Quiz_GetQuestionCountForMap(mapId)` - counts total questions available for map

## Technical Details

### Topic Pool Structure

```c
struct QuizTopicPool {
    const struct QuizQuestion * const *questions;
    u16 count;
};

struct QuizMapConfig {
    u16 mapId;
    const struct QuizTopicPool * const *topicPools;
    u8 topicCount;
};
```

### Category Normalization

CSV categories like "Plan and Product Provisions" are normalized to match JSON keys:
- "Plan and Product Provisions" → "Plan Provisions"
- Ensures topic pools are correctly linked to map configs

## Decisions Made

| ID | Decision | Rationale |
|----|----------|-----------|
| TOPIC-001 | Generate in questions_gen.c | Avoids extern complexity |
| TOPIC-002 | Category/topic mapping | Aligns CSV with JSON |
| TOPIC-003 | Fixed map names | Match map_groups.h constants |

## Deviations from Plan

1. **Map name corrections**: Fixed MAP_S_S_ANNE_1F → MAP_SSANNE_1F_CORRIDOR and MAP_PEWTER_MUSEUM_1F → MAP_PEWTER_CITY_MUSEUM_1F to match actual game constants.

## Verification Status

- [x] build_questions.py accepts --topics argument
- [x] Topic pools generated for each category+topic
- [x] Map configs generated from question_pool_by_map
- [x] Gym configs generated from gyms_by_map
- [x] Quiz_GetRandomQuestionForMap() implemented
- [x] Build compiles successfully

## Next Phase Readiness

Phase 14-04 can proceed:
- Topic pools available via Quiz_GetMapConfig()
- Random question selection via Quiz_GetRandomQuestionForMap()
- Gym mastery pools via Quiz_GetGymConfig()

## Files Modified

| File | Changes |
|------|---------|
| `tools/quiz/build_questions.py` | Added topic pool generation |
| `include/quiz/questions_gen.h` | Added topic pool structs and declarations |
| `src/quiz/questions_gen.c` | Added topic pool data and lookup functions |
| `data/questions/map_topics.json` | Fixed map name typos |
