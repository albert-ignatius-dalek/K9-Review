# K9 Behaviour Reference

Not implementation. Behaviour. Per review 013: every subsystem eventually
references this document — it specifies *what K9 does*, independent of
whether the Behaviour Layer ends up implementing it as a behaviour tree,
state machine, or planner.

**Status: scaffold only.** The Behavioural Ethology Study (the renamed
4th research stream — study K9 the way an ethologist studies an animal,
not the way a software project specs a feature) hasn't started. This
file defines the structure it lands in.

## Per-behaviour entry template

```markdown
### <Behaviour name>

**Purpose**: why this behaviour exists.
**Trigger**: what causes K9 to enter this behaviour.
**Body posture**: overall stance/orientation.
**Head posture**: where attention is directed.
**Tail**: position/motion.
**Voice**: does K9 speak during this behaviour, and how.
**Speech style**: cadence/formality specific to this behaviour, if
different from baseline.
**Priority**: where this sits in the owner-safety → mission → knowledge →
efficiency → etiquette hierarchy.
**Interruptibility**: what can pre-empt this behaviour, and what can't.
**Exit condition**: what ends the behaviour and what happens next.
```

## Target behaviour vocabulary (not yet specified — names only)

Investigate, Return Home, Follow, Guard, Observe, Escort, Search, Wait,
Greet, Patrol, Recharge, Standby, Shutdown, Wake — see
`docs/Systems_Architecture.md`'s Behaviour Layer section for where these
sit architecturally. Also from the ethology framing specifically:
Approach, Think.

## Relationship to other documents

- `docs/Biomechanics_Reference.md` — the physical motion each behaviour
  requires (a behaviour's "Body posture"/"Head posture" fields should
  cite real poses from there once both exist, not invent new ones).
- `docs/Design_Bible.md` — the character philosophy each behaviour must
  stay consistent with (professional priorities, not simulated emotion;
  more K9, never more human).
- `docs/Systems_Architecture.md` — which of the 13 systems a given
  behaviour actually coordinates (a behaviour with no system dependency
  isn't really a robot behaviour).

A behaviour specified here with no corresponding motion study or system
dependency is documented but not yet engineered — the same discipline
`Biomechanics_Reference.md` already applies to itself.
