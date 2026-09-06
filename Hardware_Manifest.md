# Hardware Manifest

Per Engineering Review 008: nothing becomes "locked" until every field
below is complete. Every component is tagged:

- **KNOWN** — purchased/fully specified, dimensions and mass verified.
- **ESTIMATED** — selected but not yet measured; datasheet/typical figures used.
- **RESERVED** — space intentionally allocated, no component selected yet.

Sources cited per row. Where this session found real prior data (searched
`/home/jwh/claw3-full-backup`, `/home/albert/old_ubuntu_keep` — actual past
project files, not guesses), it's marked "found, this session" and dated.
Where the user confirmed something directly in chat, that's cited too.

## Servos

**Supports**: Locomotion, Balance (leg/spine servos); Character (tail/
ears — body-language expression per `docs/Systems_Architecture.md` §6).

| item | status | mfr/part | qty | dims (mm) | mass | power | connector | mounting | source |
|---|---|---|---|---|---|---|---|---|---|
| STS3215 (30kg tier) | **KNOWN** | STS3215-C018 | 16 | 24.7×35.0×45.2 | 55g each | — | serial bus | U-clamp | measured, `mark6_balance.py` comment: "Jon, 2026-07-26: 19 and 30 are 55g each" |
| STS3215 (19kg tier) | **KNOWN** | (no distinct P/N found) | 8 | 24.7×35.0×45.2 (same case) | 55g each | — | serial bus | U-clamp | same as above |
| SCS0009 | **KNOWN** | FeeTech SCS0009 | 4 | 23.5×12.0×29.0 (project) / 23.2×12.1×25.25 (handoff spec — unreconciled, TD-009) | 13.2g each | — | serial bus | U-clamp | measured, same source |
| Servo inventory count found, this session | — | — | "23 base-build owned (12×STS3215-C018 19kg + 11×SCS0009) + 6×19kg-tier arrived 2026-07-22" | — | — | — | — | — | `k9quad/cad/output/SESSION_PROGRESS_20260723.md` §2 — **note this predates the current 28-servo reconciled inventory in `docs/BOM.md`; treat BOM.md as current, this row as historical provenance only** |

## Printer

| item | status | mfr/part | dims | source |
|---|---|---|---|---|
| Printer | **KNOWN, LOCKED** | FlashForge Adventurer 5M | 200×200×200mm build volume, 0.4mm nozzle | `servo_case_AD5M_CONFIRMED.gcode` embedded slicer profile — actually sliced/confirmed, not a spec-sheet guess |

## Main Battery

**Supports**: Power Distribution, Balance (CG), Thermal Management.

| item | status | mfr/part | qty | dims | mass | power | connector | source |
|---|---|---|---|---|---|---|---|---|
| Main battery | **KNOWN (spec), dimensions/connector still ESTIMATED** | 12V 3S2P Li-ion, **5000mAh** | 1 | **not measured** | ~520g\* (PROVISIONAL, engineering-report figure, predates this confirmation) | 12V nominal, runs Jetson + servo core bus | **not documented** | capacity confirmed twice: `k9quad/cad/output/SESSION_PROGRESS_20260723.md` §2 ("Jon confirmed... no new pack needed") AND directly by Jon this session (2026-09-06: "it is the 5000ma battery") — **supersedes an earlier casual "6000mAh" mention in this same session, which is now known wrong** |

**Still needed before this can move from ESTIMATED to KNOWN** (per Review
008): measured length/width/height, measured weight, connector type,
connector orientation, cable exit direction, minimum bend radius, mounting
preference, whether the pack has internal protection electronics (BMS).
`k9quad/cad/output/SESSION_PROGRESS_20260723.md` §5 flags exactly this gap
from months ago: *"Weigh the battery and read its discharge/BMS rating...
continuous amps [are the risk]... cheap 12V packs often pair a large
headline capacity with a modest BMS cutoff that trips on servo inrush."*
**This physical check was flagged as needed in July and, as far as this
search found, was never closed out.**

## Second Battery

**Supports**: Power Distribution, Balance (CG).

| item | status | dims | mass | source |
|---|---|---|---|---|
| 2S 18650 pack (7.4V mid rail) | ESTIMATED | 72×38×20mm | 100g\* | engineering report §9; `k9quad/cad/output/MARK6_COMPONENTS.md`:24 calls it "ex-Dalek" — a reused pack, not a fresh purchase |

## Compute

