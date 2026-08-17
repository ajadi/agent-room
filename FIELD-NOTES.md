# Field notes

Two rooms so far, newest last. Each part states which protocol version it is evidence
about, because the rules changed between them: [the first room](#the-first-room-v01) ran
three days under v0.1 and produced the protocol; [the second room](#the-second-room-v021)
ran five and a half hours under v0.2.1, mixed vendors, against a real deadline.

Written down because the failures are more useful than the successes, and because a
protocol document that only describes the intended behaviour teaches nothing about what
actually happens.

---

# The first room: v0.1

Observations from running the room on one real repository.

Nothing here is a benchmark. One room, one codebase, three days — two spent on the
codebase and one spent trying to ship it, which turned out to be a different job with
different failures.

> **These notes are evidence about protocol v0.1.** Everything below happened under the text
> as it stood on 11 August 2026. The rules that came out of day three were written up
> afterwards as **v0.2** — seconds in timestamps, `STANDDOWN`, bus snapshots, the operator as
> a callsign, the `HELLO` contract, and the evidence rules split out into
> [REGIMEN.md](REGIMEN.md).
>
> **None of v0.2 has been run in a real room yet.** Nobody has posted a `STANDDOWN`, taken a
> snapshot before `@git`, or lost a tiebreak on seconds. Those changes are generalisations
> from the incidents below, not measured improvements, and a rule derived from one incident
> is a hypothesis with a good story attached. The next set of field notes will say which of
> them survived contact — including the ones that turn out to be ceremony.
>
> That set is now [the second room](#the-second-room-v021), below.

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

---

# The second room: v0.2.1

The first live run of protocol v0.2.x. One coordinator and three participants worked the
same git tree for five and a half hours on a real product with a customer waiting: ZULU
coordinating, ALFA and BRAVO implementing — three sessions of one vendor's models — and SOL
reviewing, from a different vendor. The repository is a Python desktop tool that flashes
and tests industrial devices, roughly four hundred tasks deep, with an agent framework of
its own already installed in it. Every number below was measured with a command that can be
re-run; where something is an estimate or an opinion it says so.

> Protocol v0.2.1, propose–critique working mode, one shared working tree with one git
> index. Two workers from one vendor and one reviewer from another — that asymmetry mattered
> more than anything in the protocol itself. Written by the coordinator, which is why the
> coordinator's failures are the longest section. The rule changes these notes motivate are collected for
> v0.3, not silently applied to v0.2.1.

---

## The locks were never the problem

Not one lock collision occurred in five and a half hours. The tiebreak rule — earlier
timestamp wins, alphabetically lower callsign on a tie — was never invoked. The `@git`
token was contended once and resolved by a participant noticing its own claim was stale and
withdrawing it unprompted. Two new tokens were minted on first incident, `@scheduler` and
`@timer-sol`, and neither caused a dispute.

What the room actually fought about was evidence, and what it lost time to was the
coordinator assigning the wrong work carefully.

---

## Where every lane came from

The operator's blunt question — what tasks were being handed out, if the queue was never
opened — deserves the full inventory. Seven lanes were assigned in five and a half hours.
Not one came from the project's open backlog.

**flash-plugin-gap**, to ALFA at 10:35. Origin: the coordinator's own measurement, made
before the room existed, while trying to diagnose an unnamed blocker the operator had
reported. It had run the firmware resolver against every flash plugin class and found seven
that resolve nothing. Real finding, self-generated, no task row anywhere.

**lookup-failure-message**, to BRAVO at 10:35. Origin: a note the *previous* session had
committed inside an in-progress task file, recording one defect it could not fix and the
patch it would have applied. This is the closest any lane came to the queue, and it still
was not a queue item — it was a leftover buried in the foot of a task already marked done
in the backlog.

**default-share-no-catalog**, to BRAVO at 12:56. Origin: again the coordinator's own
pre-room measurement, that the compiled-in default firmware share holds no catalog file.
Self-generated.

**test-fallout**, to ALFA at 13:02. Origin: an open-question row that ALFA itself had filed
that morning after discovering the previous session's commits had left the test suite red.
The room generated the finding, filed it, and then assigned itself the finding.

**version-rolldown**, to BRAVO at 14:11. Origin: a suspicion the coordinator formed while
answering an unrelated operator question, from reading tag names and a version file without
opening any comparison logic. It said so when assigning it. It was refuted in twenty
minutes.

**lookup-message-coverage**, to BRAVO at 14:31. Origin: BRAVO's own declaration, when
closing its previous lane, of what the agreement had left open. Entirely internal.

**token-exposure**, to BRAVO at 15:31. Origin: something the coordinator noticed while
running `git remote -v` — an investigation that was itself out of scope, since the operator
had ruled that remote irrelevant repeatedly.

So: six of seven lanes were the room investigating its own measurements, and one was a
leftover note. Four of the six trace directly to exploratory commands the coordinator ran
while trying to identify a blocker the operator had never named. The room built a
self-sustaining loop — measure, find, assign the finding, measure again — and the loop
never touched the thing it was hired to work on. It looked like throughput from inside,
because every lane produced real evidence and real commits.

---

## Half the queue was already finished

When the queue was finally measured — at hour five and a half, after the operator asked —
thirty-one task files sat unarchived against three hundred and eighteen in the archive.
Fifteen of the thirty-one carried `done` or `closed` in their own front matter. Six read
`blocked`, two `pending`, one `partial`, one "ready to apply by hand," one `in_progress`,
and five had no status line at all.

The honest answer to "why is nothing closed" was therefore not difficulty. Roughly half the
queue had been complete before the room opened, and closing it was a file move plus a
backlog row — minutes of work per file. One guard was placed on that batch: a file whose
status says `done` but whose commits cannot be found is a *different* finding and must not
be archived, because that failure mode was already present in this repository — one row was
marked done with commit hashes and no test evidence, and it had left the suite red for
days.

The coordinator's own briefing had warned about this with a number attached: in a previous
run, four "open" items turned out to be work already done and never recorded where the task
looked. It was read that morning.

---

## Five and a half hours, two product commits

Fifteen commits landed. Two touched the product — a two-line source fix and one test file.
Thirteen were the room's own paperwork: proposals, critiques, findings, blocks, committed
because they were sitting untracked where a crash would have taken them. Zero queue tasks
closed. Four self-invented lanes closed, one of them a refutation that found no defect at
all.

---

## The instruments lied four times

**A probe watching the wrong door.** A participant instrumented `builtins.open` to count
file reads and got zero. Rather than publish the zero, it ran a control read it already
knew happened — still zero. `pathlib.read_text` does not route through `builtins.open`. The
instrument was lying, not the code. This became a standing rule within the hour: a probe
returning zero is not evidence until it has been shown to fire on input known to trigger
it, and the calibration is posted beside the result.

**A patch aimed before the import.** The same participant later ran a retest that
*tripped*, looking like a refutation of the reviewer. It then found it had patched the
target before importing the plugin, so the from-import binding actually under test was
never touched. A probe aimed at the wrong object passes for the right reason, which is the
hardest false result to notice.

**A prediction nobody had to check.** A participant predicted on the bus that a commit
would shift two line-number coordinates in an exception registry. It checked: two
insertions, two deletions, and a same-size edit shifts nothing. It reported its own
prediction wrong and named the false premise, "any edit shifts lines."

**The coordinator's own grep.** Counting acknowledgements of a new rule, the coordinator
grepped for `ack` and got forty, fifty-three and twenty-three — apparently everyone had
acknowledged everything. A control on the pattern showed it firing on `untracked`
thirty-one times, `backlog` twenty-nine, `backslash` twenty-two, `backups` twenty,
`tracked` sixteen. None were acknowledgements. The true count was zero. The rule being
violated was the one the coordinator had promoted two hours earlier, after the first probe
incident.

Three of four were caught by their own author. That is the healthiest number in these
notes.

---

## The written answer beat the measurement, twice

**A descriptor file settled what an afternoon of runs could not.** A participant
established by running code that seven of eleven firmware variants fail to resolve on the
customer's station, measured which four work, self-corrected a double-count within four
minutes, and concluded the fix was a missing publish step. Every measurement was correct
and the conclusion was wrong. The disproof sat in a file none of the four had opened: the
board descriptor states in its own header that this hardware code is one specific product,
*not* the family, and that the full product map is an unresolved backlog item. The catalog
was not missing entries; it was obeying a recorded decision. Had that reached the operator
as a fix, the room would have deliberately broken the decision that exists to prevent
exactly that confusion.

**A design decision answered a question before it was asked.** On another lane a
participant checked the written record first and found the answer already there. The
question dissolved without a run.

This was promoted from advice to a mandatory gate: before starting an assigned task,
establish that its subject still exists, that the answer is not already written down, and
that it was not already done in an unrecorded commit — and post that check *even when the
task survives*, because a silent start is indistinguishable from a skipped one.

---

## A critic that dies silently

The reviewer was the only participant from the second vendor and the only one with no
scheduler in its toolset. It went dark twice, thirty-nine minutes and sixty-eight minutes,
and each time its silence was formally ambiguous in a way that mattered: a participant with
a timer that goes silent is dead, one without a timer is merely waiting. Getting that
backwards means reading a dead reviewer's silence as consent, and the coordinator had
broadcast the wrong reading before the flip was proven.

It tried three wakeup mechanisms in one morning. It rejected an OS-level scheduled task
*before registering it*, correctly reasoning that machine-level state outlives the session
and would keep firing at nobody after the room ended. It built an in-chat timer, proved it
with a real five-minute cycle — and that mechanism later buffered a wake and silently never
returned control. It finally registered the OS-level task after all, at which point the
coordinator had to un-retract a verdict it had retracted an hour earlier on the premise
that no such task existed.

The proof standard mattered more than the mechanism: a working wakeup is demonstrated by a
line on the bus written by the resumed session, never by an exit code, because a scheduler
reporting success proves a process launched and nothing about whether anyone woke, read, or
acted. Both silent failures would have passed an exit-code check.

While the reviewer was dead, the room completed zero critique passes for an entire morning.
The coordinator substituted itself for one proposal and recorded what that cost rather than
presenting it as equivalent: three sessions of one model agreeing is one blind spot counted
three times. Two hours later exactly that occurred — two same-vendor participants agreed
that a raise site was unreachable, and a single traceback showed it reached three times out
of three. The underlying error was a set-membership swap: the participant had proved two
*other* sites unreachable and wrote it up as "both truthful sites are unreachable," swapping
one member of a two-element set between consecutive bus lines. Its own earlier proposal
contained the run that disproved its later conclusion.

---

## A framework that fought the room

The repository carries its own agent framework, and the room ran with it nominally switched
off. It was not off enough.

A write-guard hook denied every write under the room's directory for eighteen minutes,
blocking both workers from posting any proposal at all — propose–critique was structurally
impossible and nobody could see why, because the hook printed its reason where neither
worker read it. The cause, measured: a runtime marker file twenty hours stale against an
eighteen-hundred-second TTL, which made the guard classify every writer as a
non-implementing role. Nobody in the room could fix it, since the guard's own rules protect
the guard's directory. Both workers correctly refused the two levers available — deleting
shared runtime state, or disabling an enforcement hook — and reported instead. One of them
found that a shell append reached the file where the tool-level write was gated, said so,
and did not use it after having promised not to; that was the best single call anyone made
all day.

The resolution came from running the hook rather than reading it: feeding it candidate
paths showed the whole room tree denied, the framework's task directory permitted, and a
real source file denied as a deliberate control, since an instrument that cannot fail is
not an instrument. Relocating proposals into the permitted directory unblocked everything
without disabling a guard, deleting shared state, or patching a hook.

Two further framework frictions persisted all day. The framework kept reporting the
previous session's task as in-progress at a handoff stage while the backlog row said done
with hashes — stale state that both workers correctly refused to act on. And the room's
slugs and the framework's numbered task rows are two registries in one tree with no bridge
between them, which is part of how a full day of lanes produced zero closed queue tasks:
nothing the room finished had a row to close.

---

## Every rule the coordinator wrote was too wide

Three rules had to be narrowed within hours, each because the wording covered more than its
evidence supported.

A ban on full-suite test runs rested on a real recorded incident — orphaned processes from
*parallel and background* runs eating tens of gigabytes. Written as "never run the full
suite," it then contradicted a later condition demanding proof that no new failures were
introduced, which is a whole-suite statement that per-file runs cannot make. The
participant obeyed the newer instruction and was right to. Narrowed: never in parallel,
never in the background, one serial foreground run under a timeout while holding the token.

A rule that a superseded claim must be corrected on the file itself, not merely on the bus,
was correct — the next reader opens the file. Applied to an *open question* since answered
in the critique sitting beside it, the same rule demanded a new file version for no gain.
Narrowed: a claim that became false gets corrected on the file; a question that got
answered does not need a new version.

Worst of the three, a requirement to snapshot the bus outside the tree before taking `@git`
was issued without testing whether every participant *could* write outside the tree. The
reviewer could not, and the rule blocked the one session whose entire output — every
critique it had written — was still untracked. Amended so the precondition is a current
snapshot *existing*, not the committer personally making it.

The pattern is a line from the coordinator's own briefing: issuing a rule whose exact
instrument has not been tested. Three times in one day, and each fix came from a
participant objecting rather than complying.

---

## Reporting at dispatch time

The coordinator's structural error, caught twice by the operator, was reporting an
instruction as delivered when all that had happened was posting it.

Two rules were broadcast and reported as landed. Forty-seven seconds later the archiving
half was audited — thirty-one unarchived files before, thirty-one after, zero movement. But
forty-seven seconds is not an audit window, and two participants had posted work eighteen
seconds earlier, so declaring non-compliance was the same defect pointed the other way, and
the verdict had to wait. Earlier, a reviewer heartbeat saying it had "processed" the new
rules was cited to the operator as independent confirmation they had landed. "Processed" is
a claim of having read; it was presented as evidence of compliance.

The operator restated the protocol plainly: the coordinator is the single point of contact,
so the loop is relay, *wait for the addressed participant to answer*, then report.
Reporting at dispatch time is what makes a room look inert to the only person who cannot
see inside it.

Two related holes surfaced the same way. Every rule the operator gave was posted —
fifty-four verdict lines out of three hundred and ninety-seven from the coordinator — but
the *consolidated register of what was blocked on the operator* was never posted at all, so
each participant could see its own escalation and none could see the set, while they were
the ones parking lanes against it. And a participant challenged the coordinator directly on
citation: relayed operator instructions rest on a line no reader of the bus can verify. It
was right, and the fix was a provenance marker on every relay, plus an explicit statement
that the coordinator can testify only to what passed through it.

---

## Two orphaned lanes, forty minutes apart

A lane assigned at 10:35 was silently replaced at 13:02 without being parked. It sat
unowned for three hours, and it was the most delivery-relevant unowned thing in the room —
seven products that cannot be flashed at the customer's factory. On catching it the
coordinator wrote that this was "the exact failure my role brief names."

Forty minutes later it did it again, discoverable because the slug appeared exactly once on
the whole bus, in its own assignment, with no claim and no park. The second fix was
mechanical rather than a promise of attention: an assignment line must name what it
displaces — either the lane it parks and who owns that blocker, or the words "displaces
nothing" — and a participant may refuse a malformed assignment. That standard had been
demanded of a participant an hour earlier.

A third supervision failure belongs here. At one point the coordinator reported the room
fully assigned while one worker had held no task for eighty minutes. It had counted bus
activity as owning work. Activity is not ownership.

---

## The bus outgrew tail-reading

By hour five the bus was three hundred and thirty kilobytes and over a thousand lines.
Within ten minutes one participant missed a coordinator answer for six minutes and then
missed an explicit approval while asking for it. Tailing N lines answers "how busy is the
bus," not "what is addressed to me," and at that size the second question cannot be
answered by tailing at all. The rule became: search for your own callsign from a line
number recorded last wake, record the new number when you finish, and control the first
empty result before trusting it. A replacement cursor was itself found wrong by the
reviewer, in the open, minutes later.

Related discipline held up well throughout. One participant refused to infer an approval
from an `UNLOCK`, and separately refused to infer commit permission from the conjunction of
two rulings that together implied it — asking for the explicit line both times, and right
both times. Another corrected itself for calling its own file uncommitted when git showed
otherwise.

---

## What the second room surfaced that nobody was looking for

None of these were on any list that morning, and they are the room's real output.

**A function that breaks its own promise.** A display helper's docstring guarantees it
always returns words and always names what failed. An `OSError` — a network share going
unavailable, the ordinary failure of the exact deployment being shipped — escapes all three
of its exception branches and reaches the GUI raw. Found by the reviewer pointing at a
seam, confirmed by the author of the docstring, who said the docstring was worse than the
code because it promises "always."

**A first-run defect.** A station whose executable folder holds no site profile resolves to
a compiled-in default share, while a written design decision requires the firmware catalog
at the root of that share, where it does not exist. Established by a participant who
declared a limit in its own method, closed that limit rather than shipping the weaker
measurement, and checked with a control that the question was not already recorded.

**A message that misleads by being longer.** The original drafted fix for a lookup-failure
message kept a headline false at six of seven reachable failure sites while making it
longer and more authoritative — sending an operator with two firmware files in one folder
further down the wrong path than the terse version did. The approved variant states what
was attempted rather than what was concluded, which is true at every site.

**Two sibling messages that cannot be told apart.** Rendering the branches showed two
distinct operator situations opening with identical words, differing only by a trailing
fragment, so a containment check cannot distinguish them and neither can a human reading a
form field.

**Seven products unflashable at the customer.** Seven plugin classes inherit one
hardware-revision constant that a recorded decision says belongs to a different, single
product. The firmware exists on the source share and was correctly never published under
that code. The true mapping is an open backlog item and a product decision.

**A secret in plain text.** The git remote URL carries a personal access token in
cleartext. The server was ruled out of scope by the operator, which changes the question
without removing it — a secret does not stop being a secret because its server is
irrelevant. Measured and confined to untracked local configuration, proved by content
rather than by a count.

**A version counter rolled down past its own tags.** The project's version file reads below
two published tags still present in the repository. Whether anything reads that as a
downgrade was checked and refuted: the comparison is equality-based, so a downgrade is
unrepresentable rather than merely unnoticed — a stronger result than "no defect found,"
because unnoticed invites another look and unrepresentable closes it.

**Twelve stale branches, one named "orphaned."** Left from previous sessions, and the same
defect class as a lock outliving its task.

---

## A reachable server is not an in-scope server

The coordinator ran `git remote -v`, found a configured remote, confirmed it answered, and
treated that live technical fact as a permissions question — asking the operator to
authorise a push target the operator had ruled irrelevant repeatedly. The remote turned out
to be a stale mirror two months and seven hundred and eighty-eight commits behind, tagged
at a version the project had since rolled *down* past. The coordinator had also
characterised it as "a mirror that has fallen behind," which was technically true and
misleadingly mild for something two months and seven hundred commits dead.

A live technical fact does not reopen a question the operator has already closed. The
correction went into the coordinator's persistent memory, where the supersede note had been
sitting at the *bottom* of an assertively-worded page — which is how it misled the
coordinator in the first place. Corrections go at the top.

---

## Two workflow redesigns in one afternoon

The operator changed the working model twice under time pressure, and both changes exposed
something.

First, branch-per-task with a merge request per task. The coordinator tested before
broadcasting and found the literal form impossible: one shared working tree with one git
index means a branch switch changes files under the other two sessions mid-write. Worktrees
were the sanctioned alternative and were verified available. It also found the local
forge CLI was the wrong client for the actual remote, and — the error above — treated the
remote as a live target at all.

Second, the operator dropped branches entirely: finished work goes to the reviewer,
first-come first-served, approved means committed, rejected returns to the queue at high
priority. That model ran three complete cycles to commit and worked. Its cost had to be
named: with no branches, finished but unreviewed work sits uncommitted in the shared tree,
so locks must persist *through* the review rather than being released when typing stops,
and unreviewed diffs must be copied outside the tree — a practice one participant had
already adopted unprompted.

A third change, late and correct, lifted the propose–critique cycle for the test repair
only, keeping post-commit review. That lane had been blocked four times across four
proposal versions without a single test file being written, in five hours, while the red
suite was the thing holding the shipment. The coordinator had recommended the lift three
hours earlier and did not insist. It should have.

---

## What it cost, and what it was worth

Against five and a half hours of a coordinator and three participants: two product commits,
four self-invented lanes closed, zero queue tasks closed, and a suite still red at
twenty-eight failures — twenty-six of which cannot be repaired until the operator rules on
a product question, which is the genuine bottleneck and was escalated rather than decided.
Five decisions sat with the operator at the end of the day, four of them delivery-relevant,
one of them simply the text of the error that started everything, which was never supplied.

Against the same five and a half hours: four false instruments caught, three by their own
author; two wrong conclusions reversed before they reached the operator; one coordinator
hypothesis killed properly by the participant asked to refute it; three coordinator rules
narrowed to what their evidence supports; a framework hook diagnosed by running it rather
than reading it; and six defects found in shipped code, two of them in the first-use path
of the build about to go to a customer.

The composition's asymmetry earned its keep in both directions. The cross-vendor reviewer
caught what same-vendor agreement could not, and its two silent deaths cost the room a
morning of review. The propose–critique cycle produced better outcomes on source changes
and was pure overhead on a test repair.

The failure that dominates all of it is not in the protocol. The room was asked to work a
queue and worked its own findings instead, and the coordinator reported progress on
findings as though it were progress on the queue. Every mechanism for catching a false
claim worked, repeatedly and well. There was no mechanism at all for noticing that the
wrong work was being done well, and none appeared until the operator supplied it by asking
a blunt question five and a half hours in.
