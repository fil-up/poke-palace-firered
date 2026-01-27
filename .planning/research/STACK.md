# Stack Research

## Standard Stack for GBA ROM Hacks (2025)

### Base Decomp
| Component | Recommendation | Rationale |
|-----------|----------------|-----------|
| **pokefirered** | ✓ Already using | Well-maintained pret decomp, active community |
| **agbcc** | ✓ Already using | Nintendo-matching compiler for accurate builds |
| **DevkitARM** | Available | Alternative for MODERN=1 builds, faster compilation |

### Build Tooling
| Tool | Purpose | Notes |
|------|---------|-------|
| **GNU Make** | Build orchestration | Standard for pret decomps |
| **Python 3** | Data processing | CSV → C table generation |
| **arm-none-eabi-gcc** | Modern compilation | Optional MODERN=1 mode |

### Data Pipeline (Recommended)
```
CSV (human-editable)
    ↓ Python validator
    ↓ Python generator
C tables (compile-time constants)
    ↓ Makefile dependency
GBA ROM
```

**Why this approach:**
- No runtime parsing (GBA has no JSON/CSV libraries)
- Validation catches errors before build
- C tables are fast and memory-efficient
- Makefile ensures regeneration on data changes

### Content Tools (Reference)
| Tool | Purpose | Status |
|------|---------|--------|
| **Porymap** | Map/encounter editing | Useful for route design |
| **Poryscript** | Event scripting | Not needed for quiz system |
| **pokeemerald-expansion** | Pattern reference | Borrow ideas, don't migrate |

## Version Recommendations

| Component | Recommended Version | Notes |
|-----------|---------------------|-------|
| Python | 3.9+ | For f-strings, typing |
| agbcc | Latest from pret/agbcc | Matches pokefirered |
| DevkitARM | r62+ | If using MODERN=1 |

## Don't Hand-Roll

| Category | Don't Build | Use Instead |
|----------|-------------|-------------|
| Text boxes | Custom text renderer | Existing `BattlePutTextOnWindow()` |
| Save data | Custom flash routines | Existing SaveBlock system |
| Input handling | Raw button reads | Existing menu systems |
| RNG | Custom random | Existing `Random()` function |

## Confidence Levels

| Recommendation | Confidence | Verified |
|----------------|------------|----------|
| Use existing SaveBlock | High | Researched save.c structure |
| CSV → C pipeline | High | Standard pattern in ROM hacks |
| Multi-page text boxes | Medium | Needs battle message system review |
| B_WIN_QUIZ window | High | Already defined in constants/battle.h |