**Supports**: Vision, Speech, Character, Learning, Navigation
(Jetson — the shared processing node for multiple systems, not "just a
computer"); Locomotion, Balance (Pico/XIAO — real-time reflex loop);
Locomotion, Balance, Character (Waveshare bus driver — the physical
channel every servo command and body-language cue travels through).

| item | status | mfr/part | dims | power/thermal | source |
|---|---|---|---|---|---|
| Jetson | **MEDIUM CONFIDENCE** — module identified, carrier/cooling/mounting still open | **Jetson Orin Nano 8GB**, module P3767-0003, carrier board P3768-0000 | 103×90×35mm (carrier+heatsink, engineering-report estimate) | needs airflow, exact heat load not measured | module/carrier IDs found this session in `/home/albert/old_ubuntu_keep/.../.nvsdkm/hwdata/families/jetson/devices/jetson-orin-nano-8gb.json` — NVIDIA SDK Manager's own device metadata, meaning this exact module was actually flashed/set up on this machine at some point, not a guess |
| Raspberry Pi Pico | MEDIUM-HIGH — board known, packaging not frozen | RP2040/RP2350 (per engineering report §9's "legs: XIAO RP2040 or RP2350") | ~21×17.5mm (XIAO form factor, if that's the actual board) | — | engineering report §9 — **note: report's compute plan actually specifies XIAO RP2040/RP2350 and XIAO ESP32-S3 Sense, not a Pico specifically; Review 008 says "Raspberry Pi Pico" — flag this naming mismatch rather than silently picking one** |
| Servo bus interface | **KNOWN** (found this session, not in current BOM.md — gap) | 2× Waveshare "General Driver for Robots" | — | onboard UART-USB, up to 253 ST3215 servos per bus | `k9quad/cad/output/SESSION_PROGRESS_20260723.md` §2: "covered... No USB→TTL adapters need buying." **Caveat carried in that same note: bus header power path ~5A ≈ 5 non-stalled ST3215, so full-dog wiring must split power per tier rail and share data only** — a real constraint for the wiring/power-distribution unknowns in Review 008 §5. **This resolves part of Review 008's "servo controller: unknown" item** — should be reconciled into `docs/BOM.md`. |

## Sensors

**Supports**: Locomotion, Balance, Navigation (IMU — real, already
confirms the exact "not just a sensor" point from Review 008: it
supports three systems at once); Navigation (RPLIDAR C1, if committed);
Vision, Character (cameras/mics/speakers, once selected).

| item | status | source |
|---|---|---|
| IMU + barometric + temp/humidity | **KNOWN** (found this session) | Waveshare Sense HAT (Pi Zero form factor) — `SESSION_PROGRESS_20260723.md` §2: "deletes the BME280 line item AND supplies the IMU" |
| RPLIDAR C1 | **RESERVED, explicitly not committed** | same source, §6: "available, deliberately NOT committed" |
| Cameras/mics/speakers | RESERVED | no source found — open per Review 008 §7 |

## Power Distribution

**Supports**: Power Distribution System, Safety (current limits/e-stop),
Thermal Management (regulator heat).

| item | status | notes |
|---|---|---|
| Main fuse, master switch, distribution board, current monitoring, voltage regulators, emergency disconnect | **RESERVED — all open**, per Review 008 §4 | 5A buck (Jetson) and 5V buck (logic) appear in `docs/BOM.md`'s payload table as PROVISIONAL/ESTIMATE envelopes only, no board selected |

## Wiring

**Supports**: Communications, Power Distribution, Safety, Maintenance
(service loops/connector access are a servicing concern as much as an
electrical one).

Open per Review 008 §5: connector family, wire gauges, harness routing,
service loops, strain relief, connector access. **One real constraint
found this session**: the Waveshare bus driver's ~5A header limit means
per-tier power rails with shared data-only bus wiring is already a
structural requirement, not a future nice-to-have — factor this into any
future wiring standard.

## Cooling

RESERVED — all open per Review 008 §6 (Jetson, regulators, servo
controller airflow, fan sizing, intake/exhaust, filter strategy).

## Mass Distribution — the biggest open item (per Review 008 §10)

Per `docs/Weight_Budget.md`: the project's own servo mass ledger was found
to be undercounted by 246.4g this session (root-caused, not just
estimated). Every payload mass in `docs/BOM.md` is still an ESTIMATE
pending a bench weigh. **No component in this manifest has a KNOWN,
measured mass yet except the servos.** CG calculation, static stability,
and gait/pitch/roll analysis all wait on this.

## Reconciliation TODOs (found by cross-referencing old projects against current docs)

1. Update `docs/BOM.md` to include the Waveshare servo bus driver (2×
   General Driver for Robots) — currently missing entirely from the
   current BOM despite being a real, already-owned part.
2. Update `docs/BOM.md`'s IMU/BME280 line: per the 2026-07-23 note, the
   Waveshare Sense HAT supersedes the separate BME280 + IMU line items —
   current BOM still lists them separately.
3. Resolve the Pico vs. XIAO RP2040/RP2350 naming mismatch (Review 008 vs.
   engineering report) before treating either as settled.
4. Get the main battery's physical dimensions/connector/BMS status
   measured — flagged as needed in July 2026, still open as of this
   manifest.
