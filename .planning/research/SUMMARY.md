# Research Summary

## Key Findings

### Stack Decisions (Locked)
| Decision | Choice | Confidence |
|----------|--------|------------|
| Base decomp | pokefirered (already using) | High |
| Compiler | agbcc (MODERN=0) | High |
| Data pipeline | CSV → Python → C tables | High |
| Save approach | Extend SaveBlock1 | High |
| Text display | Use existing battle window system | High |

### Architecture Patterns
1. **Question Bank:** Compile-time C tables from CSV
2. **Save Data:** Bitmask per (scope, species) in SaveBlock1
3. **State Machine:** UNSEEN → LEARNING → CAPTURE_PENDING → CLEARED
4. **Encounter Filter:** Reroll cleared species (max 10 attempts)
5. **Text Handling:** Multi-page via battle message callbacks

### Critical Findings

**Already Working:**
- `B_WIN_QUIZ` window ID defined (constant 25)
- `BattlePutTextOnWindow()` ready for text display
- Quiz hooks infrastructure established
- Damage override system functional

**Must Build:**
- CSV → C build pipeline
- SaveBlock1 extension for quiz data
- Question selection from mastery mask
- Capture quiz state machine
- Cleared species encounter filter
- Multi-page text pagination

### Table Stakes Features
- Question display ✓
- Answer selection ✓
- Progress persistence ⬜
- Capture quiz flow ⬜
- Species clearing ⬜

### Pitfalls to Avoid
1. **SaveBlock overflow** — Calculate size upfront
2. **Text length overflow** — Validate in build tool
3. **State machine gaps** — Test full loop
4. **Encounter infinite loops** — Reroll cap
5. **Battle engine breakage** — Always check Quiz_IsActive()

## Recommended Phase Structure

| Phase | Focus | Key Deliverable |
|-------|-------|-----------------|
| 1 | Foundation | CSV → C pipeline, build tools |
| 2 | Persistence | SaveBlock integration, mastery bits |
| 3 | Core Loop | Question bank, selection, grading |
| 4 | Capture | State machine, capture quiz flow |
| 5 | Filter | Cleared species reroll |
| 6 | Polish | Multi-page text, explanations |
| 7 | Content | Route 1 question bank |
| 8 | Content | Viridian Forest question bank |
| 9 | Content | Route 2, Pewter City questions |
| 10 | Stretch | Gym exam (if time) |
| 11 | Stretch | Trainer reinforcement (if time) |

## Ready for Roadmap

Research confirms:
- Existing codebase has good foundation
- Clear integration points identified
- Data patterns established
- Risk areas documented

**Proceed to requirements definition and roadmap creation.**
