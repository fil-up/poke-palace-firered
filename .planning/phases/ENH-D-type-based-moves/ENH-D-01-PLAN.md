---
phase: ENH-D-type-based-moves
plan: 01
type: execute
wave: 1
depends_on: []
files_modified:
  - pokefirered/src/quiz/type_moves.c
  - pokefirered/src/quiz/type_moves.h
  - pokefirered/src/quiz/quiz_hooks.c
autonomous: true

must_haves:
  truths:
    - "Wild Pokemon have 4 moves based on their type(s), not level-up learnsets"
    - "Dual-type Pokemon have 2 moves from each type"
    - "Single-type Pokemon have 2 type moves + 2 random filler moves"
    - "Evolution stage determines move tier (Basic=Weak, Stage1=Medium, Stage2/Legend=Strong)"
    - "All Pokemon in battle (player and enemy) use type-based moves"
  artifacts:
    - path: "pokefirered/src/quiz/type_moves.c"
      provides: "Type-based move assignment logic"
      exports: ["TypeMoves_AssignToMon", "TypeMoves_AssignToParty"]
      min_lines: 180
    - path: "pokefirered/src/quiz/type_moves.h"
      provides: "Public API for type moves"
      exports: ["TypeMoves_AssignToMon", "TypeMoves_AssignToParty"]
    - path: "pokefirered/src/quiz/quiz_hooks.c"
      provides: "Battle init integration"
      contains: "TypeMoves_AssignToParty"
  key_links:
    - from: "pokefirered/src/quiz/quiz_hooks.c"
      to: "pokefirered/src/quiz/type_moves.c"
      via: "TypeMoves_AssignToParty call in QuizHooks_OnBattleInit"
      pattern: "TypeMoves_AssignToParty.*gPlayerParty"
    - from: "pokefirered/src/quiz/type_moves.c"
      to: "gSpeciesInfo"
      via: "Type lookup for move selection"
      pattern: "gSpeciesInfo\\[.*\\]\\.types"
    - from: "pokefirered/src/quiz/type_moves.c"
      to: "gEvolutionTable"
      via: "Evolution stage detection"
      pattern: "gEvolutionTable\\[.*\\]"
---

<objective>
Implement type-based move assignment for all Pokemon in battle.

Purpose: Replace vanilla level-up movesets with a simplified type-determined system. Each Pokemon gets moves based on its type(s) and evolution stage, making battles more predictable and thematic for the quiz-driven gameplay.

Output:
- `type_moves.c` — Move assignment module with type→moves mapping and evolution detection
- `type_moves.h` — Public API header
- Updated `quiz_hooks.c` — Integration that assigns type-based moves at battle start
</objective>

<execution_context>
@C:\Users\phill\.claude/get-shit-done/workflows/execute-plan.md
@C:\Users\phill\.claude/get-shit-done/templates/summary.md
</execution_context>

<context>
@.planning/PROJECT.md
@.planning/ROADMAP.md
@.planning/STATE.md
@.planning/phases/ENH-D-type-based-moves/ENH-D-CONTEXT.md
@.planning/phases/ENH-D-type-based-moves/ENH-D-RESEARCH.md
@pokefirered/src/quiz/quiz_hooks.c
@pokefirered/include/constants/moves.h
@pokefirered/include/constants/pokemon.h
</context>

<tasks>

<task type="auto">
  <name>Task 1: Create type_moves module with data and evolution detection</name>
  <files>pokefirered/src/quiz/type_moves.c, pokefirered/src/quiz/type_moves.h</files>
  <action>
Create `type_moves.h` with:
- Function declarations for `TypeMoves_AssignToMon(struct Pokemon *mon)` and `TypeMoves_AssignToParty(struct Pokemon *party, u8 count)`
- Include guards

Create `type_moves.c` with:

1. **Includes:** global.h, pokemon.h, constants/pokemon.h, constants/moves.h, constants/species.h, data/pokemon/species_info.h (for gSpeciesInfo), quiz/type_moves.h

