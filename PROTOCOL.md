# The protocol

The machinery: the shared file, the message format, locks, shared git, timers. Its
companion is [REGIMEN.md](REGIMEN.md) — the rules about evidence and knowledge, which apply
whether or not you are in a room.

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
| HH:MM:SS | FROM | TYPE | TO | TARGETS | TEXT |
```

- **HH:MM:SS** — taken from a real clock call in the same command that writes the line.
  Never from memory. A coordinator that estimates its own timestamps will drift, and
  timestamps are what resolve conflicts.

  Seconds are there because the tiebreak needs them: two locks taken in the same minute are
  otherwise separated only by callsign, which decides by alphabet a question the clock could
  have answered. Lines written in the older `HH:MM` form read as `HH:MM:00`.
- **FROM** — your callsign.
- **TO** — a callsign, a comma-separated list, or `*` for everyone.
- **TARGETS** — exact file paths, or pseudo-resources (below). Never a bare directory
  unless you really are rewriting the whole tree. Write `-` when the message has no target,
  as `HELLO` and `BEAT` do not.
- **TEXT** — no pipe characters.

**Append only.** Write with `>>`, never `>`. Never edit, delete, reorder or trim another
participant's lines. A single missing angle bracket in a shell command turns an append into
an overwrite; in our own run this destroyed 2200 lines of bus and was recovered by luck.

**UTF-8 only, pinned explicitly.** Do not rely on a default: what a redirect writes depends
on the shell, its version and the host, and one participant writing UTF-16 puts null bytes
in the file, at which point every grep- and tail-based reader classifies the bus as binary
and the entire room goes blind simultaneously. Ours did. This is the most likely first
failure when a new vendor joins, and the recipes per tool are in
[VENDORS.md](VENDORS.md).

---

## 2. Message types

| Type | Meaning | Who |
|---|---|---|
| `HELLO` | Joining. Five mandatory elements, below. | any |
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
| `STANDDOWN` | Stopping without leaving: timer cancelled, locks released, uncommitted work named by path. | any |
| `BYE` | Leaving. All own locks are void from this line on. | any |

A `VERDICT` is complied with on the next turn. Disagree with an `ASK`; if the verdict is
repeated, comply and record the objection.

### What a HELLO must contain

Five things, and the last two are the ones people leave out:

1. **Session id** — whatever identifies this session to its own tooling.
2. **Process id**, so a human can tell a hung session from a closed one.
3. **Vendor and model.** Recorded once, on entry, for diagnosis if behaviour drifts — never
   to weight anyone's vote. Your callsign is your name from here on.
4. **Timer job id, or the words `no timer`.** The id, not the intention: one session in our
   run believed it had a timer for an hour and did not. Read it back from the scheduler and
   quote what the scheduler said.
5. **What your silence means** — `dead or closed` if your timer is armed, `waiting` if it is
   not. Both readings are live in any mixed room, they are exact opposites, and no third
   party can tell them apart from the file.

```
| 09:14:22 | ALFA | HELLO | * | - | session 7f3a1c pid 41288 vendor-x model-y timer job 12 silence means dead-or-closed |
```

A coordinator adds where it keeps its bus snapshots (§ 5).

### STANDDOWN is not BYE

A session that has said `BYE` is gone and its callsign is free. A session that has stood
down still exists and can be restarted by a human with its context intact — but it has
cancelled its timer, so it is not reading the bus and nothing written there will reach it.

That is a **third meaning of silence**, and it has to be stated because the other two are
already in use: with a timer, silence means dead or closed; without one, it means waiting.
After a `STANDDOWN` it means neither — *not listening until a human intervenes*. Ours came
from the operator cutting the room to one worker for a day; the two that stood down
cancelled their timers, posted what they held, and said plainly that only a human could
bring them back. That last sentence is the part that cannot be inferred.

A stand-down line carries what is unfinished, every path held uncommitted, and the locks
released. **Releasing them is not optional.** A session that is not reading the bus cannot
answer an `ASK` about a lock, so anything it kept would block the room until the operator
noticed. The uncommitted paths stay its property: work you did not create is not yours to
revert, stash or sweep, and a stood-down participant is not there to defend it.

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

## 5. Keeping the bus

The bus is the most fragile artifact in the room and the only one everyone depends on. In a
single day ours was destroyed twice: truncated by a `git stash` in the shared tree — about
2200 lines, recovered from a dangling object by luck — and filled with null bytes by a
participant writing UTF-16, at which point every grep- and tail-based reader declared it
binary and all four participants went blind at the same instant.

The code it coordinated had backups and integrity checks. The bus had neither.

### Snapshots

A snapshot is a copy of the file with a timestamp in its name, kept **outside the working
tree**. Every loss we have recorded came from an operation inside the tree, so a copy
inside it is not a backup. If the bus is untracked — often the right choice, since it holds
internal discussion — then git cannot restore it either, and a snapshot is the only copy
that exists.

Take one at the two moments that matter:

- **Before taking `@git`.** Snapshot first, then take the lock. Every documented loss came
  from a git operation in the shared tree.
- **On every coordinator wake.** This bounds the worst case to one coordinator interval.

```sh
# POSIX shell
mkdir -p ../bus-backups
cp Busfile.md "../bus-backups/Busfile.$(date +%Y%m%d-%H%M%S).md"
```

```powershell
# PowerShell. Copy-Item preserves bytes; do not round-trip the file through a redirect
New-Item -ItemType Directory -Force ..\bus-backups | Out-Null
Copy-Item Busfile.md "..\bus-backups\Busfile.$(Get-Date -Format yyyyMMdd-HHmmss).md"
```

Restoring is one command against the newest snapshot:

```sh
cp "$(ls -t ../bus-backups/Busfile.*.md | head -1)" Busfile.md
```

```powershell
Copy-Item (Get-ChildItem ..\bus-backups\Busfile.*.md |
  Sort-Object LastWriteTime -Descending | Select-Object -First 1) Busfile.md
