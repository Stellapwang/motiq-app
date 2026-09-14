# MotiQ — handoff to a native iPadOS build

## Read this first

`index.html` is a **prototype**, not a design to be ported line for line. It is a single
self-contained file that runs the whole battery in Safari on an iPad, and it exists to
establish the task rules and the data schema on real hardware with real participants.

**Two of its behaviours are workarounds for limits a web page cannot escape, and removing
those limits is the reason the native app exists.** A developer handed only this file will
reimplement the workarounds instead of the requirements. See *What the native build must
do*, below.

Three sources, in order of what they are good for:

| File | What it carries |
|---|---|
| `index.html` | The rules, the geometry, the data schema, the participant-facing wording. Heavily commented; the comments are the specification. |
| `DECISIONS.md` | Why each rule is the shape it is. What was tried first and how it failed. Generated from the commit history. |
| `SCOPE.md`, `CLAUDE.md` | Project constraints and working rules the build inherits. |

Line numbers below are **as of BUILD v12.43 (2026-09-13)**. They drift. Each entry also
gives a search string that does not.

---

## What the native build must do

These four are on the Device tab of the running app, under *Notes for the app developer*
(`index.html` line 326, search `Notes for the app developer`). Items 1 and 2 are the reason
this project exists.

1. **Pen and touch are not simultaneous.** While an Apple Pencil is in contact, iPadOS
   discards concurrent finger touches as palm contact, and no web API turns that off.
   `PEN_TONE` and `PEN_FAST` therefore do not run as true dual tasks in the prototype, and
   their data is flagged `dataHeld`. UIKit delivers both streams; Safari does not.
2. **Pencil hover height is unavailable.** The platform tells a web page *where* the pen
   hovers, never *how high*. Hover at all requires an M2 iPad — measured 2026-09-11: none on
   an 11-inch M1 iPad Pro, about 59 Hz on the M2. The native build must record hover onset,
   hover dwell before first contact (separately from time-to-first-contact), and the
   **height profile through the descent** — steady, stalled, or repeatedly approaching and
   withdrawing. That last one a web build cannot capture at all. Full rationale in the
   `PENCIL HOVER` comment block, line 1136.
3. **Voice recording needs an https origin.** A property of the address, not the device.
4. **Pen lifts during drawing** must be first-class events with count and duration, rather
   than inferred from stroke boundaries.

**Open hardware question, unresolved at handoff.** On the study iPad, Safari reports every
contact as `pointerType: "touch"` / `touchType: "direct"` — no stylus events at all, in 140
contacts. A paired Apple Pencil always reports as a pen on iPadOS, so this is a pairing or
hardware question rather than an app one, but it is not yet closed. The app detects and
reports it (`deviceEverSawPen`, search `SEEN_TYPES`). Verify Pencil identification early in
the native build.

---

## The battery

| What | Line | Search for |
|---|---|---|
| **All 16 tasks, with flags** | 996 | `CATALOGUE — the one place tasks are defined` |
| Renamed task codes, old to new | 1745 | `const TASK_ALIAS=` |
| Which hands a task can use | — | `function handsAllowed` |
| Queue construction, block interleaving | 3551 | `function buildQueue()` |

Each catalogue row carries the flags that drive everything else: `spiral`, `pent`, `tap`,
`audio`, `zones`, `voice`, `count`, `hands`, `repeats`, `lengthS`, `sustainedS`,
`implement`, `tier`. Read the row and the behaviour follows.

---

## Task rules

### Spiral (`PEN_SPRL`, `FIN_SPRL`, and every dual-task spiral)

| What | Line | Search for |
|---|---|---|
| Template geometry | 4425 | `function spiralPath(` |
| Geometry per implement (stylus vs finger differ deliberately) | 4215 | `const geomPitch =` |
| Sustained-spiral rationale and the angle detector | 4219 | `SUSTAINED SPIRAL: the end zone` |
| Net angle accumulator, with the start-zone gate | 4329 | `function makeAngleAccumulator(` |
| Angle threshold, allowing for the ungated region | 4311 | `function angleNeededRad(` |
| **A pass ends when the pen is lifted** | 4276 | `A PASS ENDS WHEN THE PEN IS LIFTED` |
| **A pass begins in the middle, every pass** | 5971 | `A PASS BEGINS IN THE MIDDLE` |
| Anchored start: the window opens on first contact in the start zone | 5852 | `SUSTAINED SPIRAL, and the anchoring` |

Four rules here look arbitrary and are not. All four have a commit in `DECISIONS.md`
explaining the failure that produced them:

- **The window opens on the first pen-down inside the start zone**, not at START. Hesitation
  before the first stroke is a measure, not overhead, and a clock from START would penalise
  it.
- **Angle is not counted inside the start zone.** Near the centre a small movement sweeps a
  huge angle — measured, three seconds of fidgeting produced 35 turns against the 5.7 needed
  to advance.
- **Every pass begins in the start zone, not just the first.** Passes 2+ used to begin at the
  previous pass's end, out at the rim, where the return journey banks angle at large radius.
