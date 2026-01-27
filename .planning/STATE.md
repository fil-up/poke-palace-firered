# Project State

## Project Reference

See: .planning/PROJECT.md (updated 2026-01-24)

**Core value:** Quiz-driven gameplay loop must work end-to-end
**Current focus:** Post-MVP Enhancements (ENH-G through ENH-D)

## Current Position

Phase: 10 of 12 (Route 2 & Pewter Content)
Plan: 1 of 1 in current phase
Status: Complete — MVP DONE!
Last activity: 2026-01-26 — Phase 10 completed ✓

Progress: [████████████████████] 100% (MVP)

## Performance Metrics

**Velocity:**
- Total plans completed: 0
- Average duration: N/A
- Total execution time: 0 hours

**By Phase:**

| Phase | Plans | Total | Avg/Plan |
|-------|-------|-------|----------|
| - | - | - | - |

## Accumulated Context

### Decisions

Decisions are logged in PROJECT.md Key Decisions table.
Recent decisions affecting current work:

- [Init]: Use CSV → Python → C tables pipeline
- [Init]: Extend SaveBlock1 for quiz data
- [Init]: B_WIN_QUIZ (ID 25) for question display

### Pending Todos

**Bug Fixes (Priority):**

| ID | Issue | Status | Description |
|----|-------|--------|-------------|
| BUG-001 | Capture quiz healthbox overlay | ✓ DONE | Subsequent questions covered by opponent name/HP window |
| BUG-002 | Question text cutoff | ✓ DONE | Fixed OBJ window masking + line width increased to 42 chars |
| BUG-003 | Non-attacking moves | ✓ DONE | Remove all non-damage moves from the game entirely |
| BUG-004 | Secondary effects & status moves | ✓ DONE | Disable secondary effects; Pokemon should only use damage moves |
| BUG-005 | Multi-Pokemon trainer questions | ✓ DONE | Questions should be per-Pokemon, not per-battle; each Pokemon shows its species' questions |
| BUG-006 | Abilities & type immunities | ✓ DONE | Disable special ability checks; ignore type immunities (e.g., Ghost hit by Normal) |

**Stretch Goals:**
- Gym battles: Each Pokemon asks TWO questions instead of one

**Post-MVP Enhancements (Priority Order):**

| ID | Feature | Size | Status | Description |
|----|---------|------|--------|-------------|
| ENH-G | Explanation A-to-advance | Tiny | ✓ DONE | Explanation persists until A pressed, no timed delay |
| ENH-E | Encounter initiation label | Tiny | ✓ DONE | Show "Quiz Encounter!" or "Catch Opportunity!" at battle start |
| ENH-B | Progress X/Y display | Small | ✓ DONE | PP area shows mastery/encounter progress; TYPE area shows encounter label |
| ENH-A | 5-line question box | Medium | Pending | Expand question box to 5 lines, never overlap command menu |
| ENH-F | Capture animation on final correct | Medium | Pending | Final correct answer triggers guaranteed catch animation + boxing |
| ENH-C | Route completion 25% encounters | Medium-High | Pending | Grass/water tracked separately per route; 25% rate when complete |
| ENH-D | Type-based moves (no effects) | High | Pending | 2 moves per type, dual-type gets 2+2, no secondary effects ever |

### Blockers/Concerns

None yet.

## Session Continuity

Last session: 2026-01-27
Stopped at: Completed ENH-B (Progress X/Y display)
Resume with: ENH-A (5-line question box)

### What's Working
- Build tools (CSV → C generation)
- Question bank integration
- Save persistence (mastery tracking)
- Core loop (questions don't repeat, state transitions)
- Capture flow (CAPTURE_PENDING → sequential quiz → party add)
- Encounter filter (cleared species skipped, reroll capped at 10)
- UI polish (explanations shown after correct answers)
- Route 1 content (Rattata: 5 questions, Pidgey: 5 questions)
- Viridian Forest content (Caterpie, Weedle, Metapod, Kakuna, Pikachu: 5 questions each)
- Route 2 content (Nidoran-F, Nidoran-M: 5 questions each)
- Gym leader mechanic (Brock: 2 questions per Pokemon, 3 for final)
- Area pool aggregation (all prior area questions available for gym battles)
- Encounter label ("Quiz Encounter!" / "Catch Opportunity!" at battle start)
- Progress display (PP area shows "Progress X/Y", TYPE area shows encounter type label)
- Debug quick start mode (auto-advance through intro, instant text)

### What's Next
- **MVP COMPLETE!** Core quiz-driven gameplay loop is fully functional
- Working through post-MVP enhancements (ENH-G → ENH-D)
- Phases 11-12: Stretch goals (gym exam, trainer reinforcement) deferred

## Debug Quick Start Mode

A development feature to speed up testing by skipping the game intro sequence.

**Activation:** Set `DEBUG_QUICK_START 1` in:
- `pokefirered/src/new_game.c` - Sets up player name "RED", Charmander, flags
- `pokefirered/src/oak_speech.c` - Auto-advances through all intro screens

**What it does:**
1. Auto-advances through Controls Guide (all 3 pages)
2. Auto-advances through Pikachu intro (all pages)
3. Sets instant text speed for Oak's dialogue
4. Auto-selects BOY gender
5. Auto-selects first default name for player and rival
6. Auto-confirms name selections
7. Sets player name to "RED" with level 5 Charmander
8. Sets flags to bypass Oak stopping player on Route 1
9. Gives 10 Poke Balls for testing captures
10. Warps to Pallet Town near Route 1 exit

**To disable:** Set `DEBUG_QUICK_START 0` in both files before release builds.
