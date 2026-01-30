# Phase 13: Uniform Damage/Accuracy, Cosmetic Animations - Context

**Gathered:** 2026-01-29  
**Status:** Ready for planning

<domain>
## Phase Boundary

Make all moves behave the same for damage/accuracy while keeping their original animations and cosmetic text. This applies broadly to battles, but Safari encounters are excluded. No new mechanics beyond uniform damage/accuracy and cosmetic-only move animations.

</domain>

<decisions>
## Implementation Decisions

### Damage & accuracy rules
- Player correct answer: deal fixed 1000 damage (instant KO), no enemy damage that turn.
- Player wrong answer: player loses half current HP (min 1), regardless of enemy move.
- Accuracy is 100% and moves never miss.
- Type effectiveness and STAB are disabled.
- Critical hits are disabled.

### Where it applies
- Apply to all battles (wild, trainer, gym, special, legendary, scripted).
- Includes double battles and battle facilities.
- Excludes Safari Zone encounters.
- Applies equally to player and enemy moves.
- PP is infinite; Struggle should never occur.
- Move names remain unchanged in menus.

### Move selection & data presentation
- Status/special moves remain selectable; they are normalized to the uniform behavior.
- Move summaries keep original power/accuracy values (cosmetic only).
- Move descriptions remain unchanged.
- PP area remains replaced by quiz progress (no PP/infinite indicator).

### Animation policy
- Use each move’s original animation and SFX.
- Status moves keep their original animations.
- Multi-hit animations should render as single-hit visuals.
- Move text should show the original move name.
- Force animations to target the opponent (ignore self/field targeting).
- Healing/weather animations are allowed as visuals only.
- If an animation causes glitches, it may be skipped.

### Claude's Discretion
None — all rules specified.

</decisions>

<specifics>
## Specific Ideas

No additional specifics beyond the decisions above.

</specifics>

<deferred>
## Deferred Ideas

None identified.

</deferred>

---

*Phase: 13-make-all-moves-the-same-from-a-damage-accuracy-perspective-but-retain-only-the-move-animation-for-cosmetic-purposes*  
*Context gathered: 2026-01-29*
