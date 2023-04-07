# **MINA Instruction Set Architecture V2**

## **Contents**

1. **Introduction**
    1. *ISA Overview*
    2. *Exceptions And Interrupts*
    3. *Data Types*
    4. *System Registers*
    5. *Exception Handling*
2. **MINA Base Integer Instruction Set, Version 2**
    1. *Overview*
    2. *Base Instruction Formats*
    3. *Instruction Set Summary*

***

## **Introduction**

MINA is an open instruction set architecture (ISA) designed to be flexible and extendable.

Current and future goals include:
* an ISA suitable for hardware implementation
* division, floating-point and vector extensions
* opcode space for custom instruction set extensions
* 32-bit and 64-bit address space variants.

***

### **1.1 ISA Overview**

The MINA ISA is defined as a base integer ISA with optional division, floating-point, vector and user defined instruction extensions. The two base integer ISAs are **MINA32** (32-bit address space) and **MINA64** (planned 64-bit address space variant).

One of the two base integer ISAs must be present in any implementation.

***

### **1.2 Exceptions And Interrupts**

The MINA ISA defines two runtime events: *exceptions* and *interrupts*.

Exceptions are events that occur *synchronously* to the current MINA thread. They transfer control to an exception handler.

Interrupts are events that occur *asynchronously* to the current MINA thread. They transfer control to an exception handler at the end of the currently executing instruction.

All types of exceptions that can be generated by instructions are described in the detailed instruction description sections in this ISA manual.

***

### **1.3 Data Types**

The MINA ISA currently defines the following data types:
* **Byte** (8-bit)
* **Halfword** (16-bit)
* **Word** (32-bit)
* **Longword** (64-bit).

***

### **1.4 System Registers**

The MINA ISA defines a word addressable 12-bit address space for special purpose registers called *system registers* (SR). Programmers can access SRs with the `MTS` (Move To System) and `MFS` (Move From System) instructions.

The following addresses are currently defined:

|--------------------------------------------------------------------------|
| Addr | Name       | Description                   | Access | Priv Level  |
|--------------------------------------------------------------------------|
|------- CPU Identification -----------------------------------------------|
| 000h | CPUID_L    | CPU IDentification Lo         | R      | USR/SYS/HVC |
| 001h | CPUID_H    | CPU IDentification Hi         | R      | USR/SYS/HVC |
|------- Exception --------------------------------------------------------|
| 024h | ESC_SYS    | SYS Exception System Control  | R/W    |     SYS/HVC |
| 025h | ECO_SYS    | SYS Exception COde            | R      |     SYS/HVC |
| 026h | EPC_SYS    | SYS Exception Program Counter | R/W    |     SYS/HVC |
| 027h | EMA_SYS    | SYS Exception Memory Address  | R      |     SYS/HVC |
| 02Ch | ESC_HVC    | HVC Exception System Control  | R/W    |         HVC |
| 02Dh | ECO_HVC    | HVC Exception COde            | R      |         HVC |
| 02Eh | EPC_HVC    | HVC Exception Program Counter | R/W    |         HVC |
| 02Fh | EMA_HVC    | HVC Exception Memory Address  | R      |         HVC |
| 034h | EVA_SYS    | SYS Exception Vector Address  | R/W    |     SYS/HVC |
| 03Ch | EVA_HVC    | HVC Exception Vector Address  | R/W    |         HVC |
|------- System Control ---------------------------------------------------|
| 040h | PSTATE     | Processor STATE               | R      | USR/SYS/HVC |
| 042h | CTRL_USR   | USR ConTRoL                   | R/W    | USR/SYS/HVC |
| 046h | CTRL_SYS   | SYS ConTRoL                   | R/W    |     SYS/HVC |
| 04Eh | CTRL_HVC   | HVC ConTRoL                   | R/W    |         HVC |
|------- Program ----------------------------------------------------------|
| 060h | LA_USR     | USR Link Address              | R/W    | USR/SYS/HVC |
| 061h | BA_USR     | USR Base Address              | R/W    | USR/SYS/HVC |
| 064h | LA_SYS     | SYS Link Address              | R/W    |     SYS/HVC |
| 065h | BA_SYS     | SYS Base Address              | R/W    |     SYS/HVC |
| 06Ch | LA_HVC     | HVC Link Address              | R/W    |         HVC |
| 06Dh | BA_HVC     | HVC Base Address              | R/W    |         HVC |
|--------------------------------------------------------------------------|

Addresses <= 7FFh are reserved, addresses >= 800h are implementation defined.

