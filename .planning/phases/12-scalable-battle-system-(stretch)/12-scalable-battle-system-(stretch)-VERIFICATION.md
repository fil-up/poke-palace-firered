---
phase: 12-scalable-battle-system-(stretch)
verified: 2026-01-29T00:00:00Z
status: human_needed
score: 6/6 must-haves verified
human_verification:
  - test: "Trainer question scaling across encounter tiers"
    expected: "Regular trainers ask 1+ modifiers; gym trainers and leaders apply tier base + evolution modifiers; legendary trainer Pokemon always require 10 questions; last-Pokemon bonus applies for non-regular tiers."
    why_human: "Requires in-game battle flow across multiple trainers to confirm runtime behavior."
  - test: "Pool vs species bank sourcing in multi-question trainer battles"
    expected: "Gym leaders/rival/bosses draw from trainer pools; gym trainers and special encounters draw from species banks while still allowing multi-question flow."
    why_human: "Selection source is runtime behavior and depends on encounter context."
  - test: "Ghost/special encounter quiz initialization"
    expected: "Ghost battles (Pokemon Tower ghost and Marowak) initialize quiz state and use trainer scaling rules rather than wild logic."
    why_human: "Requires entering ghost encounters to confirm quiz initialization and scaling behavior."
---
 
# Phase 12: Scalable Battle System (Stretch) Verification Report
 
**Phase Goal:** Create a scalable battle system (trainer question scaling across trainer/gym/special encounters).
**Verified:** 2026-01-29T00:00:00Z
**Status:** human_needed
**Re-verification:** No — initial verification
 
## Goal Achievement
 
### Observable Truths
 
| # | Truth | Status | Evidence |
| --- | --- | --- | --- |
| 1 | Trainer and special encounters produce consistent scaling inputs so question counts don't fluctuate between retries. | VERIFIED | Scaling inputs come from deterministic trainer ids/classes/battle flags and evolution table scans in `quiz.c` and `trainer_scaling.c`. |
| 2 | Legendary trainer Pokemon consistently use the legendary question total rule. | VERIFIED | `Quiz_GetTrainerQuestionsRequired` returns 10 when `Quiz_IsLegendarySpecies` is true. |
| 3 | Evolution stage and line length influence trainer question totals consistently without per-species overrides. | VERIFIED | `Quiz_GetTrainerQuestionsRequired` adds `Quiz_GetEvolutionStage` and `Quiz_GetEvolutionLineLength` modifiers derived from `gEvolutionTable`. |
| 4 | Trainer-facing encounters compute per-Pokemon question totals using base + modifiers, with last-Pokemon bonus for non-regular trainers. | VERIFIED | `Quiz_GetTrainerQuestionsRequired` uses tier base + evolution/line modifiers and adds last-Pokemon bonus when tier is non-regular. |
| 5 | Gym leaders/rival/boss trainers continue using trainer pools; gym trainers use species banks with multi-question logic. | VERIFIED | `Quiz_GetTrainerQuestionSource` routes leaders/rivals to pools; `Quiz_InitWildEncounter` builds pools for pool sources and species banks for others in multi-question mode. |
| 6 | Ghost/special encounters initialize quiz state and apply trainer scaling (not wild logic). | VERIFIED | `DoGhostBattle` and `StartMarowakBattle` call `QuizHooks_OnWildBattleStart` after `BATTLE_TYPE_GHOST` is set; `Quiz_InitWildEncounter` treats special encounters as trainer battles. |
 
**Score:** 6/6 truths verified
 
### Required Artifacts
 
| Artifact | Expected | Status | Details |
| --- | --- | --- | --- |
| `pokefirered/include/quiz/trainer_scaling.h` | Trainer-scaling helper declarations. | VERIFIED | Exists and declares trainer tier + evolution helpers; used by `quiz.c`. |
| `pokefirered/src/quiz/trainer_scaling.c` | Trainer classification, legendary list, and evolution helpers. | VERIFIED | Exists with concrete implementations and table scans; imported by `quiz.c`. |
| `pokefirered/src/quiz/quiz.c` | Trainer question scaling integration and multi-question handling. | VERIFIED | Integrates trainer tiers, modifiers, and pool/species selection in `Quiz_InitWildEncounter`. |
| `pokefirered/include/quiz/quiz.h` | Updated trainer multi-question API comments/semantics. | VERIFIED | Exposes trainer multi-question helpers used by quiz hooks/UI. |
| `pokefirered/src/battle_setup.c` | Ghost/special battle quiz initialization. | VERIFIED | Ghost and Marowak battles call `QuizHooks_OnWildBattleStart` after flags set. |
 
### Key Link Verification
 
| From | To | Via | Status | Details |
| --- | --- | --- | --- | --- |
| `trainer_scaling.c` | `evolution.h` | `gEvolutionTable` scan | VERIFIED | Helpers scan `gEvolutionTable` for pre-evo/evo detection. |
| `trainer_scaling.c` | `constants/opponents.h` | Gym trainer id list | VERIFIED | `sGymTrainerIds` uses `TRAINER_*` constants. |
| `quiz.c` | `trainer_scaling.c` | Tier + evolution helpers | VERIFIED | Calls `Quiz_GetTrainerTier`, `Quiz_GetEvolutionStage`, `Quiz_GetEvolutionLineLength`, `Quiz_IsLegendarySpecies`. |
| `battle_setup.c` | `quiz_hooks.c` | `QuizHooks_OnWildBattleStart` | VERIFIED | Ghost/Marowak calls wire battle setup to quiz init. |
 
### Requirements Coverage
 
| Requirement | Status | Blocking Issue |
| --- | --- | --- |
| Phase 12 requirements | N/A | ROADMAP lists requirements as TBD. |
 
### Anti-Patterns Found
 
| File | Line | Pattern | Severity | Impact |
| --- | --- | --- | --- | --- |
| `pokefirered/src/quiz/quiz.c` | 908 | TODO comment | Warning | Non-blocking note about save index collision handling. |
 
### Human Verification Required
 
### 1. Trainer question scaling across encounter tiers
 
**Test:** Fight a regular trainer, gym trainer, gym leader, rival/boss, and a legendary trainer Pokemon.
**Expected:** Question totals match tier base + evolution modifiers; legendary trainer Pokemon always require 10; last-Pokémon bonus applies for non-regular tiers.
**Why human:** Requires in-game battles to confirm runtime behavior.
 
### 2. Pool vs species bank sourcing in multi-question trainer battles
 
**Test:** Compare question selection in gym leader/rival/boss vs gym trainer/special encounters.
**Expected:** Leaders/rivals/bosses draw from trainer pools; gym trainers/specials draw from species banks.
**Why human:** Selection source is runtime behavior.
 
### 3. Ghost/special encounter quiz initialization
 
**Test:** Trigger Pokemon Tower ghost battle and the Marowak battle with/without Silph Scope.
**Expected:** Quiz initializes and uses trainer scaling rules in both cases.
**Why human:** Requires in-game flow and battle setup sequence.
 
### Gaps Summary
 
No code-level gaps found for the phase must-haves. Human verification is required to confirm in-game behavior.
 
---
 
_Verified: 2026-01-29T00:00:00Z_
_Verifier: Claude (gsd-verifier)_
