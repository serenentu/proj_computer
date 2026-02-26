# Logisim Circuit Before / After

Exact XML diff of every change made to the two circuit files.  
Original: commit `bc1eb6c` (your uploaded files) → Final: commit `f0ab230`

---

## 🗺️ Where to Look in Logisim

When you open a `.circ` file in Logisim, the **circuit tabs** appear in the left-hand panel under the project tree. Here is exactly which tab/sub-circuit contains each change:

| Change | File to open | Sub-circuit tab to click | Where on canvas |
|--------|-------------|--------------------------|-----------------|
| ① Fast adder wired up | `IE4756-fast-adder.circ` | **`add-32`** | Centre of canvas — you should see the 32-bit Adder box with wires connecting to the two Input pins at top and Output pin at bottom-right |
| ② ALU bridge wires | `IE4756-mips-cpu.circ` | **`ALU (32-bit)`** | Right side of canvas around y=280–400 (OR/NOR/XOR input region) and y=970–990 (SLTU comparator) — look for six short 1-grid wires |
| ③ ALU bus split | `IE4756-mips-cpu.circ` | **`ALU (32-bit)`** | The vertical control-bit bus running at x=160 — was two long wires, now four short segments (y=80→170→260→350→440) and two (y=750→850→930) |
| ④ jalr write path | `IE4756-mips-cpu.circ` | **`CPU (MIPS32)`** | Top-right area around (1010, 150) — a new OR gate visible near the jal/JumpReg signal wires |
| ⑤ jalr RegWrite | `IE4756-mips-cpu.circ` | **`CPU control unit`** | Bottom of the control unit at around y=1080–1310 — a new 3-input AND gate at (470,1300) feeding a new OR gate at (560,1080) |

### Step-by-step: opening a sub-circuit in Logisim

```
1. File → Open → select the .circ file
2. In the left panel, expand the project tree
3. Double-click the sub-circuit name (e.g. "add-32" or "ALU (32-bit)")
   → the canvas switches to show that sub-circuit
4. Use Ctrl+Shift+H (Fit to Window) to see the whole circuit
```

---

## File 1 — `IE4756-fast-adder.circ`

**Circuit changed:** `add-32`  
**What changed:** The `add-32` sub-circuit had Input/Output pins but no components or wires between them — any call to it would silently output 0.  Added a 32-bit `Adder` component with carry-in tied to `Constant(0)` and 14 wires.

### BEFORE — `add-32` (original)

```xml
<circuit name="add-32">
    <a name="circuit" val="add-32"/>
    <a name="clabel" val=""/>
    <a name="clabelup" val="east"/>
    <a name="clabelfont" val="SansSerif plain 12"/>
    <!-- ↑ NO wires, NO Adder component — just the appear block and 6 pins below -->
    <appear>
      <path d="M66,51 Q70,61 74,51" fill="none" stroke="#808080" stroke-width="2"/>
      <rect fill="none" height="30" stroke="#000000" stroke-width="2" width="30" x="55" y="50"/>
      <text font-family="SansSerif" font-size="12" text-anchor="middle" x="69" y="75">32</text>
      <text font-family="SansSerif" font-size="12" text-anchor="middle" x="69" y="63">add</text>
      <circ-port height="8"  pin="60,100"  width="8"  x="56" y="46"/>  <!-- Input A -->
      <circ-port height="8"  pin="150,100" width="8"  x="66" y="46"/>  <!-- Input B -->
      <circ-port height="10" pin="260,100" width="10" x="75" y="45"/>  <!-- Output  -->
      <circ-port height="10" pin="150,270" width="10" x="55" y="75"/>
      <circ-port height="10" pin="170,270" width="10" x="65" y="75"/>
      <circ-port height="8"  pin="190,270" width="8"  x="76" y="76"/>
      <circ-anchor facing="east" height="6" width="6" x="57" y="47"/>
    </appear>
    <comp lib="0" loc="(170,270)" name="Pin">
      <a name="facing" val="north"/>
      <a name="output" val="true"/>
      <a name="labelloc" val="east"/>
    </comp>
    <comp lib="0" loc="(150,100)" name="Pin">       <!-- Input B pin -->
      <a name="facing" val="south"/>
      <a name="width" val="32"/>
      <a name="tristate" val="false"/>
    </comp>
    <comp lib="0" loc="(190,270)" name="Pin">
      <a name="facing" val="north"/>
      <a name="tristate" val="false"/>
    </comp>
    <comp lib="0" loc="(60,100)" name="Pin">        <!-- Input A pin -->
      <a name="facing" val="south"/>
      <a name="width" val="32"/>
      <a name="tristate" val="false"/>
    </comp>
    <comp lib="0" loc="(260,100)" name="Pin">       <!-- Output pin -->
      <a name="facing" val="south"/>
      <a name="output" val="true"/>
      <a name="width" val="32"/>
      <a name="labelloc" val="east"/>
    </comp>
    <comp lib="0" loc="(150,270)" name="Pin">
      <a name="facing" val="north"/>
      <a name="output" val="true"/>
      <a name="labelloc" val="east"/>
    </comp>
  </circuit>
```

