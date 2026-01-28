# Phase ENH-D: Type-Based Moves - Research

**Researched:** 2026-01-28
**Domain:** pokefirered decomp / Pokemon move assignment
**Confidence:** HIGH

## Summary

This research investigates how to replace the vanilla level-up moveset system with type-based moves in pokefirered. The decomp provides clear hook points for move assignment: the `GiveBoxMonInitialMoveset()` function in `pokemon.c` and the existing `QuizHooks_FillEnemyMoves()` pattern in `quiz_hooks.c`.

The implementation approach is straightforward: replace or hook into `GiveBoxMonInitialMoveset()` to assign moves based on Pokemon type(s) and evolution stage rather than from `gLevelUpLearnsets`. The existing quiz hooks already demonstrate how to modify moves post-creation, providing a tested pattern to follow.

**Primary recommendation:** Create a new `type_moves.c` module that provides a `GiveTypeBasedMoveset()` function, called from a modified `GiveBoxMonInitialMoveset()` or from `QuizHooks_OnBattleInit()` to replace vanilla moves with type-based moves.

## Standard Stack

This is a decomp modification - no external libraries. Uses existing pokefirered structures.

### Core Data Structures

| Structure | Location | Purpose |
|-----------|----------|---------|
| `gSpeciesInfo[species].types[2]` | `src/data/pokemon/species_info.h` | Pokemon type lookup (type1, type2) |
| `gEvolutionTable[species][EVOS_PER_MON]` | `src/data/pokemon/evolution.h` | Evolution chain data |
| `gBattleMoves[move]` | `src/data/battle_moves.h` | Move properties (power, accuracy, PP, effect) |
| `gLevelUpLearnsets[species]` | `src/data/pokemon/level_up_learnsets.h` | Vanilla level-up moves (to be replaced) |
| `SetMonData()` / `SetMonMoveSlot()` | `src/pokemon.c` | Move assignment functions |

### Key Constants

| Constant | File | Value | Purpose |
|----------|------|-------|---------|
| `TYPE_*` | `include/constants/pokemon.h` | 0-17 | Type identifiers (TYPE_NORMAL=0 through TYPE_DARK=17) |
| `MOVE_*` | `include/constants/moves.h` | 1-354 | Move identifiers |
| `MAX_MON_MOVES` | `include/constants/pokemon.h` | 4 | Maximum moves per Pokemon |
| `NUM_SPECIES` | `include/constants/species.h` | 412 | Total species count |
| `EVOS_PER_MON` | Defined in code | ~5 | Max evolutions per species |

## Architecture Patterns

### Recommended Project Structure

```
pokefirered/src/
├── quiz/
│   ├── quiz.c                    # Existing quiz logic
│   ├── quiz_hooks.c              # Existing battle hooks (modify here)
│   └── type_moves.c              # NEW: Type-based move assignment
│   └── type_moves.h              # NEW: Header for type moves
```

### Pattern 1: Move Assignment via Quiz Hooks (Recommended)

**What:** Modify moves in `QuizHooks_OnBattleInit()` after vanilla `CreateMon()` runs

**When to use:** This is the cleanest approach - no core engine modifications needed

**Why this works:** The existing `QuizHooks_FillEnemyMoves()` already demonstrates post-creation move replacement. Type-based moves follow the same pattern.

**Example:**

```c
// In quiz_hooks.c (after includes)
#include "quiz/type_moves.h"

void QuizHooks_OnBattleInit(void)
{
    // Reset quiz state at the start of each new battle
    Quiz_ResetBattleState();
    
    // NEW: Replace moves with type-based moves for all party Pokemon
    TypeMoves_AssignToParty(gPlayerParty, PARTY_SIZE);
    TypeMoves_AssignToParty(gEnemyParty, PARTY_SIZE);
}
```

### Pattern 2: Evolution Stage Detection

**What:** Determine if species is Basic, Stage 1, Stage 2, or Legendary

**Implementation approach:** Walk the evolution table to determine stage:

```c
// Returns: 0 = Basic, 1 = Stage 1, 2 = Stage 2/Legendary
u8 GetEvolutionStage(u16 species)
{
    // Check if legendary (hardcoded list)
    if (IsLegendarySpecies(species))
        return 2;  // Strong tier
    
    // Check if anything evolves INTO this species (it's not basic)
    u16 preEvo = GetPreEvolutionSpecies(species);
    if (preEvo == SPECIES_NONE)
        return 0;  // Basic - nothing evolves into this
    
    // Check if the pre-evolution also has a pre-evolution
    if (GetPreEvolutionSpecies(preEvo) != SPECIES_NONE)
        return 2;  // Stage 2 - two pre-evolutions exist
    
    return 1;  // Stage 1 - one pre-evolution
}
```

### Pattern 3: Type-to-Moves Mapping

**What:** Static arrays mapping types to their tier moves

```c
// Move tiers per type (indexed by TYPE_* constants)
static const u16 sTypeMoves[NUMBER_OF_MON_TYPES][3][2] = {
    [TYPE_NORMAL] = {
        { MOVE_TACKLE, MOVE_SCRATCH },        // Weak
        { MOVE_BODY_SLAM, MOVE_STRENGTH },    // Medium
        { MOVE_DOUBLE_EDGE, MOVE_HYPER_BEAM } // Strong
    },
    [TYPE_FIRE] = {
        { MOVE_EMBER, MOVE_FIRE_SPIN },
        { MOVE_FLAME_WHEEL, MOVE_FIRE_PUNCH },
        { MOVE_FLAMETHROWER, MOVE_FIRE_BLAST }
    },
    // ... etc for all 18 types
};
```

### Anti-Patterns to Avoid

- **Modifying CreateMon directly:** Would affect all Pokemon creation including trades, eggs, etc. Use hooks instead.
- **Hardcoding species-specific logic:** Use data-driven approach with type/evolution tables.
- **Ignoring existing BUG fixes:** The quiz hooks already handle status moves - don't duplicate.

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Type lookup | Manual species switch | `gSpeciesInfo[species].types[0/1]` | Canonical source of truth |
| Move properties | Hardcoded power values | `gBattleMoves[move]` | Already has power/accuracy/PP |
| Evolution data | Manual family trees | `gEvolutionTable` | Complete, maintained |
| Move assignment | Direct memory writes | `SetMonData()` / `SetMonMoveSlot()` | Handles encryption |

**Key insight:** The decomp provides complete data tables for species, moves, and evolution. Use them directly rather than recreating.

## Common Pitfalls

### Pitfall 1: Forgetting PP Assignment

**What goes wrong:** Setting moves without setting PP leaves Pokemon with 0 PP
**Why it happens:** `SetMonMoveSlot()` handles this, but direct `SetMonData(MON_DATA_MOVE*)` doesn't
**How to avoid:** Always use `SetMonMoveSlot()` or set both `MON_DATA_MOVEn` AND `MON_DATA_PPn`
**Warning signs:** Moves show but can't be used in battle

### Pitfall 2: Evolution Table Lookup Direction

**What goes wrong:** Looking for "what does X evolve into" when you need "what evolves into X"
**Why it happens:** `gEvolutionTable[species]` shows targets, not sources
**How to avoid:** Write a reverse lookup function `GetPreEvolutionSpecies()`
**Warning signs:** Stage 2 Pokemon wrongly classified as Basic

### Pitfall 3: Single-Type Pokemon Detection

**What goes wrong:** Treating `types[0] == types[1]` incorrectly
**Why it happens:** Single-type Pokemon have same type in both slots (e.g., Charmander is `{TYPE_FIRE, TYPE_FIRE}`)
**How to avoid:** Check `types[0] == types[1]` to detect single-type, then add random filler moves
**Warning signs:** Single-type Pokemon get duplicate moves

### Pitfall 4: Move Availability in Gen 3

**What goes wrong:** Trying to use moves that don't exist in FireRed
**Why it happens:** Some moves in CONTEXT.md may be from later generations
**How to avoid:** Verify all moves exist in `include/constants/moves.h` before adding to type tables
**Warning signs:** Compile errors or invalid move IDs

### Pitfall 5: Interaction with TM/HM Learning

**What goes wrong:** Type-based moves get overwritten when learning TMs
**Why it happens:** TM learning replaces moves normally
**How to avoid:** This is intended behavior per CONTEXT.md - TMs/HMs should still work
**Warning signs:** None - this is expected

### Pitfall 6: Trainer Pokemon Custom Movesets

