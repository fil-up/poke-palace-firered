# Structure

## Repository Layout

```
poke-palace-firered/
├── agbcc/                    # GBA C compiler (submodule)
├── pokefirered/              # FireRed decomp (submodule, modified)
│   ├── src/                  # C source files
│   │   ├── quiz/             # ★ Custom quiz system
│   │   │   ├── quiz.c        # Quiz state machine & UI
│   │   │   └── quiz_hooks.c  # Battle integration hooks
│   │   ├── battle_*.c        # Battle engine files
│   │   ├── wild_encounter.c  # Wild encounter logic
│   │   └── ...               # 200+ other source files
│   ├── include/              # Header files
│   │   ├── quiz/             # ★ Custom quiz headers
│   │   │   ├── quiz.h
│   │   │   └── quiz_hooks.h
│   │   └── ...
│   ├── data/                 # Game data (maps, pokemon, etc.)
│   ├── graphics/             # Sprites, tilesets, etc.
│   ├── sound/                # Music and sound effects
│   ├── constants/            # Game constants (species IDs, etc.)
│   ├── asm/                  # Assembly source files
│   ├── build/                # Build output
│   └── Makefile              # Build system
├── data/                     # ★ Custom question data
│   └── questions/
│       └── routes/           # Per-route question banks (planned)
├── tools/                    # ★ Custom build tools (planned)
├── generated/                # ★ Generated C files (planned)
├── docs/                     # Documentation
├── skills/                   # AI skill definitions
│   ├── peer-review-checklist/
│   └── get-shit-done/        # GSD workflow system
├── DESIGN.md                 # ★ Detailed gameplay spec
├── ENGINEERING_BRIEF.md      # ★ Technical architecture
├── BATTLE_SPEC.md            # ★ Battle system spec
└── README.md                 # Project overview
```

## Key Directories

### Custom Code (What We're Building)

| Path | Purpose |
|------|---------|
| `pokefirered/src/quiz/` | Core quiz logic (state, hooks, UI) |
| `pokefirered/include/quiz/` | Quiz header files |
| `data/questions/` | Question bank JSON files |
| `tools/` | Python build/validation tools |
| `generated/` | Auto-generated C tables |

### FireRed Engine (What We Hook Into)

| Path | Purpose |
|------|---------|
| `pokefirered/src/battle_*.c` | Battle system (~20 files) |
| `pokefirered/src/wild_encounter.c` | Wild encounter generation |
| `pokefirered/src/save.c` | Save/load system |
| `pokefirered/include/battle*.h` | Battle constants & types |

## File Naming Conventions

| Pattern | Meaning |
|---------|---------|
| `*.c` | C source file |
| `*.h` | C header file |
| `*.s` | ARM assembly |
| `*.json` | Data files (questions) |
| `*.md` | Documentation |
| `*_gen.c` | Auto-generated (do not edit) |

## Important Files

| File | Description |
|------|-------------|
| `pokefirered/src/quiz/quiz.c` | Main quiz state machine |
| `pokefirered/src/quiz/quiz_hooks.c` | Battle engine hooks |
| `pokefirered/src/battle_controller_player.c` | Player battle input (modified) |
| `pokefirered/src/battle_script_commands.c` | Battle script execution |
| `pokefirered/Makefile` | Build configuration |
| `DESIGN.md` | Gameplay behavior spec |
| `BATTLE_SPEC.md` | Detailed battle integration spec |
