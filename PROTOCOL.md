# The protocol

**Version 0.2.1** — see [Changes](#changes) at the end. State the version you are running in
your `HELLO` if your room mixes vendors, because a participant working from an older copy
will read `HH:MM` timestamps and will not know what a `STANDDOWN` is.

The machinery: the shared file, the message format, locks, liveness, arbitration. It assumes
only that several independent sessions share **one mutable thing and one record** — a
working tree, a document set, a bench, a database. Code is the case we have run; it is not
the case the rules are about.

What the core cannot know is which resources your domain has and which operations on them
are dangerous. That is a **profile** (§ 4), and there is currently one, because there is
evidence for one.

Its companion is [REGIMEN.md](REGIMEN.md) — the rules about evidence and knowledge, which
apply whether or not you are in a room.

## Terms

| Term | Meaning |
|---|---|
| **the bus** | The shared append-only file, `Busfile.md`. Everything crosses it and nothing is private on it. |
| **the room** | The whole arrangement: participants, the bus, the rules below. |
| **participant** | An independent session. Not spawned by anything in this repository. |
| **coordinator** | A participant that arbitrates, assigns and checks claims. Has no special powers over anyone's process. |
| **operator** | The human. May hold a callsign (§ 6); alone can sanction an irreversible action. |
| **callsign** | A participant's name in the bus. Deliberately not the model name. |
| **profile** | The domain-specific half: resources, rules and bans for one kind of shared work (§ 4). |

---

## Requirement language

Since v0.2.1 the normative parts are marked, so that a reader — or a tool built on this
document — can tell a rule from an explanation.

| Marker | Meaning |
|---|---|
| **MUST**, **MUST NOT** | Breaking it breaks the room for other participants. Not a matter of judgement. |
| **SHOULD** | Do it unless you have a reason not to, and say the reason in the bus. |
| **MAY** | Permitted. Named here so that nobody has to ask. |

Text without a marker is narration, evidence, or the incident a rule came from. Those
carry no obligation and are not there by accident: **a rule whose incident is deleted
becomes folklore**, and the first person to find it inconvenient will delete it.

A few rules deliberately carry no marker. Their force was genuinely ambiguous in v0.2 and
assigning one here would have been a change disguised as an edit; they keep their original
wording and are listed under [Undetermined](#undetermined-in-v02) at the end.

---

## 1. The line format

A line MUST have six columns, pipe-separated, one message per line:

```
| HH:MM:SS | FROM | TYPE | TO | TARGETS | TEXT |
```

- **HH:MM:SS** — MUST come from a real clock call in the same command that writes the line,
  never from memory. A coordinator that estimates its own timestamps will drift, and
  timestamps are what resolve conflicts.

  Seconds are there because the tiebreak needs them: two locks taken in the same minute are
  otherwise separated only by callsign, which decides by alphabet a question the clock could
  have answered. Lines in the older `HH:MM` form MUST be read as `HH:MM:00`.
- **FROM** — your callsign.
- **TO** — MUST be a callsign, a comma-separated list of them, or `*` for everyone.
- **TARGETS** — exact resources: paths, or the pseudo-resource tokens your profile defines
  (§ 3). MUST NOT be a bare container — a directory, a whole schema — unless you really are
  rewriting all of it. MUST be `-` when the message has no target, as `HELLO` and `BEAT`
  have none.
- **TEXT** — MUST NOT contain a pipe character.

**Append only.** You MUST write with `>>`, never `>`, and MUST NOT edit, delete, reorder or
trim anyone's lines, including your own past ones. To correct something, append a line
naming the timestamp of the one you are correcting. A single missing angle bracket in a shell command
turns an append into a silent replacement of the whole record with your one line, and if
the bus is untracked there is nothing to restore it from except a snapshot (§ 5). Ours was
in fact destroyed twice in one day — by a `git stash` and by a wrong encoding — which is
why § 5 exists.

**UTF-8 only, pinned explicitly.** The file MUST be UTF-8 without a byte-order mark, and you
MUST NOT rely on a shell's default to make it so: what a redirect writes depends
on the shell, its version and the host, and one participant writing UTF-16 puts null bytes
in the file, at which point every grep- and tail-based reader classifies the bus as binary
and the entire room goes blind simultaneously. Ours did. This is the most likely first
failure when a new vendor joins, and the recipes per tool are in
[VENDORS.md](VENDORS.md).

Line endings MAY be LF or CRLF. Readers MUST accept both, because a mixed-platform room
produces both in the same file.

### The line, formally

For anyone writing a validator. This describes the message lines only — a bus also contains
the prose and headings it was started from. A line that does not match the grammar is not a
message: readers have always skipped such lines rather than erroring on them, which is what
lets the bus carry its own instructions at the top.

```abnf
line       = "|" pad timestamp pad "|" pad from pad "|" pad type pad
             "|" pad to  pad "|" pad targets pad "|" pad text pad "|"

pad        = *SP                    ; alignment padding; readers trim it

timestamp  = 2DIGIT ":" 2DIGIT [ ":" 2DIGIT ]   ; HH:MM is legacy, read as HH:MM:00
from       = callsign
type       = "HELLO" / "BEAT" / "CLAIM" / "LOCK" / "UNLOCK" / "COMMIT" /
             "ASK" / "ANSWER" / "NOTE" / "BLOCK" / "VERDICT" /
             "STANDDOWN" / "BYE"
to         = "*" / callsign *( "," pad callsign )
targets    = "-" / target  *( "," pad target )
target     = pseudo / path
pseudo     = "@" name
callsign   = name
name       = 1*( ALPHA / DIGIT / "-" / "_" )
path       = 1*pchar
text       = *tchar

pchar      = %x21-7B / %x7D-7E / %x80-10FFFF    ; no "|", no space
tchar      = %x20-7B / %x7D-7E / %x80-10FFFF    ; no "|", no CR, no LF
```

What the grammar cannot say, and the prose above can: that the file is append-only, that a
timestamp comes from a real clock, that a `VERDICT` comes from the coordinator, and what
any of the types mean. **A line can be perfectly well-formed and a serious violation.**

Worked examples, valid and deliberately broken, are in [conformance/](conformance/).

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
| `STANDDOWN` | Stopping without leaving: timer cancelled, locks released, unlanded work named by path. | any |
| `BYE` | Leaving. All own locks are void from this line on. | any |

A `TYPE` MUST be one of these, and a participant MUST NOT write a `VERDICT`.

A `VERDICT` MUST be complied with on the next turn. You MAY disagree with an `ASK`; if the
verdict is repeated you MUST comply, and MUST record the objection.

### What a HELLO must contain

A `HELLO` MUST carry five things, and the last two are the ones people leave out:

1. **Session id** — whatever identifies this session to its own tooling.
2. **Process id**, so a human can tell a hung session from a closed one.
3. **Vendor and model.** Recorded once, on entry, for diagnosis if behaviour drifts — never
   to weight anyone's vote. Your callsign is your name from here on.
4. **Timer job id, or the words `no timer`.** The id, not the intention: one session in our
   run believed it had a timer for an hour and did not. You MUST read it back from the
   scheduler and quote what the scheduler said.
5. **What your silence means** — `dead or closed` if your timer is armed, `waiting` if it is
   not. Both readings are live in any mixed room, they are exact opposites, and no third
   party can tell them apart from the file.

```
| 09:14:22 | ALFA | HELLO | * | - | session 7f3a1c pid 41288 vendor-x model-y timer job 12 silence means dead-or-closed |
```

A coordinator MUST add a sixth: where it keeps its bus snapshots (§ 5).

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

A stand-down line MUST carry what is unfinished, every path it holds unlanded —
uncommitted, unsaved, unsent — and the locks released. **Releasing them is not optional:**
you MUST release every lock you hold. A session that is not reading the bus cannot answer
an `ASK` about a lock, so anything it kept would block the room until the operator noticed.
Those paths stay its property — you MUST NOT revert or sweep work you did not create, and a
stood-down participant is not there to defend it.

---

## 3. Locks

You MUST NOT write to a shared resource without an active `LOCK` covering it. Reads are
free: you MAY read anything, at any time, without announcing it.

### Pseudo-resources

Shared state that is not a file still needs claiming. Such a resource is written `@name`
and locked exactly like a path.

Which tokens exist is a property of the domain, so the list lives in the profile (§ 4) —
`@git`, `@tests`, `@build`, `@env` and `@hardware` are the ones the coding profile defines.
You SHOULD derive your own from the question *"what does this mutate outside my locked
paths?"*, and SHOULD extend the list on the first incident rather than designing it up
front.

### Acquisition

You MUST follow all four steps, in order:

1. Read the tail of the bus and build the set of active locks.
2. Overlap with an active lock → you MUST NOT take it. Work on something else, `ASK` for an
   ETA, or `BLOCK` to the coordinator.
3. No overlap → append your `LOCK`.
4. Re-read the tail. If someone claimed the same target in the same moment: **earlier
   timestamp wins; on a tie, the alphabetically lower callsign wins.** If you lost, you MUST
   append `UNLOCK` immediately and back off.

### Discipline

- You SHOULD lock the narrowest target set that works, and MUST NOT lock `.` or `*`.
- **Releasing your lock is part of finishing:** you MUST `UNLOCK` when the work is done. A
  lock outliving its task is the most common defect we saw — a closed, archived task still
  holding an entire directory, silently making other participants stop and verify.
- Unlanded work you did not create belongs to someone else. You MUST NOT revert it or sweep
  it into your own. Your profile names the specific forms this takes.

---

## 4. Domain profiles

Everything above is domain-neutral: it assumes several sessions, one mutable thing, one
record. What it cannot know is which resources your domain has and which operations on them
destroy something.

A **profile** supplies exactly that — the pseudo-resource tokens, the rules for the domain's
sharpest edges, and its bans with their sanctioned alternatives. The core applies in full
alongside it; a profile adds and never overrides.

| Profile | Domain | Evidence |
|---|---|---|
| [coding in a shared git tree](profiles/coding-shared-git.md) | several sessions in one working tree with one git index | three days, one room — [FIELD-NOTES.md](FIELD-NOTES.md) |

This section held the git rules until v0.2.1. They are in the profile now, unchanged.

**There is one profile because there is evidence for one.** How to write another is at the
end of that file, and the requirement is the same as for any rule here: it comes from a room
that ran and something that broke, or it is marked untested. A profile invented at a desk
would be exactly the confident, fluent, unfounded text this protocol exists to resist.

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

Who owes one was settled in the field, after the first wording blocked the one participant
that could not write outside the tree
([FIELD-NOTES.md](FIELD-NOTES.md#every-rule-the-coordinator-wrote-was-too-wide)):

- The coordinator MUST take a snapshot **on every wake** — it is the one participant with
  this in its cycle, and it bounds the worst case to one coordinator interval.
- Before any operation your profile names as dangerous to the file, a current snapshot MUST
  **exist — whoever took it**. The duty is on the existence, not on the person about to
  act: the per-committer form was a rule a participant could be structurally unable to
  obey. In the coding profile the dangerous operation is `@git`, because every loss we have
  recorded came from a git operation in the shared tree.

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

The coordinator MUST state the snapshot location in its `HELLO` — a backup nobody else can
find is not one, and a precondition on a snapshot *existing* is checkable only if everyone
knows where to look. What is lost on restore is everything appended since the last snapshot, so if
you restore the bus you MUST say so in it, and from when — the gap is invisible otherwise,
and lines that were acted on will appear never to have been written.

### Integrity

Before acting on what you read, check the file is a file: no null bytes, and a plausible
line count against what you last saw. Both failures announce themselves this way and both
blind everyone at once.

Whatever you use for that check, you SHOULD run it once against a deliberately broken copy
— a truncated file, or one with a null byte in it. There are ready-made broken ones in
[conformance/](conformance/). Our own integrity check reported null bytes
in every line of a clean file because its pattern matched everything, and the who-is-idle
check read the recipient column instead of the sender. Neither was caught by verification;
both were caught by the output looking odd. See [REGIMEN.md § 2](REGIMEN.md#2-what-counts-as-evidence).

### Reading at size

Tailing N lines answers "how busy is the bus," not "what is addressed to me," and past some
size it cannot answer the second question at all. At three hundred and thirty kilobytes and
a thousand lines, one participant missed a coordinator answer for six minutes and then
missed an explicit approval *while asking for it*
([FIELD-NOTES.md](FIELD-NOTES.md#the-bus-outgrew-tail-reading)).

Past roughly that size, or on the first missed addressed message, you SHOULD read by cursor
instead: search for your own callsign from the line number you recorded on your last wake,
and record the new number when you finish. Control the first empty result before trusting it
— a replacement cursor in that same room was itself found wrong, in the open, minutes later.
This is SHOULD rather than MUST only because it depends on what a participant's tooling can
do; the failure it prevents does not.

### Rollover — proposed, untested

The live file is never made smaller: editing or trimming anyone's lines stays MUST NOT at
any size. The sanctioned route for a bus that has outgrown reading is to **close the file
whole and start a new one**. The incident behind this exists for the problem — the thousand
lines above — not for the cure: no room has run a rollover yet, and like the v0.2 additions
at their release it is marked so. The first room that runs one will find out which parts are
ceremony.

1. **Trigger.** The coordinator SHOULD roll over on its own wake at a threshold — around a
   thousand lines or three hundred kilobytes, the size at which ours failed — or at a day
   boundary, which also removes the ambiguity of dateless `HH:MM:SS` timestamps crossing
   midnight.
2. **Mechanics.** Snapshot first (§ 5 applies to the closing file like any other moment).
   Rename the live file to a dated archive — `Busfile.archive.YYYYMMDD-HHMMSS.md`, kept
   beside the bus, with a copy outside the tree — and start a fresh `Busfile.md` whose
   header carries the protocol version, the name of its predecessor, and a `ROLLOVER` line
   saying what was closed and when. The archive is never edited, by anyone, for anything.
3. **State crosses by re-assertion, not by summary.** The danger of a rollover is that the
   active state — locks, lanes, open `ASK`s and `BLOCK`s — lives in the tail being closed.
   The coordinator posts in the new file only a *list* of what it believes is active. Every
   owner MUST re-assert its own locks and lanes in fresh lines on its first wake; the
   coordinator's list is not a source — the same rule as authorisations, a quote rather than
   a paraphrase. A lock not re-asserted within one coordinator interval is released.
4. **Cursors.** A rollover resets line numbers. The `ROLLOVER` line is the first thing a
   reader checks before trusting a cursor recorded against the previous file (see Reading at
   size, above).

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
room, coordinator included, MAY grant permission on the operator's behalf. That rule is
weak when authorisation arrives as somebody's paraphrase and strong when it is a line in
the bus with a timestamp anyone can quote. If a participant cites an authorisation, you
SHOULD ask which line it is. Absence is not disproof: the operator has channels you cannot
see, so you MUST NOT claim that no such authorisation exists — you can testify to what
passed through the bus and to nothing else.

Rules, such as they are:

- The operator MAY take a callsign like everyone else (`OSCAR` in our template) and appear
  in the participants table.
- **Only the operator sanctions an irreversible action** — pushing, publishing, deleting
  something already shipped to third parties, anything a product or risk decision hangs on.
  A `VERDICT` MUST NOT substitute for it.
- The operator has no timer, so its silence means *waiting*, not *dead*. It MUST declare
  that in its `HELLO` exactly as a participant with a timer declares the opposite.
- If the operator writes to a shared resource it MUST take a `LOCK` like anyone. Speaking in
  the bus needs none.
- The operator is not obliged to answer, and a question addressed to it does not block the
  room unless the answer gates the work. You SHOULD say which.

**Relaying, new in v0.3** — both rules from the coordinator being caught, twice, reporting a
delivery it had not seen ([FIELD-NOTES.md](FIELD-NOTES.md#reporting-at-dispatch-time)):

- **A relayed operator instruction MUST carry a provenance marker** — whose line it relays,
  or which channel outside the bus it arrived on. A participant challenged exactly this: a
  relay without provenance rests on a line no reader of the bus can verify. The testimony
  rule above extends to relays — the coordinator testifies to what passed through it, and to
  nothing else.
- The coordinator MUST NOT report an instruction as delivered until the addressee has
  answered. **The loop is relay, wait, report** — posting is dispatch, not delivery, and a
  heartbeat saying "processed" is a claim of having read, not evidence of compliance.
  Reporting at dispatch time is what makes a room look inert to the only person who cannot
  see inside it.

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

Each participant arms a repeating self-wakeup whose prompt is: re-read the tail of the bus,
act on anything addressed to you, advance your task by one step. The interval SHOULD be five
minutes for a participant and an hour for the coordinator, which is driven by what others
write rather than by polling.

If you arm one, you MUST verify it by listing your scheduled jobs and quoting the job id —
"armed" from memory does not count. If you cannot arm one at all, the room still works;
see [VENDORS.md](VENDORS.md) and say `no timer`, because the meaning of your silence
inverts.

You MUST state the consequence explicitly in your `HELLO`:

- With a timer, **your silence means dead or session closed** — never "finished".
- Without one, **your silence means waiting** — the opposite reading.
- After a `STANDDOWN`, **it means not listening until a human intervenes** — a third
  reading, and the one nobody guesses.
- Timers are usually session-scoped: they die with the session and will not resurrect it
  after a usage limit. Only a human can restart a dead session.

### The declaration must stay true

`no timer` stays a supported, declared state — requiring a scheduler would exclude whole
tools, against the point of the design. But the declaration alone has now been run and
found insufficient: a schedulerless reviewer went dark for thirty-nine and then sixty-eight
minutes, its silence was read backwards, and review stopped for a morning
([FIELD-NOTES.md](FIELD-NOTES.md#a-critic-that-dies-silently)). Three rules came out of
that morning:

- **A change of timer state MUST be announced the moment it happens** — armed to cancelled,
  registered to found dead, `no timer` to armed. The room reads your silence by your last
  declaration, and a stale declaration is a lie on the bus.
- **Proof of a working wakeup is a line on the bus written by the resumed session** — never
  an exit code, never the scheduler's own report. This tightens "verify by listing your
  scheduled jobs" above: the listing proves registration; only the line proves waking. A
  scheduler reporting success proves a process launched, nothing about whether anyone woke,
  read, or acted — both silent timer deaths that morning would have passed an exit-code
  check.
- A participant that has declared `no timer` SHOULD NOT hold a gating role — a reviewer in
  a propose–critique cycle, anything the room stops and waits for — without a named relay.
  While the reviewer was dead, the room completed zero critique passes for a morning.

---

## 9. Ending a turn

The last line of a turn MUST say what is unfinished, what you hold unlanded by path, and
which locks you released. "I have stopped" and "I am busy" look identical from outside, and
distinguishing them is the coordinator's most common blind spot.

---

## 10. Bans and their sanctioned alternatives

Two participants independently reached for `git stash` within half an hour, for the same
legitimate reason — wanting a clean baseline to compare against — and one of them destroyed
the shared log doing it. The ban existed. The sanctioned path did not, and **a rule with no
sanctioned path is a rule people route around.**

Every prohibition in this protocol is listed here with the permitted way to reach the same
goal. **Every left-hand cell is a MUST NOT; every right-hand cell is the sanctioned route to
the same goal.** If you find a ban without an alternative, that is a defect in the rule: say
so in the bus rather than inventing your own way round.

| Do not | Because | Do this instead |
|---|---|---|
| Write the bus with `>` | One missing angle bracket silently replaces the whole record with your one line | `>>`, always — and snapshots (§ 5), because the bus has been lost to other operations too |
| Rely on your shell's default encoding | Defaults differ by shell, version and host; UTF-16 blinds every reader at once | Pin UTF-8 explicitly. On PowerShell: `[IO.File]::AppendAllText("Busfile.md", $line + [Environment]::NewLine, [Text.UTF8Encoding]::new($false))` |
| Edit, delete, reorder or trim anyone's lines, including your own | The record's only value is that nobody can revise it | Append a correction naming the timestamp of the line you are correcting. For a bus that has outgrown reading: a rollover (§ 5), never a trim |
| Lock `.`, `*`, or a bare container | Stops everyone and hides what you are actually touching | Name the resources. If you truly are rewriting all of it, say so and get a `VERDICT` |
| Write to anything shared without a lock | The lock is the only thing anyone can see | Take the lock first. Reads are free — read as much as you like |
| Take a lock that overlaps a live one | Two writers, one file, one survivor | Work elsewhere, `ASK` for an ETA, or `BLOCK` to the coordinator |
| Revert or sweep unlanded work you did not create | It belongs to a session that may be mid-edit | `ASK` its owner; if nobody answers, `BLOCK` and let the coordinator rule |
| Stamp a timestamp from memory | The coordinator's drifted by over an hour, twice, and timestamps arbitrate | A real clock call in the same command that writes the line |
| Push, publish, or delete anything shipped to third parties | The only irreversible actions available to you | Ask the operator, naming the target. The authorisation is then a bus line anyone can quote |

Add a row on the first incident rather than designing the list up front — the same rule as
the pseudo-resources in § 3.

**Your profile extends this table.** The coding profile adds five rows, all of them about
git: [`profiles/coding-shared-git.md` § 4](profiles/coding-shared-git.md#4-bans-and-their-sanctioned-alternatives).
Together the two tables are exactly the fourteen bans that stood in v0.2.

---

## 11. Assignments

New in v0.3. The costliest failures of our second room were not false claims — every
mechanism for catching those worked, repeatedly. They were assignments: a lane silently
replaced and orphaned, lanes with no origin anywhere, five and a half hours of real work on
the wrong queue. These rules bind whoever assigns, which is normally the coordinator.

**An assignment line MUST name what it displaces** — either the lane it parks *and who owns
the blocker it is parked against*, or the words `displaces nothing`. A participant MAY
refuse a malformed assignment. Ours orphaned two lanes forty minutes apart by assigning over
them: one sat unowned for three hours, and it was the most delivery-relevant thing in the
room ([FIELD-NOTES.md](FIELD-NOTES.md#two-orphaned-lanes-forty-minutes-apart)).

**An assignment line MUST name its origin** — the queue or backlog row it comes from, or the
words `finding, not on the queue`. And on every wake the coordinator MUST post **one line of
queue reconciliation**: queue tasks open, closed this interval, active lanes from the queue
against active lanes from findings. One line, not a report — the point is the ratio, visible
to everyone including the operator, and a reconciliation that grows into ceremony will be
skipped. This pair is the only mechanism this protocol has against its dominant recorded
failure: seven lanes assigned in five and a half hours, not one from the open backlog, half
the queue already finished the whole time, and progress on findings reported as progress on
the queue. Every rule then in force was obeyed, and none could have noticed
([FIELD-NOTES.md](FIELD-NOTES.md#where-every-lane-came-from),
[what it cost](FIELD-NOTES.md#what-it-cost-and-what-it-was-worth)).

**The coordinator SHOULD keep a register of what is blocked on the operator** and post it
consolidated at a regular point, not only as escalations scattered through the bus. In our
second room fifty-four verdict lines carried individual escalations and the register was
never posted once — so each participant could see its own blocker and none could see the
set, while they were the ones parking lanes against it
([FIELD-NOTES.md](FIELD-NOTES.md#reporting-at-dispatch-time)). Five decisions sat with the
operator at the end of that day; the operator learned the count from these notes.

---

## Conformance for tools

Nothing here needs a tool, and people will write them anyway — validators, append helpers,
snapshotters, dashboards that count how long locks were held. That is a good thing and this
section exists so that they can be built without asking us anything.

A conforming tool:

- **MUST leave the bus readable and appendable by a participant with nothing but a shell.**
  This is the whole rule. Everything else follows from it.
- **MUST NOT be a condition of joining.** If a session cannot participate without running
  your tool, the tool is not conforming, however useful it is.
- **MUST NOT write a line the grammar in § 1 cannot express**, and MUST NOT add a column, a
  header, a sidecar index or a lock file that other participants have to know about.
- **MAY do anything else on top**: validate, snapshot, project a view, compute metrics from
  a finished bus, warn about stale locks, tail the file into somebody's terminal.

The test is a subtraction: **stop the tool mid-run and the room MUST carry on.** If it
cannot, the tool had become the protocol.

Why this is a rule rather than a preference: the participants worth having are the ones from
an unfamiliar vendor, because their blind spots are in different places from everyone
else's — that is the only cross-vendor finding we have. A tool requirement excludes exactly
those participants, and it does it silently, by making them look incompatible when they are
merely different.

This is the only rule v0.2.1 adds. It constrains tools, not participants; no behaviour
required of a session changed.

---

## Undetermined in v0.2

Rules whose force was genuinely ambiguous when the requirement language was introduced in
v0.2.1. Each keeps its original wording, because deciding it here would have been a change
of the protocol disguised as an edit.

**This is a list of admissions, not a roadmap.** Most of them are settled the same way:
somebody runs a room and finds out which reading survives contact.

A room ran v0.2.1 on 2026-08-17 ([FIELD-NOTES.md](FIELD-NOTES.md#the-second-room-v021)) and produced field
answers for the first two rows. The rows record them; the decisions belong to 0.3, because
settling them here would be a rule change in a patch.

| Rule | The two readings | What would settle it |
|---|---|---|
| Arming a timer (§ 8) | § 8 tells every participant to arm one; the `HELLO` contract (§ 2) and [VENDORS.md](VENDORS.md) both treat `no timer` as a supported, declared state. MUST, or MAY-with-declaration? | **Ran, 2026-08-17.** The declaration was not enough: a schedulerless reviewer went dark twice, its silence was read backwards, and review stopped for a morning ([FIELD-NOTES.md](FIELD-NOTES.md#a-critic-that-dies-silently)). Decision for 0.3 |
| Snapshotting the bus (§ 5) | Imperative in the text, but the scope is unstated: every participant always, or the coordinator on our behalf? | **Ran, 2026-08-17**, and answered differently than either reading: a per-committer duty blocked the one participant that could not write outside the tree. The room amended it to "a current snapshot exists, whoever took it" ([FIELD-NOTES.md](FIELD-NOTES.md#every-rule-the-coordinator-wrote-was-too-wide)). Decision for 0.3 |
| Snapshot before a dangerous operation (§ 5, profile § 3) | Force follows the rule above | — |
| Snapshot on every coordinator wake (§ 5) | Force follows the rule above | — |
| Announcing the snapshot location in `HELLO` (§ 2, § 5) | Force follows the rule above; cannot be stronger than the rule it announces | — |
| Checking the bus is intact before acting (§ 5) | Imperative, but no frequency: before every read, or once per wake? Before every read is expensive for a five-minute participant | Measuring what the check costs against how often the bus is actually damaged |
| A resource path containing a space (§ 1) | Nothing in v0.2 forbids it, and nothing says how to trim the column around it. The grammar added in v0.2.1 excludes spaces from a path rather than inventing a quoting rule | A room on a codebase with spaces in its filenames, which is most Windows work |

If you have run a room and one of these has an obvious answer from experience, that is a
[rule challenge](CONTRIBUTING.md) worth opening.

---

## Changes

The protocol is versioned because rooms mix vendors and copies drift. Each entry says what
would break for a participant running the older text.

**2026-08-17.** v0.2.1 has now been run: one room, five and a half hours, mixed vendors —
[the second room](FIELD-NOTES.md#the-second-room-v021). The rule changes it motivates are
collected for 0.3; none are made retroactively here.

| | Meaning for a participant on the previous version |
|---|---|
| **patch** (0.2 → 0.2.1) | Nothing changes in a working room. Text moved, or was said more precisely. |
| **minor** (0.1 → 0.2) | New rules or message types. You still interoperate, but you read the room incompletely — an unknown type looks like noise. |
| **major** | The line format or the tiebreak changed. You are not compatible; the room must agree on one version. |

### 0.2.1 — 2026-08-16 · recompaction, no change of behaviour

**Semantic diff of participant-facing rules against 0.2: empty.** If you are running 0.2,
nothing you do is now wrong. What changed is where things live and how precisely they are
stated.

- **The core is domain-neutral, and git moved to a profile** (§ 4). The protocol claimed to
  be about several sessions sharing one mutable thing while assuming a git working tree
  throughout. Rules are unchanged and unmoved in force; five of the fourteen bans are now in
  [profiles/coding-shared-git.md](profiles/coding-shared-git.md), which is the only profile,
  because it is the only one with evidence.
- **Requirement language** (MUST / SHOULD / MAY), so a rule can be told from an explanation.
- **A grammar for the line** (§ 1), for anyone writing a validator, plus
  [conformance/](conformance/) — worked buses, valid and deliberately broken. Data, not a
  test suite.
- **Conformance for tools** — the one rule this version adds. It constrains tools, not
  participants: a tool must leave the bus usable by a session that has nothing but a shell,
  and must not be a condition of joining.
- **Seven questions the requirement language could not answer** are listed under
  [Undetermined](#undetermined-in-v02) rather than settled by whoever happened to be
  editing. The sharpest: § 8 tells every participant to arm a timer while § 2 and
  [VENDORS.md](VENDORS.md) both treat `no timer` as a supported state.

Numbering is unchanged, including § 4, which is now the pointer to profiles: other files
link to the anchors for § 5, § 6 and § 10, and renumbering would have broken them silently.

### 0.2 — 2026-08-16 · not yet exercised in a live room

Everything in this release is generalised from an incident in
[FIELD-NOTES.md](FIELD-NOTES.md), but **no room has run it yet** *(true when written; a
room ran v0.2.1 on 2026-08-17 — see the note above)*. The evidence in this
repository is evidence about v0.1. Treat the additions below as proposals with good
provenance: nobody has posted a `STANDDOWN`, lost a tiebreak on seconds, or restored the bus
from a snapshot. If one of them turns out to be ceremony, that is a finding and it should be
removed.

- **Timestamps carry seconds** (§ 1). Older `HH:MM` lines read as `HH:MM:00`. A participant
  writing minute-resolution timestamps still interoperates; it just loses tiebreaks it
  might have won.
- **`STANDDOWN`** (§ 2). A participant that does not know the type will read it as an
  unknown line and may mistake a stood-down session for a dead one.
- **`HELLO` has five mandatory elements** (§ 2), the timer job id and the meaning of your
  silence among them.
- **`TARGETS` is `-`** when a message has no target.
- **Keeping the bus** (§ 5) — snapshots outside the working tree, and the requirement to
  test an integrity check against a deliberately broken file.
- **The operator may hold a callsign** (§ 6), which turns an authorisation from a paraphrase
  into a line anyone can quote.
- **The rules about evidence moved to [REGIMEN.md](REGIMEN.md)** (§ 7), because they hold
  for a single session with no room at all. Three of them are new here: measure third rather
  than first, date and re-test a hazard, and grade findings `BLOCKS` or `SHIPS WITH A NOTE`
  with no third option.
- **Every ban is tabulated against its sanctioned alternative** (§ 10).

### 0.1 — 2026-08-11

First public text, from two days of one room on one codebase.
