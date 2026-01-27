# Get Shit Done (GSD)

A meta-prompting, context engineering and spec-driven development system for AI-assisted coding.

## What is GSD?

GSD is a workflow system that helps you build projects with AI assistants like Claude. It solves "context rot" — the quality degradation that happens as the AI fills its context window.

## Core Workflow

```
/gsd:new-project → /gsd:plan-phase → /gsd:execute-phase → repeat
```

### 1. Initialize Project
```
/gsd:new-project
```
- Deep questioning to understand your idea
- Optional domain research
- Requirements definition with scoping
- Roadmap creation with phases

### 2. Plan a Phase
```
/gsd:plan-phase 1
```
- Creates detailed execution plans
- 2-3 tasks per plan
- Includes verification criteria

### 3. Execute the Phase
```
/gsd:execute-phase 1
```
- Runs plans (parallel when possible)
- Atomic commits per task
- Verifies against goals

### 4. Verify Work
```
/gsd:verify-work 1
```
- Manual user acceptance testing
- Diagnoses failures automatically
- Creates fix plans if needed

## Key Commands

| Command | What it does |
|---------|--------------|
| `/gsd:new-project` | Full initialization |
| `/gsd:map-codebase` | Analyze existing codebase |
| `/gsd:discuss-phase N` | Capture implementation decisions |
| `/gsd:plan-phase N` | Research + plan + verify |
| `/gsd:execute-phase N` | Execute all plans |
| `/gsd:verify-work N` | User acceptance testing |
| `/gsd:progress` | Check status and next steps |
| `/gsd:quick` | Ad-hoc task with GSD guarantees |
| `/gsd:debug` | Systematic debugging |

## Files Created

```
.planning/
├── PROJECT.md            # Project vision
├── ROADMAP.md            # Phase breakdown
├── STATE.md              # Project memory
├── REQUIREMENTS.md       # Scoped requirements
├── config.json           # Workflow settings
├── research/             # Domain research
└── phases/
    └── 01-foundation/
        ├── 01-01-PLAN.md
        └── 01-01-SUMMARY.md
```

## Using GSD in Cursor

Since GSD was designed for Claude Code CLI, in Cursor you can:

1. **Reference the commands directly**: Read the command files in `skills/get-shit-done/commands/gsd/` to understand what each command does, then ask the AI to follow that workflow.

2. **Use the agents as context**: The agent files in `skills/get-shit-done/agents/` contain detailed instructions for specific tasks (planning, executing, debugging).

3. **Follow the templates**: Use templates in `skills/get-shit-done/core/templates/` when creating planning artifacts.

### Example Usage in Cursor

Instead of typing `/gsd:new-project`, say:
> "I want to start a new GSD project. Please read the new-project command from skills/get-shit-done and walk me through the initialization process."

Or for planning:
> "Let's plan phase 1 using the GSD workflow. Please read the plan-phase command and the gsd-planner agent to guide the process."

## Philosophy

- **Solo Developer Focus**: One person + one AI = ship fast
- **Plans Are Prompts**: PLAN.md files are the actual prompts for execution
- **Context Engineering**: Stay under 50% context usage for quality
- **Atomic Commits**: Each task gets its own commit

## Resources

- [GitHub Repo](https://github.com/glittercowboy/get-shit-done)
- Commands: `skills/get-shit-done/commands/gsd/`
- Agents: `skills/get-shit-done/agents/`
- Templates: `skills/get-shit-done/core/templates/`
- Workflows: `skills/get-shit-done/core/workflows/`
