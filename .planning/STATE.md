# Project State

## Project Reference

See: .planning/PROJECT.md (updated 2026-01-24)

**Core value:** Quiz-driven gameplay loop must work end-to-end
**Current focus:** Phase 15 - Per-Species Questions with Unique Location Species

## Current Position

Phase: 15 of 15 (Per-Species Questions)
Plan: 4 of 4 complete
Status: Core implementation complete, refinement may be needed
Next: Test gameplay, verify species uniqueness works as intended
Last activity: 2026-02-05 — Completed Phase 15 (species_map.json + wild_encounters.json)

Progress: [████████████████████] 100% (all plans complete)

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

### Phase 15 Summary (Complete)

**Per-Species Questions with Unique Location Species**

1. **15-01**: Encounter table redesign — COMPLETE
   - Species-location mapping document created (15-SPECIES-MAP.md)
   - Updated Routes 5, 6, 7, 8, 9, 11 for Sections 3-4
   - Updated Safari Zone (Center, East, West) for Section 5
   - Updated Power Plant, Victory Road for Section 8
   - Vermilion City fishing updated

2. **15-02**: Question-to-species assignment — COMPLETE
   - species_map.json created with 148 species mapped
   - 730 questions distributed across all species (~5 per species)
   - questions_gen.c regenerated with full species banks

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

**Potential refinements:**
- Some routes may need further species redistribution
- LeafGreen versions of encounter tables not updated (use FireRed version)
- S.S. Anne Charmander not yet implemented (requires special handling)
- Safari Zone North not updated (needs Rhyhorn, Nidorina, Nidorino)
- Some water/fishing routes not yet updated

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
- Test gameplay with new species distribution
- Verify per-species questions work correctly in wild encounters
- Consider additional refinements (remaining routes, LeafGreen versions)
- Optional: Add S.S. Anne Charmander (requires special encounter logic)

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
