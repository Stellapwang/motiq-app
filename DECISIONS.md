# MotiQ web prototype — decision log

Every change made on `feat/battery-v12`, oldest first, as written at the time. This is the
reasoning that is NOT in the code: what was tried, what it broke, and why the rule ended up
the shape it is. Several rules here are counter-intuitive and were arrived at only after a
simpler version failed on a real iPad with a real participant task.

Read this before changing any rule in `index.html`. A rule that looks arbitrary usually is
not, and the commit that introduced it says why.

Generated from git; do not edit by hand.

---

## Battery to the study's table: renamed codes, new conditions, held data

The catalogue now matches the Battery tab of the demographics workbook. It
replaces prompt 2's catalogue section, which disagreed with it in almost every
row.

RENAMES, with aliases so nothing already on a device is orphaned. PEN_DRAW becomes
PEN_SPRL, FIN_DRAW becomes FIN_SPRL, VOC_ODD becomes VOC_VODD. Records keep the
code they were written with -- nothing saved is rewritten -- but lookups resolve
through TASK_ALIAS, a saved plan keyed by an old code still finds its settings, and
a queue interrupted across the rename is migrated rather than dropped.

NEW. VOC_VODD is the oddball administered alone, with no drawing and no tapping --
the first condition in the battery with no hand at all, which needed a "none" hand
mode: handsAllowed would otherwise have offered it "each hand" and quietly run it
twice. PEN_TODD is spiral plus tapping the target tones, the manual-response twin
of PEN_VODD, so the pair isolates response channel from monitoring load; nothing
else in the battery does that.

TIERS follow the table, not the previous grouping. The bimanual tapping conditions
move up to core; PEN_VCNT, PEN_TONE and PEN_FAST move down to exploratory. Tapping
conditions go to 3 trials, TAP_TONE to 2 x 50 s at a 1000 ms mean -- about 100
responses, which is what ex-Gaussian fitting needs, with the floor left at 600 ms
because it exists to stop a response being cut off by the next tone. Every spiral
condition gets a 60 s sustained window. (Superseded at v13.01: the window is no
longer something every spiral condition has. Left standing because it is what was
decided here; see "One spiral and sixty seconds of spirals are two tasks".)

DATA HELD is a new state, and it is not a low tier. Blocked means a task cannot
run; exploratory means nobody has decided about it; HELD means it runs and records
but the platform stops it being the measure it is designed as. PEN_FAST, PEN_TONE
and PEN_TODD are held for palm rejection: their UI is built and their trials are
worth collecting for everything except the tap channel, so they stay selectable,
the plan table says DATA HELD in red beside them, and every trial carries the
reason and an instruction not to analyse its dual-task contrast.

PEN_PENT is deliberately absent. It is a pentagon copy, not a spiral, and needs its
own template, rendering and scoring; it is scoped separately.

One reading to confirm: FIN_SPRL is given as "Both, alternating" with 1 trial,
which cannot both be true of a single trial. Implemented as 1 per hand, since a
single trial cannot alternate.

Export schema motiq_prototype/12 -> /13, CSV 190 -> 192 columns. BUILD v12.04.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>

## Group the plan by tier, and split Participant from Checks

VOC_VODD WAS INVISIBLE. The plan table grouped its rows by task-code prefix --
TAP_, PEN_, FIN_ -- and iterated the groups, so any task whose code matched none of
them was simply never rendered. VOC_VODD was in the catalogue, had a plan entry and
a queue path, and could not be ticked because no group claimed it. That is why the
new battery did not appear in the session: the condition existed everywhere except
on screen.

Grouping is now by TIER, ordered within a tier by the administration sequence, so
reading down the page gives the order the session runs in. Every task has a tier, so
nothing can fall outside the grouping the way it could with prefixes -- and a task
whose tier has no group now renders a visible "this is a bug" row instead of
vanishing. Losing a row silently is the failure worth engineering against; the
grouping itself is a preference.

PARTICIPANT AND CHECKS ARE NOW SEPARATE PAGES. They were one page because they are
done in one sitting, but they are different kinds of thing: who the participant is,
versus whether this device and this person can produce usable data today. The first
is typed once from paperwork; the second is worked through with the participant in
front of you, and two of the three write their own records.

The step numbers in the tab labels and the page element ids no longer match -- the
checks page is element 7 and step 3. Renumbering the elements would have meant
rewriting seventeen goPage() calls whose numbers are load-bearing, and one missed
call is a dead button, so the tab order is expressed as data instead. The resize
handler that lays out the dot-tap pad follows it to the new page.

BUILD v12.05. No schema change -- this is presentation and navigation only.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>

## Trim the plan to what runs, and drop two fields that asked for nothing

Six changes, all from review of the running build.

ADMINISTRATION LANGUAGE IS GONE. It asked the examiner to declare something that,
with one registry entry and an English-only UI and instruction text, could only
ever be English -- so recording it implied a choice that did not exist. The
LANGUAGES registry stays, because it is what supplies PEN_VCNT's counting floor and
that constant needs one findable home rather than being buried in the audio
defaults. administration_language is still exported, as a fixed fact rather than a
collected one, because an analysis joining these records needs it without reading
the build. The moment a second language is administered, this is the field to bring
back and the registry is where it reads from.

PARTICIPANT LANGUAGE stays -- it is in the study's demographics table, and testing
someone in a language they are not strongest in depresses exactly the verbal tasks
this battery scores. Now free text, since its option list is still to be decided,
with the mismatch warning kept.

"SELF-PACED TRIAL ASSUMED TO LAST" IS GONE. It existed to give the session-length
projection a number for trials with no length until they were run. Every spiral
condition now runs a fixed sustained window, so their length is known before the
session starts and the assumption has nothing left to stand in for. A field that
changed nothing about what was administered, and can no longer change the
projection either, is worse than absent: it invites tuning that does nothing.

The handedness field is labelled Handedness rather than Dominant hand, and the
nominated side for an ambidextrous participant is marked required -- every task set
to "the dominant hand" needs a definite side, and choosing one silently would put an
arbitrary decision into the data unrecorded.

VOC_VODD and PEN_VODD move to exploratory. They were core and greyed, which is the
worst of both: presented as essential, shown as broken.

EXPLORATORY TASKS ARE NO LONGER LISTED BY DEFAULT. Fourteen rows of which nine were
exploratory made this look like a fourteen-task battery, and greying them read as
broken rather than as available. They are pulled in from a picker at the foot of the
group, and a pulled-in task is an ordinary row -- same controls, not greyed,
removable again. The plan now opens on the six conditions that actually run.

BUILD v12.06. No schema change.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>

## Matched sequence seeding, a required nominated hand, VODD back to core

MATCHED SEEDING, and it is now the default. The question was whether the stream is
the same between subjects and different between tasks. It was neither: the two modes
were "random", a new stream every trial, and "fixed", seed 1 for EVERYTHING -- which
gave every task the same stream as every other task, so TAP_TONE and PEN_VODD were
the same sequence and any comparison between them was confounded with having heard
it twice.

The new mode derives the seed from the task, the hand and the repeat number, and
from nothing else. Participant id is deliberately absent, which is the whole point:
every participant hears the identical stream on a given task and repeat, so a group
difference is a difference in the participants rather than partly a difference in
the tones they happened to get. Repeat number is in the seed because two repeats of
one task should not be the same stream -- the second would be partly a memory test.
The repeat is read the way nextIds() will assign it at save time, so the seed matches
the repeat it is filed under. Verified: two participants get identical seeds on the
same task and repeat, different seeds on different tasks, and different seeds on
repeat 2 which are still identical to each other.

THE NOMINATED HAND IS NOW REQUIRED. It defaulted to Right, so an examiner who never
looked at the field still produced a definite side in the data with nothing recording
that it had been defaulted rather than decided. The select starts unset, dominantSide()
returns null rather than inventing "right", and building a trial list or running an
ad-hoc trial is blocked until a side is chosen -- the same treatment a missing
participant ID gets, because it is the same kind of gap. Most of the battery is
specified as "the dominant hand" and an ambidextrous participant does not have one
until someone says so.

VOC_VODD and PEN_VODD return to CORE, per the battery table. They were moved out
because they appeared greyed, but the greying was voice-consent blocking, not a
statement about their tier -- moving them was treating the symptom.

BUILD v12.07. No schema change.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>

## Denser oddball, recorded target spacing, and a reachable consent box

DENSER STANDARDS, RARE TARGETS. VOC_VODD, PEN_VODD and PEN_TODD move from an 800 ms
mean and 400 ms floor to 500 ms and 300 ms. Over 60 s that is ~120 tones and ~30
targets against ~75 and ~19. Rarity is what makes this a vigilance task, so the
density comes out of the MEAN and the target probability stays at 0.25 -- raising
the target rate to get more events would spend the thing that made the design worth
having.

ADJACENT TARGETS WERE ALREADY IMPOSSIBLE, which is worth stating because the request
was to disallow them and the honest answer is that makeSequence has always drawn
positions so that two targets cannot touch -- not improbably, but by construction.
What was missing is that it was never RECORDED: adjacent_targets_allowed was a CSV
column nothing ever set. It is now exported as false alongside min_target_gap_tones,
so a guarantee appears in the data as a guarantee rather than as an empty cell.

The denser stream exposes a real constraint, and the comment in the first draft of
this change got it wrong, so it is worth being exact. Non-adjacent means a minimum
gap of TWO intervals, which at a 300 ms floor is 610 ms measured -- not the ~1 s the
mean invites you to assume. A spoken number takes about 800 ms, so on the closest
pair the second target can arrive before the first response is finished. It is rare,
it costs a scoreable response rather than corrupting the trial, and both
min_target_gap_tones and min_isi_ms are exported so how often it actually happened
is answerable. The fix, if it matters, is a three-tone minimum separation; that is a
decision about the paradigm and is left to be made deliberately.

THE CONSENT BOX IS NOW REACHABLE. Six of the fifteen conditions record voice, so an
unticked consent box greys out most of the battery, and the row only said where the
box lived -- which on a long page is not the same as being able to get to it. A
blocked voice row now offers a button that jumps to the box and marks it. It does
NOT tick it: consent is the participant's, not a formality to be auto-satisfied
because it is inconvenient.

BUILD v12.08. No schema change.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>