- **A pass ends on a lift, with no position test.** Any radius threshold can be crossed while
  the pen is still drawing; a spiral drawn 30% larger than the template crosses every one of
  them during its last turn.

### Pentagon copy (`PEN_PENT`)

| What | Line | Search for |
|---|---|---|
| The model's vertices, constructed not traced | 4406 | `function pentagonModel(` |
| Largest side that fits the space | 4390 | `function pentSideFor(` |
| Layout, split by drawing hand | 4965 | `function layout(){` |

The figure is **constructed, not copied from a published instrument**. It satisfies the
scoring criterion but is not pixel-identical to the MMSE or MoCA figure, and a copy scored
against published norms for those would be scored against a figure the participant did not
see. The model is exported with every trial for exactly this reason. If norm-comparability
is wanted, replace the vertices and nothing else changes.

### Tapping (`TAP_FAST`, `TAP_BMIN`, `TAP_BANT`)

| What | Line | Search for |
|---|---|---|
| Zone geometry and labels | 5041 | `function buildZones(` |

Zones are separate hit targets, not painted on the drawing canvas: a finger arriving while
the pen is down must not disturb the pen's stroke.

### Auditory streams

| What | Line | Search for |
|---|---|---|
| Two stream types, not one per task | 1297 | `TWO STREAM TYPES` |
| The two streams | 1330 | `const STREAMS = {` |
| Which task uses which, and its response channel | 1348 | `const TASK_STREAM = {` |
| Everything else derived from those | 1360 | `function buildAudioDefaults()` |
| **Stream generation** | 4555 | `function makeSequence(` |
| Seeds: matched across participants, not across tasks | 4533 | `function hash32(` |
| Tone synthesis | 4687 | `function tone(at,freq,durMs,level)` |

- **Intervals are floor + exponential, never Gaussian and never uniform jitter.** An
  exponential above a floor has a flat hazard: having waited tells the participant nothing
  about when the next tone is due. Uniform jitter has a *rising* hazard and turns a reaction
  time into a synchronisation task.
- **The floor is a property of the response channel**, not the task: a tap needs ~600 ms, a
  spoken number ~800. On an oddball the standards are not answered at all, so their floor is
  perceptual separation (300 ms) and the response constraint moves to target-to-target
  spacing.
- **Targets are never adjacent**, and the required gap widens once the running count reaches
  double digits, because "twenty-seven" takes longer to say than "seven".

---

## Data

| What | Line | Search for |
|---|---|---|
| **The trial record** (schema `motiq_prototype/14`) | 6968 | `function buildRecord(` |
| CSV columns, with a length assertion | 7387 | `const CSV_HEAD=` |
| Response attribution to tones | 6628 | `function attribute(` |
| Omissions | 6657 | `function omissionStats(` |
| Longest attributable response | 3979 | `function bindingLimitMs(` |
| Voice capture | 4731 | `const RECORDER={` |

The record is the contract. It carries raw samples, not summaries: pen samples with
timestamp, millimetre position, pressure, tilt, pointer type, pass number and in/out of
window; taps with zone and timestamp; hover samples; the full tone schedule; the template
actually drawn; the complete parameter set verbatim; and the device setup the examiner
confirmed. **One record per trial, no aggregation layer.**

Three fields that exist because their absence caused a problem:

- `geometry_valid` — false if the page was zoomed or the window resized mid-trial, which
  makes every millimetre in that record incomparable.
- `implement_verified` — whether the device confirmed the implement, rather than the
  examiner's selection being taken as fact.
- `off_zone_starts` — contacts outside the start zone before the trial began. One is
  somebody who has not read the circle; five is a visuospatial finding.

Millimetres come from a **card calibration** (any bank card is 85.60 × 53.98 mm), so the
figure is the same physical size on every device rather than the same fraction of a screen.
Search `function applyCal()`, line 1828.

---

## Participant-facing wording

| What | Line | Search for |
|---|---|---|
| Instruction assembly | 1514 | `function instructionFor(` |
| The drawing rules, shared across tasks | 1497 | `function drawingRules(` |

The wording has been through many rounds with the study lead and is not placeholder text.
Lines wrapped in `**` are the emphasised statement; the plain lines under one are its
bullets. Notable: no em dashes anywhere a participant reads; the task code is an examiner
reminder in small grey type, not a heading; and the instruction names which index finger,
because on `TAP_FAST` the hand alternates between repeats.

---

## What NOT to carry across

- **The pointer diagnostics** on the Practice tab exist to chase the Apple Pencil question
  above. If the native build identifies the Pencil correctly, they have no purpose.
- **The palm-rejection workarounds** — `dataHeld` flags, the "pen and touch not
  simultaneous" warnings. The native build removes the cause.
- **The touch-event fallback** in the practice pad, which reads the touch stream when the
  pointer stream withholds the pencil. A Safari-specific rescue.
- **The single-file structure.** It is a deployment constraint, not an architecture.
