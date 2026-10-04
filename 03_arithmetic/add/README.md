# ADD Operations and EFLAGS

All programs are 32-bit NASM, built with:

```bash
nasm -f elf32 -g -F dwarf prog.asm -o prog.o
ld -m elf_i386 prog.o -o prog
gdb ./prog
```

## Program 1: `add1.asm`

**Operation:** `add al, [num2]` with AL = 120 (0x78) and num2 = 10 (0x0A), 8-bit

**Result:** AL = 0x82

**EFLAGS before:** `0x202 [ IF ]`
**EFLAGS after:** `0xa96 [ PF AF SF IF OF ]`

```
(gdb) info registers eax eflags
eax            0x82                130
eflags         0xa96               [ PF AF SF IF OF ]
```

```
  0111 1000   (120)
+ 0000 1010   ( 10)
  ---------
  1000 0010   (0x82)
```

| Flag | Status | Why |
|------|--------|-----|
| CF | Cleared | 120 + 10 = 130 fits in 8 unsigned bits, so there is no carry out of bit 7 |
| OF | Set | Two positive numbers gave a result with the sign bit set (0x82 = -126 signed), so the signed result overflowed |
| SF | Set | Bit 7 of the result is 1 |
| ZF | Cleared | The result is not zero |
| PF | Set | The low byte 10000010 has two 1-bits (even parity) |
| AF | Set | 0x8 + 0xA = 0x12, so there was a carry from bit 3 to bit 4 |

**Takeaway:** CF and OF disagree. The result is valid as an unsigned number but overflows as a signed one. (IF was already set before the add and is unrelated.)

## Program 2: `add2.asm`

**Operation:** `add ax, [num2]` with AX = 32000 (0x7D00) and num2 = 500 (0x01F4), 16-bit

**Result:** AX = 0x7EF4 (32500)

**EFLAGS before:** `0x202 [ IF ]`
**EFLAGS after:** `0x202 [ IF ]`

```
(gdb) info registers eax eflags
eax            0x7ef4              32500
eflags         0x202               [ IF ]
```

| Flag | Status | Why |
|------|--------|-----|
| CF | Cleared | 32500 fits in 16 unsigned bits, so there is no carry out of bit 15 |
| OF | Cleared | 32500 is below the signed maximum of 32767, so no signed overflow |
| SF | Cleared | Bit 15 of the result is 0 |
| ZF | Cleared | The result is not zero |
| PF | Cleared | The low byte 0xF4 = 11110100 has five 1-bits (odd parity); PF only checks the low 8 bits |
| AF | Cleared | 0x0 + 0x4 produces no carry from bit 3 to bit 4 |

**Takeaway:** EFLAGS did not change at all, because the sum fits both the unsigned and signed 16-bit ranges. IF was already set and is unrelated to the add.