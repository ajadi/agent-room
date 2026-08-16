# The regimen

Rules about knowledge: what makes an assertion believable, what counts as evidence, and
what order to find things out in.

These came out of a room of several independent sessions, but **none of them needs a
room.** A session working alone breaks every rule below in exactly the same way — it just
has nobody to notice. If you run one assistant and no protocol at all, this is still the
useful half. [PROTOCOL.md](PROTOCOL.md) is the other half: the machinery of several
sessions sharing one working tree.

Each rule carries the incident that produced it. The incidents are written up in
[FIELD-NOTES.md](FIELD-NOTES.md); the shapes that recur have names in
[PATTERNS.md](PATTERNS.md).

---

## 1. Two kinds of assertion

Every assertion is one of two kinds, and you say which.

**Claim** — carries evidence a reader can re-run: file and line, command and output, commit
hash. A claim without evidence is void and is struck.

**Opinion** — a judgement about quality or about the future. No evidence is possible. Label
it rather than dressing it as fact.

The point is not catching liars. It is stopping a confident tone from counting as proof.

A bare "agreed" is not a turn either. Agreement means *with which specific claim, and what
convinced me.*

---

## 2. What counts as evidence

**Count constructs, never search hits.** A match you have not opened and read is not
evidence. Our most expensive error was a search that found `from core.netutils import ...`
in a file and concluded the module was in use there. The line was inside a docstring
showing plugin authors which imports exist — not executable code. Four participants
propagated it and it reached the operator as established fact. The rule that came out of it
was then violated seven more times in the same day, by everyone including the coordinator.
Knowing the rule is not the same as having the reflex.

**A count is a measurement: publish its method and its date.** Four methods counting "the
same" thing gave 6, 7, 5 and 8 — each wrong in a different way, which means agreement
between any two of them would have been shared error reading as corroboration. When a
measurement ages, date it rather than deleting it: it was true when it was taken.

**Ask for the command, not the number.** A two-versus-three disagreement about a file count
was settled in one exchange that way: the other participant's glob had swept in two
non-test files. *My glob was wrong, not my arithmetic.*

**Reading gives a hypothesis; running gives a fact.** Say which you did.

**A green test proves nothing until you have seen it fail.** If you cannot show the failure
path, report the test as unproven.

**A truncated or incomplete check is DID NOT COMPLETE, never PASS.** Re-run that one by
name; do not average several checks into an impression.

**A check whose population can be empty must report what it examined**, not only its
verdict. "Checked 0 entries" cannot be misread as safety. A silent pass can.

**Test your instruments on input you know is bad.** A hook meant to warn about stale locks
searched the lock file for a key its format does not contain, and had never fired once. Two
of the coordinator's own wake-up checks read the wrong pattern and the wrong column, and
both were caught by the results looking odd rather than by any verification. A check that
cannot fail is not a check, and its silence reads as an all-clear. **The checks you run on
the work are exactly as likely to be broken as the work.**

---

## 3. Measure third, not first

In this order:

1. **Is it written down?** Specification, standard, ticket, README.
2. **Has it been built before?** Archive, previous release, closed task.
3. **Then measure.**

A room that has learned *run it, don't guess* will run things rather than read the file that
already contains the answer. Three sessions were dispatched to establish empirically why a
packaged application resolved no firmware paths. The operator answered in six words — *it
was in the spec* — and it was, in two rows of a requirements document open for three weeks.
Worse, a handover kit built a fortnight earlier by the same room had already solved that
exact failure deliberately, and had been archived unread.

Both instincts are correct and they compete. This is their order.

**Refine a measurement only when a decision hangs on the difference.** Asked how much
firmware had to be copied, three sessions produced 14.7 MB, then 44.5 MB, then 33.5 MB,
with excellent reasoning about which population each figure counted; two of them corrected
each other's arithmetic and both corrections were right. The real answer was 49 MB. One
`df` ended it: 27 GB free, absolute worst case 2.6 GB — every number in the disputed range
led to the same action.