2. **Static move table** `sTypeMoves[NUMBER_OF_MON_TYPES][3][2]` (18 types × 3 tiers × 2 moves):
   Use exact moves from ENH-D-CONTEXT.md:
   ```c
   static const u16 sTypeMoves[NUMBER_OF_MON_TYPES][3][2] = {
       [TYPE_NORMAL] = {
           { MOVE_TACKLE, MOVE_SCRATCH },           // Weak
           { MOVE_BODY_SLAM, MOVE_STRENGTH },       // Medium
           { MOVE_DOUBLE_EDGE, MOVE_HYPER_BEAM }    // Strong
       },
       [TYPE_FIRE] = { ... },
       // ... all 18 types from CONTEXT.md
   };
   ```
   Note: Skip TYPE_MYSTERY (index 9) - leave as {0,0} or use Normal moves as fallback.

3. **Evolution stage detection:**
   - `static u16 GetPreEvolutionSpecies(u16 species)` — Reverse lookup of gEvolutionTable; iterate all species and evolution slots to find what evolves INTO this species. Return SPECIES_NONE if nothing evolves into it.
   - `static bool8 IsLegendarySpecies(u16 species)` — Hardcoded switch for Gen 1-3 legendaries: ARTICUNO, ZAPDOS, MOLTRES, MEWTWO, MEW, RAIKOU, ENTEI, SUICUNE, LUGIA, HO_OH, CELEBI, REGIROCK, REGICE, REGISTEEL, LATIAS, LATIOS, KYOGRE, GROUDON, RAYQUAZA, JIRACHI, DEOXYS.
   - `static u8 GetEvolutionStage(u16 species)` — Returns 0 (Basic/Weak), 1 (Stage1/Medium), or 2 (Stage2/Strong). Legendaries always return 2. Otherwise count pre-evolution chain depth.

**Important:** Include `data/pokemon/evolution.h` to access gEvolutionTable. Check the actual header path in the codebase - it may be at `src/data/pokemon/evolution.h` and require extern declaration.
  </action>
  <verify>
Files compile without errors:
```bash
cd pokefirered && make -j$(nproc) 2>&1 | head -50
```
Check for any undefined reference or type errors related to type_moves.
  </verify>
  <done>
- `type_moves.h` exists with TypeMoves_AssignToMon and TypeMoves_AssignToParty declarations
- `type_moves.c` exists with sTypeMoves array (108 moves), evolution detection functions
- No compile errors from these files
  </done>
</task>

<task type="auto">
  <name>Task 2: Implement move assignment and integrate with quiz_hooks</name>
  <files>pokefirered/src/quiz/type_moves.c, pokefirered/src/quiz/quiz_hooks.c</files>
  <action>
In `type_moves.c`, add:

1. **TypeMoves_AssignToMon(struct Pokemon *mon):**
   ```c
   void TypeMoves_AssignToMon(struct Pokemon *mon)
   {
       u16 species = GetMonData(mon, MON_DATA_SPECIES, NULL);
       if (species == SPECIES_NONE)
           return;
       
       u8 type1 = gSpeciesInfo[species].types[0];
       u8 type2 = gSpeciesInfo[species].types[1];
       u8 stage = GetEvolutionStage(species);  // 0, 1, or 2
       
       // Slots 0-1: Primary type moves
       SetMonMoveSlot(mon, sTypeMoves[type1][stage][0], 0);
       SetMonMoveSlot(mon, sTypeMoves[type1][stage][1], 1);
       
       if (type1 != type2)
       {
           // Dual-type: slots 2-3 from secondary type
           SetMonMoveSlot(mon, sTypeMoves[type2][stage][0], 2);
           SetMonMoveSlot(mon, sTypeMoves[type2][stage][1], 3);
       }
       else
       {
           // Single-type: slots 2-3 random from any type
           u8 randType1 = Random() % NUMBER_OF_MON_TYPES;
           u8 randType2 = Random() % NUMBER_OF_MON_TYPES;
           SetMonMoveSlot(mon, sTypeMoves[randType1][stage][Random() % 2], 2);
           SetMonMoveSlot(mon, sTypeMoves[randType2][stage][Random() % 2], 3);
       }
   }
   ```
   Note: Include random.h for Random(). Handle TYPE_MYSTERY by rerolling if selected.

