# DIV Operations and EFLAGS

All programs are 32-bit NASM, built with:

```bash
nasm -f elf32 -g -F dwarf prog.asm -o prog.o
ld -m elf_i386 prog.o -o prog
gdb ./prog
```

## How DIV sets flags

`div` is an unsigned divide. The dividend is AX (8-bit divisor), DX:AX (16-bit divisor) or EDX:EAX (32-bit divisor). The quotient goes in AL/AX/EAX and the remainder in AH/DX/EDX. According to the Intel manual, **CF, OF, SF, ZF, AF and PF are all undefined** after DIV, so any flag GDB shows is not a result of the division and cannot be explained from the quotient or remainder. Dividing by zero, or a quotient too big for the destination register, does not set a flag. It raises a divide error (#DE) instead.

## Program 1: `div1.asm`

**Operation:** `div bl` with AX = 100 and BL = 7, 8-bit unsigned

**Result:** AL = 14 (0x0E) quotient, AH = 2 remainder, so AX = 0x020E. Check: 14 x 7 + 2 = 100.

**EFLAGS before:** `0x202 [ IF ]`
**EFLAGS after:** `0x212 [ AF IF ]`

```
(gdb) info registers eax ebx eflags
eax            0x20e               526
ebx            0x7                 7
eflags         0x212               [ AF IF ]
```

GDB prints `526` because it shows AH:AL as one number. The quotient is the low byte (0x0E = 14) and the remainder is the high byte (0x02 = 2).

| Flag | Status | Why |
|------|--------|-----|
| CF | Cleared (undefined) | Undefined after DIV. It shows 0, which is not meaningful |
| OF | Cleared (undefined) | Undefined after DIV |
| SF | Cleared (undefined) | Undefined after DIV. It is not tied to the quotient or remainder |
| ZF | Cleared (undefined) | Undefined after DIV. It is not set from the quotient or remainder, even when they are zero |
| PF | Cleared (undefined) | Undefined after DIV |
| AF | Set (undefined) | Undefined after DIV. It changed from 0 to 1 on this CPU, but this is not a result of the division and has no defined cause |
| IF | Set | Not from the div. It was already set before and is unrelated |

**Takeaway:** The results of DIV are in the registers (quotient in AL, remainder in AH), not in the flags. AF changed even though the manual defines no flag results.

## Program 2: `div2.asm`

**Operation:** `div bx` with DX:AX = 0:50000 and BX = 300, 16-bit unsigned

**Result:** AX = 166 (0xA6) quotient, DX = 200 (0xC8) remainder. Check: 166 x 300 + 200 = 50000.

**EFLAGS before:** `0x202 [ IF ]`
**EFLAGS after:** `0x212 [ AF IF ]`

```
(gdb) info registers eax edx ebx eflags
eax            0xa6                166
edx            0xc8                200
ebx            0x12c               300
eflags         0x212               [ AF IF ]
```

| Flag | Status | Why |
|------|--------|-----|
| CF | Cleared (undefined) | Undefined after DIV. It shows 0, which is not meaningful |
| OF | Cleared (undefined) | Undefined after DIV |
| SF | Cleared (undefined) | Undefined after DIV. It is not tied to the quotient or remainder |
| ZF | Cleared (undefined) | Undefined after DIV. The quotient and remainder are non-zero, but ZF would not be set from them even if they were zero |
| PF | Cleared (undefined) | Undefined after DIV |
| AF | Set (undefined) | Undefined after DIV. It changed from 0 to 1 on this CPU, but this is not a result of the division and has no defined cause |
| IF | Set | Not from the div. It was already set before and is unrelated |

**Takeaway:** Here the 16-bit form puts the remainder in DX instead of AH. As in `div1`, AF changed even though the manual defines no flag results, which is why a program must never branch on flags after a DIV.