Measuring can be the avoidance behaviour. It feels like work, it produces defensible
artefacts, and it is much more comfortable than doing the thing and finding out.

---

## 4. Hazards expire differently from facts

A documented **fact** goes stale and becomes wrong: someone reads it and is misled.

A documented **hazard** goes stale and becomes an obstacle you impose on yourself. Nothing
about a remembered danger announces its own expiry, and the cautious-looking response to a
hazard is always to avoid the thing.

A coordinator briefed a live task from a seven-week-old note about a build script that hung
and left unkillable processes, and on that basis told the session to *describe the procedure
a human would follow* instead of building anything. It had been fixed long before. The
session then ran the build end to end in about a minute and produced a correct artifact.

So: **date every hazard, and re-test one before you brief anyone on it.**

---

## 5. Independence has to be mechanical

When you want genuinely independent opinions, a shared record is harmful: the second
reviewer reads the first before writing, and three opinions become one opinion and two
agreements.

For those rounds each participant writes to its own file and they are opened together.
Independence is protected by the mechanics rather than requested in a prompt.

**A reviewer told the answer cannot corroborate it.** Handing a checker a correction
mid-flight is often the right thing to do — better than letting it report something you know
is wrong — but it is no longer a witness afterwards. Record that you did it, and read its
verdict accordingly.

---

## 6. Where knowledge lives

**Knowledge that constrains a file belongs in that file.** A note in a task record dies when
the task is archived; the constraint outlives the task.

**Record once, with pointers.** Three copies of one caveat rot at different rates and then
disagree with each other.

**Deleting a thing does not close what was asked about it.** Of twenty-seven open questions
awaiting a human answer, eight were dead: six had had their subject deleted from the
codebase and two had been answered by the code itself — and four of the six died to a single
deletion nobody swept after. Questions do not become answered when their subject
disappears. They become unanswerable, and the two look identical from outside.

**Compare a label against the artifact it describes.** Two scripts were deleted as
"superseded"; the document actually shipped to customers turned out to descend from the one
that was deleted. The word had been sitting in a task specification for months and had never
been checked against the thing it named.

---

## 7. Check that your own instructions were carried out

The commonest failure is not a refusal. It is a decision nobody executed and nobody checked.
Three coordinator rulings sat undone in a single day. An order needs the same execution
check as a commit.

---

## 8. Two grades and no third

The rule that ends an open-ended hunt is not "stop looking". It is that every finding is
graded as exactly one of:

- **BLOCKS** — the deliverable does not go out until this is fixed.
- **SHIPS WITH A NOTE** — it goes out, and the finding becomes a line in the release note.

There is no third grade and there are no ungraded findings. A morning that had been
producing find-fix-find closed on two blockers, with everything else becoming lines in a
release note. The release note stopped being an afterthought and became the place findings
*go to*, which is what made it safe to stop looking.

The room could grade severity perfectly well before. What it lacked was a grading scheme
with a **terminal state**.

Related, for whatever you hand over: **when the reader is going to do something, write the
procedure, not the cause.** A session explaining why some fields were blank caught its own
error a minute later — the operator needed the next three keystrokes and what to read
afterwards. Its own phrasing is the keeper: *I documented a cause where the operator needed
a procedure.* An explanation is what you write when the reader is you.

---

## 9. Every ban carries a sanctioned alternative

A rule that forbids the only route somebody knows is a rule they route around. Two
participants independently reached for `git stash` within half an hour, for the same
legitimate reason — wanting a clean baseline to compare against — and one of them destroyed
the shared log doing it. The ban existed. The sanctioned path did not.

So when you write a rule, write the permitted way to reach the same goal next to it. The
bans in this protocol are listed with their alternatives in
[PROTOCOL.md § 9](PROTOCOL.md#9-bans-and-their-sanctioned-alternatives). If you meet a ban
with no alternative, that is a defect in the rule: report it rather than inventing your own
way round.
