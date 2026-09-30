# Unsigned Multiplication and EFLAGS

This folder was tested with `mul1.asm` and `mul2.asm`. Both use the unsigned `MUL` instruction.

For `MUL`, only CF and OF have architecturally defined meanings. They are both clear when the upper half of the product is zero and both set when the upper half is nonzero. The values of SF, ZF, AF, and PF are undefined after `MUL`; any values displayed for those flags by GDB must not be interpreted as properties of the result.

## Building and checking with GDB

For each program, assemble and link with debug information:

```bash
nasm -f elf32 -g -F dwarf mul1.asm -o mul1.o
ld -m elf_i386 mul1.o -o mul1
gdb ./mul1
```

For `mul1.asm`, stop at `_start`, execute the operand load and `MUL`, and inspect AX and EFLAGS:

```text
(gdb) break *_start
(gdb) run
(gdb) stepi 2
(gdb) info registers eax eflags
```

Repeat the same procedure for `mul2.asm`, inspecting both AX and DX because the 16-bit multiplication produces a 32-bit result in DX:AX.

## Program 1: `mul1.asm`

This program multiplies two unsigned bytes:

```text
25 * 10 = 250
AX = 0x00FA
```

| Flag | Status | Explanation |
|---|---|---|
| CF | Clear | The upper half of the product is AH, and AH is `0x00`. The product fits entirely in AL. |
| OF | Clear | OF follows the same rule as CF for `MUL`; the upper half is zero. |
| SF | Undefined | `MUL` does not define SF. |
| ZF | Undefined | `MUL` does not define ZF, even though the product is nonzero. |
| AF | Undefined | `MUL` does not define AF. |
| PF | Undefined | `MUL` does not define PF. |

Defined flags: `CF=0, OF=0`.

## Program 2: `mul2.asm`

This program multiplies two unsigned words:

```text
3000 * 200 = 600000
DX:AX = 0x0009:0x27C0
```

| Flag | Status | Explanation |
|---|---|---|
| CF | Set | The upper half of the product, DX, is `0x0009`, so the full product does not fit in AX alone. |
| OF | Set | OF follows the same rule as CF for `MUL`; the upper half is nonzero. |
| SF | Undefined | `MUL` does not define SF. |
| ZF | Undefined | `MUL` does not define ZF. |
| AF | Undefined | `MUL` does not define AF. |
| PF | Undefined | `MUL` does not define PF. |

Defined flags: `CF=1, OF=1`.

## Conclusion

The two examples show the intended `MUL` flag rule. The 250 result fits in the lower half, clearing CF and OF. The 600000 result needs a nonzero upper half, setting both flags. The other arithmetic flags are undefined rather than meaningfully set or cleared.
