# K9 Software Migration Report

Engineering migration assessment of the previous K9 software project
("K9claw," a Pi Zero 2 WH-based project on `claw3`), produced from a
direct audit of `/home/jwh/claw3-full-backup/home/albert/K9_PROJECT/`
and related files — not a summary of intentions, an audit of what
actually exists in code.

## Correction to the framing this assignment started from

The premise was "we've built the body, now reconnect it with the mind
that already exists." That's accurate for **one** layer and wrong for
another:

- **Character/personality**: real, reusable, worth migrating directly.
- **Body control** (motors, servos, sensors, navigation): **the previous
  project is a 4-wheeled differential-drive robot with hobby PWM servos**
  (2× via PCA9685 — head pan, tail wag), not a legged platform. It has
  zero STS3215/SCS0009 serial-bus-servo code, zero ROS, zero navigation
  implementation. There is effectively nothing here to "reconnect" to
  Mark 6's legs — Mark 6's body-control software starts from zero
  regardless of this prior work. What *does* transfer is an
  **architectural pattern** (below), not code.

## 1. Voice System

**TTS**: Piper (local neural TTS) is the live, working implementation —
generic voice (`en_GB-northern_english_male-medium`), not K9-specific
yet. An earlier `edge-tts` plan in `K9_GATE.md` was superseded before
being built — historical only.

**STT / wake word**: live code uses Google's cloud STT
(`speech_recognition` + `recognize_google`) with plain substring matching
against `["k9","k-9","kay nine","hey k9"]` — continuous transcription,
not a dedicated low-power wake-word engine. **This contradicts the
project's own architecture doc**, which specifies Whisper — a real
doc-vs-code gap, not a design decision to inherit either way.

