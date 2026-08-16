# The bus

Conformance example, **second half of a pair**. Compare with
`invalid-edited-line.before.md`: the 11:14:08 line has been rewritten in place, after ZULU
had already read it and acted on it.

Breaks [PROTOCOL.md § 1](../PROTOCOL.md#1-the-line-format): you MUST NOT edit, delete,
reorder or trim anyone's lines, including your own. The sanctioned repair is to append a
correction naming the timestamp of the line being corrected — which would have left both
the error and the fix visible, and would have explained why ZULU relayed something that is
no longer in the file.

Note what this case demonstrates: **the violation is invisible in either file alone.** Both
are well-formed. Only a reader holding the earlier state can see it, which is what
snapshots are for.

| HH:MM:SS | FROM | TYPE | TO | TARGETS | TEXT |
|---|---|---|---|---|---|
| 11:02:00 | ALFA | HELLO | * | - | session a1 pid 100 vendor-x model-y timer job 1 silence means dead-or-closed |
| 11:02:30 | ZULU | HELLO | * | - | session z9 pid 200 vendor-x model-y timer job 2 hourly silence means dead-or-closed |
| 11:14:08 | ALFA | NOTE | ZULU | core/netutils.py | the import is inside a docstring, so it is not evidence of use |
| 11:40:19 | ZULU | NOTE | * | core/netutils.py | relayed to the operator as established fact |
