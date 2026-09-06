# Milestones

Traceability log — one entry per export from the private engineering
repo. Each entry names the private repo's commit this snapshot came from,
so "what changed since last time" is always answerable without diffing
STEP files by eye.

| version | date | headline | private repo commit |
|---|---|---|---|
| v1 | 2026-09-06 | Mechanical Pelvis Alpha — first structure generated within a validated motion envelope (28.7 cm³ intrusion vs. 54.6/37.2/43.3 cm³ for the three authored candidates) | `52bb147` (branch `rear-drive`) |
| v2 | 2026-09-06 | Alpha reframed as Proof of Method -- intrusion classified by feature (~100% Structural/Bearing/Servo, 0% Incidental); PR #1 merge gate formalized; 7 core principles locked | `55cd017` (branch `rear-drive`) |

| v3 | 2026-09-06 | Systems Architecture (13 systems + K9 Behaviour Engine) and full K9 software migration audit -- previous project confirmed as an unrelated wheeled platform; Character/personality migrates directly, body-control software does not | `6a1029f` (branch `rear-drive`) |

| v4 | 2026-09-06 | Review 013 refinements: Behaviour Layer (not Engine), Behavioural Modes, three memory types, candidate axiom preserved pending scrutiny | `ee0a4e7` (branch `rear-drive`) |

## Convention for future exports

A commit in the private repo becomes export-worthy when it's tagged with
a headline — a one-line summary written for an external reviewer, not an
internal commit-message detail. Only tagged commits get mirrored here;
everything else (in-progress experiments, dead ends, routine fixes) stays
private by default. This keeps curation a deliberate act, not something
a cron job infers from commit volume or file changes.
