# Does a room beat one model run several times?

**Status: method only. Nothing has been run. This document is published before any result
so that it can be attacked as a method, which is much easier than withdrawing a number.**

If you can see a hole in it, that is the most valuable issue this repository can receive
right now. Open one.

---

## The claim under test

The only claim this project makes is narrow: *a room finds defects that no single
participant finds alone.* We have one room's worth of anecdote behind it and no measurement.
[FIELD-NOTES.md](../FIELD-NOTES.md) contains the anecdotes, including a cross-vendor
participant that found in ten minutes an error four same-vendor sessions had propagated and
reported upward as established fact. That is a sample of one, and it is also exactly the
result a lucky draw would produce.

Two things could be true instead, and the design has to be able to distinguish them:

1. **The room is a budget effect.** Four sessions cost about four times one session. A
   single model run four times, with its findings pooled, might do just as well.
2. **The room is a rules effect.** The gain, if any, might come from the evidence rules in
   [REGIMEN.md](../REGIMEN.md) — which need no room at all — rather than from having several
   participants.

Hence a 2×2, not a head-to-head.

---

## Design

|  | Without the regimen | With the regimen |
|---|---|---|
| **Single session, run K times, findings pooled** | cell A | cell B |
| **Room of N participants + coordinator** | cell C | cell D |

The comparison people expect is A vs D. The two that decide what the protocol is *for* are
**B vs D** (does having a room add anything once the epistemic rules are in place?) and
**A vs B** (are the rules doing the work by themselves?).

Note what the "without the regimen" arm still has: rooms in cells C need locks, the message
format and the bus, or they cannot share a tree at all. The factor being varied is the
**evidence rules** — claims versus opinions, count constructs, method and date, measure
third — not the coordination machinery. Removing the machinery from a room does not produce
a room without rules; it produces a broken room, and that experiment tells you nothing you
did not already know.

### Budget equalisation

This is the part that decides whether the result means anything.

**Equalise on total output tokens across the four cells**, not on number of sessions, wall
clock, or turns. A room of four participants plus a coordinator that spends 5× the tokens of
one session and finds 20% more defects has lost, and any design that lets it declare victory
is measuring spending.

- Fix a total output-token budget per cell before starting. Every cell gets the same.
- Within a cell, the budget is divided however that arm naturally works: cell D splits it
  between participants and coordinator; cell A spends it on K independent runs.
- **K is derived, not chosen:** K = whatever number of single-session runs fits the same
  budget. If a room of five spends its budget in one pass, K is about five.
- Record actual spend per cell. If a cell overruns, say so and by how much; do not
  normalise after the fact.

Cost per confirmed defect is a reported metric, not an afterthought.

### Pooling in the single-session arms

The single-session baseline must be as strong as we can honestly make it, because a weak
baseline is the standard way this kind of comparison lies.

- K runs, each starting from an identical prompt and a clean context. No memory between them.
- **Findings are pooled by union**, deduplicated mechanically on (file, line, defect id).
- Pooling is the analogue of a room's shared bus, and it favours the baseline — a room has
  to beat the *union* of K independent attempts, not the average of them. That is the
  comparison worth winning.

---

## Material

**Seed real defects, not invented ones.** Take a repository with a real history, find
commits that fixed genuine bugs, and revert them. The defect is then something that actually
happened to somebody, at a plausible density, in code that was written without knowing it
would be tested.

Requirements:

- **A known set of seeded defects, typed.** Wrong result, crash, data loss, resource leak,
  race, stale-configuration, silent fallback. Type is recorded because per-type recall is more
  informative than a single number — our expectation is that rooms help most on defects that
  need cross-checking between files, and least on local logic errors.
- **A sealed answer key**: file, line, type, and the original fix. Sealed means the people
  running the arms cannot read it, and neither can any judge who scores the unseeded
  findings.
- **A codebase small enough to be read end to end** by one session inside the budget, or the
  experiment measures reading speed rather than defect-finding. State its size in files and
  lines.
- **Not a codebase any participating model is likely to have memorised.** Say what you chose
  and why; if it is a well-known open-source project, that is a limitation to report, not a
  disqualification.

