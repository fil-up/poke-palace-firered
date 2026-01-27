# Features Research

## Educational Game Features (Quiz-Based)

### Table Stakes (Must Have)
| Feature | Description | Priority |
|---------|-------------|----------|
| **Question display** | Show question during encounter | ✓ Exists |
| **Answer selection** | Map moves to A/B/C/D choices | ✓ Exists |
| **Immediate feedback** | Correct/wrong result affects battle | ✓ Exists |
| **Progress persistence** | Mastery survives save/load | Critical |
| **Completion tracking** | Know which questions answered | Critical |
| **Species clearing** | Mastered species don't reappear | Critical |

### Differentiators (Competitive Advantage)
| Feature | Description | Priority |
|---------|-------------|----------|
| **Capture quiz flow** | Perfect run to catch Pokémon | High |
| **Explanation display** | Learn why answer is correct | High |
| **Topic-based progression** | Categories map to areas | Medium |
| **Difficulty scaling** | Harder questions on later routes | Medium |
| **Spaced repetition** | Trainer reinforcement | Stretch |
| **Gym section exams** | Topic mastery gates | Stretch |

### Anti-Features (Deliberately NOT Building)
| Feature | Why Not |
|---------|---------|
| Traditional battles | Defeats the learning purpose |
| Catching with Pokéballs | Replaced by quiz capture |
| Wild Pokémon damage variance | Fixed half-HP penalty is cleaner |
| Move effectiveness | Irrelevant in quiz mode |
| Status conditions | Adds complexity without learning value |
| Items in battle | Keeps focus on questions |

## Feature Complexity Assessment

| Feature | Implementation Complexity | Context Usage |
|---------|---------------------------|---------------|
| Question bank loading | Medium | ~15% |
| Save persistence | Medium-High | ~20% |
| Capture quiz flow | Medium | ~15% |
| Cleared species filter | Low-Medium | ~10% |
| Multi-page text | Low | ~10% |
| Explanation display | Low | ~10% |
| Gym exams | High | ~25% |
| Trainer reinforcement | Medium | ~15% |

## Feature Dependencies

```
Question Bank System
    ↓
Save Persistence ←─────┐
    ↓                  │
Mastery Tracking ──────┤
    ↓                  │
Capture Quiz Flow      │
    ↓                  │
Cleared Species Filter │
    ↓                  │
Route 1 Complete ──────┘
    ↓
Multi-Route Expansion
    ↓
Gym Exams (stretch)
    ↓
Trainer Reinforcement (stretch)
```

## MVP Feature Set

Based on "Route 1 through Pewter Gym" scope:

| Phase | Features |
|-------|----------|
| Foundation | Question bank, CSV pipeline, build tools |
| Core Loop | Mastery tracking, save persistence |
| Capture | Capture quiz flow, species clearing |
| Polish | Multi-page text, explanations |
| Content | Route 1, Viridian Forest, Pewter |
| Stretch | Gym exam, trainer reinforcement |