## Pentagon copy, and no clock on a sustained spiral

TWO FIXES IN ONE, because the first is what the second would have inherited.

PEN_VODD SHOWED A RUNNING CLOCK. "untimed" means the TRIAL has no automatic end,
which is true of every spiral -- they all stop at END so the participant can finish
the stroke they are in -- but a sustained spiral also has a WINDOW, and the HUD
branched on the wrong one of the two. So a 60 s windowed trial displayed an elapsed
count, which is exactly the pacing cue the no-clock rule exists to remove: hold back
early, sprint at the end, and the within-trial decrement being measured is the thing
that moves. A sustained trial now shows nothing while it runs and STOP when the
window closes. Genuinely untimed spirals keep their elapsed readout, which is
examiner information on a task with no window to pace against.

PEN_PENT, the intersecting pentagon copy. The eighth core condition and the only one
from the battery table that was missing.

It is not a spiral and reuses almost nothing from one. There is no template to
trace, so the model is shown BESIDE a blank area rather than under the pen, and it
goes on the side the drawing hand is not -- a model the participant's own hand
covers is not a model. The copy area is marked with a dashed box and labelled,
because an empty half of a white screen does not say "draw here" and a participant
copying into the model's half is a lost trial rather than a bad copy. Its
instructions are its own: the spiral rules tell you not to lift the pen, which is
wrong for a figure drawn in separate lines.

THE APP DOES NOT SCORE IT, and says so in the record. The criterion is ten angles
present and the overlap forming a four-sided figure, which is a judgement about a
shape rather than a threshold these strokes can be put through. What it records is
the drawing and the model that was in front of the participant, with the vertices in
the same millimetre frame as the pen samples so the two can be overlaid directly.

One caveat is written into the source rather than left to be discovered: the
vertices are CONSTRUCTED -- two regular pentagons, one rotated, separated so the
overlap is a quadrilateral -- and are not pixel-identical to the MMSE or MoCA
figure. A copy scored against published norms for those instruments would be scored
against a figure the participant did not see. Everything reads from pentagonModel(),
so swapping in an instrument's real vertices is a one-function change.

Found while testing: an early return from instructionFor handed back a bare array
where every caller expects an object, which took the instruction screen down with a
TypeError before the trial could start.

Export schema motiq_prototype/13 -> /14, CSV 192 -> 195 columns. BUILD v12.09.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>

## Size the pentagon figure to the screen instead of to a guess

The model and the copy area were both built around a 22 mm side, which was a number
chosen against an imagined screen rather than measured against a real one. On the
iPad it left most of the area empty and made the angles harder to judge than they
needed to be -- and the angles are the scoring criterion.

The side length is now SOLVED from the space actually available, from both
dimensions, taking whichever binds. And because the pair is much wider than it is
tall, which way the screen is split decides how large the figure can get -- and the
answer flips with the aspect ratio. Both splits are solved and the larger wins:
side by side on a landscape screen, stacked on a portrait one. Fixing one
orientation would have thrown away half the area on the other.

Measured: 22 mm side before, 51.7 mm side by side in landscape and 43.4 mm stacked
in portrait. The figure is between two and two and a half times larger either way.

Both halves get the SAME side length, and the copy box is sized from the model plus
one margin rather than from the half-width. A copy task where the model and the
space to copy into are different sizes is asking for a scaled reproduction, which is
a different task.

One thing the stacked layout broke and this fixes: the drawing origin. toMM and
toCanvas were anchored on the vertical centre of the stage, which on a stacked
layout is the DIVIDER, not the middle of the copy area -- so strokes would have been
recorded relative to a point the participant was never drawing around. The origin
now follows the copy half.

The rendered side_mm goes into every record, which matters more now that it varies:
two iPads of different sizes present different-sized models, and a copy can only be
compared with the model it was actually made from.

BUILD v12.10.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>

## Unpredictable target counts, a real tail guard, and no false route alarms

Four fixes from review of the running build.

THE COUNT WAS ALWAYS THE SAME NUMBER. With n and p both fixed, the target count
was round(n*p) on every oddball trial -- 30, five times in a session, across
VOC_VODD and four PEN_VODD trials all 60 s long. A participant who notices that
stops counting and reports the number they remember, which scores perfectly and
measures nothing. The count is now jittered around n*p from the seed, so it varies
between trials while staying identical across participants for a given task and
repeat. Measured over 40 sequences: counts from 24 to 35, twelve distinct values.

THE GAP NOW GROWS WITH THE COUNT. Saying "twenty-seven" takes longer than saying
"seven", so spacing that is comfortable at the start of a trial is not comfortable
at the end. Targets after the ninth require 1150 ms rather than 800 ms.

And the first attempt at that got it wrong in a way worth recording. Placing
targets from a shuffled list cannot check this rule, because the gap a target needs
depends on how many came BEFORE it -- and a shuffled pass does not yet know the
ordinal. It left one to three violations in every stream and looked like it worked.
Placement now walks forward in time, where the ordinal is simply how many have been
placed. Measured over 40 sequences: zero violations, no adjacent targets.

NOTHING LANDS AT THE VERY END any more. A target in the last moments left no room
for the response and the beep itself could be clipped by the window closing. The
last target is now held 1400 ms clear of the end; measured clearance ran 1585 ms to
3876 ms.

THE AUDIO ROUTE ALARM WAS FIRING ON ITSELF. The fingerprint included the microphone
device, and the microphone is acquired one lead-in AFTER the trial starts -- so on
every voice trial the fingerprint gained fields partway through and the watcher read
its own recorder starting as the route changing. It printed "AUDIO ROUTE CHANGED
mid-trial" in red on trials where nothing had changed, including trials where the
mic had failed. The watch is for OUTPUT latency, which is what a speaker-to-headset
switch actually changes, so the input device is now recorded as context at the start
and end rather than used as a trigger.

Also: the plan table showed a self-paced/fixed dropdown next to the sustained window
on spiral conditions -- two controls for one quantity, one of them inert since
sustained windows arrived. Gone, and "fixed window" is replaced by what the
participant actually does.

And the admin bar is 86 px down to 54 px, with the top padding following it, which
is 10 mm of drawing height returned. The spiral is 120 mm across at the default
pitch and turns, so the bar was costing about a fifth of it. The end zone goes from
0.8 to 0.95 of the pitch -- the neighbouring turn is a full pitch away, so the
margin was larger than it needed to be and the zone was small enough that finishing
a spiral meant aiming.

BUILD v12.11.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>

## Make the spiral end itself reliably, and cut the instructions down

THE END ZONE WAS NOT THE PROBLEM. "I reached the end and it did not advance" had a
cause, and it was in the angle detector: the accumulator kept its previous angle
across a PEN LIFT, so a participant who lifted at one point and touched down at
another had that jump read as travel. Touching down "behind" where they lifted
SUBTRACTED. Measured: one full turn became half a turn from a single lift. Angle
already earned quietly disappeared, and the spiral then refused to advance although
it looked finished on screen. The reference is now cleared on every lift; the total
is kept.

And the zone was the wrong SHAPE. What proves a spiral is finished is the angle --
net angle cannot reach the template's total except by going round it. The circle was
never evidence of completion; it was there to stop someone scribbling small circles
near the centre, which accumulates angle without moving outward. A small circle asks
that question badly: it turns finishing into an aiming task, and aiming is not what
is being measured. A RADIUS BAND asks the same thing and cannot be missed -- the pen
has to be out near the rim, which it unavoidably is if the spiral was traced. The
circle stays as a visual marker and now LIGHTS UP once the angle condition is met,
so "you have drawn enough, head for the end" is visible rather than inferred.

The examiner button stays as the exception path it was always meant to be. Tapping
it every pass would put the examiner's reaction time into every boundary and take
their eyes off the participant to watch the pen, which is worse than either problem
it would solve.

INSTRUCTIONS: fewer lines, and one of them carries the task. Five equal bullets mean
five things to hold, and the one that matters gets no more weight than "sit
comfortably". Each task now leads with a single emphasised line in blue saying what
to DO, with support beneath it. The spiral says TRACE rather than draw. The
sustained conditions get their own emphasised line saying a new spiral appears and
to keep going until STOP -- the single thing most likely to be missed, since a
participant who finishes one spiral and stops has ended the trial as far as they are
concerned. PEN_SPRL is four lines where it was six; VOC_VODD four where it was
seven.

Two lines were wrong rather than merely long. "Keep your voice up even while you are
drawing" was read out on VOC_VODD, which has no drawing. And the footer said "ends
when the administrator stops it" directly under a line promising STOP.

BUILD v12.13.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>

## Stop the route alarm firing on itself, and make the required fields required

THE AUDIO ROUTE ALARM HAD A SECOND CAUSE, and the first fix did not reach it.
Removing the microphone from the fingerprint was right but not enough:
outputLatency reads 0 until the AudioContext is actually running and then jumps to
its real value, so the first sample of every tone trial was taken before the stream
started and the second was not. As a STRING, 0 -> 0.032 is indistinguishable from
unplugging a headset. The value also carries small jitter between reads, which five
decimal places turn into a difference.

So the test is no longer string equality. It compares output latency numerically
with an 8 ms tolerance, and a transition out of zero is not a change at all -- it is
the context waking up. A speaker-to-headphone switch moves latency by tens of
milliseconds and clears that comfortably. Measured: 27 samples across a tone trial,
zero changes, where the old test flagged the trial.

REQUIRED FIELDS are now required. Participant ID, sex assigned at birth, birth year,
birth month and handedness carry a red mark and the session will not start without
them. Handedness is there for the same reason as the ID: the battery is specified in
terms of the dominant hand, so without it there is no hand to run half the trials
on. Missing fields are outlined and scrolled to, not just listed -- on a page this
long, a list of field names is not the same as knowing where to look.

INSTRUCTIONS. The spiral now names the green circle as the starting point, makes
"one continuous line, do not lift" its own emphasised line, and says explicitly that
finishing one spiral brings another and to start again from the middle. The dual
tasks had been reduced to "exactly as before", which assumes the participant still
holds four rules from a screen several trials ago; the two that decay first --
accuracy under load, and not lifting -- are repeated. The pentagon gets the size
disclaimer, without which participants try to match the model's dimensions when the
criterion is the angles and the overlap.

