# Contributing

**An issue describing what happened is usually worth more here than a pull request.**

This is a protocol document, not a program. It is short on purpose, and almost everything it
still gets wrong is invisible from inside the one room that produced it: one codebase, three
days, several sessions of one model family plus ten minutes of another. Every rule in it is
generalised from a sample of one incident. Text is cheap to write and evidence is not.

So the contribution that helps most is a report of contact with reality.

---

## What is most wanted

### You ran it and something broke

The most useful thing in this repository is [FIELD-NOTES.md](FIELD-NOTES.md), and the most
useful parts of that are the failures. Say what you expected, what happened, and what the
room did about it — including nothing, if the answer is nothing.

A failure that made you abandon the protocol is not an embarrassment to us; it is the
finding. → **Failure report** template.

### You ran it with a vendor mix we have not tried

Our cross-vendor experience is about ten minutes long. In that time the new participant
broke the shared file, fixed both its own mistakes without being told twice, and found an
error four same-vendor sessions had propagated and reported upward as fact. One data point
in each direction.

Anything beyond that is new evidence, especially the mechanical friction: how your tool
appends UTF-8, whether it can wake itself, and what its silence therefore means.
→ **Vendor mix report** template, and consider a row in [VENDORS.md](VENDORS.md).

### A rule is wrong, or right for the wrong reason

Every rule came from a specific incident, and an incident is a sample of one. If you can
show a case where following a rule makes things worse than not following it, that is the
most valuable issue you can open. Rules have been removed from this protocol before.

Bring the case, not the objection in the abstract. → **Rule challenge** template.

### Your bus, anonymised

A finished `Busfile.md` is a dataset: every line is typed, timestamped and attributed, so
lock hold times, unexecuted verdicts, who corrected whom, and the share of claims carrying
evidence are all countable from the file. Every number in our field notes was counted by
hand from one such file.

A corpus of real runs, from other people on other vendor mixes, is the only route we can see
to answering the measurement question honestly. → **Bus donation** template.

### The measurement problem

Nobody has established that a room beats one model run several times over with the same
budget. We think it finds defects a single participant does not, and that is the only claim
this repository makes.

[experiments/DESIGN.md](experiments/DESIGN.md) is a method published before any result, so
that it can be criticised as a method. **If you can show a hole in it, do that before anyone
runs it** — a flawed experiment that has already produced a number is much harder to
withdraw.

---

## Anonymising a bus before you post it

A bus from real work contains real work. Before attaching one:

1. **Paths and filenames** — replace project-specific paths consistently (`core/parser.py`
   is fine; `clients/acme/2026-tender/pricing.xlsx` is not). Keep the *shape*: the same file
   should be the same placeholder throughout, or lock behaviour becomes unreadable.
2. **People and organisations** — the operator's name, colleagues, customers, vendors you do
   not want named. Callsigns are already pseudonyms; leave them.
3. **Secrets** — credentials, tokens, hostnames, internal URLs, ticket ids. Search for them
   rather than trusting your memory of the run; sessions quote command output verbatim and
   command output contains more than you remember.
4. **Free text** — `NOTE` and `ANSWER` lines carry the most incidental disclosure, because
   that is where people explain things.
5. **Then read the whole file once, top to bottom.** Not a sample. This is the step people
   skip, and it is the one that catches what the searches did not.

Keep timestamps as they are — they are the data. If the run's timing is itself sensitive,
shift every line by a constant and say that you did.

We would rather have a heavily redacted bus than none. A file with every `TEXT` field
replaced by `[redacted]` is still evidence about locks, types, timing and who answered whom.

---

## Pull requests

Welcome, with two preferences:

- **A rule change arrives with its incident.** New rules in this protocol are supposed to be
  generalisations of something that actually happened, and one that is not says so in the
  text: mark it *proposed, untested*. That is an acceptable state for a rule to be in — it
  is unmarked speculation that is not.
- **Keep the copies in step.** The same rules appear in [PROTOCOL.md](PROTOCOL.md),
  [Busfile.template.md](Busfile.template.md) and both files under [prompts/](prompts/),
  because a prompt has to work when pasted into a session that cannot read this repository.
  PROTOCOL.md is the source of truth when they disagree. Three copies of one caveat rot at
  different rates and then contradict each other, which is itself one of the rules.

Prose style, since this is mostly prose: state the rule, then the incident it came from. If
there is no incident, say so.
