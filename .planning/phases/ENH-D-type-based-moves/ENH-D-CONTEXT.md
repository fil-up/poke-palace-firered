# Phase ENH-D: Type-Based Moves - Context

**Gathered:** 2026-01-28
**Status:** Ready for planning

<domain>
## Phase Boundary

Replace vanilla level-up movesets with type-determined moves. Each type has 6 moves across 3 tiers (Weak/Medium/Strong). Pokemon get moves based on their type(s) and evolution stage. All moves deal flat 1000 damage with 100% accuracy and unlimited PP.

</domain>

<decisions>
## Implementation Decisions

### Move Selection
- Use existing Gen 3 FireRed moves (substitute closest alternative for missing moves)
- All moves deal flat 1000 damage (ignores base power, stats, STAB)
- Force 100% accuracy on all moves
- Unlimited PP (moves never run out)
- Physical/special split irrelevant (damage is fixed)
- Pick iconic/recognizable moves per type

### Move Tier System
- 6 moves per type: 2 Weak, 2 Medium, 2 Strong
- Evolution stage determines tier:
  - **Basic Pokemon** → Weak tier moves
  - **Stage 1 (first evolution)** → Medium tier moves
  - **Stage 2 (final evolution)** → Strong tier moves
  - **Legendaries** → Strong tier moves

### Move Assignment
- **Dual-type Pokemon:** 2 moves from primary type + 2 moves from secondary type = 4 moves total
- **Single-type Pokemon:** 2 moves from their type + 2 random moves from any type (randomized per encounter)
- **Move slot order:** Primary type in slots 1-2, secondary type in slots 3-4
- Both types use same evolution tier (e.g., Charizard gets Strong Fire AND Strong Flying)

### Move Learning
- Level thresholds for tier progression:
  - **Weak tier:** Learnable at levels 1-10
  - **Medium tier:** Learnable at levels 20-30
  - **Strong tier:** Learnable at levels 40-50
- Player can choose to learn new tier moves at these levels

### When Moves Apply
- Wild Pokemon have type-based moves only (no vanilla level-up moves)
- Player Pokemon follow same rules
- Starters begin with Weak tier moves (+ random fillers if single-type)
- TMs and HMs all work and can be taught
- HMs can be forgotten (unlike vanilla)

### Damage & Type System
- Flat 1000 damage on all moves (no type effectiveness multipliers)
- Type immunities ignored (all moves hit everything)
- No secondary effects (damage only)

### AI Behavior
- All Pokemon (wild and trainer) choose moves randomly
- Same random AI for gym leaders and all trainers

### Claude's Discretion
- Exact move substitutions for moves not in Gen 3
- Level thresholds within the specified ranges (e.g., 5 vs 8 for Weak)
- Random filler move selection algorithm

</decisions>

<specifics>
## Specific Ideas

### User-Provided Move List by Type

```json
{
  "Normal": {
    "Weak": ["Tackle", "Scratch"],
    "Medium": ["Body Slam", "Strength"],
    "Strong": ["Double-Edge", "Hyper Beam"]
  },
  "Fire": {
    "Weak": ["Ember", "Fire Spin"],
    "Medium": ["Flame Wheel", "Fire Punch"],
    "Strong": ["Flamethrower", "Fire Blast"]
  },
  "Water": {
    "Weak": ["Water Gun", "Bubble"],
    "Medium": ["Surf", "Water Pulse"],
    "Strong": ["Hydro Pump", "Water Spout"]
  },
  "Electric": {
    "Weak": ["Thundershock", "Spark"],
    "Medium": ["Thunderbolt", "Shock Wave"],
    "Strong": ["Thunder", "Volt Tackle"]
  },
  "Grass": {
    "Weak": ["Absorb", "Vine Whip"],
    "Medium": ["Razor Leaf", "Giga Drain"],
    "Strong": ["SolarBeam", "Frenzy Plant"]
  },
  "Ice": {
    "Weak": ["Powder Snow", "Icy Wind"],
    "Medium": ["Ice Beam", "Aurora Beam"],
    "Strong": ["Blizzard", "Sheer Cold"]
  },
  "Fighting": {
    "Weak": ["Karate Chop", "Rock Smash"],
    "Medium": ["Brick Break", "Vital Throw"],
    "Strong": ["Cross Chop", "Sky Uppercut"]
  },
  "Poison": {
    "Weak": ["Poison Sting", "Smog"],
    "Medium": ["Sludge", "Acid"],
    "Strong": ["Sludge Bomb", "Poison Fang"]
  },
  "Ground": {
    "Weak": ["Mud-Slap", "Mud Shot"],
    "Medium": ["Dig", "Bone Rush"],
    "Strong": ["Earthquake", "Fissure"]
  },
  "Flying": {
    "Weak": ["Peck", "Gust"],
    "Medium": ["Wing Attack", "Aerial Ace"],
    "Strong": ["Drill Peck", "Sky Attack"]
  },
  "Psychic": {
    "Weak": ["Confusion", "Psybeam"],
    "Medium": ["Psychic", "Extrasensory"],
    "Strong": ["Psychic", "Future Sight"]
  },
  "Bug": {
    "Weak": ["Leech Life", "Fury Cutter"],
    "Medium": ["Signal Beam", "Silver Wind"],
    "Strong": ["Megahorn", "Signal Beam"]
  },
  "Rock": {
    "Weak": ["Rock Throw", "Rollout"],
    "Medium": ["Rock Slide", "Ancient Power"],
    "Strong": ["Rock Slide", "Rock Blast"]
  },
  "Ghost": {
    "Weak": ["Lick", "Night Shade"],
    "Medium": ["Shadow Ball", "Shadow Punch"],
    "Strong": ["Shadow Ball", "Shadow Punch"]
  },
  "Dragon": {
    "Weak": ["Twister", "Dragon Breath"],
    "Medium": ["Dragon Claw", "Dragon Rage"],
    "Strong": ["Outrage", "Dragon Claw"]
  },
  "Dark": {
    "Weak": ["Pursuit", "Thief"],
    "Medium": ["Bite", "Faint Attack"],
    "Strong": ["Crunch", "Faint Attack"]
  },
  "Steel": {
    "Weak": ["Metal Claw", "Steel Wing"],
    "Medium": ["Iron Tail", "Metal Claw"],
    "Strong": ["Meteor Mash", "Iron Tail"]
  }
}
```

**Note:** Some moves may need Gen 3 substitutions:
- Water Spout, Volt Tackle, Frenzy Plant, Sheer Cold, Luster Purge, Meteor Mash, Aeroblast — verify availability
- Some original entries had wrong-type moves (e.g., Muddy Water for Poison) — corrected above

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope

</deferred>

---

*Phase: ENH-D-type-based-moves*
*Context gathered: 2026-01-28*
