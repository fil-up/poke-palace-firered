---
phase: ENH-C-route-completion
plan: 01
type: execute
wave: 1
depends_on: []
files_modified:
  - pokefirered/src/quiz/route_completion.c
  - pokefirered/include/quiz/route_completion.h
  - pokefirered/src/wild_encounter.c
  - pokefirered/src/quiz/quiz.c
autonomous: true

must_haves:
  truths:
    - "Encounter rate drops to 25% when all species in terrain type are CLEARED"
    - "Completion check performs efficiently with caching"
    - "Each terrain type (grass, surf, fishing rods) checked independently"
  artifacts:
    - path: "pokefirered/src/quiz/route_completion.c"
      provides: "Completion checking logic with caching"
      exports: ["IsLandTerrainCompleted", "IsWaterTerrainCompleted", "IsFishingRodCompleted", "GetTerrainProgress", "InvalidateCompletionCache"]
    - path: "pokefirered/include/quiz/route_completion.h"
      provides: "Public API for route completion"
      contains: "TERRAIN_TYPE_"
    - path: "pokefirered/src/wild_encounter.c"
      provides: "Rate modifier integration"
      contains: "ApplyRouteCompletionEncounterRateMod"
  key_links:
    - from: "pokefirered/src/wild_encounter.c"
      to: "route_completion.c"
      via: "ApplyRouteCompletionEncounterRateMod calling IsXXXTerrainCompleted"
      pattern: "IsLandTerrainCompleted|IsWaterTerrainCompleted|IsFishingRodCompleted"
    - from: "pokefirered/src/quiz/route_completion.c"
      to: "quiz.c"
      via: "Quiz_IsSpeciesCleared calls"
      pattern: "Quiz_IsSpeciesCleared"
    - from: "pokefirered/src/quiz/quiz.c"
      to: "route_completion.c"
      via: "Cache invalidation on species state change"
      pattern: "InvalidateCompletionCache"
---

<objective>
Implement route completion detection and 25% encounter rate reduction.

Purpose: When all species in a terrain type reach CLEARED state, encounters become less frequent (25% of normal rate), rewarding player mastery while still allowing occasional encounters for grinding.

Output: New route_completion module with completion checking functions, caching for performance, and integration with wild_encounter.c rate calculation.
</objective>

<execution_context>
@C:\Users\phill\.claude/get-shit-done/workflows/execute-plan.md
@C:\Users\phill\.claude/get-shit-done/templates/summary.md
</execution_context>

<context>
@.planning/PROJECT.md
@.planning/STATE.md
@.planning/phases/ENH-C-route-completion/ENH-C-RESEARCH.md
@.planning/phases/ENH-C-route-completion/ENH-C-CONTEXT.md
@pokefirered/include/quiz/quiz.h
@pokefirered/include/wild_encounter.h
@pokefirered/src/wild_encounter.c
</context>

<tasks>

<task type="auto">
  <name>Task 1: Create route completion module</name>
  <files>
    pokefirered/src/quiz/route_completion.c
    pokefirered/include/quiz/route_completion.h
  </files>
  <action>
Create the route completion checking module with:

**Header file (route_completion.h):**
- Terrain type enum: TERRAIN_LAND, TERRAIN_WATER, TERRAIN_OLD_ROD, TERRAIN_GOOD_ROD, TERRAIN_SUPER_ROD
- Function declarations for completion checking and progress retrieval
- Cache invalidation function declaration

**Source file (route_completion.c):**
- Include wild_encounter.h for WildPokemonHeader, gWildMonHeaders, slot counts
- Include quiz/quiz.h for Quiz_IsSpeciesCleared

**Completion checking functions:**
1. `bool8 IsLandTerrainCompleted(u16 headerId)`:
   - Return FALSE if headerId is HEADER_NONE (0xFFFF) or landMonsInfo is NULL
   - Iterate LAND_WILD_COUNT (12) slots, collect unique species
   - For each unique species, call Quiz_IsSpeciesCleared
   - Return TRUE only if ALL unique species are cleared

2. `bool8 IsWaterTerrainCompleted(u16 headerId)`:
   - Same pattern as land but use waterMonsInfo and WATER_WILD_COUNT (5)

3. `bool8 IsFishingRodCompleted(u16 headerId, u8 rod)`:
   - Use fishingMonsInfo with rod-specific slot ranges:
     - OLD_ROD: slots 0-1
     - GOOD_ROD: slots 2-4  
     - SUPER_ROD: slots 5-9
   - Same deduplication and checking pattern

**Progress retrieval:**
4. `void GetTerrainProgress(u16 headerId, u8 terrainType, u8 *cleared, u8 *total)`:
   - Count unique species (total) and how many are cleared
   - Handles all 5 terrain types via switch

**Caching system:**
- Static cache array: `sCompletionCache[MAX_CACHED_HEADERS]` with headerId, per-terrain completion flags, valid flag
- Wrap completion checks to consult cache first
- `void InvalidateCompletionCache(void)` - called when species state changes

