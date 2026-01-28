# Roadmap: Poke Palace FireRed

## Overview

Build a quiz-driven Pokémon FireRed ROM hack from current state (hardcoded single question) to complete MVP (Route 1 through Pewter Gym with full quiz loop, save persistence, and capture system).

## Phases

- [x] **Phase 1: Build Tools** — CSV validation and C table generation
- [x] **Phase 2: Question Bank System** — Data structures and lookup
- [x] **Phase 3: Save Persistence** — Mastery tracking in SaveBlock
- [x] **Phase 4: Core Loop Enhancement** — Question selection from bank
- [x] **Phase 5: Capture Flow** — State machine and capture quiz
- [x] **Phase 6: Encounter Filter** — Cleared species reroll
- [x] **Phase 7: UI Polish** — Multi-page text and explanations
- [x] **Phase 8: Route 1 Content** — Populate question banks
- [x] **Phase 9: Viridian Forest Content** — Populate question banks
- [x] **Phase 10: Route 2 & Pewter Content** — Complete MVP area coverage
- [ ] **Phase ENH-C: Route Completion** — 25% encounter rate on terrain completion
- [ ] **Phase ENH-D: Type-Based Moves** — 2 moves per type, no effects
- [ ] **Phase 11: Gym Exam (Stretch)** — Topic mastery gate
- [ ] **Phase 12: Trainer Reinforcement (Stretch)** — Spaced repetition

## Phase Details

### Phase 1: Build Tools
**Goal**: Automated pipeline from CSV to compiled C tables
**Depends on**: Nothing (first phase)
**Requirements**: BUILD-01, BUILD-02, BUILD-03
**Success Criteria**:
  1. `python tools/quiz/validate_questions.py` catches invalid CSV entries
  2. `python tools/quiz/build_questions.py` generates questions_gen.h and questions_gen.c
  3. `make questions` runs both tools in sequence
  4. Generated files compile without errors
**Plans**:
  - 01-01-PLAN.md: Python validation + generation scripts
  - 01-02-PLAN.md: Makefile integration + species mapping

---

### Phase 2: Question Bank System
**Goal**: Runtime question lookup by species and location
**Depends on**: Phase 1
**Requirements**: BANK-01, BANK-02, BANK-03
**Success Criteria**:
  1. `Quiz_GetBank(species)` returns valid bank for mapped species
  2. Bank contains questions with all fields populated
  3. Hardcoded question replaced with bank lookup
**Plans**:
  - 02-01-PLAN.md: Integrate question bank into quiz.c

---

### Phase 3: Save Persistence
**Goal**: Quiz progress survives save/load
**Depends on**: Phase 2
**Requirements**: SAVE-01, SAVE-02, SAVE-03
**Success Criteria**:
  1. Answer question correctly, save, reload → question still marked mastered
  2. Mastery state persists across game sessions
  3. New save format doesn't corrupt existing vanilla saves
**Plans**:
  - 03-01-PLAN.md: Add QuizSaveData to SaveBlock1, accessor functions

---

### Phase 4: Core Loop Enhancement
**Goal**: Dynamic question selection based on mastery
**Depends on**: Phase 3
**Requirements**: LOOP-01, LOOP-02, LOOP-03
**Success Criteria**:
  1. Previously answered questions don't repeat
  2. Correct answer updates mastery mask
  3. All questions mastered → state transitions to CAPTURE_PENDING
**Plans**:
  - 04-01-PLAN.md: Mastery filtering + state transitions

---

### Phase 5: Capture Flow
**Goal**: Complete capture quiz state machine
**Depends on**: Phase 4
**Requirements**: CAPT-01, CAPT-02, CAPT-03
**Success Criteria**:
  1. CAPTURE_PENDING species triggers capture quiz (all questions in sequence)
  2. Perfect run adds Pokémon to party, marks CLEARED
  3. Any wrong answer → flee, state stays CAPTURE_PENDING
**Plans**: (created by /gsd:plan-phase)

---

### Phase 6: Encounter Filter
**Goal**: Cleared species don't appear in wild encounters
**Depends on**: Phase 5
**Requirements**: FILT-01, FILT-02, FILT-03
**Success Criteria**:
  1. Cleared Rattata no longer appears on Route 1
  2. Reroll happens automatically without player noticing
  3. If all species cleared, encounter gracefully skipped
**Plans**: (created by /gsd:plan-phase)

---

### Phase 7: UI Polish
**Goal**: Professional text display and learning feedback
**Depends on**: Phase 4
**Requirements**: UI-01, UI-02
**Success Criteria**:
  1. Long questions paginate with "▼" indicator and button wait
  2. Correct answer shows explanation before battle ends
  3. Text doesn't overlap or clip
