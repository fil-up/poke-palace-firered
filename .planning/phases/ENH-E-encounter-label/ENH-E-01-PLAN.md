---
phase: ENH-E-encounter-label
plan: 01
type: execute
wave: 1
depends_on: []
files_modified:
  - pokefirered/include/constants/battle_string_ids.h
  - pokefirered/src/battle_message.c
  - pokefirered/src/battle_main.c
  - pokefirered/include/quiz/quiz.h
autonomous: true
must_haves:
  truths:
    - Wild encounters show "Quiz Encounter!" label at battle start (learning mode)
    - Capture-ready encounters show "Catch Opportunity!" label at battle start
    - Trainer battles and safari zone battles show no label
    - Labels appear after player's Pokemon is sent out, before first turn
  artifacts:
    - STRINGID_QUIZ_CATCH_OPPORTUNITY in battle_string_ids.h
    - sText_QuizCatchOpportunity in battle_message.c
    - Updated BattleIntroPrintQuizTest() to select correct string
  key_links:
    - Quiz_IsCaptureMode() returns TRUE when species is in CAPTURE_PENDING state
    - STRINGID_QUIZ_TEST displays "Quiz Encounter!" for learning mode
    - STRINGID_QUIZ_CATCH_OPPORTUNITY displays "Catch Opportunity!" for capture mode
---

<objective>
Display contextual labels at battle start to inform player of encounter type:
- "Quiz Encounter!" when encountering a species in learning mode (still mastering questions)
- "Catch Opportunity!" when encountering a species ready for capture quiz (all questions mastered)

This small UX improvement helps players understand the encounter stakes before the command menu appears.
</objective>

<context>
@.planning/PROJECT.md
@.planning/STATE.md
@pokefirered/src/battle_main.c (lines 2840-2860 - BattleIntroPrintQuizTest)
@pokefirered/src/battle_message.c (lines 500-503, 888-894 - quiz strings)
@pokefirered/include/constants/battle_string_ids.h (lines 380-395 - string IDs)
@pokefirered/src/quiz/quiz.c (Quiz_IsCaptureMode, Quiz_IsActive)
</context>

<tasks>
<task type="auto">
  <name>Add capture opportunity string ID and text</name>
  <files>
    pokefirered/include/constants/battle_string_ids.h
    pokefirered/src/battle_message.c
  </files>
  <action>
  1. In battle_string_ids.h:
     - Add `#define STRINGID_QUIZ_CATCH_OPPORTUNITY 387` after STRINGID_QUIZ_TEST
     - Update `BATTLESTRINGS_COUNT` from 387 to 388
  
  2. In battle_message.c:
     - Update sText_QuizTestWild to: `static const u8 sText_QuizEncounter[] = _("Quiz Encounter!");`
     - Add new string: `static const u8 sText_QuizCatchOpportunity[] = _("Catch Opportunity!");`
     - Update the gBattleStringsTable entry from sText_QuizTestWild to sText_QuizEncounter
     - Add new entry: `[STRINGID_QUIZ_CATCH_OPPORTUNITY - BATTLESTRINGS_TABLE_START] = sText_QuizCatchOpportunity`
  </action>
  <verify>
  - grep for STRINGID_QUIZ_CATCH_OPPORTUNITY in battle_string_ids.h
  - grep for sText_QuizCatchOpportunity in battle_message.c
  - Confirm BATTLESTRINGS_COUNT equals 388
  </verify>
  <done>
  - New string ID STRINGID_QUIZ_CATCH_OPPORTUNITY (387) defined
  - Two strings exist: "Quiz Encounter!" and "Catch Opportunity!"
  - Both strings registered in gBattleStringsTable
  </done>
</task>

