# Phase 13: Uniform Damage/Accuracy, Cosmetic Animations - Research

**Researched:** 2026-01-30  
**Domain:** Pokefirered battle engine hooks (C)
**Confidence:** MEDIUM

## Summary

This phase can be implemented by extending the existing quiz battle hooks already wired into the battle engine. The repo already overrides accuracy, PP deduction, move effects, type immunities, and damage via `QuizHooks_*` functions in `battle_script_commands.c` and `quiz/quiz_hooks.c`. These are the primary integration points for uniform damage/accuracy behavior.

Key gaps relative to Phase 13 decisions: critical hits and STAB are still applied in the standard flow, type effectiveness still modifies damage outside immunities, and non-damage moves are currently replaced with filler moves (conflicting with the requirement to keep all moves selectable). Move animations target `gBattlerTarget` as-is; for cosmetic-only animations, the target should be forced to the opponent for self/field moves, and multi-hit animations should be clamped to single-hit visuals.

**Primary recommendation:** Build on existing `QuizHooks_*` hooks in `battle_script_commands.c` to neutralize crit/STAB/type multipliers and adjust animation targeting/multi-hit visuals, while removing the non-damage move replacement in `quiz_hooks.c`.

## Standard Stack

The established libraries/tools for this domain:

### Core
| Library | Version | Purpose | Why Standard |
|---------|---------|---------|-------------|
| Pokefirered battle engine (C) | repo current | Battle resolution, scripts, animations | Existing engine entry points already hooked for quiz logic |
| Quiz hooks module (`quiz_hooks.c/.h`) | repo current | Battle behavior overrides | Centralized, already integrated with battle scripts |

### Supporting
| Library | Version | Purpose | When to Use |
|---------|---------|---------|-------------|
| Battle scripts (`battle_script_commands.c`) | repo current | Damage/accuracy/PP, move effects | Use for minimal, engine-safe interception points |
| Battle animations (`battle_anim.c`) | repo current | Animation target selection | Use to force cosmetic-only targeting |

### Alternatives Considered
| Instead of | Could Use | Tradeoff |
|------------|-----------|----------|
| Script hooks | Move data edits in `gBattleMoves` | High risk, broad surface area; breaks move UI/summary cosmetics |

**Installation:** N/A (no external dependencies).

## Architecture Patterns

### Recommended Project Structure
```
pokefirered/src/
├── quiz/               # Quiz feature state and battle hooks
├── battle_script_commands.c  # Battle engine hooks (accuracy, damage, effects)
└── battle_anim.c       # Animation attacker/target plumbing
```

### Pattern 1: Damage override at adjust-normal-damage
**What:** Override final computed damage after all standard modifiers.  
**When to use:** Enforcing fixed damage values or uniform damage logic.  
**Example:**
```
// Source: pokefirered/src/battle_script_commands.c
static void Cmd_adjustnormaldamage(void)
{
    ...
    QuizHooks_ApplyDamageOverride(&gBattleMoveDamage, gBattlerAttacker, gBattlerTarget);
    gBattlescriptCurrInstr++;
}
```

### Pattern 2: Accuracy override at accuracy check
**What:** Short-circuit accuracy logic when quiz mode is active.  
**When to use:** Uniform "always hit" behavior.  
**Example:**
```
// Source: pokefirered/src/battle_script_commands.c
static void Cmd_accuracycheck(void)
{
    ...
    if (QuizHooks_ShouldAlwaysHit())
    {
        ...
        gBattlescriptCurrInstr += 7;
        return;
    }
    ...
}
```

### Pattern 3: Effect override to neutral scripts
**What:** Override move effect to "do nothing" (Splash) or fixed-damage scripts.  
**When to use:** Normalizing status/secondary effects while keeping move name/animation.  
**Example:**
```
// Source: pokefirered/src/quiz/quiz_hooks.c
u8 QuizHooks_GetMoveEffectOverride(u16 move, u8 effect)
{
    ...
    // Wrong answers -> EFFECT_SPLASH
    ...
    return EFFECT_DRAGON_RAGE;
}
```

### Anti-Patterns to Avoid
- **Editing move tables for power/accuracy:** Breaks cosmetic move summaries and spreads changes beyond battle flow.
- **Replacing battle flow logic wholesale:** High risk; existing hooks already cover needed entry points.

## Don't Hand-Roll

Problems that look simple but have existing solutions:

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Fixed damage per turn | New damage pipeline | `QuizHooks_ApplyDamageOverride` | Already called in adjust-normal-damage |
| Always-hit behavior | New accuracy system | `QuizHooks_ShouldAlwaysHit` in `Cmd_accuracycheck` | Existing battle script hook |
| Disable secondary effects | Manual per-move edits | `QuizHooks_ShouldDisableSecondaryEffects` | Centralized, already used |

**Key insight:** The battle engine already exposes script-level hooks where uniform behavior can be enforced without altering move data or UI.

## Common Pitfalls

