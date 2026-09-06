# Mark 6 Design Standards

The project "constitution." Every module (Mechanical Pelvis Assembly first) must
comply. When a new part needs something not covered here, propose an
addition here rather than making a one-off local choice — see
`ECR_Process.md` if the addition would also require changing a locked
datum in `cad/master_reference/`.

## Servo standard

- **STS3215 class** (30kg and 19kg tiers, same case: 24.7×35.0×45.2mm) for
  all major joints; **FeeTech SCS0009** (23.5×12.0×29.0mm project-measured)
  for ears and tail.
- Integrated saddle + **removable heavy-duty U-clamp** retains the servo.
- **The servo is never the structural bearing.** Load path rule: if the
  answer to "where does this load go" is "through the servo," redesign the
  joint. Loads travel through printed pivots (trunnions/bushings); the servo
  only supplies torque, ideally via a broad drive flange/link, not a
  cantilevered horn tab.
- Every servo must be replaceable via one panel + a few screws — never by
  dismantling head/neck/torso/battery/electronics first.

## Pivot standard

- Large printed pivot pins, replaceable bushings, broad bearing surfaces,
  large fillets. Avoid tiny shafts carrying bending loads.
- Current rear-hip approach (PROVISIONAL, ChatGPT handoff): print-in-place
  fixed trunnion + captive sleeve. Ø16mm axle preferred, Ø14mm fallback,
  0.25–0.30mm radial clearance. No bearing, no separate steel axle needed.
- Hip system architecture: **PREFERRED, not LOCKED** (downgraded
  2026-09-06 — do not freeze until the actual mechanism is developed and
  physically validated). Working intuition: primary structural pivot →
  bearing support → sidestep axis → frontstep axis, upper servo mounted
  above/lower below, structural pivots independent of servo output shaft,
  modular rotate-to-service cassette. Shoulders would reuse this language
  once it's actually locked. Decide the real orientation from: shortest
  load path, bearing placement, serviceability, cable routing,
  printability, CG, and servo-output-bearing side-load — not from
  guessing a box orientation in a volume reservation (which is exactly
  what `cad/mechanical_pelvis/pelvis_alpha.py`'s servo envelope currently
  does, flagged there as a first guess).

## Fastener standard

- **M3 through-bolts + captured nuts**, project-wide. Avoid threaded
  plastic where practical. No heat-set inserts, threaded rod, or bearings
  currently in use anywhere in the project — introducing one is a new
  decision, not a continuation of anything already standardized.
- Brass-tube joints (legacy leg construction): 3× Ø3.35×12mm socket
  triangle, Ø3.0mm OD brass tube, running-clearance fit (not press fit).

## Clearance standard