<task type="auto">
  <name>Add Quiz_GetEncounterLabel helper function</name>
  <files>
    pokefirered/include/quiz/quiz.h
    pokefirered/src/quiz/quiz.c
  </files>
  <action>
  1. In quiz.h, add declaration:
     `u16 Quiz_GetEncounterLabelStringId(void);`
  
  2. In quiz.c, add function:
     ```c
     u16 Quiz_GetEncounterLabelStringId(void)
     {
         // Return 0 if quiz not active (no label needed)
         if (!sQuizState.active)
             return 0;
         
         // Return appropriate string ID based on capture mode
         if (sQuizState.captureMode)
             return STRINGID_QUIZ_CATCH_OPPORTUNITY;
         else
             return STRINGID_QUIZ_TEST;
     }
     ```
  3. Add include for `#include "constants/battle_string_ids.h"` if not present
  </action>
  <verify>
  - grep for Quiz_GetEncounterLabelStringId in quiz.h and quiz.c
  - Function returns correct string ID based on captureMode state
  </verify>
  <done>
  - Quiz_GetEncounterLabelStringId() function exists and exported
  - Returns STRINGID_QUIZ_TEST for learning mode
  - Returns STRINGID_QUIZ_CATCH_OPPORTUNITY for capture mode
  - Returns 0 when quiz not active
  </done>
</task>

<task type="auto">
  <name>Update BattleIntroPrintQuizTest to use dynamic label</name>
  <files>
    pokefirered/src/battle_main.c
  </files>
  <action>
  1. Add include for quiz.h if not present: `#include "quiz/quiz.h"`
  
  2. Modify BattleIntroPrintQuizTest() function:
     ```c
     static void BattleIntroPrintQuizTest(void)
     {
         u16 labelStringId;
         
         if (gBattleControllerExecFlags)
             return;
         
         // Get contextual label string ID (0 if none needed)
         labelStringId = Quiz_GetEncounterLabelStringId();
         
         if (labelStringId != 0 && !(gBattleTypeFlags & (BATTLE_TYPE_TRAINER | BATTLE_TYPE_SAFARI)))
             PrepareStringBattle(labelStringId, 0);
         
         gBattleMainFunc = TryDoEventsBeforeFirstTurn;
     }
     ```
  </action>
  <verify>
  - grep for Quiz_GetEncounterLabelStringId in battle_main.c
  - Function calls correct string ID dynamically
  - No label shown for trainer/safari battles
  </verify>
  <done>
  - BattleIntroPrintQuizTest() uses Quiz_GetEncounterLabelStringId()
  - Shows "Quiz Encounter!" for learning mode encounters
  - Shows "Catch Opportunity!" for capture-ready encounters
  - Shows nothing for trainer/safari battles
  </done>
</task>

<task type="checkpoint:human-verify">
  <name>Visual verification of encounter labels</name>
  <files>None (testing)</files>
  <action>
  Build and test the ROM:
  1. `make` to compile
  2. Start new game, get starter, go to Route 1
  3. Encounter wild Rattata (should be in learning mode) - verify "Quiz Encounter!" appears
  4. Answer all 5 Rattata questions correctly across encounters to reach CAPTURE_PENDING
  5. Encounter Rattata again - verify "Catch Opportunity!" appears
  6. (Optional) Fight a trainer battle - verify no label appears
  </action>
  <verify>
  User confirms:
  - "Quiz Encounter!" appears for unmastered species
  - "Catch Opportunity!" appears for capture-ready species
  - No label for non-quiz battles
  </verify>
  <done>
  - Both labels display correctly based on species quiz state
  - Labels do not appear inappropriately
  </done>
</task>
</tasks>

<verification>
1. Build completes without errors: `make`
2. New string ID exists: `grep STRINGID_QUIZ_CATCH_OPPORTUNITY pokefirered/include/constants/battle_string_ids.h`
3. Both strings defined: `grep -E "sText_Quiz(Encounter|CatchOpportunity)" pokefirered/src/battle_message.c`
4. Helper function exists: `grep Quiz_GetEncounterLabelStringId pokefirered/src/quiz/quiz.c`
</verification>

<success_criteria>
1. Wild encounters in learning mode show "Quiz Encounter!" after "Go, [Pokemon]!"
2. Wild encounters in capture-pending mode show "Catch Opportunity!" after "Go, [Pokemon]!"
3. Trainer battles show no quiz label
4. Safari battles show no quiz label
5. Build completes without warnings related to these changes
</success_criteria>
