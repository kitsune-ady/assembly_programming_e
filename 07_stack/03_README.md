# 03.asm — call/return with no explicit data push

## What the program demonstrates

This Linux x86-64 program shows that a function call uses the stack even when the function never writes a `push` or `pop` instruction. The stack holds the return address while `calculate` adds five to a value in `RAX`.

## Execution and stack flow

Let `S` be the initial `RSP` at `_start`.

1. `mov rax, 10` puts `10` in `RAX`; it does not touch the stack.
2. `call calculate` implicitly pushes the address of `after_call` (8 bytes), so `RSP` becomes `S-8`, then jumps to `calculate`.
3. `add rax, 5` changes `RAX` from `10` to `15`. There is no explicit stack data push here.
4. `ret` implicitly pops the return address, restores `RSP` to `S`, and continues at `after_call`.
5. The comment says `RAX` should be `15`; the `nop` does not alter it.
6. The exit setup replaces `RAX` with syscall number `60`, sets `RDI=0`, and exits successfully. This replacement means `15` is not returned as the process exit code or printed.

The stack's role here is control flow: `call` saves where execution should resume, and `ret` consumes that saved address. The return address is balanced, so the stack pointer after `ret` is back to its starting value.

## Run observations

The rebuilt ELF64 program exited with status `0` and emitted no output. The code path computes `RAX=15` before the exit setup, but provides no output instruction, so that intermediate result is only represented in the register.
