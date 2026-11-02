# Points to Note

## Key concepts

1. NASM macros are expanded before execution; they are not runtime functions.
2. `%macro` and `%endmacro` define a macro.
3. The number after a macro name specifies how many arguments it expects.
4. `%1`, `%2`, and later numbers refer to the supplied arguments.
5. A zero-parameter macro is declared with `%macro NAME 0`.
6. Labels written as `%%label` are local to one macro expansion.
7. `%rep` repeats instructions while the source is being assembled.
8. A macro can invoke another macro, as demonstrated by `PRINT_LINE`.
9. Procedures use `call` and `ret`; `call` places the return address on the stack.
10. Linux x86-64 system calls use `RAX` for the syscall number and registers such as `RDI`, `RSI`, and `RDX` for arguments.

## Program-specific observations

- `m2.asm` handles unsigned, non-negative results only. Its number-conversion procedure does not add a minus sign for negative values.
- Division uses `RDX:RAX` as the dividend, so `RDX` must be cleared before `div`.
- `m3.asm` correctly uses local `%%zero` and `%%done` labels because `CHECK_ZERO` is expanded three times.
- `m4.asm` creates five copies of the print instructions. A runtime loop would usually produce a smaller executable.
- `m5.asm` changes `RAX` to 13 and `RBX` to 18, but the later write system call overwrites `RAX` and does not display those values.
- `macros.asm` uses the ordinary label `loop_delay`. It works because `delay` is expanded once, but it should use `%%loop_delay` if the macro may be invoked more than once.
- The delay examples are busy-wait loops. Their duration depends on processor speed and they consume CPU time while waiting.
- A real application would normally use a timer or sleep system call instead of a large busy-wait loop.

## Questions for discussion

1. When is a macro preferable to a procedure?
2. Why can repeated macro expansion increase executable size?
3. What does `call` place on the stack?
4. Why does a procedure end with `ret`?
5. Why are local macro labels written with two percent signs?
6. What would happen if `m2.asm` tried to print a negative result?
7. How could `m5.asm` be extended to print the final register values?
8. Why is a busy-wait delay unsuitable for most real programs?

## Build commands

For any of the 64-bit examples:

```bash
nasm -f elf64 program.asm -o program.o
ld -m elf_x86_64 program.o -o program
./program
```

Replace `program.asm` with the required source filename.
