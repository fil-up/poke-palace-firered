# Phase 12: Scalable Battle System (Stretch) - Context

**Gathered:** 2026-01-29
**Status:** Ready for planning

<domain>
## Phase Boundary

Implement scalable question counts for trainer-facing encounters: each Pokemon in trainer battles asks a fixed number of questions, the last Pokemon asks more, and this applies to trainer battles, gym trainers/leaders, and special encounters (e.g., Ghost Marowak), but not wild encounters.

</domain>

<decisions>
## Implementation Decisions

### Trainer base counts
- Regular trainer: base 1 question; no last-Pokemon bonus.
- Gym trainer: base 2 questions.
- Gym leader: base 5 questions.
- Rival/boss: base 5 questions.
- Last-Pokemon bonus: +2 questions for non-regular trainer battles.

### Evolution stage modifiers
- Basic: +0.
- Stage1: +1.
- Stage2: +2.
- Legendary: 10 total questions (override other modifiers).

### Evolution-line length modifiers
- Basic Pokemon with no evolutions (non-legendary): +2.
- Pokemon with only one evolution (two-stage line): +1.

### Claude's Discretion
None — all rules specified.

</decisions>

<specifics>
## Specific Ideas

No specific requirements — open to standard approaches.

</specifics>

<deferred>
## Deferred Ideas

- Legendary trainer/special encounters always result in a catch after questions — out of scope for this phase.

</deferred>

---

*Phase: 12-scalable-battle-system-(stretch)*
*Context gathered: 2026-01-29*
