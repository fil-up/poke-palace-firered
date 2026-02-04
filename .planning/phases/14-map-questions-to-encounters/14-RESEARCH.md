# Phase 14: Section-Based Topic Pools with Re-Capturable Species - Research

**Researched:** 2026-02-04
**Domain:** GBA ROM Hacking / Pokémon FireRed Decomp / Quiz System Architecture
**Confidence:** HIGH

## Summary

This phase involves a significant architectural change from species-based question selection to section/topic-based selection. The current system ties questions to species (`Quiz_GetBank(species)`), while the new system ties questions to map locations via topic pools.

Key findings:
1. **Save data must expand** from 64 bytes to ~386 bytes to track per-section cleared/mastery status
2. **400 bytes of unused save space** exists at `unused_348C` in SaveBlock1 - sufficient for expansion
3. **Wild encounter filtering** in `wild_encounter.c` uses `Quiz_IsSpeciesCleared()` - must become section-aware
4. **Build tools** need to generate topic-indexed question pools instead of species-indexed banks
5. **Quiz flow functions** (`Quiz_InitWildEncounter`, mastery tracking) need section context

**Primary recommendation:** Implement a two-phase migration - first add section infrastructure and save data expansion, then convert question selection from species-based to topic-based.

## Current System Architecture

### Save Data Structure (64 bytes at `0x3A94`)

```c
// Current: pokefirered/include/global.h
#define QUIZ_SAVE_SPECIES_COUNT 31

struct QuizSpeciesSave
{
    u8 masteryMask;  // Bitmask of mastered questions (up to 8)
    u8 state;        // QUIZ_STATE_* constant
};

struct QuizSaveData
{
    u8 version;      // Save format version
    u8 padding;
    struct QuizSpeciesSave species[31];  // 62 bytes
};  // Total: 64 bytes
```

**Location in SaveBlock1:** `gSaveBlock1Ptr->quizData` at offset `0x3A94`

**Key limitation:** Uses modulo indexing (`species % 31`), causing hash collisions for 412 species mapped to 31 slots.

### Quiz State Constants

```c
// pokefirered/include/quiz/quiz.h
#define QUIZ_STATE_NONE             0  // Never encountered
#define QUIZ_STATE_MASTERING        1  // Learning mode (answering questions)
#define QUIZ_STATE_CAPTURE_PENDING  2  // All questions mastered, can capture
#define QUIZ_STATE_CLEARED          3  // Species captured/completed
```

### Current Quiz Flow

1. **Encounter initialization:** `Quiz_InitWildEncounter(species, mapGroup, mapNum)`
   - Gets question bank via `Quiz_GetBank(species)` (generated lookup by species)
   - Checks `Quiz_GetSpeciesState(species)` for global cleared status
   - Selects unmastered questions via `Quiz_SelectUnmasteredQuestion()`

2. **Question selection:** Species-indexed via generated `questions_gen.c`
   - `Quiz_GetBank(species)` returns `&gQuizBank_SPECIES_XXX`
   - Each bank contains ~5 questions assigned to that species

3. **Wild encounter filtering:** `wild_encounter.c` rerolls if species cleared
   - `Quiz_IsSpeciesCleared(species)` checks global state
   - Prevents encounters with completed species

4. **Mastery tracking:** Per-species bitmask (8 bits = 8 questions max)
   - `Quiz_GetMasteryMask(species)` / `Quiz_SetMasteryMask(species, mask)`
   - Uses modulo-31 slot mapping

### Available Save Data Space

| Location | Offset | Size | Purpose |
|----------|--------|------|---------|
| `unused_348C` | 0x348C | 400 bytes | **Available for expansion** |
| `quizData` | 0x3A94 | 64 bytes | Current quiz data |
| `unused_3D24` | 0x3D24 | 16 bytes | Small overflow buffer |

**Total available:** 416 bytes for expansion (400 + 16)

## Architecture Patterns

### Recommended Save Data Structure (V2)

