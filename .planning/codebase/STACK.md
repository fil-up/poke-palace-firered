# Stack

## Languages

| Language | Usage | Version/Notes |
|----------|-------|---------------|
| **C** | Primary game code | C89/GNU89 (via agbcc) |
| **ARM Assembly** | Low-level routines, boot code | ARM7TDMI instruction set |
| **Python** | Build tools, validators | Python 3.x |
| **Make** | Build system | GNU Make |

## Compilers & Toolchain

| Tool | Purpose |
|------|---------|
| **agbcc** | Nintendo-compatible GBA C compiler (legacy, non-modern builds) |
| **arm-none-eabi-gcc** | Modern GCC toolchain (optional MODERN=1 builds) |
| **DevkitARM** | Alternative ARM toolchain |
| **arm-none-eabi-as** | Assembler for ARM assembly |
| **arm-none-eabi-ld** | Linker |
| **arm-none-eabi-objcopy** | Binary manipulation |

## Build System

- **GNU Make** with modular .mk includes
- Build targets: `make rom`, `make validate`, `make clean`
- Output: `pokefirered.gba` (Game Boy Advance ROM)
- Separate build directories per configuration

## Platform Constraints

| Constraint | Value |
|------------|-------|
| **Target Hardware** | Game Boy Advance (ARM7TDMI, 16MHz) |
| **RAM** | 32KB IWRAM + 256KB EWRAM |
| **ROM Size** | 16-32MB typical |
| **Save Data** | Flash memory (up to 128KB) |
| **Display** | 240x160 pixels, 15-bit color |

## External Dependencies

| Dependency | Purpose | Location |
|------------|---------|----------|
| **agbcc** | GBA C compiler | `/agbcc/` (submodule) |
| **pret/pokefirered** | Base decomp | `/pokefirered/` (submodule) |

## Custom Tools (Planned)

| Tool | Purpose | Status |
|------|---------|--------|
| `validate_questions.py` | Schema + length validation | Not yet implemented |
| `build_questions.py` | JSON → C table generator | Not yet implemented |