- General PETG clearance: to be established from a physical test coupon on
  the Adventurer 5M (no PETG-specific clearance figure exists yet — the
  legacy `CLR_POCKET=0.25mm` figure was tuned for resin/MSLA, not FDM PETG.
  Don't reuse it un-verified).
- Hot-servo air gap (STS3215 runs 50–70°C under load): resin-era design used
  `AIRGAP=1.2mm` between case and material — re-verify this figure for
  PETG's higher heat tolerance rather than assuming it still applies as-is.
- Standardized tolerances live in **one** config (this project's
  `cad/master_reference/dimensions.py` for datums; a future
  `cad/*/clearances.py` for print-specific tolerances) — no per-part
  guessing.

## Print standard

- **Printer: FlashForge Adventurer 5M (FDM).**
- **Material: PLA / Strong-PLA for prototyping now; PETG for final parts**,
  once a design is physically confirmed on PLA first (Jon, 2026-09-06).
- No trapped supports, minimal support, self-supporting overhangs where
  possible, large fillets, consistent clearances, flat print surfaces.
- No print-in-place *structural* joints (the hip trunnion/sleeve is the one
  explicitly approved exception, since its whole point is print-in-place).
- Every servo must remain removable without damaging the print.
- Every part should state: print orientation, support requirement,
  estimated print time/weight, infill, wall count (per master-brief
  principle #3) — track this per-part once real STLs exist under `stl/`.

## Volume-first generation (2026-09-06, review 006/007 — permanent standard)

For any subsystem sharing space with a moving part (a limb, a spine
pivot, anything with a real motion envelope):

```
Available Structural Volume =
    Body Volume
    − Motion Envelope
    − Bearing Envelope
    − Servo Envelope
```

Generate structure *within* this volume (verified by boolean
intersection, a geometric guarantee) rather than authoring a shape and
checking it against the envelope afterward. Proven on the Mechanical
Pelvis Alpha (`cad/mechanical_pelvis/PELVIS_ALPHA.md`): identical
structural geometry to an author-then-check candidate, but measurably
less motion-envelope intrusion (28.7cm³ vs. 54.6/37.2/43.3cm³) purely from
generation order. Apply the same 5-step method to future subsystems that
share space with moving parts (shoulders, ribcage, neck, head, tail base):
define motion → generate motion envelope → reserve hardware → generate
allowable volume → design within it.

## Functional vs. character geometry (2026-09-06)

Every geometric feature is one of two kinds, and neither should compromise
the other:

- **Functional geometry** — load paths, articulation, mounting,
  serviceability, manufacturing. Governed by engineering (this document,
  the Master Reference, the volume-first method above).
- **Character geometry** — silhouette, visual identity, posture, K9
  recognition. Governed by the Character Definition
  (`docs/Design_Bible.md`'s Vision/Character sections).

A feature that can't be classified as one or the other, or that quietly
serves neither, should be questioned before it's added.

## Engineering zones (Jon, 2026-09-06)

Every module's internal structure separates into three zones. **Never allow
these to mix** — a wiring channel cut through a primary load-bearing wall,
or a servo boss doubling as a structural rib, is a design error, not a
style choice. This is what makes servicing tractable as the robot grows.

| Zone | Purpose | Owns |
|---|---|---|
| **A — Structural** | load paths only | shells, ribs, fillets, torque boxes |
| **B — Mechanical** | servos, pivots, bushings, maintenance | brackets, saddles, U-clamps, bushings, trunnions |
| **C — Utilities** | wiring, connectors, future sensors | wire channels, connector mounts |

Each module's `interfaces.json` entry states which of its `may_create` items
belong to which zone (see the `mechanical_pelvis_assembly` entry for the first example).
Adding a new zone-crossing feature to any module needs a documented reason,
not a shortcut.

## Naming standard

- Datum names (used in `dimensions.py`, `interfaces.json`, and every STEP
  label) are lowercase snake_case, mirrored joints suffixed `_L`/`_R`
  (e.g. `hip_L`, `shoulder_R`), stations prefixed `station:` (e.g.
  `station:S0-pelvis`).
- CAD source files: one module = one folder under `cad/`, named after the
  module (`cad/mechanical_pelvis/`, not `cad/MechanicalPelvis/` or a version-numbered
  variant).
- **No `_Final_Final_v3`-style files.** One source of truth per module; use
  git history and tags for versions, not filename suffixes.

## Revision / documentation rules

- Every value in `dimensions.py` (and any future per-module dimension file)
  carries a status tag: **LOCKED / ADOPTED / PROVISIONAL / PROPOSED / OPEN**
  (definitions in `cad/master_reference/Mark6_Master_Reference.md`). Never
  silently resolve a conflict between two tagged sources — state both and
  flag it.
- Changing a LOCKED datum requires an ECR (`docs/ECR_Process.md`), not a
  direct edit.
- Per-part documentation (once parts exist): purpose, version, weight,
  print orientation, material, hardware, assembly notes, known issues.

## Standing engineering principles (carried from the 2026-09-05 master brief)

Load path over servo · real anatomy sources joint locations · printing-first
· one-panel serviceability · fastener standardization · one design language
across shoulders/hips/legs/head · minimize unique parts, mirror L/R ·
weight distribution (battery lowest, compute central, head/tail lightest) ·
collision-check every revision · fully parametric (nothing hard-coded) ·
never solve a problem twice · print small before printing big · automatic
mass/print-time/CoM estimation per part · one tolerance config, referenced
everywhere · fixed module hierarchy, no module directly modifies another ·
one master assembly, no version-suffix files · stress-analysis reasoning
required on structural parts · cost tracking per revision · design as a
platform (reserve provision for future sensors/battery/compute upgrades).
