# Testing Patterns

**Analysis Date:** 2026-01-30

## Test Framework

**Runner:**
- No formal unit test framework (GBA C code)
- Build verification via `make` compilation
- SHA1 checksum comparison for vanilla matching
- Python validation scripts for data files

**Build Commands:**
```bash
make                    # Build FireRed ROM
make leafgreen          # Build LeafGreen variant
make firered_rev1       # Build revision 1.1
make modern             # Build with modern GCC
make compare            # Build and verify SHA1 hash
make clean              # Remove build artifacts
make -j$(nproc)         # Parallel build for speed
```

## Test File Organization

**Location:**
- No formal test directory (decomp project pattern)
- Validation scripts in `pokefirered/tools/quiz/`
- Build verification via Make targets

**Naming:**
- `validate_*.py` for data validation
- `build_*.py` for code generation

**Structure:**
```
pokefirered/
├── tools/
│   └── quiz/
│       ├── validate_questions.py  # CSV schema validation
│       └── build_questions.py     # C code generation
```

## Build Verification

### Primary Verification Method

Build success is the primary test for C code. If compilation succeeds without errors, the code is syntactically valid.

```bash
# Standard build (uses agbcc)
make

# Modern GCC with more warnings
make modern

# Verify against original ROM checksum
make compare
```

**Expected output:**
```
pokefirered.gba: OK
```

### Linker Map Verification

Check memory usage and symbol placement:

```bash
# Memory usage printed during link
cd build/firered && arm-none-eabi-ld ... --print-memory-usage

# Check for undefined symbols
arm-none-eabi-nm pokefirered.elf | grep "U "
```

## Data Validation

### Question CSV Validation

**Script:** `pokefirered/tools/quiz/validate_questions.py`

**Purpose:** Validate quiz question CSV before code generation

**Checks performed:**
- Required columns present (Question_ID, Question_String, Option_Label, etc.)
- Exactly 4 options (A, B, C, D) per question
- Question text length ≤ 500 characters (paginated)
- Answer text length ≤ 110 characters
- Correct_Answer is valid (A, B, C, or D)
- Difficulty_Level is 1-5
- No empty required fields

**Usage:**
```bash
python pokefirered/tools/quiz/validate_questions.py "GH 101 Question Bank - Sheet1.csv"

# With strict mode (warns on length limits, duplicates)
python pokefirered/tools/quiz/validate_questions.py questions.csv --strict
```

**Sample output:**
```
Validating: questions.csv
Limits: question=500 chars, answer=110 chars

✓ Validated 155 questions
VALIDATION PASSED
```

### Code Generation

**Script:** `pokefirered/tools/quiz/build_questions.py`

**Purpose:** Generate C source from validated CSV

**Generates:**
- `include/quiz/questions_gen.h` - Extern declarations
- `src/quiz/questions_gen.c` - String data and bank arrays

**Usage:**
```bash
python pokefirered/tools/quiz/build_questions.py questions.csv \
    --mapping species_map.json \
    --out-dir pokefirered/
```

## CI/CD Integration

### GitHub Actions

**Workflow:** `.github/workflows/ai-pr-review.yml`

**Current capabilities:**
- Triggered on PR open/update
- AI-powered code review via AutoAgent

**Future enhancements (not yet implemented):**
- Run `validate_questions.py`
- Build ROM in CI
- Verify build success

### Manual Pre-Commit Checks

Before committing:
```bash
# 1. Validate question data (if changed)
python pokefirered/tools/quiz/validate_questions.py questions.csv

# 2. Regenerate C code (if CSV changed)
python pokefirered/tools/quiz/build_questions.py questions.csv --out-dir pokefirered/

# 3. Build ROM
cd pokefirered && make -j$(nproc)

# 4. Test in emulator
```

## Manual Testing Protocol

### Emulator Testing

**Recommended emulators:**
- mGBA (most accurate, debugging features)
- VBA-M (widely compatible)

### Test Scenarios