### AFTER — `add-32` (completed)

Lines marked `+++` are the additions (21 new lines, nothing deleted):

```xml
<circuit name="add-32">
    <a name="circuit" val="add-32"/>
    <a name="clabel" val=""/>
    <a name="clabelup" val="east"/>
    <a name="clabelfont" val="SansSerif plain 12"/>
+++ <wire from="(60,100)"  to="(60,60)"/>          <!-- Input A → up to routing row -->
+++ <wire from="(60,60)"   to="(320,60)"/>          <!-- across to Adder column -->
+++ <wire from="(320,60)"  to="(320,140)"/>         <!-- down to Adder input 1 -->
+++ <wire from="(150,100)" to="(150,160)"/>         <!-- Input B → down to routing row -->
+++ <wire from="(150,160)" to="(320,160)"/>         <!-- across to Adder input 2 -->
+++ <wire from="(190,270)" to="(190,170)"/>         <!-- Output pin → up -->
+++ <wire from="(190,170)" to="(340,170)"/>         <!-- across to Adder sum output -->
+++ <wire from="(360,150)" to="(260,150)"/>         <!-- Adder result → Output pin column -->
+++ <wire from="(260,150)" to="(260,100)"/>         <!-- down to Output pin -->
+++ <wire from="(340,130)" to="(340,40)"/>          <!-- carry-out → route away (unused) -->
+++ <wire from="(340,40)"  to="(50,40)"/>
+++ <wire from="(50,40)"   to="(50,270)"/>
+++ <wire from="(50,270)"  to="(150,270)"/>
+++ <wire from="(170,250)" to="(170,270)"/>         <!-- carry-in constant → carry-in pin -->
+++ <comp lib="3" loc="(360,150)" name="Adder">    <!-- 32-bit Adder component (NEW) -->
+++   <a name="width" val="32"/>
+++ </comp>
+++ <comp lib="0" loc="(170,250)" name="Constant"> <!-- carry-in = 0 (NEW) -->
+++   <a name="facing" val="north"/>
+++   <a name="value"  val="0x0"/>
+++ </comp>
    <appear>  ... (unchanged) ... </appear>
    ... (all 6 Pin components unchanged) ...
  </circuit>
```

### Logical layout (after)

```
  (60,100) Input A ──┐
                     │  routes via (60,60)→(320,60)→(320,140) ──► Adder input 1
  (150,100) Input B ─┤
                     │  routes via (150,160)→(320,160)         ──► Adder input 2
  (170,250) Constant(0)  ─────────────────────────────────────► carry-in
                                               (360,150) Adder
                                                sum ──► (340,170)→(190,170)→(190,270)
                                                                          │
                                                               (260,100) Output pin
```

---

## File 2 — `IE4756-mips-cpu.circ`

Three separate sets of changes were made. All are appended just before the `</circuit>` closing tag of their respective circuits.

---

### Change A — `ALU (32-bit)` circuit: 6 missing bridge wires (commit `843991f`)

**Why:** The Logisim XML already had OR, NOR, XOR, and SLTU gate/comparator components placed on the canvas, but each had a **1-grid gap** at the junction point — so the control-bit bus could not reach them.

#### BEFORE — end of `ALU (32-bit)` circuit

```xml
    <wire from="(270,850)" to="(290,850)"/>
    <wire from="(320,850)" to="(320,840)"/>
  </circuit>   ← nothing between here and the last wire above
```

#### AFTER — end of `ALU (32-bit)` circuit