---

## Metrics

Primary, from the sealed key:

- **Recall** — seeded defects found, overall and by type.
- **Precision** — of everything reported, the share that is a seeded defect or a
  genuine defect confirmed by a judge. A room that reports twice as much will find more of
  everything, including nothing.
- **Cost per confirmed defect** — output tokens divided by confirmed findings.

Secondary, and the reason the buses are kept:

- **Findings outside the key.** Real code contains real bugs nobody seeded. These are judged
  blind — the judge sees the finding, not which arm produced it — and counted separately.
  They cannot be scored for recall, since nobody knows the denominator.
- **Corrections between participants** (cells C and D only): how often one participant
  overturned another, and how often the correction was right. This is the mechanism the
  whole protocol claims to provide; if the room wins without it, the explanation is
  something else.
- **False claims that survived** — assertions of the form "I verified this" that the key or
  the repository contradicts. Our own room produced eight in two days. A room that finds
  more defects *and* propagates more false certainty has not obviously won.

---

## Protocol for a run

1. **Pre-register.** Before running anything: the thresholds below, the exact prompts, K, N,
   the budget, and the answer key hash. Post them in the issue that will carry the result.
2. Prompts are **fixed and published**. Identical across cells except for the regimen text
   and the room-versus-single framing. Any wording difference beyond that is a confound.
3. The operator does not intervene. This is a deviation from how the protocol is meant to be
   used — an operator's six words were the most valuable message in our own run — and it is
   accepted deliberately, because an intervening human is an uncontrolled variable. **Report
   it as a limitation:** the experiment measures rooms without an operator, which is not the
   configuration the protocol recommends.
4. **Keep everything.** The complete `Busfile.md` for room cells, full transcripts for
   single-session cells, token counts per session. These go to
   [runs/](runs/) — the record is the point, and a result without its bus cannot be re-scored
   by anyone who disagrees with the scoring.
5. Score mechanically against the key first. Judge the unseeded findings second, blind.

---

## Thresholds, declared in advance

To be filled in and frozen before the first run, but the shape is:

- **The room wins** if cell D's recall exceeds cell B's by more than the observed
  run-to-run spread, at equal budget and without a fall in precision.
- **The rules win** if B beats A by a similar margin — in which case the honest headline is
  that most of the value needs no room, and the protocol should say so in its README.
- **Both are noise** if the spread between repetitions of the *same* cell is comparable to
  the differences between cells. This is the most likely outcome of a first attempt and it
  is a publishable result.

**One run per cell establishes nothing.** LLM runs vary; four numbers with no variance
estimate are four anecdotes in a table, which is worse than one honest anecdote because a
table looks like data. Repeat each cell at least three times and report the spread. If your
budget does not stretch to that, report what you did and label it a pilot.

---

## Known weaknesses of this design

Listed here because a limitation you name is a limitation, and one you do not is a finding
somebody else gets to make.

- **Reverted fixes are not a random sample of bugs.** They are bugs somebody found, which is
  a biased draw towards the findable.
- **Single vendor confounds the room arm.** Four sessions of one model may share their blind
  spots, which is precisely what the protocol claims a mixed room avoids. Ideally cell D is
  run twice: same-vendor and mixed. That doubles the cost and is the first thing to cut,
  under protest.
- **Defect-finding is not the task most rooms are for.** It is the task we can score. Day
  three of our own notes is entirely about shipping, where the failures were different in
  kind and the useful metric is unclear.
- **The judge is a model, or a person with limited time.** Either can be wrong in the same
  direction as the arm being judged. Blind judging helps; it does not fix a judge who shares
  a blind spot with a participant.
- **We are the wrong people to run it.** Everyone here believes the room works. A replication
  by somebody who expects it not to is worth more than our own first attempt, which is why
  the method is public before the result.

---

## If you run it

Open an issue with the pre-registration first, before the result exists. Then post the
outcome — including, and especially, a null one. A protocol that does not beat one model run
four times is still useful if it says so; the thing that would make it useless is claiming
otherwise on one room's anecdote, which is where it stands today.
