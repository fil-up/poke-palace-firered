# External Integrations

**Analysis Date:** 2026-01-30

## Overview

This is a GBA ROM hack project. The target platform (Game Boy Advance) has no network connectivity and runs entirely offline. All "integrations" are either:
1. Build-time dependencies (submodules, toolchains)
2. CI/CD automation (GitHub Actions)
3. Data sources processed at build time (CSV files)

**No runtime external APIs, databases, or services exist.**

## Git Submodules

### pret/pokefirered (Base Decomp)

| Aspect | Details |
|--------|---------|
| **Source** | https://github.com/pret/pokefirered |
| **Type** | Git submodule |
| **Location** | `/pokefirered/` |
| **Purpose** | Complete FireRed decompilation as project base |
| **Version** | Regularly synced with upstream |

**What it provides:**
- Full GBA ROM source code (~4500+ C files)
- Battle engine, overworld, menus, save system
- Graphics, audio, map data
- Build system and tooling

**Modifications for quiz system:**
- Battle controller hooks for quiz mode
- Custom battle window (B_WIN_QUIZ)
- Damage calculation overrides
- Encounter filtering for cleared species

### pret/agbcc (GBA C Compiler)

| Aspect | Details |
|--------|---------|
| **Source** | https://github.com/pret/agbcc |
| **Type** | Git submodule |
| **Location** | `/agbcc/` |
| **Purpose** | Nintendo-matching C compiler for GBA |

**Installation:**
```bash
cd agbcc
./build.sh
./install.sh ../pokefirered
```

**Installed to:** `pokefirered/tools/agbcc/`

**Components:**
- `agbcc` - Main C compiler
- `old_agbcc` - Legacy variant
- `agbcc_arm` - ARM-mode variant
- `libgcc.a`, `libc.a` - Runtime libraries
- `include/` - C standard library headers

## CI/CD & Automation

### GitHub Actions: AI PR Review

| Aspect | Details |
|--------|---------|
| **Workflow** | `.github/workflows/ai-pr-review.yml` |
| **Trigger** | Pull request (opened, reopened, synchronize, ready_for_review) |
| **Action** | `erans/autoagent-action@v1` |
| **Config** | `.autoagent/config.yml` |

**Required Secrets:**
- `GITHUB_TOKEN` - Automatic, for PR comments
- `OPENAI_API_KEY` - For AI review generation

**Purpose:** Automated code review on pull requests using OpenAI

## Data Sources (Build-Time)

### Question Bank CSV

| Aspect | Details |
|--------|---------|
| **File** | `GH 101 Question Bank - Sheet1.csv` |
| **Format** | CSV with 9 columns |
| **Size** | ~600+ rows |
| **Processing** | Converted to C tables at build time |

**Schema:**
| Column | Purpose |
|--------|---------|
| `Question_ID` | Unique identifier (e.g., MQ.3.001) |
| `Broad_Category` | Topic category |
| `Sub_Topic` | Specific subtopic |
| `Question_String` | Question text |
| `Option_Label` | Answer choice (A/B/C/D) |
| `Option_Text` | Answer text |
| `Correct_Answer` | Correct option label |
| `Difficulty_Level` | 1-5 difficulty scale |
| `Explanation` | Answer explanation |

**Processing Pipeline (Planned):**
```
CSV file
    ↓ validate_questions.py
    ↓   - Schema validation
    ↓   - Length checks (prompt ≤120, answer ≤40)
    ↓   - Required field verification
    ↓ build_questions.py  
    ↓   - Group by species/area
    ↓   - Generate C structs
    ↓   - Output header + source
questions_gen.h / questions_gen.c
    ↓ Makefile (include in build)
pokefirered.gba
```

## Battle Engine Integration Points

### Current Hooks (Implemented)

| Hook Point | File | Function |
|------------|------|----------|
| Wild battle start | `battle_setup.c` | `QuizHooks_OnWildBattleStart()` |
| Command menu shown | `battle_controller_player.c` | `QuizHooks_OnCommandMenuShown()` |
| Move cursor changed | `battle_controller_player.c` | `QuizHooks_OnMoveCursorChanged()` |
| Move confirmed | `battle_controller_player.c` | `QuizHooks_OnMoveConfirmed()` |
| Damage calculation | `battle_script_commands.c` | `QuizHooks_ApplyDamageOverride()` |
| Turn order | `battle_main.c` | `QuizHooks_ShouldForcePlayerFirst()` |
| Move effect | Damage calc | `QuizHooks_GetMoveEffectOverride()` |
| Accuracy check | Battle util | `QuizHooks_ShouldAlwaysHit()` |
| PP check | Battle util | `QuizHooks_ShouldInfinitePp()` |

### Planned Hooks

| Hook Point | File | Purpose |
|------------|------|---------|
| Trainer send-out | `battle_controller_opponent.c` | Reinforcement quiz |
| Gym battle start | `battle_setup.c` | Section exam mode |
| Save game | `save.c` | Persist quiz progress |
| Encounter generation | `wild_encounter.c` | Skip cleared species |

## UI Integration

### Battle Windows (constants/battle.h)

| Window | Usage |
|--------|-------|
| `B_WIN_QUIZ` | Quiz question display (custom) |
| `B_WIN_MSG` | Standard battle messages |
| `B_WIN_ACTION_MENU` | FIGHT/BAG/POKEMON/RUN |
| `B_WIN_MOVE_MENU` | Move selection = answer selection |

## External Tools (Reference Only)

These tools are mentioned in pret documentation but not integrated:

| Tool | Purpose | Status |
|------|---------|--------|
| **Porymap** | Visual map/encounter editing | Not integrated |
| **Poryscript** | Scripted events | Not integrated |
| **mGBA Lua** | Automated testing | Not integrated |

## Environment Configuration

### Required Environment Variables

| Variable | Purpose | Required |
|----------|---------|----------|
| `DEVKITARM` | Path to devkitARM toolchain | Optional (MODERN=1) |
| `DEVKITPRO` | Path to devkitPro root | Optional (MODERN=1) |

### GitHub Secrets (CI/CD)

| Secret | Purpose |
|--------|---------|
| `OPENAI_API_KEY` | AI PR review |
| `GITHUB_TOKEN` | PR comments (automatic) |

## No Runtime Dependencies

**Explicit non-integrations:**
- No network/API calls (GBA has no networking)
- No database connections (data baked into ROM)
- No auth providers (offline game)
- No webhooks (ROM, not server)
- No external file I/O (cartridge ROM is read-only)
- No real-time services (standalone game)

All data is compiled into the ROM at build time. Save data uses GBA's onboard flash memory.

---

*Integration audit: 2026-01-30*
