---
name: gsd:new-project
description: Initialize a new project with deep context gathering and PROJECT.md
---

# New Project Command

Initialize a new project through unified flow: questioning → research (optional) → requirements → roadmap.

This is the most leveraged moment in any project. Deep questioning here means better plans, better execution, better outcomes.

## What It Creates

- `.planning/PROJECT.md` — project context
- `.planning/config.json` — workflow preferences
- `.planning/research/` — domain research (optional)
- `.planning/REQUIREMENTS.md` — scoped requirements
- `.planning/ROADMAP.md` — phase structure
- `.planning/STATE.md` — project memory

## Process

### Phase 1: Setup
1. Check if project already exists (abort if so)
2. Initialize git repo if needed
3. Detect existing code (brownfield detection)

### Phase 2: Brownfield Offer
If existing code detected and codebase not mapped, offer to map first.

### Phase 3: Deep Questioning
Ask "What do you want to build?" and follow up with probing questions:
- What excited them
- What problem sparked this
- What they mean by vague terms
- What it would actually look like
- What's already decided

Continue until you understand completely, then create PROJECT.md.

### Phase 4: Write PROJECT.md
Synthesize all context into `.planning/PROJECT.md` with:
- What This Is (2-3 sentences)
- Core Value (the ONE thing that must work)
- Requirements (Active, Validated, Out of Scope)
- Context (background information)
- Constraints
- Key Decisions

### Phase 5: Workflow Preferences
Configure:
- Mode: YOLO (auto-approve) or Interactive
- Depth: Quick, Standard, or Comprehensive
- Execution: Parallel or Sequential
- Git Tracking: Commit planning docs or keep local

### Phase 6: Research Decision
Optional domain research with 4 parallel researchers:
- Stack research
- Features research
- Architecture research
- Pitfalls research

### Phase 7: Define Requirements
Present features by category, scope each to v1/v2/out of scope.

### Phase 8: Create Roadmap
Generate phases mapped to requirements with success criteria.

### Phase 9: Done
Present completion summary with next steps: `/gsd:discuss-phase 1` or `/gsd:plan-phase 1`

## Success Criteria

- [ ] .planning/ directory created
- [ ] Git repo initialized
- [ ] Deep questioning completed
- [ ] PROJECT.md captures full context
- [ ] config.json has workflow settings
- [ ] REQUIREMENTS.md created with REQ-IDs
- [ ] ROADMAP.md created with phases
- [ ] STATE.md initialized
- [ ] User knows next step