```xml
    <wire from="(270,850)" to="(290,850)"/>
    <wire from="(320,850)" to="(320,840)"/>

+++ <wire from="(190,280)" to="(200,280)"/>  ← OR  result bus bridge  (ctrl=0001)
+++ <wire from="(190,300)" to="(200,300)"/>  ← NOR result bus bridge  (ctrl=1101)
+++ <wire from="(190,370)" to="(200,370)"/>  ← XOR gate input bridge  (ctrl=1100)
+++ <wire from="(190,390)" to="(200,390)"/>  ← XOR gate input bridge  (ctrl=1100)
+++ <wire from="(380,970)" to="(390,970)"/>  ← SLTU comparator bridge (ctrl=0101)
+++ <wire from="(380,990)" to="(390,990)"/>  ← SLTU comparator bridge (ctrl=0101)
  </circuit>
```

---

### Change B — `ALU (32-bit)` circuit: split two long bus wires (commit `87feb498`)

**Why:** In Logisim, a single long wire that passes through the grid position of a gate input does **not** create a junction — the gate input floats. Splitting the wire into short segments creates proper T-junctions at each OR/NOR/XOR/SLTU tap point.

#### BEFORE — two long vertical control-bit bus wires

```xml
    <wire from="(160,80)"  to="(160,440)"/>   ← one long wire, no junctions at 170,260,350
    <wire from="(160,750)" to="(160,930)"/>   ← one long wire, no junction at 850
```

#### AFTER — split into segments so each gate can tap the bus

```xml
--- <wire from="(160,80)"  to="(160,440)"/>
--- <wire from="(160,750)" to="(160,930)"/>

+++ <wire from="(160,80)"  to="(160,170)"/>  ← segment 1 (tap at 170 for NOR)
+++ <wire from="(160,170)" to="(160,260)"/>  ← segment 2 (tap at 260 for XOR)
+++ <wire from="(160,260)" to="(160,350)"/>  ← segment 3 (tap at 350 for OR)
+++ <wire from="(160,350)" to="(160,440)"/>  ← segment 4 (continues down)
+++ <wire from="(160,750)" to="(160,850)"/>  ← segment 1 (tap at 850 for SLTU)
+++ <wire from="(160,850)" to="(160,930)"/>  ← segment 2 (continues down)
```

---

### Change C — `CPU (MIPS32)` circuit: jalr link-register write path (commit `6ac5edca`)

**Why:** `jalr rd, rs` must write `PC+4` into `rd`. The existing `jal` instruction already had a signal that selected "write PC+4 as register data". This OR gate extends that to also fire when `JumpReg=1` (which is already true for both `jr` and `jalr` since they share `opcode=0, funct[3]=1`).

#### BEFORE — end of `CPU (MIPS32)` circuit

```xml
    <comp lib="0" loc="(30,500)" name="Clock"/>
  </circuit>   ← jalr has no way to write PC+4 into rd
```

#### AFTER — end of `CPU (MIPS32)` circuit

```xml
    <comp lib="0" loc="(30,500)" name="Clock"/>

+++ <comp lib="1" loc="(1010,150)" name="OR Gate">
+++   <a name="size" val="30"/>
+++ </comp>
+++ <wire from="(1010,90)"  to="(1010,140)"/>  ← jal "use PC+4" signal → OR input 1
+++ <wire from="(1010,140)" to="(980,140)"/>
+++ <wire from="(430,160)"  to="(980,160)"/>   ← JumpReg signal → OR input 2
+++ <wire from="(1010,150)" to="(1010,160)"/>  ← OR output → write-data mux select
  </circuit>
```

Signal diagram:
```
  jal signal (fires for jal) ──────────────────────────┐
                                                        ├──[OR]──► "write PC+4 as Rd data"
  JumpReg (fires for jr AND jalr, funct[3]=1,op=0) ────┘

  jr:   JumpReg=1, rd destination suppressed by RegWrite=0  → link reg NOT written  ✓
  jalr: JumpReg=1, rd destination enabled by RegWrite=1     → PC+4 written into rd  ✓
  jal:  jal signal=1, rd=$31 hardcoded                      → PC+4 written into $31 ✓
```

---

### Change D — `CPU control unit` circuit: jalr RegWrite enable (commit `6ac5edca`)

