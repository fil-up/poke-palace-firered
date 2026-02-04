# Phase 14 Context: Section-Based Topic Pools

## Design Decisions (Locked)

### 1. Section-Based Progression
The game is divided into 8 sections, each gated by a gym:

| Section | Gym Gate | Areas |
|---------|----------|-------|
| 1 | Brock | Pallet Town, Route 1, Viridian City, Viridian Forest, Pewter City |
| 2 | Misty | Route 3, Mt Moon, Route 4, Cerulean City, Route 24, Route 25 |
| 3 | Lt. Surge | Route 5, Route 6, Vermilion City, S.S. Anne, Route 11, Diglett's Cave |
| 4 | Erika | Route 9, Route 10, Rock Tunnel, Lavender Town, Celadon City, Pewter Museum |
| 5 | Koga | Route 12-15, Fuchsia City, Safari Zone |
| 6 | Sabrina | Saffron City |
| 7 | Blaine | Route 19-21, Seafoam Islands, Cinnabar Island, Pokemon Mansion |
| 8 | Giovanni/E4 | Route 22-23, Victory Road, Indigo Plateau |

### 2. Topic Pools by Map
Questions are selected based on the current map's topic pool, NOT by species.

### 3. Re-Capturable Species
- Species mastery/cleared is tracked **per-section**
- A Rattata cleared in Section 1 can be encountered and re-captured in Section 7
- Each capture uses the current section's topic pool

### 4. Mastery Per-Species
- Still need 5 correct answers to capture a species
- But this resets for each section the species appears in

## Validated Topic Configuration

