# Project State

## Project Reference

See: .planning/PROJECT.md (updated 2026-01-24)

**Core value:** Quiz-driven gameplay loop must work end-to-end
**Current focus:** Phase 15 - Per-Species Questions with Unique Location Species

## Current Position

Phase: 15 of 15 (Per-Species Questions)
Plan: 2 of 4 complete (15-03, 15-04 done; 15-01, 15-02 partially done)
Status: Core code changes complete, content expansion pending
Next: Complete encounter table redesign and species-question assignment
Last activity: 2026-02-05 — Completed Phase 15 code changes (save system, quiz logic)

Progress: [██████████░░░░░░░░░░] 50% (core code done, content pending)

## Performance Metrics

**Velocity:**
- Total plans completed: 25
- Average duration: N/A
- Total execution time: ~2 hours for Phase 14

**By Phase:**

| Phase | Plans | Total | Avg/Plan |
|-------|-------|-------|----------|
| 14 | 5 | ~2h | ~24min |
| 15 | 2 | ~1h | ~30min |

## Accumulated Context

### Decisions

Decisions are logged in PROJECT.md Key Decisions table.
Recent decisions affecting current work:

- [Init]: Use CSV → Python → C tables pipeline
- [Init]: Extend SaveBlock1 for quiz data
- [Init]: B_WIN_QUIZ (ID 25) for question display
- [14-01]: 8 sections aligned with gym progression
- [14-02]: QuizSaveDataV2 in unused_348C (386 bytes)
- [14-03]: Topic pools generated in questions_gen.c
- [14-04]: Quiz_GetRandomQuestionForMap for question selection
- [14-05]: Derive CAPTURE_PENDING from mastery count >= 5
- [15-03]: Global species tracking in QuizSaveDataV2 (V3 format)
- [15-04]: Species bank selection via Quiz_GetBank(species)

### Phase 15 Summary (In Progress)

**Per-Species Questions with Unique Location Species**

1. **15-01**: Encounter table redesign — PARTIAL
   - Species-location mapping document created (15-SPECIES-MAP.md)
   - Sections 1-2 encounter tables updated
   - Remaining sections pending content expansion

2. **15-02**: Question-to-species assignment — PARTIAL
   - species_map.json updated for Section 1-2 species
   - Remaining species pending

3. **15-03**: Save system simplification — COMPLETE
   - QuizSaveDataV2 updated to V3 format (global tracking)
   - speciesCleared[20] as bit array (160 species)
   - masteryCount[152] per species
   - Removed section-aware accessors, added global equivalents

4. **15-04**: Quiz logic reversion — COMPLETE
   - Wild encounters use Quiz_GetBank(species) directly
   - Single-question trainers use species bank with pool fallback
   - Quiz_GetMasteryProgress uses global mastery count
   - Quiz_OnMoveConfirmed updates global mastery
   - Quiz_GetEncounterType fixed for multi-question trainer progress display

### Bug Fixes Applied

| Bug | Issue | Fix |
|-----|-------|-----|
| Mastery not incrementing | currentQuestionIndex == 0xFF skipped update | Always increment count using currentSection |
| Progress stuck at 0/5 | Old bitmask counting logic | Return count directly, show X/5 |
| Capture not triggering | CAPTURE_PENDING not persisted | Derive from masteryMask >= 5 |
| Map name typos | MAP_S_S_ANNE_1F, MAP_PEWTER_MUSEUM_1F | Fixed to match map_groups.h constants |
| Symbol duplication | Quiz_GetMasteryCount defined twice | Removed wrapper functions, use macros in quiz.h |
| Implicit declaration | Functions called before definition | Added forward declarations |
| Trainer progress display | Multi-question trainers showed mastery | Fixed Quiz_GetEncounterType to use isGymLeader flag |

### Pending Todos

**Todo Queue:** 0 items

**Remaining Phase 15 Work:**
- Complete encounter table redesign for Sections 3-8
- Assign all 727 questions to species in species_map.json
- Regenerate questions_gen.c with full species banks

### Blockers/Concerns

None - Core code changes complete, ready for content expansion

## Session Continuity

Last session: 2026-02-05
Stopped at: Phase 15 code changes complete (15-03, 15-04)
Resume file: None

### What's Working
- Build tools (CSV → C generation)
- Question bank integration
- Save persistence (global mastery tracking)
- Core loop (questions from species banks)
- Capture flow (5 correct → capture mode)
- Capture animation (Master Ball throw on quiz mastery)
- Encounter filter (cleared species skipped globally)
- UI polish (explanations, progress X/5)
- Route completion (25% rate when terrain species cleared)
- Per-species questions (from species bank)
- Multi-question trainer progress display (X/N for gym trainers, leaders, etc.)

### What's Next
- Complete encounter table redesign (Sections 3-8 in wild_encounters.json)
- Expand species_map.json with all 150+ species and question assignments
- Regenerate questions_gen.c with full species banks
- Test capture flow with new species

### Phase 15 Architecture

**Per-Species Questions with Unique Location Species**

Key architectural changes implemented:
- Each species appears in one primary location
- Questions selected from species-specific bank via Quiz_GetBank()
- Global mastery tracking (removed section awareness)
- Save data: speciesCleared[20] bits + masteryCount[152] per species

Code changes:
- global.h: QuizSaveDataV2 simplified to V3 format
- quiz.h: Global accessor declarations, removed section functions
- quiz.c: Global mastery functions, species bank selection, forward declarations