```c
// New section-aware save data structure
#define QUIZ_NUM_SECTIONS 8
#define QUIZ_CLEARED_BYTES_PER_SECTION 20  // 160 species as bits
#define QUIZ_MASTERY_SLOTS_PER_SECTION 28  // Modulo-28 for mastery

struct QuizSaveDataV2
{
    u8 version;                             // 1 byte - format version (2)
    u8 currentSection;                      // 1 byte - player's current section
    u8 sectionCleared[8][20];               // 160 bytes - cleared bits per section
    u8 masteryMask[8][28];                  // 224 bytes - mastery per section
};  // Total: 386 bytes - fits in unused_348C
```

**Bit packing for cleared status:**
```c
// Check if species is cleared in section
bool8 Quiz_IsSpeciesClearedInSection(u16 species, u8 section)
{
    u8 byteIndex = species / 8;
    u8 bitIndex = species % 8;
    return (gSaveBlock1Ptr->quizDataV2.sectionCleared[section][byteIndex] >> bitIndex) & 1;
}
```

### Section Mapping

```c
// Map ID → Section lookup
// Generated from section_config.json
u8 Quiz_GetSectionForMap(u16 mapId)
{
    switch (mapId)
    {
    // Section 1: Pre-Brock
    case MAP_PALLET_TOWN:
    case MAP_ROUTE1:
    case MAP_VIRIDIAN_CITY:
    case MAP_VIRIDIAN_FOREST:
    case MAP_ROUTE2:
    case MAP_PEWTER_CITY:
        return 1;
    
    // Section 2: Pre-Misty
    case MAP_ROUTE3:
    case MAP_MT_MOON_1F:
    case MAP_MT_MOON_B1F:
    case MAP_MT_MOON_B2F:
    case MAP_ROUTE4:
    case MAP_CERULEAN_CITY:
    case MAP_ROUTE24:
    case MAP_ROUTE25:
        return 2;
    
    // ... sections 3-8
    
    default:
        return 1;  // Default to section 1
    }
}
```

### Topic Pool Structure

```c
// New: Topic-based question pool
struct QuizTopicPool
{
    const struct QuizQuestion * const *questions;
    u16 count;
    const u8 *categoryName;
    const u8 *topicName;
};

// Map-specific pool aggregation
struct QuizMapConfig
{
    u16 mapId;
    u8 section;
    const struct QuizTopicPool * const *topicPools;
    u8 topicCount;
};

// Generated lookup
const struct QuizMapConfig *Quiz_GetMapConfig(u16 mapId);
```

### Recommended Project Structure Changes

```
pokefirered/
├── data/
│   └── questions/
│       ├── questions.csv          # Master question bank
│       ├── species_map.json       # DEPRECATED - remove after migration
│       ├── section_config.json    # NEW: Map → Section mapping
│       └── map_topics.json        # NEW: Map → Topic pools
├── include/
│   └── quiz/
│       ├── quiz.h                 # Core quiz API
│       ├── quiz_hooks.h           # Battle hooks
│       ├── questions_gen.h        # MODIFIED: Add topic pools
│       ├── section_config.h       # NEW: Section definitions
│       └── topic_pools.h          # NEW: Topic pool declarations
├── src/
│   └── quiz/
│       ├── quiz.c                 # MODIFIED: Section-aware logic
│       ├── questions_gen.c        # MODIFIED: Topic-indexed questions
│       ├── section_config.c       # NEW: Map → section mapping
│       └── topic_pools.c          # NEW: Topic pool data
└── tools/
    └── quiz/
        ├── build_questions.py     # MODIFIED: Generate topic pools
        └── build_sections.py      # NEW: Generate section config
```

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Bit array manipulation | Custom bitfield code | Inline helper macros | Error-prone, easy to get indexing wrong |
| Section determination | Runtime map iteration | Generated switch/lookup table | O(1) vs O(n), no runtime parsing |
| Question pool building | Dynamic allocation | Static const arrays | GBA has limited RAM, static is safer |
| Save data migration | Manual byte manipulation | Version-checked migration function | Ensures backwards compatibility |

