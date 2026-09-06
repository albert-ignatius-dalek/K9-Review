# REVIEW REQUEST #1 — Mechanical Pelvis Alpha

**Milestone**: first structure generated *within* a validated motion
envelope, rather than authored freely and checked against one afterward.

## Files changed (this snapshot)

- `STL/Mechanical_Pelvis_Alpha_v1.stl` — the new Alpha geometry
- `STL/Manufacturable_Structural_Core.stl` — the allowed-volume bound
  Alpha was generated inside
- `STEP/Candidate_A_TorqueBox.step`, `Candidate_B_SemiMonocoque.step`,
  `Candidate_C_SpaceFrame.step` — the three prior candidates Alpha's
  approach is compared against
- `STEP/Mark6_Master_Reference.step` — the datum skeleton everything else
  is built from
- `Hardware_Manifest.md`, `Biomechanics_Reference.md` — new this
  milestone (see `Engineering_Decisions.md` for why)

## Headline result

| | intrusion into the sitting-pose motion envelope |
|---|---|
| Candidate A — torque box | 54.6 cm³ |
| Candidate B — semi-monocoque | 37.2 cm³ |
| Candidate C — space-frame | 43.3 cm³ |
| **Alpha (generated within the bound)** | **28.7 cm³** |

Alpha uses the *same* structural shape as Candidate A (same loft, same
wall thickness) — the improvement comes entirely from generation order
(intersect with the allowed volume before adding hardware features,
rather than checking afterward), not from a cleverer shape.

## Questions for review

1. Is 28.7 cm³ of residual intrusion (from the hip bosses/spine clevis
   legitimately reaching toward the hip axis) an acceptable floor, or
   does the hardware-mounting geometry itself need rethinking?
2. Candidate C remains under consideration but has no stiffness
   evaluation yet (Bredt-Batho doesn't fit its topology) — what method
   should replace it?
3. Sitting/lying/crouch/play-bow angles are currently a design choice
   (a target posture, not a biomechanical replication) — does the
   incoming terrier biomechanics study change any of the current
   PROPOSED sitting-pose angles?

## Known risks

- Alpha is a mesh-boolean result, not reconstructed parametric CAD — no
  STEP export exists for it yet.
- Servo box orientation in the reserved envelope is a first guess, not
  the finalized stacked sidestep+frontstep arrangement.
- No fastener clearance reservation yet (no real bolt layout exists).

## Unknowns (see `Hardware_Manifest.md` for the full list)

- Main battery physical dimensions/connector/BMS status — capacity
  confirmed (12V 3S2P 5000mAh), everything else still needs measuring.
- Front-leg motion envelope not yet built (hind leg only so far).
- Lying/crouch/play-bow motion envelopes not yet built (standing/walking/
  sitting only).

Please review.
