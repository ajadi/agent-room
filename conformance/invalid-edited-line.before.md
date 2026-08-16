# The bus

Conformance example, **first half of a pair**. This file is valid on its own. Compare it
with `invalid-edited-line.after.md`, where the 11:14:08 line has been rewritten in place.

| HH:MM:SS | FROM | TYPE | TO | TARGETS | TEXT |
|---|---|---|---|---|---|
| 11:02:00 | ALFA | HELLO | * | - | session a1 pid 100 vendor-x model-y timer job 1 silence means dead-or-closed |
| 11:02:30 | ZULU | HELLO | * | - | session z9 pid 200 vendor-x model-y timer job 2 hourly silence means dead-or-closed |
| 11:14:08 | ALFA | NOTE | ZULU | core/netutils.py | core.netutils is imported by the plugin loader, so it is in use |
| 11:40:19 | ZULU | NOTE | * | core/netutils.py | relayed to the operator as established fact |
