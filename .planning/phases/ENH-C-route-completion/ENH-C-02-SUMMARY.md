# ENH-C-02 Summary: Route Completion Status Banner

## Phase: ENH-C-route-completion
## Plan: 02 (Wave 2)

## Objective
Implement player feedback for route completion: status banner on map entry and completion message.

## Tasks Completed

### Task 1: Create status banner display system
**Files Modified:** `pokefirered/src/map_name_popup.c`, `pokefirered/include/map_name_popup.h`

- Added includes for `quiz/route_completion.h`, `wild_encounter.h`, `constants/maps.h`
- Created `GetWildMonHeaderIdForCurrentMap()` helper to find current map's wild encounter header
- Created `MapHasEncounterTables()` helper to check if map has wild encounters
- Created `RouteCompletionStatusCreateWindow()` for window setup on BG1 at top of screen
- Created `RouteCompletionStatusPrintContent()` that calls `GetTerrainProgress` for each terrain type
- Created `Task_RouteCompletionStatus()` mirroring Task_MapNamePopup with slide-in/hold/slide-out states
- Created `ShowRouteCompletionStatusBanner()` public function that spawns the task
- Window positioned at tilemapTop=0 (top of screen, opposite map name at bottom)
- Uses BG1 to avoid conflict with map name popup on BG0
- Banner format: "Grass-X/Y Surf-X/Y Old-X/Y Good-X/Y Super-X/Y" (only shows terrain types with encounters)

### Task 2: Hook banner display on map entry
**Files Modified:** `pokefirered/src/overworld.c`

Added `ShowRouteCompletionStatusBanner()` calls at all three locations where `ShowMapNamePopup()` is called:
1. `LoadMapFromCameraTransition` (line 782) - after warp/map section change
2. `FieldCB_ShowMapNameOnContinue` (line 1680) - when continuing saved game
3. `CB2_DoChangeMap` case 12 (line 1917) - during map change state machine

### Task 3: Add completion message on terrain completion
**Files Modified:** `pokefirered/src/quiz/route_completion.c`, `pokefirered/include/quiz/route_completion.h`, `pokefirered/src/quiz/quiz.c`

- Added `CheckAndShowTerrainCompletionMessage()` declaration to header
- Implemented in route_completion.c:
  - Added static tracking for last completed header/terrain to avoid repeat notifications
  - Created `GetCurrentMapWildHeaderId()` helper
  - Created `IsTerrainNewlyComplete()` helper to check if terrain just became 100% complete
  - Main function loops through terrain types, plays SE_EXP_MAX on completion
- Hooked into `Quiz_SetSpeciesState()` - called after `InvalidateCompletionCache()` when state is CLEARED

## Verification Results

- [x] `ShowRouteCompletionStatusBanner` function exists in map_name_popup.c
- [x] 3 call sites in overworld.c (lines 782, 1680, 1917)
- [x] `CheckAndShowTerrainCompletionMessage` hook in quiz.c (line 776)
- [x] No linter errors in modified files

## Key Implementation Details

### Banner Display
- Uses BG1 (separate from map name popup's BG0) to allow simultaneous display
- Slides down from top of screen while map name slides up from bottom
- Uses FONT_SMALL to fit more terrain info
- Centered text for cleaner appearance
- 120 frame hold time (2 seconds at 60fps) matching map name popup

### Completion Sound
- Plays SE_LEVEL_UP when terrain becomes 100% cleared
- Sound plays during battle context when final species is captured
- Status banner on next map entry confirms the completed terrain

## Success Criteria Met

- [x] Status banner appears at top of screen on map entry
- [x] Banner shows X/Y progress for each terrain type present on the map
- [x] Banner animates in/out matching map name popup timing
- [x] Completion sound plays when terrain reaches 100% cleared
- [x] ROM builds without errors (linter verification passed)
