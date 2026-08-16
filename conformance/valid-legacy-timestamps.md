# The bus

Conformance example: **valid**. A bus written before v0.2, at minute resolution.

A reader MUST accept `HH:MM` and treat it as `HH:MM:00`
([PROTOCOL.md § 1](../PROTOCOL.md#1-the-line-format)). Note what it costs: the two `LOCK`
lines at 09:16 are indistinguishable in time, so the tiebreak falls through to the
alphabet, and CHARLIE loses a race it may well have won on the clock.

| HH:MM | FROM | TYPE | TO | TARGETS | TEXT |
|---|---|---|---|---|---|
| 09:14 | ALFA | HELLO | * | - | session 7f3a1c pid 41288 vendor-x model-y timer job 12 silence means dead-or-closed |
| 09:15 | CHARLIE | HELLO | * | - | session 91ab pid 41377 vendor-y model-z no timer silence means waiting |
| 09:16 | ALFA | LOCK | * | core/parser.py | rewriting the token scanner |
| 09:16 | CHARLIE | LOCK | * | core/parser.py | same file, I did not see ALFA in time |
| 09:17 | CHARLIE | UNLOCK | * | core/parser.py | ALFA wins on the alphabet at equal timestamps, backing off |
| 09:41 | ALFA | UNLOCK | * | core/parser.py | done |