Use small local arrays (max 12 elements) for species deduplication with O(n²) linear search - acceptable for small datasets per research.
  </action>
  <verify>
File compiles: `cd pokefirered && make -j$(nproc) 2>&1 | grep -E "(error|route_completion)"` shows no errors.
  </verify>
  <done>
route_completion.c and route_completion.h exist with all 5 completion check functions, progress retrieval, and caching system. Module compiles successfully.
  </done>
</task>

<task type="auto">
  <name>Task 2: Integrate rate modifier into wild encounter system</name>
  <files>
    pokefirered/src/wild_encounter.c
  </files>
  <action>
Add 25% rate reduction when terrain is completed:

**1. Add include:**
```c
#include "quiz/route_completion.h"
```

**2. Create rate modifier function:**
```c
static void ApplyRouteCompletionEncounterRateMod(u32 *encounterRate, u8 terrainType, u16 headerId)
{
    bool8 isCompleted = FALSE;
    
    switch (terrainType)
    {
    case TERRAIN_LAND:
        isCompleted = IsLandTerrainCompleted(headerId);
        break;
    case TERRAIN_WATER:
        isCompleted = IsWaterTerrainCompleted(headerId);
        break;
    // Fishing handled separately in FishingWildEncounter
    }
    
    if (isCompleted)
        *encounterRate = *encounterRate / 4;  // 25% of normal
}
```

**3. Modify DoWildEncounterRateTest:**
Add terrain type parameter and apply modifier AFTER Cleanse Tag but BEFORE ability modifiers (per research order).

Current order in DoWildEncounterRateTest:
1. Base rate × 16
2. Bike modifier
3. Encounter rate buff
4. Flute modifier
5. Cleanse Tag modifier
6. **NEW: Completion modifier** ← Insert here
7. Ability modifier

Need to also pass headerId and terrainType to the function. Update signature and all call sites (StandardWildEncounter land/water, RockSmashWildEncounter, SweetScentWildEncounter).

**4. Modify FishingWildEncounter:**
For fishing, apply the rate reduction before the encounter check. Check IsFishingRodCompleted for the specific rod type. Since fishing doesn't use DoWildEncounterRateTest the same way, add a separate check:
```c
if (IsFishingRodCompleted(headerId, rod))
{
    // 25% chance to actually get encounter
    if (Random() % 4 != 0)
        return;  // Skip encounter 75% of time
}
```
  </action>
  <verify>
ROM builds successfully: `cd pokefirered && make -j$(nproc)` completes without errors. Test in emulator: walk in grass on Route 1, verify encounters still happen.
  </verify>
  <done>
ApplyRouteCompletionEncounterRateMod integrated into DoWildEncounterRateTest. All encounter types (land, water, fishing) check completion status and apply 25% rate reduction when terrain is cleared.
  </done>
</task>

<task type="auto">
  <name>Task 3: Add cache invalidation hook</name>
  <files>
    pokefirered/src/quiz/quiz.c
  </files>
  <action>
Hook cache invalidation into species state changes:

**1. Add include in quiz.c:**
```c
#include "quiz/route_completion.h"
```

**2. Modify Quiz_SetSpeciesState:**
After updating species state to CLEARED, call InvalidateCompletionCache():

```c
void Quiz_SetSpeciesState(u16 species, u8 state)
{
    // ... existing state update code ...
    
    // Invalidate completion cache when species becomes cleared
    // (completion status may have changed for any terrain containing this species)
    if (state == QUIZ_STATE_CLEARED)
        InvalidateCompletionCache();
}
```

This ensures that when a species is captured and marked CLEARED, the completion cache is invalidated so the next completion check recalculates. The cache will be rebuilt lazily on next encounter check.

**Note:** Only invalidate on CLEARED state, not on other state changes (WILD, CAPTURE_PENDING), since those don't affect terrain completion.
  </action>
  <verify>
ROM builds successfully. In emulator: capture a Pokemon, verify no crashes when walking in grass afterward (cache invalidation working).
  </verify>
  <done>
Quiz_SetSpeciesState calls InvalidateCompletionCache when species reaches CLEARED state. Cache is properly invalidated on completion events.
  </done>
</task>

</tasks>

<verification>
1. **Build verification:** `cd pokefirered && make clean && make -j$(nproc)` succeeds
2. **New files exist:** `ls pokefirered/src/quiz/route_completion.c pokefirered/include/quiz/route_completion.h`
3. **Functions exported:** `grep -E "IsLandTerrainCompleted|ApplyRouteCompletionEncounterRateMod" pokefirered/src/wild_encounter.c`
4. **Cache hook present:** `grep "InvalidateCompletionCache" pokefirered/src/quiz/quiz.c`
</verification>

<success_criteria>
- route_completion.c/h created with completion checking for all 5 terrain types
- Caching system prevents per-step overhead
- wild_encounter.c applies 25% rate modifier for completed terrains
- Cache invalidation hooked into Quiz_SetSpeciesState
- ROM builds and runs without crashes
</success_criteria>

<output>
After completion, create `.planning/phases/ENH-C-route-completion/ENH-C-01-SUMMARY.md`
</output>
