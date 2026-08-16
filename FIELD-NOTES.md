# Field notes

Observations from running the room on one real repository. Written down because the
failures are more useful than the successes, and because a protocol document that only
describes the intended behaviour teaches nothing about what actually happens.

Nothing here is a benchmark. One room, one codebase, three days — two spent on the
codebase and one spent trying to ship it, which turned out to be a different job with
different failures.

---

## The locks were the cheap part

The protocol was designed around file conflicts. There were **three** in a full day, and
all three were resolved by the timestamp tiebreak with no discussion between participants.

Assertions that did not survive checking: **eight**. Two tasks proposed fixing files that
had been deleted the same day. One participant cited the result of a test run it had not
performed. Another was about to remove a protective rule after misreading why it existed.

The value of a shared record is not that it separates agents. It is that a claim becomes
checkable by someone who did not make it.

---

## The single most expensive error

A search found the text `from core.netutils import ...` in a file, and four participants —
three workers and the coordinator — concluded the module was in use there.

The line was inside a docstring: a documentation example showing plugin authors which
imports are available. Not executable code.

The wrong conclusion was written into two task records and reported to the operator as
established fact before an outside reviewer opened the line and read it.

The rule that came out of it — *count constructs, never search hits* — was then violated
**seven more times in the same day**, by every participant including the coordinator, and
twice by the coordinator inside five minutes while investigating whether a credential had
leaked. Knowing the rule is not the same as having the reflex.

---

## Counting is harder than it looks

Asked how many plugin families used a particular mechanism, four methods produced four
answers: a text search said 6, a path-based scan said 7, a syntax-tree pass said 5, and
counting direct-plus-inherited usage said 8.

Each was wrong in a different way, which means **agreement between any two of them would
have been shared error mode reading as corroboration**. The syntax-tree pass gave the
lowest number and was not the sloppiest — it was the most literal, and it could not see
inheritance.

The number that had been sitting in a task record was true when measured and had gone stale
when two modules migrated. Hence: a count carries its method and its date, or it is not a
fact. When it ages, date it rather than deleting it — it was true of the moment it was
taken.

A separate two-vs-three disagreement about a file count was settled in one exchange because
one participant asked for the **command** instead of arguing the number. The other's glob
had swept in two non-test files. "My glob was wrong, not my arithmetic."

---

## What a shared git index actually does to you

One participant's commit swept in three files belonging to another, because they were
sitting staged in the shared index. It noticed and reported it before fixing anything. The
owner of the swept work replied within a minute that the cause was half theirs — they had
staged files without holding the lock. Both independently concluded: do not rewrite history,
annotate instead.

Later the same rule worked **preventively**: an index audit before a commit revealed a
source-file deletion staged by another session's in-flight subagent. The pathspec kept it
out. The participant's own formulation: *the pathspec protected, the audit informed.*

Both halves are needed and they do different jobs. The pathspec does not require vigilance;
the audit catches what the pathspec cannot — notably a rename, where a pathspec commit
leaves the staged deletion behind.

Two participants independently reached for `git stash` within half an hour, for the same
reason: wanting a clean baseline to compare against. One of them destroyed the shared log
doing it. The ban existed; what was missing was a **sanctioned alternative**, and a rule
with no sanctioned path is a rule people route around.

---

## The log is the most fragile thing in the room

Twice in one day.

**Truncated.** A `git stash` in the shared tree clobbered it. About 2200 lines gone. The
file is deliberately untracked, so git could not restore it — it was recovered from a
dangling object left by the very operation that destroyed it. Luck, not design.

**Corrupted.** A newly joined participant from a different vendor wrote its greeting in
UTF-16. Null bytes entered the file, every grep- and tail-based reader classified it as
binary, and all four participants went blind at the same instant. Repairing it took three
attempts: the first two mangled neighbouring lines because a UTF-16 newline is two bytes
and splitting on `\n` shifted the alignment.

The lesson is not "be careful". It is that the artifact everything depends on had no backup
and no integrity check, while the code it coordinated had both.

---

## Cross-vendor, honestly

A model from a different vendor joined as an independent commit reviewer. It lasted about
ten minutes.

In that time it broke the shared file (above), fixed both of its mistakes in one pass
without being told twice, and found the docstring error that four same-vendor participants
had propagated.

One finding. But one that four others had missed, and which had already been reported
upward as verified fact. That is a real argument for differing blind spots and a sample of
one. It does not establish a method.

The practical friction arrived before any benefit did: encoding, timestamp format, and no
scheduler available to that participant at all — meaning it could not be woken, so its
silence meant *waiting* while everyone else's silence meant *dead*. Two opposite readings
of the same quiet file, live in the same room.

