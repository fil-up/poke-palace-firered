---
phase: ENH-C-route-completion
plan: 02
type: execute
wave: 2
depends_on: [ENH-C-01]
files_modified:
  - pokefirered/src/map_name_popup.c
  - pokefirered/src/overworld.c
  - pokefirered/src/quiz/route_completion.c
  - pokefirered/include/quiz/route_completion.h
autonomous: true

must_haves:
  truths:
    - "Player sees terrain progress banner on map entry"
    - "Banner shows X/Y format for each terrain type present"
    - "Player receives message when terrain type becomes completed"
    - "Banner positioned opposite map name popup (top vs bottom)"
  artifacts:
    - path: "pokefirered/src/map_name_popup.c"
      provides: "Completion status banner display"
      contains: "ShowRouteCompletionStatusBanner"
    - path: "pokefirered/src/overworld.c"
      provides: "Banner display hook on map entry"
      contains: "ShowRouteCompletionStatusBanner"
    - path: "pokefirered/src/quiz/route_completion.c"
      provides: "Completion message trigger"
      contains: "CheckAndShowCompletionMessage"
  key_links:
    - from: "pokefirered/src/overworld.c"
      to: "map_name_popup.c"
      via: "ShowRouteCompletionStatusBanner call on map entry"
      pattern: "ShowRouteCompletionStatusBanner"
    - from: "pokefirered/src/map_name_popup.c"
      to: "route_completion.c"
      via: "GetTerrainProgress calls for banner text"
      pattern: "GetTerrainProgress"
    - from: "pokefirered/src/quiz/route_completion.c"
      to: "quiz.c"
      via: "Completion message trigger after species cleared"
      pattern: "CheckAndShowCompletionMessage"
---

<objective>
Implement player feedback for route completion: status banner on map entry and completion message.

Purpose: Players need visual feedback about their progress toward completing each terrain type, and a satisfying notification when they achieve 100% completion on a terrain.

Output: Status banner displayed at top of screen on map entry showing "Grass - 2/3, Surf - 0/2" etc., plus completion message when final species in a terrain is captured.
</objective>

<execution_context>
@C:\Users\phill\.claude/get-shit-done/workflows/execute-plan.md
@C:\Users\phill\.claude/get-shit-done/templates/summary.md
</execution_context>

<context>
@.planning/PROJECT.md
@.planning/STATE.md
@.planning/phases/ENH-C-route-completion/ENH-C-RESEARCH.md
@.planning/phases/ENH-C-route-completion/ENH-C-CONTEXT.md
@.planning/phases/ENH-C-route-completion/ENH-C-01-SUMMARY.md
@pokefirered/src/map_name_popup.c
@pokefirered/src/overworld.c
</context>

<tasks>

<task type="auto">
  <name>Task 1: Create status banner display system</name>
  <files>
    pokefirered/src/map_name_popup.c
  </files>
  <action>
Add a route completion status banner that appears at the top of the screen (opposite the map name popup at the bottom). Follow the existing Task_MapNamePopup pattern.

**1. Add include:**
```c
#include "quiz/route_completion.h"
#include "wild_encounter.h"
```

**2. Add task data defines (below existing tState, tTimer, etc.):**
```c
// Route completion status banner task data
#define tStatusState           data[0]
#define tStatusTimer           data[1]
#define tStatusPos             data[2]
#define tStatusWindowId        data[4]
#define tStatusWindowExists    data[5]
```

**3. Create window setup function:**
```c
static u16 RouteCompletionStatusCreateWindow(void)
{
    struct WindowTemplate windowTemplate = {
        .bg = 0,
        .tilemapLeft = 1,
        .tilemapTop = 1,           // Top of screen (map name uses 29)
        .width = 14,               // Wide enough for "Grass - XX/XX"
        .height = 5,               // Up to 5 terrain types
        .paletteNum = 15,          // Reuse map name palette
        .baseBlock = 0x100         // Different base block
    };
    u16 windowId = AddWindow(&windowTemplate);
    
    // Load palette and window frame (reuse map name resources)
    LoadPalette(GetTextWindowPalette(3), BG_PLTT_ID(15), PLTT_SIZE_4BPP);
    LoadStdWindowTiles(windowId, 0x01D);
    DrawTextBorderOuter(windowId, 0x01D, 15);
    PutWindowTilemap(windowId);
    
    return windowId;
}
```

