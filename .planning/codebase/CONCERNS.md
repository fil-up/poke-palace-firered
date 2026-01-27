# Concerns

## Technical Debt

### 1. Hardcoded Question (Critical)
**Location**: `src/quiz/quiz.c`
**Issue**: Single hardcoded question instead of question bank
**Impact**: Must be replaced before MVP completion
**Plan**: Implement question bank system per ENGINEERING_BRIEF.md

### 2. Answer Always Slot 0 (Critical)
**Location**: `src/quiz/quiz.c::Quiz_OnMoveConfirmed()`
**Issue**: `moveSlot == 0` hardcoded as correct answer
**Impact**: Not connected to actual question data
**Plan**: Grade against loaded question's `answer_index`

### 3. No Save Persistence (Critical)
**Location**: Not implemented
**Issue**: Mastery progress not saved
**Impact**: Players lose all progress on game restart
**Plan**: Implement `quiz_save.c` per ENGINEERING_BRIEF.md

### 4. No Capture Quiz Flow (High)
**Location**: Not implemented
**Issue**: Missing CAPTURE_PENDING → CLEARED state machine
**Impact**: Cannot complete core gameplay loop
**Plan**: Implement per DESIGN.md state machine

### 5. No Cleared Species Filtering (High)
**Location**: `wild_encounter.c` not modified
**Issue**: Cleared species still appear as encounters
**Impact**: Players re-encounter completed Pokémon
**Plan**: Add reroll logic per DESIGN.md

## Architecture Concerns

### 6. B_WIN_QUIZ Window Not Defined
**Severity**: Unknown
**Issue**: `B_WIN_QUIZ` referenced but may need to be added to battle interface
**Verify**: Check if window ID is properly defined

### 7. Quiz Window Positioning
**Severity**: Medium
**Issue**: Question text may overlap with other UI elements
**Plan**: Test and adjust window coordinates

## Build & Tooling

### 8. Missing Build Tools
**Location**: `tools/` directory empty
**Issue**: `validate_questions.py` and `build_questions.py` not created
**Impact**: No automated data validation
**Plan**: Create tools as specified in ENGINEERING_BRIEF.md

### 9. No CI Pipeline
**Location**: `.github/workflows/`
**Issue**: Only AI PR review, no build verification
**Impact**: Broken builds may be merged
**Plan**: Add build + validate workflow

## Documentation Gaps

### 10. DATA_SCHEMA.md Missing
**Issue**: Referenced in ENGINEERING_BRIEF.md but not created
**Impact**: No formal schema for question data
**Plan**: Create before populating question banks

## Risk Areas

### Battle Engine Modifications
- **Risk**: Changes to core battle files may break with upstream updates
- **Mitigation**: Keep modifications minimal, route through quiz_hooks.c

### Save Data Versioning
- **Risk**: Save format changes could corrupt player saves
- **Mitigation**: Include version field, wipe quiz progress on mismatch

### GBA Memory Constraints
- **Risk**: Large question banks may exceed ROM/RAM limits
- **Mitigation**: Compact encoding, test with full question sets

## Priority Order

1. **Question Bank System** — Replace hardcoded question
2. **Save Persistence** — Mastery must survive save/load
3. **Capture Quiz Flow** — Complete core loop
4. **Cleared Species Filter** — Prevent repeat encounters
5. **Build Tools** — Validate data before build
6. **CI Pipeline** — Catch issues early
