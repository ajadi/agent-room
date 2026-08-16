# Profile: coding in a shared git tree

Several sessions in one working tree, with one git index between them.

A profile adds to [the core protocol](../PROTOCOL.md); it does not replace any of it. The
core applies in full — the bus, the line format, locks, timers, the operator, and the bans
listed there. What a profile supplies is the part the core cannot know: which resources
this domain has, which operations on them are dangerous, and what to do instead.

**This is the only profile with evidence behind it.** Every rule below comes from an
incident in [FIELD-NOTES.md](../FIELD-NOTES.md) — three days, one room, one codebase. It is
also where the whole protocol came from, which is why the core spent a version with these
rules inside it.

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

- **Explicit pathspec on every commit, and a separate audit call before it.** Run the
  staged-file listing, read it, unstage anything that is not yours, then commit. The
  pathspec is the mechanism — it does not need vigilance. The audit is the check — it
  catches what the pathspec cannot.
- **A rename needs both paths named.** A pathspec commit does not carry the staged
  deletion, so the old path stays staged for whoever commits next.
- Never `git stash`, never `git add .` or `-A`, never a sweeping `git checkout --`. To get
  a baseline for comparison, read the committed version into a temporary directory instead.
  Two of our participants independently reached for `stash` within half an hour, for the
  same reason, and one of them destroyed the bus doing it.
- Reverting **a single path you own** with `git restore` is correct and is not the banned
  sweeping form. Reading a committed blob over the file instead can leave it looking
  modified on a repository with line-ending normalisation, which reads as a failed revert.
- **Take a commit hash from the output of the command that made it**, never from a later
  log read. In a shared tree the top of `git log` is not necessarily yours.

Uncommitted or untracked work you did not create is the git form of the core rule that
work you hold is yours and nobody else's ([core § 3](../PROTOCOL.md#3-locks)): never revert
it, never stash it, never sweep it into your commit. `ASK` its owner; if nobody answers,
`BLOCK` and let the coordinator rule.

---

## 3. Snapshot the bus before taking `@git`

Snapshots are a core rule ([core § 5](../PROTOCOL.md#5-keeping-the-bus)) — what belongs here
is the trigger, because `@git` is a resource of this domain.

Snapshot first, then take the lock. Every loss of the bus we have recorded came from a git
operation in the shared tree: one `git stash` truncated it, and the file is often untracked,
so git could not restore what git had destroyed.

```sh
mkdir -p ../bus-backups
cp Busfile.md "../bus-backups/Busfile.$(date +%Y%m%d-%H%M%S).md"
```

---

## 4. Bans and their sanctioned alternatives

These extend the core table ([core § 10](../PROTOCOL.md#10-bans-and-their-sanctioned-alternatives)),
which continues to apply.

| Do not | Because | Do this instead |
|---|---|---|
| `git stash` | It clobbers untracked files, the bus among them | Snapshot the bus, then read the committed version out of the tree: `git show HEAD:path > ../baseline/file` |
| `git add .` or `-A` | Sweeps in other sessions' staged work | Explicit pathspec, plus `git diff --cached --name-only` read before every commit |
| A sweeping `git checkout -- .` | Same, destructively | `git restore <one path you own>` — the single-path form is not the banned one |
| Rewrite history to undo a bad commit | Other sessions have already read and built on it | Annotate: a `NOTE` naming what was swept and by whom, then a follow-up commit |
| Take a commit hash from `git log` | In a shared tree the top of the log is not necessarily yours | Take it from the output of the command that made it |

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

What a profile must not do: contradict the core, require a tool in order to participate, or
restate core rules in its own words. Three copies of one caveat rot at different rates and
then disagree.
