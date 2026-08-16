# conformance

Worked examples of buses — valid ones and deliberately broken ones — for anyone writing a
validator, an append helper, or a check of their own.

**These are data, not a test suite.** There is no runner here and there will not be one:
requiring a tool in order to participate is the one thing this protocol rules out. Point
whatever you have written at these files and see whether it says what the table says.

They also exist for a rule in [REGIMEN.md § 2](../REGIMEN.md#2-what-counts-as-evidence):
*test your instruments on input you know is bad.* Our own bus-integrity check reported null
bytes in every line of a clean file, because its pattern matched everything, and nobody
noticed until the output looked odd. A check that has never seen a failure is not known to
be able to produce one.

---

## The files

| File | Verdict | What it exercises |
|---|---|---|
| [`valid-minimal.md`](valid-minimal.md) | valid | An ordinary short run: two participants greet, one locks and releases, an `ASK` is answered, a commit lands, one stands down, one leaves |
| [`valid-legacy-timestamps.md`](valid-legacy-timestamps.md) | valid | Pre-v0.2 `HH:MM` timestamps, which a reader must accept as `HH:MM:00` |
| [`valid-tiebreak.md`](valid-tiebreak.md) | valid | Two locks on one target in the same second; the alphabetical tiebreak decides and the loser releases |
| [`invalid-pipe-in-text.md`](invalid-pipe-in-text.md) | invalid | A pipe inside `TEXT`, which silently creates a seventh column |
| [`invalid-truncated.md`](invalid-truncated.md) | invalid | The file ends mid-line, as it does when a write is interrupted |
| [`invalid-utf16.md.bin`](invalid-utf16.md.bin) | invalid | UTF-16 with null bytes — the failure that blinded our whole room at once |
| [`invalid-edited-line.before.md`](invalid-edited-line.before.md) · [`invalid-edited-line.after.md`](invalid-edited-line.after.md) | invalid | Someone edited a line already written. Only visible as a pair — see below |

---

## Two of them need explanation

**`invalid-utf16.md.bin`** carries the `.bin` extension because it really does contain null
bytes, and a repository full of tools that treat such a file as binary is the point. Git
will not diff it, most editors will warn, and `grep` will call it binary — which is exactly
what happened to the room. Rename it to `.md` if your validator insists on the extension.

**The edited-line pair** is the interesting case: no single file can show the violation. The
`before` and `after` files are byte-identical except that one line's `TEXT` has been
rewritten in place. Each is individually well-formed. What is broken is the relationship
between them, and only a reader who saw the earlier state can tell.

That is a real limit and worth stating: **append-only is not verifiable from one snapshot of
the file.** A validator can check the grammar, the types and the lock discipline; it cannot
tell you nobody has been rewriting history unless it kept a copy — which is one more reason
for the snapshots in [PROTOCOL.md § 5](../PROTOCOL.md#5-keeping-the-bus).

---

## Adding a case

Send one with any [failure report](../CONTRIBUTING.md). A file that broke your reader is
worth more than a file we invented, because we can only invent the failures we already know
about — and the two most expensive ones we have had, an encoding and a truncation, are both
in here precisely because we did not think of them in advance.

One rule: each invalid file MUST break exactly one named rule, and the table above MUST say
which. A file that is broken in three ways teaches nothing about any of them.
