---
phase: 14-map-questions-to-encounters
plan: 02
subsystem: save-data
tags: [save-data, section-aware, quiz, migration, v2]

# Dependency graph
requires:
  - phase: 14-01
    provides: Section infrastructure (Quiz_GetSectionForMap, section constants)
provides:
  - QuizSaveDataV2 struct (386 bytes) in unused_348C space
  - Section-aware accessor functions for cleared/mastery tracking
  - V1 to V2 migration function
  - Global accessors now use current section internally
affects: [14-03, 14-04, 14-05]

# Tech tracking
tech-stack:
  added: []
  patterns:
    - Bit-array storage for cleared species per section
    - Modulo-28 slots for mastery tracking per section
    - Section-aware accessors with bounds checking

key-files:
  created: []
  modified:
    - pokefirered/include/global.h
    - pokefirered/include/quiz/quiz.h
    - pokefirered/src/quiz/quiz.c

key-decisions:
  - "386-byte V2 struct fits in 400-byte unused_348C space"
  - "Use bit-arrays for cleared species (160 species × 8 sections)"
  - "Use modulo-28 slots for mastery (collision-tolerant)"
  - "V1 migration resets mastery data (acceptable for section system)"

patterns-established:
  - "Section-aware accessors: InSection(species, section) pattern"
  - "Global accessors: internally call Quiz_GetCurrentSection()"

# Metrics
duration: ~8min
completed: 2026-02-04
---

# Phase 14 Plan 02: Section-Aware Save Data Summary

**QuizSaveDataV2 struct with section-aware cleared/mastery tracking using 386 of 400 bytes in unused_348C space**

## Performance

- **Duration:** ~8 min
- **Started:** 2026-02-04T20:15:00Z
- **Completed:** 2026-02-04T20:23:00Z
- **Tasks:** 3
- **Files modified:** 3

## Accomplishments
- QuizSaveDataV2 struct (386 bytes) occupies unused_348C space with STATIC_ASSERT verification
- sectionCleared[8][20] tracks 160 species × 8 sections as bits
- masteryMask[8][28] tracks mastery with modulo-28 slots per section
- Quiz_MigrateSaveData handles V1→V2 migration
- Existing global accessors now internally use current section

## Task Commits

Each task was committed atomically:

1. **Task 1: Define QuizSaveDataV2 struct in global.h** - `f287155da` (feat)
2. **Task 2: Add section-aware accessor declarations to quiz.h** - `620b7e04c` (feat)
3. **Task 3: Implement section-aware accessors in quiz.c** - `9c982f71e` (feat)

## Files Created/Modified
- `pokefirered/include/global.h` - Added QuizSaveDataV2 struct at unused_348C, marked V1 quizData as deprecated
- `pokefirered/include/quiz/quiz.h` - Added section-aware accessor declarations
- `pokefirered/src/quiz/quiz.c` - Implemented all V2 accessors, updated global accessors to use current section

## Decisions Made
- Used STATIC_ASSERT to verify struct size <= 400 bytes at compile time
- V1 migration resets all mastery data since V1 used species-indexed slots incompatible with section+species indexing
- Species bounds check at 160 (fits in 20 bytes × 8 bits per section)
- Section bounds check 1-8 (1-indexed to match QUIZ_SECTION_* constants)

## Deviations from Plan

None - plan executed exactly as written.

## Issues Encountered
None

## User Setup Required

None - no external service configuration required.

## Next Phase Readiness
- V2 save data structure is ready for use
- Quiz_GetCurrentSection() from 14-01 enables section-aware tracking
- Ready for 14-03 to integrate section lookup with question pool assignment

---
*Phase: 14-map-questions-to-encounters*
*Completed: 2026-02-04*
