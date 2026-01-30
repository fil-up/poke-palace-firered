# ENH-C-01: Route Completion Detection and Encounter Rate Reduction

## Summary

Implemented route completion detection with 25% encounter rate reduction when all species in a terrain type reach CLEARED state. This rewards player mastery while still allowing occasional encounters for grinding.

## Tasks Completed

### Task 1: Create route completion module
**Files Created:**
- `pokefirered/include/quiz/route_completion.h`
- `pokefirered/src/quiz/route_completion.c`

**Implementation:**
- Created `TerrainType` enum with LAND, WATER, OLD_ROD, GOOD_ROD, SUPER_ROD, and COUNT values
- Implemented completion checking functions:
  - `IsLandTerrainCompleted(headerId)` - checks all grass/land species
  - `IsWaterTerrainCompleted(headerId)` - checks all surfing species
  - `IsFishingRodCompleted(headerId, rod)` - checks rod-specific species by slot ranges
- Implemented progress retrieval: `GetTerrainProgress(headerId, terrainType, *cleared, *total)`
- Added caching system with 8-entry cache to prevent per-step overhead:
  - `sCompletionCache[]` stores per-header completion status for all terrain types
  - `GetOrCreateCacheEntry()` manages cache lookup and allocation
  - `InvalidateCompletionCache()` clears all cached entries

**Key Design Decisions:**
- Species deduplication uses small local arrays (max 12) with O(n²) linear search (acceptable for small datasets)
- Cache populates all terrain types on first access for any terrain type
- Fishing slots mapped: OLD_ROD (0-1), GOOD_ROD (2-4), SUPER_ROD (5-9)

### Task 2: Integrate rate modifier into wild encounter system
**Files Modified:**
- `pokefirered/src/wild_encounter.c`

**Implementation:**
- Added `#include "quiz/route_completion.h"`
- Created `ApplyRouteCompletionEncounterRateMod(u32 *encounterRate, u8 terrainType, u16 headerId)`:
  - Checks completion status via terrain-specific functions
  - Applies `/4` modifier (25% of normal rate) when terrain is completed
- Modified `DoWildEncounterRateTest()` to accept `terrainType` and `headerId` parameters
- Rate modifier applied AFTER Cleanse Tag, BEFORE ability modifiers (per specification)
- Updated call sites:
  - Land encounters: pass `TERRAIN_TYPE_LAND`
  - Water encounters: pass `TERRAIN_TYPE_WATER`  
  - Rock Smash: pass `TERRAIN_TYPE_COUNT` (no reduction)

### Task 3: Add cache invalidation hook
**Files Modified:**
- `pokefirered/src/quiz/quiz.c`

**Implementation:**
- Added `#include "quiz/route_completion.h"`
- Modified `Quiz_SetSpeciesState()` to call `InvalidateCompletionCache()` when species state changes to `QUIZ_STATE_CLEARED`

## Must-Haves Verification

### Truths ✓
- [x] Encounter rate drops to 25% when all species in terrain type are CLEARED
- [x] Completion check performs efficiently with caching (8-entry cache, full invalidation on state change)
- [x] Each terrain type (grass, surf, fishing rods) checked independently

### Artifacts ✓
- [x] `pokefirered/src/quiz/route_completion.c` - Completion checking logic with caching
  - Exports: `IsLandTerrainCompleted`, `IsWaterTerrainCompleted`, `IsFishingRodCompleted`, `GetTerrainProgress`, `InvalidateCompletionCache`
- [x] `pokefirered/include/quiz/route_completion.h` - Public API for route completion
  - Contains: `TERRAIN_TYPE_` enum
- [x] `pokefirered/src/wild_encounter.c` - Rate modifier integration
  - Contains: `ApplyRouteCompletionEncounterRateMod`

### Key Links ✓
- [x] `wild_encounter.c` → `route_completion.c` via `ApplyRouteCompletionEncounterRateMod` calling `IsXXXTerrainCompleted`
- [x] `route_completion.c` → `quiz.c` via `Quiz_IsSpeciesCleared` calls
- [x] `quiz.c` → `route_completion.c` via `InvalidateCompletionCache` on species state change

## Notes

- Fishing encounter rate reduction is not applied at the encounter check level since fishing encounters are determined by the fishing minigame. The species reroll logic already handles cleared species.
- Rock Smash encounters pass `TERRAIN_TYPE_COUNT` which doesn't match any case, so no rate reduction is applied (consistent with plan not specifying rock smash terrain).
