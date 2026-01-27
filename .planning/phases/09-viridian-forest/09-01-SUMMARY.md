# Phase 9 Plan 01: Viridian Forest Content Summary

## One-Liner
Populated question banks for all Viridian Forest species (Caterpie, Weedle, Metapod, Kakuna, Pikachu) with 5 questions each using MQ.3 chapter content, enabling a fully playable forest experience.

## Tasks Completed

| Task | Description | Status |
|------|-------------|--------|
| 1 | Update species_map.json with Caterpie questions (5) | ✓ Complete |
| 2 | Update species_map.json with Weedle questions (5) | ✓ Complete |
| 3 | Update species_map.json with Metapod questions (5) | ✓ Complete |
| 4 | Update species_map.json with Kakuna questions (5) | ✓ Complete |
| 5 | Update species_map.json with Pikachu questions (5) | ✓ Complete |
| 6 | Regenerate questions_gen files | ✓ Complete |
| 7 | Verify compilation | ✓ Complete |

## Files Modified

- `pokefirered/data/questions/species_map.json` - Added question IDs for forest species
- `pokefirered/src/quiz/questions_gen.c` - Regenerated with populated banks
- `pokefirered/include/quiz/questions_gen.h` - Regenerated

## Key Changes

### species_map.json Updates

```json
"SPECIES_CATERPIE": {
  "maps": ["MAP_VIRIDIAN_FOREST"],
  "difficulty_range": [1, 2],
  "questions": ["MQ.3.001", "MQ.3.002", "MQ.3.006", "MQ.3.010", "MQ.3.012"]
},
"SPECIES_WEEDLE": {
  "maps": ["MAP_VIRIDIAN_FOREST"],
  "difficulty_range": [1, 2],
  "questions": ["MQ.3.001", "MQ.3.002", "MQ.3.006", "MQ.3.010", "MQ.3.015"]
},
"SPECIES_METAPOD": {
  "maps": ["MAP_VIRIDIAN_FOREST"],
  "difficulty_range": [2, 3],
  "questions": ["MQ.3.002", "MQ.3.003", "MQ.3.006", "MQ.3.007", "MQ.3.014"]
},
"SPECIES_KAKUNA": {
  "maps": ["MAP_VIRIDIAN_FOREST"],
  "difficulty_range": [2, 3],
  "questions": ["MQ.3.002", "MQ.3.003", "MQ.3.007", "MQ.3.010", "MQ.3.016"]
},
"SPECIES_PIKACHU": {
  "maps": ["MAP_VIRIDIAN_FOREST"],
  "difficulty_range": [3, 5],
  "questions": ["MQ.3.003", "MQ.3.004", "MQ.3.005", "MQ.3.008", "MQ.3.009"]
}
```

### Difficulty Strategy

| Species | Rarity | Difficulty Range | Rationale |
|---------|--------|------------------|-----------|
| Caterpie | Common | 1-2 | Easiest questions for high-frequency encounters |
| Weedle | Common | 1-2 | Easiest questions for high-frequency encounters |
| Metapod | Uncommon | 2-3 | Medium difficulty for evolution species |
| Kakuna | Uncommon | 2-3 | Medium difficulty for evolution species |
| Pikachu | Rare | 3-5 | Hardest questions as reward for rare encounter |

### Question Source
All questions from MQ.3 chapter (Chapter 3 content), with some overlap allowed between common species since questions are tracked per-species in mastery system.

## Build Status
- ROM compiles successfully
- All forest species have populated question banks

## Requirements Satisfied

- [x] CONT-02: Caterpie, Weedle, Metapod, Kakuna, Pikachu banks populated
- [x] CONT-02: Questions mapped appropriately by difficulty
- [x] Forest species can be mastered and captured

## Gameplay Flow (Viridian Forest)

```
Common encounters (Caterpie/Weedle):
  └─► 5 easier questions to master
      └─► After all mastered → CAPTURE_PENDING
          └─► Answer all 5 correctly → Capture!

Evolution encounters (Metapod/Kakuna):
  └─► 5 medium questions to master
      └─► Same capture flow

Rare encounter (Pikachu):
  └─► 5 harder questions to master
      └─► Same capture flow (rewarding for persistence)
```

## Next Phase Readiness

- [x] Viridian Forest is fully playable with quiz mechanics
- [x] All 5 forest species have question content
- [ ] Ready for Phase 10: Route 2 & Pewter Content