---

## Timers

Before timers, instructions reached participants only when the human relayed them by hand.
One session sat idle for an hour holding an unread assignment.

After: five-minute self-wakeups, verified by listing scheduled jobs rather than asserted.
Every participant then stated in its greeting what its silence means.

The coordinator's own hourly timer did not survive a session restart, and it discovered
this only by checking — after a day spent telling everyone else to verify theirs.

---

## What the room surfaced that nobody was looking for

**Guards that cannot fire.** A hook meant to warn about stale locks searched the lock file
for a key that the file's format does not contain. It had never fired once. A guard that
cannot fire is worse than no guard, because its silence reads as an all-clear.

**A record that proved itself.** Fourteen lock entries belonged to tasks long closed and
archived. One of those entries claimed the whole task directory — meaning the record
documenting that stale locks exist was itself written inside a stale lock.

**A leak, not a backlog.** Two hours after clearing all fourteen, the population was
growing again. The newest one had been created by the participant who was documenting the
problem, when it closed a task and did not release the lock. The defect is generative, not
historical.

**Selection bias in the evidence.** The lock file is populated only by participants who use
it. A conscientious session leaves stale entries; a session that never registers leaves
nothing. So a clean file is equally consistent with everyone being disciplined and with
nobody using it, and no assertion about that file can claim the tree is unclaimed.

**Questions that outlived their subject.** Of twenty-seven open questions awaiting a human
answer, eight were dead: six had had their subject deleted from the codebase and two had
been answered by the code itself. Four of the six died to a single deletion that nobody
swept after. Deleting a thing does not close what was asked about it — the questions
quietly become unanswerable rather than answered.

**A label nobody had checked against the artifact.** Two scripts were deleted as
"superseded". A review found that the document actually shipped to customers descends from
the one that was deleted — established by unpacking the document and probing five
distinctive strings: the deleted script produces all five, the survivor one. The word
"superseded" had been in the task specification for months and had never been compared
against the thing it described.

The follow-up is the better half. The coordinator ordered the file restored. The participant
went and read the project's own formatting standard — which the coordinator had not read —
and **reversed the order**: that standard forbids keeping those generators in the
repository at all, the port is manual, so nothing regenerates the document from either
script and restoring one would break compliance to fix a bookkeeping error. It was right.

---

## The thing that surprised the operator most

The coordinator was corrected **nine times in one day, and every correction was justified**.
Each one was written into the shared file, in public, and each one changed a ruling.

An arrangement where the arbitrator can be corrected by evidence, and where that correction
is part of the permanent record, behaves better than one where the arbitrator cannot be
corrected at all. That turned out to be the point of the whole exercise.

---

# Day three: the room tries to ship

The first two days were spent finding and fixing defects, and the room was good at it. On
the third day the operator stopped that work with one sentence — *you can find bugs and fix
them in a circle forever* — and asked instead for a release he could hand to factory
operators after lunch.

Almost everything below is a failure mode that only appears when a deadline is real and the
output is a folder somebody else has to use. None of them showed up in two days of
bug-hunting.

---

## Grading findings as two kinds, and nothing in between

The instruction that broke the find-fix-find loop was not "stop finding things". It was that
every finding had to be graded **BLOCKS HANDOVER** or **SHIPS WITH A NOTE**, with no third
option.

That single rule converted an open-ended hunt into a closing list. Of everything the three
sessions surfaced in a morning, two items blocked and the rest became lines in a release
note. The release note stopped being an afterthought and became the place findings *go to*,
which is what made it safe to stop looking.

The room had been perfectly capable of grading severity before. What it lacked was a
grading scheme with a **terminal state**.

---

## Three sessions refining a number that no decision depended on

Asked how much firmware had to be copied, the room produced 14.7 MB, then 44.5 MB, then
33.5 MB, with excellent reasoning about which population each figure counted, whether a plan
double-counts, and what a "floor" means. Two of the three sessions corrected each other's
arithmetic, and both corrections were right.

The real answer was **49 MB**. None of the three estimates was close.

The coordinator ended it by running `df`: 27 GB free, and the entire source tree — the
absolute worst case — is 2.6 GB. Every number in the disputed range led to the same action.

The rule that came out of it: **refine a measurement only when a decision hangs on the
difference.** The coordinator's share of the blame is the larger one; the check that
dissolved the question took one command and could have been run an hour earlier.

