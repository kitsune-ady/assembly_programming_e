# SUB Operations and EFLAGS

All programs are 32-bit NASM, built with:

```bash
nasm -f elf32 -g -F dwarf prog.asm -o prog.o
ld -m elf_i386 prog.o -o prog
gdb ./prog
```

## Program 1: `sub1.asm`

**Operation:** `sub al, [num2]` with AL = 50 (0x32) and num2 = 80 (0x50), 8-bit

**Result:** AL = 0xE2 (226 unsigned, -30 signed)

**EFLAGS before:** `0x202 [ IF ]`
**EFLAGS after:** `0x287 [ CF PF SF IF ]`

```
(gdb) info registers eax eflags
eax            0xe2                226
eflags         0x287               [ CF PF SF IF ]
```

```
  0011 0010   (50)
- 0101 0000   (80)
  ---------
  1110 0010   (0xE2)
```

| Flag | Status | Why |
|------|--------|-----|
| CF | Set | Unsigned 50 < 80, so the subtraction needed a borrow from beyond bit 7 |
| OF | Cleared | Signed 50 - 80 = -30 fits in the range -128 to 127, so there is no signed overflow |
| SF | Set | Bit 7 of the result 0xE2 is 1 (it reads as -30 signed) |
| ZF | Cleared | The result is not zero |
| PF | Set | The low byte 11100010 has four 1-bits (even parity) |
| AF | Cleared | Low nibbles: 0x2 - 0x0 needs no borrow from bit 4 |

**Takeaway:** CF reports a borrow for unsigned numbers, while OF reports signed overflow. Here CF=1 and OF=0, so the result is wrong as an unsigned number but correct as a signed one (-30). IF was already set before the sub and is unrelated.

## Program 2: `sub3.asm`

**Operations:** `sub ax, [num2]` with AX = 0 and num2 = 1, then `sbb ax, 0`, 16-bit

**Results:** after `sub`, AX = 0xFFFF; after `sbb`, AX = 0xFFFE

```
(gdb) info registers eax eflags      # after sub
eax            0xffff              65535
eflags         0x297               [ CF PF AF SF IF ]
(gdb) info registers eax eflags      # after sbb
eax            0xfffe              65534
eflags         0x282               [ SF IF ]
```

### After `sub ax, 1`

| Flag | Status | Why |
|------|--------|-----|
| CF | Set | Unsigned 0 < 1, so a borrow was needed |
| OF | Cleared | Signed 0 - 1 = -1 fits in 16 bits |
| SF | Set | Bit 15 of 0xFFFF is 1 |
| ZF | Cleared | The result is not zero |
| PF | Set | Low byte 0xFF has eight 1-bits (even parity) |
| AF | Set | 0x0 - 0x1 needs a borrow from bit 4 into the low nibble |

### After `sbb ax, 0`

| Flag | Status | Why |
|------|--------|-----|
| CF | Cleared | 0xFFFF - (0 + 1) needs no borrow, since 0xFFFF >= 1 |
| OF | Cleared | Signed -1 - 0 - 1 = -2 fits in 16 bits |
| SF | Set | Bit 15 of 0xFFFE is 1 |
| ZF | Cleared | The result is not zero |
| PF | Cleared | Low byte 0xFE has seven 1-bits (odd parity) |
| AF | Cleared | 0xF - 0 - 1 = 0xE, so there is no borrow from bit 4 |

**Takeaway:** SBB subtracts the previous CF as well as the operand, so the borrow from the `sub` was carried into the `sbb`. Every flag is recomputed from the new result, which is why CF, PF and AF went back to 0.