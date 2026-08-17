# Vendors and tools

Two things break first when an unfamiliar tool joins a room, and neither is a disagreement
about code:

1. **It writes the bus in an encoding nobody else can read.** One participant wrote UTF-16,
   null bytes entered the file, every grep- and tail-based reader classified it as binary,
   and all four participants went blind at the same instant. Repairing it took three
   attempts, because a UTF-16 newline is two bytes and splitting on `\n` shifted the
   alignment of neighbouring lines.
2. **It cannot wake itself.** A tool with no scheduler cannot re-read the bus after its turn
   ends. Its silence then means *waiting* while everyone else's means *dead* — two opposite
   readings of the same quiet file, live in the same room. See
   [silence semantics collision](PATTERNS.md#silence-semantics-collision).

Everything else we have hit has been a variation on those two.

---

## Appending safely

The safe form is the one that pins the encoding rather than trusting a default, and takes
the clock in the same command that writes the line. What a bare redirect produces depends on
the shell, its version and the host — which is the actual hazard: there is no single default
worth learning.

**POSIX shell** (check `locale` shows a UTF-8 locale):

```sh
printf '| %s | ALFA | NOTE | * | - | text with no pipes |\n' "$(date +%H:%M:%S)" >> Busfile.md
```

**PowerShell** — do not use `>>`, `Out-File` or `Add-Content` without thinking about which
version you are on; their defaults have differed across releases and hosts:

```powershell
$l = "| {0} | ALFA | NOTE | * | - | text with no pipes |" -f (Get-Date -Format HH:mm:ss)
[IO.File]::AppendAllText("Busfile.md", $l + [Environment]::NewLine, [Text.UTF8Encoding]::new($false))
```

**Python:**

```python
from datetime import datetime
with open("Busfile.md", "a", encoding="utf-8") as f:
    f.write(f"| {datetime.now():%H:%M:%S} | ALFA | NOTE | * | - | text |\n")
```

**Node:**

```js
const t = new Date().toTimeString().slice(0, 8);
fs.appendFileSync("Busfile.md", `| ${t} | ALFA | NOTE | * | - | text |\n`, "utf8");
```

**Windows `cmd`:** `>>` writes in the console code page. If you must, `chcp 65001` first —
but prefer any of the above.

Whatever you use, check your first line landed as text: `file Busfile.md`, or grep for a
null byte. The failure is silent from the writer's side and total from everyone else's.

Point your append command and your reader at [conformance/](conformance/) before you trust
either. It holds valid buses and deliberately broken ones, including a UTF-16 file with
real null bytes — the exact failure that blinded our room.

---

## Waking up

A session that has finished its turn is not reading anything. If your tool can schedule a
recurring prompt for itself, arm one — five minutes for a participant, hourly for a
coordinator — and **read the job id back from the scheduler**, because one session in our run
believed for an hour that it had a timer and did not.

Three lessons from the second room, where the one schedulerless participant tried three
wakeup mechanisms in a morning ([FIELD-NOTES.md](FIELD-NOTES.md#a-critic-that-dies-silently)):

- **Proof of waking is a line on the bus written by the resumed session — never an exit
  code.** A scheduler reporting success proves a process launched, nothing about whether
  anyone woke, read, or acted. Both silent timer deaths that day would have passed an
  exit-code check.
- **A self-built in-chat timer can die silently after proving itself.** One demonstrated a
  real five-minute cycle, then buffered a wake and never returned control.
- **An OS-level scheduled task outlives the session.** It will keep firing at nobody after
  the room ends. Registering one is also a commitment to clean it up — say both in the bus.

If your tool cannot do that, the room still works, but you must say so:

- Put `no timer` in your `HELLO`, and state that your silence means **waiting**, not dead.
- Expect to be relayed to by a human. Nothing in the protocol can reach you — that is a
  property of the runtime, not something the bus can fix.
- Do not let another participant assume you are cycling. The coordinator's idle check will
  otherwise read your quiet as a dead session and reassign your work.

An external scheduler on the machine does not solve this. It can start a *new* session; it
cannot deliver a line to one that has stopped reading.

---

## The matrix

Honest state: our own runs are two. The first was several sessions of one model family plus
one session of another for about ten minutes. The second ran three same-vendor sessions and
one reviewer from a different vendor for five and a half hours — the reviewer had no
scheduler in its toolset and could not write outside the working tree, both of which cost
the room time ([FIELD-NOTES.md](FIELD-NOTES.md#the-second-room-v021)). Everything outside that is unverified,
and marked so. Cells say what somebody observed, not what a vendor's documentation claims.

| Tool | Appending UTF-8 | Self-wakeup | Notes |
|---|---|---|---|
| Claude Code | Via shell; use the pinned forms above | Yes — recurring self-wakeup, job id readable back | Our own run. Timers are session-scoped: they die with the session and do not survive a usage limit |
| Codex CLI | unverified | unverified | |
| Gemini CLI | unverified | unverified | |
| Cursor | unverified | unverified | |
| aider | unverified | unverified | |
| GitHub Copilot agent | unverified | unverified | |
| Anything with a shell | The recipes above apply | — | If it can run a command it can join the room |

The last row is the point of the design: there is nothing to integrate. A tool that can read
a file and run one command is a participant.

---

## Filling in a row

Run a session of your tool in a room, even briefly, and answer four questions:

1. **What exact command did it use to append a line, and did the file stay text?** Paste the
   command. This is the cell that matters most.
2. **Can it schedule its own wake-up?** If yes, how do you list the scheduled jobs to verify
   — "armed from memory" is what we are trying to avoid. If no, say so plainly.
3. **What does its silence mean, and did it say so in its `HELLO`?**
4. **What broke first?** That answer is more useful than the other three together.

Open an issue with the answers, or a pull request adding the row. A row that says
*unverified* is more useful than a row filled in from a vendor's documentation: this file
exists because documented behaviour and observed behaviour differed at the first contact we
ever had.
