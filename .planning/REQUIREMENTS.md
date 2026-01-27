# Requirements

## v1 Requirements

### Build Tools
- [ ] **BUILD-01**: Python script validates CSV question format (prompt ≤120 chars, answer ≤40 chars, required fields present)
- [ ] **BUILD-02**: Python script generates C tables from validated CSV (questions_gen.h/.c)
- [ ] **BUILD-03**: Makefile target `make questions` regenerates tables on CSV change

### Question Bank
- [ ] **BANK-01**: Question struct contains prompt, 4 choices, correct index, difficulty, explanation
- [ ] **BANK-02**: Bank lookup by (species, mapGroup, mapNum) returns question array
- [ ] **BANK-03**: 5-10 questions per species (rarer Pokémon = more questions)

### Save Persistence
- [ ] **SAVE-01**: Mastery bitmask (u8) persists per (scope, species) pair
- [ ] **SAVE-02**: Quiz progress survives save/load cycle without corruption
- [ ] **SAVE-03**: Save version field enables future-safe migrations

### Core Loop
- [ ] **LOOP-01**: Question selected from unmastered pool (filter by masteryMask)
- [ ] **LOOP-02**: Correct answer sets corresponding bit in masteryMask
- [ ] **LOOP-03**: All questions mastered (mask full) → CAPTURE_PENDING state

### Capture Flow
- [ ] **CAPT-01**: CAPTURE_PENDING state triggers capture quiz on next encounter
- [ ] **CAPT-02**: Perfect capture quiz = Pokémon added to party/PC, state → CLEARED
- [ ] **CAPT-03**: Failed capture quiz = Pokémon flees, state remains CAPTURE_PENDING

### Encounter Filter
- [ ] **FILT-01**: Cleared species skipped during wild encounter generation
- [ ] **FILT-02**: Reroll capped at 10 attempts to prevent infinite loops
- [ ] **FILT-03**: If all species in area cleared, return "no encounter"

### UI Enhancements
- [ ] **UI-01**: Multi-page text boxes auto-paginate long questions (wait for button)
- [ ] **UI-02**: Show explanation text when player answers correctly (foe faints)

### Content (MVP Scope)
- [ ] **CONT-01**: Route 1 question banks populated (Rattata, Pidgey — 5 questions each)
- [ ] **CONT-02**: Viridian Forest question banks populated (Caterpie, Weedle, Pikachu, etc.)
- [ ] **CONT-03**: Route 2 and Pewter City area question banks populated

---

## v2 Requirements (Deferred)

- [ ] **V2-01**: Gym section exam mode (topic mastery gate before gym leader)
- [ ] **V2-02**: Trainer reinforcement (spaced repetition on trainer Pokémon)
- [ ] **V2-03**: Difficulty-based question selection within bank
- [ ] **V2-04**: Statistics tracking (questions answered, accuracy rate)
- [ ] **V2-05**: Multiple subject CSV support (swap question banks)

---

## Out of Scope

- Traditional battle mechanics — replaced by quiz system
- Catching with Pokéballs — replaced by capture quiz
- Online/link features — ROM hack, offline only
- Difficulty modes — fixed question counts for MVP
- Custom sprites/graphics — use existing FireRed assets

---

## Traceability

| Requirement | Phase |
|-------------|-------|
| BUILD-01, BUILD-02, BUILD-03 | Phase 1: Build Tools |
| BANK-01, BANK-02, BANK-03 | Phase 2: Question Bank |
| SAVE-01, SAVE-02, SAVE-03 | Phase 3: Save Persistence |
| LOOP-01, LOOP-02, LOOP-03 | Phase 4: Core Loop |
| CAPT-01, CAPT-02, CAPT-03 | Phase 5: Capture Flow |
| FILT-01, FILT-02, FILT-03 | Phase 6: Encounter Filter |
| UI-01, UI-02 | Phase 7: UI Polish |
| CONT-01 | Phase 8: Route 1 Content |
| CONT-02 | Phase 9: Viridian Forest Content |
| CONT-03 | Phase 10: Route 2 / Pewter Content |
| V2-01 | Stretch: Gym Exam |
| V2-02 | Stretch: Trainer Reinforcement |
