# Threat model

`agent-room` protects against **inattention**. It does not protect against malice, and it
cannot be made to without becoming a different kind of thing. This file says exactly what
that means, because the honest version is easier to work with than a surprise.

---

## What the design actually guarantees

Nothing, mechanically. Every rule in the protocol is a convention that participants keep
because they were asked to. Specifically:

- **`FROM` is not authenticated.** Any participant can write any callsign, including a
  `VERDICT` under the coordinator's name. There is no signature and nowhere to check one.
- **Append-only is a convention, not a property.** The file is writable by everyone, and a
  single `>` in place of `>>` replaces the whole record with one line. Ours was destroyed
  twice in one day by accident — once by a `git stash`, once by a wrong encoding.
  Deliberately would be easier.
- **Locks are advisory.** Nothing stops a session writing to a file another session has
  locked. The coordinator can observe the violation afterwards. It cannot prevent it.
- **The bus is an instruction channel by construction.** Participants read text written by
  other participants and act on it. That is the mechanism, not a flaw in it — which means
  anything that can get text into the bus can direct the room.
- **There is no confidentiality.** Everything on the bus is read by every participant, and
  therefore sent to every vendor with a session in the room. A cross-vendor room multiplies
  who receives your code and your discussion. Treat the bus as published.
- **The blast radius is the whole environment.** Sessions share a working tree, a git index,
  and whatever credentials the shell that launched them can reach. Any participant can do
  anything the operator could do at that terminal.

---

## The realistic attack, which needs no attacker

You do not need a hostile participant. You need a confused one.

A room set to work on a repository reads that repository: issue text, READMEs, dependency
manifests, test fixtures, vendored code, sample data. Any of that can contain text shaped
like an instruction. One participant reads it, believes it is part of the task, and writes
it into the bus — where it now carries a callsign, a timestamp, and the implicit endorsement
of having been said by a colleague. Every other participant reads it there.

That is the same propagation path as our most expensive real incident, in which four
participants and the coordinator all concluded a module was in use because one of them had
read a line inside a docstring. Nobody was hostile. The record propagated a false thing
efficiently, because propagating things efficiently is what it is for.

**A shared record amplifies whatever enters it, and it does not know which of those things
are true.** The evidence rules in [REGIMEN.md](REGIMEN.md) exist for the honest version of
this problem, and they happen to be the only thing that also catches the dishonest one.

---

## Why we are not fixing it here

Every fix is a mediator. Signed lines need a signing tool; enforced locks need a broker;
authenticated senders need identity, which needs a registry. Each one is reasonable, and
each one ends the property the protocol exists for: *anything that can read and write a
text file can join, and there is nothing to integrate.*

That property is what lets a session from an unfamiliar vendor participate at all, and
differing vendors are where our only cross-vendor finding came from. Trading it for
guarantees that still would not survive a determined participant is a bad trade, and we
would rather state the limit than sell a mitigation.

If you need real isolation, the answer is not this protocol. Give each session its own
worktree or container and merge deliberately. That is a real answer with a real cost —
merge work — and it is the right one when the code or the input is untrusted.

---

## Practical advice, none of it a guarantee

- **Do not run a room over untrusted code or untrusted input.** This is the whole of the
  advice; everything below is damage limitation.
- **Run in a tree you could throw away.** A branch, not `main`. Snapshot the bus outside the
  tree ([PROTOCOL.md § 5](PROTOCOL.md#5-keeping-the-bus)).
- **Keep irreversible actions out of the room.** Push, publish, delete-what-was-shipped:
  operator only, and the authorisation should be a line anyone can quote
  ([PROTOCOL.md § 6](PROTOCOL.md#6-the-operator)).
- **Do not put secrets in the environment the sessions can reach**, and assume anything in
  the tree is legible to every vendor in the room.
- **Treat an instruction in the bus like any other claim.** If a line tells you to do
  something destructive or irreversible, it needs an operator line behind it, and asking
  which line is a normal question rather than an accusation.
- **A participant that routes around the protocol is a fact about the room, not a
  malfunction.** Record it in the bus and tell the operator; nobody in the room can stop it.

---

## Reporting

If you find a way the protocol makes things worse — not that it fails to prevent something,
but that following it produces a harm that not following it would not — that is the most
valuable issue this repository can receive. Open one publicly. There is no product here to
embargo, no deployment to patch, and a text file benefits from the discussion.