```yaml
question_pool_by_map:
  # ============================================
  # SECTION 1: Pre-Brock (Intro topics)
  # ============================================
  MAP_PALLET_TOWN:
    - { category: "Plan Provisions", topic: "Product Design" }
    - { category: "Manual Rates", topic: "Dental Claim Costs" }
    - { category: "Plan Provisions", topic: "Life Insurance" }
    - { category: "Plan Provisions", topic: "ACA Regulations" }
    - { category: "Underwriting", topic: "General Principles" }

  MAP_ROUTE1:
    - { category: "Plan Provisions", topic: "Product Design" }
    - { category: "Manual Rates", topic: "Dental Claim Costs" }
    - { category: "Plan Provisions", topic: "Life Insurance" }
    - { category: "Plan Provisions", topic: "ACA Regulations" }
    - { category: "Underwriting", topic: "General Principles" }

  MAP_ROUTE2:  # AUTO-FILLED: inherits from Route 1
    - { category: "Plan Provisions", topic: "Product Design" }
    - { category: "Manual Rates", topic: "Dental Claim Costs" }
    - { category: "Plan Provisions", topic: "Life Insurance" }
    - { category: "Plan Provisions", topic: "ACA Regulations" }
    - { category: "Underwriting", topic: "General Principles" }

  MAP_VIRIDIAN_CITY:
    - { category: "Plan Provisions", topic: "Product Design" }
    - { category: "Manual Rates", topic: "Dental Claim Costs" }
    - { category: "Plan Provisions", topic: "Life Insurance" }
    - { category: "Plan Provisions", topic: "ACA Regulations" }
    - { category: "Underwriting", topic: "General Principles" }

  MAP_VIRIDIAN_FOREST:
    - { category: "Manual Rates", topic: "Medical Claim Costs" }
    - { category: "Manual Rates", topic: "Medical Cost Trend" }
    - { category: "ASOPs", topic: "Data Quality" }

  MAP_PEWTER_CITY:
    - { category: "Manual Rates", topic: "Medical Claim Costs" }
    - { category: "Manual Rates", topic: "Medical Cost Trend" }
    - { category: "ASOPs", topic: "Data Quality" }

  # ============================================
  # SECTION 2: Pre-Misty (Medical/Dental focus)
  # ============================================
  MAP_ROUTE3:  # AUTO-FILLED: transition to Mt Moon topics
    - { category: "Manual Rates", topic: "Medical Claim Costs" }
    - { category: "Manual Rates", topic: "Medical Cost Trend" }
    - { category: "ASOPs", topic: "Data Quality" }

  MAP_MT_MOON_1F:
    - { category: "Manual Rates", topic: "Medical Claim Costs" }
    - { category: "Manual Rates", topic: "Medical Cost Trend" }
    - { category: "ASOPs", topic: "Data Quality" }

  MAP_MT_MOON_B1F:  # AUTO-FILLED: inherits from Mt Moon 1F
    - { category: "Manual Rates", topic: "Medical Claim Costs" }
    - { category: "Manual Rates", topic: "Medical Cost Trend" }
    - { category: "ASOPs", topic: "Data Quality" }

  MAP_MT_MOON_B2F:  # AUTO-FILLED: inherits from Mt Moon 1F
    - { category: "Manual Rates", topic: "Medical Claim Costs" }
    - { category: "Manual Rates", topic: "Medical Cost Trend" }
    - { category: "ASOPs", topic: "Data Quality" }

  MAP_ROUTE4:  # AUTO-FILLED: post-Mt Moon, transition to Cerulean
    - { category: "Manual Rates", topic: "Medical Claim Costs" }
    - { category: "Manual Rates", topic: "Dental Claim Costs" }

  MAP_CERULEAN_CITY:
    - { category: "Manual Rates", topic: "Dental Claim Costs" }
    - { category: "Plan Provisions", topic: "Life Insurance" }

  MAP_ROUTE24:
    - { category: "Manual Rates", topic: "Dental Claim Costs" }
    - { category: "Plan Provisions", topic: "Life Insurance" }

  MAP_ROUTE25:
    - { category: "Manual Rates", topic: "Dental Claim Costs" }
    - { category: "Plan Provisions", topic: "Life Insurance" }

  # ============================================
  # SECTION 3: Pre-Surge (Pharmacy/Trend focus)
  # ============================================
  MAP_ROUTE5:  # AUTO-FILLED: inherits from Vermilion
    - { category: "Manual Rates", topic: "Pharmacy Claim Costs" }

  MAP_ROUTE6:  # AUTO-FILLED: inherits from Vermilion
    - { category: "Manual Rates", topic: "Pharmacy Claim Costs" }

  MAP_VERMILION_CITY:
    - { category: "Manual Rates", topic: "Pharmacy Claim Costs" }

  MAP_S_S_ANNE_1F:
    - { category: "Manual Rates", topic: "Pharmacy Claim Costs" }

  MAP_ROUTE11:
    - { category: "Manual Rates", topic: "Trend Analysis" }
    - { category: "Manual Rates", topic: "Medical Cost Trend" }

  MAP_DIGLETTS_CAVE_B1F:  # AUTO-FILLED: main cave area
    - { category: "Manual Rates", topic: "Trend Analysis" }
    - { category: "Manual Rates", topic: "Medical Cost Trend" }

  # ============================================
  # SECTION 4: Pre-Erika (ASOPs/Underwriting intro)
  # ============================================
  MAP_ROUTE7:  # AUTO-FILLED: Celadon approach
    - { category: "ASOPs", topic: "Risk Classification" }
    - { category: "ASOPs", topic: "Credibility" }

  MAP_ROUTE8:  # AUTO-FILLED: Lavender approach
    - { category: "Underwriting", topic: "Selection" }
    - { category: "Underwriting", topic: "Risk Assessment" }

  MAP_PEWTER_MUSEUM_1F:
    - { category: "Manual Rates", topic: "Life Insurance" }
    - { category: "Plan Provisions", topic: "Life Insurance" }

  MAP_ROUTE9:
    - { category: "ASOPs", topic: "Risk Classification" }
    - { category: "ASOPs", topic: "Data Quality" }
    - { category: "ASOPs", topic: "Credibility" }
    - { category: "ASOPs", topic: "Actuarial Communications" }

  MAP_ROUTE10:
    - { category: "Underwriting", topic: "Experience Rating" }
    - { category: "Underwriting", topic: "Funding Methods" }
    - { category: "Employee Benefits", topic: "COB" }

  MAP_ROCK_TUNNEL_1F:
    - { category: "Underwriting", topic: "Stop Loss" }
    - { category: "Underwriting", topic: "Funding Methods" }

  MAP_ROCK_TUNNEL_B1F:  # AUTO-FILLED: inherits from Rock Tunnel 1F
    - { category: "Underwriting", topic: "Stop Loss" }
    - { category: "Underwriting", topic: "Funding Methods" }

  MAP_LAVENDER_TOWN:
    - { category: "Underwriting", topic: "Selection" }
    - { category: "Underwriting", topic: "Risk Adjustment" }
    - { category: "Underwriting", topic: "Risk Assessment" }

  MAP_POKEMON_TOWER_1F:  # AUTO-FILLED: ghost Pokemon, inherits Lavender
    - { category: "Underwriting", topic: "Selection" }
    - { category: "Underwriting", topic: "Risk Adjustment" }
  MAP_POKEMON_TOWER_2F:
    - { category: "Underwriting", topic: "Selection" }
    - { category: "Underwriting", topic: "Risk Adjustment" }
  MAP_POKEMON_TOWER_3F:
    - { category: "Underwriting", topic: "Selection" }
    - { category: "Underwriting", topic: "Risk Assessment" }
  MAP_POKEMON_TOWER_4F:
    - { category: "Underwriting", topic: "Selection" }
    - { category: "Underwriting", topic: "Risk Assessment" }
  MAP_POKEMON_TOWER_5F:
    - { category: "Underwriting", topic: "Risk Adjustment" }
    - { category: "Underwriting", topic: "Risk Assessment" }
  MAP_POKEMON_TOWER_6F:
    - { category: "Underwriting", topic: "Risk Adjustment" }
    - { category: "Underwriting", topic: "Risk Assessment" }
  MAP_POKEMON_TOWER_7F:
    - { category: "Underwriting", topic: "Risk Adjustment" }
    - { category: "Underwriting", topic: "Risk Assessment" }

  MAP_CELADON_CITY:
    - { category: "Underwriting", topic: "Total Risk Analysis" }

  # ============================================
  # SECTION 5: Pre-Koga (Employee Benefits focus)
  # ============================================
  MAP_ROUTE12:  # AUTO-FILLED: inherits Fuchsia approach
    - { category: "Employee Benefits", topic: "Benefit Strategy" }
    - { category: "Employee Benefits", topic: "COB" }

  MAP_ROUTE13:  # AUTO-FILLED
    - { category: "Employee Benefits", topic: "Benefit Strategy" }
    - { category: "Employee Benefits", topic: "COB" }

  MAP_ROUTE14:  # AUTO-FILLED
    - { category: "Employee Benefits", topic: "Benefit Strategy" }
    - { category: "Plan Provisions", topic: "ACA Regulations" }

  MAP_ROUTE15:  # AUTO-FILLED
    - { category: "Employee Benefits", topic: "Benefit Strategy" }
    - { category: "Plan Provisions", topic: "ACA Regulations" }

  MAP_FUCHSIA_CITY:
    - { category: "Employee Benefits", topic: "Benefit Strategy" }
    - { category: "Employee Benefits", topic: "COB" }
    - { category: "Plan Provisions", topic: "ACA Regulations" }

  MAP_SAFARI_ZONE_CENTER:  # AUTO-FILLED: Safari Zone inherits Fuchsia
    - { category: "Employee Benefits", topic: "Benefit Strategy" }
    - { category: "Employee Benefits", topic: "COB" }
  MAP_SAFARI_ZONE_EAST:
    - { category: "Employee Benefits", topic: "Benefit Strategy" }
    - { category: "Employee Benefits", topic: "COB" }
  MAP_SAFARI_ZONE_NORTH:
    - { category: "Employee Benefits", topic: "Benefit Strategy" }
    - { category: "Plan Provisions", topic: "ACA Regulations" }
  MAP_SAFARI_ZONE_WEST:
    - { category: "Employee Benefits", topic: "Benefit Strategy" }
    - { category: "Plan Provisions", topic: "ACA Regulations" }

  # ============================================
  # SECTION 6: Pre-Sabrina (Advanced Benefits)
  # ============================================
  MAP_SAFFRON_CITY:
    - { category: "Employee Benefits", topic: "Benefit Strategy" }
    - { category: "Employee Benefits", topic: "COB" }
    - { category: "Plan Provisions", topic: "ACA Regulations" }

  # ============================================
  # SECTION 7: Pre-Blaine (Cumulative review)
  # ============================================
  MAP_ROUTE19:  # AUTO-FILLED: water route, cumulative topics
    - { category: "Manual Rates", topic: "Medical Claim Costs" }
    - { category: "Underwriting", topic: "Experience Rating" }
    - { category: "ASOPs", topic: "Credibility" }

  MAP_ROUTE20:  # AUTO-FILLED: water route
    - { category: "Manual Rates", topic: "Pharmacy Claim Costs" }
    - { category: "Underwriting", topic: "Stop Loss" }
    - { category: "ASOPs", topic: "Data Quality" }

  MAP_ROUTE21_NORTH:  # AUTO-FILLED
    - { category: "Plan Provisions", topic: "Product Design" }
    - { category: "Underwriting", topic: "Funding Methods" }
    - { category: "Employee Benefits", topic: "COB" }

  MAP_ROUTE21_SOUTH:  # AUTO-FILLED
    - { category: "Plan Provisions", topic: "Life Insurance" }
    - { category: "Underwriting", topic: "Selection" }
    - { category: "Employee Benefits", topic: "Benefit Strategy" }

  MAP_SEAFOAM_ISLANDS_1F:  # AUTO-FILLED: Articuno dungeon
    - { category: "Manual Rates", topic: "Trend Analysis" }
    - { category: "Underwriting", topic: "Risk Assessment" }
  MAP_SEAFOAM_ISLANDS_B1F:
    - { category: "Manual Rates", topic: "Trend Analysis" }
    - { category: "Underwriting", topic: "Risk Assessment" }
  MAP_SEAFOAM_ISLANDS_B2F:
    - { category: "Manual Rates", topic: "Medical Claim Costs" }
    - { category: "Underwriting", topic: "Total Risk Analysis" }
  MAP_SEAFOAM_ISLANDS_B3F:
    - { category: "Manual Rates", topic: "Medical Claim Costs" }
    - { category: "Underwriting", topic: "Total Risk Analysis" }
  MAP_SEAFOAM_ISLANDS_B4F:
    - { category: "ASOPs", topic: "Credibility" }
    - { category: "Underwriting", topic: "Total Risk Analysis" }

  MAP_CINNABAR_ISLAND:  # AUTO-FILLED: city hub
    - { category: "Plan Provisions", topic: "Product Design" }
    - { category: "Manual Rates", topic: "Medical Claim Costs" }
    - { category: "Underwriting", topic: "General Principles" }

  MAP_POKEMON_MANSION_1F:  # AUTO-FILLED: Moltres dungeon
    - { category: "Underwriting", topic: "Risk Assessment" }
    - { category: "Underwriting", topic: "Total Risk Analysis" }
  MAP_POKEMON_MANSION_2F:
    - { category: "Underwriting", topic: "Risk Assessment" }
    - { category: "Underwriting", topic: "Selection" }
  MAP_POKEMON_MANSION_3F:
    - { category: "Underwriting", topic: "Risk Adjustment" }
    - { category: "Underwriting", topic: "Selection" }
  MAP_POKEMON_MANSION_B1F:
    - { category: "Underwriting", topic: "Risk Adjustment" }
    - { category: "Underwriting", topic: "Total Risk Analysis" }

  # ============================================
  # SECTION 8: Pre-Champion (Final challenge - all topics)
  # ============================================
  MAP_ROUTE22:  # AUTO-FILLED: Victory Road approach
    - { category: "Plan Provisions", topic: "Product Design" }
    - { category: "Manual Rates", topic: "Medical Claim Costs" }
    - { category: "Underwriting", topic: "General Principles" }
    - { category: "Employee Benefits", topic: "Benefit Strategy" }

  MAP_ROUTE23:  # AUTO-FILLED: Victory Road approach
    - { category: "Plan Provisions", topic: "ACA Regulations" }
    - { category: "Manual Rates", topic: "Trend Analysis" }
    - { category: "Underwriting", topic: "Experience Rating" }
    - { category: "ASOPs", topic: "Credibility" }

  MAP_VICTORY_ROAD_1F:  # AUTO-FILLED: final dungeon
    - { category: "Underwriting", topic: "Funding Methods" }
    - { category: "Underwriting", topic: "Stop Loss" }
    - { category: "ASOPs", topic: "Risk Classification" }
  MAP_VICTORY_ROAD_2F:
    - { category: "Underwriting", topic: "Selection" }
    - { category: "Underwriting", topic: "Risk Assessment" }
    - { category: "ASOPs", topic: "Data Quality" }
  MAP_VICTORY_ROAD_3F:
    - { category: "Underwriting", topic: "Risk Adjustment" }
    - { category: "Underwriting", topic: "Total Risk Analysis" }
    - { category: "ASOPs", topic: "Actuarial Communications" }

  MAP_INDIGO_PLATEAU_EXTERIOR:  # AUTO-FILLED: E4 approach
    - { category: "Employee Benefits", topic: "Benefit Strategy" }
    - { category: "Plan Provisions", topic: "Product Design" }
    - { category: "Manual Rates", topic: "Medical Claim Costs" }
    - { category: "Underwriting", topic: "Total Risk Analysis" }

gyms_by_map:
  MAP_PEWTER_CITY_GYM:
    gym: 1
    mastery_pool:
      - { category: "Plan Provisions", topic: "Product Design" }
      - { category: "Manual Rates", topic: "Dental Claim Costs" }
      - { category: "Plan Provisions", topic: "Life Insurance" }
      - { category: "Underwriting", topic: "General Principles" }

  MAP_CERULEAN_CITY_GYM:
    gym: 2
    mastery_pool:
      - { category: "Manual Rates", topic: "Medical Claim Costs" }
      - { category: "Manual Rates", topic: "Dental Claim Costs" }
      - { category: "Manual Rates", topic: "Medical Cost Trend" }

  MAP_VERMILION_CITY_GYM:
    gym: 3
    mastery_pool:
      - { category: "Manual Rates", topic: "Medical Claim Costs" }
      - { category: "Manual Rates", topic: "Dental Claim Costs" }
      - { category: "Manual Rates", topic: "Pharmacy Claim Costs" }
      - { category: "Manual Rates", topic: "Trend Analysis" }
      - { category: "Manual Rates", topic: "Medical Cost Trend" }
      - { category: "Manual Rates", topic: "Life Insurance" }

  MAP_CELADON_CITY_GYM:
    gym: 4
    mastery_pool:
      - { category: "ASOPs", topic: "Risk Classification" }
      - { category: "ASOPs", topic: "Data Quality" }
      - { category: "ASOPs", topic: "Credibility" }
      - { category: "ASOPs", topic: "Actuarial Communications" }

  MAP_FUCHSIA_CITY_GYM:
    gym: 5
    mastery_pool:
      - { category: "Underwriting", topic: "Experience Rating" }
      - { category: "Underwriting", topic: "Funding Methods" }
      - { category: "Underwriting", topic: "Stop Loss" }

  MAP_SAFFRON_CITY_GYM:
    gym: 6
    mastery_pool:
      - { category: "Underwriting", topic: "Selection" }
      - { category: "Underwriting", topic: "Risk Adjustment" }
      - { category: "Underwriting", topic: "Risk Assessment" }
      - { category: "Underwriting", topic: "Total Risk Analysis" }

  MAP_CINNABAR_ISLAND_GYM:  # AUTO-FILLED: Gym 7 - cumulative review
    gym: 7
    mastery_pool:
      - { category: "Plan Provisions", topic: "Product Design" }
      - { category: "Manual Rates", topic: "Medical Claim Costs" }
      - { category: "Manual Rates", topic: "Trend Analysis" }
      - { category: "ASOPs", topic: "Credibility" }
      - { category: "Underwriting", topic: "Experience Rating" }
      - { category: "Employee Benefits", topic: "Benefit Strategy" }

  MAP_VIRIDIAN_CITY_GYM:  # AUTO-FILLED: Gym 8 (Giovanni) - all advanced topics
    gym: 8
    mastery_pool:
      - { category: "Underwriting", topic: "Funding Methods" }
      - { category: "Underwriting", topic: "Stop Loss" }
      - { category: "Underwriting", topic: "Selection" }
      - { category: "Underwriting", topic: "Risk Assessment" }
      - { category: "Underwriting", topic: "Risk Adjustment" }
      - { category: "Underwriting", topic: "Total Risk Analysis" }
      - { category: "ASOPs", topic: "Risk Classification" }
      - { category: "ASOPs", topic: "Actuarial Communications" }

elite_four:
  - member: "E4_1"
    focus: ["Plan Provisions"]
  - member: "E4_2"
    focus: ["Manual Rates"]
  - member: "E4_3"
    focus: ["Underwriting"]
  - member: "E4_4"
    focus: ["Employee Benefits"]
  - member: "CHAMPION"
    focus: ["Any"]
```

