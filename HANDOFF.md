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

Line numbers below are **as of BUILD v14.01 (2026-10-01)** where they have been rechecked
and older where they have not. They drift. Each entry also gives a search string that does
not, and the search string is the one to use.

**v14.01 rebuilt the battery against two specification documents** — `MotiQ Task
description.docx` and `MotiQ_auditory_verbal_element_spec_rev.docx`. The sections below have
been corrected where those documents changed the answer; anything describing a rule this
file no longer implements is marked as superseded rather than deleted, because a developer
reading an older record needs to know what it meant.

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
| **All tasks, with flags** | — | `CATALOGUE — the one place tasks are defined` |
| Renamed task codes, old to new | 1745 | `const TASK_ALIAS=` |
| Which hands a task can use | — | `function handsAllowed` |
| Queue construction, block interleaving | — | `function buildQueue()` |
| Administration order, examiner-movable | — | `ADMINISTRATION ORDER` |
| Finger template counterbalancing | — | `function finTemplateOrder(` |
| Template names as the description gives them | — | `const spiralTemplateName` |

Each catalogue row carries the flags that drive everything else: `spiral`, `pent`, `tap`,
`audio`, `zones`, `voice`, `count`, `hands`, `repeats`, `lengthS`, `sustainedS`,
`implement`, `tier`, and from v14 `avObserve` and `finBlock`. Read the row and the behaviour
follows.

**The twelve conditions the task description specifies are the core battery.** The others
are at tier `second` or `expl` and sit outside the default plan. `VOC_VDGT` and `PEN_VDGT`
are new in v14: spoken-digit detection, which pairs with the tone conditions to separate
monitoring load from counting load.

**Administration order is drawing first and tapping last**, which is the description's own
order and reverses the graded order earlier builds used. The catalogue number is a default,
not the protocol: `ORDER_OVERRIDE` is the live order and the examiner moves it from the
Session sequence panel. The one constraint that is not a preference is enforced in a
warning rather than in the order — every dual task wants both of its single-task baselines
collected first.

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
| **A pass ends on a held second in the end zone** | — | `const END_ZONE_HOLD_MS` |
| The hold, as one function both the trial and the pad call | — | `function endZoneHold(` |
| **A pass begins in the middle, every pass** | 5971 | `A PASS BEGINS IN THE MIDDLE` |
| Anchored start: the window opens on first contact in the start zone | 5852 | `SUSTAINED SPIRAL, and the anchoring` |

**How a spiral ends, as of v14.** The gate is 4.4 of the 5 turns -- stated as the fraction
0.88 so it carries to a short template -- plus four rings drawn and four seconds elapsed.
The trigger is then **one continuous second with the pen inside the red circle**. Movement
inside the circle does not interrupt it, because a participant with tremor will shake and
this is a hold inside a 10 mm circle rather than a demand to be still. Leaving sends the
count back to zero rather than pausing it, and a pen lift counts as leaving. Nothing else
ends a pass: not a lift, and not a completed sweep that never reached the circle. A
participant who cannot hold it is covered by the examiner's **Next spiral**, recorded as
`advanced_by: "examiner"`.

**Two earlier triggers are superseded and a record may name either.** `end_zone` with no
dwell (v12.55 to v13.01) ended a pass the moment the segment between two samples crossed
the zone; `laps` ended it on a completed sweep that never reached the circle. The reasoning
for no dwell was that overshooting the end of a line is ordinary motor behaviour and
commoner in the impaired, so stopping dead on a 10 mm dot would be a second task. The
specification answers that in the instruction instead of in the detector -- the participant
is told that if they go past the circle they should come back and hold inside it.

Four rules below look arbitrary and are not. All four have a commit in `DECISIONS.md`
explaining the failure that produced them:

- **The window opens on the first pen-down inside the start zone**, not at START. Hesitation
  before the first stroke is a measure, not overhead, and a clock from START would penalise
  it.
