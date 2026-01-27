# Phase 8 Plan 01: Route 1 Content Summary

## One-Liner
Populated question banks for Rattata and Pidgey with 5 GH 101 questions each, enabling a fully playable Route 1 experience where players can master and capture both species.

## Tasks Completed

| Task | Description | Status |
|------|-------------|--------|
| 1 | Update species_map.json with Rattata questions (5) | ✓ Complete |
| 2 | Update species_map.json with Pidgey questions (5) | ✓ Complete |
| 3 | Regenerate questions_gen files | ✓ Complete |
| 4 | Verify compilation | ✓ Complete |

## Files Modified

- `pokefirered/data/questions/species_map.json` - Added question IDs for Route 1 species
- `pokefirered/src/quiz/questions_gen.c` - Regenerated with populated banks
- `pokefirered/include/quiz/questions_gen.h` - Regenerated

## Key Changes

### species_map.json Updates

```json
"SPECIES_RATTATA": {
  "maps": ["MAP_ROUTE1"],
  "difficulty_range": [1, 3],
  "questions": ["CH06-001", "CH06-002", "CH06-003", "CH06-004", "CH06-005"]
},
"SPECIES_PIDGEY": {
  "maps": ["MAP_ROUTE1", "MAP_ROUTE2"],
  "difficulty_range": [1, 3],
  "questions": ["CH06-006", "CH06-007", "CH06-008", "CH06-009", "CH06-010"]
}
```

### Generated Question Banks

**Rattata (5 questions):**
- CH06-001: Class III dental services
- CH06-002: Dental PPO reimbursement
- CH06-003: Antiselection mitigation (incentive coinsurance)
- CH06-004: ACA pediatric dental requirements
- CH06-005: DHMO reimbursement (capitation)

**Pidgey (5 questions):**
- CH06-006: LEAT provision
- CH06-007: Dental underwriting/rating
- CH06-008: Scheduled indemnity balance billing
- CH06-009: Orthodontic benefits (lifetime maximum)
- CH06-010: Dental moral hazard

## Question Bank Structure

```c
static const struct QuizQuestion * const sQuestions_SPECIES_RATTATA[] = {
    &sQuestion_CH06_001,
    &sQuestion_CH06_002,
    &sQuestion_CH06_003,
    &sQuestion_CH06_004,
    &sQuestion_CH06_005,
};

const struct QuizBank gQuizBank_SPECIES_RATTATA = {
    .questions = sQuestions_SPECIES_RATTATA,
    .count = 5,
};
```

## Build Status
- ROM compiles successfully
- ROM size: 15,474,953 bytes (46.12% of 32MB)
- EWRAM: 261,200 bytes (99.64%)

## Deviations from Plan
- Initially used wrong question ID format (underscores vs hyphens)
- Corrected to match CSV format: `CH06-001` instead of `CH06_001`

## Requirements Satisfied

- [x] CONT-01: Rattata bank has 5 GH 101 questions
- [x] CONT-01: Pidgey bank has 5 GH 101 questions
- [x] Player can master both species and capture them

## Gameplay Flow (Route 1)

```
Encounter Rattata:
  └─► 5 questions to master (CH06-001 through CH06-005)
      └─► After all mastered → CAPTURE_PENDING
          └─► Answer all 5 correctly in sequence → Capture!
              └─► Rattata added to party, species CLEARED

Encounter Pidgey:
  └─► 5 questions to master (CH06-006 through CH06-010)
      └─► Same capture flow as Rattata
```

## Next Phase Readiness

- [x] Route 1 is fully playable with quiz mechanics
- [x] Both species have question content
- [ ] Ready for Phase 9: Viridian Forest Content
