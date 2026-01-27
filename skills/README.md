# Skills

Local skills catalog for this repo.

## Available Skills

- **peer-review-checklist** - Checklist for peer reviews
- **get-shit-done** - Meta-prompting and spec-driven development system for AI-assisted coding

## Get Shit Done (GSD)

A powerful workflow system for building projects with AI assistants. Solves "context rot" — the quality degradation that happens as the AI fills its context window.

### Quick Start

1. Read the SKILL.md: `skills/get-shit-done/SKILL.md`
2. Ask the AI to follow the workflow, e.g.:
   > "Let's start a new GSD project. Please read the new-project command and walk me through initialization."

### Key Commands

| Command | Description |
|---------|-------------|
| `/gsd:new-project` | Initialize project with deep questioning |
| `/gsd:map-codebase` | Analyze existing codebase |
| `/gsd:plan-phase N` | Create detailed execution plan |
| `/gsd:execute-phase N` | Execute plans with atomic commits |
| `/gsd:progress` | Check status and next steps |
| `/gsd:quick` | Ad-hoc task with GSD guarantees |

### Structure

```
skills/get-shit-done/
├── SKILL.md              # Main documentation
├── commands/gsd/         # Command definitions
├── agents/               # Specialized agent prompts
└── core/
    └── templates/        # Planning document templates
```

See `skills/get-shit-done/SKILL.md` for full documentation.