***

### **1.5 Exception Handling**

TODO...

***

## **MINA32 Base Integer Instruction Set, Version 2**

The following section describes version 2 of the MINA32 base integer ISA.

***

### **2.1. Overview**

MINA32 has 32 32-bit general-purpose registers (*r0-r31*).

***

### **2.2. Base Instruction Formats**

MINA32 has four core instruction formats (R/S/LB/LL). Instructions must be aligned on a 4-byte boundary.

```
Instruction Formats:

|-----------------------------------------------------------------|
| 3 3 2 2 2 2 2 2 2 2 2 2 1 1 1 1 1 1 1 1 1 1                     |
| 1 0 9 8 7 6 5 4 3 2 1 0 9 8 7 6 5 4 3 2 1 0 9 8 7 6 5 4 3 2 1 0 |
|-----------------------------------------------------------------|
| o o o o o o o o 0 0 i i i i i i i i i i i i i i i i i i i i i i | (LB) Long immediate - Branch
|-----------------------------------------------------------------|
| o o o o o o o o 0 1 D D D D D i i i i i i i i i i i i i i i i i | (LL) Long immediate - Load
|-----------------------------------------------------------------|
| o o o o o o o o 1 0 D D D D D A A A A A i i i i i i i i i i i i | (S) Short immediate
|-----------------------------------------------------------------|
| o o o o o o o o 1 1 D D D D D A A A A A B B B B B o o o o o o o | (R) Register
|-----------------------------------------------------------------|
```

**LB**: LB (Long immediate - Branch) is an immediate type format. It supplies a small set of branch instructions (`BL`/`BLT`/`BRA`/`BT`) with a signed 22-bit immediate, left-shifted by 2.

**LL**: LL (Long immediate - Load) is an immediate type format. It supplies special load-store instructions (`LPS`/`LPP`/`LWS`/`LWP`/`SPS`/`SWS`) with an unsigned 17-bit immediate, left-shifted by 2.

**S**: S (Short immediate) is the standard immediate type format. It supplies all ALU instructions with a 12-bit immediate and a register operand.

**R**: R (Register) is the register type format. It supplies all ALU instructions with two register operands.

***

### **2.3. Instruction Set Summary**

```
LOAD    - 0000 PSSz Fm |   P = Pair   | S = Sign-extend |    Sz = Size    | Fm = Format
STORE   - 0001 P-Sz Fm |   P = Pair   |                 |    Sz = Size    | Fm = Format
SLOAD   - 0010 PRSz Fm |   P = Pair   | R = PC-relative |    Sz = Size    | Fm = Format
SSTORE  - 0011 P-Sz Fm |   P = Pair   |                 |    Sz = Size    | Fm = Format
CONTROL - 0100 Opco Fm |              |  Opco = Opcode  |                 | Fm = Format
SALU    - 1010 CSSz Fm |   C = Clip   | S = Sign-extend |    Sz = Size    | Fm = Format
BRANCH  - 1110 R-CL Fm | R = Relative | C = Conditional |    L  = Link    | Fm = Format
EXTEND  - 1111 xxxx Fm |              |                 |                 | Fm = Format
```

