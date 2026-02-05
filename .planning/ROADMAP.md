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
- [x] **Phase ENH-C: Route Completion** — 25% encounter rate on terrain completion
- [ ] **Phase ENH-D: Type-Based Moves** — 2 moves per type, no effects
- [ ] **Phase 12: Scalable Battle System (Stretch)** — Trainer question scaling
- [x] **Phase 13: Make all moves the same from a damage/accuracy perspective, but retain only the move animation for cosmetic purposes** — Uniform damage/accuracy with cosmetic-only animations
- [x] **Phase 14: Section-Based Topic Pools** — Map-based topics with re-capturable species per section
- [ ] **Phase 15: Per-Species Questions** — Unique species per location with species-specific question banks

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
  - [x] ENH-C-01-PLAN.md — Completion logic + rate reduction
  - [x] ENH-C-02-PLAN.md — Status banner + completion message

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

### Phase 12: Scalable Battle System (Stretch)
**Goal**: Create a scalable battle system
**Depends on**: Phase 10
**Requirements**: TBD
**Success Criteria**:
  1. Trainer-facing encounters ask a fixed number of questions per Pokémon
  2. The last Pokémon in a battle asks more questions than the others (non-regular trainers)
  3. Applies to trainer battles, gym trainers, and special encounters (e.g., Ghost Marowak)
  4. Does not apply to wild encounters
**Plans**: 2 plans
  - [ ] 12-01-PLAN.md — Trainer scaling helpers + evolution analysis
  - [ ] 12-02-PLAN.md — Integrate scaling + ghost encounter init

---

### Phase 13: Make all moves the same from a damage/accuracy perspective, but retain only the move animation for cosmetic purposes
**Goal:** Uniform damage/accuracy for all moves while preserving original move names and animations (cosmetic only), excluding Safari battles.
**Depends on:** Phase 12
**Plans:** 2 plans

Plans:
- [x] 13-01-PLAN.md — Uniform quiz rules + keep status moves selectable
- [x] 13-02-PLAN.md — Force opponent-target animations + single-hit visuals

**Details:**
- Use existing quiz hooks to normalize damage/accuracy/PP/type/crit behavior.
- Keep all moves selectable while neutralizing real effects.
- Force animations to target the opponent and clamp multi-hit visuals.

---

### Phase 14: Section-Based Topic Pools with Re-Capturable Species

**Goal:** Implement section-based question system where topics are tied to map locations, and species can be re-captured when encountered in new sections
**Depends on:** Phase 13
**Requirements:** TBD
**Success Criteria:**
  1. Questions selected by map's topic pool, not by species
  2. Game divided into 8 sections (gym-gated progression)
  3. Species mastery/cleared tracked per-section (Rattata cleared in Section 1 can be re-captured in Section 7)
  4. Gym battles test cumulative topics from their section
**Plans:** 5 plans in 3 waves

Plans:
- [x] 14-01-PLAN.md — Section infrastructure (constants, map→section mapping, JSON config)
- [x] 14-02-PLAN.md — Save data expansion (QuizSaveDataV2, section-aware accessors)
- [x] 14-03-PLAN.md — Topic pool build tools (generate topic-indexed pools)
- [x] 14-04-PLAN.md — Quiz logic conversion (topic-based selection, section mastery)
- [x] 14-05-PLAN.md — Wild encounter re-capture (section-aware filtering)

**Details:**
Section-based system:
- Section 1 (→Brock): Plan Provisions, General Principles
- Section 2 (→Misty): Medical/Dental Claim Costs, Trend
- Section 3 (→Lt. Surge): Pharmacy, Trend Analysis
- Section 4 (→Erika): ASOPs topics
- Section 5 (→Koga): Experience Rating, Funding, Stop Loss
- Section 6 (→Sabrina): Selection, Risk topics
- Section 7 (→Blaine): Late game topics
- Section 8 (→Giovanni/E4): Mastery

Key changes:
- New map_topics.json: Maps → Section → Topics
- Save data: speciesCleared[section][species] instead of speciesCleared[species]
- Quiz logic: Select questions from current map's topic pool
- Re-capture: Same species in new section = new capture opportunity with new topics

---

### Phase 15: Per-Species Questions with Unique Location Species

**Goal:** Replace section-based topic pools with per-species question banks, making each species appear in only one location (or limited locations for evolved forms). Expand catchable roster.
**Depends on:** Phase 14
**Requirements:** TBD
**Success Criteria:**
  1. Each species appears in exactly one primary location
  2. Questions selected from species-specific bank (not topic pool)
  3. Global mastery tracking (no section awareness needed)
  4. Expanded roster: trade evolutions, fossils, starters, LeafGreen exclusives
**Plans:** 4 plans

Plans:
- [~] 15-01-PLAN.md — Encounter table redesign (unique species per location) — Sections 1-2 complete
- [~] 15-02-PLAN.md — Question-to-species assignment (difficulty tiers) — Sections 1-2 complete
- [x] 15-03-PLAN.md — Save system simplification (global mastery)
- [x] 15-04-PLAN.md — Quiz logic reversion (species bank selection)

**Details:**
Key changes:
- wild_encounters.json: Redistribute species so each appears in one location only
- species_map.json: Full 150+ species mapped to questions by difficulty tier
- Save data: Global speciesCleared and masteryCount (not per-section)
- Quiz logic: Quiz_GetBank(species) instead of topic pools
- Roster expansion: Trade evos, fossils, starters, version exclusives all catchable

**Completed (2026-02-05):**
- QuizSaveDataV2 simplified to global tracking (V3 format)
- Quiz logic uses Quiz_GetBank(species) for wild and single-question trainers
- Multi-question trainer progress display fixed (X/N turns)
- Forward declarations added for global mastery functions

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
| ENH-C. Route Completion | 2/2 | ✓ Complete | 2026-02-02 |
| ENH-D. Type-Based Moves | 0/? | Pending | - |
| 12. Scalable Battle System | 0/? | Stretch | - |
| 13. Uniform Move Rules | 2/2 | ✓ Complete | 2026-01-30 |
| 14. Section-Based Topic Pools | 5/5 | ✓ Complete | 2026-02-04 |
| 15. Per-Species Questions | 2/4 | In Progress | - |