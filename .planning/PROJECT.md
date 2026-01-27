# Poke Palace FireRed

## What This Is

A Pokémon FireRed ROM hack that replaces traditional battle mechanics with quiz questions. Players learn by answering questions during wild encounters, master species through correct answers, and "capture" them by completing a perfect quiz run. Currently focused on actuarial exam prep (CAS GH 101) but designed to be subject-agnostic via CSV-based question banks.

## Core Value

**Quiz-driven gameplay loop must work end-to-end**: encounter → question → answer → mastery → capture quiz → species cleared. If this loop doesn't feel satisfying, nothing else matters.

## Requirements

### Validated

<!-- Shipped and confirmed valuable. -->

- ✓ Quiz question appears during wild encounter — existing
- ✓ Move selection maps to answer choices (A/B/C/D) — existing
- ✓ Correct answer = 1000 damage (instant KO) — existing
- ✓ Wrong answer = 1 damage, enemy deals maxHP/2 — existing
- ✓ Player always acts first in quiz mode — existing
- ✓ Moves always hit, infinite PP — existing

### Active

<!-- Current scope. Building toward these. -->

- [ ] Question bank system loads questions from generated C tables
- [ ] Questions sourced from CSV file in repo
- [ ] 5-10 questions per species (rarer Pokémon = more questions)
- [ ] Questions mapped by topic AND difficulty (topics for gyms, difficulty for routes)
- [ ] Mastery tracking persists through save/load
- [ ] Mastery state machine: UNSEEN → LEARNING → CAPTURE_PENDING → CLEARED
- [ ] Capture quiz triggers after mastering all questions for a species
- [ ] Perfect capture quiz = Pokémon caught + species cleared
- [ ] Failed capture quiz = Pokémon flees, retry on next encounter
- [ ] Cleared species no longer appear in wild encounters (reroll logic)
- [ ] Multi-page text boxes for long questions (auto-paginate)
- [ ] Show explanations on correct answers (when foe faints)
- [ ] MVP covers Route 1 through Pewter Gym

### Out of Scope

<!-- Explicit boundaries. Includes reasoning to prevent re-adding. -->

- Trainer reinforcement (spaced repetition) — Milestone 5, stretch goal only
- Gym section exams — stretch goal, not required for MVP
- Traditional battle mechanics — entire point is quiz replacement
- Difficulty modes — MVP uses fixed question counts
- Multiple save slots for quiz progress — single save is fine
- Online features — ROM hack, offline only

## Context

### Target Audience
- Primary: Personal learning tool for actuarial exam prep
- Future: Public ROM hack release for others studying professional exams

### Technical Environment
- Base: pret/pokefirered decompilation
- Compiler: agbcc (Nintendo-matching GBA C compiler)
- Platform: Game Boy Advance (ARM7TDMI, 16MHz, 32KB+256KB RAM)
- Question source: CSV files converted to C tables at build time

### Current State
- At approximately Milestone 1 of ENGINEERING_BRIEF.md
- Basic quiz hooks implemented with hardcoded single question
- Battle engine integration points established
- No save persistence, no question bank, no capture flow

### Question Bank
- Source: `GH 101 Question Bank - Sheet1.csv` (~150+ questions)
- Format: Question_ID, Broad_Category, Sub_Topic, Question_String, Option_Label, Option_Text, Correct_Answer, Difficulty_Level, Explanation
- Topics map to gym themes, difficulty maps to route progression

## Constraints

- **Platform**: GBA hardware limits (240x160 display, limited RAM)
- **Text Length**: Questions may exceed screen width, need multi-page handling
- **Build System**: Must integrate with existing Makefile, generate C tables from CSV
- **Save Format**: Must fit in existing FireRed save structure with versioning
- **Decomp Compatibility**: Minimize changes to core battle engine files

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Use move menu for answer selection | Natural 4-choice mapping to A/B/C/D | ✓ Good |
| Force player first, always hit | Simplifies battle flow for quiz mode | ✓ Good |
| CSV → C tables at build time | No runtime parsing, fits GBA constraints | — Pending |
| 5-10 questions per species | Balances learning depth with progression | — Pending |
| Show explanations on correct only | Reward correct answers with learning | — Pending |
| Multi-page text boxes | Handles long actuarial questions | — Pending |

---
*Last updated: 2026-01-24 after GSD initialization*
