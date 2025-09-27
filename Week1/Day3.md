
# 📘 Day 3: Digital Logic Optimization

Digital circuits are either **combinational** (output depends only on current inputs) or **sequential** (output depends on inputs and stored state). This guide covers techniques for optimizing both types to create more efficient hardware.

The primary goals of logic optimization are:

  * **Smaller Area:** Use fewer gates to reduce silicon footprint.
  * **Lower Power:** Reduce switching activity and power consumption.
  * **Higher Speed:** Shorten critical paths to increase clock frequency.

-----

## Combinational Logic Optimization

These techniques simplify logic expressions to reduce gate count and propagation delay.

### Key Techniques

1.  **Constant Propagation**: If an input to a logic gate is a constant (e.g., tied to `1'b0` or `1'b1`), the logic can be simplified. For instance, `(A · 0)` simplifies to `0`, removing the need for an AND gate.

2.  **Boolean Logic Optimization**: This involves simplifying logic expressions using methods like Karnaugh Maps (K-maps) or algebraic manipulation (e.g., De Morgan's laws). Complex conditional logic, often written with ternary operators, can be reduced to simpler primitive gates.

### 🧪 Lab: Verifying Combinational Optimization

In this lab, we use **Yosys** and the **Sky130 PDK** to synthesize simple Verilog modules and observe how the tool applies these optimizations.

**Typical Yosys Workflow**

```bash
# Load the standard cell library
read_liberty -lib <path_to_sky130>.lib

# Read the Verilog source
read_verilog <source_file>.v

# Run synthesis for the top module
synth -top <module_name>

# Perform technology mapping and final optimizations
abc -liberty <path_to_sky130>.lib

# Display the resulting schematic
show
```

#### Example 1: `opt_check.v` (MUX to AND Gate)

A multiplexer with a constant `0` input simplifies to a basic AND gate.

  * **Code**: `assign y = a ? b : 0;`
  * **Logic**: `y = (a · b) + (!a · 0)`
  * **Result**: `y = a & b`

<!-- end list -->

```verilog
module opt_check (input a, input b, output y);
	assign y = a ? b : 0;
endmodule
```

#### Example 2: `opt_check2.v` (MUX to OR Gate)

A multiplexer with a constant `1` input simplifies to an OR gate.

  * **Code**: `assign y = a ? 1 : b;`
  * **Logic**: `y = (a · 1) + (!a · b)`
  * **Result**: `y = a | b`

<!-- end list -->

```verilog
module opt_check2 (input a, input b, output y);
	assign y = a ? 1 : b;
endmodule
```

#### Example 3: `opt_check3.v` (Nested MUX to 3-Input AND)

Nested conditional logic can often be flattened into a single, more efficient gate.

  * **Code**: `assign y = a ? (c ? b : 0) : 0;`
  * **Logic**: `y = a · c · b`
  * **Result**: A 3-input AND gate.

<!-- end list -->

```verilog
module opt_check3 (input a, input b, input c, output y);
	assign y = a ? (c ? b : 0) : 0;
endmodule
```

#### Example 4: `opt_check4.v` (Complex MUX to XNOR)

Even complex expressions can be reduced significantly. This example simplifies to an XNOR gate.

  * **Code**: `assign y = a ? (b ? (a & c) : c) : (!c);`
  * **Result**: `y = a ⊙ c` (A XNOR C)

<!-- end list -->

```verilog
module opt_check4 (input a, input b, input c, output y);
	assign y = a ? (b ? (a & c) : c) : (!c);
endmodule
```

#### Example 5: `multiple_module_opt.v` (Hierarchical Optimization)

Synthesis tools can optimize across module boundaries. The `flatten` command in Yosys merges hierarchies, allowing for more aggressive optimization by removing redundant modules and logic.

```verilog
module sub_module1(input a, input b, output y);
  assign y = a & b;
endmodule

module sub_module2(input a, input b, output y);
  assign y = a ^ b;
endmodule

module multiple_module_opt(input a, input b, input c, input d, output y);
    wire n1,n2,n3;

    // U1 simplifies to y=a since b=1'b1
    sub_module1 U1 (.a(a) , .b(1'b1) , .y(n1));
    // U2 simplifies to y=n1 since b=1'b0
    sub_module2 U2 (.a(n1), .b(1'b0) , .y(n2));
    sub_module2 U3 (.a(b), .b(d) , .y(n3));

    assign y = c | (b & n1);
endmodule
```

-----

## Sequential Logic Optimization

Optimizing sequential logic is more complex because it involves state elements (flip-flops) that depend on clock edges and asynchronous signals.

### Key Techniques

1.  **Sequential Constant Propagation**: If a flip-flop's input is tied to a constant and its state cannot be changed by any other signal (like an async set/reset), the flip-flop can be replaced by a constant `0` or `1` driver.

2.  **Unused Output Removal**: Logic and flip-flops that do not contribute to any primary output are considered "unused" and can be safely removed. This is a powerful technique for reducing area and power.

3.  **Advanced Techniques**:

      * **State Optimization**: Removes unreachable or merges equivalent states in a Finite State Machine (FSM).
      * **Retiming**: Moves logic across flip-flop boundaries to balance path delays and improve `fmax`.
      * **Logic Cloning**: Duplicates a flip-flop to drive physically distant logic, reducing long wire delays.

### 🧪 Lab: Optimizing Flip-Flops and Counters

This lab explores when flip-flops can (and cannot) be optimized away.

**Yosys Command for Sequential Mapping**
The `dfflibmap` command maps generic DFFs to technology-specific cells from the library.

#### Example 1: `dff_const1.v` (Non-Optimizable Flop)

The output `q` is not simply `~reset` because its transition from `0` to `1` is synchronized to the clock edge. Therefore, the **flip-flop must be retained**.

```verilog
module dff_const1(input clk, input reset, output reg q);
    always @(posedge clk, posedge reset) begin
        if(reset)
            q <= 1'b0;
        else
            q <= 1'b1;
    end
endmodule
```

#### Example 2: `dff_const2.v` (Optimizable to Constant `1`)

Here, `q` is driven to `1` under all conditions (reset and otherwise). The synthesis tool correctly identifies this and **replaces the flop with a constant `1` driver**.

```verilog
module dff_const2(input clk, input reset, output reg q);
    always @(posedge clk, posedge reset) begin
        if(reset)
            q <= 1'b1;
        else
            q <= 1'b1;
    end
endmodule
```

#### Example 3: `counter_opt.v` (Unused Output Removal)

This module describes a 3-bit counter, but only the LSB `count[0]` is connected to the primary output `q`. The synthesis tool removes the flops for `count[1]` and `count[2]`, leaving only a single toggle flip-flop.

  * **Before Optimization**: 3 flip-flops.
  * **After Optimization**: 1 flip-flop.

<!-- end list -->

```verilog
module counter_opt (input clk, input reset, output q);
    reg [2:0] count;
    assign q = count[0];

    always @(posedge clk ,posedge reset) begin
        if(reset)
            count <= 3'b000;
        else
            count <= count + 1;
    end
endmodule
```

#### Example 4: `counter_opt2.v` (All Logic Retained)

In this case, the output `q` depends on all three bits of the counter (`count[2:0] == 3'b100`). Because all bits are part of the output logic cone, **all three flip-flops are retained**.

```verilog
module counter_opt (input clk, input reset, output q);
    reg [2:0] count;
    assign q = (count[2:0] == 3'b100);

    always @(posedge clk ,posedge reset) begin
        if(reset)
            count <= 3'b000;
        else
            count <= count + 1;
    end
endmodule
```

-----


  * **Gate-Level Simulation (GLS)**: Verifying the synthesized netlist with real gate delays.
  * **Blocking vs. Non-Blocking Assignments**: Understanding their impact on synthesis and simulation.
  * **Synthesis vs. Simulation Mismatches**: Identifying and preventing common RTL pitfalls.
