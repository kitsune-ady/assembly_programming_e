# 06.asm — 32-bit stack values and reverse-order restoration

## What the program demonstrates

This Linux 32-bit program parallels `05.asm`, using `EAX` and `EBX`. It pushes two 32-bit register values, then pops them back in reverse order. In 32-bit mode each push or pop changes `ESP` by 4 bytes.

## Execution and stack flow

Let `S` be `ESP` at `_start`.

1. `EAX` is set to `0x11223344`, and `EBX` to `0x1234`; these moves leave `ESP` unchanged.
2. `push eax` moves `ESP` down 4 bytes to `S-4` and stores `0x11223344` there.
3. `push ebx` moves `ESP` down another 4 bytes to `S-8`; `0x1234` is now the top value.
4. `pop ebx` loads that top value (`0x1234`) back into `EBX`, then moves `ESP` up to `S-4`.
5. `pop eax` loads `0x11223344` into `EAX` and returns `ESP` to `S`.
6. The exit setup changes `EAX` to the 32-bit Linux syscall number `1`, sets `EBX=0`, and executes `int 0x80`; the process exits with status zero.

The push order is `EAX` then `EBX`; the pop order is `EBX` then `EAX`. This is LIFO, and the pair of pops balances the two pushes. As a 32-bit program it uses `ESP` (the 32-bit stack-pointer name; `RSP` is used in 64-bit mode).

## Run observations

The rebuilt ELF32 executable ran under WSL, exited with status `0`, and produced no terminal output. It completes both pops before changing the registers for the exit syscall, leaving the stack at its original depth. The source contains no print operation, so the demonstrated values are not visible in standard output.