THE INSTRUCTION SCREEN WAS CUT OFF AT THE TOP, and the cause is worth recording:
justify-content:center on a scrolling flex column pushes the top of an overflowing
column outside the scroll area, where scrolling cannot reach it. It is exactly the
first line -- the one saying what to do -- that disappears. Now "safe center", which
centres only while there is room, and the body is scrolled to the top on open.

Also: tone length 50 ms to 100 ms; the count prompt no longer tells the examiner to
ask the participant, which is what the heading already says; and the Parameters page
explains what the Tones column counts, that the high-tone count is jittered per trial
so it cannot be learned, and that intervals are a floor plus an exponential rather
than even or Gaussian.

BUILD v12.14.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>

## Rehearse the sustained spiral in the checks, before it costs a trial

The one thing about the sustained conditions that cannot be conveyed in a sentence
is that finishing a spiral clears it and puts up another. A participant meeting that
for the first time DURING PEN_SPRL spends the first pass working out what happened,
and that pass is data. Here it costs nothing and can be repeated as often as needed.

It uses the SAME advance rule as the trial -- net accumulated angle plus the radius
band -- rather than an approximation, so what is rehearsed is what happens. The end
marker lights up at the same threshold. Three turns instead of six, because the point
is the transition, not the endurance: a practice run takes seconds rather than a
minute.

The examiner demonstrates and hands the iPad over. A counter says how many spirals
have been finished, and the first advance says plainly what just happened, because
the behaviour is more convincing seen than described. Nothing is recorded -- no trial
record, no screening record.

One bug found in the practice was the trial engine's own, reintroduced: ending the
stroke on advance. The pen is still down when a spiral completes -- the advance
happens under the participant's hand -- so clearing the stroke would make them lift
and touch again to carry on, which is the opposite of the behaviour being rehearsed.
Verified: four consecutive spirals without lifting, and a fifth after a deliberate
lift.

Also fixes something the page split left behind in v12.05: the resize handler still
watched page 2 for the check pads, which had moved to page 7, so they stopped
re-laying-out on rotation -- the one orientation change an iPad makes most. Both pads
now redraw, and both are laid out when the page is opened.

BUILD v12.15.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>

## Stop the spiral advancing on its own, and fold the pen check into the practice

THE SPURIOUS ADVANCE HAD A MEASURABLE CAUSE. Angle was counted from 0.5 mm out,
while the start zone is 4 mm across -- and near the centre a small movement sweeps a
huge angle: at 1 mm out, half a millimetre sideways is about 30 degrees. Measured:
three seconds of a hand settling inside the start circle produced 35 turns, against
the 5.7 needed to advance. The next time the pen reached the rim the pass advanced
and the drawing was discarded, which is exactly what was reported. Angle is now
counted only outside the start zone, and a pass cannot end in under four seconds --
nobody traces six turns that fast, and the cost of being wrong is the participant's
drawing.

That fix then broke advancing altogether, which is worth recording because the two
faults look nothing alike and the second was introduced by the first. On an
Archimedean spiral the radius grows in proportion to the angle, so gating the inner
4 mm of a 60 mm spiral silently swallows 4/60 of the turns -- 0.4 of six, leaving 5.6
available against a 5.7 threshold. The spiral could be traced perfectly and never
advance. The threshold is now a fraction of the angle ACTUALLY REACHABLE outside the
gate: 5.32 needed against 5.60 available.

THE PEN CHECK IS GONE, folded into the spiral practice. They were the same drawing
twice: a participant traced a spiral to prove the stylus registers, then traced
another to learn the task. The practice now measures sample rate, pressure range and
pointer type from the first completed spiral on each hand -- over a whole spiral
rather than whatever stroke was made, so the rate estimate depends less on stroke
length -- and PENCHECK is filled exactly as before.

Also in this pass: the end-of-window screen is a calm END with "you can stop
drawing" instead of a full red STOP, which reads as an error when the participant has
just done nothing wrong. A progress arc fills round the spiral as the pass nears
completion, so neither the participant nor the examiner has to guess whether the next
one is imminent. The pentagons are tilted by different amounts -- two upright ones
meeting vertex-to-vertex give an overlap whose axes can be read off the page, and a
figure easier than the instrument it is modelled on. "Say the new number each time"
is cut, "Play both" becomes "Practice — count the high ones", the pentagon
instruction loses a line, and a finger task no longer shows a drawing of a hand
holding a stylus.

One start-up fault fixed along the way: applyCal() was laying out the practice pad,
which runs above the practice code, so every constant that code touches was still in
its temporal dead zone -- and a const read there throws rather than returning
undefined, taking the rest of the script with it. The pad is laid out when the page
opens, which is when it can be seen.

BUILD v12.16.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>

## Say which parameters are not this build's, and name the task in the END caption

"I don't see anything changed on the Parameters tab" was right about the numbers
and wrong about the cause, and the cause is worth fixing rather than explaining
again.

The page IS this build's -- checked in the rendered DOM rather than in the file: the
geometry prose is trimmed, the Tones column is explained, the interval distribution
is described, and tone length is 100 ms in memory. What overrides it is the SAVED
SESSION: restoreState applies the parameters a session was saved with, which is
correct, because an examiner's edits should survive a reload. But it also means a
value changed in a new build does not appear until the session is cleared, and the
only symptom is a number that looks unchanged. That is indistinguishable from the
change not having happened, and I have now explained it twice, which is the sign it
should be the app's job rather than mine.

So the build's own defaults are snapshotted before any saved state can overlay them,
and the Parameters page lists every value that differs, saying what the build ships,
with a button to restore them. A setting that silently ignores the build is a trap; a
setting that says "this came from your saved session, the build ships 100" is a
setting.

And the END caption said "you can stop drawing" on every task, including the tapping
ones where nobody had drawn anything. It now names what the participant was actually
doing -- "you can stop tapping", "you can stop drawing and counting" -- assembled
from the task rather than written per task, so a new condition cannot be added
without one.

BUILD v12.17.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>

## Let untouched parameters follow the build, and keep only the deliberate ones

The previous attempt told the examiner which values were stale and left them to fix
it, which is not a fix -- it is the same trap with a label on it. "Am I expected to
see all Tone changed to 100? They are still 50" is exactly the right question, and
the honest answer was no, you were expected to notice a warning and press a button.

The reason parameters were restored wholesale was that an examiner's edits must
survive a reload, and there was no way to tell an edit from an old default: both are
just a number in the saved session. Saving what the build SHIPPED alongside the
values supplies the missing half. A saved value that still equals the default it was
saved against was never touched, so it follows this build. A value that differs was
chosen, so it is kept.

Verified against a session built the way an old build would have left one: tone 50
everywhere plus one real edit. After restoring, tone reads 100 across all seven rows
and VOC_VODD's mean interval picks up 500 -- neither was ever touched, so both follow
the build -- while the edited floor of 900 survives and is the only thing listed as
differing.

What remains on the Parameters page is a list of values somebody actually chose,
which is worth showing, with a button to adopt the build's instead. Nothing stale is
silently in force any more, so the list is short and means something.

A session saved before this build carries no record of what it shipped against. Those
are treated as edits and kept, which is the safe direction: a deliberate value
preserved is recoverable, a deliberate value overwritten is not.

BUILD v12.18.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>

## Let a pre-migration session follow the build too

v12.18 made untouched parameters follow the build, but only for sessions saved by
v12.18 or later. A session saved before it carries no record of what its build
shipped, so nothing could be told apart from a deliberate edit -- and those were
kept. That was the cautious choice and it was the wrong one: every parameter changed
since stayed frozen at the old value, silently, for the life of the session. Which is
precisely the failure the change existed to end, preserved in the name of caution.
The reported symptom was the same as before: tone still reads 50.

So for a session with no record of its defaults, THIS BUILD'S VALUES STAND, and the
saved ones are set aside and listed on the Parameters page with a button to put them
back. Nothing is lost, nothing stale is in force, and a value that really was
deliberate is one click away.

Verified by reproducing the reported session exactly -- tone 50 and floor 900
throughout, no defaults record. After restoring: tone reads 100 across all seven
rows, floors return to 600/300/800, the oddball means to 500, and the 17 displaced
values are listed and recoverable.

The direction was worth reconsidering rather than defending. Losing an edit is
recoverable when the old value is shown next to the new one; running a whole session
on parameters nobody chose, with nothing on screen saying so, is not.

BUILD v12.19.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>

## Two tone streams instead of seven rows of numbers

You proposed this, I agreed it was right, and then waited for a confirmation instead
of building it. Building it.

There were seven per-task rows and most of the differences between them were
historical rather than reasoned -- PEN_TONE at 1200 ms against TAP_TONE at 1000 ms
said nothing about the two tasks, only that they had been edited at different times.
Seven independent rows also gave seven chances for a single-task baseline and its
dual-task partner to drift apart, which turns a dual-task cost into a cost plus a
stream difference.

Now two, distinguished by what the participant has to DO:

  every    a response to every tone
  oddball  a response only to the rare high tone, with dense standards so that
           rarity means something

Everything else is derived and therefore cannot drift: the lead-in from whether
there is drawing to get going first, the length from the trial, and the floor from
the RESPONSE CHANNEL.

Two things the attempt got wrong on the way, both worth recording because both look
like details and are not.

THE FLOOR MEANS A DIFFERENT THING IN EACH STREAM. One rule for both does not merely
read badly, it throws: a 500 ms mean cannot sit above an 800 ms floor. Where every
tone is answered the tone-to-tone gap IS the response gap, so the floor is the
response time. On an oddball the standards are not answered -- their spacing only has
to be perceptually separable, and it should be short, because dense standards are
what make the target rare. The response constraint there applies to target-to-target
spacing and is enforced where targets are placed. PEN_TODD now gets tighter targets
than the spoken oddballs, correctly, because a tap is quicker than "twenty-seven".

AND THE MEAN HAS TO FOLLOW THE FLOOR. Fixing one mean for both channels put TAP_TONE
at 41 tones where ex-Gaussian fitting needs about 50 per trial -- I had unified on
PEN_TONE's historical number without noticing what it cost. The gap between floor and
mean IS the randomness, so they move together: a fixed ratio gives a tap floor of 600
a 1000 ms mean, which is 50 responses per trial and 100 across two, and a voice floor
of 800 about 1340 ms, because a spoken number cannot be asked for every second.