**Plans**: (created by /gsd:plan-phase)

---

### Phase 8: Route 1 Content
**Goal**: Fully playable Route 1 experience
**Depends on**: Phase 6, Phase 7
**Requirements**: CONT-01
**Success Criteria**:
  1. Rattata bank has 5 GH 101 questions
  2. Pidgey bank has 5 GH 101 questions
  3. Player can master both species and capture them
**Plans**: (created by /gsd:plan-phase)

---

### Phase 9: Viridian Forest Content
**Goal**: Fully playable Viridian Forest experience
**Depends on**: Phase 8
**Requirements**: CONT-02
**Success Criteria**:
  1. Caterpie, Weedle, Metapod, Kakuna, Pikachu banks populated
  2. Questions mapped appropriately by difficulty
  3. Forest species can be mastered and captured
**Plans**: (created by /gsd:plan-phase)

---

### Phase 10: Route 2 & Pewter Content
**Goal**: Complete MVP geographic scope
**Depends on**: Phase 9
**Requirements**: CONT-03
**Success Criteria**:
  1. Route 2 species banks populated
  2. Any Pewter City area species banks populated
  3. Player can progress from Pallet Town to Pewter Gym with quiz mechanics
**Plans**: (created by /gsd:plan-phase)

---

### Phase ENH-C: Route Completion
**Goal**: Reduce encounter rate to 25% when all species in terrain type are CLEARED
**Depends on**: Phase 10
**Requirements**: ENH-C
**Success Criteria**:
  1. Encounter rate drops to 25% when all terrain species CLEARED
  2. Each terrain type (grass, surf, fishing rods) tracked independently
  3. Status banner on map entry shows progress per terrain type
  4. Completion message when terrain reaches 100%
**Plans**: 2 plans
  - [ ] ENH-C-01-PLAN.md — Completion logic + rate reduction
  - [ ] ENH-C-02-PLAN.md — Status banner + completion message

---

### Phase ENH-D: Type-Based Moves
**Goal**: Give Pokemon moves based on type instead of level-up movesets
**Depends on**: Phase ENH-C
**Requirements**: ENH-D
**Success Criteria**:
  1. Each type has 6 moves (2 per tier: Weak/Medium/Strong)
  2. Dual-type Pokemon get 2+2 moves from their types
  3. Single-type Pokemon get 2 type moves + 2 random filler moves
  4. Evolution stage determines move tier (Basic=Weak, Stage1=Medium, Stage2/Legend=Strong)
  5. No secondary effects (flat 1000 damage on all moves)
**Plans**: 1 plan
  - [ ] ENH-D-01-PLAN.md — Type-based move assignment module

---

### Phase 11: Gym Exam (Stretch)
**Goal**: Topic mastery gate before gym leader
**Depends on**: Phase ENH-D
**Requirements**: V2-01
**Success Criteria**:
  1. Entering Pewter Gym triggers section exam (X questions)
  2. Pass threshold allows gym battle
  3. Fail returns player outside gym
**Plans**: (created by /gsd:plan-phase)

---

### Phase 12: Trainer Reinforcement (Stretch)
**Goal**: Spaced repetition via trainer battles
**Depends on**: Phase 11
**Requirements**: V2-02
**Success Criteria**:
  1. Trainer Pokémon send-out has 15% chance to trigger review question
  2. Review pulls from already-mastered pool
  3. Wrong answer applies damage penalty
**Plans**: (created by /gsd:plan-phase)

---

## Progress

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 1. Build Tools | 2/2 | ✓ Complete | 2026-01-24 |
| 2. Question Bank | 1/1 | ✓ Complete | 2026-01-24 |
| 3. Save Persistence | 1/1 | ✓ Complete | 2026-01-24 |
| 4. Core Loop | 1/1 | ✓ Complete | 2026-01-24 |
| 5. Capture Flow | 1/1 | ✓ Complete | 2026-01-24 |
| 6. Encounter Filter | 1/1 | ✓ Complete | 2026-01-24 |
| 7. UI Polish | 1/1 | ✓ Complete | 2026-01-24 |
| 8. Route 1 Content | 1/1 | ✓ Complete | 2026-01-24 |
| 9. Viridian Forest | 1/1 | ✓ Complete | 2026-01-26 |
| 10. Pewter Content | 1/1 | ✓ Complete | 2026-01-26 |
| ENH-C. Route Completion | 0/2 | In Progress | - |
| ENH-D. Type-Based Moves | 0/? | Pending | - |
| 11. Gym Exam | 0/? | Stretch | - |
| 12. Trainer Reinforcement | 0/? | Stretch | - |
