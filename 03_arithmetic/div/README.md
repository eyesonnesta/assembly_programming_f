# Unsigned Division and EFLAGS

This folder was tested with `div1.asm` and `div2.asm`. Both use the unsigned `DIV` instruction.

Unlike `ADD` and `SUB`, `DIV` does not define the arithmetic status flags. CF, PF, AF, ZF, SF, and OF are all undefined after a successful division. GDB may display a value for each flag, but those displayed values are not guaranteed results of `DIV` and must not be used to describe whether the quotient is zero, positive, negative, even, or overflowing.

## Building and checking with GDB

For each program, assemble and link with debug information:

```bash
nasm -f elf32 -g -F dwarf div1.asm -o div1.o
ld -m elf_i386 div1.o -o div1
gdb ./div1
```

For `div1.asm`, stop at `_start`, execute the two operand loads and the division, and inspect AX and EFLAGS:

```text
(gdb) break *_start
(gdb) run
(gdb) stepi 3
(gdb) info registers eax eflags
```

For `div2.asm`, execute four instructions from `_start` before inspecting AX, DX, and EFLAGS because it loads three registers before `DIV`:

```text
(gdb) break *_start
(gdb) run
(gdb) stepi 4
(gdb) info registers eax edx eflags
```

Inspection must occur before `xor ebx, ebx`, which would define a new set of flags unrelated to the division.

## Program 1: `div1.asm`

This program divides a 16-bit dividend by an 8-bit divisor:

```text
100 / 7 = 14 remainder 2
AL = 0x0E
AH = 0x02
```

The quotient 14 fits in AL, so the division succeeds without a divide error.

| Flag | Status after `DIV` | Explanation |
|---|---|---|
| CF | Undefined | `DIV` does not define CF. |
| PF | Undefined | `DIV` does not define PF. |
| AF | Undefined | `DIV` does not define AF. |
| ZF | Undefined | `DIV` does not define ZF; it cannot be used to test whether the quotient is zero. |
| SF | Undefined | `DIV` does not define SF. |
| OF | Undefined | Quotient overflow is reported with a divide-error exception, not OF. |

## Program 2: `div2.asm`

This program divides the unsigned 32-bit value in DX:AX by a 16-bit divisor:

```text
50000 / 300 = 166 remainder 200
AX = 0x00A6
DX = 0x00C8
```

The quotient 166 fits in AX, so the division completes successfully.

| Flag | Status after `DIV` | Explanation |
|---|---|---|
| CF | Undefined | `DIV` does not define CF. |
| PF | Undefined | `DIV` does not define PF. |
| AF | Undefined | `DIV` does not define AF. |
| ZF | Undefined | `DIV` does not define ZF, regardless of the nonzero quotient. |
| SF | Undefined | `DIV` does not define SF. |
| OF | Undefined | A quotient too large for AX would cause a divide-error exception rather than setting OF. |

## Conclusion

The register results can be explained exactly, but no arithmetic EFLAGS status can be inferred from either division. Treating the raw flags printed by GDB as defined results would be incorrect.
