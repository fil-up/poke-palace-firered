# Technology Stack

**Analysis Date:** 2026-01-30

## Languages

**Primary:**
- **C** (C89/GNU89) - All game logic, battle system, quiz mechanics
- **ARM Assembly** - Low-level routines, boot code, critical performance paths

**Secondary:**
- **Python 3** - Build tools (CSV validation, C table generation)
- **GNU Make** - Build system orchestration
- **Bash** - Build scripts, installation scripts

## Runtime

**Target Hardware:**
- **Game Boy Advance** (ARM7TDMI processor)
- Clock: 16.78 MHz
- RAM: 32KB IWRAM + 256KB EWRAM
- Display: 240x160 pixels, 15-bit color
- ROM Size: 16-32MB typical
- Save: Flash memory (up to 128KB)

**Emulator Development:**
- mGBA (recommended)
- VBA-M (alternative)

## Compilers & Toolchain

**Legacy Build (MODERN=0, default):**
| Tool | Purpose |
|------|---------|
| `agbcc` | Nintendo-matching GBA C compiler |
| `old_agbcc` | Older agbcc variant for specific files |
| `agbcc_arm` | ARM-mode variant |

**Modern Build (MODERN=1):**
| Tool | Purpose |
|------|---------|
| `arm-none-eabi-gcc` | Modern GCC for ARM |
| `arm-none-eabi-as` | ARM assembler |
| `arm-none-eabi-ld` | Linker |
| `arm-none-eabi-objcopy` | Binary manipulation |

**Toolchain Location:**
- agbcc: `pokefirered/tools/agbcc/` (installed from `/agbcc/` submodule)
- DevkitARM: `$DEVKITARM` environment variable (optional)

## Build System

**Core Files:**
- `pokefirered/Makefile` - Main build orchestration
- `pokefirered/config.mk` - Build configuration (version, revision, language)
- `pokefirered/make_tools.mk` - Tool building rules

**Build Targets:**
```bash
make                    # Build pokefirered.gba
make modern             # Build with arm-none-eabi-gcc
make compare            # Verify ROM matches original
make leafgreen          # Build LeafGreen variant
make firered_rev1       # Build FireRed 1.1
make clean              # Clean build artifacts
make tools              # Build helper tools
```

**Configuration Variables:**
| Variable | Default | Purpose |
|----------|---------|---------|
| `GAME_VERSION` | FIRERED | ROM variant (FIRERED/LEAFGREEN) |
| `GAME_REVISION` | 0 | Version (0=1.0, 1=1.1) |
| `GAME_LANGUAGE` | ENGLISH | Localization |
| `MODERN` | 0 | Use modern GCC (1=yes) |
| `COMPARE` | 0 | Verify against checksum |
| `O_LEVEL` | 2 | Optimization level |

**Output:**
- ROM: `pokefirered.gba` (or `pokeleafgreen.gba`)
- Map file: `pokefirered.map`
- Symbol file: `pokefirered.sym`
- Build directory: `build/<build_name>/`

## Build Tools (in pokefirered/tools/)

| Tool | Purpose | Source |
|------|---------|--------|
| `gbagfx` | Graphics conversion (PNG ↔ GBA) | C |
| `wav2agb` | Audio conversion (WAV → GBA) | C |
| `mid2agb` | MIDI conversion | C |
| `scaninc` | Include dependency scanning | C |
| `preproc` | C preprocessor extensions | C |
| `ramscrgen` | RAM script generation | C |
| `gbafix` | ROM header fixing | C |
| `mapjson` | Map JSON processing | C |
| `jsonproc` | JSON processing | C |
| `bin2c` | Binary to C array conversion | C |
| `rsfont` | Font conversion | C |

## Platform Constraints

| Constraint | Value | Impact |
|------------|-------|--------|
| **RAM** | 288KB total | Limits active data structures |
| **ROM** | ~16MB typical | Ample for code + question bank |
| **CPU** | 16MHz ARM7 | No runtime parsing (CSV→C at build) |
| **Display** | 240x160 | Question text must fit or paginate |
| **Save** | 128KB max | Mastery data must fit save structure |
| **Endianness** | Little-endian | Standard for ARM |

## External Dependencies (Submodules)

| Dependency | Location | Source |
|------------|----------|--------|
| **pokefirered** | `/pokefirered/` | https://github.com/pret/pokefirered |
| **agbcc** | `/agbcc/` | https://github.com/pret/agbcc |

## Development Environment Requirements

**Linux/WSL (Recommended):**
```bash
sudo apt install build-essential binutils-arm-none-eabi git libpng-dev
```

**macOS:**
```bash
brew install libpng
# + devkitPro for arm-none-eabi tools
```

**Windows:**
- WSL1/WSL2 (fastest, recommended)
- msys2 (2x slower than WSL)
- Cygwin (5-6x slower than WSL)

## Quiz System Stack (Project-Specific)

**Data Pipeline:**
```
CSV (GH 101 Question Bank - Sheet1.csv)
    ↓ validate_questions.py (planned)
    ↓ build_questions.py (planned)
C tables (questions_gen.h/.c)
    ↓ Makefile dependency
GBA ROM
```

**Key Files:**
- Question source: `GH 101 Question Bank - Sheet1.csv`
- Question format: Question_ID, Broad_Category, Sub_Topic, Question_String, Option_Label, Option_Text, Correct_Answer, Difficulty_Level, Explanation

## Custom Tools (Planned)

| Tool | Purpose | Status |
|------|---------|--------|
| `validate_questions.py` | Schema + length validation | Not yet implemented |
| `build_questions.py` | CSV → C table generator | Not yet implemented |

## Version Requirements

| Component | Version | Notes |
|-----------|---------|-------|
| **Python** | 3.9+ | f-strings, typing support |
| **agbcc** | Latest | From pret/agbcc |
| **DevkitARM** | r62+ | If using MODERN=1 |
| **GNU Make** | 4.0+ | Standard |
| **binutils-arm-none-eabi** | Latest | For assembler/linker |

---

*Stack analysis: 2026-01-30*
