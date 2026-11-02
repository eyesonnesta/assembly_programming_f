# NASM Macros and Procedures

## Overview

This folder demonstrates how NASM macros reduce repeated Assembly code. A macro is defined once and expanded by the assembler wherever it is invoked. The examples also compare macros with procedures, which are executed at runtime through `call` and `ret`.

## Macro syntax

```nasm
%macro MACRO_NAME number_of_parameters
    ; instructions
%endmacro
```

Arguments are referenced as `%1`, `%2`, and so on. A macro with no arguments uses `0` as its parameter count.

## Program notes

### `m1.asm` — Printing with parameters

Defines a `PRINT` macro with two parameters: the message address and message length. The same macro prints three different messages. This shows how parameters make one macro reusable with different data.

Expected output:

```text
Welcome to Assembly!
Learning NASM macros.
One macro, multiple messages.
```

### `m2.asm` — Arithmetic and number conversion

Defines `ADD_NUMBERS`, `SUB_NUMBERS`, and `NEWLINE` macros. The arithmetic macros place their result in `RAX`. The `print_number` procedure repeatedly divides the result by 10, converts each remainder to an ASCII digit, and prints the completed decimal number.

Expected output:

```text
Addition: 40
Subtraction: 30
```

### `m3.asm` — Conditional macro

The `CHECK_ZERO` macro uses `test` and `jz` to determine whether a register contains zero. Labels beginning with `%%` are local to each macro expansion, preventing duplicate-label errors when the macro is used several times.

Expected output:

```text
Number is zero.
Number is not zero.
Number is not zero.
```

### `m4.asm` — Repetition during assembly

Uses `%rep 5` inside `PRINT_FIVE_TIMES`. NASM expands the write instructions five times while assembling the program. This is compile-time repetition rather than a runtime loop.

Expected result: `Hello!` is printed five times.

### `m5.asm` — Registers as macro arguments

The `INCREMENT` and `DECREMENT` macros accept registers as arguments. `RAX` starts at 10 and is incremented three times, producing 13. `RBX` starts at 20 and is decremented twice, producing 18. The program prints only a completion message, so the numeric values are observed through code inspection or a debugger.

### `m6.asm` — Printing and exiting

Defines `PRINT` for the Linux write system call and `EXIT_PROGRAM` for the exit system call. It demonstrates that macros can represent complete, commonly repeated operations.

Expected output:

```text
Program completed successfully.
```

The program exits with status code 0.

### `m7.asm` — Nested macros

The `PRINT_LINE` macro invokes the existing `PRINT` macro twice: once for the supplied text and once for a newline. This demonstrates how a higher-level macro can reuse another macro.

Expected output:

```text
First line
Second line
```

### `macros.asm` — Macro-based delay

Uses a printing macro and a delay macro. The delay loads a large value into `RCX` and repeatedly executes `loop` until `RCX` reaches zero.

Expected behavior: print `start ...`, pause, and then print `stop ...`.

### `procedures.asm` — Procedure-based equivalent

Implements printing and delay as procedures. Arguments are placed in registers before `call`, and each procedure returns with `ret`. It produces the same visible result as `macros.asm`, allowing macros and procedures to be compared.

## Macros versus procedures

| Macros | Procedures |
|---|---|
| Expanded during assembly | Called while the program is running |
| No `call` or `ret` overhead | Use `call`, `ret`, and the stack |
| May increase executable size | Reuse one copy of the instructions |
| Parameters use `%1`, `%2`, etc. | Arguments are commonly passed in registers or on the stack |

## Test result

All nine programs assemble and link as 64-bit ELF executables. Each program runs successfully and produces its expected output.