**What goes wrong:** Trainer-defined movesets get replaced with type-based moves
**Why it happens:** `QuizHooks_OnBattleInit` runs after `CreateTrainerParty`
**How to avoid:** Per CONTEXT.md, all Pokemon should use type-based moves, so this is correct
**Warning signs:** Gym leaders don't have their signature movesets (intended)

## Code Examples

### Move Assignment Function

```c
// Source: Based on existing pattern in quiz_hooks.c
void TypeMoves_AssignToMon(struct Pokemon *mon)
{
    u16 species = GetMonData(mon, MON_DATA_SPECIES, NULL);
    if (species == SPECIES_NONE)
        return;
    
    u8 type1 = gSpeciesInfo[species].types[0];
    u8 type2 = gSpeciesInfo[species].types[1];
    u8 stage = GetEvolutionStage(species);  // 0=Basic, 1=Stage1, 2=Stage2/Legend
    
    u16 move;
    u8 pp;
    
    // Slot 0-1: Primary type moves
    move = sTypeMoves[type1][stage][0];
    SetMonMoveSlot(mon, move, 0);
    
    move = sTypeMoves[type1][stage][1];
    SetMonMoveSlot(mon, move, 1);
    
    if (type1 != type2)
    {
        // Dual-type: slots 2-3 from secondary type
        move = sTypeMoves[type2][stage][0];
        SetMonMoveSlot(mon, move, 2);
        
        move = sTypeMoves[type2][stage][1];
        SetMonMoveSlot(mon, move, 3);
    }
    else
    {
        // Single-type: slots 2-3 random from any type
        u8 randType1 = Random() % NUMBER_OF_MON_TYPES;
        u8 randType2 = Random() % NUMBER_OF_MON_TYPES;
        
        move = sTypeMoves[randType1][stage][Random() % 2];
        SetMonMoveSlot(mon, move, 2);
        
        move = sTypeMoves[randType2][stage][Random() % 2];
        SetMonMoveSlot(mon, move, 3);
    }
}
```

### Evolution Stage Detection

```c
// Reverse lookup: find what species evolves INTO this one
static u16 GetPreEvolutionSpecies(u16 species)
{
    u16 i, j;
    for (i = 1; i < NUM_SPECIES; i++)
    {
        for (j = 0; j < EVOS_PER_MON; j++)
        {
            if (gEvolutionTable[i][j].targetSpecies == species)
                return i;
        }
    }
    return SPECIES_NONE;
}

// Legendary detection (hardcoded list)
static bool8 IsLegendarySpecies(u16 species)
{
    switch (species)
    {
    case SPECIES_ARTICUNO:
    case SPECIES_ZAPDOS:
    case SPECIES_MOLTRES:
    case SPECIES_MEWTWO:
    case SPECIES_MEW:
    case SPECIES_RAIKOU:
    case SPECIES_ENTEI:
    case SPECIES_SUICUNE:
    case SPECIES_LUGIA:
    case SPECIES_HO_OH:
    case SPECIES_CELEBI:
    case SPECIES_REGIROCK:
    case SPECIES_REGICE:
    case SPECIES_REGISTEEL:
    case SPECIES_LATIAS:
    case SPECIES_LATIOS:
    case SPECIES_KYOGRE:
    case SPECIES_GROUDON:
    case SPECIES_RAYQUAZA:
    case SPECIES_JIRACHI:
    case SPECIES_DEOXYS:
        return TRUE;
    }
    return FALSE;
}
```

### Move Availability Verification

All moves from CONTEXT.md verified against `include/constants/moves.h`:

