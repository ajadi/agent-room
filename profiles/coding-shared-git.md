# Profile: coding in a shared git tree

Several sessions in one working tree, with one git index between them.

A profile adds to [the core protocol](../PROTOCOL.md); it does not replace any of it. The
core applies in full — the bus, the line format, locks, timers, the operator, and the bans
listed there. What a profile supplies is the part the core cannot know: which resources
this domain has, which operations on them are dangerous, and what to do instead.

**This is the only profile with evidence behind it.** Every rule below comes from an
incident in [FIELD-NOTES.md](../FIELD-NOTES.md) — two rooms now: the three days the whole
protocol came from, which is why the core spent a version with these rules inside it, and
the second room's five and a half hours, which supplied §§ 5–6 and re-scoped § 3.

---

## 1. Resources

Pseudo-resource tokens for this domain. Locked exactly like files, by the same rules
([core § 3](../PROTOCOL.md#3-locks)):

| Token | Covers |
|---|---|
| `@git` | staging, committing, branch operations |
| `@tests` | running the test suite (memory: never two at once) |
| `@build` | build scripts and output directories |
| `@env` | installing packages, changing the virtualenv |
| `@hardware` | physical devices, serial ports, benches |

Derive additions from the question *"what does this mutate outside my locked paths?"* and
extend the list on the first incident rather than designing it up front.

---

## 2. Shared git

One index for several sessions is the sharpest edge in the room.

Marked with the same [requirement language](../PROTOCOL.md#requirement-language) as the
core.

- **Explicit pathspec on every commit, and a separate audit call before it.** You MUST name
  a pathspec, and MUST read the staged-file listing before committing, unstaging anything
  that is not yours. The pathspec is the mechanism — it does not need vigilance. The audit
  is the check — it catches what the pathspec cannot.
- **A rename needs both paths named:** you MUST name both. A pathspec commit does not carry
  the staged deletion, so the old path stays staged for whoever commits next.
- You MUST NOT use `git stash`, `git add .` or `-A`, or a sweeping `git checkout --`. To get
  a baseline for comparison, read the committed version into a temporary directory instead.
  Two of our participants independently reached for `stash` within half an hour, for the
  same reason, and one of them destroyed the bus doing it.
- You MAY revert **a single path you own** with `git restore`; that is not the banned
  sweeping form. Reading a committed blob over the file instead can leave it looking
  modified on a repository with line-ending normalisation, which reads as a failed revert.
- **Take a commit hash from the output of the command that made it.** You MUST NOT take it
  from a later log read: in a shared tree the top of `git log` is not necessarily yours.

Uncommitted or untracked work you did not create is the git form of the core rule that work
someone holds is theirs ([core § 3](../PROTOCOL.md#3-locks)). You MUST NOT revert it, stash
it, or sweep it into your commit. `ASK` its owner; if nobody answers, `BLOCK` and let the
coordinator rule.

---

## 3. A current snapshot must exist before `@git` is taken

Snapshots are a core rule ([core § 5](../PROTOCOL.md#5-keeping-the-bus)) — what belongs here
is the trigger, because `@git` is a resource of this domain.

Before taking the lock, a current snapshot MUST exist — **whoever took it**. Usually that is
you: snapshot first, then lock. But the duty is on the existence, not the committer
personally; the per-committer wording blocked the one participant that could not write
outside the tree
([FIELD-NOTES.md](../FIELD-NOTES.md#every-rule-the-coordinator-wrote-was-too-wide)). Every
loss of the bus we have recorded came from a git operation in the shared tree: one
`git stash` truncated it, and the file is often untracked, so git could not restore what git
had destroyed.

```sh
mkdir -p ../bus-backups
cp Busfile.md "../bus-backups/Busfile.$(date +%Y%m%d-%H%M%S).md"
```

---

## 4. Bans and their sanctioned alternatives

These extend the core table ([core § 10](../PROTOCOL.md#10-bans-and-their-sanctioned-alternatives)),
which continues to apply. As there: every left-hand cell is a MUST NOT, every right-hand
cell is the sanctioned route to the same goal.

| Do not | Because | Do this instead |
|---|---|---|
| `git stash` | It clobbers untracked files, the bus among them | Snapshot the bus, then read the committed version out of the tree: `git show HEAD:path > ../baseline/file` |
| `git add .` or `-A` | Sweeps in other sessions' staged work | Explicit pathspec, plus `git diff --cached --name-only` read before every commit |
| A sweeping `git checkout -- .` | Same, destructively | `git restore <one path you own>` — the single-path form is not the banned one |
| Rewrite history to undo a bad commit | Other sessions have already read and built on it | Annotate: a `NOTE` naming what was swept and by whom, then a follow-up commit |
| Take a commit hash from `git log` | In a shared tree the top of the log is not necessarily yours | Take it from the output of the command that made it |
| Disable a host framework's enforcement hook | It is someone else's guard, and you cannot see everything it protects | Relocate your writes into a directory the guard permits, and report the block (§ 5) |
| Delete another framework's runtime state, however stale | Shared state you did not create is not yours to clear | Report it with the measurement that shows it stale, and route around it |

---

## 5. Host frameworks and hooks

New in v0.3. A repository can carry an agent framework of its own: hooks that gate writes,
runtime state that goes stale, a second task registry beside the room's. Ours did — nominally
switched off, and not off enough. A write-guard hook denied every write under the room's
directory for eighteen minutes, on the strength of a marker file twenty hours stale against
a thirty-minute TTL; propose–critique was structurally impossible and nobody could see why,
because the hook printed its reason where neither worker read it
([FIELD-NOTES.md](../FIELD-NOTES.md#a-framework-that-fought-the-room)).

- **Diagnose a hook by running it, not by reading it.** Feed it candidate paths, and include
  a control that must be denied — an instrument that cannot fail is not an instrument
  ([REGIMEN.md § 2](../REGIMEN.md#2-what-counts-as-evidence)). Reading the hook tells you
  what it was meant to do; the run tells you what it does.
- **The sanctioned fix is relocation** into a directory the guard permits. You MUST NOT
  disable someone else's enforcement hook and MUST NOT delete someone else's runtime state,
  however stale — both levers were available in our incident, both workers refused them, and
  relocation ended the block with no guard disabled, no state deleted, no hook patched.
- **Two task registries in one tree need an explicit bridge** — a stated mapping between the
  room's lanes and the framework's rows. Without one, finished work has no row to close: a
  full day of lanes closed zero queue tasks in part because nothing the room finished had a
  row anywhere.

---

## 6. Branchless review

New in v0.3. Some rooms run without branches: finished work goes to a reviewer first-come
first-served, approved means committed, rejected returns to the queue at high priority. The
model works — ours ran three full cycles to commit — but it changes what *finished* means,
and two rules follow
([FIELD-NOTES.md](../FIELD-NOTES.md#two-workflow-redesigns-in-one-afternoon)):

- **Locks persist through the review.** Finished-but-unreviewed work sits uncommitted in the
  shared tree, so a lock is released on the verdict — approve-and-commit, or rejection — not
  when the typing stops. Releasing earlier tells the room the paths are free while the only
  copy of the work still sits in them.
- **The diff is copied outside the tree until the verdict.** Unreviewed work in a shared
  tree has no other copy anywhere. One participant adopted this unprompted before it was a
  rule, which is the usual sign of a rule worth writing down.

---

## Writing a profile

A profile is four things: the resources of a domain, the rules for its sharpest edges, the
bans with their sanctioned alternatives, and the field notes they came from.

The last one is not decoration. A profile written from imagination would be exactly the
kind of unfounded, fluent text this protocol exists to resist, and it would be indexed and
quoted as if it were experience. So: **run a room in your domain first, then write the
profile from what broke.** A partial profile with two rules and a real incident behind each
is worth more than a complete-looking one with none.

Domains that would obviously benefit and have nobody's evidence yet: shared databases and
migrations, hardware benches, document production, incident response, data pipelines. If
you run one, the **vendor mix** and **failure report** templates take the raw material, and
a profile can be assembled from it afterwards.

A profile MUST NOT contradict the core, MUST NOT require a tool in order to participate
([core: conformance for tools](../PROTOCOL.md#conformance-for-tools)), and SHOULD NOT
restate core rules in its own words. Three copies of one caveat rot at different rates and
then disagree.
