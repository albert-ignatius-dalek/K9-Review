# Project Axioms

## Purpose

The irreducible philosophical foundation of the Mark series. Not
standards (`docs/Design_Standards.md` — how ideas get implemented). Not
engineering decisions (`Engineering_Decisions.md` — a log of specific
choices and why). Just the truths everything else is built on.

```
PROJECT_AXIOMS.md   — what truths define Mark 6?
        │ defines
        ▼
Design_Bible.md      — given those axioms, how do we design Mark 6?
        │ implements
        ▼
Design_Standards.md  — exactly how do we implement those ideas?
        │ constrains
        ▼
Subsystem documents
```

## Seven Axioms

1. We are building the actual K9 character using modern engineering.
2. Movement creates appearance.
3. Biomechanics define allowable structure.
4. Geometry is generated from validated constraints.
5. Biology informs behaviour. Engineering implements behaviour.
6. Protect the architecture.
7. Every subsystem should simplify the next subsystem.

## Change Policy

These axioms represent the philosophical foundation of the Mark series.
They should only be modified when overwhelming engineering evidence
demonstrates that an axiom no longer serves the project's purpose. New
axioms should be added only if they are fundamental, broadly applicable,
and cannot be derived from the existing set.

## Revision History

- **2026-09-06** — formalized, at the point the project's engineering
  philosophy stabilized (independent design review's "Architectural
  Stability Phase 1" — the philosophy and methodology becoming internally
  consistent, not the robot nearing completion). Moved out of
  `docs/Design_Bible.md`'s "Core principles" section rather than
  duplicated there, per this project's own configuration-management rule.
