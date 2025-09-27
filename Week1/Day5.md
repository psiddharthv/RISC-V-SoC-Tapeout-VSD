

## 📘 Day 5: Writing Synthesis-Friendly Verilog

How you write your Verilog code directly impacts the quality of the hardware synthesized by tools like Yosys. A clean, explicit coding style not only prevents common issues like unintended latches but also helps the synthesis tool create a more optimal circuit in terms of area, speed, and power.

This guide covers best practices for common Verilog constructs: conditional statements (`if`, `case`), handling all possibilities, and creating repetitive logic (`for` vs. `generate`).

-----

### Conditional Logic: `if-else` vs. `case`

When you need to select one output from many inputs, both `if-else` chains and `case` statements can get the job done. However, they can produce different hardware structures.

#### `if-else-if` Chains ➡️ Priority Encoders

An `if-else-if` structure is evaluated sequentially. This tells the synthesis tool to create a **priority encoder**, where the first condition to evaluate as true takes precedence. This creates a cascade of logic that can be slow if the chain is long.

  * **RTL Code (`if_else_mux.v`)**
    ```verilog
    module if_else_mux(
        input [2:0] din,
        input [1:0] sel,
        output reg y
    );
        always @(*) begin
            if (sel == 2'b00)
                y = din[0];
            else if (sel == 2'b01)
                y = din[1];
            else if (sel == 2'b10)
                y = din[2];
            else
                y = 1'b0;
        end
    endmodule
    ```
  * **Synthesis Result**: The hardware is a chain of multiplexers. The path for `sel == 2'b10` is longer than the path for `sel == 2'b00`.

#### `case` Statements ➡️ Balanced Multiplexers

A `case` statement implies **parallel selection**. All conditions are evaluated concurrently, which typically results in a more balanced and faster multiplexer structure. For selecting between multiple data inputs, this is the preferred style.

  * **RTL Code (`case_mux.v`)**
    ```verilog
    module case_mux(
        input [2:0] din,
        input [1:0] sel,
        output reg y
    );
        always @(*) begin
            case(sel)
                2'b00 : y = din[0];
                2'b01 : y = din[1];
                2'b10 : y = din[2];
                default : y = 1'b0;
            endcase
        end
    endmodule
    ```
  * **Synthesis Result**: Yosys infers a single, balanced 4x1 multiplexer (`sky130_fd_sc_hd__mux4_1`), which is generally more efficient.

-----

### Avoiding Latches: `full_case` and `parallel_case`

One of the most common Verilog pitfalls is creating an unintentional latch. This happens when an `always` block does not specify an output value for every possible condition. To maintain its state, the synthesis tool must infer a latch.

#### The Problem: Incomplete `case`

If you forget a `case` item and don't include a `default` statement, you create a latch.

  * **RTL Code (`latch_case.v`)**
    ```verilog
    module latch_case (
        input [1:0] sel,
        input a, b,
        output reg y
    );
        // This case is incomplete (missing sel == 2'b10 and 2'b11)
        // and has no default. A latch will be inferred for 'y'.
        always @(*) begin
            case(sel)
                2'b00: y = a;
                2'b01: y = b;
            endcase
        end
    endmodule
    ```
  * **Synthesis Result**: Yosys will generate warnings about inferring a latch and will instantiate a latch cell in the netlist. Latches are generally undesirable in synchronous designs.

#### The Solution (and a Warning)

The **best practice** is to always include a `default` statement in your `case` to ensure all conditions are covered.

However, you may also see synthesis pragmas like `//synopsys full_case` and `//synopsys parallel_case`.

  * `full_case`: Tells the synthesis tool to assume that all possible cases are covered, even if they aren't written in the code. Any uncovered conditions are treated as "don't cares."
  * `parallel_case`: Tells the tool to treat the conditions as parallel, even if they overlap (which isn't possible in a standard `case` statement but can be in `casex`).

**⚠️ Warning**: Using these pragmas is **highly discouraged**. They can hide bugs in your RTL and create a **simulation-synthesis mismatch**, where your RTL simulation behaves differently from your synthesized hardware. Always favor writing explicit, complete logic with a `default` case.

-----

### Repetitive Logic: `for` loop vs. `generate`

When you need to create multiple instances of the same logic, Verilog provides two mechanisms: `for` loops and `generate` blocks. They are used for different purposes.

#### `for` loops ➡️ Unrolled Combinational Logic

A `for` loop is a procedural construct used *inside* a block like `always`. During synthesis, the loop is **unrolled** into a large block of combinational logic. This is suitable for operations that don't have a regular, repeatable hardware structure.

  * **RTL Code (`for_loop_inverter.v`)**
    ```verilog
    module for_loop_inverter (
        input [3:0] a,
        output reg [3:0] b
    );
        integer i;
        always @(*) begin
            for(i=0; i<4; i=i+1) begin
                b[i] = ~a[i];
            end
        end
    endmodule
    ```
  * **Synthesis Result**: Yosys creates four `sky130_fd_sc_hd__inv_1` cells, but they are part of a flattened, unstructured netlist.

#### `generate` loops ➡️ Structured Instantiation

A `generate` block is a structural construct used to **instantiate** modules or logic blocks multiple times. This is the preferred method for creating regular structures like arrays of inverters, full adders, or register files. It preserves hierarchy and often leads to a cleaner, more readable netlist.

  * **RTL Code (`generate_inverter.v`)**
    ```verilog
    module generate_inverter(
        input [3:0] a,
        output [3:0] b
    );
        genvar i;
        generate
            for(i=0; i<4; i=i+1) begin : inverter_array
                assign b[i] = ~a[i];
            end
        endgenerate
    endmodule
    ```
  * **Synthesis Result**: Yosys also creates four inverter cells, but the structure is more explicit and maps clearly to the generated instances, which can be helpful in debugging and physical design.

| Construct     | Best Use Case                                        | How it Works                                      |
| :------------ | :--------------------------------------------------- | :------------------------------------------------ |
| **`for` loop** | Procedural logic inside an `always` block.           | Unrolls into a single large combinational logic cone. |
| **`generate`** | Creating multiple instances of modules or assignments. | Creates distinct, structured hardware instances.          |