| Type | Weak 1 | Weak 2 | Medium 1 | Medium 2 | Strong 1 | Strong 2 |
|------|--------|--------|----------|----------|----------|----------|
| Normal | MOVE_TACKLE ✓ | MOVE_SCRATCH ✓ | MOVE_BODY_SLAM ✓ | MOVE_STRENGTH ✓ | MOVE_DOUBLE_EDGE ✓ | MOVE_HYPER_BEAM ✓ |
| Fire | MOVE_EMBER ✓ | MOVE_FIRE_SPIN ✓ | MOVE_FLAME_WHEEL ✓ | MOVE_FIRE_PUNCH ✓ | MOVE_FLAMETHROWER ✓ | MOVE_FIRE_BLAST ✓ |
| Water | MOVE_WATER_GUN ✓ | MOVE_BUBBLE ✓ | MOVE_SURF ✓ | MOVE_WATER_PULSE ✓ | MOVE_HYDRO_PUMP ✓ | MOVE_WATER_SPOUT ✓ |
| Electric | MOVE_THUNDER_SHOCK ✓ | MOVE_SPARK ✓ | MOVE_THUNDERBOLT ✓ | MOVE_SHOCK_WAVE ✓ | MOVE_THUNDER ✓ | MOVE_VOLT_TACKLE ✓ |
| Grass | MOVE_ABSORB ✓ | MOVE_VINE_WHIP ✓ | MOVE_RAZOR_LEAF ✓ | MOVE_GIGA_DRAIN ✓ | MOVE_SOLAR_BEAM ✓ | MOVE_FRENZY_PLANT ✓ |
| Ice | MOVE_POWDER_SNOW ✓ | MOVE_ICY_WIND ✓ | MOVE_ICE_BEAM ✓ | MOVE_AURORA_BEAM ✓ | MOVE_BLIZZARD ✓ | MOVE_SHEER_COLD ✓ |
| Fighting | MOVE_KARATE_CHOP ✓ | MOVE_ROCK_SMASH ✓ | MOVE_BRICK_BREAK ✓ | MOVE_VITAL_THROW ✓ | MOVE_CROSS_CHOP ✓ | MOVE_SKY_UPPERCUT ✓ |
| Poison | MOVE_POISON_STING ✓ | MOVE_SMOG ✓ | MOVE_SLUDGE ✓ | MOVE_ACID ✓ | MOVE_SLUDGE_BOMB ✓ | MOVE_POISON_FANG ✓ |
| Ground | MOVE_MUD_SLAP ✓ | MOVE_MUD_SHOT ✓ | MOVE_DIG ✓ | MOVE_BONE_RUSH ✓ | MOVE_EARTHQUAKE ✓ | MOVE_FISSURE ✓ |
| Flying | MOVE_PECK ✓ | MOVE_GUST ✓ | MOVE_WING_ATTACK ✓ | MOVE_AERIAL_ACE ✓ | MOVE_DRILL_PECK ✓ | MOVE_SKY_ATTACK ✓ |
| Psychic | MOVE_CONFUSION ✓ | MOVE_PSYBEAM ✓ | MOVE_PSYCHIC ✓ | MOVE_EXTRASENSORY ✓ | MOVE_PSYCHIC* | MOVE_FUTURE_SIGHT ✓ |
| Bug | MOVE_LEECH_LIFE ✓ | MOVE_FURY_CUTTER ✓ | MOVE_SIGNAL_BEAM ✓ | MOVE_SILVER_WIND ✓ | MOVE_MEGAHORN ✓ | MOVE_SIGNAL_BEAM* |
| Rock | MOVE_ROCK_THROW ✓ | MOVE_ROLLOUT ✓ | MOVE_ROCK_SLIDE ✓ | MOVE_ANCIENT_POWER ✓ | MOVE_ROCK_SLIDE* | MOVE_ROCK_BLAST ✓ |
| Ghost | MOVE_LICK ✓ | MOVE_NIGHT_SHADE ✓ | MOVE_SHADOW_BALL ✓ | MOVE_SHADOW_PUNCH ✓ | MOVE_SHADOW_BALL* | MOVE_SHADOW_PUNCH* |
| Dragon | MOVE_TWISTER ✓ | MOVE_DRAGON_BREATH ✓ | MOVE_DRAGON_CLAW ✓ | MOVE_DRAGON_RAGE ✓ | MOVE_OUTRAGE ✓ | MOVE_DRAGON_CLAW* |
| Dark | MOVE_PURSUIT ✓ | MOVE_THIEF ✓ | MOVE_BITE ✓ | MOVE_FAINT_ATTACK ✓ | MOVE_CRUNCH ✓ | MOVE_FAINT_ATTACK* |
| Steel | MOVE_METAL_CLAW ✓ | MOVE_STEEL_WING ✓ | MOVE_IRON_TAIL ✓ | MOVE_METAL_CLAW* | MOVE_METEOR_MASH ✓ | MOVE_IRON_TAIL* |