**Key insight:** The GBA has limited RAM (256KB EWRAM + 32KB IWRAM). All question data should be in ROM as `static const`. Only save data tracking (cleared/mastery status) goes in SaveBlock.

## Common Pitfalls

### Pitfall 1: Save Data Size Overflow
**What goes wrong:** Expanding save data beyond SaveBlock1 boundaries causes corruption
**Why it happens:** SaveBlock1 is fixed-size, overflow writes to adjacent memory
**How to avoid:** 
- Calculate exact byte requirements before implementation
- Use `STATIC_ASSERT` to verify struct sizes at compile time
- Test save/load with maximum possible data

**Warning signs:** Save corruption, game freezes on load

### Pitfall 2: Section Transition Edge Cases
**What goes wrong:** Species cleared in section N still blocked in section N+1
**Why it happens:** Forgetting to check section-specific cleared status
**How to avoid:**
- Replace ALL calls to `Quiz_IsSpeciesCleared()` with section-aware version
- Grep for all usages before starting implementation

**Warning signs:** Cannot encounter species that should be re-capturable

### Pitfall 3: Mastery Not Resetting Between Sections
**What goes wrong:** Player doesn't need to re-master questions when recapturing
**Why it happens:** Using global mastery instead of section-specific
**How to avoid:** Store mastery per-section, reset on section change

### Pitfall 4: Build Tool Questions-to-Topic Mapping
**What goes wrong:** Questions assigned to wrong topics or missing entirely
**Why it happens:** CSV parsing errors, inconsistent category/topic names
**How to avoid:**
- Validate topic names against a known list
- Generate warnings for unmapped questions
- Include question counts in build output for verification

### Pitfall 5: Map ID Collisions
**What goes wrong:** Different floors of same location return different sections
**Why it happens:** Map IDs like `MAP_MT_MOON_1F`, `MAP_MT_MOON_B1F` not grouped
**How to avoid:** Test all map IDs in each dungeon, document groupings

## Code Examples

### Section-Aware Species Cleared Check

```c
// Replace Quiz_IsSpeciesCleared with section-aware version
bool8 Quiz_IsSpeciesClearedInSection(u16 species, u8 section)
{
    u8 byteIndex, bitIndex;
    
    if (section == 0 || section > QUIZ_NUM_SECTIONS)
        return FALSE;
    
    if (species >= 160)  // Max tracked species
        return FALSE;
    
    byteIndex = species / 8;
    bitIndex = species % 8;
    
    return (gSaveBlock1Ptr->quizDataV2.sectionCleared[section - 1][byteIndex] 
            >> bitIndex) & 1;
}

// Wild encounter filtering update
bool8 Quiz_IsSpeciesCleared(u16 species)
{
    u8 currentSection = Quiz_GetCurrentSection();
    return Quiz_IsSpeciesClearedInSection(species, currentSection);
}
```

### Topic Pool Selection

```c
// Select question from current map's topic pool
const struct QuizQuestion *Quiz_SelectQuestionForMap(u16 mapId)
{
    const struct QuizMapConfig *config;
    const struct QuizTopicPool *pool;
    u8 topicIndex;
    u16 questionIndex;
    
    config = Quiz_GetMapConfig(mapId);
    if (config == NULL || config->topicCount == 0)
        return NULL;
    
    // Random topic from map's pool
    topicIndex = Random() % config->topicCount;
    pool = config->topicPools[topicIndex];
    
    if (pool == NULL || pool->count == 0)
        return NULL;
    
    // Random question from topic
    questionIndex = Random() % pool->count;
    return pool->questions[questionIndex];
}
```

### Save Data Migration

