You are working inside a Pokémon FireRed decompilation codebase (pret/pokefirered) with a custom feature called “QuizHack”.

Core rule: DO NOT INVENT FIRE-RED INTERNALS.
- Never create or reference a function, struct field, constant, global, or file path unless it is verified to exist in this repository.
- If you need a hook point, first locate it by searching the repo and cite the exact file and function name you found.
- If you cannot find it quickly, stop and propose 2–3 candidate hook locations with evidence (search results / call sites), then ask for guidance.

Primary objective
Implement QuizHack using the modular architecture below. Minimize edits to battle/engine internals. Prefer adding code under src/quiz/ and calling it from a small number of well-defined hook points.

Repository architecture (must follow)
- All new feature logic belongs under:
  - src/quiz/*
  - include/quiz/*
  - tools/*
  - data/questions/*
  - generated/*
- The ONLY place allowed to touch “engine/battle internals” is:
  - src/quiz/quiz_hooks.c (and its header)
- Everything else must be engine-agnostic.

Workflow rules
1) FIND BEFORE WRITE
- Before coding, identify the exact existing function(s) to extend or call.
- Quote the exact identifier names and file paths.
- If adding new helpers, add them to src/quiz/*, not random engine files.

2) SMALL DIFFS
- Keep changes narrow and localized to the current issue.
- Avoid “cleanup”, “refactor”, or unrelated formatting changes.

3) STATE MACHINES, NOT BLOCKING CALLS
- Assume UI and battle flows are state-machine driven.
- Do not assume you can block the game loop waiting for input unless you have verified how the UI works in this repo.

4) SAVE DATA IS HIGH RISK
- Any changes to save structures must be versioned and safe.
- Never overwrite existing save fields without confirming they are unused.
- If save changes are required, add a version field and a safe “wipe on mismatch” behavior.

5) GENERATION PIPELINE
- Question content must come from data/questions/*.json.
- Tools should validate and generate C tables in generated/questions_gen.{h,c}.
- No runtime JSON parsing on GBA.

QuizHack gameplay rules (must implement exactly)
Wild encounter:
- If species in this scope is NOT cleared:
  - Ask 1 question (from that species bank).
  - Correct: wild Pokémon is instantly KO’d and encounter ends.
  - Wrong: player’s active Pokémon loses HALF of its CURRENT HP (integer floor), clamped to minimum 1 HP, and encounter ends.
Mastery:
- Each species bank has N questions (default 5, cap 8).
- Correctly answering all N marks CAPTURE_PENDING.
Capture quiz:
- The NEXT encounter with that species triggers capture quiz mode.
- Ask ALL N questions sequentially (index order is acceptable for MVP).
- Any wrong: apply wrong penalty (half current HP clamp >=1) and encounter ends; capture remains pending.
- All correct: Pokémon is granted to party/PC WITHOUT inventory/Poké Balls; mark species CLEARED.
Cleared species:
- Cleared species should not appear in wild encounters.
- If encounter generator rolls a cleared species, re-roll up to MAX_REROLLS=10; if still cleared, return “no encounter”.

Implementation guidance / preferences
- Prefer adding a small, explicit API in src/quiz/:
  - Quiz_OnWildEncounter(species, scope)
  - Quiz_OnTrainerMonSendOut(species, scope) (later)
  - Quiz_OnGymBattleStart(gymId) (later)
- All engine touchpoints call into these functions.

How to respond when you propose code changes
Always include:
- What you changed (1–2 sentences)
- EXACT files edited
- Where you found the hook point (file + function name + how you verified it)
- How to test it in-game (steps on Route 1 with Rattata/Pidgey)
- What could go wrong (1–2 bullet points)

If uncertain about FireRed specifics
- Do not guess.
- Search the repo for:
  - call sites of the behavior you need
  - existing patterns used for similar features
  - existing UI/menu functions used for multi-choice prompts
- Present evidence and ask for a decision rather than writing speculative code.