Verified: TAP_TONE and PEN_TONE now identical by construction, VOC_VODD and PEN_VODD
likewise, and TAP_TONE back to exactly 100 responses across its two trials.

BUILD v12.20.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>

## Parameters by stream, and eight fixes from the iPad pass

PARAMETERS: THREE ROWS, NOT SEVEN.

The seven task rows were still there because only the numbers behind them had been
unified, not the page showing them. Seven rows invite seven answers to a question
that has three, and every pair of numbers that can differ eventually does -- which
is how a baseline and its dual-task partner ended up on different streams in the
first place. Rows are now stream x response channel, because the floor is a
property of the RESPONSE:

  every / tap      a tap finishes fast, so tones can come about every second
  every / spoken   a spoken number takes ~800 ms, so they cannot
  oddball          standards are not answered at all, so their spacing is not a
                   response time

Editing a row moves every task in it, and there is deliberately no way to set one
member alone -- including from the Next-up panel on the session page, which was
editing a single task from a panel that does not show its partner. Length and
lead-in left the table entirely: both are derived, so neither can drift.

The drift report printed the same three edits across seven tasks as twenty-two
lines, which reads as twenty-two problems. One line per row now. The tone count in
that table also read selfPacedDurS -- the leftover "assume 180 s" guess -- and so
reported 180 tones for a task running a 60 s window.

THE SPIRAL ADVANCED WITH THE OUTER FIFTH UNDRAWN. The radius band was 0.82, so the
template vanished under the pen partway round the last turn: a lost fifth of every
pass and an unpleasant surprise. The angle test already asks 95% of the reachable
sweep, so 0.92 is the matching radius and neither test now fires far ahead of the
other. The last few per cent stay unrequired, and Next spiral still covers a
participant who stops short.

AND THE END OF A SILENT SPIRAL ANNOUNCED ITSELF FOR 1.8 SECONDS. Everywhere else
the sign acknowledges a cue the participant already has -- tones stopping, the
examiner speaking. On PEN_SPRL there is no such cue: at 60 s the ink simply stops
counting, which looks like nothing happening. A sign that comes and goes while
someone is looking down at their own drawing is the same as no sign. It now stays
up until the examiner moves on.

Also:

- The participant's instruction screen led with the task code and the catalogue
  label at 27 px, directly above the emphasised line -- the same thing twice, once
  in a code nobody outside the study reads. The code is an examiner reminder and is
  now sized as one; the instruction is the only thing written for the participant.
- The spiral instruction still said "keep going until you see STOP". The sign has
  said END since v12.16.
- A voice-only task drew a hand selector under Run now, offering a choice the trial
  does not use.
- The Length column spent three lines saying a new spiral appears. One now.
- "Event" named nothing -- it is "Log event", and "Logged (n)" once pressed.
- Commentary removed from four user-facing strings, including the paragraph
  explaining that the intervals are not Gaussian. Nobody expected them to be.

BUILD v12.21.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>

## Spiral practice took a finger and ignored the Apple Pencil

Nothing in this file was rejecting the pencil -- neither practice handler looks at
pointerType at all. The events were never arriving intact.

The trial canvas gets away without any of this because the stage locks the body:
position fixed, overflow hidden, touchmove and the gesture events swallowed
document-wide. There is no competing page gesture for Safari to award the pencil
to. The Checks page is an ordinary scrolling page, and on iPadOS a pencil drag
there is a candidate for scrolling, for selection and for Scribble. Safari settles
that by issuing pointerdown, deciding the gesture belongs to the page, and then
sending POINTERCANCEL -- which was wired to the same handler as a pen lift, so the
stroke ended before a single sample was kept. A finger is unaffected because
touch-action:none settles it for touch, and this file already records that
touch-action alone is not enough on iPadOS (see the stage guard).

setPointerCapture on pointerdown fixes it: once captured, every event for that
pointer comes here regardless of what the page would rather do with it. Belt and
braces for the window before the capture takes effect: non-passive touchmove and
gesture handlers on the pad itself.

A cancel is also no longer silently treated as a lift. If a device still cancels a
pencil despite the capture, the pad says so in words rather than looking like a pad
that does not take a pencil.

TWO THINGS THIS BUG COULD ONLY HAVE BEEN FOUND WITH, both now present.

The Implement selector recorded what the examiner MEANT to use and nothing ever
compared it with what drew the spiral -- so a spiral drawn with a finger was filed
as a stylus reading, with a "Pointer: touch" buried mid-sentence as the only
evidence. It now says so outright. Mouse counts as a stylus, for the same reason
drawOK accepts it in the trial: it is how the app is reviewed on a desktop.

And PENCHECK kept the first completed spiral per hand and ignored every later one,
so a first attempt that went wrong -- a pencil the pad refused, a finger used by
mistake -- was frozen in as that hand's reading and no amount of trying again
replaced it. That is the exact trap this bug walks into. Start again now means
start again.

Verified: a synthetic pen stroke traces and advances, a cancelled pen stroke names
itself, touch under Implement=Stylus is flagged, mouse is not, and a retry replaces
the stored reading (touch then pen).

BUILD v12.22.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>

## Consent reads as a blocker; developer notes stop pretending to be for the examiner

CONSENT IS RED UNTIL ANSWERED, GREEN ONCE TICKED. It was the only item on the
Participant page that blocks work, and it looked like every other tick on it -- six
of the fifteen conditions cannot be planned without it, so an unticked box is the
reason most of the battery is greyed out two pages later. Nothing about its
appearance said so.

THE HTTPS NOTE MOVED, AND STOPPED BEING A FIXED PARAGRAPH. It sat beside the
consent tick, which put a fact about the URL in the middle of a question about the
participant. They are different things, and the one an examiner can act on is the
tick. It is now item 3 of the developer notes, written live: it always says which
way round the current address is, rather than appearing only when broken and
leaving silence to mean "fine".

"KNOWN LIMITATIONS" WAS NEITHER. Not a caveat about the study, which is what it
read as to whoever opened the app, and not addressed to them: all four items are
addressed to whoever builds the native instrument. It says so in the heading now,
and in a first line stating that nothing in the box is an examiner action.

The prose was also about three times this length. The reasoning belongs in the
source comments, which is where it now lives -- a box nobody finishes reading
states nothing at all.

AND THE FIRST TAB IS NOT ABOUT THE SCREEN. It carries the developer notes, the card
calibration and the app-state reset: all per-device, done once. "Device".

BUILD v12.23.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>

## Make the practice pad say what it is receiving, and honour the implement

I have now guessed twice at why the pencil does not draw here, and each guess costs
a round trip to an iPad that is not on this desk. This is the CHECKS page -- telling
an examiner whether their hardware reaches the app is the job it exists to do -- and
it was answering with a silent canvas.

It now reports the input it receives, counted in two places, because they answer
different questions:

  pad    what reached the canvas
  page   what reached the document at all, via a capture-phase listener

Zero pen on the page means Safari is not delivering pencil events to this page and
no handler on the pad can recover it. Pen on the page but none on the pad means
something between the two is taking them. Pen on both, with cancels, means the
capture is not holding. Those are three different faults and they were
indistinguishable.

The no-stylus verdict is gated on the examiner having actually tried with the pad
set to Stylus, so a single finger tap during setup does not accuse a pencil nobody
picked up.

AND THE PAD NOW HONOURS THE IMPLEMENT SELECTOR, which is the second half of the
report: the whole rectangle was drawable with a finger. The trial canvas has always
honoured it -- drawOK rejects a finger on a stylus condition and a stylus on a
finger condition -- and the practice pad, which exists to rehearse the trial,
accepted anything that touched it. A pad set to Stylus drew under a palm, a knuckle
or a sleeve and filed the result as a stylus reading. Mouse passes as either, for
desktop review.

This is a diagnostic, not a claimed fix for the pencil. The capture from v12.22
stays in place; if it is working, the readout will show pen on the pad with no
cancels, and the next thing to look at is elsewhere.

Verified: finger on a Stylus pad draws nothing and says why; pen draws; a cancel is
counted and named; a stylus on a finger pad is refused; a quiet pad accuses nobody.

BUILD v12.24.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>

## Drive the practice pad from the document, and stop confusing two opposite faults

Stylus draws nothing, Index finger draws. Two changes, one of which may be the fix
and one of which will say so either way.

THE HOLE IN YESTERDAY'S DIAGNOSTIC: it counted pointerdown and nothing else. If
iPadOS delivers the pencil's hover and movement but never resolves the contact into
a pointerdown -- which is exactly what a contact taken as a system gesture looks
like from in here -- then a pointerdown-only counter reports "no pencil at all",
which is the same answer it gives for a pencil that is flat. Those are opposite
problems with opposite remedies. Every pointer event type the pen produces is now
counted and named, so "seen, hovering, never lands" appears as itself.

AND THE HANDLERS MOVED TO THE DOCUMENT, capture phase, hit-tested against the pad's
rectangle, instead of sitting on the canvas.

Listening on the element assumes the event reaches the element, and that assumption
is the thing in doubt. Anything that stops a pencil event on its way down the tree
-- a Safari gesture recognizer, an ancestor handler, a stray overlay -- leaves a
canvas listener waiting for something that never arrives, and from inside the
listener that is indistinguishable from hardware sending nothing. At capture phase
on the document there is nothing above left to intercept: if the browser dispatches
into this page at all, this sees it first. The canvas keeps a listener of its own,
purely as a counter, so "pad" still means what reached the element and "page" what
reached the document -- the two disagreeing is now itself the finding.

The stage swallows touchmove and the gesture events document-wide while it is up,
and that, not touch-action, is why the trial canvas never loses a pencil stroke --
this file already records that touch-action alone is insufficient on iPadOS. The
pad now gets the same treatment, scoped to its own rectangle so the rest of the
Checks page still scrolls.

Verified through the document path: a hover-only pencil is named as such, a full pen
stroke records 30 samples, a finger on a Stylus pad is refused.

BUILD v12.25.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>

## Read the touch stream when the pointer stream withholds the pencil, and make the pad reportable

