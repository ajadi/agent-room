# The bus

Conformance example: **invalid**. The file ends mid-line, as it does when a write is
interrupted or a redirect is cut off.

Breaks [PROTOCOL.md § 1](../PROTOCOL.md#1-the-line-format): the last line has three columns
and no closing pipe. A reader that splits on `|` and takes what it finds will report a
message from `CHARL` with an empty type, or skip it silently — and the second behaviour is
worse, because the room then acts on a bus that is missing its most recent line without
anyone knowing one is missing.

A length check against the last known line count catches this; a grammar check on the final
line catches it faster.

| HH:MM:SS | FROM | TYPE | TO | TARGETS | TEXT |
|---|---|---|---|---|---|
| 10:05:00 | ALFA | HELLO | * | - | session a1 pid 100 vendor-x model-y timer job 1 silence means dead-or-closed |
| 10:06:12 | ALFA | LOCK | * | core/parser.py | reading the scanner |
| 10:31:40 | ALFA | UNLOCK | * | core/parser.py | done, 41 tests green |
| 10:32:05 | CHARL