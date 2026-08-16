# agent-room

Several independent AI coding sessions working in the same repository at the same time,
coordinating through one append-only text file.

No server. No database. No framework. Nothing to install.

The participants are not spawned by anything — they are separate programs that already
exist, possibly from different vendors, that have agreed to write to the same file.

---

## The problem this solves

An AI coding assistant is a process: it reads files, edits them, runs tests, commits.
One of them is predictable.

Several of them in one working tree is not. They cannot see each other. Each assumes it
owns the files. Two edit the same file and one edit is lost. One stages a change, another
commits without a pathspec and carries it away. Two start the test suite at once and the
machine swaps.

The usual answer is to give each one its own copy and merge later. That is real merge
work, and some tasks need shared state anyway.

`agent-room` is the other answer: let them share the tree, and give them somewhere to talk.

---

## The mechanism, in sixty seconds

A file called `Busfile.md` sits in the repository root. Every participant appends lines to
it. Nobody edits or deletes anyone else's lines.

```
| HH:MM:SS | FROM | TYPE  | TO   | TARGETS        | TEXT                          |
| 14:02:11 | ALFA | LOCK  | *    | core/parser.py | rewriting the token scanner   |
| 14:03:40 | BRAVO| ASK   | ALFA | core/parser.py | how long, I need it after you |
| 14:05:02 | ALFA | UNLOCK| *    | core/parser.py | done, 41 tests green          |
```

Participants use **callsigns**, not model names. Before touching a file you announce a
`LOCK`. When you are finished you `UNLOCK`. If two sessions claim the same file at once,
the earlier timestamp wins; on a tie, the alphabetically lower callsign wins. No
negotiation.

One participant acts as **coordinator**: arbitrates conflicts, assigns work, and checks
claims. The coordinator is not a filter — every participant writes into the same file
directly, and the coordinator is corrected in it like everyone else.

That is the whole system.

---

## Why not subagents, or a role plugin for your editor

Those tools let a main agent hire helpers: assign roles, sometimes assign a different
model per role. It looks like a team. Structurally it is a manager with interns.

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
- **Nothing stands between a participant and the record.** A disagreement lands in the
  file verbatim and everyone reads the same text.
- Participants may be from different vendors. Models from one family fail in similar ways
  — similar training data, similar habits. Four of the same can miss the same thing
  together. A different vendor's blind spots are in different places.
- Any tool that can read and write a text file can join. There is nothing to integrate.

---

## Prior art

None of the machinery is new, and it is worth saying so before someone else does.

A shared structure that independent specialists read from and write to, never calling each
other directly, is a **blackboard architecture** — Hearsay-II was doing it for speech
understanding in the 1970s. Coordinating through a shared store rather than through
messages between named parties is **Linda's tuple space**, from the 1980s. An append-only
record where the log is the source of truth is older than either. Callsigns, timestamps and
"say what your silence means" are radio procedure.

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
- **The medium is a text file.** Not a bus you implement — a file, readable by a human and
  by anything that can run one shell command.

---

## Three rules that turned out to matter more than the locks

**Claims and opinions are different, and you say which you are making.**
A claim carries evidence anyone can re-run: a file and line, a command and its output, a
commit hash. An opinion cannot be proved and is labelled as an opinion. A claim without
evidence is struck. The point is not catching liars — it is stopping a confident tone from
counting as proof.

**Callsigns, not model names.**
A vote should not weigh more because of whose model produced it, or the argument decays
into status. Vendor and version are recorded once on entry, for diagnosis if behaviour
drifts, not for authority.

**Closed rounds when you want genuinely independent opinions.**
A shared file is harmful here: the second critic reads the first before writing, and three
opinions collapse into one plus two agreements. For those rounds each participant writes
to a separate file and they are opened together. Independence is enforced by the
mechanics, not requested in a prompt.

---

## Timers: the part without which none of it runs

An assistant that has finished its turn stops reading anything. It is not idling and
watching the file — it is waiting for input and will not see a word you write.

