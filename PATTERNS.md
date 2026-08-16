# Named failures

Ways that several AI sessions — or one, mostly — go wrong in a way that repeats. Each has a
name because a pattern you can point at is one somebody can object to in a sentence, and
half of these were only recognised in our own run *after* they had happened twice.

Every entry links to the incident it was drawn from in [FIELD-NOTES.md](FIELD-NOTES.md).
Most of them cost us something. None of them requires several sessions to occur — a single
assistant produces the same shapes, with nobody in the room to notice.

---

### Hit counted as construct

Treating a search match as evidence of the thing you were searching for, without opening
the file. Grep does not distinguish executable code from a comment, a docstring, a test
fixture or a changelog entry.

Ours: a search found `from core.netutils import ...` and four participants concluded the
module was in use there. The line was inside a docstring listing imports available to plugin
authors. It reached the operator as established fact, and the rule that came out of it was
broken seven more times the same day by everyone including the coordinator — twice by the
coordinator within five minutes, while investigating whether a credential had leaked.

*[The single most expensive error](FIELD-NOTES.md#the-single-most-expensive-error)*

---

### Shared error mode reading as corroboration

Two methods agree, so the answer looks confirmed. But if both methods share a blind spot,
their agreement measures the blind spot, not the answer. Agreement is only evidence when the
methods fail differently — which you have to establish, not assume.

Ours: four ways of counting the same thing gave 6, 7, 5 and 8, each wrong differently. Any
two of them agreeing would have looked like proof. This is also the argument for mixing
vendors: four sessions of one model family are four instances of one method.

*[Counting is harder than it looks](FIELD-NOTES.md#counting-is-harder-than-it-looks)*

---

### Measurement as avoidance

Refining a number that no decision depends on. It is comfortable, it looks like rigour, it
produces defensible artefacts, and it postpones the act that would actually settle the
question. The tell is that you cannot say which decision changes if the number changes.

Ours: three sessions produced 14.7 MB, then 44.5 MB, then 33.5 MB, correcting each other's
arithmetic correctly along the way. The answer was 49 MB, and one `df` showed 27 GB free
against a worst case of 2.6 GB — every value in the range led to the same action.

*[Three sessions refining a number that no decision depended on](FIELD-NOTES.md#three-sessions-refining-a-number-that-no-decision-depended-on)*

---

### Stale hazard inversion

A recorded fact goes stale and misleads the reader. A recorded **hazard** goes stale and
becomes an obstacle you impose on yourself — and nothing about a remembered danger announces
its own expiry, because the cautious-looking response to a hazard is always to avoid it.

Ours: a seven-week-old note about a build script that hung turned a live task into a
documentation exercise. It had been fixed long before; the session then ran the build end to
end in about a minute.

*[Briefing a live task from a stale memory note](FIELD-NOTES.md#briefing-a-live-task-from-a-stale-memory-note)*

---

### Measuring what is already written down

A room that has learned *run it, don't guess* will run things rather than read the document
that contains the answer, and rebuild rather than check the archive. Both instincts are
correct; what is missing is their order.

Ours: three sessions were dispatched to establish empirically why a package resolved no
firmware paths. The operator answered in six words — *it was in the spec* — and it was, in
two rows open for three weeks. A kit built a fortnight earlier by the same room had already
solved that exact failure, deliberately, and had been archived unread.

*[Sending three agents to measure what the specification already answered](FIELD-NOTES.md#sending-three-agents-to-measure-what-the-specification-already-answered)*

---

### Guard that cannot fire

A check that has never fired, cannot fire, and whose silence is read as an all-clear. Worse
than no check, because no check at least looks like no check.

Ours: a hook meant to warn about stale locks searched the lock file for a key its format
does not contain. Later, two of the coordinator's own wake-up checks read the wrong pattern
and the wrong column — and one of them did it twice, hours apart, having been noticed in
between. The uncomfortable symmetry: the checks a coordinator runs on the room are exactly
as likely to be broken as the checks the room runs on the code, and nobody reviews them.

*[What the room surfaced that nobody was looking for](FIELD-NOTES.md#what-the-room-surfaced-that-nobody-was-looking-for)*
· *[The coordinator's own instruments, again](FIELD-NOTES.md#the-coordinators-own-instruments-again)*

---

### The volunteer register

A record that only conscientious participants write to cannot support any claim about the
population. A clean file is equally consistent with everyone being disciplined and with
nobody using it at all, and the two readings are indistinguishable from inside.

Ours: the lock file is populated only by sessions that use it, so no assertion about it
could establish that the tree was unclaimed. A conscientious session leaves stale entries; a
session that never registers leaves nothing, and looks better for it.

*[What the room surfaced that nobody was looking for](FIELD-NOTES.md#what-the-room-surfaced-that-nobody-was-looking-for)*

---

### The record proving itself

The artifact documenting a defect turns out to contain the defect. Funny once, and then a
reliable sign that you are looking at a live generator rather than a backlog.

Ours: of fourteen stale locks, one was held by a task whose whole directory was claimed by
the record documenting stale locks. Two hours after all fourteen were cleared the population
was growing again, and the newest had been created by the participant writing up the
problem, when it closed a task and did not release the lock. **The defect was generative,
not historical** — which is a different repair.

*[What the room surfaced that nobody was looking for](FIELD-NOTES.md#what-the-room-surfaced-that-nobody-was-looking-for)*

---

### Questions outliving their subject

Deleting a thing does not answer what was asked about it. The question quietly becomes
unanswerable instead, and an unanswerable question is indistinguishable from an open one in
every list that holds it.

Ours: of twenty-seven questions awaiting a human answer, eight were dead — six had had their
subject deleted and two had been answered by the code itself. Four of the six died to a
single deletion that nobody swept after.

*[What the room surfaced that nobody was looking for](FIELD-NOTES.md#what-the-room-surfaced-that-nobody-was-looking-for)*

---

### The unchecked label

A word in a specification — *superseded*, *deprecated*, *unused*, *legacy* — that nobody has
ever compared against the artifact it describes. It propagates by being quoted, and every
quotation makes it look more settled.

Ours: two scripts were deleted as "superseded". The document actually shipped to customers
descends from the one that was deleted, established by unpacking it and probing five
distinctive strings. The word had been in the task specification for months.

*[What the room surfaced that nobody was looking for](FIELD-NOTES.md#what-the-room-surfaced-that-nobody-was-looking-for)*

---

### Silence semantics collision

Two participants in the same room whose silence means opposite things. A session with a
timer that goes quiet is dead or closed; a session without one is waiting to be spoken to;
a session that has stood down is alive and not listening. The file looks identical in all
three cases.

Ours: a cross-vendor participant had no scheduler available at all, so its silence meant
*waiting* while everyone else's meant *dead*, live in the same room and unannounced. This is
the pattern that makes `HELLO` state what your silence means, and it is the first thing that
breaks when an unfamiliar tool joins.

*[Cross-vendor, honestly](FIELD-NOTES.md#cross-vendor-honestly)*
· *[Shrinking the room to one, on purpose](FIELD-NOTES.md#shrinking-the-room-to-one-on-purpose)*

---

### Grading without a terminal state

A find-fix loop that cannot close, because every finding is real and none of them is filed
anywhere that means *done deciding*. The room is not wrong; it simply has no state to stop
in, and severity language without a terminal grade does not create one.

Ours: the instruction that broke the loop was not "stop finding things". It was that every
finding must be graded `BLOCKS` or `SHIPS WITH A NOTE`, with no third option — at which
point a morning's output became two blockers and a release note.

*[Grading findings as two kinds, and nothing in between](FIELD-NOTES.md#grading-findings-as-two-kinds-and-nothing-in-between)*

---

### A cause where a procedure was needed

Explaining why something is the way it is, to a reader who needs to know what to press. The
explanation is usually correct, which is what makes it hard to notice: it is written for the
person who investigated, not the person who has to act.

Ours, in the session's own words: *I documented a cause where the operator needed a
procedure.* The shipped instruction changed from "trust the blank fields" to "leave the box
unticked, press Run, and read the three log lines that name the firmware folder and both
files".

*[Documenting a cause where the operator needed a procedure](FIELD-NOTES.md#documenting-a-cause-where-the-operator-needed-a-procedure)*

---

## Adding one

A pattern belongs here when it has happened at least once to somebody, and the write-up says
what it looked like from inside — the version where it seemed reasonable. A failure that
only reads as stupidity has not been understood yet, and nobody recognises themselves in it.

Open an issue with the incident. Names are cheap and can be changed; the incident is the
part that is hard to get.
