# K9 Mark 6 — Engineering Review Mirror

This is a **curated public snapshot** of milestone-quality mechanical
engineering work from the Mark 6 (K9) project — not a live development
repository. Full development history, in-progress experiments, and dead
ends stay in a private repository; only work that's reached review-ready
status lands here.

**Scope, deliberately incomplete**: mechanical engineering only. Software
architecture, the K9 personality/behavior model, and anything considered
a novel part of the project are intentionally not included here.

## Contents

- [`REVIEW.md`](REVIEW.md) — the current review request: what changed,
  open questions, known risks and unknowns.
- [`Engineering_Decisions.md`](Engineering_Decisions.md) — a running log
  of major decisions and why they were made.
- [`Design_Bible.md`](Design_Bible.md), [`Design_Standards.md`](Design_Standards.md),
  [`Hardware_Manifest.md`](Hardware_Manifest.md), [`Biomechanics_Reference.md`](Biomechanics_Reference.md)
  — the current state of the project's primary references.
- `STEP/`, `STL/` — exported geometry for the current milestone
  (Mechanical Pelvis Alpha and the three architecture candidates it
  superseded).

## How to use this if you're reviewing

Read `REVIEW.md` first — it names the specific questions this milestone
needs answered. `Engineering_Decisions.md` has the reasoning behind
anything that looks like it needs justifying. The STEP files are the
authoritative geometry; STL is provided for quick visual inspection.
