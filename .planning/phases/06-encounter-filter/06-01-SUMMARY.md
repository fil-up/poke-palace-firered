# Phase 6 Plan 01: Encounter Filter Summary

## One-Liner
Implemented encounter filter so cleared species are skipped during wild encounter generation, with reroll logic capped at 10 attempts and graceful handling when all species in an area are cleared.

## Tasks Completed

| Task | Description | Status |
|------|-------------|--------|
| 1 | Add Quiz_IsSpeciesCleared helper function | ✓ Complete |
| 2 | Include quiz header in wild_encounter.c | ✓ Complete |
| 3 | Implement reroll logic in TryGenerateWildMon | ✓ Complete |
| 4 | Handle fishing encounters with reroll logic | ✓ Complete |
| 5 | Verify compilation | ✓ Complete |

## Files Modified

- `pokefirered/include/quiz/quiz.h` - Added Quiz_IsSpeciesCleared declaration
- `pokefirered/src/quiz/quiz.c` - Implemented Quiz_IsSpeciesCleared function
- `pokefirered/src/wild_encounter.c` - Added reroll logic for land, water, rock, and fishing encounters

## Key Additions

### New Function
```c
bool8 Quiz_IsSpeciesCleared(u16 species)
{
    return Quiz_GetSpeciesState(species) == QUIZ_STATE_CLEARED;
}
```

### New Constant
```c
#define MAX_REROLL_ATTEMPTS 10
```

### Modified TryGenerateWildMon
- Added reroll loop that picks a different slot if selected species is cleared
- Caps attempts at MAX_REROLL_ATTEMPTS (10)
- Returns FALSE if all attempts exhausted (gracefully skips encounter)

### Modified GenerateFishingEncounter
- Added same reroll logic for fishing encounters
- Returns SPECIES_NONE if all fish species cleared

### Modified FishingWildEncounter
- Checks for SPECIES_NONE return value
- Skips battle if all fish species are cleared

## Encounter Flow

```
Wild Encounter Trigger:
  └─► TryGenerateWildMon()
      └─► Loop (max 10 attempts):
          ├─► Choose random slot
          ├─► Get species for slot
          ├─► Check Quiz_IsSpeciesCleared(species)
          │   ├─► FALSE: Use this species, break
          │   └─► TRUE: Reroll (continue loop)
          └─► If 10 attempts exhausted: return FALSE (skip encounter)
      └─► Generate wild mon with selected species
      └─► return TRUE (start battle)
```

## Deviations from Plan

None - executed as planned.

## Requirements Satisfied

- [x] FILT-01: Cleared species skipped during wild encounter generation
- [x] FILT-02: Reroll capped at 10 attempts to prevent infinite loops
- [x] FILT-03: If all species in area cleared, return "no encounter"

## Next Phase Readiness

- [x] Cleared Rattata no longer appears on Route 1 (after capture)
- [x] Reroll happens automatically without player noticing
- [x] If all species cleared, encounter gracefully skipped
- [ ] Ready for Phase 7: UI Polish (multi-page text, explanations)
