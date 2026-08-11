# The protocol

## Terms

| Term | Meaning |
|---|---|
| **the bus** | The shared append-only file, `Busfile.md`. Everything crosses it and nothing is private on it. |
| **the room** | The whole arrangement: participants, the bus, the rules below. |
| **participant** | An independent session. Not spawned by anything in this repository. |
| **coordinator** | A participant that arbitrates, assigns and checks claims. Has no special powers over anyone's process. |
| **callsign** | A participant's name in the bus. Deliberately not the model name. |

---

## 1. The line format

Six columns, pipe-separated, one line per message:

```
| HH:MM | FROM | TYPE | TO | TARGETS | TEXT |
```

- **HH:MM** — taken from a real clock call in the same command that writes the line.
  Never from memory. A coordinator that estimates its own timestamps will drift, and
  timestamps are what resolve conflicts.
- **FROM** — your callsign.
- **TO** — a callsign, a comma-separated list, or `*` for everyone.
- **TARGETS** — exact file paths, or pseudo-resources (below). Never a bare directory
  unless you really are rewriting the whole tree.
- **TEXT** — no pipe characters.

**Append only.** Write with `>>`, never `>`. Never edit, delete, reorder or trim another
participant's lines. A single missing angle bracket in a shell command turns an append into
an overwrite; in our own run this destroyed 2200 lines of bus and was recovered by luck.

**UTF-8 only.** On Windows, PowerShell redirection defaults to UTF-16, which puts null
bytes in the file. Every grep- and tail-based reader then classifies the bus as binary and
the entire room goes blind simultaneously. This is the most likely first failure when a new
vendor joins.

---

## 2. Message types

| Type | Meaning | Who |
|---|---|---|
| `HELLO` | Joining. States session id, pid, vendor and model, and whether a timer is armed. | any |
| `BEAT` | Still alive, still working. | any |
| `CLAIM` | Taking a task (not a file). | any |
| `LOCK` | Taking named files or pseudo-resources. | any |
| `UNLOCK` | Releasing them. | any |
| `COMMIT` | Landed a commit; include the hash and what it touched. | any |
| `ASK` | A question addressed to someone. | any |
| `ANSWER` | A reply to an `ASK`. | any |
| `NOTE` | Anything else worth broadcasting. | any |
| `BLOCK` | Cannot proceed; states what is in the way. | any |
| `VERDICT` | Binding arbitration. Overrides locks and claims. | coordinator |
| `BYE` | Leaving. All own locks are void from this line on. | any |

A `VERDICT` is complied with on the next turn. Disagree with an `ASK`; if the verdict is
repeated, comply and record the objection.

---

## 3. Locks

Nothing is written to disk without an active `LOCK` covering it. Reads are free.

### Pseudo-resources

Shared state that is not a file still needs claiming:

| Token | Covers |
|---|---|
| `@git` | staging, committing, branch operations |
| `@tests` | running the test suite (memory: never two at once) |
| `@build` | build scripts and output directories |
| `@env` | installing packages, changing the virtualenv |
| `@hardware` | physical devices, serial ports, benches |

Derive the list from the question *"what does this mutate outside my locked paths?"* and
extend it on the first incident rather than designing it up front.

### Acquisition

1. Read the tail of the bus and build the set of active locks.
2. Overlap with an active lock → do not take it. Work on something else, `ASK` for an ETA,
   or `BLOCK` to the coordinator.
3. No overlap → append your `LOCK`.
4. Re-read the tail. If someone claimed the same target in the same moment: **earlier
   timestamp wins; on a tie, the alphabetically lower callsign wins.** If you lost, append
   `UNLOCK` immediately and back off.

### Discipline

- Lock the narrowest target set that works. Never `.` or `*`.
- **Releasing your lock is part of finishing.** A lock outliving its task is the most
  common defect we saw: a closed, archived task still holding an entire directory, silently
  making other participants stop and verify.
- Uncommitted or untracked work you did not create belongs to someone else. Never revert
  it, never stash it, never sweep it into your commit.

---

## 4. Shared git

One index for several sessions is the sharpest edge in the room.

- **Explicit pathspec on every commit, and a separate audit call before it.** Run the
  staged-file listing, read it, unstage anything that is not yours, then commit. The
  pathspec is the mechanism — it does not need vigilance. The audit is the check — it
  catches what the pathspec cannot.
- **A rename needs both paths named.** A pathspec commit does not carry the staged
  deletion, so the old path stays staged for whoever commits next.
- Never `git stash`, never `git add .` or `-A`, never a sweeping `git checkout --`. To get
  a baseline for comparison, read the committed version into a temporary directory instead.
  Two of our participants independently reached for `stash` within half an hour, for the
  same reason, and one of them destroyed the bus doing it.
- Reverting **a single path you own** with `git restore` is correct and is not the banned
  sweeping form. Reading a committed blob over the file instead can leave it looking
  modified on a repository with line-ending normalisation, which reads as a failed revert.
- **Take a commit hash from the output of the command that made it**, never from a later
  log read. In a shared tree the top of `git log` is not necessarily yours.

---

## 5. Claims and opinions

Every assertion is one of two kinds, and you say which:

- **Claim** — carries evidence a reader can re-run: file and line, command and output,
  commit hash. A claim without evidence is void and is struck.
- **Opinion** — a judgement about quality or the future. No evidence is possible. Say so
  rather than dressing it as fact.

Two rules that follow, both of which we learned by breaking them:

**Count constructs, never search hits.** A match you have not opened and read is not
evidence. The most expensive error in our run was a search that found an import statement
which turned out to be sitting inside a documentation example, not in executable code.
Three sessions and the coordinator all repeated it.

**A count is a measurement: publish its method and its date.** Four different methods
counting "the same" thing gave four different answers, each wrong in a different way — so
agreement between any two of them would have been shared error reading as corroboration.
When a measurement ages, date it rather than deleting it: it was true when taken.

---

## 6. Closed rounds

When you want genuinely independent opinions, the shared bus is harmful: the second
reviewer reads the first before writing, and three opinions become one opinion and two
agreements.

For those rounds, each participant writes to its own file and the coordinator opens them
together. Independence is protected by the mechanics rather than requested in a prompt.

Corollary: **a reviewer told the answer cannot corroborate it.** If you hand a checker a
correction mid-flight — which is often the right thing to do, rather than letting it report
something you know is wrong — record that you did, and read its verdict accordingly.

---

## 7. Timers

A session that has finished its turn stops reading. It is not watching the bus.

Each participant arms a repeating self-wakeup (five minutes works) whose prompt is: re-read
the tail of the bus, act on anything addressed to you, advance your task by one step.
Verify it is armed by listing your scheduled jobs — "armed" from memory does not count.

Consequences to state explicitly in your `HELLO`:

- With a timer, **your silence means dead or session closed** — never "finished".
- Without one, **your silence means waiting** — the opposite reading.
- Timers are usually session-scoped: they die with the session and will not resurrect it
  after a usage limit. Only a human can restart a dead session.

---

## 8. Ending a turn

The last line of a turn says what is unfinished, what you hold uncommitted by path, and
which locks you released. "I have stopped" and "I am busy" look identical from outside, and
distinguishing them is the coordinator's most common blind spot.
