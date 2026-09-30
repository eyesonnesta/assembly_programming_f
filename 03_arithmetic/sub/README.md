# Subtraction and EFLAGS

This folder was tested with `sub1.asm` and `sub2.asm`. The analysis focuses on the arithmetic status flags changed by `SUB`: Carry (CF), Parity (PF), Auxiliary Carry (AF), Zero (ZF), Sign (SF), and Overflow (OF).

## Building and checking with GDB

For each program, assemble and link with debug information:

```bash
nasm -f elf32 -g -F dwarf sub1.asm -o sub1.o
ld -m elf_i386 sub1.o -o sub1
gdb ./sub1
```

Inside GDB, stop at `_start`, run the load and subtraction, and inspect the state immediately after `SUB`:

```text
(gdb) break *_start
(gdb) run
(gdb) stepi 2
(gdb) info registers eax eflags
```

Repeat with `sub2.asm`. The later `xor ebx, ebx` must not execute before the flags are inspected because it would replace the flags produced by `SUB`.

## Program 1: `sub1.asm`

The program performs an 8-bit subtraction:

```text
50 - 80 = -30
0x32 - 0x50 = 0xE2
```

The wrapped 8-bit result in AL is `0xE2`, which represents -30 as a signed byte and 226 as an unsigned byte.

| Flag | Status | Explanation |
|---|---|---|
| CF | Set | In subtraction, CF indicates an unsigned borrow. Since 50 is below 80, a borrow is required. |
| PF | Set | The low byte `0xE2` contains four 1 bits, which is even parity. |
| AF | Clear | The low nibble subtraction `0x2 - 0x0` does not require a borrow from bit 4. |
| ZF | Clear | The result `0xE2` is not zero. |
| SF | Set | Bit 7 of the 8-bit result is 1, matching the negative signed result. |
| OF | Clear | Both operands are positive signed values and -30 is representable in an 8-bit signed value. |

Observed arithmetic flags: `CF=1, PF=1, AF=0, ZF=0, SF=1, OF=0`.

## Program 2: `sub2.asm`

The program performs a 16-bit subtraction:

```text
1000 - 2000 = -1000
0x03E8 - 0x07D0 = 0xFC18
```

The wrapped 16-bit result in AX is `0xFC18`, which represents -1000 as a signed word.

| Flag | Status | Explanation |
|---|---|---|
| CF | Set | An unsigned borrow is required because 1000 is below 2000. |
| PF | Set | PF examines the low byte `0x18`, which contains two 1 bits and therefore has even parity. |
| AF | Clear | The low nibble subtraction `0x8 - 0x0` does not require a borrow from bit 4. |
| ZF | Clear | The result `0xFC18` is not zero. |
| SF | Set | Bit 15 of the result is 1, so the signed result is negative. |
| OF | Clear | The signed result -1000 fits in the 16-bit signed range. |

Observed arithmetic flags: `CF=1, PF=1, AF=0, ZF=0, SF=1, OF=0`.

## Conclusion

Both examples require an unsigned borrow, so CF is set. Neither has signed overflow: their negative mathematical results are representable at the selected operand size, so OF remains clear.
