# Biomechanics Reference

Per the Canine Biomechanics Study Programme (2026-09-06): as important a
document as `Design_Bible.md`, not a subordinate one. Mark 6 is designed
from motion — every external shape should exist because the underlying
biomechanics require it, not the reverse.

**Status: scaffold only.** The research itself (veterinary gait resources,
kinematic studies, real-terrier video references, motion-capture data) is
coming from a dedicated ChatGPT study — this file defines the structure it
lands in, so each behaviour has one canonical place from day one rather
than being retrofitted into shape later.

## Per-behaviour entry template

```markdown
### <Behaviour name>

**Video references**: side / front / rear / top (if available)

**Mechanical observation**: what the dog is actually doing, mechanically
(e.g. "sitting: pelvis rotates rearward, femurs rotate forward and
abduct, stifles fold tightly, hocks fold underneath, body descends
between the legs, rear of pelvis becomes the support point").

**Affected joints**: which joints move, and how much (ROM, not just
direction).

**Engineering translation**: what functional requirement this creates for
Mark 6's hardware (e.g. "requires the hip ab/adduction axis to
contribute to sitting, not just the pitch axis").

**Packaging implications**: what volume this behaviour occupies/frees,
and where (feeds `cad/skeleton_study/motion_envelopes.py`).

**Structural implications**: what this demands of load paths /
serviceability / part boundaries.

**Collision implications**: what this behaviour must stay clear of
(other limbs, torso, ground, tail).

**Visual implications**: what this contributes to reading as
"authentically canine" per the Design Bible's final design test.
```

## Canonical motion set (per the study programme — target coverage)

Neutral stand · Relaxed stand · Walk · Trot · Slow approach · Stop · Sit ·
Lie · Play bow · Turn · Look around · Step over obstacle · Climb step ·
Sniff ground · Wait · Tail wag · Greeting

**Coverage so far**: Standing (`cad/skeleton_study/leg_kinematics.py`,
validated against real locked data), Sitting (`cad/skeleton_study/poses.py`,
PROPOSED target posture — see `TECHNICAL_DEBT.md` TD-008), Walking
(`cad/skeleton_study/motion_envelopes.py`, a PROPOSED ±20° stride sweep,
explicitly not real gait data). Everything else on the canonical list is
not yet started.

## How this feeds the CAD

Every entry here should be traceable to a concrete artifact:
`cad/skeleton_study/envelope_<behaviour>.stl` (the swept occupied-space
volume) and, once real target angles exist for that behaviour, an update
to `leg_kinematics.py`/`motion_envelopes.py`'s pose parameters. A
behaviour entry with no corresponding envelope file is documented but not
yet engineered — don't let this file get ahead of the CAD it's supposed to
justify.