"No readout line" is itself information, and it points at the gap in the last two
builds: everything was watching Pointer Events. On iOS a Touch carries `touchType`,
"direct" for a finger and "stylus" for an Apple Pencil. If Safari is delivering the
pencil as a touch and withholding the pointer event, nothing in the Pointer Events
API can see it -- from in there it is indistinguishable from a pencil that is not
in the room. That is the one hypothesis nothing so far could test, and it is now
both tested and, if true, fixed:

  - touch events are logged alongside pointer events, with touchType named;
  - a stylus touch on the pad with no pen pointerdown ever seen ARMS A TOUCH
    FALLBACK that drives the pad from the touch stream directly.

Safari fires pointerdown before touchstart, so on a device where pointer events
work the fallback never arms; and a pen pointerdown arriving later disarms it,
because leaving it armed would feed every stroke in twice -- once from each stream.
Found that by testing it, not by reasoning about it.

WHAT TO SEND, ANSWERED. There is now a Send input diagnostics button that POSTs
build, user agent, maxTouchPoints, every counter, and the last 48 raw events to the
review server's existing /report endpoint -- the same endpoint the pointer probe
used, so a 40-line report does not have to survive transcription between two
devices. Where that endpoint does not exist it falls back to a textarea to copy.

Totals alone were not enough for that report anyway: a pointerdown followed
instantly by a pointercancel, and a pointerdown followed by nothing, have identical
totals and opposite causes. The raw log keeps the order, collapsing runs so one
stroke is not four hundred identical lines.

The readout also carries the build string now. "Which build is this" should not be
a question that has to be asked.

Verified: a stylus-only touch stream draws 10 samples from 10 moves with zero
pointer events; both streams together still draw 10, not 20; the send button
reports success or falls back to copyable text.

BUILD v12.26.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>

## The stylus never reaches the browser as a stylus -- say so, and stop blocking on it

Diagnostics from the iPad, and they are unambiguous:

  pen event types: none
  touch.touchType seen: direct=140
  pointerdown on document: touch=2

Not one pointer event with pointerType "pen", not one Touch with touchType
"stylus", in a hundred and forty contacts. On iPadOS Safari a paired Apple Pencil
always arrives as a pen, and a capacitive stylus is electrically a finger and
always arrives as a touch. So there was never a practice-pad bug to find: three
builds of pointer capture, document-level capture-phase handlers and a touch-stream
fallback were all solving a problem that is not where any of them were looking. The
raw event log is what showed it, which is the argument for having built it.

This affects EVERY PEN CONDITION IN THE BATTERY, not the practice pad. The trial
canvas has always required pointerType "pen" for a stylus condition and silently
dropped anything else, so on this device a pen trial records a blank canvas and
looks like an app that ignores the pen.

THREE CHANGES, none of which pretend to fix the hardware.

The device's pointer types are remembered across sessions in localStorage, because
"has this hardware ever once produced a pen event" is a fact about the hardware, not
about the session.

The practice pad FAILS OPEN in exactly that case. Refusing a finger on a stylus pad
is right where the device can tell them apart; where it has never once reported a
pen, refusing leaves a dead pad and an examiner with nothing to do about a cause
that is not theirs. The contact is accepted, the drawing is kept, and the reading
carries implement_verified: false -- so the record never claims a stylus drew it.

And the trial says it out loud instead of dropping the contact in silence. A refused
contact is correct for a palm and wrong for the only implement the participant has.

BUILD v12.27.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>

## The tap zone says which finger, and three tasks stop contradicting each other

WHICH HAND WAS THE ONE THING THE SCREEN WOULD NOT SAY. A centred circle labelled
"tap" says nothing about the hand, and on TAP_FAST the hand alternates between
repeats -- so the thing that changes trial to trial was the thing left unstated.
The zone now sits on the side of the hand that taps and is labelled with it:

  RIGHT
  index finger

Centring was itself a fix, and the comment defending it is right: an earlier build
offset TAP_FAST and left TAP_TONE in the middle, so comparing the two also compared
reach distance. The answer is not to centre both, it is to treat both the same --
every one-zone task is offset now, by the same fraction, so reach still cancels and
both say which hand.

The bimanual zones said "left hand" / "right hand". On a circle touched with one
fingertip that is an instruction to use the hand somehow, and a participant who taps
with a thumb or two fingers has not done the task. They name the effector now.

TAP_BMIN HAD THE EMPHASIS BACKWARDS. "Tap both circles together — AS FAST AS YOU
CAN" leads with the load and buries the measure. In-phase tapping measures how well
the hands stay locked; speed is what makes the coupling break down, and a
participant who reads the shouted half first trades the measure for the load.
Together is now the emphasised line and speed is the qualifier under it.

TAP_BANT PRESCRIBED A STARTING HAND it has no reason to care about. "Tap left,
right, left, right" makes starting on the non-preferred side a demand of its own,
and that demand lands hardest on exactly the population being measured. What
antiphase means is that the two hands are never down together; which goes first is
arbitrary, and is recoverable from the data rather than dictated. "Start with
whichever hand you like."

AND THE TWO-SCREENS-VERSUS-ONE WAS NEVER A DECISION. showInstructionScreen's Ready
button sets LAST_TASK before calling readyTrial, so the hand-change test saw
TRIAL.task===LAST_TASK as already true -- and if the previous trial used a different
hand, the change screen fired directly on top of the instruction screen just read.
TAP_FAST (left following a right) and TAP_BMIN ("both" following a one-handed task)
hit it; TAP_BANT follows TAP_BMIN, is also "both", and did not. Three tasks in a
row, three different numbers of screens, for a reason belonging to none of them.
The change screen exists to catch a change no instruction screen announced; when one
just did, there is nothing to catch.

BUILD v12.28.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>

## Replace a screen that could not be used with a rehearsal that can

THE DOT TAP SCREEN WAS ALREADY GONE, more thoroughly than it looked. Its markup
went when the checks were split out in v12.05; its ~150 lines of JavaScript, its
record schema and its description on the Results page did not. Every entry point
began `if(!$("dotpad")) return`, so the whole thing was unreachable while reading
like a feature the battery has -- which is how a later reader concludes this
collects a visual reaction-time screen.

Nothing of value was lost. Its own record carried measure_grade "screen" and a note
that a 10 s window yields far too few responses to estimate response-time
variability, so what it produced could not be used as a measure. The screening
STORAGE layer stays: records written before v12.05 still sit on devices, and they
still list and export. The Results page now says no current screen writes there,
rather than naming one that does not exist.

IN ITS PLACE, A TAPPING PRACTICE, and the distinction from a screen decides every
choice in it. TAP_TONE's measure is the whole response distribution -- ex-Gaussian
tau is the point of it -- so the opening responses of an unpractised trial, where
the participant is still working out that a beep means a tap, enter the data as slow
responses rather than as the learning they are. The sound check plays the tones but
never asks for a response, so nothing confirmed the pairing before a scored trial
did.

Ten seconds rather than five: at the tap stream's 600 ms floor and 1000 ms mean,
five seconds is about five tones, which is roughly how many it takes to work out
what is being asked. Ten leaves a few after the penny drops, and is still shorter
than a TAP_FAST trial.

