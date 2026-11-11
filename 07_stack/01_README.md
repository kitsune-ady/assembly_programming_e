# 01.asm — basic 64-bit push and pop

## What the program demonstrates

This is a small Linux x86-64 program showing that the stack is last-in, first-out (LIFO). It places two register values on the stack and removes them in reverse order. It does not print anything; the values are visible in registers while it runs.

## Execution and stack flow

Let `S` mean the value of `RSP` when `_start` begins. The exact address depends on the process environment, so the important facts are the changes relative to `S`.

1. `mov rax, 100` and `mov rbx, 200` set up the values. These instructions do not change `RSP`.
2. `push rax` reserves 8 bytes on this 64-bit stack: `RSP` becomes `S - 8`, and `100` is stored at `[S - 8]`.
3. `push rbx` reserves another 8 bytes: `RSP` becomes `S - 16`, with `200` at the top. The stack top now contains `200`, above `100`.
4. `pop rcx` copies the top value (`200`) into `RCX`, then releases 8 bytes: `RSP` returns to `S - 8`.
5. `pop rdx` copies `100` into `RDX` and releases the remaining 8 bytes: `RSP` returns to `S`.
6. The program sets `RAX=60` and `RDI=0`, then invokes the Linux 64-bit `exit` syscall. The syscall ends the process with status zero; there is no explicit stack data left to clean up.

The `nop` instructions have no effect on registers or the stack. Each `push`/`pop` moves `RSP` by 8 bytes, and the reverse pop order demonstrates LIFO behavior.

## Run observations

Assembled as ELF64 with NASM and linked with `ld -m elf_x86_64`, the program ran to completion with exit status `0`. It produced no terminal output. The values are demonstrated through the register transfers (`RCX=200`, then `RDX=100`), not through printed output.
