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
| HH:MM | FROM | TYPE  | TO      | TARGETS                | TEXT                          |
| 14:02 | ALFA | LOCK  | *       | core/parser.py         | rewriting the token scanner   |
| 14:03 | BRAVO| ASK   | ALFA    | core/parser.py         | how long, I need it after you |
| 14:05 | ALFA | UNLOCK| *       | core/parser.py         | done, 41 tests green          |
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

So each participant arms a repeating self-wakeup, typically five minutes: wake, re-read the
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
- **It does not protect against malice**, only against inattention.
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

## Files

| File | What it is |
|---|---|
| [`PROTOCOL.md`](PROTOCOL.md) | The message format, lock rules, and arbitration in full |
| [`Busfile.template.md`](Busfile.template.md) | Drop-in starter log with the protocol sections filled |
| [`prompts/participant.md`](prompts/participant.md) | Startup prompt for a worker session |
| [`prompts/coordinator.md`](prompts/coordinator.md) | Startup prompt for the coordinator |
| [`FIELD-NOTES.md`](FIELD-NOTES.md) | Measured observations from real use, including failures |
