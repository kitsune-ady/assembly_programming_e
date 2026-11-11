# 04.asm — 32-bit function frame and local variables

## What the program demonstrates

This is the 32-bit counterpart to the local-variable example. `work` uses `ESP` as the stack pointer and `EBP` as a stable frame pointer. A 32-bit `call`/`ret` uses 4-byte return addresses, and the function reserves two 4-byte local slots.

## Execution and stack flow

Let `S` be `ESP` at `_start` before `call work`.

1. `call work` pushes the address after the call (4 bytes), so `ESP=S-4`.
2. `push ebp` saves the old frame pointer (4 more bytes): `ESP=S-8`.
3. `mov ebp, esp` sets `EBP=S-8` as this function's frame base.
4. `sub esp, 8` reserves 8 bytes for two 32-bit locals, making `ESP=S-16`.
5. The stores place `100` at `[EBP-4]` and `200` at `[EBP-8]`. The loads copy them into `EAX=100` and `EBX=200`; these memory operations do not change `ESP`.
6. `mov esp, ebp` drops the local area and restores `ESP` to `S-8`.
7. `pop ebp` restores the caller's `EBP` and advances `ESP` to `S-4`.
8. `ret` pops the return address and restores `ESP` to `S`, continuing after the call.
9. The program invokes 32-bit Linux `exit` using `EAX=1` and `EBX=0` with `int 0x80`, so its status is zero.

In 32-bit mode the stack pointer is `ESP` (the low 32-bit stack-pointer register; `RSP` is the 64-bit name). Pushes and pops move it by 4 bytes here. `EBP` stays fixed while the locals are accessed, and resetting `ESP` to `EBP` discards the allocated local space.

## Run observations

Reassembled as ELF32 and run under WSL, it exited with status `0` and printed nothing. The local values are loaded into `EAX` and `EBX`, but then `EAX` is replaced by the exit syscall number and `EBX` by the exit status. The 32-bit call/return and frame restoration complete without a runtime error.