```
|--------------------------------------------------------------------------------------------------------------------------|
| Group   | Format                 | Name             | Abstract                                        | Code             |
|--------------------------------------------------------------------------------------------------------------------------|
| LOAD    | LB     RD, (RA, #imm)  | Load Byte        | r8(RA + exts(#imm)) -> RD                       | 0000000010DDDDDA |
| (S)     |                        |                  |                                                 | AAAAiiiiiiiiiiii |
|--------------------------------------------------------------------------------------------------------------------------|
| LOAD    | LB     RD, (RA, RB)    | Load Byte        | r8(RA + RB) -> RD                               | 0000000011DDDDDA |
| (R)     |                        |                  |                                                 | AAAABBBBB0000000 |
|--------------------------------------------------------------------------------------------------------------------------|
| LOAD    | LBS    RD, (RA, #imm)  | Load Byte Signed | r8(RA + exts(#imm)) -> tmp                      | 0000010010DDDDDA |
| (S)     |                        |                  | exts(tmp) -> RD                                 | AAAAiiiiiiiiiiii |
|--------------------------------------------------------------------------------------------------------------------------|
| LOAD    | LBS    RD, (RA, RB)    | Load Byte Signed | r8(RA + RB) -> tmp                              | 0000010111DDDDDA |
| (R)     |                        |                  | exts(tmp) -> RD                                 | AAAABBBBBiiiiiii |
|--------------------------------------------------------------------------------------------------------------------------|
| LOAD    | LH     RD, (RA, #imm)  | Load Halfword    | r16(RA + 2 * exts(#imm)) -> RD                  | 0000000110DDDDDA |
| (S)     |                        |                  |                                                 | AAAAiiiiiiiiiiii |
|--------------------------------------------------------------------------------------------------------------------------|
| LOAD    | LH     RD, (RA, RB)    | Load Halfword    | r16(RA + RB) -> RD                              | 0000000111DDDDDA |
| (R)     |                        |                  |                                                 | AAAABBBBB0000000 |
|--------------------------------------------------------------------------------------------------------------------------|
| LOAD    | LHS    RD, (RA, #imm)  | Load Halfword    | r16(RA + 2 * exts(#imm)) -> tmp                 | 0000010110DDDDDA |
| (S)     |                        | Signed           | exts(tmp) -> RD                                 | AAAAiiiiiiiiiiii |
|--------------------------------------------------------------------------------------------------------------------------|
| LOAD    | LHS    RD, (RA, RB)    | Load Halfword    | r16(RA + RB) -> tmp                             | 0000010111DDDDDA |
| (R)     |                        | Signed           | exts(tmp) -> RD                                 | AAAABBBBBiiiiiii |
|--------------------------------------------------------------------------------------------------------------------------|
| LOAD    | LW     RD, (RA, #imm)  | Load Word        | r32(RA + 4 * exts(#imm)) -> RD                  | 0000001010DDDDDA |
| (S)     |                        |                  |                                                 | AAAAiiiiiiiiiiii |
|--------------------------------------------------------------------------------------------------------------------------|
| LOAD    | LW     RD, (RA, RB)    | Load Word        | r32(RA + RB) -> RD                              | 0000001011DDDDDA |
| (R)     |                        |                  |                                                 | AAAABBBBB0000000 |
|--------------------------------------------------------------------------------------------------------------------------|
| LOAD    | LP     PD, (RA, #imm)  | Load Pair        | r32(RA + 4 * exts(#imm) + 0) -> PD+0            | 0000101010PPPP0A |
| (S)     |                        |                  | r32(RA + 4 * exts(#imm) + 4) -> PD+1            | AAAAiiiiiiiiiiii |
|--------------------------------------------------------------------------------------------------------------------------|
| LOAD    | LP     PD, (RA, RB)    | Load Pair        | r32(RA + RB + 0) -> PD+0                        | 0000101011PPPP0A |
| (R)     |                        |                  | r32(RA + RB + 4) -> PD+1                        | AAAABBBBB0000000 |
|--------------------------------------------------------------------------------------------------------------------------|
|--------------------------------------------------------------------------------------------------------------------------|
| STORE   | SB     RD, (RA, #imm)  | Store Byte       | RD -> w8(RA + exts(#imm))                       | 0001000010DDDDDA |
| (S)     |                        |                  |                                                 | AAAAiiiiiiiiiiii |
|--------------------------------------------------------------------------------------------------------------------------|
| STORE   | SB     RD, (RA, RB)    | Store Byte       | RD -> w8(RA + RB)                               | 0001000011DDDDDA |
| (R)     |                        |                  |                                                 | AAAABBBBB0000000 |
|--------------------------------------------------------------------------------------------------------------------------|
| STORE   | SH     RD, (RA, #imm)  | Store Halfword   | RD -> w16(RA + 2 * exts(#imm))                  | 0001000110DDDDDA |
| (S)     |                        |                  |                                                 | AAAAiiiiiiiiiiii |
|--------------------------------------------------------------------------------------------------------------------------|
| STORE   | SH     RD, (RA, RB)    | Store Halfword   | RD -> w16(RA + RB)                              | 0001000111DDDDDA |
| (R)     |                        |                  |                                                 | AAAABBBBB0000000 |
|--------------------------------------------------------------------------------------------------------------------------|
| STORE   | SW     RD, (RA, #imm)  | Store Word       | RD -> w32(RA + 4 * exts(#imm))                  | 0001001010DDDDDA |
| (S)     |                        |                  |                                                 | AAAAiiiiiiiiiiii |
|--------------------------------------------------------------------------------------------------------------------------|
| STORE   | SW     RD, (RA, RB)    | Store Word       | RD -> w32(RA + RB)                              | 0001001011DDDDDA |
| (R)     |                        |                  |                                                 | AAAABBBBB0000000 |
|--------------------------------------------------------------------------------------------------------------------------|
| STORE   | SP     PD, (RA, #imm)  | Store Pair       | PD+0 -> w32(RA + 4 * exts(#imm) + 0)            | 0001101011PPPP0A |
| (S)     |                        |                  | PD+1 -> w32(RA + 4 * exts(#imm) + 4)            | AAAAiiiiiiiiiiii |
|--------------------------------------------------------------------------------------------------------------------------|
| STORE   | SP     PD, (RA, RB)    | Store Pair       | PD+0 -> w32(RA + RB + 0)                        | 0001101011PPPP0A |
| (R)     |                        |                  | PD+1 -> w32(RA + RB + 4)                        | AAAABBBBB0000000 |
|--------------------------------------------------------------------------------------------------------------------------|
|--------------------------------------------------------------------------------------------------------------------------|
| SLOAD   | LWB    RD, (BA, #imm)  | Load Word Base   | r32(BA + 4 * #imm) -> RD                        | 0010001001DDDDDi |
| (LL)    |                        |                  |                                                 | iiiiiiiiiiiiiiii |
|--------------------------------------------------------------------------------------------------------------------------|
| SLOAD   | LWP    RD, (PC, #imm)  | Load Word PC     | r32(PC + 4 * #imm) -> RD                        | 0010011001DDDDDi |
| (LL)    |                        |                  |                                                 | iiiiiiiiiiiiiiii |
|--------------------------------------------------------------------------------------------------------------------------|
| SLOAD   | LPB    PD, (BA, #imm)  | Load Pair Base   | r32(BA + 4 * #imm + 0) -> PD+0                  | 0010101001PPPP0i |
| (LL)    |                        |                  | r32(BA + 4 * #imm + 4) -> PD+1                  | iiiiiiiiiiiiiiii |
|--------------------------------------------------------------------------------------------------------------------------|
| SLOAD   | LPP    PD, (PC, #imm)  | Load Pair PC     | r32(PC + 4 * #imm + 0) -> PD+0                  | 0010111001PPPP0i |
| (LL)    |                        |                  | r32(PC + 4 * #imm + 4) -> PD+1                  | iiiiiiiiiiiiiiii |
|--------------------------------------------------------------------------------------------------------------------------|
|--------------------------------------------------------------------------------------------------------------------------|
| SSTORE  | SWB    RD, (BA, #imm)  | Store Word Base  | RD -> w32(BA + 4 * #imm)                        | 0011001001DDDDDi |
| (LL)    |                        |                  |                                                 | iiiiiiiiiiiiiiii |
|--------------------------------------------------------------------------------------------------------------------------|
| SSTORE  | SPB    PD, (BA, #imm)  | Store Pair Base  | PD+0 -> w32(BA + 4 * #imm + 0)                  | 0011101001PPPP0i |
| (LL)    |                        |                  | PD+1 -> w32(BA + 4 * #imm + 4)                  | iiiiiiiiiiiiiiii |
|--------------------------------------------------------------------------------------------------------------------------|
|--------------------------------------------------------------------------------------------------------------------------|
| CONTROL | CLRT                   | CLeaR T          | 0 -> PSTATE.T                                   | 0100000000000000 |
| (LB)    |                        |                  |                                                 | 0000000000000000 |
|--------------------------------------------------------------------------------------------------------------------------|
| CONTROL | SETT                   | SET T            | 1 -> PSTATE.T                                   | 0100000100000000 |
| (LB)    |                        |                  |                                                 | 0000000000000000 |
|--------------------------------------------------------------------------------------------------------------------------|
| CONTROL | NOTT                   | NOT T            | ~(PSTATE.T) -> PSTATE.T                         | 0100001000000000 |
| (LB)    |                        |                  |                                                 | 0000000000000000 |
|--------------------------------------------------------------------------------------------------------------------------|
|--------------------------------------------------------------------------------------------------------------------------|
| ALU     | ADD    RD, RA, #imm    | ADD              | RA + exts(#imm) -> RD                           | 1000000010DDDDDA |
| (S)     |                        |                  |                                                 | AAAAiiiiiiiiiiii |
|--------------------------------------------------------------------------------------------------------------------------|
| ALU     | ADD    RD, RA, RB      | ADD              | RA + RB -> RD                                   | 1000000011DDDDDA |
| (S)     |                        |                  |                                                 | AAAABBBBB0000000 |
|--------------------------------------------------------------------------------------------------------------------------|
| ALU     | SUB    RD, RA, RB      | SUBtract         | RA - RB -> RD                                   | 1000000111DDDDDA |
| (S)     |                        |                  |                                                 | AAAABBBBB0000000 |
|--------------------------------------------------------------------------------------------------------------------------|
| ALU     | NEG    RD, RB          | NEGate           | 0 - RB -> RD                                    | 1000001111DDDDD0 |
| (S)     |                        |                  |                                                 | 0000BBBBB0000000 |
|--------------------------------------------------------------------------------------------------------------------------|
| ALU     | AND    RD, RA, #imm    | AND              | RA & #imm -> RD                                 | 1001000010DDDDDA |
| (S)     |                        |                  |                                                 | AAAAiiiiiiiiiiii |
|--------------------------------------------------------------------------------------------------------------------------|
| ALU     | AND    RD, RA, RB      | AND              | RA & RB -> RD                                   | 1001000011DDDDDA |
| (S)     |                        |                  |                                                 | AAAABBBBB0000000 |
|--------------------------------------------------------------------------------------------------------------------------|
| ALU     | ANDN   RD, RA, #imm    | AND Not          | RA & ~(#imm) -> RD                              | 1001000110DDDDDA |
| (S)     |                        |                  |                                                 | AAAAiiiiiiiiiiii |
|--------------------------------------------------------------------------------------------------------------------------|
| ALU     | ANDN   RD, RA, RB      | AND Not          | RA & ~(RB) -> RD                                | 1001000111DDDDDA |
| (S)     |                        |                  |                                                 | AAAABBBBB0000000 |
|--------------------------------------------------------------------------------------------------------------------------|
| ALU     | OR     RD, RA, #imm    | OR               | RA | #imm -> RD                                 | 1001010010DDDDDA |
| (S)     |                        |                  |                                                 | AAAAiiiiiiiiiiii |
|--------------------------------------------------------------------------------------------------------------------------|
| ALU     | OR     RD, RA, RB      | OR               | RA | RB -> RD                                   | 1001010011DDDDDA |
| (S)     |                        |                  |                                                 | AAAABBBBB0000000 |
|--------------------------------------------------------------------------------------------------------------------------|
| ALU     | NOT    RD, RB          | NOT              | ~(RB) -> RD                                     | 1001011111DDDDD0 |
| (S)     |                        |                  |                                                 | 0000BBBBB0000000 |
|--------------------------------------------------------------------------------------------------------------------------|
| ALU     | XOR    RD, RA, #imm    | XOR              | RA ^ exts(#imm) -> RD                           | 1001100010DDDDDA |
| (S)     |                        |                  |                                                 | AAAAiiiiiiiiiiii |
|--------------------------------------------------------------------------------------------------------------------------|
| ALU     | XOR    RD, RA, RB      | XOR              | RA ^ RB -> RD                                   | 1001100011DDDDDA |
| (S)     |                        |                  |                                                 | AAAABBBBB0000000 |
|--------------------------------------------------------------------------------------------------------------------------|
|--------------------------------------------------------------------------------------------------------------------------|
| SALU    | EXTBS  RD, RA          | Extend Byte      | exts(RA AND FFh) -> RD                          | 1010010010DDDDDA |
| (S)     |                        | Signed           |                                                 | AAAA000000000000 |
|--------------------------------------------------------------------------------------------------------------------------|
| SALU    | EXTH   RD, RA          | Extend Halfword  | ext(RA AND FFFFh) -> RD                         | 1010000110DDDDDA |
| (S)     |                        |                  |                                                 | AAAA000000000000 |
|--------------------------------------------------------------------------------------------------------------------------|
| SALU    | EXTHS  RD, RA          | Extend Halfword  | exts(RA AND FFFFh) -> RD                        | 1010010110DDDDDA |
| (S)     |                        | Signed           |                                                 | AAAA000000000000 |
|--------------------------------------------------------------------------------------------------------------------------|
| SALU    | CLPB   RD, RA          | CLiP Byte        | if RA > FFh: FFh -> RD, 1 -> PSTATE.T           | 1010100010DDDDDA |
| (S)     |                        |                  | else:         RA -> RD, 0 -> PSTATE.T           | AAAA000000000000 |
|--------------------------------------------------------------------------------------------------------------------------|
| SALU    | CLPBS  RD, RA          | CLiP Byte Signed | if RA >  7Fh:  7Fh -> RD, 1 -> PSTATE.T         | 1010110010DDDDDA |
| (S)     |                        |                  | if RA < -80h: -80h -> RD, 1 -> PSTATE.T         | AAAA000000000000 |
|         |                        |                  | else:           RA -> RD, 0 -> PSTATE.T         |                  |
|--------------------------------------------------------------------------------------------------------------------------|
| SALU    | CLPH   RD, RA          | CLiP Halfword    | if RA > FFFFh: FFFFh -> RD, 1 -> PSTATE.T       | 1010100110DDDDDA |
| (S)     |                        |                  | else:             RA -> RD, 0 -> PSTATE.T       | AAAA000000000000 |
|--------------------------------------------------------------------------------------------------------------------------|
| SALU    | CLPHS  RD, RA          | CLiP Halfword    | if RA >  7FFFh:  7FFFh -> RD, 1 -> PSTATE.T     | 1010110110DDDDDA |
| (S)     |                        | Signed           | if RA < -8000h: -8000h -> RD, 1 -> PSTATE.T     | AAAA000000000000 |
|         |                        |                  | else:               RA -> RD, 0 -> PSTATE.T     |                  |
|--------------------------------------------------------------------------------------------------------------------------|
|--------------------------------------------------------------------------------------------------------------------------|
| BRANCH  | BT RD                  | Branch if True   | if T: RD -> PC                                  | 1110000001DDDDD0 |
| (LL)    |                        |                  |                                                 | 0000000000000000 |
|--------------------------------------------------------------------------------------------------------------------------|
| BRANCH  | CT RD                  | Call if True     | if T: PC + 4 -> LA                              | 1110000101DDDDD0 |
| (LL)    |                        |                  |       RD -> PC                                  | 0000000000000000 |
|--------------------------------------------------------------------------------------------------------------------------|
| BRANCH  | B RD                   | Branch           | RD -> PC                                        | 1110001001DDDDD0 |
| (LL)    |                        |                  |                                                 | 0000000000000000 |
|--------------------------------------------------------------------------------------------------------------------------|
| BRANCH  | CALL RD                | CALL             | PC + 4 -> LA                                    | 1110001101DDDDD0 |
| (LL)    |                        |                  | RD -> PC                                        | 0000000000000000 |
|--------------------------------------------------------------------------------------------------------------------------|
| BRANCH  | BRT LABEL              | Branch Relative  | if T: PC + 4 * exts(#imm) -> PC                 | 1110100000iiiiii |
| (LB)    |                        | if True          |                                                 | iiiiiiiiiiiiiiii |
|--------------------------------------------------------------------------------------------------------------------------|
| BRANCH  | BRT RD                 | Branch Relative  | if T: PC + RD -> PC                             | 1110100001DDDDD0 |
| (LL)    |                        | if True          |                                                 | 0000000000000000 |
|--------------------------------------------------------------------------------------------------------------------------|
| BRANCH  | CRT LABEL              | Call Relative    | if T: PC + 4 -> LA                              | 1110100100iiiiii |
| (LB)    |                        | if True          |       PC + 4 * exts(#imm) -> PC                 | iiiiiiiiiiiiiiii |
|--------------------------------------------------------------------------------------------------------------------------|
| BRANCH  | CRT RD                 | Call Relative    | if T: PC + 4 -> LA                              | 1110100101DDDDD0 |
| (LL)    |                        | if True          |       PC + RD -> PC                             | 0000000000000000 |
|--------------------------------------------------------------------------------------------------------------------------|
| BRANCH  | BR LABEL               | Branch Relative  | PC + 4 * exts(#imm) -> PC                       | 1110101000iiiiii |
| (LB)    |                        |                  |                                                 | iiiiiiiiiiiiiiii |
|--------------------------------------------------------------------------------------------------------------------------|
| BRANCH  | BR RD                  | Branch Relative  | PC + RD -> PC                                   | 1110101001DDDDD0 |
| (LL)    |                        |                  |                                                 | 0000000000000000 |
|--------------------------------------------------------------------------------------------------------------------------|
| BRANCH  | CR LABEL               | Call Relative    | PC + 4 -> LA                                    | 1110101100iiiiii |
| (LB)    |                        |                  | PC + 4 * exts(#imm) -> PC                       | iiiiiiiiiiiiiiii |
|--------------------------------------------------------------------------------------------------------------------------|
| BRANCH  | CR RD                  | Call Relative    | PC + 4 -> LA                                    | 1110101101DDDDD0 |
| (LL)    |                        |                  | PC + RD -> PC                                   | 0000000000000000 |
|--------------------------------------------------------------------------------------------------------------------------|
```