This is the inverse of day two's lesson. There, measuring instead of assuming caught almost
everything. Here, measuring *was* the avoidance behaviour — it feels like work, it produces
defensible artefacts, and it is much more comfortable than copying 49 MB and finding out.

---

## Briefing a live task from a stale memory note

The coordinator told a session that the build script could not be run from inside the
harness, citing a note recorded seven weeks earlier about a hang that left unkillable
processes. On that basis the session was told to *describe the procedure a human would
follow* instead of building anything.

The operator's reply: that was fixed long ago.

The session then ran the build, end to end, in about a minute, and produced a correct
artifact. The stale note had converted a one-minute action into a documentation exercise —
and would have done so silently, because nothing about a remembered hazard announces its own
expiry.

A documented **hazard** is the one kind of record that inverts when someone fixes it. A
documented fact goes stale and becomes wrong; a documented hazard goes stale and becomes an
*obstacle you impose on yourself*. Date them, and re-test one before you brief anyone on it.

---

## Sending three agents to measure what the specification already answered

The packaged application started, identified its site correctly, and showed no firmware
paths. The coordinator gathered evidence, formed a plausible hypothesis about why, and
dispatched all three sessions to confirm or refute it by running things.

The operator answered in six words: **it was in the spec.**

It was. Two rows in the requirements document, open since three weeks earlier, stated (a)
that the site profile configures exactly one device family and every other family needs its
own per-plugin file, and (b) that a file consisting only of comments falls back to the
factory share **silently** — which is precisely the shape the kit had shipped.

Worse: a handover kit had been built two weeks before, by the same room, and archived. That
kit had deliberately shipped a *deliberately invalid* path so that a misconfiguration would
fail loudly. Nobody read it, and the new kit reintroduced the silent shape it existed to
prevent.

The pattern is not ignorance of the document. It is that **measurement had become the
reflex**, and a room that has learned "run it, don't guess" will run things rather than read
the file that already contains the answer. Both instincts are correct and they compete;
nothing in the protocol said which comes first. It now does — [REGIMEN.md § 3](REGIMEN.md#3-measure-third-not-first):
**check whether it is written down before you measure, and check the archive before you
rebuild.**

---

## Documenting a cause where the operator needed a procedure

When the empty fields turned out to be expected behaviour — they are override boxes,
disabled unless you tick a checkbox — the first response was to explain *why* they are
blank.

The session that wrote it caught its own error a minute later: the operator did not need an
explanation, he needed **the next three keystrokes and what to read afterwards**. The
shipped instruction changed from "trust the blank fields" to "leave the box unticked, press
Run, and read the three log lines that name the firmware folder and both files".

Its own phrasing is the keeper: *I documented a CAUSE where the operator needed a
PROCEDURE.* An explanation is what you write when the reader is you.

---

## Shrinking the room to one, on purpose

Mid-morning the operator cut the room to a single worker for the day. The two stood down
cleanly: cancelled their timers, posted what they held, and stated plainly that they were
now unreachable and only a human could restart them.

That last sentence matters more than it sounds. A session with a timer has a defined
silence — dead or closed. A session that has *cancelled* its timer has a different one, and
if it does not say so, the two are indistinguishable from outside.

The order was reversed within the hour, which is the honest reason there is no measurement
here of what a one-session day costs. What can be said is what the room had already
measured about itself the night before: of nine improvements one participant made to a
single document, **eight came from contact with somebody or something else** — four from
another session's review, one from a coordinator ruling, two from running the artefact, and
one self-generated. Removing two of three participants removes most of that, and the
remaining substitute is execution rather than review.

---

## The coordinator's own instruments, again

Two of the coordinator's wake-up checks read the wrong thing on day three.

The bus-integrity check reported null bytes in every line of a clean file, because the
pattern it used matched everything. The who-is-idle check reported one participant's last
message under another participant's name, because the pattern matched the **recipient**
column instead of the sender column — and it did that twice, hours apart, having been
noticed in between.

Neither caused harm, both were caught by the results looking wrong rather than by any
verification. The uncomfortable part is the symmetry with the guards the room spent two days
cataloguing: **the checks a coordinator runs on the room are exactly as likely to be broken
as the checks the room runs on the code**, and nobody is reviewing them.

---

## What day three did not answer

The handover did not happen before lunch. At the point these notes were written the package
existed, started, read its own configuration and resolved firmware for eleven of eleven
supported boards — and the room had just discovered that it might be resolving them from the
wrong share, one that exists on the build machine and not at the destination.

That is the honest state: a room that can build a deliverable in a morning, and cannot yet
tell you whether the deliverable works anywhere but the desk it was built on. Nobody has
flashed a physical board with it.
