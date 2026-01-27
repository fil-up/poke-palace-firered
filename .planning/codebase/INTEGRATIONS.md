# Integrations

## External Dependencies

### 1. pret/pokefirered (Base Decomp)
| Aspect | Details |
|--------|---------|
| **Source** | https://github.com/pret/pokefirered |
| **Type** | Git submodule / fork |
| **Version** | Latest (regularly updated) |
| **Purpose** | Complete FireRed decompilation as base |
| **Modifications** | Battle controller hooks, quiz window |

### 2. agbcc (GBA C Compiler)
| Aspect | Details |
|--------|---------|
| **Source** | https://github.com/pret/agbcc |
| **Type** | Git submodule |
| **Purpose** | Nintendo-matching C compiler for GBA |
| **Notes** | Required for MODERN=0 builds |

## Battle Engine Integration Points

### Current Hooks (Implemented)

| Hook Point | File | Function Called |
|------------|------|-----------------|
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

### Battle Windows (B_WIN_*)
| Window | Usage |
|--------|-------|
| `B_WIN_QUIZ` | Quiz question display (custom) |
| `B_WIN_MSG` | Standard battle messages |
| `B_WIN_ACTION_MENU` | FIGHT/BAG/POKEMON/RUN |
| `B_WIN_MOVE_MENU` | Move selection (answer selection) |

## Data Pipeline (Planned)

```
data/questions/*.json
        │
        ▼
tools/validate_questions.py  ──> Errors if invalid
        │
        ▼
tools/build_questions.py
        │
        ▼
generated/questions_gen.{h,c}  ──> Included in ROM build
```

## GitHub Integration

### AI PR Review
| Component | Purpose |
|-----------|---------|
| `.github/workflows/ai-pr-review.yml` | AutoAgent review workflow |
| `.autoagent/config.yml` | Review prompts and filters |
| `OPENAI_API_KEY` secret | API access |

## Future Integrations (Considered)

| Tool | Purpose | Priority |
|------|---------|----------|
| **Porymap** | Visual map/encounter editing | Medium |
| **Poryscript** | Scripted events | Low |
| **mGBA Lua** | Automated testing | Low |
