# Systems Architecture

## The Behaviour Layer — what makes hardware recognizably K9

**Not "the K9 Behaviour Engine"** (renamed 2026-09-06, review 013): a
single "Engine" implies one executable module, and this is more likely a
collection of cooperating behaviours — Attention, Conversation,
Locomotion Style, Body Language, Interaction, Task Arbitration, Idle
Behaviour, Recovery, Social Behaviour — that may never naturally collapse
into one component. **The Behaviour Layer** names the architectural
position without presupposing the implementation:

```
Character           (who K9 is — docs/Design_Bible.md, PROJECT_AXIOMS.md)
     │
Behaviour Layer      (how K9 acts it out — cooperating behaviours, not one engine)
     │
Decision Making      (owner safety → mission → knowledge → efficiency → etiquette)
     │
Robot Systems        (the 13 systems below)
     │
Hardware
```

Character sits at the top, hardware at the bottom — deliberately the
reverse of how most robotics projects evolve. The Behaviour Layer
arbitrates movement style, speech timing, attention, priorities,
interaction, posture, confidence, and curiosity across every system
below it. Whether any given cooperating behaviour ends up built as a
behaviour tree, hierarchical state machine, or planner is an
implementation detail — architecturally it's its own layer, not
something Locomotion or Speech does on their own.

**Governing rule, elevated from implementation to architecture (review
013)**: **K9 speaks intentionally, never accidentally.** The
implementation of this rule may evolve (currently: an explicit arm-gate
before any heard query reaches the brain, per
`K9_Software_Migration_Report.md` §2/§4); the rule itself should not.

**Behavioural Ethology Study** — the fourth research stream (renamed from
"Embodied Behaviour," review 013): study K9 the way an ethologist studies
an animal — catalogue Approach, Observe, Wait, Guard, Escort, Search,
Think, Follow, Recharge, Idle as behavioural specifications, not software
requirements. Not started. Result should populate
`docs/K9_Behaviour_Reference.md` (scaffolded, see below).

**Reusable behaviour states** (Behaviour-Layer-level, coordinate multiple
systems each — status: none implemented, this is the target vocabulary):
Investigate, Return Home, Follow, Guard, Observe, Escort, Search, Wait,
Greet, Patrol, Recharge, Standby, Shutdown, Wake. Supersedes the shorter
placeholder list in the Character System entry below.

---

The Hardware Manifest answers "what is this component?" This document
answers **"why does it exist?"** — every major hardware choice should
trace to at least one system below. A component that supports no system
here is a candidate for removal, not a curiosity.

**Status honesty**: most systems below are still HARDWARE-ONLY or
UNSTARTED on the software side. This is a taxonomy to organize future
work against, not a claim that Locomotion/Vision/Speech/etc. already
work. `docs/K9_Software_Migration_Report.md` (complete, 2026-09-06) has
now assessed what's real from the previous K9 software project for
Speech, Character, and Learning specifically — those three sections
below are updated accordingly; Locomotion/Balance/Navigation/Vision/
Safety/Health/Power/Thermal/Communications/Maintenance have no prior
software asset and start from zero, confirmed rather than assumed.

Each system: Purpose, Inputs, Outputs, Hardware, Software, Dependencies,
Failure modes, Future expansion — fields left TBD stay TBD rather than
being filled with plausible-sounding invention. Per the roadmap's own
Priority 1 ("do not increase detail simply because detail is possible"),
richness of the template doesn't mean every cell gets content before
the underlying architecture actually exists.

## Training vs. Learning — a distinction the project now makes explicit

- **AI Training** — large, pretrained models (speech, vision, language).
  Not something Mark 6 does; something Mark 6 *uses*.
- **Robot Learning** — specific to this physical K9: owner preferences,
  home map, daily routine, names, voice recognition, favourite locations,
  allowed rooms, charging routine, behaviour refinement. This is what
  Mark 6 should actually improve over time. See System 7 below.

## 1. Locomotion System
**Purpose**: move K9 safely and naturally.
**Inputs**: gait command, terrain state (future).
**Outputs**: joint angle targets to the servo bus.
**Hardware**: STS3215 servos, Mechanical Pelvis Assembly, spine, legs,
IMU (Waveshare Sense HAT), foot contact sensing (RESERVED, no component
selected).
**Software**: IK, gait generation, balance, recovery, slip detection,
terrain adaptation. **Status: unstarted** — the current motion-envelope
work (`cad/skeleton_study/`) is mechanical packaging, not a gait
controller. No prior software asset exists (previous K9 project was
wheeled — `K9_Software_Migration_Report.md`).
**Dependencies**: Balance System (shares IMU, cannot be validated
independently).
**Failure modes**: TBD — no controller exists to analyze yet.
**Future expansion**: TBD.

