# Codebase Concerns

**Analysis Date:** 2026-01-30

## Tech Debt

**Duplicate Phase 13 Directories:**
- Issue: Two phase 13 directories exist with inconsistent completion states
- Files: `.planning/phases/13-damage-only-moves/` (incomplete, only has 13-01-PLAN.md)
- Files: `.planning/phases/13-make-all-moves-the-same-from-a-damage-accuracy-perspective-but-retain-only-the-move-animation-for-cosmetic-purposes/` (complete with summaries)
- Impact: Confusion about which implementation plan was executed; the shorter-named directory expects `move_rules.c/h` artifacts that don't exist
- Fix approach: Delete the incomplete `13-damage-only-moves` directory after confirming the longer-named phase is the canonical implementation

**Missing move_rules Module (Planned but Not Created):**
- Issue: `13-damage-only-moves/13-01-PLAN.md` specifies creating `pokefirered/src/quiz/move_rules.c` and `pokefirered/include/quiz/move_rules.h` with `Quiz_IsDamageOnlyMove` function, but these files don't exist
- Files: `pokefirered/src/quiz/` (missing `move_rules.c`)
- Files: `pokefirered/include/quiz/` (missing `move_rules.h`)
- Impact: If this plan was the intended approach, the damage-only enforcement is incomplete; current implementation uses a different approach in `quiz_hooks.c`
- Fix approach: Either delete the incomplete plan or implement the module if stricter move validation is desired

**Untracked temp-gsd Directory:**
- Issue: `temp-gsd/` directory is untracked and contains what appears to be GSD tooling artifacts (package.json, CHANGELOG.md, hooks/)
- Files: `temp-gsd/`
- Impact: Leftover development artifacts cluttering the repository
- Fix approach: Either add to `.gitignore` or delete if no longer needed

**Pending Todo Files Not Completed:**
- Issue: Two pending todo files describe features that haven't been implemented
- Files: `.planning/todos/pending/2026-01-29-ensure-only-attacking-moves.md`
- Files: `.planning/todos/pending/2026-01-30-make-hms-forgettable.md`
- Impact: Tracked work that may or may not still be relevant after phase 13 changes
- Fix approach: Review todos and either complete, mark as done (if addressed by phase 13), or remove if no longer needed

**Species Save Slot Collision Risk:**
- Issue: `Quiz_GetSaveIndex()` uses simple modulo mapping (`species % QUIZ_SAVE_MAX_SPECIES`) with a TODO comment acknowledging collision risk
- Files: `pokefirered/src/quiz/quiz.c` (line 908)
- Impact: Different species may share save slots, causing mastery data corruption if colliding species are both encountered
- Fix approach: Implement a proper hash or direct mapping table for supported species

## Known Bugs

**BUG-010: Capture Nickname Freeze (Party Full):**
- Symptoms: Capture success freezes after choosing "No" on nickname prompt when party is full
- Files: `pokefirered/src/quiz/quiz.c`, battle scripts
- Trigger: Capture a Pokémon when party is full, decline nickname prompt
- Workaround: Always accept the default nickname when party is full
- Status: OPEN (documented in STATE.md)

**BUG-007: Command Menu Visual Distortion (Partial):**
- Symptoms: Some visual distortion in FIGHT/BAG/POKEMON/RUN section at bottom of screen during quiz battles
- Files: `pokefirered/src/quiz/quiz.c` (Quiz_OnCommandMenuShown)
- Trigger: Quiz window display during battle
- Workaround: None required - cosmetic only, gameplay functional
- Status: PARTIAL fix applied; quiz displays correctly but command menu has minor distortion

## Documentation Inconsistencies

**ROADMAP.md vs STATE.md Mismatch:**
- Issue: ROADMAP.md shows ENH-C (Route Completion) and ENH-D (Type-Based Moves) as incomplete, but STATE.md shows ENH-C as DONE
- Files: `.planning/ROADMAP.md` (line 19, 227)
- Files: `.planning/STATE.md` (line 77)
- Impact: Confusion about actual project status
- Fix approach: Update ROADMAP.md to reflect current completion state

**Outdated PROJECT.md Active Requirements:**
- Issue: PROJECT.md "Active" section still lists requirements as unchecked that have been implemented (save persistence, capture flow, etc.)
- Files: `.planning/PROJECT.md` (lines 28-41)
- Impact: Misleading project status for new contributors
- Fix approach: Update checkboxes to reflect completed work

## Performance Bottlenecks

**Route Completion Cache Invalidation:**
- Problem: `InvalidateCompletionCache()` clears entire cache on any species state change
- Files: `pokefirered/src/quiz/route_completion.c` (lines 53-60)
- Cause: Conservative invalidation strategy; could invalidate only affected map header
- Improvement path: Track which header IDs contain which species and invalidate selectively

