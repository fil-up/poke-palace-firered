# Phase ENH-C: Route Completion 25% Encounters - Context

**Gathered:** 2026-01-28
**Status:** Ready for planning

<domain>
## Phase Boundary

When a player completes all Pokemon species in an area's terrain type, wild encounter rate for that terrain drops to 25% of normal — easier travel without eliminating encounters. Each terrain type (grass, surf, fishing rods) tracked independently per encounter table.

</domain>

<decisions>
## Implementation Decisions

### Completion definition
- **Trigger:** Both captured AND mastered — species must reach CLEARED state
- **Scope:** ALL species in the encounter table must be CLEARED (no partial credit)
- **Rare species:** All required, including low-rate species like Pikachu (5%)
- **Special encounters:** Excluded — swarms, legendaries, story-triggered don't count

### Terrain types (5 independent tracks)
1. **Grass** — walking in tall grass
2. **Surf** — swimming on water
3. **Old Rod** — fishing encounters
4. **Good Rod** — fishing encounters  
5. **Super Rod** — fishing encounters

Each terrain type on each route tracks and reduces independently.

### Route/area tracking
- **Granularity:** Per encounter table (how game groups wild Pokemon)
- **Shared species:** Once CLEARED anywhere, counts everywhere (Rattata cleared on Route 1 counts for Route 2)
- **Rock Smash:** Excluded for now

### Rate behavior
- **Meaning:** 25% of normal encounter rate (multiplier, not flat 25%)
  - If grass is normally 8.5% per step, becomes ~2.1%
- **Repel interaction:** Stacks — Repel does level filtering, 25% reduction applies independently
- **Movement types:** Applies to all (walking, running, biking)
- **Scope:** Wild encounters only, trainer battles unaffected

### Player feedback
- **Completion message:** Display message when terrain type cleared (e.g., "Route 1 grass complete!")
- **Area status banner:** On zone entry, show progress per terrain type opposite the area name banner
  - Format: "Grass - 2/3, Surf - 1/4, Old Rod - 0/2" (one line per terrain)
  - Always shown, even if all cleared or nothing to track
  - Cleared terrain shows full fraction ("Grass - 3/3"), not checkmark

### Claude's Discretion
- Multi-area maps with multiple encounter tables — how to group
- Save data approach — derive from CLEARED status vs cache completion flags
- Banner position — opposite area name or stacked below
- Missing terrain display — hide line or show "N/A"

</decisions>

<specifics>
## Specific Ideas

- Player described the banner as appearing "on the other side of top" from the area name banner
- Each terrain type gets its own line in the banner
- Consistent fraction display (X/Y) for both in-progress and completed

</specifics>

<deferred>
## Deferred Ideas

- **Time-of-day encounters:** If time mechanics added later, each time slot (morning/day/night) should be tracked independently — separate phase
- **Rock Smash encounters:** Excluded for now, could be added as another terrain type later

</deferred>

---

*Phase: ENH-C-route-completion*
*Context gathered: 2026-01-28*