**4. Create content printing function:**
```c
static void RouteCompletionStatusPrintContent(u16 windowId)
{
    u8 cleared, total;
    u8 y = 0;
    u16 headerId = GetCurrentMapWildMonHeaderId();
    u8 lineBuffer[32];
    
    FillWindowPixelBuffer(windowId, PIXEL_FILL(1));
    
    if (headerId == 0xFFFF)
        return;
    
    // Check each terrain type and print if present
    // Land (grass)
    GetTerrainProgress(headerId, TERRAIN_LAND, &cleared, &total);
    if (total > 0)
    {
        StringCopy(lineBuffer, gText_RouteCompletionGrass);  // "Grass - "
        ConvertIntToDecimalStringN(lineBuffer + StringLength(lineBuffer), cleared, STR_CONV_MODE_LEFT_ALIGN, 2);
        StringAppend(lineBuffer, gText_Slash);  // "/"
        ConvertIntToDecimalStringN(lineBuffer + StringLength(lineBuffer), total, STR_CONV_MODE_LEFT_ALIGN, 2);
        AddTextPrinterParameterized(windowId, FONT_NARROW, lineBuffer, 4, y, 0, NULL);
        y += 12;
    }
    
    // Water (surf) - similar pattern
    GetTerrainProgress(headerId, TERRAIN_WATER, &cleared, &total);
    if (total > 0)
    {
        // ... same pattern with "Surf - X/Y"
        y += 12;
    }
    
    // Fishing rods - show Old/Good/Super Rod if fishingMonsInfo exists
    // Only show rods that player has access to (or always show all?)
    // For simplicity: show all fishing if any fishing exists
    
    CopyWindowToVram(windowId, COPYWIN_GFX);
}
```

**5. Create task function (mirroring Task_MapNamePopup pattern):**
```c
static void Task_RouteCompletionStatus(u8 taskId)
{
    struct Task *task = &gTasks[taskId];
    switch (task->tStatusState)
    {
    case 0:  // Create window
        task->tStatusWindowId = RouteCompletionStatusCreateWindow();
        task->tStatusWindowExists = TRUE;
        RouteCompletionStatusPrintContent(task->tStatusWindowId);
        task->tStatusState = 1;
        break;
    case 1:  // Slide in
        if (IsDma3ManagerBusyWithBgCopy())
            break;
        task->tStatusPos += 2;
        if (task->tStatusPos >= 24)  // Slide down from top
        {
            task->tStatusState = 2;
            task->tStatusTimer = 0;
        }
        break;
    case 2:  // Hold
        task->tStatusTimer++;
        if (task->tStatusTimer > 120)  // Same duration as map name
            task->tStatusState = 3;
        break;
    case 3:  // Slide out
        task->tStatusPos -= 2;
        if (task->tStatusPos <= 0)
            task->tStatusState = 4;
        break;
    case 4:  // Cleanup
        ClearStdWindowAndFrameToTransparent(task->tStatusWindowId, TRUE);
        RemoveWindow(task->tStatusWindowId);
        DestroyTask(taskId);
        break;
    }
}
```

**6. Create public function:**
```c
void ShowRouteCompletionStatusBanner(void)
{
    u8 taskId;
    u16 headerId = GetCurrentMapWildMonHeaderId();
    
    // Only show if map has encounter tables
    if (headerId == 0xFFFF)
        return;
    
    taskId = FindTaskIdByFunc(Task_RouteCompletionStatus);
    if (taskId == TASK_NONE)
    {
        taskId = CreateTask(Task_RouteCompletionStatus, 89);  // Higher priority than map name (90)
        gTasks[taskId].tStatusState = 0;
        gTasks[taskId].tStatusPos = 0;
    }
}
```