2. **TypeMoves_AssignToParty(struct Pokemon *party, u8 count):**
   ```c
   void TypeMoves_AssignToParty(struct Pokemon *party, u8 count)
   {
       u32 i;
       for (i = 0; i < count; i++)
       {
           if (GetMonData(&party[i], MON_DATA_SPECIES, NULL) != SPECIES_NONE)
               TypeMoves_AssignToMon(&party[i]);
       }
   }
   ```

In `quiz_hooks.c`:

3. **Add include:** `#include "quiz/type_moves.h"` at top with other includes

4. **Modify QuizHooks_OnBattleInit():**
   Replace the existing calls to `QuizHooks_FillPartyMoves()` and `QuizHooks_FillEnemyMoves()` with type-based assignment:
   ```c
   void QuizHooks_OnBattleInit(void)
   {
       Quiz_ResetBattleState();
       
       // Assign type-based moves to all battle participants
       TypeMoves_AssignToParty(gPlayerParty, PARTY_SIZE);
       TypeMoves_AssignToParty(gEnemyParty, PARTY_SIZE);
   }
   ```

5. **Keep QuizHooks_FillPartyMoves and QuizHooks_FillEnemyMoves** — they may still be called elsewhere. Just remove calls from OnBattleInit.
  </action>
  <verify>
1. Build ROM:
```bash
cd pokefirered && make -j$(nproc)
```

2. Test in emulator (if available):
   - Start wild encounter with a Pokemon (e.g., Rattata on Route 1)
   - Check player Pokemon has 4 moves matching its type(s)
   - Check wild Pokemon has 4 moves matching its type(s)
   - Verify a single-type Pokemon (like Charmander) has 2 Fire moves + 2 random
   - Verify a dual-type Pokemon (like Pidgey - Normal/Flying) has 2 Normal + 2 Flying moves
  </verify>
  <done>
- `TypeMoves_AssignToMon` assigns 4 moves based on type(s) and evolution stage
- `TypeMoves_AssignToParty` iterates party and calls AssignToMon
- `QuizHooks_OnBattleInit` calls TypeMoves_AssignToParty for both parties
- ROM compiles successfully
- Wild encounters show type-based moves on both player and enemy Pokemon
  </done>
</task>

</tasks>

<verification>
1. **Compile check:** `cd pokefirered && make clean && make -j$(nproc)` completes without errors

2. **Code inspection:**
   - `type_moves.c` contains sTypeMoves array with all 18 types
   - `type_moves.c` contains GetEvolutionStage, IsLegendarySpecies, GetPreEvolutionSpecies
   - `quiz_hooks.c` includes type_moves.h and calls TypeMoves_AssignToParty

3. **Grep patterns:**
   - `rg "TypeMoves_AssignToParty" pokefirered/src/` shows calls in quiz_hooks.c
   - `rg "sTypeMoves\[TYPE_" pokefirered/src/` shows move table entries

4. **Gameplay verification (manual):**
   - Start new game, enter tall grass
   - FIGHT menu shows player Pokemon with type-appropriate moves
   - Enemy Pokemon uses type-appropriate moves
</verification>

<success_criteria>
- All Pokemon in battle receive moves based on type(s) and evolution stage
- Dual-type Pokemon get 2+2 moves from their types
- Single-type Pokemon get 2 type moves + 2 random moves
- Evolution stage (Basic/Stage1/Stage2/Legendary) determines move tier
- No compile errors, ROM boots and battles function correctly
</success_criteria>

<output>
After completion, create `.planning/phases/ENH-D-type-based-moves/ENH-D-01-SUMMARY.md`
</output>
