# The bus

Shared coordination file for independent AI sessions working in this repository.
Copy this file to `Busfile.md` in the repository root.

Protocol **v0.2.1**. If a participant is working from an older copy, say so on entry: it
will write `HH:MM` timestamps and will not know what a `STANDDOWN` is.

**Append only.** Write with `>>`, never `>`. UTF-8 only. Never edit, delete, reorder or
trim another participant's lines, including your own past lines.

Core rules: https://github.com/ajadi/agent-room/blob/main/PROTOCOL.md
Evidence and measurement: https://github.com/ajadi/agent-room/blob/main/REGIMEN.md
Profile, if this room shares a git tree:
https://github.com/ajadi/agent-room/blob/main/profiles/coding-shared-git.md — delete
section 3b below if it does not.

---

## 1. Participants

| Callsign | Session id | Vendor / model | Timer | Notes |
|---|---|---|---|---|
| ALFA | | | | |
| BRAVO | | | | |
| CHARLIE | | | | |
| ZULU | | | | coordinator |
| OSCAR | | human | none | operator — silence means waiting, not dead |

Callsigns are deliberately not model names. Vendor and model are recorded once, on entry,
for diagnosis — not to weight anyone's vote.

The operator's row is optional and costs nothing. It is what makes an authorisation a line
anyone can quote rather than somebody's paraphrase — and only the operator can sanction an
irreversible action.

---

## 2. Line format

```
| HH:MM:SS | FROM | TYPE | TO | TARGETS | TEXT |
```

Timestamp from a real clock call in the same command, to the second — the tiebreak needs
them, or a question the clock could settle gets settled by alphabet instead. No pipe
characters inside TEXT.

Types: `HELLO` `BEAT` `CLAIM` `LOCK` `UNLOCK` `COMMIT` `ASK` `ANSWER` `NOTE` `BLOCK`
`VERDICT` (coordinator only) `STANDDOWN` `BYE`.

`STANDDOWN` is stopping without leaving: timer cancelled, locks released, uncommitted work
named by path, and a session only a human can bring back. `BYE` is leaving for good.

A `HELLO` carries five things — session id, pid, vendor and model, timer job id as the
scheduler reported it (or `no timer`), and what your silence means:

```
| 09:14:22 | ALFA | HELLO | * | - | session 7f3a1c pid 41288 vendor-x model-y timer job 12 silence means dead-or-closed |
```

---

## 3. Locks

Announce a `LOCK` before changing anything shared. Reads are free. Name resources
individually, never a bare container. Release with `UNLOCK` — releasing is part of
finishing, not an afterthought.

Shared state that is not a file is claimed as an `@token`. This room's tokens:

| Token | Covers |
|---|---|
| | |

Add tokens on the first incident rather than designing the list up front.

**Collision:** earlier timestamp wins; on a tie, the alphabetically lower callsign wins.
No negotiation. If you lost, `UNLOCK` at once.

---

## 3b. Profile — delete this section unless the room shares a git tree

| Token | Covers |
|---|---|
| `@git` | staging, committing, branch operations |
| `@tests` | running the test suite |
| `@build` | build scripts and output directories |
| `@env` | installing packages, changing the environment |

- Explicit pathspec on every commit, plus a separate audit of the staged list before it.
- A rename needs both paths named.
- Never `git stash`, `git add .`, `-A`, or a sweeping checkout in this tree. For a baseline,
  read the committed version out of the tree: `git show HEAD:path > ../baseline/file`.
- Never revert, stash or sweep another session's uncommitted or untracked work.
- Snapshot this file outside the tree before taking `@git`.

---

## 4. Standing rules

- Claims carry evidence a reader can re-run. Opinions are labelled as opinions.
- Count constructs, not search hits. A match you have not opened is not evidence.
- A count carries its method and its date.
- Written down, then archive, then measure — in that order.
- A hazard carries a date and is re-tested before anyone is briefed on it.
- At handover time every finding is `BLOCKS` or `SHIPS WITH A NOTE`. There is no third.
- Unlanded work you did not create belongs to someone else. Leave it, `ASK` the owner,
  `BLOCK` if nobody answers.
- Nobody pushes or publishes anywhere without an explicit instruction naming the target —
  ask the operator, and the authorisation becomes a line anyone can quote.
- Every ban has a sanctioned alternative; they are tabulated in `PROTOCOL.md § 10`. A ban
  with no alternative is a defect in the rule. Say so instead of routing around it.
- Say whether your silence means dead or waiting. It depends on whether your timer is armed.
- Snapshot this file to a timestamped copy outside the tree on every coordinator wake. A
  copy inside the tree is not a backup.

---

## 5. Journal — append below, newest at the bottom

| HH:MM:SS | FROM | TYPE | TO | TARGETS | TEXT |
|---|---|---|---|---|---|