```

The coordinator states the snapshot location in its `HELLO`. A backup nobody else can find
is not one. What is lost on restore is everything appended since the last snapshot, so say
in the bus that you have restored it and from when — the gap is invisible otherwise, and
lines that were acted on will appear never to have been written.

### Integrity

Before acting on what you read, check the file is a file: no null bytes, and a plausible
line count against what you last saw. Both failures announce themselves this way and both
blind everyone at once.

Whatever you use for that check, run it once against a deliberately broken copy — a
truncated file, or one with a null byte in it. Our own integrity check reported null bytes
in every line of a clean file because its pattern matched everything, and the who-is-idle
check read the recipient column instead of the sender. Neither was caught by verification;
both were caught by the output looking odd. See [REGIMEN.md § 2](REGIMEN.md#2-what-counts-as-evidence).

---

## 6. The operator

The human can be a participant, with a callsign and lines in the bus like anyone else. This
is optional — the room runs without it, and ours mostly did — but it is cheap and it closes
a hole that nothing else closes.

Two of the most consequential messages in our run were the operator's, and both were short.
*It was in the spec* ended an afternoon of measurement by three sessions. *You can find bugs
and fix them in a circle for ever* ended the find-fix loop and started the work that
actually shipped. A human's turn costs the room nothing and is often the highest-value line
of the day.

**What it buys: authorisations become citable.** The standing rule is that nobody in the
room, coordinator included, can grant permission on the operator's behalf. That rule is
weak when authorisation arrives as somebody's paraphrase and strong when it is a line in
the bus with a timestamp anyone can quote. If a participant cites an authorisation, ask
which line it is. Absence is not disproof: the operator has channels you cannot see, so
"there is no such authorisation" is a claim nobody in the room is in a position to make —
you can testify to what passed through the bus and to nothing else.

Rules, such as they are:

- The operator takes a callsign like everyone else (`OSCAR` in our template) and appears in
  the participants table.
- **Only the operator sanctions an irreversible action** — pushing, publishing, deleting
  something already shipped to third parties, anything a product or risk decision hangs on.
  A `VERDICT` cannot substitute for it.
- The operator has no timer, so its silence means *waiting*, not *dead*. It declares that in
  its `HELLO` exactly as a participant with a timer declares the opposite.
- If the operator writes to the working tree, it takes a `LOCK` like anyone. Speaking in the
  bus needs no lock.
- The operator is not obliged to answer, and a question addressed to it does not block the
  room unless the answer gates the work. Say which.

---

## 7. Evidence, measurement and independence

This file is the machinery. The rules about **knowledge** — what makes an assertion
believable, what counts as evidence, what order to find things out in — live in
[REGIMEN.md](REGIMEN.md), because they apply to a session working alone just as much as to
a room.

The five that the bus depends on directly, in one line each:

- Every assertion is a **claim** (carries re-runnable evidence) or an **opinion** (labelled
  as one). A claim without evidence is struck.
- **Count constructs, never search hits.** A match you have not opened is not evidence.
- **A count carries its method and its date.** Two methods agreeing may be one error twice.
- **Check whether it is written down, then whether it has been built before, and only then
  measure.**
- **A hazard carries a date and is re-tested before anyone is briefed on it.** A stale fact
  misleads a reader; a stale hazard becomes an obstacle you impose on yourself.

**Closed rounds.** When you want genuinely independent opinions, the shared bus is harmful:
the second reviewer reads the first before writing, and three opinions become one opinion
and two agreements. For those rounds each participant writes to its own file and the
coordinator opens them together — independence protected by the mechanics rather than
requested in a prompt.

Full text, with the incident behind each rule: [REGIMEN.md](REGIMEN.md).

---

## 8. Timers

A session that has finished its turn stops reading. It is not watching the bus.

Each participant arms a repeating self-wakeup (five minutes for participants; hourly for the coordinator, which is driven by what others write rather than by polling) whose prompt is: re-read
the tail of the bus, act on anything addressed to you, advance your task by one step.
Verify it is armed by listing your scheduled jobs — "armed" from memory does not count.

Consequences to state explicitly in your `HELLO`:

- With a timer, **your silence means dead or session closed** — never "finished".
- Without one, **your silence means waiting** — the opposite reading.
- After a `STANDDOWN`, **it means not listening until a human intervenes** — a third
  reading, and the one nobody guesses.
- Timers are usually session-scoped: they die with the session and will not resurrect it
  after a usage limit. Only a human can restart a dead session.

---

## 9. Ending a turn

The last line of a turn says what is unfinished, what you hold uncommitted by path, and
which locks you released. "I have stopped" and "I am busy" look identical from outside, and
distinguishing them is the coordinator's most common blind spot.

---

## 10. Bans and their sanctioned alternatives

Two participants independently reached for `git stash` within half an hour, for the same
legitimate reason — wanting a clean baseline to compare against — and one of them destroyed
the shared log doing it. The ban existed. The sanctioned path did not, and **a rule with no
sanctioned path is a rule people route around.**

Every prohibition in this protocol is listed here with the permitted way to reach the same
goal. If you find one without an alternative, that is a defect in the rule: say so in the
bus rather than inventing your own way round.

| Do not | Because | Do this instead |
|---|---|---|
| Write the bus with `>` | One missing angle bracket overwrites the file; it cost us 2200 lines | `>>`, always |
| Rely on your shell's default encoding | Defaults differ by shell, version and host; UTF-16 blinds every reader at once | Pin UTF-8 explicitly. On PowerShell: `[IO.File]::AppendAllText("Busfile.md", $line + [Environment]::NewLine, [Text.UTF8Encoding]::new($false))` |
| Edit, delete, reorder or trim anyone's lines, including your own | The record's only value is that nobody can revise it | Append a correction naming the timestamp of the line you are correcting |
| `git stash` | It clobbers untracked files, the bus among them | Snapshot the bus, then read the committed version out of the tree: `git show HEAD:path > ../baseline/file` |
| `git add .` or `-A` | Sweeps in other sessions' staged work | Explicit pathspec, plus `git diff --cached --name-only` read before every commit |
| A sweeping `git checkout -- .` | Same, destructively | `git restore <one path you own>` — the single-path form is not the banned one |
| Rewrite history to undo a bad commit | Other sessions have already read and built on it | Annotate: a `NOTE` naming what was swept and by whom, then a follow-up commit |
| Take a commit hash from `git log` | In a shared tree the top of the log is not necessarily yours | Take it from the output of the command that made it |
| Lock `.`, `*`, or a bare directory | Stops everyone and hides what you are actually touching | Name the files. If you truly are rewriting a tree, say so and get a `VERDICT` |
| Write to disk without a lock | The lock is the only thing anyone can see | Take the lock first. Reads are free — read as much as you like |
| Take a lock that overlaps a live one | Two writers, one file, one survivor | Work elsewhere, `ASK` for an ETA, or `BLOCK` to the coordinator |
| Revert, stash or sweep uncommitted work you did not create | It belongs to a session that may be mid-edit | `ASK` its owner; if nobody answers, `BLOCK` and let the coordinator rule |
| Stamp a timestamp from memory | The coordinator's drifted by over an hour, twice, and timestamps arbitrate | A real clock call in the same command that writes the line |
| Push, publish, or delete anything shipped to third parties | The only irreversible actions available to you | Ask the operator, naming the target. The authorisation is then a bus line anyone can quote |

Add a row on the first incident rather than designing the list up front — the same rule as
the pseudo-resources in § 3.
