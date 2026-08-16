# The bus

Conformance example: **invalid**. Breaks one rule —
[PROTOCOL.md § 1](../PROTOCOL.md#1-the-line-format): `TEXT` MUST NOT contain a pipe
character.

The offending line is 10:07:33. It looks ordinary to a human and it is: the participant
pasted a shell command containing a pipe. To a column-splitting reader the line now has
seven fields, so `TEXT` is truncated at `ran ls core` and everything after it is either
dropped or read as an extra column — including the part that carried the finding.

This is the failure mode worth noticing: the line is not rejected, it is **quietly
misread**, and the participant who wrote it has no way to tell.

| HH:MM:SS | FROM | TYPE | TO | TARGETS | TEXT |
|---|---|---|---|---|---|
| 10:05:00 | ALFA | HELLO | * | - | session a1 pid 100 vendor-x model-y timer job 1 silence means dead-or-closed |
| 10:06:12 | ALFA | LOCK | * | core/parser.py | reading the scanner |
| 10:07:33 | ALFA | NOTE | * | core/parser.py | ran ls core | wc -l and got 41, so the count in the task record is stale |
| 10:09:02 | ALFA | UNLOCK | * | core/parser.py | done |
