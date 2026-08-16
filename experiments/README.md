# experiments

Two things live here: a method published before any result, and a place to keep the runs.

- **[DESIGN.md](DESIGN.md)** — how to test whether a room actually beats one model run
  several times over at equal budget. Nothing has been run. If you can find a hole in the
  method, that issue is worth more to us than a result would be.
- **[runs/](runs/)** — buses and transcripts from real rooms, ours and other people's.

---

## Why keep the buses

A finished `Busfile.md` is a dataset that nobody had to design. Every line is typed,
timestamped and attributed, so a run answers questions after the fact:

- how long locks were held, and how many outlived the task that took them;
- how many verdicts were issued and how many were carried out;
- who corrected whom, and whether the correction stood;
- what share of assertions carried evidence a reader could re-run;
- how the intervals between messages moved when a timer died.

Every number in [FIELD-NOTES.md](../FIELD-NOTES.md) was counted by hand, from one file, from
one room. A corpus makes those counts reproducible and comparable, and it is the only route
we can see to answering the measurement question with more than an anecdote.

A donated bus does not need to come from an experiment. A record of ordinary work is more
useful than a clean one from a contrived task.

---

## Layout

```
runs/
  2026-08-11-first-room/
    README.md        what was being worked on, who was in the room, how it went
    Busfile.md       the bus, anonymised
    transcripts/     optional, per callsign
    scoring.md       only for experiment runs: pre-registration, key hash, results
```

Directory name: `YYYY-MM-DD-short-label`. The date is the day the run started.

Each run's `README.md` states, at minimum:

- **Participants** — how many, which tools and models, who coordinated, whether the operator
  was a callsign in the room.
- **Protocol version** the room was running, and any local deviations.
- **Duration and rough budget.**
- **What the room was doing**, in general terms.
- **What went wrong.** This is the field people leave blank and the one that gets read.
- **Anonymisation** — what was replaced, and whether timestamps were shifted.

---

## Donating one

Use the **bus donation** issue template, or open a pull request adding a directory here. The
anonymising checklist is in [CONTRIBUTING.md](../CONTRIBUTING.md#anonymising-a-bus-before-you-post-it);
the short version is that sessions quote command output verbatim, and command output
contains more than you remember.

A heavily redacted bus is still worth sending. Even with every `TEXT` field replaced, it
carries the types, the timing, the locks and the shape of the conversation.

---

## What is not here yet

Our own three-day run — the one every number in the field notes came from. It is a real
project's bus, with real paths, real customers and real credentials quoted in command
output, and it has not been anonymised. That is the operator's decision to make and it is
not made. Until it is, this directory is empty, which is an accurate representation of the
evidence base.
