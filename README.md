# IE4756 – Simple MIPS CPU (Logisim)

**Student:** u2321215g

## Part 1 – Logisim Circuit

### (1) Fast Adder (`IE4756-fast-adder.circ`)
Implements `add-32`: a 32-bit adder subcircuit using Logisim's built-in Adder component with carry-in tied to 0.  The `main` test harness instantiates `add-32` and connects two 32-bit inputs to the output.

### (2) ALU (`IE4756-mips-cpu.circ` → `ALU (32-bit)`)
The 32-bit ALU is implemented inside the CPU circuit file. It supports all operations specified by the 4-bit ALU Control Bits:

| Ctrl | Operation |
|------|-----------|
| 0000 | AND       |
| 0001 | OR        |
| 0010 | ADD (signed) |
| 0011 | ADDU (unsigned) |
| 0100 | SLT       |
| 0101 | SLTU      |
| 0110 | SUB (signed) |
| 0111 | SUBU (unsigned) |
| 1000 | SLL       |
| 1010 | SRL       |
| 1100 | XOR       |
| 1101 | NOR       |
| 1110 | SRA       |
| 1111 | LUI       |

Sub-circuits from `IE4756-arithmetic-logic-no-adder.circ` provide equality/compare and variable-shift logic; the fast adder from `IE4756-fast-adder.circ` provides addition/subtraction.

### (3) `jalr` Support
The `jalr` instruction (opcode `000000`, funct `001001`) was added by:
- **CPU Control Unit:** an AND gate detecting `funct = 001001₂ (0x09)` — specifically `funct[3]=1 ∧ funct[0]=1 ∧ UseRD=1` (bit 3 and bit 0 of the 6-bit funct field are both 1, uniquely matching jalr among instructions used in this CPU) — feeds an OR gate that drives the `Register Write` output high.
- **CPU (MIPS32) main circuit:** an OR gate combines the existing `jal` PC+4-write signal with the `JumpReg` signal (which already fires for both `jr` and `jalr` because both share `funct[3]=1`) so that PC+4 is forwarded as write-data for jalr's link register.

Together these ensure that `jalr rd, rs` sets `PC = rs` and writes `PC+4` into `rd`.

---

## Part 2 – MIPS Hex Encoding (`IE4756-mips-hex.txt`)

Student ID 7-digit part: **2321215**  
Section assigned: **2321215 % 9 = 7**  → encode **Section 7** and **Section ★**

### Section 7 (addresses `0x00400118`–`0x00400134`)
| Instruction | Encoding |
|---|---|
| `ori $8, $1, 4` | `34280004` |
| `lw $5, 0($8)` | `8D050000` |
| `ori $2, $0, 0` | `34020000` |
| `ori $3, $0, 0` | `34030000` |
| `lui $1, 64` | `3C010040` |
| `ori $31, $1, 308` | `343F0134` |
| `j 0x00400084` | `08100021` |
| `lui $1, 4097` | `3C011001` |

### Section ★ (addresses `0x004000C0`–`0x004000D4`)
| Instruction | Encoding |
|---|---|
| `bne $17, $0, -28` | `1620FFF9` |
| `ori $3, $4, 0` | `34830000` |
| `jr $31` | `03E00008` |
| `addi $29, $29, -4` | `23BDFFFC` |
| `sw $31, 0($29)` | `AFBF0000` |
| `lui $8, 4097` | `3C081001` |