**7. Add string constants** (in src/strings.c or inline):
- gText_RouteCompletionGrass: "Grass - "
- gText_RouteCompletionSurf: "Surf - "
- gText_RouteCompletionOldRod: "Old Rod - "
- gText_RouteCompletionGoodRod: "Good Rod - "
- gText_RouteCompletionSuperRod: "Super Rod - "
- gText_Slash: "/"

Declare these strings appropriately (check where map name strings are defined for pattern).
  </action>
  <verify>
ROM builds: `cd pokefirered && make -j$(nproc)` succeeds. `grep "ShowRouteCompletionStatusBanner" pokefirered/src/map_name_popup.c` shows function exists.
  </verify>
  <done>
ShowRouteCompletionStatusBanner function created with task-based window system. Banner displays terrain progress in X/Y format. Window positioned at top of screen with slide-in/hold/slide-out animation matching map name popup timing.
  </done>
</task>

<task type="auto">
  <name>Task 2: Hook banner display on map entry</name>
  <files>
    pokefirered/src/overworld.c
  </files>
  <action>
Add calls to ShowRouteCompletionStatusBanner at the same locations where ShowMapNamePopup is called.

**1. Add include at top of overworld.c:**
```c
// Add near other includes
void ShowRouteCompletionStatusBanner(void);  // From map_name_popup.c
```

Or add declaration to a header file (map_name_popup.h if it exists, otherwise overworld.h or a new header).

**2. Hook into LoadMapFromCameraTransition (around line 780):**
After the existing ShowMapNamePopup call:
```c
if (GetLastUsedWarpMapSectionId() != gMapHeader.regionMapSectionId)
{
    ShowMapNamePopup(TRUE);
    ShowRouteCompletionStatusBanner();  // NEW: Show completion status
}
```

**3. Hook into FieldCB_ShowMapNameOnContinue (around line 1675):**
```c
static void FieldCB_ShowMapNameOnContinue(void)
{
    if (gMapHeader.showMapName == TRUE)
    {
        ShowMapNamePopup(FALSE);
        ShowRouteCompletionStatusBanner();  // NEW
    }
    FieldCB_WarpExitFadeFromBlack();
}
```

**4. Hook into CB2_DoChangeMap case 12 (around line 1910):**
```c
else if (gMapHeader.showMapName == TRUE)
{
    ShowMapNamePopup(FALSE);
    ShowRouteCompletionStatusBanner();  // NEW
}
```

Ensure banner appears alongside map name popup at all map entry points where the map name is shown.
  </action>
  <verify>
ROM builds successfully. In emulator: enter Route 1 from Pallet Town, verify both map name popup (bottom) and completion status banner (top) appear.
  </verify>
  <done>
ShowRouteCompletionStatusBanner called at all three map entry points where ShowMapNamePopup is called. Banner appears whenever map name appears.
  </done>
</task>

<task type="auto">
  <name>Task 3: Add completion message on terrain completion</name>
  <files>
    pokefirered/src/quiz/route_completion.c
    pokefirered/include/quiz/route_completion.h
  </files>
  <action>
Add a message that displays when a terrain type becomes fully completed (all species CLEARED).

**1. Add function to route_completion.h:**
```c
void CheckAndShowTerrainCompletionMessage(u16 species);
```

