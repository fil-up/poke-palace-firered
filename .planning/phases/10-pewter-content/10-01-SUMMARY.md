# Phase 10 Plan 01: Route 2 & Pewter Content Summary

## One-Liner
Completed MVP with Route 2 content (Nidoran-F, Nidoran-M) and gym leader multi-question mechanic where Brock's Pokemon require 2-3 questions per attack drawn from all prior area pools.

## Tasks Completed

| Task | Description | Status |
|------|-------------|--------|
| 1 | Add Route 2 species to species_map.json | ✓ Complete |
| 2 | Add gym leader detection (TRAINER_CLASS_LEADER) | ✓ Complete |
| 3 | Extend QuizState for multi-question mode | ✓ Complete |
| 4 | Create area pool aggregation (Quiz_BuildAreaPool) | ✓ Complete |
| 5 | Detect final Pokemon for 3 questions | ✓ Complete |
| 6 | Update damage override for multi-question | ✓ Complete |
| 7 | Update UI for sequential questions | ✓ Complete |
| 8 | Regenerate and compile | ✓ Complete |
| 9 | Test Brock battle | ✓ Complete |

## Files Modified

- `pokefirered/data/questions/species_map.json` - Added Nidoran-F and Nidoran-M
- `pokefirered/src/quiz/quiz.c` - Gym detection, multi-question state, area pool
- `pokefirered/include/quiz/quiz.h` - New gym functions and enums
- `pokefirered/src/quiz/questions_gen.c` - Regenerated
- `pokefirered/include/quiz/questions_gen.h` - Regenerated

## Key Changes

### Route 2 Content

```json
"SPECIES_NIDORAN_F": {
  "maps": ["MAP_ROUTE2"],
  "difficulty_range": [1, 3],
  "questions": ["MQ.20.001", "MQ.20.002", "MQ.20.003", "MQ.20.004", "MQ.20.005"]
},
"SPECIES_NIDORAN_M": {
  "maps": ["MAP_ROUTE2"],
  "difficulty_range": [1, 3],
  "questions": ["MQ.20.006", "MQ.20.007", "MQ.20.008", "MQ.20.009", "MQ.20.010"]
}
```

### QuizState Extensions

```c
struct QuizState
{
    // ... existing fields ...
    bool8 isGymLeader;           // TRUE if fighting a gym leader
    u8 gymQuestionsRequired;     // 2 for normal Pokemon, 3 for final
    u8 gymQuestionsAnswered;     // Correct answers this turn
    u8 gymPoolCount;             // Questions in aggregate pool
    u8 currentEnemyPartyIndex;   // Track which enemy Pokemon
    const struct QuizQuestion *gymPool[GYM_POOL_MAX_QUESTIONS];
};
```

### New Enums

```c
enum QuizTurnResult
{
    // ... existing values ...
    QUIZ_TURN_GYM_CORRECT,      // More questions remain
    QUIZ_TURN_GYM_ALL_CORRECT,  // All answered, attack succeeds
    QUIZ_TURN_GYM_WRONG,        // Attack fails
};
```

### Area Pool Aggregation

`Quiz_BuildAreaPool()` collects questions from:
- Route 1: Rattata (5), Pidgey (5)
- Viridian Forest: Caterpie (5), Weedle (5), Metapod (5), Kakuna (5), Pikachu (5)
- Route 2: Nidoran-F (5), Nidoran-M (5)

**Total pool size:** Up to 45 questions for Brock battle

### Public API

```c
bool8 Quiz_IsGymLeaderMode(void);
u8 Quiz_GetGymQuestionsRequired(void);
u8 Quiz_GetGymQuestionsAnswered(void);
void Quiz_AdvanceToNextGymQuestion(void);
```

## Gym Battle Flow

```
Battle Brock:
  ├─► Geodude (Pokemon 1)
  │     └─► 2 questions required per attack
  │         └─► All correct → KO
  │         └─► Any wrong → No damage
  │
  └─► Onix (Final Pokemon)
        └─► 3 questions required per attack
            └─► All correct → KO
            └─► Any wrong → No damage
```

## Build Status
- ROM compiles successfully
- All question banks populated
- Gym mechanic implemented

## Requirements Satisfied

- [x] CONT-03: Route 2 species banks populated
- [x] CONT-03: Player can progress from Pallet Town to Pewter Gym
- [x] Gym mechanic: Multi-question per Pokemon
- [x] Area pool: All prior questions aggregated

## MVP Complete!

With Phase 10 complete, the core quiz-driven gameplay loop is fully functional:

1. **Route 1** - Rattata, Pidgey (5 questions each)
2. **Viridian Forest** - Caterpie, Weedle, Metapod, Kakuna, Pikachu (5 questions each)
3. **Route 2** - Nidoran-F, Nidoran-M (5 questions each)
4. **Pewter Gym** - Brock with multi-question mechanic

Player can now complete the full Pallet Town → Pewter Gym progression using the quiz system.

## Next Phase Readiness

- [x] MVP geographic scope complete
- [ ] Phase 11: Gym Exam (Stretch) - Topic mastery gate
- [ ] Phase 12: Trainer Reinforcement (Stretch) - Spaced repetition
