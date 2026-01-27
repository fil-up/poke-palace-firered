---
name: gsd:plan-phase
description: Create detailed execution plan for a specific phase
argument-hint: "<phase-number>"
---

# Plan Phase Command

Create detailed execution plans for a specific phase.

## Usage

```
/gsd:plan-phase 1
```

## What It Does

1. **Loads Project State**
   - Reads STATE.md for current position and decisions
   - Reads ROADMAP.md for phase goals
   - Reads any CONTEXT.md from `/gsd:discuss-phase`

2. **Gathers Phase Context**
   - Phase goal from roadmap
   - What exists already
   - Dependencies met

3. **Breaks Into Tasks**
   - Decomposes phase into tasks
   - Maps dependencies (needs/creates)
   - Identifies parallel opportunities

4. **Groups Into Plans**
   - 2-3 tasks per plan
   - ~50% context target
   - Wave assignments for parallelism

5. **Writes PLAN.md Files**
   - Creates `.planning/phases/XX-name/{phase}-{NN}-PLAN.md`
   - Includes objective, context, tasks, verification

## Plan Structure

```markdown
---
phase: XX-name
plan: NN
type: execute
wave: N
depends_on: []
files_modified: []
autonomous: true
must_haves:
  truths: []
  artifacts: []
  key_links: []
---

<objective>
What this plan accomplishes
</objective>

<context>
@.planning/PROJECT.md
@.planning/ROADMAP.md
</context>

<tasks>
<task type="auto">
  <name>Task 1</name>
  <files>path/to/file.ext</files>
  <action>Specific implementation</action>
  <verify>Command or check</verify>
  <done>Acceptance criteria</done>
</task>
</tasks>

<verification>
Overall checks
</verification>

<success_criteria>
Measurable completion
</success_criteria>
```

## Task Types

| Type | Use For |
|------|---------|
| `auto` | Everything Claude can do independently |
| `checkpoint:human-verify` | Visual/functional verification |
| `checkpoint:decision` | Implementation choices |
| `checkpoint:human-action` | Truly unavoidable manual steps (rare) |

## Task Sizing

Each task should take Claude 15-60 minutes to execute:
- < 15 min: Too small — combine
- 15-60 min: Right size
- > 60 min: Too large — split

## Output

Creates `.planning/phases/XX-name/{phase}-{NN}-PLAN.md` files

## Next Step

After planning: `/gsd:execute-phase {phase}`
