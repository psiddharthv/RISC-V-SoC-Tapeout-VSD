

## 📘 Day 4: Verifying Your Design with Gate-Level Simulation (GLS)

**Gate-Level Simulation (GLS)** is a critical verification step where you simulate the post-synthesis **netlist**—your design described as a collection of standard cells like `AND`, `OR`, and `DFF`s—using the **same testbench** you used for your RTL code.

Think of it as a final sanity check. You're asking: "Does the circuit that the synthesis tool actually built behave identically to my original Verilog description?"

### Why is GLS Essential?

1.  ✅ **Confirms Logical Correctness**: It proves that the synthesis tool's optimizations didn't accidentally break your design's functionality.
2.  ⏱️ **Validates Timing**: When run with delay-annotated libraries, GLS can reveal timing violations that a purely functional RTL simulation would miss.
3.  ⚠️ **Catches Synthesis Mismatches**: It exposes subtle bugs from poor RTL coding that simulate correctly in RTL but synthesize into different hardware. This is the main focus of today's lab.

### The GLS Flow

The process is straightforward: you swap the RTL design with the synthesized netlist and include the standard cell library's Verilog models during compilation.

1.  **Simulate RTL**: First, verify your design's logic with a standard RTL simulation.
2.  **Synthesize**: Use a tool like Yosys to convert your RTL into a gate-level netlist.
3.  **Run GLS**: Compile your testbench, the generated netlist, and the library's Verilog models to run the final, gate-accurate simulation.

<!-- end list -->

```bash
# GLS compilation command using Icarus Verilog
iverilog <library_models>.v <design_netlist>.v <testbench>.v
```

Let's see this in action with a simple multiplexer.

### 🧪 Lab 1: A Successful GLS Run

This Verilog code for a 2x1 MUX uses a ternary operator, which is a clean and standard way to describe combinational logic.

  * **RTL Code (`ternary_operator_mux.v`)**
    ```verilog
    module ternary_operator_mux (input i0, input i1, input sel, output y);
        assign y = sel ? i1 : i0;
    endmodule
    ```
  * **Synthesis**: Yosys correctly maps this to a `sky130_fd_sc_hd__mux2_1` cell.
  * **Result**: The RTL and GLS waveforms are **identical**. This is the expected outcome for well-written code. ✅

-----

## ⚠️ The Danger of Simulation-Synthesis Mismatches

Sometimes, Verilog code that *simulates* correctly at the RTL level produces hardware that behaves differently. This dangerous gap is a **simulation-synthesis mismatch**. GLS is your primary tool for catching these issues.

Two common causes are incomplete sensitivity lists and the misuse of blocking/non-blocking assignments.

### Mismatch Cause \#1: Incomplete Sensitivity List

In an `always` block for combinational logic, the sensitivity list tells the simulator when to re-evaluate the block. If a signal is missing, the simulation will not update correctly.

#### 🧪 Lab 2: The "Bad Mux" Example

This MUX code has a critical flaw: the sensitivity list is missing the inputs `i0` and `i1`.

  * **Flawed RTL Code (`bad_mux.v`)**
    ```verilog
    module bad_mux (input i0, input i1, input sel, output reg y);
        // Problem: The block only triggers on 'sel' changes.
        // It ignores changes on i0 and i1.
        always @ (sel) begin
            if (sel)
                y <= i1;
            else
                y <= i0;
        end
    endmodule
    ```
  * **RTL Simulation Result**: The simulation is **wrong**. The output `y` doesn't change when `i0` or `i1` change; it incorrectly behaves like a latch.
  * **Synthesis Result**: The synthesis tool is smart. It ignores the flawed sensitivity list and correctly infers a MUX.
  * **GLS Result**: The GLS waveform is **correct**. It behaves like a proper MUX because it's simulating the actual synthesized hardware.

This mismatch highlights a key rule: **Always use `always @(*)` for combinational logic** to let the tool infer the complete sensitivity list for you.

### Mismatch Cause \#2: Improper Use of Blocking vs. Non-Blocking

This is one of the most common sources of bugs for Verilog beginners.

  * **Blocking (`=`):** Statements execute **sequentially** within a block. The next line doesn't start until the current one is complete. Use this for **combinational logic**.
  * **Non-Blocking (`<=`):** All right-hand side expressions are evaluated **first**, and then all left-hand side assignments happen **concurrently**. Use this for **sequential (clocked) logic**.

#### 🧪 Lab 3: The Blocking Assignment Caveat

This code intends to model the combinational logic `d = (a | b) & c`. However, the use of blocking assignments (`=`) in the wrong order creates a simulation-synthesis mismatch.

  * **Flawed RTL Code (`blocking_caveat.v`)**
    ```verilog
    module blocking_caveat (input a, input b, input c, output reg d);
        reg x;
        always @ (*) begin
            d = x & c;  // 'd' gets the OLD value of 'x' from the previous run
            x = a | b;  // 'x' is updated AFTER 'd' has already been assigned
        end
    endmodule
    ```
  * **RTL Simulation Result**: The simulation is **wrong**. Because `d` uses the old value of `x`, the output lags by one evaluation cycle, almost like there's a flip-flop.
  * **Synthesis Result**: The synthesizer sees the intended logic `(a | b) & c` and builds a simple AND/OR circuit.
  * **GLS Result**: The GLS waveform is **correct**, matching the synthesized hardware.

The fix is simple: either reorder the blocking assignments or use intermediate wires to avoid this pattern entirely.
-----