## Question Bank Stats (from updated CSV)

| Category | Topic | ~Questions |
|----------|-------|------------|
| Employee Benefits | Benefit Strategy | 124 |
| Manual Rates | Medical Claim Costs | 60 |
| Manual Rates | Pharmacy Claim Costs | 45 |
| Underwriting | Risk Assessment | 42 |
| Plan Provisions | Product Design | 37 |
| Underwriting | Funding Methods | 36 |
| Manual Rates | Trend Analysis | 31 |
| Plan Provisions | Life Insurance | 30 |
| Underwriting | Stop Loss | 29 |
| Plan Provisions | ACA Regulations | 29 |
| Underwriting | Selection | 26 |
| ASOPs | Credibility | 25 |
| Employee Benefits | COB | 23 |
| ASOPs | Data Quality | 23 |
| Manual Rates | Dental Claim Costs | 21 |
| Underwriting | Experience Rating | 20 |
| Underwriting | Total Risk Analysis | 17 |
| Underwriting | Risk Adjustment | 16 |
| ASOPs | Actuarial Communications | 16 |
| Underwriting | General Principles | 14 |
| Manual Rates | Life Insurance | 14 |
| Manual Rates | Medical Cost Trend | 10 |
| ASOPs | Risk Classification | 10 |

## Key Implementation Requirements

1. **New data file**: `map_topics.json` or `section_config.json`
   - Maps each MAP_* to a section (1-8)
   - Maps each section to its topic pool

2. **Save data changes**: 
   - Change from `speciesCleared[NUM_SPECIES]` to `speciesCleared[NUM_SECTIONS][NUM_SPECIES]`
   - Change mastery tracking similarly

3. **Quiz logic changes**:
   - `Quiz_GetQuestionsForMap(mapId)` instead of `Quiz_GetBank(species)`
   - Select questions from current map's topic pool

4. **Build tool changes**:
   - Generate topic pools instead of species banks
   - New CSV parsing to group by Broad_Category + Sub_Topic
