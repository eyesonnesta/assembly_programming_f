# Addition and EFLAGS

This folder was tested with `add1.asm` and `add2.asm`. The analysis focuses on the arithmetic status flags changed by `ADD`: Carry (CF), Parity (PF), Auxiliary Carry (AF), Zero (ZF), Sign (SF), and Overflow (OF).

## Building and checking with GDB

For each program, assemble and link with debug information:

```bash
nasm -f elf32 -g -F dwarf add1.asm -o add1.o
ld -m elf_i386 add1.o -o add1
gdb ./add1
```

Inside GDB, stop at `_start`, run the two instructions that load the first operand and perform the addition, and then inspect the result and flags:

```text
(gdb) break *_start
(gdb) run
(gdb) stepi 2
(gdb) info registers eax eflags
```

Repeat with `add2.asm`. The inspection must happen before the later `xor ebx, ebx`, because `XOR` changes the arithmetic flags.

## Program 1: `add1.asm`

The program performs an 8-bit addition:

```text
120 + 10 = 130
0x78 + 0x0A = 0x82
```

The result in AL is `0x82` (`10000010b`). As an unsigned byte, this is 130. As a signed byte, the same bit pattern represents -126.

| Flag | Status | Explanation |
|---|---|---|
| CF | Clear | The unsigned result 130 fits in 8 bits, so there is no carry out of bit 7. |
| PF | Set | The low byte `0x82` contains two 1 bits, which is even parity. |
| AF | Set | The low nibbles, `0x8 + 0xA`, produce a carry from bit 3 to bit 4. |
| ZF | Clear | The result `0x82` is not zero. |
| SF | Set | Bit 7 of the 8-bit result is 1, so the result is negative when interpreted as signed. |
| OF | Set | Two positive signed operands produced a result with the sign bit set. The signed sum 130 is outside the range -128 to 127. |

Observed arithmetic flags: `CF=0, PF=1, AF=1, ZF=0, SF=1, OF=1`.

## Program 2: `add2.asm`

The program performs a 16-bit addition:

```text
32000 + 500 = 32500
0x7D00 + 0x01F4 = 0x7EF4
```

The result in AX is `0x7EF4`.

| Flag | Status | Explanation |
|---|---|---|
| CF | Clear | The unsigned result 32500 is below 65536, so there is no carry out of bit 15. |
| PF | Clear | PF uses only the low byte. `0xF4` contains five 1 bits, which is odd parity. |
| AF | Clear | The low nibbles, `0x0 + 0x4`, do not carry from bit 3 to bit 4. |
| ZF | Clear | The result `0x7EF4` is not zero. |
| SF | Clear | Bit 15 of the 16-bit result is 0. |
| OF | Clear | The signed result 32500 is within the 16-bit signed range of -32768 to 32767. |

Observed arithmetic flags: `CF=0, PF=0, AF=0, ZF=0, SF=0, OF=0`.

## Conclusion

`add1.asm` demonstrates signed overflow without unsigned carry: OF is set while CF is clear. `add2.asm` produces a result that fits in both the unsigned and signed 16-bit ranges, so both CF and OF are clear.
