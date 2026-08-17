# Changelog

The full history, newest first. The normative version of this list — what breaks for a
participant running the older text — is [PROTOCOL.md § Changes](PROTOCOL.md#changes); this
file adds the context around each release and records the dated events between releases.

The versioning scheme, from the protocol:

| | Meaning for a participant on the previous version |
|---|---|
| **patch** | Nothing changes in a working room. Text moved, or was said more precisely. |
| **minor** | New rules or message types. You still interoperate, but you read the room incompletely. |
| **major** | The line format or the tiebreak changed. You are not compatible. |

---

## 2026-08-17 — the second room (no version change)

Not a release: an evidence update. A room ran protocol v0.2.1 for five and a half hours —
one coordinator, two same-vendor workers, one reviewer from a different vendor, one shared
git tree, a real deadline. The first live contact for everything added in v0.2.

- [FIELD-NOTES.md](FIELD-NOTES.md#the-second-room-v021) gained the second room, split from
  the first by protocol version.
- [PATTERNS.md](PATTERNS.md) gained seven named failures — among them *working the findings
  instead of the queue*, the run's dominant one — and second incidents for four old ones.
- Two of the seven [undetermined questions](PROTOCOL.md#undetermined-in-v02) got field
  answers: the `no timer` declaration was not enough on its own, and the snapshot duty
  belongs to "a snapshot exists," not to the committer personally. The decisions are queued
  for 0.3, not patched into 0.2.1.
- [VENDORS.md](VENDORS.md#waking-up): proof of waking is a line on the bus from the resumed
  session, never an exit code.
- [ROADMAP.md](ROADMAP.md) published: what 0.3 will decide, every item with its incident.

## 0.2.1 — 2026-08-16 · patch

Recompaction, no change of behaviour. **Semantic diff of participant-facing rules against
0.2: empty.**

- The core became domain-neutral; the git rules moved unchanged into
  [profiles/coding-shared-git.md](profiles/coding-shared-git.md) — the only profile,
  because it is the only one with evidence.
- Requirement language (MUST / SHOULD / MAY), so a rule can be told from an explanation.
- A formal grammar for the bus line, plus [conformance/](conformance/) — worked buses,
  valid and deliberately broken, for anyone writing a validator.
- One new norm, about tools rather than participants: a conforming tool leaves the bus
  usable by a session that has nothing but a shell, and is never a condition of joining.
- Seven questions the requirement language could not answer honestly are listed under
  [Undetermined](PROTOCOL.md#undetermined-in-v02) instead of being settled by an editor.

## 0.2 — 2026-08-16 · minor

Everything generalised from the first room's incidents; untested at release (and first
exercised on 2026-08-17, above).

- Timestamps carry seconds; older `HH:MM` lines read as `HH:MM:00`.
- `STANDDOWN` — stopping without leaving, with its own third meaning of silence.
- The `HELLO` contract: five mandatory elements, the timer job id and the meaning of your
  silence among them.
- `TARGETS` is `-` when a message has no target.
- Keeping the bus: snapshots outside the working tree, integrity checks tested against
  deliberately broken files.
- The operator may hold a callsign, which turns an authorisation from a paraphrase into a
  line anyone can quote.
- The evidence rules moved to [REGIMEN.md](REGIMEN.md), because they hold for a single
  session with no room at all.
- Every ban tabulated against its sanctioned alternative.

## 0.1 — 2026-08-11

First public text, from two days of one room on one codebase: the bus, the six-column
line, locks with the timestamp tiebreak, callsigns, the coordinator, timers, claims versus
opinions.
