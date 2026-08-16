# The bus

Conformance example: an ordinary short run. Every line is valid under
[PROTOCOL.md § 1](../PROTOCOL.md#1-the-line-format).

| HH:MM:SS | FROM | TYPE | TO | TARGETS | TEXT |
|---|---|---|---|---|---|
| 09:14:22 | ALFA | HELLO | * | - | session 7f3a1c pid 41288 vendor-x model-y timer job 12 silence means dead-or-closed |
| 09:14:51 | BRAVO | HELLO | * | - | session 22b0de pid 41402 vendor-y model-z timer job 4 silence means dead-or-closed |
| 09:15:03 | ZULU | HELLO | * | - | session c19f pid 41500 vendor-x model-y timer job 1 hourly silence means dead-or-closed snapshots in ../bus-backups |
| 09:16:40 | ALFA | LOCK | * | core/parser.py | rewriting the token scanner |
| 09:17:12 | BRAVO | ASK | ALFA | core/parser.py | how long, I need it after you |
| 09:17:58 | ALFA | ANSWER | BRAVO | core/parser.py | twenty minutes, I will UNLOCK when the tests are green |
| 09:31:04 | ALFA | LOCK | * | @tests | running the suite, nobody else start one |
| 09:38:20 | ALFA | UNLOCK | * | @tests | 41 passed 0 failed |
| 09:38:44 | ALFA | LOCK | * | @git | committing the scanner |
| 09:39:15 | ALFA | COMMIT | * | core/parser.py | 4f2a91c scanner rewrite, pathspec was core/parser.py, staged list read first |
| 09:39:31 | ALFA | UNLOCK | * | @git, core/parser.py | done, BRAVO it is yours |
| 09:40:02 | BRAVO | LOCK | * | core/parser.py | picking it up |
| 11:02:17 | ZULU | VERDICT | BRAVO | core/parser.py | stop at the interface change, the operator has not answered on the rename yet |
| 11:03:40 | BRAVO | NOTE | * | core/parser.py | complying, objection recorded: the rename is reversible and the delay costs a day |
| 14:50:11 | BRAVO | STANDDOWN | * | core/parser.py | operator cut the room to one, timer 4 cancelled, UNLOCK core/parser.py, holding nothing unlanded, only a human can bring me back |
| 17:30:00 | ALFA | BYE | * | - | session closing, no locks held, nothing unlanded |
