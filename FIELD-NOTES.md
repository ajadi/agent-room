# Field notes

Observations from running the room on one real repository. Written down because the
failures are more useful than the successes, and because a protocol document that only
describes the intended behaviour teaches nothing about what actually happens.

Nothing here is a benchmark. One room, one codebase, two days.

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
