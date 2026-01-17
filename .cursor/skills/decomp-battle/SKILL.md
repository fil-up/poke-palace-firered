---
name: decomp-battle
description: Provide deep, practical expertise in Pokémon decomp hacking with a focus on **battle mechanics**, **battle controllers**, **damage calculation**, and **engine-safe modifications**. Optimize for correctness, engine stability, and minimal surface-area changes.

---

### **Core Mental Model (MANDATORY)**

* Pokémon games are **state machines**, not linear scripts
* Battle flow is controlled by:

  * setup → controllers → tasks → callbacks
* UI events (menus, cursor movement) are **not** the same as turn resolution
* Damage is best modified at **calculation time**, not via fake moves or stats

---

### **Hard Rules (non-negotiable)**

1. **Never invent engine internals**

   * No fictional helpers, structs, globals, or macros
   * All identifiers must be verified via repo search

2. **Search before writing**

   * Always identify file + function + call chain
   * Cite evidence before proposing a change

3. **Prefer interception over replacement**

   * Hook existing flows
   * Do not reimplement battle logic unless unavoidable

4. **Battle UI ≠ Battle Resolution**

   * Menus and cursor movement are UI layers
   * Damage, speed, and turn order live deeper

5. **Damage overrides > fake mechanics**

   * Prefer fixed damage injection
   * Avoid adding custom moves or modifying move tables unless required

---

### **Battle-Specific Expertise**

This skill understands:

#### Battle Phases

* Battle setup vs intro text vs command menu vs move select vs execution
* When it is safe to show UI
* When it is unsafe to block execution

#### Player Interaction

* Command menu (`FIGHT / BAG / POKEMON / RUN`)
* Move menu cursor changes
* Move confirmation vs hover state

#### Damage & Turn Resolution

* Fixed damage application
* Turn order enforcement
* Speed manipulation vs forced priority
* Post-damage callbacks

#### Engine Safety Patterns

* Use flags/state structs instead of globals when possible
* Reset per-turn quiz state after execution
* Avoid modifying persistent stats mid-battle

---

### **Common Footguns This Skill Avoids**

* ❌ Injecting UI before battle controllers are ready
* ❌ Modifying move data to represent quiz answers
* ❌ Overwriting speed stats permanently
* ❌ Blocking battle tasks waiting for input
* ❌ Adding logic inside animation callbacks

---

### **Preferred Implementation Patterns**

* Add minimal hooks in:

  * battle setup (to arm mode)
  * player controller (menu & cursor)
  * damage calculation (final override)

* Keep feature logic in isolated modules (`src/quiz/*`)

* Engine files should only *call into* feature code

---

### **How This Skill Responds to Tasks**

When given a task, this skill will:

1. Identify correct battle phase
2. Locate existing hook points
3. Propose minimal intervention
4. Warn about instability risks
5. Stop before over-engineering

---

### **Explicit Non-Goals**

This skill will **not**:

* Redesign the battle engine
* Introduce custom scripting languages
* Add new Pokémon mechanics unless specified
* Optimize performance prematurely

---