- **Angle is not counted inside the start zone.** Near the centre a small movement sweeps a
  huge angle — measured, three seconds of fidgeting produced 35 turns against the 5.7 needed
  to advance.
- **Every pass begins in the start zone, not just the first.** Passes 2+ used to begin at the
  previous pass's end, out at the rim, where the return journey banks angle at large radius.
- **A pass does not end on a lift, and did briefly.** Advancing on a lift made the app
  reward the thing the instruction forbids, and a participant who had drawn the required
  turns and lifted to rest their hand got a new spiral for it. Lifts are counted and timed
  per pass instead.

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
| **The whole specification, in one block** | — | `THE AUDITORY-VERBAL LOAD ELEMENT` |
| The gap law | — | `const AV_GAP` |
| Fixed target counts by trial index | — | `const AV_TARGET_COUNTS` |
| Placement rules | — | `const AV_PLACE` |
| Predefined seeds | — | `const AV_SEEDS` |
| Which stimulus materials a task uses | — | `const AV_KIND` |
| **Stream generation** | — | `function makeSequence(opts)` |
| Target placement, and why it cannot fail | — | `function avPlaceTargets(` |
| Stimulus materials, tones and digits | — | `STIMULUS MATERIALS` |
| The digit recordings, and what installing them means | — | `const DIGIT_SET` |
| Tone synthesis | — | `function tone(at,freq,durMs,level)` |

**v14 replaced the stream's description rather than retuning it.** Three properties of the
earlier design are specified away, and a native build should not reimplement them:

- **The target count is not sampled.** It was `round(n * p)` with seeded jitter, so the
  number of targets was a random variable and two participants on the same trial answered a
  different number of times. `AV_TARGET_COUNTS` states it per trial index. No trial uses a
  count of 10, which is the number a participant guesses when they have lost the thread,
  and the two stimulus kinds disagree on every dual trial so a count cannot carry over.
- **Gaps are not rescaled.** The old generator drew n intervals and scaled the set to fill
  the window, bending every interval to make the arithmetic land. The law is exact per gap:
  `gap = 600 + X`, `X = -402.1 * ln(1 - U)`, redraw above 2900, so the finished gap is 600
  to 3500 and averages 1000. **The 402.1 is not 400 and must not be rounded to it** -- only
  oversized draws are discarded, so the surviving gaps are missing their largest members
  and 400 would average 998. If the minimum, mean, maximum or discard rule changes, that
  constant has to be recalculated.
- **The floor no longer depends on the response channel.** It was 600 ms for a tap and 800
  for a spoken number. Only the targets are answered, so the constraint belongs between
  TARGETS, where `AV_PLACE` puts it at 2000 ms; widening the gap between unanswered
  non-targets only thins them out, and thin non-targets are what make a target rare. One
  floor, 600 ms, every stream -- which also makes the usable response window the target
  separation rather than twice the floor.

**Intervals are floor + exponential, never Gaussian and never uniform jitter.** An
exponential above a floor has a flat hazard: having waited tells the participant nothing
about when the next sound is due. Uniform jitter has a *rising* hazard and turns a reaction
time into a synchronisation task. The 3500 ms maximum is there because an uncapped tail
occasionally runs past four seconds, which does not sound like a long gap -- it sounds like
the iPad has stopped working, and the examiner would stop a good trial.

**The digit recordings are not in this build.** `DIGIT_SET` ships empty and the two digit
conditions refuse to be planned. The recordings have to be one speaker, recorded once, all
five in a fixed slot of 450 ms or less, each word at the start of its slot and padded with
silence rather than time-stretched, RMS-matched with peaks limited. **The slot length has to
be measured before the minimum interval is fixed**, because the silence between consecutive
words is the minimum interval minus the slot and must be at least 150 ms.

---

## Data

| What | Line | Search for |
|---|---|---|
| **The trial record** (schema `motiq_prototype/15`) | — | `function buildRecord(` |
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

