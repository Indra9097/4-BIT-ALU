# Design and Implementation of a 4-Bit ALU in 90nm CMOS

This project implements a full-custom, transistor-level **4-Bit Arithmetic Logic Unit (ALU)** in **Cadence Virtuoso** using the **90nm Generic PDK (`gpdk090`)** at a **1.2 V** supply voltage. The circuit is built from the ground up using a bottom-up hierarchical design flow.

## What I Did in This Project

1. **Designed Transistor-Level Leaf Cells:**
   Built, sized ($L = 100\text{ nm}$, $W_n = 240\text{ nm}$, $W_p = 480\text{ nm}$), and simulated all foundational CMOS and Transmission Gate (TG) blocks:
   * **Inverter (`INV_1X`):** 2T static CMOS inverter for bitwise NOT and control inversion.
   * **AND & OR Gates (`AND2_CMOS`, `OR2_CMOS`):** 6T static CMOS logic gates.
   * **XOR Gate (`XOR2_TG`):** 6T transmission-gate XOR with full rail-to-rail ($0\text{ V}$ to $1.2\text{ V}$) output swing.
   * **Multiplexers (`MUX_2x1_TG`, `MUX_8x1_TG`):** 4T 2x1 transmission-gate MUX scaled into a 3-stage 8x1 MUX tree to select between the 8 ALU operations.
   * **Full Adder (`FA_Mirror_28T`):** 28-transistor symmetrical CMOS mirror full adder for fast carry propagation.

2. **Assembled the 1-Bit ALU Slice (`ALU_1Bit`):**
   Combined all leaf cells into a single 1-bit ALU slice (84 transistors) controlled by a 3-bit opcode (`S2, S1, S0`). Reused the same 28T Full Adder for both **Addition** (`000`) and **2's Complement Subtraction** (`001`) by passing input `B` through an XOR gate controlled by `S0` ($A + \overline{B} + 1$).

3. **Built the 4-Bit Cascaded ALU (`ALU_4Bit`):**
   Connected four `ALU_1Bit` slices in parallel (336 transistors total) using ripple-carry chaining (`Cout` to `Cin`) and inter-slice wire routing to perform 1-bit Logical Shift Left and Logical Shift Right without extra transistors.

4. **Verified Functionality & Physical Layout:**
   * Simulated transient waveforms across all 8 opcodes in **Cadence Spectre (ADE L)** for both the 1-bit slice and the 4-bit top-level ALU (`A = 1010`, `B = 0011`).
   * Created custom physical layouts in **Virtuoso Layout XL** and verified them using **Assura DRC** (Design Rule Check) and **LVS** (Layout vs. Schematic).

## Supported Operations (3-Bit Opcode)

| Opcode (`S2 S1 S0`) | Operation | Function |
| :---: | :--- | :--- |
| `000` | **Addition** | `ALU_Out = A + B` (`Cin = 0`) |
| `001` | **Subtraction** | `ALU_Out = A - B` (`A + ~B + 1`, `Cin = 1`) |
| `010` | **Bitwise AND** | `ALU_Out = A & B` |
| `011` | **Bitwise OR** | `ALU_Out = A \| B` |
| `100` | **Bitwise XOR** | `ALU_Out = A ^ B` |
| `101` | **Bitwise NOT** | `ALU_Out = ~A` |
| `110` | **Shift Left** | `ALU_Out = A << 1` |
| `111` | **Shift Right** | `ALU_Out = A >> 1` |

## Tools & Technology Used
* **EDA Tool:** Cadence Virtuoso (Schematic Editor L, ADE L / Spectre, Layout Suite XL)
* **Verification:** Cadence Assura (DRC & LVS)
* **Technology Library:** `gpdk090` (90nm CMOS, $V_{DD} = 1.2\text{ V}$)
