# Pitfalls Research

## Critical Mistakes to Avoid

### 1. SaveBlock Overflow
**What goes wrong:** Adding too much data to SaveBlock1/2 exceeds sector limits.

**Warning signs:**
- Build fails with static assertion error
- Save corruption in testing
- Random crashes after loading save

**Prevention:**
- Calculate save data size before implementing
- Use bitmasks for mastery (1 byte per 8 questions)
- Test save/load early in development
- Add size check: `STATIC_ASSERT(sizeof(struct QuizSaveData) <= MAX_QUIZ_SAVE_SIZE)`

**Phase to address:** Phase 2 (Save Persistence)

---

### 2. Text Length Overflow
**What goes wrong:** Questions exceed screen width, text wraps awkwardly or crashes.

**Warning signs:**
- Questions cut off mid-word
- Text overlaps with battle UI
- Visual glitches during display

**Prevention:**
- Build tool validates max lengths:
  - Prompt: 120 chars (3 lines on GBA)
  - Answer: 40 chars (1 line)
- Use multi-page text for longer content
- Test with longest questions first

**Phase to address:** Phase 1 (Build Tools)

---

### 3. Hardcoded Question Patterns
**What goes wrong:** Code assumes specific question count or structure.

**Warning signs:**
- Works for Route 1, breaks on Route 2
- Adding questions requires code changes
- Magic numbers scattered through code

**Prevention:**
- All question counts from data, not code
- Use `bank->questionCount` not `5`
- Question selection handles variable bank sizes

**Phase to address:** Phase 1 (Question Bank System)

---

### 4. State Machine Gaps
**What goes wrong:** Edge cases in mastery/capture flow cause stuck states.

**Warning signs:**
- Player can't progress after mastering
- Capture quiz never triggers
- Species reappears after being cleared

**Prevention:**
- Explicit state enum (UNSEEN, LEARNING, CAPTURE_PENDING, CLEARED)
- Log state transitions during testing
- Test full loop: fresh save → mastery → capture → cleared

**Phase to address:** Phase 3 (Capture Flow)

---

### 5. Encounter Table Modification
**What goes wrong:** Modifying encounter tables breaks vanilla spawning or causes infinite loops.

**Warning signs:**
- No encounters in area
- Wrong species appearing
- Game freeze during encounter generation

**Prevention:**
- Don't modify tables; filter at selection time
- Reroll with MAX_REROLLS cap (10)
- Fallback: "no encounter" if all species cleared
- Test with mostly-cleared save state

**Phase to address:** Phase 4 (Cleared Species Filter)

---

### 6. Battle Engine Modifications
**What goes wrong:** Changes to battle files break trainer battles or cause desyncs.

**Warning signs:**
- Trainer battles behave wrong
- Link battles crash
- Safari Zone broken

**Prevention:**
- All quiz logic through quiz_hooks.c
- Check `Quiz_IsActive()` before applying any override
- Test trainer battles after every battle engine change

**Phase to address:** Ongoing

---

### 7. Memory Leaks in EWRAM
**What goes wrong:** Not cleaning up quiz state between battles.

**Warning signs:**
- Stale question shown
- Wrong mastery state
- Memory fills up over long play sessions

**Prevention:**
- Clear quiz state at battle end
- Initialize state at battle start
- Don't allocate dynamically; use static EWRAM_DATA

**Phase to address:** Phase 1 (Quiz State Management)

---

### 8. CSV Parsing Edge Cases
**What goes wrong:** Special characters in questions break build tool.

**Warning signs:**
- Build tool crashes on certain questions
- Incorrect text in game
- Missing questions in generated tables

**Prevention:**
- Handle commas in CSV (proper quoting)
- Escape special characters for GBA charmap
- Validate every question in CSV
- Unit test build tool with edge cases

**Phase to address:** Phase 1 (Build Tools)

---

## Pitfall Priority by Phase

| Phase | Pitfalls to Address |
|-------|---------------------|
| Phase 1 | Text Length, Hardcoded Patterns, CSV Parsing, Memory Leaks |
| Phase 2 | SaveBlock Overflow |
| Phase 3 | State Machine Gaps |
| Phase 4 | Encounter Table Modification |
| All | Battle Engine Modifications |