**What v14 added to the record**, all of it from the specification's own list: the generated
stimulus list as index/type/onset triples rather than three parallel arrays that can be
mis-zipped; which digit each slot played; the actual onsets with both clocks recorded so
they are reconstructible rather than asserted; the planned target count and the table row it
came from; the gap law in force; the audibility outcome on every trial of an auditory
condition; whether practice ran, was skipped, or is not specified for the condition; the
examiner's five entry fields stored separately; the audio path including whether headphones
were used; the template name; and the sentence the examiner was given to say.

`p_target` was REMOVED rather than emptied. It held a target probability and the count is
not sampled from one any more; a reader seeing that column blank would conclude the stream
had no targets.

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

---

## The session, as administered

These are v14 and have no earlier equivalent in this file. A native build that reimplements
the battery without them reimplements a different battery.

| What | Search for |
|---|---|
| The audibility check, both forms | `THE AUDIBILITY CHECK` |
| Practice, inside the session | `PRACTICE, INSIDE THE SESSION` |
| The examiner entry screen | `THE EXAMINER ENTRY SCREEN` |
| What the examiner says before each trial | `const TRIAL_SCRIPTS` |
| The start and end-of-task screens | `function showTaskStartScreen(` |

**The audibility check runs in two forms, and that is a departure from the specification as
written.** It says the check runs once per session at the first auditory condition, which is
right for a session with one kind of stimulus. This battery has two, and they ask different
questions: tones ask whether two pitches can be told apart, digits whether five spoken words
can be understood. Presbycusis takes the high frequencies consonants live in, so a
participant can pass pitch discrimination and still not tell "two" from "three". The tone
form runs once at the first tone condition, the digit form once at the first digit
condition. **A failed check takes every condition of that stimulus kind out of the session**,
each skipped trial carrying the reason.

**Practice is administered to the participant and is not saved.** Seven conditions have one.
What is practised is not the movement but the rule -- the hold, the single-word response, and
on a dual task that neither half may stop while the other is done. There is **no SKIP
control on the dual tasks**, which is the specification being deliberate: the dual task is
the only place a participant can do both halves correctly in isolation and still not do the
task. The practice screens are also the only place in the battery where the correct count is
shown, because practice is where a misunderstanding is supposed to be found.

**The examiner entry screen enforces an order, and the order is the measurement.** Field 1
is what the examiner HEARD; field 2 is what the participant claims. Field 2 is unreachable
until field 1 is in, and field 1 cannot be edited afterwards -- both are judgments about the
same quantity and the second arrives with the answer attached. The two are never merged in
the record. The reported total is scored in three levels against the trial's target count,
not as a continuous error.

---

## Participant-facing wording

| What | Line | Search for |
|---|---|---|
| Instruction assembly | 1514 | `function instructionFor(` |
| The drawing rules, shared across tasks | 1497 | `function drawingRules(` |

The wording has been through many rounds with the study lead and is not placeholder text.
As of v14 the auditory and sustained-spiral wording comes from `MotiQ Task description.docx`
and the per-trial sentences are that document's scripts word for word, in `TRIAL_SCRIPTS`.
A generated fallback covers a plan with more repeats than the document scripts, and the
record says which of the two produced the line the participant actually heard.
Lines wrapped in `**` are the emphasised statement; the plain lines under one are its
bullets. Notable: no em dashes anywhere a participant reads; the task code is an examiner
reminder in small grey type, not a heading; and the instruction names which index finger,
because on `TAP_FAST` the hand alternates between repeats.

---

## What NOT to carry across

- **The pointer diagnostics** beside the pen check on the device page exist to chase the
  Apple Pencil question
  above. If the native build identifies the Pencil correctly, they have no purpose.
- **The palm-rejection workarounds** — `dataHeld` flags, the "pen and touch not
  simultaneous" warnings. The native build removes the cause.
- **The touch-event fallback** in the pen-check pad, which reads the touch stream when the
  pointer stream withholds the pencil. A Safari-specific rescue.
- **The single-file structure.** It is a deployment constraint, not an architecture.
