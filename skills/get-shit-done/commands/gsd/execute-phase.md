---
name: gsd:execute-phase
description: Execute all plans in a phase
argument-hint: "<phase-number>"
---

# Execute Phase Command

Execute all plans in a phase with parallel execution where possible.

## Usage

```
/gsd:execute-phase 1
```

## What It Does

1. **Loads Phase Plans**
   - Finds all PLAN.md files in phase directory
   - Groups by wave number
   - Identifies dependencies

2. **Executes Plans in Waves**
   - Wave 1 plans run in parallel
   - Wave 2 waits for Wave 1 to complete
   - And so on...

3. **For Each Plan**
   - Reads task instructions
   - Implements code changes
   - Runs verification
   - Creates atomic commit per task
   - Creates SUMMARY.md

4. **Updates State**
   - Updates STATE.md position
   - Logs decisions
   - Records blockers/concerns

5. **Verifies Phase Goals**
   - Checks codebase against success criteria
   - Creates VERIFICATION.md if issues found

## Execution Flow

```
Load plans → Group by wave → Execute parallel → Verify → Summary
```

## Atomic Commits

Each task gets its own commit:

```
feat(01-02): add user registration endpoint
fix(01-02): correct email validation
```

## Deviation Handling

While executing, Claude will discover work not in the plan:

**Rule 1: Auto-fix bugs**
- Fix immediately, track for Summary

**Rule 2: Auto-add missing critical functionality**
- Add immediately if essential for correctness/security

**Rule 3: Auto-fix blocking issues**
- Fix if it prevents task completion

**Rule 4: Ask about architectural changes**
- STOP and return checkpoint for user decision

## Checkpoints

When a plan has checkpoints:
- Execute until checkpoint
- STOP and present status
- Wait for user response
- Continue with fresh context

## Output

- SUMMARY.md for each plan
- Atomic commits for each task
- Updated STATE.md

## Next Steps

After execution:
- `/gsd:verify-work {phase}` — user acceptance testing
- `/gsd:progress` — see what's next