| ID | Scenario | Steps | Expected Result |
|----|----------|-------|-----------------|
| T1 | Wild encounter quiz | Walk in Route 1 grass | Quiz question appears in B_WIN_QUIZ |
| T2 | Correct answer | Select answer matching correctIndex | Wild Pokémon KO'd, battle ends |
| T3 | Wrong answer | Select wrong answer | Player takes half HP, battle ends |
| T4 | Mastery tracking | Answer all questions for species correctly | Species state transitions to CAPTURE_PENDING |
| T5 | Capture quiz | Encounter species in CAPTURE_PENDING state | All questions asked in sequence |
| T6 | Perfect capture | Answer all capture questions correctly | Pokémon added to party, species CLEARED |
| T7 | Failed capture | Answer any capture question wrong | Pokémon flees, remains CAPTURE_PENDING |
| T8 | Cleared species | Encounter cleared species | Encounter rerolls (species not encountered) |
| T9 | Trainer battle | Battle any trainer | Quiz shows, no capture mechanics |
| T10 | Save/load | Save mid-progress, reload | Mastery bits persist correctly |

### Gym/Trainer Testing

| ID | Scenario | Expected Result |
|----|----------|-----------------|
| T11 | Gym trainer battle | 2+ questions required per Pokémon |
| T12 | Gym leader battle | 5+ questions, pool from area |
| T13 | Rival battle | Full question pool available |
| T14 | Wrong answer (trainer) | Attack fails, streak resets |
| T15 | All correct (trainer) | Enemy Pokémon KO'd |

## Debug Aids

### In-Game Debugging

```c
// Static state accessible via debugger
static EWRAM_DATA struct QuizState sQuizState = {0};

// Key fields to inspect:
// - sQuizState.active
// - sQuizState.turnResult
// - sQuizState.captureMode
// - sQuizState.gymQuestionsAnswered
```

### Build-Time Debugging

```bash
# Verbose make output
make VERBOSE=1

# Keep intermediate files (.i, .s)
make KEEP_TEMPS=1

# Generate symbol file
make syms
```

### Emulator Debugging (mGBA)

```bash
# Start mGBA with GDB server
mgba -g pokefirered.gba

# Connect with arm-none-eabi-gdb
arm-none-eabi-gdb pokefirered.elf
(gdb) target remote localhost:2345
(gdb) break Quiz_OnMoveConfirmed
```

## Coverage

**Requirements:** None enforced (no automated testing infrastructure)

**Practical coverage:**
- Build success covers syntax/linking
- Manual testing covers gameplay flow
- Data validation covers question integrity

## Test Types

### Build Tests (Automated)

- Compilation success
- No linker errors
- SHA1 match (compare target)

### Data Validation (Semi-Automated)

- CSV schema validation
- String length limits
- Required field presence

### Integration Tests (Manual)

- Full gameplay flow
- Save/load persistence
- Edge cases (empty party slot, etc.)

### E2E Tests

- Not currently implemented
- Future: mGBA scripting for automated playthrough

## Common Patterns

### Verifying Quiz State

In emulator debugger or by adding temporary debug output:

```c
// Temporary debug: print quiz state to battle message
// (Remove before commit)
#ifdef DEBUG
if (Quiz_IsActive())
    BattlePutTextOnWindow("Quiz Active", B_WIN_MSG);
#endif
```

### Testing Save Data

```c
// After save/load, verify:
u8 mask = Quiz_GetMasteryMask(SPECIES_RATTATA);
u8 state = Quiz_GetSpeciesState(SPECIES_RATTATA);
// Compare against expected values
```

### Forcing Specific States (Debug)

```c
// Force species to CAPTURE_PENDING for testing
Quiz_SetSpeciesState(SPECIES_RATTATA, QUIZ_STATE_CAPTURE_PENDING);

// Force mastery complete
Quiz_SetMasteryMask(SPECIES_RATTATA, 0x1F);  // All 5 bits set
```

## Known Limitations

- No automated unit test framework for GBA C code
- Manual testing required for all UI/gameplay verification
- Emulator behavior may differ slightly from real hardware
- No CI build currently configured (only AI PR review)
- Coverage metrics not tracked

---

*Testing analysis: 2026-01-30*
