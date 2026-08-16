# The bus

Conformance example: **valid**. Two collisions resolved by
[PROTOCOL.md § 3](../PROTOCOL.md#3-locks) — earlier timestamp wins; on a tie, the
alphabetically lower callsign wins.

The first pair is separated by one second, so the clock decides and BRAVO wins despite
being later in the alphabet. The second pair lands in the same second, so the alphabet
decides. Both losers release immediately and without discussion, which is the point of
having a rule rather than a negotiation.

| HH:MM:SS | FROM | TYPE | TO | TARGETS | TEXT |
|---|---|---|---|---|---|
| 10:02:11 | ALFA | HELLO | * | - | session a1 pid 100 vendor-x model-y timer job 1 silence means dead-or-closed |
| 10:02:12 | BRAVO | HELLO | * | - | session b2 pid 101 vendor-x model-y timer job 2 silence means dead-or-closed |
| 10:02:13 | CHARLIE | HELLO | * | - | session c3 pid 102 vendor-y model-z timer job 3 silence means dead-or-closed |
| 10:05:41 | BRAVO | LOCK | * | docs/handover.md | drafting the operator procedure |
| 10:05:42 | ALFA | LOCK | * | docs/handover.md | same file |
| 10:05:55 | ALFA | UNLOCK | * | docs/handover.md | BRAVO is one second earlier, backing off |
| 10:20:07 | ALFA | LOCK | * | @build | packaging the kit |
| 10:20:07 | CHARLIE | LOCK | * | @build | packaging the kit |
| 10:20:19 | CHARLIE | UNLOCK | * | @build | equal timestamps, ALFA is lower in the alphabet, backing off |
| 10:34:02 | ALFA | UNLOCK | * | @build | kit built |