So each participant arms a repeating self-wakeup — five minutes for participants, hourly for the coordinator: wake, re-read the
tail of `Busfile.md`, act on anything addressed to you, take one step, sleep.

The side effect matters more than the mechanism. **With a timer, silence means the session
is dead.** Without one, silence means the opposite — alive and waiting to be poked. Those
are contradictory readings of the same quiet file, and confusing them is expensive.

---

## Quickstart

1. Copy [`Busfile.template.md`](Busfile.template.md) to `Busfile.md` in your repository root.
2. Give each session the [participant prompt](prompts/participant.md), with a callsign.
   Give one session the [coordinator prompt](prompts/coordinator.md).
3. Have every participant arm a five-minute self-wakeup and verify it, not assume it.
4. Do not commit `Busfile.md` if it would leak internal discussion — but know that an
   untracked file has no backup. Pick deliberately.

---

## What it does not do

- **It cannot wake anyone.** The protocol has no way to reach a session that has stopped
  reading. Participants wake themselves; a session killed by a usage limit stays dead
  until a human restarts it.
- **It cannot compel.** A coordinator can observe a violation and demand a correction. It
  cannot stop another session. The whole thing rests on good faith, and a participant
  determined to route around it will.
- **It does not protect against malice**, only against inattention. `FROM` is not
  authenticated, append-only is a convention rather than a property, and the bus is an
  instruction channel by construction — anything that gets text into it directs the room.
  The realistic version needs no attacker, only a participant that read something
  instruction-shaped in the repository it was working on. [THREATS.md](THREATS.md) sets out
  what follows from that and why the fix would cost more than it buys.
- **Vendor differences bite early.** The first cross-vendor incident in our own run was not
  a disagreement about code — one participant wrote the log in a different text encoding
  and every other tool declared the shared file binary. The whole room went blind at once.

---

## What we are willing to claim

Narrowly: **a room finds defects that no single participant finds alone.**

We do not claim it produces better solutions. To measure that you would need to know the
best solution in advance.

See [FIELD-NOTES.md](FIELD-NOTES.md) for what two days of real use actually produced,
including the parts that went wrong.

---

## Contributing

**Issues with ideas are the point of this repository.** The protocol came out of one room
on one codebase, and most of what it still gets wrong is invisible from inside that room.

Especially wanted:

- **You ran it and something broke.** Failures are worth more here than successes — the
  most useful sections of [FIELD-NOTES.md](FIELD-NOTES.md) are the ones about things that
  went wrong. Say what you expected, what happened, and what the room did about it.
- **You ran it with a vendor mix we have not tried.** Ours was three sessions of one model
  plus one of another, for ten minutes. Anything beyond that is new evidence.
- **A rule here is wrong, or right for the wrong reason.** Every rule in
  [PROTOCOL.md](PROTOCOL.md) came from a specific incident, and an incident is a sample of
  one. If you can show a case where a rule makes things worse, that is the most valuable
  issue you can open.
- **A rule is missing.** Deadlocks, more than a handful of participants, sessions on
  different machines, humans participating in the room as callsigns — all unexplored.
- **The measurement problem.** Nobody here has measured whether a room beats one model run
  several times over. If you have a way to test that honestly, open an issue about the
  method before the result.

Pull requests are welcome too, but an issue describing what happened is usually the more
useful contribution: the protocol is short on purpose and it needs evidence more than it
needs text.

---

## Files

| File | What it is |
|---|---|
| [`PROTOCOL.md`](PROTOCOL.md) | The message format, lock rules, and arbitration in full |
| [`Busfile.template.md`](Busfile.template.md) | Drop-in starter log with the protocol sections filled |
| [`prompts/participant.md`](prompts/participant.md) | Startup prompt for a worker session |
| [`prompts/coordinator.md`](prompts/coordinator.md) | Startup prompt for the coordinator |
| [`FIELD-NOTES.md`](FIELD-NOTES.md) | Measured observations from real use, including failures |
