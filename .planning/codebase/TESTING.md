# Testing

## Current Testing Approach

### Manual Testing (Primary)
- Build ROM with `make rom`
- Run in emulator (mGBA, VBA-M, etc.)
- Trigger wild encounters on Route 1
- Verify quiz UI appears and answers work

### Build Verification
- Successful compilation = basic syntax check
- SHA1 comparison (for vanilla FireRed matching, less relevant for hacks)

## Planned Testing Infrastructure

### 1. Data Validation (`tools/validate_questions.py`)
**Purpose**: Validate question bank JSON files

**Checks**:
- [ ] JSON schema compliance
- [ ] Prompt length ≤ 120 characters
- [ ] Answer length ≤ 40 characters per choice
- [ ] Exactly 4 choices per question
- [ ] Valid answer_index (0-3)
- [ ] Unique question IDs
- [ ] No duplicate questions per scope/species

**Usage** (planned):
```bash
make validate  # or: python tools/validate_questions.py
```

### 2. Automated Build Testing (CI)
**GitHub Actions workflow** (planned):
- Run `validate_questions.py`
- Build ROM with `make rom`
- Check for build errors/warnings

### 3. Emulator-Based Testing (Future)
**Headless testing with mGBA scripting**:
- Automate save/load cycles
- Verify mastery persistence
- Test capture quiz flow

## Test Scenarios

### MVP Test Cases

| ID | Scenario | Expected Result |
|----|----------|-----------------|
| T1 | Wild encounter on Route 1 | Quiz question appears |
| T2 | Select correct answer (move slot 0) | Wild Pokémon KO'd |
| T3 | Select wrong answer | Player takes half HP damage |
| T4 | Quiz UI during trainer battle | Should NOT appear |
| T5 | Save/load mid-encounter | State preserved (future) |

### Integration Test Cases (Future)

| ID | Scenario | Expected Result |
|----|----------|-----------------|
| T10 | Master all questions for species | Capture quiz triggers next encounter |
| T11 | Perfect capture quiz | Pokémon added to party |
| T12 | Failed capture quiz | Pokémon flees, retry available |
| T13 | Cleared species reroll | No wild encounters for that species |

## Debug Aids

### In-Game Debug
- `sQuizQuestionText` is hardcoded for easy verification
- Answer slot 0 is always correct (temporary)

### Build-Time Debug
```bash
# Verbose build
make VERBOSE=1 rom

# Check for undefined symbols
arm-none-eabi-nm pokefirered.elf | grep "U "
```

## Known Limitations

- No automated test framework for GBA code
- Manual testing required for UI verification
- Emulator behavior may differ from hardware
