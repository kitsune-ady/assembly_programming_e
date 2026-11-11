# 05.asm — 64-bit stack values and reverse-order restoration

## What the program demonstrates

This Linux x86-64 program pushes two distinct values and pops them back into the original registers in last-in, first-out order. It illustrates both the 8-byte stack slot size and how a pop destination can differ from the register that originally supplied the value.

## Execution and stack flow

Let `S` be the initial `RSP`.

1. `RAX` is initialized to `0x1122334455667788` and `RBX` to `0x1234`. `mov` does not change `RSP`.
2. `push rax` decreases `RSP` by 8 to `S-8` and stores the 64-bit `RAX` value at the new top.
3. `push rbx` decreases `RSP` by another 8 to `S-16` and places `0x1234` on top of the earlier value.
4. `pop rbx` retrieves the top value (`0x1234`) into `RBX`, then increases `RSP` by 8 to `S-8`.
5. `pop rax` retrieves `0x1122334455667788` into `RAX` and increases `RSP` to `S`.
6. The exit setup replaces `RAX` with `60`, sets `RDI=0`, and invokes the Linux 64-bit exit syscall. It exits with status zero.

The two pushes move `RSP` down by a total of 16 bytes; the two pops move it back up by 16 bytes. The values return to their original registers because each pop matches the reverse order of the pushes. `nop` changes nothing.

## Run observations

The rebuilt ELF64 executable exited with status `0` and printed no output. Execution completes both pops before the exit syscall, so the stack depth created by this example is fully undone. The numeric values are register/stack data and are not printed.
