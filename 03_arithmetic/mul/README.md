# MUL Operations and EFLAGS

All programs are 32-bit NASM, built with:

```bash
nasm -f elf32 -g -F dwarf prog.asm -o prog.o
ld -m elf_i386 prog.o -o prog
gdb ./prog
```

## How MUL sets flags

`mul` is an unsigned multiply. For a byte operand, `AX = AL * operand`. CF and OF are the only flags with a defined meaning: they are both set if the upper half of the product (AH for 8-bit) is non-zero, and both cleared if the product fits in the lower half. SF, ZF, AF and PF are **undefined** after MUL according to the Intel manual, so any value GDB shows for them is not a result of the multiplication.

## Program 1: `mul1.asm`

**Operation:** `mul byte [num2]` with AL = 25 (0x19) and num2 = 10 (0x0A), 8-bit unsigned

**Result:** AX = 0x00FA (250), so AH = 0x00 and AL = 0xFA

**EFLAGS before:** `0x202 [ IF ]`
**EFLAGS after:** `0x202 [ IF ]`

```
(gdb) info registers eax eflags
eax            0xfa                250
eflags         0x202               [ IF ]
```

| Flag | Status | Why |
|------|--------|-----|
| CF | Cleared | The upper half of the product (AH) is 0, so the product fits in AL |
| OF | Cleared | Set together with CF, and AH is 0 |
| SF | Cleared (undefined) | Undefined after MUL. It shows 0 here, but this is not a result of the multiply, and bit 7 of AL (0xFA) is actually 1 |
| ZF | Cleared (undefined) | Undefined after MUL. It shows 0 here, which is not meaningful |
| AF | Cleared (undefined) | Undefined after MUL. It shows 0 on this CPU, but other CPUs may differ |
| PF | Cleared (undefined) | Undefined after MUL. Not tied to the result |

**Takeaway:** Only CF and OF are meaningful after MUL. The product 250 fits in 8 bits, so both are 0. SF is a good example of the undefined flags, because it is 0 even though the top bit of AL is 1.

## Program 2: `mul2.asm`

**Operation:** `mul word [num2]` with AX = 3000 (0x0BB8) and num2 = 200 (0x00C8), 16-bit unsigned

**Result:** DX:AX = 0x0009:0x27C0 (600,000)

**EFLAGS before:** `0x202 [ IF ]`
**EFLAGS after:** `0xa03 [ CF IF OF ]`

```
(gdb) info registers eax edx eflags
eax            0x27c0              10176
edx            0x9                 9
eflags         0xa03               [ CF IF OF ]
```

| Flag | Status | Why |
|------|--------|-----|
| CF | Set | The upper half of the product (DX = 9) is non-zero, so the product did not fit in AX |
| OF | Set | Set together with CF, for the same reason |
| SF | Cleared (undefined) | Undefined after MUL. It shows 0 on this CPU, even though bit 15 of AX (0x27C0) is 0 anyway, so it tells us nothing |
| ZF | Cleared (undefined) | Undefined after MUL. The result is non-zero, but ZF is not defined to follow it |
| AF | Cleared (undefined) | Undefined after MUL |
| PF | Cleared (undefined) | Undefined after MUL |

**Takeaway:** CF and OF report whether the upper half of the product is non-zero. In `mul1` the upper half (AH) was 0, so both were cleared. Here DX = 9, so both are set. Together the two programs show both outcomes.