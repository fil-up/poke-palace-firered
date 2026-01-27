---
name: gsd:help
description: Show available GSD commands and usage guide
---

# GSD Command Reference

**GSD** (Get Shit Done) creates hierarchical project plans optimized for solo agentic development.

## Quick Start

1. `/gsd:new-project` - Initialize project
2. `/gsd:plan-phase 1` - Create detailed plan for first phase
3. `/gsd:execute-phase 1` - Execute the phase

## Core Workflow

```
/gsd:new-project → /gsd:plan-phase → /gsd:execute-phase → repeat
```

## Commands

### Project Initialization

| Command | What it does |
|---------|--------------|
| `/gsd:new-project` | Full initialization: questions → research → requirements → roadmap |
| `/gsd:map-codebase` | Analyze existing codebase before new-project |

### Phase Planning

| Command | What it does |
|---------|--------------|
| `/gsd:discuss-phase N` | Capture implementation decisions before planning |
| `/gsd:research-phase N` | Deep domain research for complex phases |
| `/gsd:list-phase-assumptions N` | See Claude's intended approach |
| `/gsd:plan-phase N` | Research + plan + verify for a phase |

### Execution

| Command | What it does |
|---------|--------------|
| `/gsd:execute-phase N` | Execute all plans in parallel waves |
| `/gsd:verify-work N` | Manual user acceptance testing |
| `/gsd:quick` | Execute ad-hoc task with GSD guarantees |

### Roadmap Management

| Command | What it does |
|---------|--------------|
| `/gsd:add-phase "desc"` | Add new phase to end |
| `/gsd:insert-phase N "desc"` | Insert urgent work between phases |
| `/gsd:remove-phase N` | Remove a future phase |

### Milestone Management

| Command | What it does |
|---------|--------------|
| `/gsd:audit-milestone` | Verify milestone completion |
| `/gsd:complete-milestone` | Archive milestone, tag release |
| `/gsd:new-milestone` | Start next version cycle |

### Navigation

| Command | What it does |
|---------|--------------|
| `/gsd:progress` | Where am I? What's next? |
| `/gsd:resume-work` | Restore from last session |
| `/gsd:pause-work` | Create handoff context |

### Utilities

| Command | What it does |
|---------|--------------|
| `/gsd:settings` | Configure workflow toggles |
| `/gsd:add-todo` | Capture idea for later |
| `/gsd:check-todos` | List pending todos |
| `/gsd:debug` | Systematic debugging |

## Files & Structure

```
.planning/
├── PROJECT.md            # Project vision
├── ROADMAP.md            # Phase breakdown
├── STATE.md              # Project memory
├── REQUIREMENTS.md       # Scoped requirements
├── config.json           # Workflow settings
├── research/             # Domain research
├── todos/                # Captured ideas
├── debug/                # Debug sessions
└── phases/
    └── 01-foundation/
        ├── 01-01-PLAN.md
        └── 01-01-SUMMARY.md
```

## Common Workflows

**Starting a new project:**
```
/gsd:new-project
/clear
/gsd:plan-phase 1
/clear
/gsd:execute-phase 1
```

**Resuming work:**
```
/gsd:progress
```

**Adding urgent work:**
```
/gsd:insert-phase 5 "Critical security fix"
/gsd:plan-phase 5.1
/gsd:execute-phase 5.1
```

**Completing a milestone:**
```
/gsd:complete-milestone 1.0.0
/clear
/gsd:new-milestone
```
