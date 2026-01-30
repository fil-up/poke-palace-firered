# Phase 12: Scalable Battle System (Stretch) - Research

**Researched:** 2026-01-29  
**Domain:** Pokémon FireRed decompilation - trainer battle quiz scaling  
**Confidence:** HIGH

## Summary

This phase extends the existing quiz battle system so trainer-facing encounters ask a fixed, scalable number of questions per enemy Pokemon, with a last-Pokemon bonus and evolution-based modifiers. The current implementation already distinguishes trainer vs wild encounters and supports multi-question trainer battles, but it hardcodes 3 questions for normal trainer Pokemon and 5 for the final Pokemon, and it only applies to specific trainer classes. The scalable rules need to replace those hardcoded counts and expand applicability to gym trainers and special encounters (e.g., Ghost Marowak), while explicitly excluding wild encounters.

The codebase already provides stable integration points: trainer quiz initialization happens in `battle_main.c` when the player chooses an action (trainer battles), and wild initialization happens earlier. Trainer class and party size are read from `gTrainers[]`, while battle type flags (e.g., `BATTLE_TYPE_TRAINER`, `BATTLE_TYPE_GHOST`) identify special encounter modes. Evolution data is available via `gEvolutionTable`, but there is no built-in helper for evolution stage, evolution-line length, or "legendary" tagging, so those need to be derived from existing tables or a small explicit list.

**Primary recommendation:** Implement a single "trainer question count" calculator in `quiz/quiz.c` that runs at trainer quiz initialization, using `gTrainers[gTrainerBattleOpponent_A]` for trainer class/party size, `gEvolutionTable` for stage/line length, and `gBattlerPartyIndexes` for accurate last-Pokemon detection; extend the trainer-special check to include `BATTLE_TYPE_GHOST` (and other scripted specials if desired) while keeping wild encounters on the existing wild path.

## Standard Stack

The established systems/tools for this domain:

### Core
| System | Purpose | Why Standard |
|--------|---------|--------------|
| `pokefirered` decomp | Base engine and battle flow | Project foundation |
| `quiz/quiz.c` | Quiz state, question selection, trainer logic | Central quiz system |
| `quiz/quiz_hooks.c` | Battle engine integration | All battle overrides funnel here |
| `battle_main.c` | Trainer quiz initialization | Trainer battle hook point |
| `include/constants/battle.h` | Battle type flags | Identifies trainer/special encounters |
| `src/data/pokemon/evolution.h` | Evolution graph | Source of evolution-line data |
| `include/battle.h` | `struct Trainer` + party size/class | Trainer metadata for scaling |

### Supporting
| System | Purpose | When to Use |
|--------|---------|-------------|
| `battle_setup.c` | Wild/special battle startup | Special encounters (ghost battle setup) |
| `Random()` | Question selection | Use for trainer pool picks |
| CSV → Python → C pipeline | Question data generation | Existing question bank workflow |

## Architecture Patterns

### Recommended Project Structure

```
pokefirered/src/
├── quiz/quiz.c                # Trainer scaling + question count calculator
├── quiz/quiz_hooks.c          # Continue using existing hooks
├── battle_main.c              # Trainer battle init hook (already in place)
└── data/pokemon/evolution.h   # Evolution data used for stage/line length
```

### Pattern 1: Trainer quiz initialization hook
**What:** Trainer battles initialize quiz state when the player selects an action.  
**When to use:** Set per-Pokemon question totals and trainer mode flags.  
**Example:**
```
// Source: pokefirered/src/battle_main.c:3175-3186
// ... existing code ...
if (!(gBattleTypeFlags & BATTLE_TYPE_SAFARI)
 && (gBattleTypeFlags & BATTLE_TYPE_TRAINER)
 && (GET_BATTLER_POSITION(gActiveBattler) & BIT_SIDE) == B_SIDE_PLAYER)
{
    u8 opponentBattler = GetBattlerAtPosition(B_POSITION_OPPONENT_LEFT);
    QuizHooks_OnWildBattleStart(gBattleMons[opponentBattler].species,
                                gSaveBlock1Ptr->location.mapGroup,
                                gSaveBlock1Ptr->location.mapNum);
}
```

### Pattern 2: Multi-question trainer mode in quiz state
**What:** Trainer multi-question logic is centralized in `Quiz_InitWildEncounter`.  
**When to use:** Replace fixed 3/5 counts with scalable rules and last-Pokemon bonus.  
**Example:**
```
// Source: pokefirered/src/quiz/quiz.c:346-389
// ... existing code ...
if (Quiz_IsMultiQuestionTrainerClass(trainerClass))
{
    sQuizState.isGymLeader = TRUE;
    // ...
    if (sQuizState.currentEnemyPartyIndex >= trainerPartySize - 1)
        sQuizState.gymQuestionsRequired = 5;
    else
        sQuizState.gymQuestionsRequired = 3;
    sQuizState.gymQuestionsAnswered = 0;
    // ...
}
```

### Pattern 3: Evolution-line derivation from `gEvolutionTable`
**What:** Evolution data is stored in `gEvolutionTable[species][EVOS_PER_MON]`.  
**When to use:** Determine stage (basic/stage1/stage2) and line length (1,2,3+).  
**Key insight:** There is no built-in helper for stage/line length; you must scan:
- Forward: does the species evolve (entries in `gEvolutionTable[species]`)?
- Backward: does any species evolve into this species (reverse lookup)?