```c
void Quiz_MigrateSaveData(void)
{
    u8 version = gSaveBlock1Ptr->quizDataV2.version;
    
    if (version == 0 || version == 1)
    {
        // Migrate from V1 (species-based) to V2 (section-based)
        // Copy old cleared species to section 1
        // Clear all other sections
        // Set version to 2
        
        // Note: Some data loss expected - old mastery doesn't map cleanly
        Quiz_InitSaveDataV2();
        gSaveBlock1Ptr->quizDataV2.version = 2;
    }
}
```

## Implementation Checklist

### Phase 1: Infrastructure

- [ ] Add `QUIZ_NUM_SECTIONS` constant (8)
- [ ] Create section_config.json with map→section mapping
- [ ] Create build_sections.py to generate section_config.c/h
- [ ] Implement `Quiz_GetSectionForMap(mapId)` lookup

### Phase 2: Save Data Expansion

- [ ] Define `QuizSaveDataV2` struct (386 bytes)
- [ ] Move to `unused_348C` location
- [ ] Implement section-aware cleared/mastery functions
- [ ] Add `Quiz_MigrateSaveData()` for version upgrade
- [ ] Add `STATIC_ASSERT` for struct size

### Phase 3: Topic Pool Generation

- [ ] Create map_topics.json configuration
- [ ] Modify build_questions.py to group by Broad_Category + Sub_Topic
- [ ] Generate topic_pools.c/h with topic-indexed questions
- [ ] Generate map_configs with topic pool references

### Phase 4: Quiz Logic Changes

- [ ] Update `Quiz_InitWildEncounter()` to use topic pools
- [ ] Update `Quiz_IsSpeciesCleared()` to be section-aware
- [ ] Update mastery tracking to be section-aware
- [ ] Update `wild_encounter.c` filtering for re-capturable species

### Phase 5: Testing

- [ ] Verify species re-capture across sections
- [ ] Verify mastery resets per section
- [ ] Verify save/load with new data structure
- [ ] Verify all maps return correct section
- [ ] Test save migration from V1 to V2

## Open Questions

1. **Gym leader battle question source**
   - What we know: Currently uses `Quiz_BuildAreaPool()` aggregating species banks
   - What's unclear: Should gym leaders use the gym map's topic pool, or a "mastery" pool covering all section topics?
   - Recommendation: Use dedicated gym mastery pools per CONTEXT.md (`gyms_by_map` configuration)

2. **Safari Zone handling**
   - What we know: Safari Zone is in Section 5 (Koga gated)
   - What's unclear: Safari Zone has special battle rules - how does quiz integrate?
   - Recommendation: Keep Safari Zone quiz-exempt per current implementation

3. **Topic pool size balance**
   - What we know: Some topics have 124 questions, others have 10
   - What's unclear: Will small topic pools cause excessive question repetition?
   - Recommendation: Monitor during testing, may need minimum pool size thresholds

## Sources

### Primary (HIGH confidence)
- `pokefirered/include/global.h` - SaveBlock1 structure and available space
- `pokefirered/src/quiz/quiz.c` - Current quiz implementation
- `pokefirered/include/quiz/quiz.h` - Quiz API and constants
- `pokefirered/src/wild_encounter.c` - Species filtering logic
- `pokefirered/tools/quiz/build_questions.py` - Build tool architecture
- `.planning/phases/14-map-questions-to-encounters/14-CONTEXT.md` - User decisions

### Secondary (MEDIUM confidence)
- `pokefirered/include/constants/map_groups.h` - All map ID definitions
- `pokefirered/src/quiz/route_completion.c` - Terrain completion tracking

## Metadata

**Confidence breakdown:**
- Save data expansion: HIGH - Verified unused space in SaveBlock1
- Section mapping: HIGH - Map constants well-documented
- Topic pool architecture: HIGH - Follows existing patterns
- Build tool changes: MEDIUM - Requires careful CSV parsing changes
- Migration path: MEDIUM - V1→V2 migration has some data loss edge cases

**Research date:** 2026-02-04
**Valid until:** 30 days (stable codebase)
