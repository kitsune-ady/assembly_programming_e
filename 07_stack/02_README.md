# 02.asm — 64-bit function frame and local variables

## What the program demonstrates

This Linux x86-64 program calls a function named `work`. The function uses the stack for the call's return address, saves the caller's frame pointer, reserves 16 bytes for two local 64-bit values, reads them into registers, and then restores the stack before returning.

## Execution and stack flow

Let `S` be `RSP` at `_start` before the `call`.

1. `call work` pushes the address of the next instruction onto the stack (8 bytes), so `RSP=S-8`, and transfers control to `work`.
2. `push rbp` saves the previous frame pointer (another 8 bytes), making `RSP=S-16`.
3. `mov rbp, rsp` establishes this function's frame base: `RBP=S-16`.
4. `sub rsp, 16` reserves 16 bytes for locals, making `RSP=S-32`. The slots `[RBP-8]` and `[RBP-16]` are within this reserved area.
5. The stores put `100` at `[RBP-8]` and `200` at `[RBP-16]`. These are memory writes, so they do not further adjust either stack pointer.
6. The loads copy those locals back into `RAX=100` and `RBX=200`. `nop` does nothing.
7. `mov rsp, rbp` discards the local area by restoring `RSP` to `S-16`.
8. `pop rbp` restores the caller's old `RBP` and advances `RSP` to `S-8`.
9. `ret` pops the saved return address, advances `RSP` to `S`, and resumes after `call work`.
10. `_start` then sets up the Linux 64-bit exit syscall (`RAX=60`, `RDI=0`). The process exits with status zero.

`RBP` stays fixed while `RSP` points to the changing top of the stack. The frame pointer makes the local offsets stable even after reserving space. The return address and saved `RBP` are control/housekeeping data; the two local numbers occupy the reserved area.

## Run observations

Reassembled as ELF64 and run, the program exited with status `0` and printed nothing. The source shows the locals loaded as `RAX=100` and `RBX=200`; the values are not displayed. The function returns normally after restoring the saved frame pointer and return address.