### Anti-Patterns to Avoid
- **Hardcoding question counts in multiple places:** Keep the count logic in one calculator, used by quiz state.
- **Mixing trainer logic into battle engine:** Battle files should only call quiz hooks; logic belongs in `quiz/`.
- **Using species matching to detect party index:** Duplicate species in trainer parties can misidentify "last Pokemon."

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Trainer classification | New battle scripts | `gTrainers[gTrainerBattleOpponent_A].trainerClass` | Existing trainer metadata |
| Battle type detection | Custom flags | `gBattleTypeFlags` in `constants/battle.h` | Standard battle mode flags |
| Question display/UI | Custom text system | `BattlePutTextOnWindow()` + `B_WIN_QUIZ` | Existing battle UI pipeline |
| Evolution data | Custom tables | `gEvolutionTable` | Canonical evolution graph |

**Key insight:** This phase should reuse existing quiz state and hook points; the scalable logic is purely a calculation and should not require new battle engine flows.

## Common Pitfalls

### Pitfall 1: Misclassifying gym trainers
**What goes wrong:** Gym trainers use standard trainer classes (e.g., Black Belt), so class-based checks miss them.  
**Why it happens:** There is no built-in "gym trainer" flag in `struct Trainer`.  
**How to avoid:** Add an explicit trainer-ID table or map-based lookup for gym trainers.  
**Warning signs:** Gym trainers still ask only 1 question despite rules.

### Pitfall 2: Incorrect last-Pokemon detection
**What goes wrong:** The current code finds the enemy index by matching species, which fails for duplicate species.  
**Why it happens:** Trainer parties can contain repeated species.  
**How to avoid:** Use `gBattlerPartyIndexes[opponentBattler]` to get the current party slot, and compare against `trainerPartySize - 1`.  
**Warning signs:** "Last Pokemon bonus" triggers on the wrong mon.

### Pitfall 3: Special encounters treated as wild
**What goes wrong:** Ghost battles (`BATTLE_TYPE_GHOST`) follow wild logic and ignore trainer scaling rules.  
**Why it happens:** Current trainer path is gated by `BATTLE_TYPE_TRAINER`.  
**How to avoid:** Extend the trainer/special check in quiz init to include `BATTLE_TYPE_GHOST` (and any other designated specials).  
**Warning signs:** Ghost Marowak shows wild quiz flow instead of trainer scaling.

### Pitfall 4: Missing legendary detection
**What goes wrong:** Legendary override never triggers.  
**Why it happens:** There is no built-in `IsLegendarySpecies()` helper.  
**How to avoid:** Create an explicit legendary species list or a helper based on existing data (e.g., egg group), and document it.  
**Warning signs:** Legendary trainer Pokemon only get base+mod questions.

## Code Examples

### Trainer multi-question setup (current behavior)
```
// Source: pokefirered/src/quiz/quiz.c:346-389
// ... existing code ...
if (Quiz_IsMultiQuestionTrainerClass(trainerClass))
{
    sQuizState.isGymLeader = TRUE;
    // Build pool...
    if (sQuizState.currentEnemyPartyIndex >= trainerPartySize - 1)
        sQuizState.gymQuestionsRequired = 5;
    else
        sQuizState.gymQuestionsRequired = 3;
    sQuizState.gymQuestionsAnswered = 0;
}
```

### Trainer battle init hook (current behavior)
```
// Source: pokefirered/src/battle_main.c:3175-3186
if (!(gBattleTypeFlags & BATTLE_TYPE_SAFARI)
 && (gBattleTypeFlags & BATTLE_TYPE_TRAINER)
 && (GET_BATTLER_POSITION(gActiveBattler) & BIT_SIDE) == B_SIDE_PLAYER)
{
    u8 opponentBattler = GetBattlerAtPosition(B_POSITION_OPPONENT_LEFT);
    QuizHooks_OnWildBattleStart(gBattleMons[opponentBattler].species,
                                gSaveBlock1Ptr->location.mapGroup,
                                gSaveBlock1Ptr->location.mapNum);
}
```

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| Fixed trainer question counts | Trainer multi-question mode (3/5) for specific classes | Present in `quiz.c` now | Not scalable; limited to leaders/rivals/boss |

**Deprecated/outdated:**
- Hardcoded 3/5 counts for trainer battles need replacement with scalable rule set.

## Open Questions

1. **How to identify gym trainers (non-leaders)?**
   - What we know: Trainer classes do not uniquely identify gym trainers.
   - What's unclear: Preferred data source (trainer ID list vs map-based flag).
   - Recommendation: Add a small explicit trainer-ID list for gym trainers.

2. **How to tag legendary species for trainer battles?**
   - What we know: No built-in legendary flag or helper exists.
   - What's unclear: Preferred source of truth (explicit list vs egg group heuristic).
   - Recommendation: Use an explicit list for correctness.

3. **Which "special encounters" beyond Ghost Marowak should use trainer scaling?**
   - What we know: `BATTLE_TYPE_GHOST` exists and is used for Ghost Marowak.
   - What's unclear: Whether other scripted wild battles should also use trainer scaling.
   - Recommendation: Define a small set of allowed special battle types in quiz init.

## Sources

### Primary (HIGH confidence)
- `pokefirered/src/quiz/quiz.c` - trainer multi-question logic and current counts
- `pokefirered/src/battle_main.c` - trainer quiz initialization hook
- `pokefirered/include/constants/battle.h` - battle type flags (trainer/ghost)
- `pokefirered/src/data/pokemon/evolution.h` - evolution table data
- `pokefirered/include/battle.h` - `struct Trainer` metadata

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH - established project systems
- Architecture: HIGH - verified in current code
- Pitfalls: MEDIUM - gym trainer + legendary detection require new data

**Research date:** 2026-01-29  
**Valid until:** 2026-02-28
