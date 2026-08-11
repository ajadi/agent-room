# The bus

Shared coordination file for independent AI sessions working in this repository.
Copy this file to `Busfile.md` in the repository root.

**Append only.** Write with `>>`, never `>`. UTF-8 only. Never edit, delete, reorder or
trim another participant's lines, including your own past lines.

Full rules: https://github.com/ajadi/agent-room/blob/main/PROTOCOL.md

---

## 1. Participants

| Callsign | Session id | Vendor / model | Timer | Notes |
|---|---|---|---|---|
| ALFA | | | | |
| BRAVO | | | | |
| CHARLIE | | | | |
| ZULU | | | | coordinator |

Callsigns are deliberately not model names. Vendor and model are recorded once, on entry,
for diagnosis — not to weight anyone's vote.

---

## 2. Line format

```
| HH:MM | FROM | TYPE | TO | TARGETS | TEXT |
```

Timestamp from a real clock call in the same command. No pipe characters inside TEXT.

Types: `HELLO` `BEAT` `CLAIM` `LOCK` `UNLOCK` `COMMIT` `ASK` `ANSWER` `NOTE` `BLOCK`
`VERDICT` (coordinator only) `BYE`.

---

## 3. Locks

Announce a `LOCK` before writing anything to disk. Reads are free. Name files, never bare
directories. Release with `UNLOCK` — releasing is part of finishing, not an afterthought.

Pseudo-resources in this repository:

| Token | Covers |
|---|---|
| `@git` | staging, committing, branch operations |
| `@tests` | running the test suite |
| `@build` | build scripts and output directories |
| `@env` | installing packages, changing the environment |

Add tokens on the first incident rather than designing the list up front.

**Collision:** earlier timestamp wins; on a tie, the alphabetically lower callsign wins.
No negotiation. If you lost, `UNLOCK` at once.

---

## 4. Standing rules

- Claims carry evidence a reader can re-run. Opinions are labelled as opinions.
- Count constructs, not search hits. A match you have not opened is not evidence.
- A count carries its method and its date.
- Explicit pathspec on every commit, plus a separate audit of the staged list before it.
- A rename needs both paths named.
- Never `git stash`, `git add .`, `-A`, or a sweeping checkout in this tree.
- Uncommitted or untracked work you did not create belongs to someone else. Leave it.
- Nobody pushes or publishes anywhere without an explicit instruction naming the target.
- Say whether your silence means dead or waiting. It depends on whether your timer is armed.

---

## 5. Journal — append below, newest at the bottom

| HH:MM | FROM | TYPE | TO | TARGETS | TEXT |
|---|---|---|---|---|---|
