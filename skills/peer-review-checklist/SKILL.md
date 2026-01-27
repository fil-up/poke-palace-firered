# Peer Review Checklist

A checklist-first review skill for consistent, high-signal PR feedback.

## When to Use

- Reviewing pull requests or patches
- Verifying correctness, safety, and maintainability
- Producing a clear review summary and decision

## Core Checklist

### Logic & Correctness
- Edge cases handled (nulls, empties, bounds)
- Behavior matches intent and spec
- Failure paths are safe and predictable

### Security & Safety
- Inputs validated and sanitized
- No secrets or sensitive data exposed
- Dangerous patterns avoided (eval, shell, unsafe deserialization)

### Performance & Scalability
- No obvious O(n^2) or unbounded loops on hot paths
- Expensive work guarded or cached where appropriate
- Avoids unnecessary allocations or repeated I/O

### Tests & Observability
- Tests cover key paths and edge cases
- Tests are deterministic and readable
- Logs/metrics are meaningful and not noisy

### Maintainability
- Clear naming and structure
- Complex logic has small helpers or comments
- Public APIs are documented or self-explanatory

## Severity Labels

- [blocking] Must fix before merge
- [important] Should fix soon; avoid merge if possible
- [suggestion] Improvement; non-blocking
- [nit] Minor clarity/consistency; non-blocking

## Review Output Template

```
## Summary
[Brief summary of what changed and overall assessment]

## Required Changes
- [blocking] ...

## Important Fixes
- [important] ...

## Suggestions
- [suggestion] ...

## Nits
- [nit] ...

## Questions
- ...

## Verdict
Approve / Request changes
```

## What Not to Focus On

- Formatting (use automated linters/formatters)
- Purely cosmetic refactors with no behavioral impact
- Generated files unless they introduce issues

## Notes

Aim for high-signal feedback: specific, actionable, and scoped to the PR.
