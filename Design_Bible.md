# Mark 6 Design Bible

**Document priority (2026-09-06 review, [`PROJECT_AXIOMS.md`](../PROJECT_AXIOMS.md)
added above the original five as the project matured): Project Axioms,
Master Reference, Design Bible, Biomechanics Reference, Interface
Definitions (`cad/master_reference/interfaces.py`), Manufacturing
Standards (`Design_Standards.md`) — these six collectively define the
robot.**

An index, not a duplicate. Configuration-management rule for this project:
**every piece of information has exactly one canonical location.** This file
routes to that location for each topic instead of re-authoring content that
already lives somewhere else — a second copy is how a project ends up with
two answers to the same question.

| # | Section | Canonical home |
|---|---|---|
| 01 | Vision | written below (nothing else owns it); see also [`Biomechanics_Reference.md`](Biomechanics_Reference.md) (as important as this file, not subordinate to it) |
| 02 | Engineering Principles | [`Design_Standards.md`](Design_Standards.md) — "Standing engineering principles" |
| 03 | Robot Architecture | written below (nothing else owns it) |
| 04 | Dimensions | [`cad/master_reference/dimensions.py`](../cad/master_reference/dimensions.py) + [`Mark6_Master_Reference.json`](../cad/master_reference/Mark6_Master_Reference.json) |
| 05 | Interfaces | [`cad/master_reference/interfaces.py`](../cad/master_reference/interfaces.py) + [`interfaces.json`](../cad/master_reference/interfaces.json) |
| 06 | Standards | [`Design_Standards.md`](Design_Standards.md) |
| 07 | Hardware | [`Design_Standards.md`](Design_Standards.md) (servo/fastener standard) + [`BOM.md`](BOM.md) (servo/electronics inventory and placement) |
| 08 | Materials | [`Design_Standards.md`](Design_Standards.md) — "Print standard" |
| 09 | CAD Rules | [`Design_Standards.md`](Design_Standards.md) — "Naming standard" / "Revision rules"; [`README.md`](../README.md) — source-vs-generated split |
| 10 | Print Rules | [`Design_Standards.md`](Design_Standards.md) — "Print standard" |
| 11 | Assembly Rules | not yet written — no assemblies exist yet (single-module project so far) |
| 12 | Wiring Rules | not yet written as a standard; one real constraint exists in [`Hardware_Manifest.md`](Hardware_Manifest.md) (Waveshare bus driver's ~5A/header limit) |
| 13 | Maintenance Rules | [`Design_Standards.md`](Design_Standards.md) — servo standard's serviceability requirements |
| 14 | Future Expansion | written below (nothing else owns it) |
| 15 | Lessons Learned | [`TECHNICAL_DEBT.md`](../TECHNICAL_DEBT.md) for CAD/engineering; [`K9_Software_Migration_Report.md`](K9_Software_Migration_Report.md) §5 for real, quoted software lessons from the previous K9 project |
| 16 | Systems Architecture | [`Systems_Architecture.md`](Systems_Architecture.md) — the 13 robot systems, the K9 Behaviour Engine layer, hardware-to-system cross-reference |
| 17 | Software Architecture Research | [`Software_Architecture_Research.md`](Software_Architecture_Research.md) — ROS2/Nav2/SLAM/whole-body-control background, not a commitment |
| 18 | Previous Software Migration | [`K9_Software_Migration_Report.md`](K9_Software_Migration_Report.md) — full audit of the prior K9 project (voice/personality/AI/control), what migrates vs. redesigns vs. is abandoned |

---

## 01. Vision

> **The success of Mark 6 will not be measured by how many features it
> has, but by how coherent it is. Every subsystem should reinforce every
> other subsystem until the robot feels inevitable — as though it could
> not have been engineered any other way.**
>
> — ChatGPT, independent design review, 2026-09-06

**The goal is not to finish quickly. The goal is to build the best
open-source quadruped robot we can. Every subsystem should be good enough
that we'd be proud to reuse it in Mark 7.** (Jon, 2026-09-06) — this is the
standard a module is measured against before it merges to `main`, not just
"does it work."

**Core principle (2026-09-06, supersedes the earlier framing): the goal is
not a robot that *looks like* a real dog — it's a robot that *behaves*
like one. Appearance is the consequence of correct biomechanics, not a
separate target.** Mark 6 is engineering a mechanically faithful terrier
skeleton; servos, pivots, printed parts, electronics, and armour are the
engineering solution for reproducing that skeleton's function, not a
"robot" that gets styled canine afterward.

Design order (skeleton-first, not the reverse):
`canine anatomy → canine biomechanics → robot skeleton → joint locations →
motion envelopes → internal packaging → structure → armour → surface
details`. Not: pretty robot → find somewhere for the servos.

**Every joint must answer three questions before approval**: which real
canine joint does this represent? What motion does the real joint perform?
Why is this range required? A joint that can't answer these should be
reconsidered, not approved on "it works."

**Biology is the starting point, not a straitjacket** (Jon, 2026-09-06,
formalized as a project-wide directive): start from biology where it
improves function, then deliberately depart from it where hardware
requirements become insurmountable or where engineering provides a
demonstrably better solution. A departure from anatomy needs the same
justification a joint needs — state why, don't silently drift from either
biology or engineering convenience. **Biology is the reference; engineering
is the implementation** — this does not replace the established hardware
(STS3215/SCS0009 servos, two-axis hip architecture, two spine pivots,
modular PETG, U-clamp retention, serviceable cassettes, the master
reference datum system, the GitHub workflow); it governs how that hardware
gets used.

**Every subsystem answers two independent questions before it's
complete**: *Mechanical* — is this the strongest, simplest, most
serviceable implementation using our established hardware? *Biomechanical*
— would a knowledgeable observer recognize this as moving like a real
terrier? Both, not one traded for the other.

**Character direction (Jon, 2026-09-06): Mark 6 represents K9 after
thousands of years of accumulated experience. His movement is deliberate
because it reflects confidence, efficiency, and mature judgment — not
mechanical limitation. When circumstances require it, he remains fully
capable of decisive action.** Not aged or worn — exceptionally long-lived,
highly maintained, like a perfectly preserved vintage aircraft still fully
operational. His movement should read as: observe → calculate → decide →
act, not impulsive. **Behavioural rule for future gait/motion-planning
work**: default movement is calm, economical, confident, purposeful,
precise (e.g. a slow, deliberate walk up to a stop beside someone in
normal conversation); rapid acceleration and fast obstacle avoidance are
reserved for situations that genuinely demand it (e.g. an emergency/
protective response). The audience should read this as *choosing* not to
waste motion, never as *can't* move quickly. This should inform the
WALKING motion envelope's stride parameters in `cad/skeleton_study/`
once gait/software work starts — a deliberate default pace, not a
generic fast gait, with a distinct higher-speed mode for genuine need.

**Character philosophy (2026-09-06, character research): K9 is not a
dog with a computer, nor a computer pretending to be a dog — he is an
artificial person whose chosen physical embodiment is canine.** Multiple
physical Mark units have existed canonically across a consistent
personality: the body is replaceable, the character is not. He behaves
professionally, not emotionally — dependable, not cold; rarely
hesitates or apologizes; reports conclusions rather than opinions.
**Build professional priorities, not robot emotions**: owner safety →
mission → knowledge → efficiency → etiquette, with emotion as emergent
behaviour from that hierarchy, never a simulated feeling-state bolted on
separately. Humour emerges from literal interpretation and deadpan
certainty, never forced jokes. Affection is earned through action, never
sought. Voice cadence (short, precise, considered) matters more than
voice content. **Governing rule for all AI/behaviour work: Mark 6 should
never become more human — it should become more K9.** Richer perception,
better movement, and deeper understanding are means to that end, not a
drift toward generic conversational-AI behaviour. See
`docs/K9_Software_Migration_Report.md` for the full character research
and the previous K9 software project's audit this is built on.

**Final design test**: rendered in plain grey CAD with no textures, would
someone watching it move immediately think "that moves like a real dog"?
Static standing-pose appearance is not sufficient — validate in standing,
walking, trotting, sitting, lying, play bow, turning, climbing, and maximum
crouch. If a subsystem only looks right standing still, the design is
incomplete, not finished. See `cad/skeleton_study/` for the first
concrete step: a bare articulated skeleton (no armour) proving natural
standing/sitting/crouch/play-bow before armour gets rebuilt around it.

A real, manufacturable engineering project, not concept art. Appearance
target: Doctor Who K9 + modern military robotics + Boston Dynamics +
industrial machinery, reading unmistakably as a real dog (deep chest,
narrow waist, compact rear, strong shoulders, muscular thighs, digitigrade
legs) — never boxy, humanoid, skeletal, cartoonish, or over-designed. The
standing rule: **never simplify the robot because CAD is hard — improve the
engineering until it supports the approved appearance instead.** Evaluate
every decision as if preparing for commercial kit manufacture: function,
plus ease of printing, assembly, maintenance, repeatability, part
standardization, and future upgrades — closer to a Unitree/Boston-Dynamics
-grade product than a merely-functional build.

## 03. Robot Architecture

Fixed module hierarchy — no module directly modifies another, every module
attaches to `cad/master_reference/` datums instead of a neighboring module's
geometry:

```
Robot
├── Head / Neck
├── Front Torso / Mid Torso / Rear Torso
├── Spine (2 pivots, 4× STS3215)
├── Shoulders  ── Front Legs
├── Hips       ── Rear Legs        (= Mechanical Pelvis Assembly, first subsystem)
├── Tail (2× SCS0009)
├── Electronics (Jetson / battery / servo bus, removable trays)
├── Armour (segmented, removable plates)
└── Hardware (fasteners, standardized across all of the above)
```

Servo standard throughout: STS3215 (30kg/19kg tiers) for major joints,
SCS0009 for ears/tail. Load path never runs through a servo shaft — see
`Design_Standards.md`.

### Packaging architecture (ENGINEERING REVIEW 003, approved 2026-09-06)

The spine is the primary structural backbone (torsional/bending loads,
spine pivots, torso interfaces, cable trunk). Equipment is a passenger,
never structural. Each torso section owns an independent, removable
equipment cassette; nothing bridges an articulated spine joint except
flexible wiring routed through the backbone's own cable trunk (Zone C).
Suspended modules taper (wedge) toward their adjacent spine joint so they
stay outside that joint's articulation envelope — generated by sweeping the
joint's real max pitch/yaw/roll, not assumed by eye. **Blocked:** no spine
ROM (pitch/yaw/roll travel in degrees) exists anywhere in the source data
yet, so this sweep can't be generated for real until that number exists —
see `TECHNICAL_DEBT.md` TD-007.

**Design standard (verbatim, add-only, do not paraphrase):**

> All suspended equipment modules shall be attached only to their parent
> torso section. No equipment module shall span an articulated spine
> joint. Suspended modules shall conform to the articulation envelope of
> their parent section using tapered or wedge-shaped geometry where
> required.

Approved section ownership: **Front torso** — Jetson Nano, Pico, cooling,
camera interface, future USB expansion. **Middle torso** — main battery,
power distribution, voltage regulators, main fuse (lowest, below the
backbone, for CG). **Mechanical Pelvis Assembly** — rear servo controller, rear
wiring hub, tail controller, rear power distribution *only* — no Jetson or
battery volume reserved here, full stop.

Underside treatment: a faceted "keel" belly (not a flat compartment) built
from the same per-section removable trays, following the approved exterior
profile — chosen because it produces the articulation-clearing wedge shape
and the approved appearance from the same geometry, not two competing
constraints.

### Sitting posture & skeleton-first packaging (ENGINEERING DIRECTIVE 008, 2026-09-06)

**Design standard (verbatim, add-only, do not paraphrase):**

> The robot's sitting posture shall be derived from canine biomechanics
> rather than simple joint rotation. The articulated hips shall allow the
> pelvis to descend naturally between the folded rear limbs. Additional
> torso packaging volume created by this articulation shall be considered
> usable engineering volume rather than empty space.

Supersedes the earlier assumption (visible in prior concept art only, never
an engineering reference) of the torso staying suspended above folded legs.
The legs define the available anatomical volume; the torso occupies it —
design from the skeleton outward, not the other way around. See
`cad/skeleton_study/` for the articulation study this requires before the
Mechanical Pelvis Assembly is frozen.

### Core principles — see PROJECT_AXIOMS.md

The project's seven governing ideas now live in
[`PROJECT_AXIOMS.md`](../PROJECT_AXIOMS.md) at the repo root — elevated
there 2026-09-06 as irreducible, almost-never-change truths, distinct
from this file's more operational content. Not restated here to avoid
two answers to the same question. Mark 6's coherence comes from having
*fewer* governing ideas as it matures, not more — resisting an eighth
axiom unless it truly earns its place. Validate methodology before
geometry, architecture before optimization, evidence before standards.

### Decision framework: protecting the architecture (2026-09-06)

Past a certain maturity, the project stops being about "designing the
robot" and starts being about protecting an established architecture.
Every major decision should satisfy three questions:

1. **Does this preserve the established architecture?** Fits the
   biomechanics-first methodology, preserves modularity, respects the
   Master Reference.
2. **Does it improve the robot as K9?** More believable movement, better
   posture, better interaction, better character.
3. **Will this make Mark 7 easier rather than harder?** Reusable, general,
   reduces technical debt rather than adding it.

If the answer is yes to all three, it's probably right. **When two
competing solutions exist, prefer the one that simultaneously improves
biomechanics, engineering, manufacturability, and the K9 character** —
even if harder to design — over one that trades one for another. These
are the decisions that compound instead of creating trade-offs (example:
anatomically correct sitting → more packaging volume → better battery
placement → better CG → better gait → stronger character — one decision
solving six problems).

Unknowns are assets, not liabilities, at this stage: not knowing the final
battery placement, the A/B/C stiffness winner, or the exact electronics
packaging is deliberately preserved open engineering question, not a gap
to paper over with a guess before the evidence exists.

## 14. Future Expansion

Reserve mounting provision for future sensors/LiDAR/stereo cameras, a
bigger battery, an alternate Jetson or servo model, and accessory payloads
— so the chassis can evolve without a full redesign. This is a design
constraint on every module (`may_create` in that module's `interfaces.json`
entry should leave room for it), not a separate future task.