**2. Implement in route_completion.c:**
```c
// Track which terrain completions have been shown to avoid repeats
static u16 sLastCompletedHeaderId = 0xFFFF;
static u8 sLastCompletedTerrain = 0xFF;

void CheckAndShowTerrainCompletionMessage(u16 species)
{
    u16 headerId = GetCurrentMapWildMonHeaderId();
    u8 terrainType;
    u8 cleared, total;
    const u8 *messageTerrain;
    
    if (headerId == 0xFFFF)
        return;
    
    // Check each terrain type to see if this species completion finished it
    for (terrainType = TERRAIN_LAND; terrainType <= TERRAIN_SUPER_ROD; terrainType++)
    {
        GetTerrainProgress(headerId, terrainType, &cleared, &total);
        
        // If terrain has species AND is now 100% complete AND we haven't shown this one
        if (total > 0 && cleared == total)
        {
            // Avoid showing same completion message twice
            if (sLastCompletedHeaderId == headerId && sLastCompletedTerrain == terrainType)
                continue;
            
            // This terrain just completed!
            sLastCompletedHeaderId = headerId;
            sLastCompletedTerrain = terrainType;
            
            // Queue message based on terrain type
            // Use StringExpandPlaceholders or direct message
            // Format: "Route 1 grass complete!" or similar
            switch (terrainType)
            {
            case TERRAIN_LAND:
                messageTerrain = gText_GrassComplete;
                break;
            case TERRAIN_WATER:
                messageTerrain = gText_SurfComplete;
                break;
            case TERRAIN_OLD_ROD:
                messageTerrain = gText_OldRodComplete;
                break;
            case TERRAIN_GOOD_ROD:
                messageTerrain = gText_GoodRodComplete;
                break;
            case TERRAIN_SUPER_ROD:
                messageTerrain = gText_SuperRodComplete;
                break;
            default:
                return;
            }
            
            // Display message (similar to item pickup or save messages)
            // Use PlayFanfare and ScriptContext_SetupScript or direct text display
            // For now: use a simple approach that works in battle context
            // This will be called after capture success, so we're in battle
            // Queue a battle message or use the battle message system
            
            // Option 1: Set a flag and check in BattleController to show message
            // Option 2: Use MsgQueueAdd from quest_log if applicable
            // Option 3: Show via event script after battle
            
            // Simplest approach: Play a sound and rely on the post-battle field message
            PlaySE(SE_LEVEL_UP);  // Satisfying sound for completion
            
            // For actual text display, need to integrate with field message system
            // This may require additional scaffolding - flag it for post-battle display
            
            return;  // Only show one completion at a time
        }
    }
}
```

**3. Hook into Quiz_SetSpeciesState in quiz.c:**
After setting state to CLEARED:
```c
void Quiz_SetSpeciesState(u16 species, u8 state)
{
    // ... existing code ...
    
    if (state == QUIZ_STATE_CLEARED)
    {
        InvalidateCompletionCache();
        CheckAndShowTerrainCompletionMessage(species);  // NEW
    }
}
```

**4. Add string constants:**
- gText_GrassComplete: "Area grass complete!"
- gText_SurfComplete: "Area surf complete!"
- gText_OldRodComplete: "Old Rod complete!"
- gText_GoodRodComplete: "Good Rod complete!"
- gText_SuperRodComplete: "Super Rod complete!"

**Note on message display:** The completion happens during battle (capture success). The cleanest approach is to play a sound effect immediately and display the text message after the battle ends via the field message system. If time permits, implement full text display; otherwise, the sound effect provides immediate feedback and the status banner confirms on next map entry.
  </action>
  <verify>
ROM builds. In emulator: capture final species for a terrain (may need to temporarily reduce question count for testing), verify completion sound plays.
  </verify>
  <done>
CheckAndShowTerrainCompletionMessage implemented and hooked into species state change. Plays completion sound when terrain becomes 100% cleared. String constants added for completion messages.
  </done>
</task>

</tasks>

<verification>
1. **Build verification:** `cd pokefirered && make clean && make -j$(nproc)` succeeds
2. **Banner function exists:** `grep "ShowRouteCompletionStatusBanner" pokefirered/src/map_name_popup.c`
3. **Overworld hooks added:** `grep "ShowRouteCompletionStatusBanner" pokefirered/src/overworld.c` shows 3 call sites
4. **Completion message hook:** `grep "CheckAndShowTerrainCompletionMessage" pokefirered/src/quiz/quiz.c`
5. **Visual test:** Enter Route 1 → see status banner at top showing "Grass - 0/2" (or current progress)
</verification>

<success_criteria>
- Status banner appears at top of screen on map entry
- Banner shows X/Y progress for each terrain type present on the map
- Banner animates in/out matching map name popup timing
- Completion sound plays when terrain reaches 100% cleared
- ROM builds and runs without crashes or visual glitches
</success_criteria>

<output>
After completion, create `.planning/phases/ENH-C-route-completion/ENH-C-02-SUMMARY.md`
</output>