## 2. Balance System
**Purpose**: maintain stability — deliberately its own system, not folded
into Locomotion.
**Inputs**: IMU, joint encoders (none yet — servos are position-commanded,
not confirmed to report back), body orientation, foot contact (future),
battery CG.
**Outputs**: weight shift, step correction, recovery, sit recovery, fall
prevention.
**Hardware**: same IMU as Locomotion; no dedicated hardware yet.
**Software**: unstarted. Future research should target dynamic
stability, centre-of-mass estimation, and whole-body balance control —
see `docs/Software_Architecture_Research.md` for the engineering
background.
**Dependencies**: Locomotion (shares actuation), Power Distribution
(battery CG affects balance directly, per `docs/BOM.md`'s battery-
placement notes).
**Failure modes**: TBD.
**Future expansion**: TBD.

## 3. Navigation System
**Purpose**: know where K9 is.
**Hardware**: RPLIDAR C1 (RESERVED, explicitly not committed —
`Hardware_Manifest.md`), camera (RESERVED), IMU, wheel odometry (none —
legged, not wheeled), leg odometry (theoretical).
**Software**: SLAM, mapping, localization, path planning, obstacle
avoidance. **Status: confirmed zero prior asset** — the previous K9
project's navigation was "entirely aspirational... no mapping, following,
obstacle avoidance, or docking code exists anywhere" (migration report
§3). Mark 6 starts from zero here because nothing was ever built, not
because anything was discarded.
**Dependencies**: Vision (camera-based localization, if adopted).
**Failure modes**: TBD.
**Future expansion**: TBD.

## 4. Vision System
**Purpose**: understand the world.
**Hardware**: camera (RESERVED), Jetson Orin Nano 8GB (KNOWN, module/
carrier IDs confirmed).
**Software**: object recognition, people/door/furniture detection,
gesture recognition, owner recognition. **Status: no prior asset** — the
previous project's camera attempt "never worked" (wrong CSI ribbon size)
and no recognition code was found (migration report §3).
**Dependencies**: Navigation (if vision-based localization is adopted).
**Failure modes**: TBD.
**Future expansion**: TBD.

## 5. Speech System
**Hardware**: microphones (RESERVED), speaker (RESERVED).
**Software**: wake word, speech recognition, speech synthesis,
conversation.
**Status: real, substantial prior assets — assessed in
`K9_Software_Migration_Report.md` §1.** Piper TTS works (generic voice,
not K9-specific). STT is Google cloud + substring wake-word matching,
not Whisper as the old project's own docs claimed — needs a real
decision, not inherited as-is. **The John Leeson voice-cloning (RVC)
work is ~60-70% complete** (real dataset, real training pipeline,
informally validated) and should be finished, not rebuilt. One hard
requirement carries forward: speak/listen mutual-exclusion gating, after
a real 2026-07-06 audio feedback incident in the prior project.
**Dependencies**: Character System (persona drives what's said; cadence
rules apply regardless of engine).
**Failure modes**: audio feedback loop (documented prior incident,
mitigation known — mutual exclusion).
**Future expansion**: finish Piper→RVC runtime integration.

