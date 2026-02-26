# Changes from Original Assignment Files

> **Student:** u2321215g &nbsp;|&nbsp; Original: `bc1eb6c` (uploaded files) → Final: `f0ab230`

This document shows every change made to the original incomplete assignment files.

---

## File-by-File Change Summary

| File | Status | Lines added | Lines removed | What changed |
|------|--------|-------------|---------------|--------------|
| `IE4756-fast-adder.circ` | Modified | +21 | 0 | Added Adder component + wires inside `add-32` |
| `IE4756-mips-cpu.circ` | Modified | +35 | +5 net | ALU bridge wires, bus fix, jalr gates/wires |
| `IE4756-mips-hex.txt` | **New file** | +79 | — | Section 7 + Section ★ hex encodings |
| `README.md` | Modified | +64 | -2 | Full documentation replaced placeholder |
| `IE4756-arithmetic-logic-no-adder.circ` | Unchanged | — | — | Pre-provided subcircuits, not modified |

---

## ① Fast Adder — `IE4756-fast-adder.circ`

**Commit:** [`defe176`](https://github.com/serenentu/proj_computer/commit/defe176895cf3e4d9337c01c7b62251e1bb0782b)  
**Change:** +21 lines inside the `add-32` sub-circuit (was completely empty)

### Before (empty circuit — no logic)
```xml
<circuit name="add-32">
  <a name="circuit" val="add-32"/>
  <!-- Input and Output pins existed but NOTHING connected them -->
  <comp lib="0" loc="(60,100)"  name="Pin"/>   <!-- Input A -->
  <comp lib="0" loc="(150,100)" name="Pin"/>   <!-- Input B -->
  <comp lib="0" loc="(260,100)" name="Pin"/>   <!-- Output  -->
  <!-- ← no wires, no Adder component; circuit would error or output 0 -->
</circuit>
```

### After (Adder connected)
```xml
<circuit name="add-32">
  <!-- Existing pins unchanged -->
  
  <!-- NEW: wires routing Input A → Adder input 1 -->
+ <wire from="(60,100)"  to="(60,60)"/>
+ <wire from="(60,60)"   to="(320,60)"/>
+ <wire from="(320,60)"  to="(320,140)"/>
  
  <!-- NEW: wires routing Input B → Adder input 2 -->
+ <wire from="(150,100)" to="(150,160)"/>
+ <wire from="(150,160)" to="(320,160)"/>
  
  <!-- NEW: wires routing Adder sum → Output pin -->
+ <wire from="(190,270)" to="(190,170)"/>
+ <wire from="(190,170)" to="(340,170)"/>
+ <wire from="(360,150)" to="(260,150)"/>
+ <wire from="(260,150)" to="(260,100)"/>
  
  <!-- NEW: carry-out wired back to 0 (not used) -->
+ <wire from="(340,130)" to="(340,40)"/>
+ <wire from="(340,40)"  to="(50,40)"/>
+ <wire from="(50,40)"   to="(50,270)"/>
+ <wire from="(50,270)"  to="(150,270)"/>
+ <wire from="(170,250)" to="(170,270)"/>
  
  <!-- NEW: 32-bit Adder component -->
+ <comp lib="3" loc="(360,150)" name="Adder">
+   <a name="width" val="32"/>
+ </comp>
  
  <!-- NEW: carry-in constant = 0 -->
+ <comp lib="0" loc="(170,250)" name="Constant">
+   <a name="facing" val="north"/>
+   <a name="value"  val="0x0"/>
+ </comp>
</circuit>
```

### Logical diagram
```
 Input A (32-bit) ──────────────┐
                                ├──► [32-bit Adder] ──► Output (32-bit)
 Input B (32-bit) ──────────────┘
 Constant(0)      ──────────────► carry-in
```

---

## ② ALU Completion — `IE4756-mips-cpu.circ`, circuit `ALU (32-bit)`

**Commits:** [`f64c420`](https://github.com/serenentu/proj_computer/commit/f64c420ac7c9a93d7a9c2f309c30aaaf691db37a) · [`843991f`](https://github.com/serenentu/proj_computer/commit/843991f1b69eaa22f6581dcf8435374e618480c5) · [`87feb498`](https://github.com/serenentu/proj_computer/commit/87feb498f6852ca938fa7dce4ec537c3ec43a24e)

The original ALU had the AND/ADD/SUB/SLT/SLL/SRL operations partially wired but had **6 missing bridge wires** that left OR, NOR, XOR, and SLTU disconnected from the output mux, and two long wires that prevented junction connections.

### Bridge wires added (commit `843991f`)

```diff
 (inside <circuit name="ALU (32-bit)">)

+ <wire from="(190,280)" to="(200,280)"/>  ← OR  operation: output bridge to mux input
+ <wire from="(190,300)" to="(200,300)"/>  ← NOR operation: output bridge to mux input
+ <wire from="(190,370)" to="(200,370)"/>  ← XOR operation: input1 bridge  
+ <wire from="(190,390)" to="(200,390)"/>  ← XOR operation: input2 bridge
+ <wire from="(380,970)" to="(390,970)"/>  ← SLTU comparator: output bridge
+ <wire from="(380,990)" to="(390,990)"/>  ← SLTU comparator: carry bridge
```

### Long wires broken into segments (commit `87feb498`)

```diff
 (inside <circuit name="ALU (32-bit)">)

 // ALU control bit 1 bus — was one long uninterrupted wire that skipped junctions:
- <wire from="(160,80)"  to="(160,440)"/>   ← single wire; OR/NOR/XOR gates couldn't tap it
- <wire from="(160,750)" to="(160,930)"/>   ← single wire; SLTU couldn't tap it

 // Replaced with short segments so each gate can connect at its junction:
+ <wire from="(160,80)"  to="(160,170)"/>
+ <wire from="(160,170)" to="(160,260)"/>
+ <wire from="(160,260)" to="(160,350)"/>
+ <wire from="(160,350)" to="(160,440)"/>
+ <wire from="(160,750)" to="(160,850)"/>
+ <wire from="(160,850)" to="(160,930)"/>
```

### ALU control-bit → operation mapping (complete)

| 4-bit Ctrl | Operation | Was it working before? |
|------------|-----------|----------------------|
| `0000` | AND | ✅ Yes |
| `0001` | OR  | ❌ No → **fixed** |
| `0010` | ADD (signed) | ✅ Yes |
| `0011` | ADDU (unsigned) | ✅ Yes |
| `0100` | SLT (signed) | ✅ Yes |
| `0101` | SLTU (unsigned) | ❌ No → **fixed** |
| `0110` | SUB (signed) | ✅ Yes |
| `0111` | SUBU (unsigned) | ✅ Yes |
| `1000` | SLL | ✅ Yes |
| `1010` | SRL | ✅ Yes |
| `1100` | XOR | ❌ No → **fixed** |
| `1101` | NOR | ❌ No → **fixed** |
| `1110` | SRA | ✅ Yes |
| `1111` | LUI | ✅ Yes |

---

## ③ jalr Instruction Support — `IE4756-mips-cpu.circ`

**Commit:** [`6ac5edca`](https://github.com/serenentu/proj_computer/commit/6ac5edca9ff7823a9ae6270379d4ab7b9773d3c9)

`jalr rd, rs` must:  1. Set `PC = rs` (already handled by existing JumpReg path)
2. Write `PC+4` into `rd` (needed **RegWrite=1** and the **PC+4 write path** — both missing)

Two small additions made:

### 3a. Main CPU circuit — forward PC+4 as write data for jalr

```diff
 (end of <circuit name="CPU (MIPS32)">)

- </circuit>
+ <comp lib="1" loc="(1010,150)" name="OR Gate"><a name="size" val="30"/></comp>
+ <wire from="(1010,90)"  to="(1010,140)"/>  ← jal signal feeds OR input 1
+ <wire from="(1010,140)" to="(980,140)"/>
+ <wire from="(430,160)"  to="(980,160)"/>   ← JumpReg signal feeds OR input 2
+ <wire from="(1010,150)" to="(1010,160)"/>  ← OR output selects "write PC+4" path
+ </circuit>
```

```
  jal  signal ─────────────────────────┐
                                       ├──[OR]──► "use PC+4 as write data" select
  JumpReg (funct[3]=1 ∧ opcode=0) ────┘

  jalr has opcode=000000 and funct=001001 → JumpReg is already 1
  → OR gate fires → PC+4 written into rd  ✓
```

### 3b. CPU control unit — enable RegWrite for jalr

```diff
 (end of <circuit name="CPU control unit">)

- </circuit>
+ <!-- 3-input AND: detects jalr uniquely (funct=001001₂ = 0x09) -->
+ <comp lib="1" loc="(470,1300)" name="AND Gate"><a name="size" val="30"/><a name="inputs" val="3"/></comp>
+ <!-- OR: existing RegWrite OR the jalr-AND output -->
+ <comp lib="1" loc="(560,1080)" name="OR Gate"><a name="size" val="30"/></comp>
+ <wire from="(530,1070)" to="(530,1080)"/>  ← existing RegWrite → OR input 1
+ <wire from="(560,1080)" to="(590,1080)"/>  ← OR output → Register Write pin
+ <wire from="(470,1300)" to="(530,1300)"/>  ← AND output → OR input 2
+ <wire from="(530,1090)" to="(530,1300)"/>
+ <wire from="(430,1260)" to="(430,1290)"/>  ← UseRD signal → AND input 1
+ <wire from="(430,1290)" to="(440,1290)"/>
+ <wire from="(190,1260)" to="(190,1300)"/>  ← funct[3] signal → AND input 2
+ <wire from="(190,1300)" to="(440,1300)"/>
+ <wire from="(250,1260)" to="(250,1310)"/>  ← funct[0] signal → AND input 3
+ <wire from="(250,1310)" to="(440,1310)"/>
+ </circuit>
```

```
  UseRD  (1 for R-type with rd dest) ─────┐
  funct[3] (1 for jalr: funct=001001) ────┼──[AND3]──┐
  funct[0] (1 for jalr: funct=001001) ────┘           │
                                                       ├──[OR]──► Register Write
  existing RegWrite (jr=0, jalr needs 1) ─────────────┘

  jalr: funct=001001 → bits[3]=1, bits[0]=1 → AND3=1 → OR=1 → RegWrite=1  ✓
  jr:   funct=001000 → bits[3]=1, bits[0]=0 → AND3=0 → OR stays 0          ✓
```

---

## ④ MIPS Hex Encoding — `IE4756-mips-hex.txt` (new file)

**Commit:** [`950616d`](https://github.com/serenentu/proj_computer/commit/950616d2e36cb540684ede59a1d4eee4369c7047)

`2321215 % 9 = 7` → Sections **7** and **★**

### Section 7 — Addresses `0x00400118` to `0x00400134`

| # | Address | Instruction | Type | Bit fields | **Hex** |
|---|---------|-------------|------|-----------|---------|
| 1 | `0x00400118` | `ori $8, $1, 4` | I | op=13(001101) rs=1(00001) rt=8(01000) imm=4(0000000000000100) | **`34280004`** |
| 2 | `0x0040011C` | `lw $5, 0($8)` | I | op=35(100011) rs=8(01000) rt=5(00101) imm=0(0000000000000000) | **`8D050000`** |
| 3 | `0x00400120` | `ori $2, $0, 0` | I | op=13(001101) rs=0(00000) rt=2(00010) imm=0(0000000000000000) | **`34020000`** |
| 4 | `0x00400124` | `ori $3, $0, 0` | I | op=13(001101) rs=0(00000) rt=3(00011) imm=0(0000000000000000) | **`34030000`** |
| 5 | `0x00400128` | `lui $1, 64` | I | op=15(001111) rs=0(00000) rt=1(00001) imm=64(0000000001000000) | **`3C010040`** |
| 6 | `0x0040012C` | `ori $31, $1, 308` | I | op=13(001101) rs=1(00001) rt=31(11111) imm=308(0000000100110100) | **`343F0134`** |
| 7 | `0x00400130` | `j 0x00400084` | J | op=2(000010) target=0x00400084>>2=0x00100021 | **`08100021`** |
| 8 | `0x00400134` | `lui $1, 4097` | I | op=15(001111) rs=0(00000) rt=1(00001) imm=4097(0001000000000001) | **`3C011001`** |

### Section ★ — Addresses `0x004000C0` to `0x004000D4`

| # | Address | Instruction | Type | Bit fields | **Hex** |
|---|---------|-------------|------|-----------|---------|
| 1 | `0x004000C0` | `bne $17, $0, -28` | I | op=5(000101) rs=17(10001) rt=0(00000) imm=-7(1111111111111001) | **`1620FFF9`** |
| 2 | `0x004000C4` | `ori $3, $4, 0` | I | op=13(001101) rs=4(00100) rt=3(00011) imm=0(0000000000000000) | **`34830000`** |
| 3 | `0x004000C8` | `jr $31` | R | op=0 rs=31(11111) rt=0 rd=0 sh=0 fn=8(001000) | **`03E00008`** |
| 4 | `0x004000CC` | `addi $29, $29, -4` | I | op=8(001000) rs=29(11101) rt=29(11101) imm=-4(1111111111111100) | **`23BDFFFC`** |
| 5 | `0x004000D0` | `sw $31, 0($29)` | I | op=43(101011) rs=29(11101) rt=31(11111) imm=0(0000000000000000) | **`AFBF0000`** |
| 6 | `0x004000D4` | `lui $8, 4097` | I | op=15(001111) rs=0(00000) rt=8(01000) imm=4097(0001000000000001) | **`3C081001`** |

#### Branch offset calculation for `bne $17, $0, -28`
```
Instruction address:  0x004000C0
PC + 4:               0x004000C4
Branch target:        0x004000A8   (0x004000C4 + (-28) = 0x004000C4 - 0x1C = 0x004000A8)
Offset in words:      (0x004000A8 - 0x004000C4) / 4 = -28 / 4 = -7
Signed 16-bit -7:     1111111111111001 = 0xFFF9   ✓
```

#### Jump target calculation for `j 0x00400084`
```
Target byte address:  0x00400084
Shift right 2:        0x00100021
J-type encoding:      op(6) | target(26) = 000010 | 00000100000000000010 0001 = 0x08100021  ✓
```

---

## ⑤ README — `README.md`

**Commit:** [`f0ab230`](https://github.com/serenentu/proj_computer/commit/f0ab230ce80ba3588ad90954cdbe175b10fc47fb)

```diff
- # proj_computer
- homework
+ # IE4756 – Simple MIPS CPU (Logisim)
+ **Student:** u2321215g
+ ## Part 1 – Logisim Circuit
+   (1) Fast Adder: add-32 using Logisim built-in 32-bit Adder
+   (2) ALU: 14 operations via 4-bit control
+   (3) jalr: funct=001001 detection, RegWrite enable, PC+4 forwarding
+ ## Part 2 – MIPS Hex Encoding
+   Section 7 + Section ★ encoding tables
```
