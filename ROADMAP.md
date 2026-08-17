# Roadmap: 0.3

What the next minor version will decide and add. Not promises and not dated: 0.3 ships
when it is written, and every item below either carries the incident it came from — all of
them from [the second room](FIELD-NOTES.md#the-second-room-v021) unless said otherwise — or
is marked untested. An item that loses its argument gets dropped, which is the normal fate
of roadmap lines and should be said out loud.

Per the [versioning scheme](PROTOCOL.md#changes), 0.3 is **minor**: a participant on 0.2.1
still interoperates, but reads the room incompletely.

---

## Two undetermined questions, now with field answers

**Timers.** The `no timer` declaration stays supported — requiring a scheduler would
exclude whole tools, against the point of the design — but the declaration alone was not
enough: a schedulerless reviewer went dark twice, its silence was read backwards, and
review stopped for a morning
([incident](FIELD-NOTES.md#a-critic-that-dies-silently)). Under consideration: a change of
timer state is announced on the bus immediately; proof of a working wakeup is a line on
the bus written by the resumed session, never an exit code; a participant with no timer is
not given a gating role without a named relay.

**Snapshots.** The per-committer duty blocked the one participant that could not write
outside the tree ([incident](FIELD-NOTES.md#every-rule-the-coordinator-wrote-was-too-wide)).
The 0.3 form: the precondition for `@git` is that a current snapshot *exists*, whoever took
it; the coordinator still takes one per wake. This settles four rows of the
[undetermined list](PROTOCOL.md#undetermined-in-v02) at once. The remaining rows stay
undetermined — no field data, no decision.

## Core rules under consideration

**An assignment names what it displaces** — the lane it parks and who owns that blocker,
or the words "displaces nothing"; a malformed assignment may be refused. *Two lanes
orphaned forty minutes apart, one unowned for three hours
([incident](FIELD-NOTES.md#two-orphaned-lanes-forty-minutes-apart)).*

**A lane names its origin** — a queue row, or "finding, not on the queue" — and the
coordinator's wake includes one line of queue reconciliation: open, closed this interval,
active lanes from the queue versus from findings. This is the only candidate mechanism
against the second room's dominant failure: five and a half hours of the room working its
own findings while half the queue sat finished
([incident](FIELD-NOTES.md#where-every-lane-came-from)).*

**Relay provenance, and reporting delivery rather than dispatch.** A relayed operator
instruction carries a marker of what it relays; the coordinator testifies only to what
passed through it; "delivered" is reported after the addressee answers, not after the line
is posted. *Caught twice by the operator
([incident](FIELD-NOTES.md#reporting-at-dispatch-time)).*

**Reading the bus by cursor.** Past some size, tailing answers "how busy is the bus," not
"what is addressed to me": search for your own callsign from a line number recorded last
wake, and control the first empty result. *330 KB, a missed answer and a missed approval
within ten minutes ([incident](FIELD-NOTES.md#the-bus-outgrew-tail-reading)).*

**A register of what is blocked on the operator**, posted consolidated rather than as
scattered escalations. *Fifty-four verdict lines, the register never posted once
([incident](FIELD-NOTES.md#reporting-at-dispatch-time)).*

**Bus rollover — proposed, untested.** The live file is never trimmed; append-only stands.
The sanctioned route for a bus that has outgrown reading is to close the file whole:
snapshot, rename to a dated archive, start a fresh `Busfile.md` whose header names its
predecessor. The active state does not survive as the coordinator's summary — every owner
re-asserts its own locks and lanes in fresh lines, and a lock nobody re-asserts is
released. Rollover resets reading cursors, and the rollover line is what a reader checks
first. The incident exists for the *problem* (the bus outgrew tail-reading); the mechanism
has been run by nobody, and 0.3 will say so where it defines it.

## Regimen additions

- **A probe returning zero is not evidence** until it has fired on input known to trigger
  it, calibration posted beside the result. *Two lying instruments in one day
  ([incident](FIELD-NOTES.md#the-instruments-lied-four-times)).*
- **A rule is no wider than its incident**, and its instrument is tested on everyone it
  binds before it is issued. *Three rules narrowed in hours, each fix from an objection
  ([incident](FIELD-NOTES.md#every-rule-the-coordinator-wrote-was-too-wide)).*
- **A pre-task gate**: before starting, establish that the subject still exists, that the
  answer is not already written down, and that the work was not already done — and post the
  check even when the task survives it. *Half the queue was already finished; a descriptor
  file disproved an afternoon of correct measurements
  ([incident](FIELD-NOTES.md#the-written-answer-beat-the-measurement-twice)).*
- **A supersede note goes at the top** of the record it corrects, not the bottom. *A stale
  mirror, misread from an assertive page with the correction at its foot
  ([incident](FIELD-NOTES.md#a-reachable-server-is-not-an-in-scope-server)).*

## Profile additions (coding in a shared git tree)

- **Host frameworks and hooks.** A repository can carry its own agent framework: hooks that
  gate writes, stale runtime state, a second task registry with no bridge to the room's.
  Diagnose a hook by running it against candidate paths — including one that must be
  denied, since an instrument that cannot fail is not an instrument — and fix by
  relocation, never by disabling someone else's enforcement
  ([incident](FIELD-NOTES.md#a-framework-that-fought-the-room)).
- **Branchless review.** When finished-but-unreviewed work sits in the shared tree, locks
  persist through the review rather than ending when typing stops, and the diff is copied
  outside the tree until the verdict
  ([incident](FIELD-NOTES.md#two-workflow-redesigns-in-one-afternoon)).

## What 0.3 will not do

- Settle the remaining undetermined questions without field data.
- Add anything against malice — [THREATS.md](THREATS.md) stays an honest refusal.
- Introduce required tooling, or change the line format or the tiebreak. Either of the
  last two would be a major version, and nothing observed argues for one.

---

If you have run a room and an item here contradicts what you saw, that is a
[rule challenge](CONTRIBUTING.md) and it outranks this file.
