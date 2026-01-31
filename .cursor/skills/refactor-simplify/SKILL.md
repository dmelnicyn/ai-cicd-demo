---
name: refactor-simplify
description: Reduce complexity and improve readability while preserving behavior (tests must stay green).
---

# Refactor & Simplify Skill

This skill is for turning “it works but it’s messy” into “it works and it’s boring.” The goal is to reduce complexity and increase maintainability without changing external behavior.

## When to Use

- After implementing a feature and the solution feels over‑engineered.
- When review feedback points to complexity, duplication, or unclear structure.
- When code is hard to test, reason about, or modify safely.
- Before release hardening to reduce risk and simplify operations.

## Guardrails

- Behavior must not change unless explicitly requested.
- Tests must stay green (add or adjust tests to lock behavior before refactoring if needed).
- Prefer small, reversible commits and incremental refactors.
- Avoid “clever” patterns. Prefer boring, obvious code.

## Refactor Strategy

1. Define the target and constraints
- Identify which module(s) or paths to refactor.
- Confirm constraints (no API changes, no DB schema changes, performance constraints, etc.).
- If unclear, ask questions.

2. Lock behavior
- Run tests; add missing tests around the riskiest or most complex logic.
- If tests don’t exist, create minimal characterization tests.

3. Measure and localize complexity
- Identify hotspots:
  - deeply nested conditionals
  - long functions (> ~50 lines)
  - duplicated logic
  - unclear responsibilities / mixed concerns
  - overly generic abstractions
- Prefer refactoring one hotspot at a time.

4. Simplify first, then structure
- Simplify logic:
  - early returns
  - small helpers with clear names
  - remove dead code / unused parameters
  - collapse unnecessary layers
- Structure:
  - improve module boundaries
  - extract cohesive units
  - reduce cross‑module coupling

5. De‑duplicate and clarify
- Replace repeated blocks with a single well‑named function.
- Normalize data shapes and naming.
- Replace “magic numbers/strings” with constants (only when it improves clarity).

6. Re‑run quality gates
- Tests, lint, type checks, build.
- If the repo has a tooling matrix, follow it.

7. Document the change
- Add a short “Refactor Notes” section in the PR/commit message describing:
  - what changed structurally
  - what did not change (behavior)
  - any trade‑offs or remaining known debt

## Heuristics for “Simpler”

- Prefer:
  - fewer indirections
  - fewer abstractions
  - fewer files touched
  - smaller public surface area
- Avoid:
- premature generalized frameworks
- complicated dependency injection for small apps
- unnecessary inheritance / “design pattern cosplay”