### Pitfall 1: Safari battles affected unintentionally
**What goes wrong:** Hooks like `QuizHooks_ShouldAlwaysHit()` and `QuizHooks_ShouldInfinitePp()` return TRUE unconditionally, so Safari battles inherit quiz rules.  
**Why it happens:** Safari battle setup does not call `QuizHooks_OnWildBattleStart`, so `Quiz_IsActive()` stays FALSE.  
**How to avoid:** Gate quiz hooks on `Quiz_IsActive()` and/or `!(gBattleTypeFlags & BATTLE_TYPE_SAFARI)`.  
**Warning signs:** Safari battles always hit or ignore PP.

### Pitfall 2: STAB and type effectiveness still apply
**What goes wrong:** `Cmd_typecalc` applies STAB and `ModulateDmgByType` applies multipliers even in quiz mode.  
**Why it happens:** Only immunity checks are gated by `QuizHooks_ShouldIgnoreTypeImmunity()`.  
**How to avoid:** Skip STAB and type multipliers when quiz mode is active.  
**Warning signs:** "Super effective" text or damage variance by type.

### Pitfall 3: Critical hits still occur
**What goes wrong:** `Cmd_critcalc` still sets `gCritMultiplier = 2` based on RNG.  
**Why it happens:** No quiz guard exists in crit calculation.  
**How to avoid:** Force `gCritMultiplier = 1` when quiz mode is active.  
**Warning signs:** "Critical hit!" message appears.

### Pitfall 4: Status moves removed instead of normalized
**What goes wrong:** `QuizHooks_FillPartyMoves` replaces non-damage moves with filler, conflicting with "status moves remain selectable."  
**Why it happens:** Earlier phase removed non-damage moves for quiz answers.  
**How to avoid:** Disable or narrow replacement logic and rely on effect override to neutralize real effects.  
**Warning signs:** Status moves vanish from move menus.

### Pitfall 5: Animations target self/field
**What goes wrong:** Self-targeting or field-targeting animations play on the user, not the opponent.  
**Why it happens:** `DoMoveAnim` uses `gBattlerTarget` to set `gBattleAnimTarget`.  
**How to avoid:** Override `gBattleAnimTarget` (or temporarily override `gBattlerTarget`) for quiz mode before `DoMoveAnim`.  
**Warning signs:** Swords Dance or Recover animations play on the user.

### Pitfall 6: Multi-hit animations still loop
**What goes wrong:** Multi-hit moves animate multiple times.  
**Why it happens:** `gMultiHitCounter` is passed to controllers and used by animations.  
**How to avoid:** Clamp `gMultiHitCounter` to 1 in quiz mode before emitting animation.  
**Warning signs:** Double/Triple-hit visuals despite uniform damage.

## Code Examples

### Damage override hook (already in place)
```
// Source: pokefirered/src/quiz/quiz_hooks.c
bool8 QuizHooks_ApplyDamageOverride(s32 *damage, u8 attacker, u8 target)
{
    ...
    *damage = 1000;
    return TRUE;
}
```

### Accuracy override hook (already in place)
```
// Source: pokefirered/src/battle_script_commands.c
if (QuizHooks_ShouldAlwaysHit())
{
    ...
    gBattlescriptCurrInstr += 7;
    return;
}
```

### Animation target assignment (needs targeting override)
```
// Source: pokefirered/src/battle_anim.c
void DoMoveAnim(u16 move)
{
    gBattleAnimAttacker = gBattlerAttacker;
    gBattleAnimTarget = gBattlerTarget;
    LaunchBattleAnimation(gBattleAnims_Moves, move, TRUE);
}
```

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| Remove non-damage moves | Neutralize effects via hooks | Phase 12/13 transition | Preserves cosmetic move list while keeping uniform behavior |

**Deprecated/outdated:**
- Removing status moves entirely: conflicts with "status moves remain selectable" decision.

## Open Questions

1. **What should "moves never miss" do about Protect/invulnerable states?**
   - What we know: Accuracy is bypassed, but Protect and airborne/underground checks still apply.
   - What's unclear: Whether protect/invulnerable states should be ignored in quiz mode.
   - Recommendation: Decide explicitly; if ignored, add a quiz guard in `Cmd_accuracycheck` and protect checks.

2. **Is uniform damage applied when quiz is inactive (e.g., Safari)?**
   - What we know: Safari skips quiz init; some hooks are unconditional.
   - What's unclear: Whether Safari should remain vanilla or be excluded per decision.
   - Recommendation: Gate hooks on `Quiz_IsActive()` or `BATTLE_TYPE_SAFARI`.

## Sources

### Primary (HIGH confidence)
- `pokefirered/src/quiz/quiz_hooks.c` - damage/accuracy/effect hooks
- `pokefirered/src/battle_script_commands.c` - accuracy, crit, type calc, damage adjust hooks
- `pokefirered/src/battle_anim.c` - animation target assignment
- `pokefirered/src/battle_setup.c` - Safari battle setup (no quiz init)

### Secondary (MEDIUM confidence)
- None (no external docs required)

### Tertiary (LOW confidence)
- None

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH - repo-native engine and hooks
- Architecture: HIGH - verified hook locations and call sites
- Pitfalls: MEDIUM - inferred from current behavior and phase decisions

**Research date:** 2026-01-30  
**Valid until:** 2026-02-27
