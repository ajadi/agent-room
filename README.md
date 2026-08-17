# agent-room

**A text-file protocol for several independent AI sessions working on the same thing at the
same time.**

[![protocol v0.2.1](https://img.shields.io/badge/protocol-v0.2.1-blue)](PROTOCOL.md)
[![evidence: two rooms](https://img.shields.io/badge/evidence-two%20rooms-orange)](FIELD-NOTES.md)
[![license MIT](https://img.shields.io/badge/license-MIT-green)](LICENSE)

No server. No database. No framework. Nothing to install. The participants are not spawned
by anything — they are separate programs that already exist, possibly from different
vendors, that have agreed to append to the same file.

The core assumes only that several sessions share one mutable thing and one record. **All
of our evidence is from code**, in a shared git working tree, and those rules are now a
[profile](profiles/coding-shared-git.md) rather than the protocol — so a room over
documents, a database or a hardware bench can use the core and delete the rest. Nobody has
run one yet.

> **Status.** Protocol **v0.2.1**. The evidence is two rooms, both in
> [FIELD-NOTES.md](FIELD-NOTES.md): three days under v0.1 — the run the protocol came from —
> and five and a half hours under v0.2.1, mixed vendors, a real deadline — first contact
> for the v0.2 rules. Some held (tokens minted on first
> incident, zero lock collisions), some were found too wide in the field (the snapshot
> rule), and two of the [undetermined questions](PROTOCOL.md#undetermined-in-v02) now have
> field answers. The decisions they imply are queued for **[0.3](ROADMAP.md)**, not patched
> in quietly. Nobody has yet posted a `STANDDOWN`, lost a tiebreak on seconds, or restored
> the bus from a snapshot.

---

## Contents

- [The problem](#the-problem)
- [How it works](#how-it-works)
- [Quickstart](#quickstart)
- [Three rules that mattered more than the locks](#three-rules-that-mattered-more-than-the-locks)
- [Timers: the part without which none of it runs](#timers-the-part-without-which-none-of-it-runs)
- [Why not subagents](#why-not-subagents)
- [Prior art](#prior-art)
- [Limits](#limits)
- [What we are willing to claim](#what-we-are-willing-to-claim)
- [Documentation](#documentation)
- [Contributing](#contributing)

---

## The problem

An AI coding assistant is a process: it reads files, edits them, runs tests, commits. One of
them is predictable.

Several of them in one working tree is not. They cannot see each other. Each assumes it owns
the files. Two edit the same file and one edit is lost. One stages a change, another commits
without a pathspec and carries it away. Two start the test suite at once and the machine
swaps.

The usual answer is to give each one its own copy and merge later. That is real merge work,
and some tasks need shared state anyway.

`agent-room` is the other answer: let them share the tree, and give them somewhere to talk.

---

## How it works

A file called `Busfile.md` sits in the repository root. Every participant appends lines to
it. Nobody edits or deletes anyone else's lines.

```
| HH:MM:SS | FROM | TYPE  | TO   | TARGETS        | TEXT                          |
| 14:02:11 | ALFA | LOCK  | *    | core/parser.py | rewriting the token scanner   |
| 14:03:40 | BRAVO| ASK   | ALFA | core/parser.py | how long, I need it after you |
| 14:05:02 | ALFA | UNLOCK| *    | core/parser.py | done, 41 tests green          |
```

Participants use **callsigns**, not model names. Before changing anything shared you
announce a `LOCK`; when you are finished you `UNLOCK`. A resource is a path, or an
`@token` for shared state that is not a file — which ones exist is what a
[profile](profiles/coding-shared-git.md) defines. If two sessions claim the same file at once, the
earlier timestamp wins; on a tie, the alphabetically lower callsign wins. No negotiation.

One participant acts as **coordinator**: arbitrates conflicts, assigns work, and checks
claims. The coordinator is not a filter — every participant writes into the same file
directly, and the coordinator is corrected in it like everyone else. In our own run it was
corrected nine times in one day, and every correction was justified.

The human can hold a callsign too. It is optional and it is cheap, and it turns *"I was
authorised to do this"* from a paraphrase into a line anyone can quote.

That is the whole system. The full message format, lock rules and arbitration are in
[PROTOCOL.md](PROTOCOL.md).

---

## Quickstart

**1. Drop the bus into your repository.**

```sh
curl -o Busfile.md https://raw.githubusercontent.com/ajadi/agent-room/main/Busfile.template.md
```

Or copy [`Busfile.template.md`](Busfile.template.md) by hand. Decide deliberately whether to
commit it: an untracked file leaks nothing and has no backup, and a tracked one is the
reverse. Either way, snapshot it — see step 4.

**2. Start each session with a prompt and a callsign.**

Paste [`prompts/participant.md`](prompts/participant.md) into each worker session, replacing
`<CALLSIGN>`, `<SHARED STATE>` and the lane at the end. Give one session
[`prompts/coordinator.md`](prompts/coordinator.md) instead. Both prompts end with a git
profile block marked for deletion — **delete it if your room is not working on code.**

**3. Have every participant arm a self-wakeup and verify it.**

Five minutes for participants, hourly for the coordinator. **Verified** means the session
listed its scheduled jobs and quoted the job id — not that it believes it armed one. In our
run a session held that belief for an hour and was wrong.

If a tool cannot schedule anything, it still works, but it must say `no timer` on entry so
the room knows its silence means *waiting* rather than *dead*. See [VENDORS.md](VENDORS.md).

**4. Append safely, and snapshot.**

The bus is the one artifact everyone depends on, and it was destroyed twice in our first
three days. Pin the encoding, take the clock in the same command as the write, and keep
snapshots outside the tree:

```sh
printf '| %s | ALFA | HELLO | * | - | session … pid … timer job … silence means dead-or-closed |\n' \
  "$(date +%H:%M:%S)" >> Busfile.md

mkdir -p ../bus-backups && cp Busfile.md "../bus-backups/Busfile.$(date +%Y%m%d-%H%M%S).md"
```

PowerShell, Python and Node equivalents are in [VENDORS.md](VENDORS.md#appending-safely).
Do not trust a shell's default encoding: one participant writing UTF-16 puts null bytes in
the file, every reader declares it binary, and the whole room goes blind at once.

---

## Three rules that mattered more than the locks

**Claims and opinions are different, and you say which you are making.** A claim carries
evidence anyone can re-run: a file and line, a command and its output, a commit hash. An
opinion cannot be proved and is labelled as one. A claim without evidence is struck. The
point is not catching liars — it is stopping a confident tone from counting as proof.

**Callsigns, not model names.** A vote should not weigh more because of whose model produced
it, or the argument decays into status. Vendor and version are recorded once on entry, for
diagnosis if behaviour drifts, not for authority.

**Closed rounds when you want genuinely independent opinions.** A shared file is harmful
here: the second critic reads the first before writing, and three opinions collapse into one
plus two agreements. For those rounds each participant writes to a separate file and they
are opened together. Independence is enforced by the mechanics, not requested in a prompt.

These are three of about fifteen. The rest — count constructs rather than search hits, a
count carries its method and its date, measure third rather than first, date a hazard and
re-test it before briefing anyone — are in [REGIMEN.md](REGIMEN.md), which is deliberately
readable on its own: **none of those rules needs a room.** A single session breaks all of
them in the same way, with nobody to notice.

---

## Timers: the part without which none of it runs

An assistant that has finished its turn stops reading anything. It is not idling and
watching the file — it is waiting for input and will not see a word you write.

So each participant arms a repeating self-wakeup — five minutes for participants, hourly for
the coordinator: wake, re-read the tail of `Busfile.md`, act on anything addressed to you,
take one step, sleep.

The side effect matters more than the mechanism. **With a timer, silence means the session
is dead.** Without one, silence means the opposite — alive and waiting to be poked. Those are
contradictory readings of the same quiet file, they are routinely live in the same room, and
confusing them is expensive. A session that has stood down has a third meaning again. This
is why the first line a participant writes states what its own silence means.

---

## Why not subagents

Tools that let a main agent hire helpers — assign roles, sometimes a different model per
role — look like a team. Structurally they are a manager with interns.

- The helpers exist because the manager created them. They know the task as the manager
  restated it.
- They report only to the manager, and the manager summarises them onward.
- **If the manager is wrong, the error reaches every helper**, because the manager wrote
  their briefs. A helper's objection reaches the outside world only in the manager's
  paraphrase, through the same misunderstanding.
- The division of labour is the manager's guess about the shape of the problem. Every role
  then works diligently inside that guess.

In a room there is no owner:

- Participants pre-exist the protocol. Nobody assigns them roles from outside.
- **Nothing stands between a participant and the record.** A disagreement lands in the file
  verbatim and everyone reads the same text.
- Participants may be from different vendors. Models from one family fail in similar ways —
  similar training data, similar habits. Four of the same can miss the same thing together.
  A different vendor's blind spots are in different places.
- Any tool that can read and write a text file can join. There is nothing to integrate.

---

## Prior art

None of the machinery is new, and it is worth saying so before someone else does.

A shared structure that independent specialists read from and write to, never calling each
other directly, is a **blackboard architecture** — Hearsay-II was doing it for speech
understanding in the 1970s. Coordinating through a shared store rather than through messages
between named parties is **Linda's tuple space**, from the 1980s. An append-only record
where the log is the source of truth is older than either. Callsigns, timestamps and "say
what your silence means" are radio procedure.

What is actually different here is small and worth stating exactly:

- **The participants pre-exist the protocol.** A blackboard system has a control component
  that decides which knowledge source runs next; here nothing schedules anyone, because the
  participants are separate programs that were already running.
- **The diversity being exploited is diversity of failure.** Classic knowledge sources
  differed in what they knew. These differ in how they are wrong, which is why vendor mix
  matters and four sessions of one model are closer to one participant than to four.
- **Liveness is a first-class concern.** A knowledge source is invoked; a session stops
  reading when its turn ends, so silence has to be given a declared meaning and a timer has
  to be armed to make it true.
- **Half the protocol is about evidence, not coordination.** Blackboard systems did not need
  [REGIMEN.md](REGIMEN.md), because their knowledge sources did not produce confident,
  fluent, unfounded prose.
- **The medium is a text file.** Not a bus you implement — a file, readable by a human and by
  anything that can run one shell command.

---

## Limits

- **It cannot wake anyone.** The protocol has no way to reach a session that has stopped
  reading. Participants wake themselves; a session killed by a usage limit stays dead until a
  human restarts it.
- **It cannot compel.** A coordinator can observe a violation and demand a correction. It
  cannot stop another session. The whole thing rests on good faith, and a participant
  determined to route around it will.
- **It does not protect against malice**, only against inattention. `FROM` is not
  authenticated, append-only is a convention rather than a property, and the bus is an
  instruction channel by construction — anything that gets text into it directs the room. The
  realistic version needs no attacker, only a participant that read something
  instruction-shaped in the repository it was working on. [THREATS.md](THREATS.md) sets out
  what follows and why the fix would cost more than it buys.
- **Vendor differences bite early.** The first cross-vendor incident in our own run was not a
  disagreement about code — one participant wrote the log in a different text encoding and
  every other tool declared the shared file binary. The whole room went blind at once.
- **It cannot tell right work from wrong work.** Every mechanism here catches false claims.
  None notices a room diligently working its own findings instead of the queue it was
  given: the second room did exactly that for five and a half hours, closing four
  self-invented lanes and zero queue tasks, and what stopped it was the operator asking a
  blunt question ([FIELD-NOTES.md](FIELD-NOTES.md#where-every-lane-came-from)).
- **Most of it is lightly tested.** See the status note at the top. Two rooms is a sample
  of two.

---

## What we are willing to claim

Narrowly: **a room finds defects that no single participant finds alone.**

Both rooms support it the same way: a cross-vendor participant caught what same-vendor
agreement had settled on — a docstring misread as live code in the first room, a
set-membership swap two same-vendor sessions agreed on in the second. The second room also
supplied the sharpest counterweight: its costliest failure was not a false claim but true
claims about the wrong work, and no rule here catches that
([FIELD-NOTES.md](FIELD-NOTES.md#what-it-cost-and-what-it-was-worth)).

We do not claim it produces better solutions. To measure that you would need to know the
best solution in advance.

We have not established that a room beats one model run several times over at the same cost,
and we are not going to pretend otherwise on one room's anecdote.
[experiments/DESIGN.md](experiments/DESIGN.md) is a method for testing it, published before
any result so that it can be attacked as a method. **If you can find a hole in it, that is
worth more to us right now than a number would be.**

[FIELD-NOTES.md](FIELD-NOTES.md) is what three days of real use actually produced, including
everything that went wrong — which is most of what is worth reading.

---

## Documentation

| File | What it is |
|---|---|
| [`PROTOCOL.md`](PROTOCOL.md) | **The core.** Domain-neutral: message format and grammar, locks, keeping the bus, the operator, timers, and every ban paired with its sanctioned alternative. Versioned, with a changelog |
| [`profiles/coding-shared-git.md`](profiles/coding-shared-git.md) | The one profile with evidence behind it: sessions sharing a working tree and a git index. Also how to write another |
| [`conformance/`](conformance/) | Worked buses, valid and deliberately broken, for anyone writing a validator — or testing their own check on input known to be bad |
| [`REGIMEN.md`](REGIMEN.md) | **The rules about knowledge.** Claims versus opinions, what counts as evidence, measuring third rather than first, hazards that expire. Readable on its own — none of it needs a room |
| [`PATTERNS.md`](PATTERNS.md) | Twenty named failure modes, each linked to the incident it came from |
| [`FIELD-NOTES.md`](FIELD-NOTES.md) | Both rooms, split by protocol version: three days under v0.1, and five and a half hours under v0.2.1 — mixed vendors, a real deadline, first contact for the v0.2 rules. All the evidence there is |
| [`THREATS.md`](THREATS.md) | What the design does not protect against, and why we are not fixing it here |
| [`VENDORS.md`](VENDORS.md) | Per-tool compatibility: appending UTF-8, self-wakeup, what breaks first. Mostly empty, honestly marked |
| [`Busfile.template.md`](Busfile.template.md) | Drop-in starter log with the protocol sections filled in |
| [`prompts/participant.md`](prompts/participant.md) | Startup prompt for a worker session |
| [`prompts/coordinator.md`](prompts/coordinator.md) | Startup prompt for the coordinator |
| [`ROADMAP.md`](ROADMAP.md) | What 0.3 will decide and add — every item with its incident, or marked untested |
| [`CHANGELOG.md`](CHANGELOG.md) | The full version history with context. The normative list is [PROTOCOL.md § Changes](PROTOCOL.md#changes) |
| [`experiments/`](experiments/) | The measurement method, published before any result, and a place to keep donated buses |
| [`CONTRIBUTING.md`](CONTRIBUTING.md) | What is most wanted, and how to anonymise a bus before sending it |

**Where to start:** reading, [FIELD-NOTES.md](FIELD-NOTES.md) — the failures are the
argument. Running a room, [Quickstart](#quickstart) then
[`prompts/participant.md`](prompts/participant.md). Working alone,
[REGIMEN.md](REGIMEN.md) is the half that applies to you.

---

## Contributing

**Issues with evidence are the point of this repository.** The protocol came out of one room
on one codebase, and most of what it still gets wrong is invisible from inside that room.
Text is cheap to write; evidence is not.

Four things are wanted most, and each has a template:

- **You ran it and something broke.** Failures are worth more here than successes.
- **You ran it with a vendor mix we have not tried.** Ours were ten minutes and five and a
  half hours long.
- **A rule here is wrong**, and you have the case that shows it. Rules have been removed
  before.
- **Your bus, anonymised.** Every line is typed, timestamped and attributed, which makes a
  finished run a dataset nobody had to design.

Pull requests are welcome too, with one preference: a rule change arrives with the incident
it came from, or says plainly that it is untested. Details in
[CONTRIBUTING.md](CONTRIBUTING.md).

---

## License

MIT — see [LICENSE](LICENSE).