**Voice cloning (RVC) — the most valuable unfinished asset found**: real,
substantive work targeting John Leeson's actual K9 voice. A 79-clip/9-min
Leeson dataset (demucs-separated, real audio), a real Colab training
notebook, and an informally-validated XTTS-v2 proof of concept already
run. No trained model weights exist in any local backup (they were meant
to land on a remote Oracle server not present here) — the
Piper→RVC runtime integration was never finished ("task #13" in the
project's own tracker). **~60-70% of the way to a genuinely K9-authentic
voice.**

**Real production incident, transferable as a hard requirement**: a
2026-07-06 audio feedback loop where K9 heard its own TTS output and
re-triggered itself, fixed with mutual-exclusion gating between
speak/listen. Any Mark 6 voice system needs this same gating from day
one, not as a retrofit discovered the hard way twice.

## 2. Personality

**LLM**: API-based — Anthropic Claude (`claude-haiku-4-5`) via the
`anthropic` SDK. Two generations exist: an early direct-call version and
a later design routing through a hosted "Oracle Cloud K9 brain" that
owns the persona server-side and returns a structured
`{speech, emotion, display, catchline}` payload driving voice+screen+tail
together. Role: conversation/Q&A only — no planning, no tool use, no
autonomous task execution anywhere in the codebase.

**Character definition — real, mature, worth migrating close to
verbatim.** Plain-text system prompt, consistent across both brain-server
generations:

> "You are K9, the robot dog from Doctor Who. You are an encyclopedic
> computer with dog-like loyalty and pedantic precision... Address owner
> as 'Master' or 'Mistress' always — 'Affirmative' for yes, 'Negative'
> for no, 'Insufficient data' for unknown — Begin new info with
> 'Information, Master...' or 'Suggestion, Master...' — Keep responses
> to 1-3 sentences unless asked to elaborate — Never say 'I' — say 'This
> unit' if self-reference needed."

Voice target explicitly named as John Leeson (warm, clipped, robotic
monotone) — deliberately distinct from an unrelated Dalek project's
ring-modulated voice, i.e. real care already taken to keep characters
separate. A concrete **emotion → tail angle / head pose / LED colour**
mapping already exists (5 states: happy/proud/sorrow/alert/aggression) —
a ready-made starting point for the body-language research this project
is doing.

**Memory: does not exist.** The project's own `memory/` directory is
empty. The only "memory" is an in-process 4-exchange rolling list, lost
on every restart. This is the single biggest gap in the "mind already
exists" framing — it needs to be designed fresh, not migrated.

**Decision logic**: Bosun's stated loop is ANALYSE → REPORT → ASSIST →
PROTECT. The one non-negotiable rule, enforced in code at multiple
layers (not just documented): **"VOICE IS NEVER AUTONOMOUS — K9 speaks
only on explicit command."** An autonomous "curiosity loop" was built,
found to leak past the mute flag via an unguarded wake-word path, and was
explicitly ripped out and re-gated with a single-shot arm mechanism. This
should migrate as a **hard architectural constraint**, not a preference.

## 3. Robot Control / Sensors / Navigation

**Motor/servo control**: not comparable to Mark 6 — 4× N20 DC motors via
TB6612 (differential drive, blocking duration-based commands), 2× hobby
PWM servos via PCA9685. No serial-bus-servo protocol, no closed-loop
feedback, no real-time constraints beyond `time.sleep`.

**ROS / state machine / behavior tree**: none found anywhere (grepped
project-wide). Bosun's health/self-heal loop is a decent lightweight
pattern but not a formal FSM/BT framework.

**Sensors, actual status**: IMU/environmental sensing (Sense HAT)
attempted but hardware was **never confirmed detected** — ran on mock
data. Camera attempted, **never worked** (wrong ribbon-cable size for
the Pi Zero's smaller CSI connector). Microphones/speaker: working.
LIDAR: not present in this project at all (RPLIDAR C1 only enters via
Mark 6's own, unrelated hardware manifest).

**Navigation: entirely aspirational.** No mapping, following, obstacle
avoidance, or docking code exists anywhere — only a diagram label in a
planning doc. Mark 6 starts from zero here regardless of this prior
project, not because of it.

**What actually transfers — the architectural pattern, not the code**:
independent per-subsystem microservices, each with a `/status` health
endpoint and mock-mode fallback when hardware isn't detected; a
Bosun-style conductor doing health-checks with fail-count thresholds and
per-manager restart cooldowns (a real fix for a real "restart storm" bug
where one missed health ping used to instantly restart a busy-but-healthy
manager); a Quartermaster-style passive service registry. These ideas are
hardware-independent and worth reusing regardless of legs vs. wheels.

## 4. Behaviour Patterns

**Found in the old code**: almost nothing — the audit found no idle,
greeting, following, protecting, searching, or thinking behaviours
implemented anywhere; the closest real artifact is the emotion→tail/
head/LED mapping table (§2) and Bosun's ANALYSE→REPORT→ASSIST→PROTECT
loop, which is a service-health pattern, not a character behaviour.

**New character research (2026-09-06), not derived from old code —
these are proposals to design toward, not migrated assets**:

- **Listening**: head rotates, body remains still, tail stops.
- **Waiting**: sits, observes, minimal movement.
- **Thinking**: very small head motion, eyes remain fixed, brief pause.

Confidence should dominate motion in all three — no fidgeting, no
wasted movement (consistent with the character direction already in
`docs/Design_Bible.md`: deliberate movement by choice, not limitation).
Idle, greeting, following, protecting, searching, learning, and
responding behaviours are explicitly **not yet defined** — open work for
the canonical pose/behaviour library (`docs/Biomechanics_Reference.md`).

## 4b. Character Philosophy (2026-09-06 character research)

**K9 is not a dog with a computer, nor a computer pretending to be a
dog — he is an artificial person whose chosen physical embodiment is
canine.** Canonically built by Professor Marius, with multiple physical
Mark units (I–VI now) across a consistent personality — the body is
replaceable, the character is not. This directly validates the "ancient
intelligence in a newly engineered body" framing already in the
project: the body is modern engineering; the mind carries accumulated
experience; confidence is earned, not programmed.

**K9 behaves professionally, not emotionally** — dependable, not cold.
Rarely hesitates, rarely apologizes, reports conclusions rather than
opinions. **Build professional priorities, not robot emotions**: owner
safety → mission → knowledge → efficiency → etiquette, with emotion as
emergent behaviour from that hierarchy, not a simulated feeling-state
bolted on separately.

Humour is never forced — it emerges from literal interpretation, deadpan
certainty, and perfect timing, not jokes. Affection is earned through
action, never sought. Voice cadence matters more than voice content:
short, precise, considered statements — never rambling, no filler.

**Governing rule for all future AI/behaviour work**: Mark 6 should never
become more human. It should become more K9. Richer perception, better
movement, and deeper understanding of people and the world are all
means to that end, not a drift toward generic conversational-AI
behaviour.

## 5. Lessons Learned (quoted, not paraphrased — these are process rules, not color)

- **"VOICE IS NEVER AUTONOMOUS"** — repeated verbatim across multiple
  docs, enforced at multiple code layers after an audit found a gap.
- **"Never report PASS without real observation; screen claims need
  Jon's eyes."** — don't let an AI assistant self-certify a physical
  result. Directly relevant to how physical validation should be handled
  going forward on Mark 6 (e.g. mechanism validation, print tests).
- **"Software before hardware blame — always."**
- **Config drift, named as a recurring root cause**: a display was wired
  for the wrong GPIO pins because a planning doc described different
  hardware than what actually shipped. This is exactly the discipline
  Mark 6's `Hardware_Manifest.md` KNOWN/ESTIMATED/RESERVED tagging
  already exists to prevent — treat that discipline as validated by a
  real prior failure, not just good practice in the abstract.
- A specific regression was remembered by name and guarded against
  ("don't re-apply the change that once wedged a working screen") rather
  than being at risk of repeating it.
- When a physical fix carried real risk (reseating a marginal connector),
  the user declined once for a stated reason, and the project recorded
  "do not ask again" — a real example of respecting a risk call instead
  of re-litigating it.

## 6. Migration Decision

| | Migrate directly | Needs redesign | Abandon |
|---|---|---|---|
| **Character** | System prompt + phrase rules (Master/Mistress, Affirmative/Negative/Insufficient data, "This unit," 1-3 sentences); emotion→tail/head/LED mapping table; "voice never autonomous" as a hard constraint | — | — |
| **Voice** | Speak/listen mutual-exclusion pattern; canned-soundbank + novel-TTS split; the RVC Leeson dataset + training pipeline (finish, don't rebuild) | STT (pick Whisper as originally intended, or a real wake-word engine — not continuous cloud transcription) | edge-tts placeholder plan |
| **Memory** | — | Everything — design fresh, there's nothing to preserve | — |
| **AI/reasoning** | "Fast local reflex + slower deliberate reasoning" separation of concerns (the idea, not the network topology) | — | The specific Pi-Zero/cloud-brain split (was a hardware-constraint workaround; Jetson Orin Nano 8GB doesn't have that constraint) |
| **Control architecture** | Microservice-per-subsystem pattern, health-check conductor with fail-thresholds/cooldowns, passive service registry, mock-mode-on-missing-hardware | — | — |
| **Motors/servos/sensors/nav** | — | — | All of it — different hardware family entirely, nothing here applies to STS3215/SCS0009/legs |

## What this means for the Systems Architecture

Cross-reference against `docs/Systems_Architecture.md`:
- **Character System**: has a real, strong starting point (this report's
  §2). Highest-confidence system in the whole taxonomy.
- **Speech System**: has a real, substantial, unfinished asset (Leeson
  RVC) worth prioritizing completion of, plus one hard lesson (feedback
  gating) to build in from the start.
- **Learning System**: confirmed to not exist — this is genuinely new
  work, not a migration.
- **Safety System**: inherits one real, hard-won, non-negotiable rule
  (voice-never-autonomous) as a starting requirement, not a future
  nice-to-have.
- **Locomotion/Balance/Navigation/Vision**: no prior software asset
  applies — these start from zero on Mark 6's own hardware, confirmed by
  this audit rather than assumed.