## 6. Character System
**Purpose**: preserve K9 — the one system that's not primarily hardware.
**Covers**: personality, dialogue, behaviour, decision priorities,
etiquette, humour, memory (character-facing, distinct from Robot
Learning's factual memory below).
**Status: strongest system in the whole taxonomy.** A real, mature
system prompt exists and is migrating close to verbatim (Master/
Mistress address, Affirmative/Negative/Insufficient data, "This unit,"
1-3 sentence default) — see `K9_Software_Migration_Report.md` §2 and the
Character Philosophy note in `docs/Design_Bible.md`. This prompt
independently converges with a separately-conducted character study
arriving at essentially the same K9 — two independent engineering paths
landing on the same character is real evidence the character is being
captured, not just one side's interpretation of it. Treat the prompt as
a canonical engineering asset (not immutable, but not to be casually
rewritten either).

A concrete **behavioural-mode → tail/head/LED mapping** exists as a
body-language starting point (renamed from "emotion mapping," review
013 — "Behavioural Modes" keeps the implementation aligned with K9's
professional personality rather than implying simulated feelings): the
prior project's 5 states (happy/proud/sorrow/alert/aggression) are a
first draft; a professionally-framed target set going forward is
Content/Focused/Concerned/Protective/Investigating/Mission/Idle.
Decision hierarchy: owner safety → mission → knowledge → efficiency →
etiquette (professional priorities, not simulated emotion). One
non-negotiable hard rule, elevated to architecture (review 013): **K9
speaks intentionally, never accidentally** — implementation may evolve
(currently an explicit arm-gate), the rule should not.

**Three distinct kinds of memory** (review 013's biggest structural
insight from the migration report) — keep these separate from the
start, don't let them collapse into one generic store:
- **Identity Memory** — never changes: K9, rules, character, values.
- **Long-Term Memory** — persistent: people, places, home, owner,
  experience. This is Robot Learning (System 7 below).
- **Working Memory** — temporary: conversation, current task, immediate
  observations. The prior project had only a crude 4-exchange version of
  this and nothing else — see Learning System below.
**Dependencies**: Speech (cadence/delivery), Locomotion (body language
during Listening/Waiting/Thinking states — see migration report §4).
**Failure modes**: the prior project's own near-miss — an autonomous
"curiosity loop" leaking past a mute flag via an unguarded wake-word
path. Guard against the same class of gap here, not just the same code.
**Future expansion**: see the KBE section above for the target behaviour
vocabulary (Investigate/Follow/Guard/Wait/Greet/etc.) — none implemented
yet, this is the open work.

## 7. Learning System
**Purpose**: Robot Learning specifically (see distinction above) — owner
preferences, home map, daily routine, names, voice recognition, favourite
locations, allowed rooms, charging routine, behaviour refinement.
**Status: confirmed to not exist.** The previous project's `memory/`
directory was empty; the only "memory" was a 4-exchange in-RAM rolling
list, lost on every restart (migration report §2). This is genuinely new
work, not a migration — no disk-backed store, no cross-session
persistence of anything K9 has learned exists anywhere in prior work.
**Dependencies**: Character System (what gets remembered should serve
the persona, not be a generic chat log).
**Failure modes**: TBD.
**Future expansion**: the whole system.

## 8. Safety System
**Purpose**: highest priority of all thirteen systems.
**Covers**: emergency stop, servo monitoring, temperature, battery,
collision avoidance, current limits, human safety.
**Status**: largely unstarted on the mechanical/electrical side. Real,
current exception: the Waveshare servo bus driver's ~5A per-header limit
is already a binding constraint (`Hardware_Manifest.md` TD-010) — power
must split per-tier rail, data-only sharing on the bus. Inherits one
real, hard-won software rule from the prior project: **voice is never
autonomous** (see Character System) — a safety/etiquette rule, not just
a personality trait.
**Dependencies**: Power Distribution, Character System.
**Failure modes**: TBD (mechanical); audio feedback loop and unguarded
wake-word forwarding (documented prior incidents, Speech/Character).
**Future expansion**: TBD.

## 9. Health System
**Purpose**: monitor the robot itself over time (distinct from Safety's
real-time protection).
**Covers**: servo wear, battery health, storage, cooling, motor
temperatures, system logs, predictive maintenance.
**Status: unstarted.**

## 10. Power Distribution System
**Purpose**: get the right voltage/current to every consumer safely.
**Hardware**: main battery (12V 3S2P, 5000mAh confirmed, physical dims/
connector/BMS still unmeasured — `Hardware_Manifest.md`), 2S 18650
second battery, 5A/5V bucks (ESTIMATE envelopes only), Waveshare bus
driver's ~5A/header limit (real, binding).
**Status: unstarted as a designed system** — currently a list of
components in `docs/BOM.md`, not a power architecture (no main fuse,
master switch, distribution board, current monitoring, or emergency
disconnect designed yet — Hardware Manifest §4).
**Dependencies**: Balance (battery CG), Safety (current limits),
Thermal Management (bucks/regulators generate heat).
**Failure modes**: TBD.
**Future expansion**: TBD.

## 11. Thermal Management System
**Purpose**: keep the Jetson, regulators, and servo controllers within
safe operating temperature — distinct from Health's after-the-fact
temperature *monitoring*, this is the active cooling design.
**Status: unstarted.** No fan sizing, intake/exhaust, or filter strategy
exists (Hardware Manifest §6).
**Dependencies**: Power Distribution, Vision (Jetson thermal budget
affects sustained inference).
**Failure modes**: TBD.
**Future expansion**: TBD.

## 12. Communications System
**Purpose**: wireless/networking — not investigated at all yet in this
taxonomy's prior passes.
**Status: unstarted.** No decision recorded on wireless hardware,
internal network topology, or communication buses (CAN/UART/USB/I2C/
SPI/Ethernet) — Hardware Manifest §8 lists these as open. Note: the
previous K9 software project used a Pi/cloud split over an external
network (Tailscale) purely as a hardware-constraint workaround for a
512MB Pi Zero; Mark 6's Jetson Orin Nano 8GB doesn't share that
constraint, so that specific topology should not be inherited (migration
report §2) — but the general fast-local/slow-deliberate separation of
concerns is worth keeping.
**Dependencies**: Speech (cloud STT/TTS calls, if retained), Character
(cloud-hosted persona was one prior design, not necessarily right here).
**Failure modes**: TBD.
**Future expansion**: TBD.

## 13. Maintenance System
**Purpose**: physical service procedures — distinct from Health's
condition *monitoring*, this is the human-facing servicing process
itself.
**Status: unstarted as a system**, though the mechanical design already
carries real serviceability requirements (every servo/pivot/bushing
replaceable, per `docs/Design_Standards.md`) — this system would define
the actual procedures (e.g. "to replace the left hip servo: step 1...")
once subsystems are built, per the roadmap's Acceptance Test concept.
**Dependencies**: every other system (a maintenance procedure only makes
sense once the thing being maintained exists).
**Failure modes**: TBD.
**Future expansion**: the eventual generated service manual, per
`ROADMAP.md`'s BOM-as-systems-inventory recommendation.

## Hardware → Systems cross-reference

See `docs/Hardware_Manifest.md` — each component row carries a
`Supports` tag back to the system(s) here. A component supporting no
system is a red flag, not a fact worth ignoring.