Its seed is a fixed constant outside the hash32(task#hand#repeat) space, so the
practice can never hand a participant the sequence they are about to be scored on.
Verified against every TAP_TONE and PEN_TONE hand/repeat combination.

And it reports NO reaction times. The retired screen printed a mean and an SD from
about a dozen responses and needed a paragraph warning nobody to read them as
variability -- a number printed beside a task is read as a result whatever the
caption says. What a rehearsal can honestly report is whether the taps arrived, how
many against how many tones, and with what: no taps at all, fewer than half, or more
than one kind of contact each say something an examiner can act on before the scored
trial.

The target is the trial's target minus the absolute positioning, on the tapping side
of its box and labelled the same way. Rehearsing against something that looks
different rehearses the wrong thing.

BUILD v12.29.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>

## Three panels move to the page each one belongs to, and Checks becomes Practice

The Checks page had become the page for anything that happens before the session,
which is not a category. Splitting it by what each panel is actually about:

THE SOUND CHECK IS A HEARING CHECK, and hearing is a property of the person. Whether
these two tones are clearly audible to THIS participant is a covariate; sitting
between the room check and the practice pads it read as equipment setup. It is now
on the Participant page with the rest of what is recorded about them.

THE ROOM IS NOT THE PARTICIPANT. The room check measures the space and the
microphone, and grouping it with a per-participant panel invited it to be re-run per
participant when once per room is what it is for. It moves to Device, next to the
other thing measured about where the session is happening, and its legend now says
"per session, per room" rather than leaving that to the prose.

WHICH LEAVES A PAGE OF REHEARSALS, so it is called Practice. Both remaining panels
are the same kind of thing: do the unfamiliar part once, before it costs a trial.

AND THE TAPPING PRACTICE GOES FIRST, because the tapping block opens the battery and
the spiral conditions come later -- rehearsing in the other order asks a participant
to practise the second thing they will do before the first. It also needs no pen, so
an examiner whose stylus is not cooperating can still finish the page.

The ordering constraint that put the sound check above the tapping practice still
holds, and now holds structurally rather than by their sharing a box: the tones have
to be at a settled level before the practice plays them, and Participant comes
before Practice.

One thing that had to move with them: goPage rendered the sound check only on the
Checks hook. Left there it would have stayed blank until somebody opened a page it
is no longer on.

BUILD v12.30.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>

## Typography for the screen the participant actually reads

"Tap the circle with your RIGHT index finger — AS FAST AS YOU CAN." needs 733px on
one line and was being given 680, so it broke and left "CAN." alone on a row of its
own. Every emphasised line in the battery was in the same position: measured, they
need 733-780px and all of them were wrapping.

The 680 was not a decision about instructions. #overlay p caps every paragraph at
680px and sets body black, and an id in a selector outranks a bare class -- so the
instruction block's own width and its own colour were both being silently
overridden. The colour one is the more embarrassing of the two: the lines that exist
to stand out were rendering in body text black on the one screen where that matters.

The measure is now 820px, which at 23px is about 62 characters -- inside the 45-75
that reads comfortably, and wide enough that every instruction except the
sustained-spiral line fits one row. text-wrap:balance splits what still has to wrap
into even halves instead of stranding a word; pretty keeps orphans off the bullets.
Both are ignored where unsupported, so nothing depends on them.

STRUCTURE, WHILE IN HERE. The emphasised lines were list items, because everything
was, and were then dragged back out of the indent with a negative margin -- a layout
fighting its own markup. Each is a statement and the plain lines beneath it are its
bullets, so that is what they are now, with the gap above a statement larger than
the one below it so each heads its own group rather than floating between two.

And a plain line before any statement is a lead-in, not a bullet. "Put one index
finger on each circle" opened both bimanual tasks as a lone bullet with nothing
above it to be a bullet of: a list of one, punctuated as though something had been
left out.

Also: the length caption still said "then STOP". The sign has said END since v12.16.

BUILD v12.31.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>

## The rule stays on the task screen, and END stays up

THE ZONE LABELS SAY WHICH FINGER, WHICH IS HALF OF IT. What distinguishes TAP_BMIN
from TAP_BANT is together versus one at a time, and that lived only on an
instruction screen read once and then gone. A short line now sits above the circles
for the whole trial, in the participant's words and sized for them rather than for
the examiner:

  TAP_BMIN  Tap both fingers at the same time, as quickly as possible
  TAP_BANT  Tap one finger at a time, as quickly as possible
  TAP_FAST  As quickly as possible

TAP_FAST gets the speed and nothing else, because a single labelled circle already
says the rest. The tone tasks get no cue at all: the tone IS the cue, and a standing
sentence competing with it would be one more thing to attend to in a task about
attention.

"NEVER BOTH TOGETHER" IS GONE, and you are right that it was doing no work. What
antiphase asks for is alternation; the negative restates it as a prohibition, which
makes an ordinary instruction sound like a warning about a mistake nobody has made
yet. "Tap the circles ALTERNATELY, one finger at a time."

And no em dashes in anything the participant reads: eight of them, replaced with the
comma, full stop or colon each one was standing in for. Two sentences got shorter as
a result, which is the usual sign the dash was carrying a clause that did not need
carrying.

END STAYS UP. It cleared after 1.8 seconds, which put the task screen back in front
of a participant who had just been told to stop, circles still sitting there looking
tappable, while the examiner reads the summary and decides whether to save. The
trial is over and the screen should say so for as long as that is true. Any end
control clears it: SAVE, REPEAT, NEXT, Exit.

The one thing that costs is a drawing task, where the sign now covers the trace and
the summary reports numbers rather than the drawing. A "Show drawing" button puts it
back, and appears only where there is something behind the sign to look at.

BUILD v12.32.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>

## A new participant gets the build's protocol, not the last participant's

TAP_FAST ships 3 repeats with hands "each", which the queue builder turns into six
trials -- dominant and non-dominant alternating, 10 s each. Verified: exactly what
you expected. What you saw was an edit from an earlier session still in force,
because nothing reset it.

newParticipant cleared the participant's own fields and left everything else
standing: the whole trial plan, the geometry, the zone sizes, the tone parameters.
So a repeat count changed to try something once quietly became the protocol for
every participant after, and the plan table gives no sign, because it shows a number
without saying whether it is the one this build ships. A protocol change belongs in
a build -- that is what the version stamp and the tag are for. A change made in the
app is for the participant in front of you.

Reset now covers the plan (includes, repeats, lengths, hands, sustained windows),
the protocol fields (geometry, start zone, zone size and spacing, template style,
tone frequencies and level, idle cut) and the tone stream parameters.

NOT the calibration. ppmm and cardok measure this iPad's glass and are true for
every participant on it; discarding a card calibration because a new person sat down
would be the same class of error in the opposite direction. The confirm says so.

AND IT NAMES WHAT IT IS ABOUT TO DISCARD, before the question, because "a new
session begins" does not tell an examiner that the three extra repeats they added an
hour ago are about to go. Same shape as the Parameters page drift report: what
differs, and what the build ships.

One thing found on the way: clearState() ran BEFORE the confirm, so cancelling
"Start a new participant?" had already destroyed the saved session. The dialog asked
a question whose answer no longer mattered. Verified both paths now -- cancel leaves
the session, the participant id and every edit untouched.

BUILD v12.33.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>

## Turning the sustained window off was a one-way door

Both options were real and only one of them was reachable twice. lengthCell drew
the window control ONLY while the window was on; clear the seconds and at the next
render the cell fell through to a plain "self-paced" label with no control anywhere
to put it back. A switched-off spiral could not be switched on again without
clearing the session.

The control is present in both states now. While the window is on it is also the
whole length, so nothing else is shown -- that part was right, since a
self-paced/fixed dropdown beside a window is two controls for one quantity with one
of them inert. Off, the underlying length control comes back, because then it is the
thing that decides.

The blank now says what it means: "blank: one spiral only, ending at END". Blank is
not an unset field, it is the single-spiral condition, chosen.

The defaults were already what you expected and are unchanged: every spiral
condition ships sustainedS 60, PEN_SPRL and FIN_SPRL included. What made one look
otherwise was the same thing as the TAP_FAST repeats -- an edit carried over from an
earlier session, which v12.33 stopped.

Found while fixing it: the row template already appended the window control when it
was off, as a partial workaround for the same bug. With lengthCell owning both
states that printed it twice on every switched-off spiral.

BUILD v12.34.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>

## Four fixes to the sustained spiral: the arc, the start, the end, and the tail

THE PROGRESS ARC IS GONE. It filled round the outside of the template as angle
accumulated, which is a live performance gauge on a task whose measure is how
somebody draws when they are not being scored at -- a gauge invites racing it, and a
second blue line moving in the corner of the eye competes with the one the
participant is meant to be watching, which is their own. The end zone lighting up
when the sweep completes says the same thing at the moment it becomes actionable,
without narrating the way there. The participant's own ink stays.

THE PEN COULD START ANYWHERE. The window has always opened on the first contact
INSIDE the start zone, but a contact outside it still laid down ink -- so most of a
spiral could be drawn from the wrong place with the trial not yet running and
nothing on screen saying so. The first contact is now gated: outside the zone,
nothing is drawn and the reminder says why. Once the window is open the pen goes
wherever the spiral goes.

THE END ZONE ASSUMED A TIDY SPIRAL. A radius band is the right shape for one, and
wrong for everything else this battery exists to measure: a tremulous or hypometric
spiral can complete its sweep and still sit well inside the band, a large one can
wander outside the template altogether, and a participant who believes they have
finished simply lifts the pen. None of those advanced, so each needed the examiner's
button -- which puts the examiner's reaction time into the pass boundary, the exact
thing auto-advance exists to keep out.

The ANGLE test stays the gate in every case: nothing advances that was not drawn.
What changed is what counts as having arrived, once it has. Three ways, recorded
separately so a pass that ended tidily and one that ended in giving up are
distinguishable in the data instead of both reading "auto":

  band       out near the rim, held briefly, as before
  lift       sweep complete and the pen up for 1.2 s
  overshoot  past 1.08 of the template's own rim, which is beyond the end zone
             rather than approaching it

AND THE TAIL OF EACH SPIRAL STAYED IN THE NEXT ONE'S END ZONE. On an advance the
pen is still down, so it is handed a fresh stroke -- correct, and it has to be, or
every sample is dropped until the participant lifts. But that stroke's first sample
is wherever the pen is at the moment of the advance, out at the rim, so a new empty
template appeared with a blue mark already in its end zone: the tail of the previous
spiral wearing the new spiral's pass number. It is now held back from the canvas
until the pen lifts once. Held back from the CANVAS: every sample is still written,
still tagged with its pass, still exported. A display rule, beside the pass filter
it sits next to.

Verified by driving a real trial: a contact outside the start zone draws nothing and
does not open the window, one inside opens it; all three advance paths fire and
record their own reason; the carried stroke is withheld from the canvas and released
on lift with every sample still in STROKES.

BUILD v12.35.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>

## A refused start is refused, not erased

Answering the question v12.35 raised: as shipped, a participant who began outside
the green circle and kept drawing produced NOTHING -- no ink, no samples, no record
that it happened -- and after 30 s the trial died at the ceiling as "no response".
Refusing to DRAW it and refusing to KNOW about it are different decisions, and only
the first one was wanted.

The refusal stands: no ink, no window, no trial clock. That is the point, and it is
what stops most of a spiral being drawn from the wrong place with the trial not yet
running. Three things change around it.

IT IS RECORDED. Each refused contact is counted, timestamped and positioned, and
goes into the record as off_zone_starts with its detail. One mis-start is a
participant who has not read the circle yet; five is a visuospatial finding, and the
difference belonged in the data rather than in nothing. A trial with
off_zone_starts > 0 and window_start_ms null is now readable as what it is: a trial
that never began, and why.

IT IS SHOWN TO THE PARTICIPANT. A pen that touches the glass and produces nothing
reads as a broken screen, and the examiner's reminder is 14 px of grey text on the
far side of the iPad. The start zone pulses instead -- the answer appearing where
the question was asked. It is the only moving thing on the template, so it cannot be
mistaken for part of the drawing.

AND IT NO LONGER RUNS OUT THE CLOCK. The 30 s ceiling is a backstop against an
UNATTENDED trial; a pen on the glass is proof of attendance. Each refused contact
restarts it rather than counting down to a "no response" on somebody who is visibly
trying. The examiner's message now carries the attempt count with it.

Nothing about a trial that has begun changes: once the window is open the pen goes
wherever the spiral goes, including well outside the start zone, and that was
verified rather than assumed.

BUILD v12.36.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>

## Start zone 8 mm to 14 mm, and stop the geometry hiding a stale value

8 mm gives a participant 4 mm of placement tolerance, which is smaller than the
aiming error of an unsteady hand -- and that is why a trial could fail to begin at
all. 14 mm gives 7 mm and triples the landing area, 50 mm2 to 154.

Chosen over 16 or 20 because the start zone is also the region where angle is NOT
counted, and the two jobs pull against each other. At 20 mm the disc swallows the
whole first turn of a 10 mm-pitch spiral; at 14 it stops at about 70% of it. The
cost is 11.7% of the sweep ungated against 6.7%, which angleNeededRad already
allows for: the advance threshold falls from 5.32 to 5.03 of 6 turns, so a
correctly traced spiral still advances and a fidget in the middle still does not.
The template line is stroked over the disc, so nothing is hidden by the change.

AND THE CHANGE WOULD NOT HAVE SHOWN. startdia is a STATE_FIELD, so a saved session
restores it like any other, and this build's 14 sits behind the session's 8 with the
number on screen looking like the build's own -- the identical trap the tone
parameters had until v12.19, on a different set of values. v12.33 resets them for a
new participant, but an examiner in an existing session had no way to see it.

The Parameters page now reports geometry drift the same way it reports tone drift:
what differs, what the build ships, and a button that puts the build's values back
without touching the calibration or the plan. Found by watching this very change
fail to appear on a reloaded session.

BUILD v12.37.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>

## The practice pad's angle gate was twelve pixels

THE TEMPLATE VANISHED MID-SPIRAL, and the cause is one line of initialisation order.
practiceReset built the angle accumulator BEFORE calling practiceRender, so on the
first call PR.view was still null, the fallback fired, and the gate was set to 12
PIXELS while the start circle it is meant to match is R*0.13 -- about nineteen.
Angle was therefore counted in the ring between them, INSIDE the visible start zone.

That is the exact fault that produced spurious advances in the trial and was fixed
there in v11: near the centre a small movement sweeps a huge angle, so settling the
hand before drawing banks turns that were never traced. reset() keeps the minR it
was built with, so the wrong gate survived every pass and every Start again for the
life of the page. Measured after the fix: four hundred samples of fidgeting inside
the start circle now bank 0.00 turns.

The practice also had neither the minimum-duration guard nor the lift and overshoot
arrivals the trial gained in v12.35. A practice that advances on rules of its own
rehearses something other than the task, so it now uses the trial's rule outright.
Verified: a correct spiral traced in under four seconds does not advance until the
guard expires; one that stops short of the band advances on the lift.

Also: practiceSetup exported the END zone as "zonePx" and never exported the start
zone at all, which is what made the wrong circle reachable and the right one not.
Both are named now.

THE ODDBALL REHEARSAL WAS A TWO-TONE METRONOME. Six strictly alternating tones: a
participant who counts it correctly has learned that every second beep is high,
which is the opposite of the rule. It is now low, high, low, high, low, low, low,
high -- two closely spaced highs to establish what "high" sounds like, then a wait
through three lows for the third. Eight tones, answer three. The button says "Count
the high beep".

AND THE SUSTAINED WINDOW WAS A FIELD TO BE EMPTIED. The mode lived in the blankness
of a number box: type a number for a window, clear it for a single spiral. Clearing
a number input is awkward on a touchscreen, and nothing on screen offered the other
mode by name -- so having set a length there was no visible way back, which is
exactly what it looked like. Two named options now, both always present, and the
seconds appear only when they mean something. Switching back on restores the length
it had rather than a blank. Round-tripped twice in both directions.

BUILD v12.38.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>

## Every pass has a start, not just the first one

THIS IS WHY THE TEMPLATE VANISHED ARBITRARILY, and it is one asymmetry.

Pass 1 could not begin until the pen was inside the start zone. Passes 2 and up
began the instant the previous one ended -- which is with the pen OUT AT THE RIM,
still down, mid-journey back to the centre. Angle accumulated from there. The
journey back across the template, or any loop made on the way, is angle swept at
large radius, and large radius is where the band test is already satisfied. So the
sweep could complete and the pass advance before the participant had drawn any of
it. Measured on the old rule: three loops at 0.95 of the outer radius bank three
turns of a six-turn requirement, on a spiral nobody has started.

From the far side of the glass that is a template disappearing for no reason,
because that is what it is.

The rule is the same for every pass now. The template is up, and NOTHING counts --
no angle, no advance -- until the pen is back in the start zone. It does not need a
fresh pen-down, since the pen is usually still on the glass: passing through the
middle is the start, which is also physically what starting a spiral is. And the
start zone says so, lit while it waits, with the reminder reading "Back to the
middle". Answering the other half of the report: the start zone was not enforced
after the first spiral, and now it is.

Verified: three loops at the rim between spirals bank 0.00 turns and advance
nothing; returning through the middle starts the pass and releases the held ink; a
real second spiral advances by the band as before. Each pass now records
started_ms alongside ended_ms, so a pass and the gap before it are separable.

AND THE INSTRUCTION SCREEN COULD NOT BE SCROLLED ON A TOUCHSCREEN. The guard that
swallows touchmove while the stage is up -- there to stop the page moving under a
pen that is drawing -- was eating every scroll gesture on the instruction screen
too, because that screen lives inside the stage. On a short instruction nobody
noticed. PEN_VODD is 742 px of content in a 470 px box on an iPad in landscape:
reproduced, and it showed the first two thirds with no way to reach the rest.

The overlay is exempt now, in the touchmove guard, the pinch guard and the
double-tap guard, and opts back into pan-y in CSS. It also says when there is more:
a scrolling box with no scrollbar above a pinned button bar looks exactly like a
finished screen, so "more below" appears while anything is under the fold and goes
when the reader reaches it.

BUILD v12.39.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>

## PEN_PENT was never planned as a manual task

"The pen is not registering" was not a pointer bug. handsAllowed() read "drawing"
as "spiral", so a task with spiral 0, tap 0 and zones 0 fell through to ["none"] --
and PEN_PENT is drawn with a pen and has all three of those as zero. It was
therefore planned as a hand-less condition, labelled "No hand (voice only)" in its
own row on the plan table, queued with hand:"none".

Everything downstream followed from that. physHand("none") takes the not-dominant
branch, so the trial ran with drawHand set to the OPPOSITE side from the one
drawing. drawHand was also only ever assigned on spiral conditions, so it was null
anyway, and every layout decision that asks which side the hand is on resolved to
"right" for everybody -- which is the answer to the third report: a left-hander got
the model placed under their own drawing hand, the one arrangement the split exists
to prevent. The pen hand was meanwhile recorded in tapHand, on a task with no taps.

Fixed at the root: a copy task is a manual task. Verified both ways -- right-handed
puts the drawing box on the right and the model on the left, left-handed mirrors
both.

THE DRAWING AREA IS NOW THE WHOLE HALF. It was drawn at the model's own size plus a
margin, which said two wrong things: that the copy must fit a rectangle the size of
the original -- people copying pentagons routinely draw larger, and cramping that is
a measurement artefact rather than a finding -- and that the region outside it was
unavailable, when in fact the whole half accepted the pen. 520x620 px where it was
about 370x195. The drawn rectangle and the accepted rectangle are now the same
rectangle, so "where it says to draw" and "where drawing works" cannot disagree.
That mismatch is the shape of a pen that seems not to register.

The guard was also a single x threshold, which does not exist in the stacked layout
the app picks on a portrait screen -- so the model half took ink there, and a
participant copying onto the original is a lost trial rather than a bad copy. It is
the box in both layouts now.

AND A REFUSED CONTACT SAYS SO. A contact of the wrong implement simply vanished,
which on the only implement in the room reads as a dead screen. It now names both
what the task expects and what arrived, and adds that this device has never reported
a stylus when that is the case.

BUILD v12.40.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>

## A spiral ends when the pen comes off, and that is the whole rule

The end rule was a radius band: sweep complete, pen out past 0.92 of the outer
radius, held 200 ms. Two independent proxies for one quantity -- how far along the
spiral somebody is -- and they disagree in both directions. A spiral drawn LARGER
than its template reaches the band with a turn still to draw, so the template
vanishes mid-stroke. One drawn SMALLER never reaches it at all. Neither failure is
visible from the far side of the glass: the drawing disappears at a moment with
nothing to do with finishing it.

A lift cannot disagree with anything. The participant is already told to draw one
continuous line, so taking the pen off the glass is what they do when they have
finished; it is their decision rather than a geometric inference about it. And THE
TEMPLATE NEVER CHANGES WHILE THE PEN IS ON IT, which ends the category of complaint
rather than narrowing it. The angle gate stays in front, so a lift part-way through
does not end a pass.

No geometric backstop. I tried one -- a runaway threshold well clear of the template
-- and tested it: a spiral drawn 30% large crosses it during the last turn, which is
the same bug at a different number. There is no threshold on radius that can tell
finishing from passing through. The backstop for a pen that never lifts is the
examiner, who is in the room, has Next spiral, and whose press is already recorded
as "examiner" so those passes stay separable.

THE DOTTED CURVE went with it. That ring was the band, drawn so it could be aimed
at; with no band there is nothing to aim at, and a dotted curve materialising across
the drawing moments before it vanished was reasonably read as part of the fault.

AND THE TRACE THAT DID NOT DISAPPEAR was the journey back. v12.39 released the
held-back strokes when the next pass began, which painted the pen's travel from the
old rim to the new centre across the fresh template. Now anything drawn before a
pass starts stays hidden, however the pen got there -- still down from the last
spiral, or lifted and put back down at the rim, which the old rule did not cover at
all. Hidden from the CANVAS: all 725 samples in the test are still recorded, 21 of
them flagged s:0 as belonging to no pass.

Verified: pen down at 1.30x the template does not advance (the old band was at
0.92); it advances on the lift; the return journey shows no ink; the new spiral
draws normally from the middle. The practice pad runs the same rule, and the
instruction now says what ends a spiral: "When you reach the end, lift the pen and a
new spiral appears."

BUILD v12.41.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>

## Say what the numbers mean, and add the four things the app cannot set

THE AUDITORY STREAM COLUMNS ARE DEFINED, one row each, with what moving them costs
and a range. A number an examiner can edit and cannot interpret is a number that
gets edited by guess.

  Floor          the shortest gap the stream will produce, and a property of the
                 RESPONSE rather than the task: raising it widens the usable window,
                 lowering it packs tones together and loses slow responses.
                 150 minimum; tap 500-700, spoken 700-900.
  Mean interval  the average gap, and THE GAP BETWEEN FLOOR AND MEAN IS THE
                 RANDOMNESS -- just above the floor is a metronome. 1.5-2x floor.
  Target prob.   the share of tones that are high. Rarity is what makes a target a
                 target. 0.15-0.30; above 0.35 it stops being an oddball.
  Tone (ms)      how long each beep sounds, not part of the timing. 80-120.
  Tones          read-only, follows from length and mean.
  Usable window  read-only, follows from the floor.

Same for the four settings below the table -- standard and target pitch (what
matters is the SEPARATION, an octave is unmistakable, keep both under 4 kHz), tone
level (not the volume, and the note now says what to do instead of only what it is
not), and Sequence, where Matched is the one a study wants and the reason is stated.

THE YELLOW BOX SAID WHAT IT WAS FOR, NOT WHAT IT WAS. "Not this build's values...
anything not listed already follows the build" is a sentence written from inside the
mechanism. It now says that the numbers in use are not the ones this version was
released with, that either somebody typed them here or the saved session is older
than the app, what will actually be used, and what to do about it.

AND THE 900 ms WARNING TALKED ABOUT LEWY BODY DEMENTIA, which is a claim about a
population in a box about a setting. The consequence is the point and it does not
need the citation: a response slower than the window cannot be attributed to the
tone that prompted it, so it is recorded as a miss rather than as a slow answer --
which removes the slowest responders' slowest responses from the measure instead of
measuring them.

BEFORE THE SESSION, ON THE DEVICE TAB. Four things a web page cannot do for itself,
each of which changes the data and none of which announces itself afterwards:
brightness at maximum with Auto-Brightness off; the iPad flat and secured, since
tapping and drawing push it and a sliding device is movement the pen cannot tell
from the participant's; rotation locked and Auto-Lock never, since a rotation
mid-trial rebuilds the drawing area under the pen and a sleep ends the trial; and
Guided Access, so a stray palm cannot switch apps or pull down a notification.

Not a blocker -- the app cannot verify any of it, and a hard stop would only teach
people to tick it. Each trial records what was confirmed, because a trial run on a
dim, sliding, unlocked iPad is not the same trial.

BUILD v12.42.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>

## The pad only speaks when something is wrong

The counters and the raw event list were always on screen. That was right while the
pencil fault was being chased and wrong as a permanent fixture: this is a page an
examiner uses with a participant in the chair, and a page that always looks like it
is reporting a problem teaches people to ignore it on the day there is one. The
example makes the case on its own -- sixty pointerover lines from a mouse crossing
the pad, before anybody had drawn anything.

Gone from the page: the always-visible counters, the raw event trail, the standalone
Send input diagnostics button and its caption. A working pad now shows nothing at
all, verified including under hover noise.

NOT DELETED, THOUGH, AND DELIBERATELY. The pencil question is still open on that
iPad, and the collection is a few counters that cost nothing and that the implement
verdicts are built from. What changed is when it speaks. The verdicts that matter --
no stylus ever seen on this device, stylus seen but the contact never registering,
strokes cancelled by the browser, contact refused for the wrong implement -- still
appear the moment they are true, and the diagnostics button now appears INSIDE that
report, with the counters, where it is one tap away at the only moment anyone wants
it.

Diagnostics on fault rather than diagnostics on display.

BUILD v12.43.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>


## One spiral and sixty seconds of spirals are two tasks

They were one task with a dropdown, and the dropdown could not do the job asked of
it. A plan holds ONE sustainedS per task, so choosing "one spiral only" for PEN_SPRL
foreclosed the 60 s version for that whole session: the two could never be run on the
same participant, which is precisely the comparison wanted. Two ids can be planned
independently -- included separately, given different repeats, placed at different
points in the order, counterbalanced -- and a mode buried inside one plan row can
express none of that.

They are also two tasks for the participant. One ends when they are told to stop; the
other tells them a new spiral will appear and to keep going, which is a sustained
demand the single spiral does not make. And in the export, a pooled PEN_SPRL
distinguished only by a sustainedS column is an analysis error waiting to happen:
anyone grouping by task id silently mixes a ten-second motor sample with a
sixty-second sustained one.

So PEN_SPRL becomes PEN_SPR60 and PEN_SPR1 joins it; FIN_SPRL becomes FIN_SPR60.
TASK_ALIAS carries both renames, and both old ids resolve to the SUSTAINED task,
because that is the window they shipped -- a session saved before this release
restores as the task it actually ran, not as the new one-spiral condition.

THE WINDOW IS NOW LOCKED ON ALL FOUR, and the lock runs both ways. PEN_SPR60 is named
for its length, so an editable seconds box would let an examiner run 90 s under an id
that says 60 -- the same defect as a filename that disagrees with its build, which
this repo already has one of. PEN_SPR1 is named for being one spiral, so the mode
dropdown would let it become a sustained task under an id that says otherwise. The
way to get the other mode is to include the other task. The dual-task spirals keep
both controls: their window is a real parameter and they have no one-only twin.

## The finger block answers two questions, and neither is a size effect

FIN_SPR1_T5 is the IMPLEMENT BRIDGE. It runs the stylus template exactly -- 12 x 5 --
so the only thing differing from PEN_SPR1 is what is held. A finger is the only input
a phone has, so whether a finger measures what a stylus measures is the question the
whole phone direction rests on. Without this condition, every finger-versus-stylus
difference in the dataset is confounded with template size and cannot be untangled
afterwards by any amount of analysis: it is fixable only by collecting the condition.
One dominant-hand trial is enough to show the two behave alike, or that they do not,
and one trial is the cheapest insurance in the battery.

FIN_SPR1_T3 is PHONE FEASIBILITY, at 10 x 3.

The arithmetic decided the geometry. Template radius is pitch x turns, so 12 x 3 is
36 mm -- 72 mm across -- and a 19.5:9 display is about 65 mm wide at 6.1 inches and
71 mm at 6.7. 11 x 3 is 66 mm, which fits neither with margin while also giving up the
constant pitch, paying for a comparison and not getting the phone. 10 x 3 is 60 mm,
which leaves 2.5 mm a side at 6.1 inches and 5.5 mm at 6.7. That is not generous, and
9 x 3 at 54 mm would be the honest choice if a 6.1-inch portrait screen turns out to
be the target. The number is revisable at no cost: the id names the TURNS, which are
what the gate and the ring test key off, and the pitch is a value in a geometry set.
Changing 10 to 9 after testing on a real phone breaks nothing.

WHAT THIS PAIR CANNOT ANSWER, stated here so that nobody discovers it in the data.
T5 to T3 moves pitch AND turns, so it is not a controlled size contrast. Template path
length is pitch x pi x turns squared: 942 mm against 283 mm, so the small template is
30% of the large one and path_length_mm, duration_s, net_angle_turns and n_pen_samples
differ by a factor of three for reasons that have nothing to do with the participant.
Nor is a corridor-width measure rescuable by normalising, because pitch IS the
precision demand and 10 mm lines are 17% tighter than 12 mm ones.

What it CAN answer is whether the instrument still works at phone scale: completion,
whether the gate fires, achieved sample rate, and the scale-free ratios -- path drawn
over template_path_mm, duration over template path, laps over template_turns. Every
denominator is already in the export, so nothing is lost; it simply has to be done
deliberately. A true size contrast would need 12 x 3 as a third condition, holding
pitch constant, and nobody has asked for one.

FIN_SPR60 is kept unchanged at 12 x 5, each hand, one trial. It is inherited rather
than chosen -- it shipped a window because v11 gave every spiral condition one -- and
its job is the sustained implement bridge plus the one feasibility question the
single-spiral conditions cannot reach: whether a person can sustain finger drawing on
glass for a minute at all, which is not the same physical task as a minute with a
stylus.

## Geometry per implement could not express the finger block

FIN_SPR1_T5 and FIN_SPR1_T3 are both drawn with a finger, run different templates,
and appear in the same session. geomSet mapped implement to one of two sets, so
whichever value the finger set held, BOTH conditions would have run it. The structure
made the block impossible, not merely awkward.

A third named set, finger_small, rather than pitch and turns scattered across
catalogue rows: the settings page stays the one place geometry is edited, and
geometry_set stays meaningful in the record, now naming three real sets instead of
two. A condition names its set; one that names none falls back to its implement, so
every task written before this release is untouched.

THE HAZARD THE OLD RULE PREVENTED IS NOW A THING TO CHECK rather than something the
structure guarantees. A dual task and its single-task baseline must resolve to the
SAME set, or dual-task cost is confounded with a geometry difference. Every dual-task
spiral is implement "pen" and names no set, so they all resolve to the pen set
alongside PEN_SPR1 and PEN_SPR60. Anything given a set explicitly has to be checked
against its baseline by hand.

## A three-turn template was unfinishable, and the practice pad already knew

advturns is one standing number, default 5, and the gate asked for advturns laps and
advturns minus one rings. On a three-turn template that is five laps from something
that crosses its end ray three times, and four rings from a template that HAS three:
unreachable by construction. The spiral would never end on its own and only Next
spiral would get the participant out. It is the same failure the "more laps than the
template has turns" warning was added to catch, arriving by a different door -- not
an examiner typing a large number, but a condition shipping a short template.

The fix was already written. The practice pad has run a three-turn spiral against the
same rule since it was built, and scales it: the same FRACTION of the template, not
the same absolute number of turns. advanceGate applies that to the trial. At the
reference geometry -- five turns, advturns at 5 -- it returns 5 laps, 4 rings and a
gate opening at 4.5, which is exactly what the numbers were before any scaling
existed, so nothing about the shipped stylus conditions moves. At three turns it
returns 3, 2 and 2.7.

The record carries both: advturns_setting is what the examiner typed, against the
reference template, and laps_required is what THIS template was actually held to. On
a 12 x 5 they are the same number, which is why both are written -- a record from a
short template would otherwise read as a failed one.

## Found while doing it: the dead-zone hazard, walked into again

Moving the geometry accessors into the geometry section killed the app outright.
applyCal() runs during start-up and calls updateGeo(), which reads GEOM_SETS -- a
const declared eight hundred lines further down, so still in its temporal dead zone.
A const read there THROWS rather than returning undefined, taking the rest of the
script with it: a blank page and one console line. The file already carries a comment
warning about exactly this, above applyCal, from the last time.

The declarations are now up with byId, where PENCHECK and PC_TURNS already live for
the same reason, and the geometry section keeps its documentation with a pointer to
where the code went. A comment warning about a hazard did not prevent the hazard; the
declarations being physically above the start-up path does.

## Why v13 and not v12.62

The precedent is the repo's own. v11.01 was feat/v11-task-restructure -- renamed
codes, new conditions -- and v12.01 was feat/sustained-spiral. A task id is the join
key every record carries, and a dataset saying PEN_SPRL and one saying
PEN_SPR1/PEN_SPR60 should be distinguishable from the version alone. A minor bump
inside v12 would not say that.

BUILD v13.01.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>