**Question Bank Build-Time Only:**
- Problem: All questions are compiled into ROM; no runtime question loading
- Files: `pokefirered/src/quiz/questions_gen.c` (~3300 lines of generated content)
- Cause: GBA platform constraints require compile-time data
- Improvement path: Not actionable for GBA; would require different platform

## Fragile Areas

**Quiz Window Z-Order Management:**
- Files: `pokefirered/src/quiz/quiz.c` (lines 714-736)
- Why fragile: Must manually set BG1 priority and blend modes; other battle operations can reset these
- Safe modification: Always verify BG1 priority is set to 0 before quiz drawing; test all battle scenarios (level-up, animation, etc.)
- Test coverage: Manual testing only; no automated verification

**Multi-Hit Counter Clamping:**
- Files: `pokefirered/src/quiz/quiz_hooks.c` (lines 331-341)
- Why fragile: Modifies global `gMultiHitCounter` to clamp multi-hit visuals; edge cases with specific moves may bypass
- Safe modification: Test with all multi-hit moves (Double Slap, Fury Attack, etc.)
- Test coverage: Manual testing only

**Trainer Tier Detection:**
- Files: `pokefirered/src/quiz/trainer_scaling.c`
- Why fragile: Relies on explicit trainer ID lists and class constants; new trainers must be manually added
- Safe modification: Update `sGymTrainerIds` array when adding new gym trainers
- Test coverage: None - requires playing through relevant battles

## Dependencies at Risk

**agbcc Submodule:**
- Risk: Git status shows `agbcc` submodule modified; may be out of sync with expected state
- Impact: Build may fail or produce unexpected results if submodule is at wrong commit
- Files: `agbcc/` (marked as modified in git status)
- Migration plan: Verify submodule is at correct commit with `git submodule update --init`

**pokefirered Submodule:**
- Risk: Git status shows `pokefirered` submodule modified; custom changes may conflict with upstream
- Impact: Upstream merges could be difficult; changes may be lost if submodule reset incorrectly
- Files: `pokefirered/` (marked as modified in git status)
- Migration plan: Ensure all quiz-related changes are properly committed within submodule

## Missing Critical Features

**DEBUG_QUICK_START State Unknown:**
- Problem: STATE.md documents DEBUG_QUICK_START feature for dev testing but unclear if properly disabled for release
- Blocks: Production builds may skip intro unintentionally
- Files: `pokefirered/src/new_game.c`, `pokefirered/src/oak_speech.c`
- Fix approach: Verify `DEBUG_QUICK_START` is set to 0 in both files before release builds

**HM Move Forgetting:**
- Problem: HM moves cannot be forgotten (vanilla behavior), pending todo created but not implemented
- Blocks: Quality-of-life improvement for move management
- Files: `.planning/todos/pending/2026-01-30-make-hms-forgettable.md`

## Test Coverage Gaps

**No Automated Battle Testing:**
- What's not tested: All quiz battle mechanics require manual in-game testing
- Files: `pokefirered/src/quiz/quiz.c`, `pokefirered/src/quiz/quiz_hooks.c`
- Risk: Regressions in battle behavior may go unnoticed
- Priority: Medium - manual testing is time-consuming but currently the only option

**Safari Zone Exclusion:**
- What's not tested: Safari battles should NOT apply quiz rules
- Files: `pokefirered/src/quiz/quiz_hooks.c` (QuizHooks_IsQuizBattle line 308)
- Risk: Safari mechanics could be broken if quiz hooks are incorrectly applied
- Priority: High - documented in VERIFICATION.md as requiring human verification

**Capture When Party Full:**
- What's not tested: BUG-010 indicates this flow is broken
- Files: Capture handling after party full state
- Risk: Players can't complete captures when party is full without workaround
- Priority: High - directly affects core gameplay loop

## Architecture Concerns

**B_WIN_QUIZ Window Definition:**
- Issue: `B_WIN_QUIZ` (ID 25 per STATE.md) is used in `quiz.c` but definition location unclear
- Files: `pokefirered/src/quiz/quiz.c` (lines 707, 734-738, 754, 816, 844)
- Files: `pokefirered/include/constants/battle.h` (defines B_WIN_* 0-24, no B_WIN_QUIZ)
- Impact: Must be defined somewhere for build to succeed; if missing, build would fail
- Fix approach: Verify B_WIN_QUIZ is defined and properly integrated with battle window system

**Quiz State Global Mutability:**
- Issue: `sQuizState` is a global EWRAM struct modified from multiple call sites
- Files: `pokefirered/src/quiz/quiz.c` (line 83)
- Impact: Race conditions unlikely on GBA (single-threaded), but state can become inconsistent if functions called in unexpected order
- Fix approach: Document expected call order; consider adding state validation assertions

---

*Concerns audit: 2026-01-30*