*Note: Some types have repeated moves across tiers in CONTEXT.md - this is intentional per user decisions.

**STATUS MOVES TO WATCH:** Several listed moves have secondary effects or special behavior:
- NIGHT_SHADE: Fixed damage (user's level) - needs damage override
- DRAGON_RAGE: Fixed 40 damage - needs damage override
- FISSURE/SHEER_COLD: OHKO moves - needs effect override
- DIG/SKY_ATTACK: Two-turn moves - may need consideration

Per CONTEXT.md, all moves deal flat 1000 damage with quiz damage override, so these effects are irrelevant.

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| Level-up movesets | Type-based movesets | This phase | All Pokemon get moves by type |
| Vanilla trainer AI | Random move selection | Already done | Trainers choose randomly |
| Status move filtering | Quiz hooks fill moves | BUG-003 fix | Non-damage moves replaced |
| Secondary effects | Disabled in quiz mode | BUG-004 fix | No status/flinch effects |

**Deprecated/outdated:**
- `GiveBoxMonInitialMoveset()` vanilla behavior: Will be overridden by type-based system
- Trainer custom movesets: Still in data but overwritten at battle start

## Open Questions

### 1. Hook Point: GiveBoxMonInitialMoveset vs QuizHooks

**What we know:** Both locations work. `GiveBoxMonInitialMoveset` runs during creation, `QuizHooks_OnBattleInit` runs at battle start.

**What's unclear:** Which is cleaner architecturally?

**Recommendation:** Use `QuizHooks_OnBattleInit` because:
- Doesn't modify core pokemon.c
- Keeps all quiz-related code in quiz directory
- Already has pattern for move replacement
- Runs AFTER trainer parties are created (correct timing)

### 2. Random Filler for Single-Type

**What we know:** Single-type Pokemon need 2 random moves from other types

**What's unclear:** Should randomness be seeded per encounter or fixed?

**Recommendation:** Use `Random()` per encounter for variety (per CONTEXT.md "randomized per encounter")

### 3. MYSTERY Type (TYPE_MYSTERY = 9)

**What we know:** This is a placeholder type (used for ??? type in some games)

**What's unclear:** No moves listed for it

**Recommendation:** Skip this type in the move tables - no Pokemon have it

## Sources

### Primary (HIGH confidence)
- `pokefirered/src/pokemon.c` lines 1755-1862 - CreateMon/CreateBoxMon/GiveBoxMonInitialMoveset
- `pokefirered/src/quiz/quiz_hooks.c` - Existing move replacement pattern
- `pokefirered/src/data/pokemon/species_info.h` - Species type data
- `pokefirered/src/data/pokemon/evolution.h` - Evolution table
- `pokefirered/include/constants/moves.h` - Move constants verification
- `pokefirered/include/constants/pokemon.h` - Type constants

### Secondary (MEDIUM confidence)
- `pokefirered/src/battle_main.c` lines 1550-1637 - CreateTrainerParty pattern

### Tertiary (LOW confidence)
- None - all findings verified against source code

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH - Direct code analysis of decomp
- Architecture: HIGH - Follows existing quiz_hooks pattern
- Pitfalls: HIGH - Based on known decomp behavior
- Move availability: HIGH - Verified against moves.h

**Research date:** 2026-01-28
**Valid until:** Indefinite (decomp is stable)

---

## Implementation Checklist for Planner

1. [ ] Create `type_moves.c` and `type_moves.h` in `pokefirered/src/quiz/`
2. [ ] Define `sTypeMoves[18][3][2]` static array with all 18 types × 3 tiers × 2 moves
3. [ ] Implement `GetEvolutionStage(species)` with reverse evolution lookup
4. [ ] Implement `IsLegendarySpecies(species)` with hardcoded list
5. [ ] Implement `TypeMoves_AssignToMon(struct Pokemon *mon)`
6. [ ] Implement `TypeMoves_AssignToParty(struct Pokemon *party, u8 count)`
7. [ ] Hook into `QuizHooks_OnBattleInit()` to call assignment functions
8. [ ] Verify all moves in tables are damage-dealing (power > 0)
9. [ ] Test with single-type and dual-type Pokemon
10. [ ] Test with Basic, Stage 1, Stage 2, and Legendary Pokemon