**Why:** `jr` has `RegWrite=0`; `jalr` needs `RegWrite=1`. The original control unit had no way to distinguish them. A 3-input AND gate detecting `UseRD=1 ∧ funct[3]=1 ∧ funct[0]=1` uniquely identifies `jalr` (funct=`001001₂`), and its output OR's into the existing `Register Write` path.

#### BEFORE — end of `CPU control unit` circuit

```xml
    <comp lib="1" loc="(470,580)" name="AND Gate">
      <a name="size" val="30"/>
      <a name="inputs" val="3"/>
    </comp>
  </circuit>   ← no jalr detection; jalr would have RegWrite=0 (wrong)
```

#### AFTER — end of `CPU control unit` circuit

```xml
    <comp lib="1" loc="(470,580)" name="AND Gate">
      <a name="size" val="30"/>
      <a name="inputs" val="3"/>
    </comp>

+++ <comp lib="1" loc="(470,1300)" name="AND Gate">  ← 3-input AND: detects jalr
+++   <a name="size" val="30"/>
+++   <a name="inputs" val="3"/>
+++ </comp>
+++ <comp lib="1" loc="(560,1080)" name="OR Gate">   ← OR: jalr OR existing RegWrite
+++   <a name="size" val="30"/>
+++ </comp>
+++ <wire from="(530,1070)" to="(530,1080)"/>  ← existing RegWrite signal → OR input 1
+++ <wire from="(560,1080)" to="(590,1080)"/>  ← OR output → Register Write output pin
+++ <wire from="(470,1300)" to="(530,1300)"/>  ← AND output → OR input 2
+++ <wire from="(530,1090)" to="(530,1300)"/>
+++ <wire from="(430,1260)" to="(430,1290)"/>  ← UseRD → AND input 1
+++ <wire from="(430,1290)" to="(440,1290)"/>
+++ <wire from="(190,1260)" to="(190,1300)"/>  ← funct[3] → AND input 2
+++ <wire from="(190,1300)" to="(440,1300)"/>
+++ <wire from="(250,1260)" to="(250,1310)"/>  ← funct[0] → AND input 3
+++ <wire from="(250,1310)" to="(440,1310)"/>
  </circuit>
```

Signal diagram:
```
  UseRD  (1 when instruction writes to rd field) ───┐
  funct[3] (= 1 for jalr: funct=001001₂) ───────────┼──[AND3]──┐
  funct[0] (= 1 for jalr: funct=001001₂) ───────────┘           │
                                                                  ├──[OR]──► Register Write
  existing RegWrite (1 for ALU/lw/jal, 0 for jr) ───────────────┘

  Verification:
  ┌──────────┬────────┬─────────┬─────────┬────────┬───────────┬────────────────┐
  │ Instr    │ UseRD  │ funct[3]│ funct[0]│ AND3   │ orig RW   │ final RegWrite │
  ├──────────┼────────┼─────────┼─────────┼────────┼───────────┼────────────────┤
  │ jr  $31  │  0     │  1      │  0      │  0     │  0        │  0   ✓ (no wr) │
  │ jalr rd  │  1     │  1      │  1      │  1     │  0        │  1   ✓ (wr rd) │
  │ ALU ops  │  1     │  varies │  varies │  0*    │  1        │  1   ✓         │
  │ lw/sw    │  varies│  0      │  varies │  0     │  1/0      │ same ✓         │
  └──────────┴────────┴─────────┴─────────┴────────┴───────────┴────────────────┘
  * For R-type ADD: funct=100000, funct[3]=0 → AND3=0 (controlled by orig path)
```

---

## Summary of all XML changes

| Circuit | File | Lines added | Lines removed | Type |
|---------|------|-------------|---------------|------|
| `add-32` | `IE4756-fast-adder.circ` | **+21** | 0 | 14 wires + 2 components |
| `ALU (32-bit)` | `IE4756-mips-cpu.circ` | **+6** | 0 | 6 bridge wires |
| `ALU (32-bit)` | `IE4756-mips-cpu.circ` | **+6** | **-2** | Bus wire split (2→6 segments) |
| `CPU (MIPS32)` | `IE4756-mips-cpu.circ` | **+5** | 0 | 1 OR gate + 4 wires (jalr write path) |
| `CPU control unit` | `IE4756-mips-cpu.circ` | **+13** | 0 | AND3 + OR gate + 11 wires (jalr detect) |
