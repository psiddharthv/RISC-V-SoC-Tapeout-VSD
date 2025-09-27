
## Day 2: Timing Libs, Synthesis Hierarchies & Flop Coding

### Lab 4: Introduction to Timing Libraries (`.lib`)

Timing library files, with a `.lib` extension, are crucial for synthesis as they characterize the standard cells of a specific technology. The filename itself often provides clues about the conditions under which the cells were characterized.

For example, in `sky130_fd_sc_hd__tt_025C_1v80.lib`:

  * **`tt`**: Represents the **Typical-Typical** process corner. This is one of several **PVT (Process, Voltage, Temperature)** corners. Others include `ss` (Slow-Slow) and `ff` (Fast-Fast), which model the worst and best-case performance of the silicon, respectively.
  * **`025C`**: Specifies the temperature at **25°C**.
  * **`1v80`**: Specifies the operating voltage at **1.80V**.

These PVT corners are essential to ensure the final chip performs reliably under varying real-world conditions. Inside the `.lib` file, you'll find:

  * **Standard Units**: Defines units for time, power, capacitance, etc.
  * **Cell Descriptions**: Detailed information for each standard cell (like `AND`, `OR`, `DFF`). This includes:
      * **Area**: The physical size of the cell.
      * **Pin Capacitance**: The input capacitance of each pin, which affects the delay of the cells driving it.
      * **Leakage Power**: Static power consumed by the cell.
      * **Timing Arcs**: Detailed delay information, specifying how long it takes for a signal to propagate through the cell (e.g., `cell_rise`, `cell_fall`) and the output signal's transition time (`rise_transition`).

<img width="1218" height="759" alt="Screenshot from 2025-09-27 20-45-02" src="https://github.com/user-attachments/assets/d4ebf6e8-a3f6-4d96-8910-b811e9d3b98a" />

-----

### Lab 5: Hierarchical vs. Flat Synthesis

When synthesizing a design with multiple sub-modules, Yosys can take two primary approaches.

<img width="1218" height="759" alt="Screenshot from 2025-09-27 20-48-02" src="https://github.com/user-attachments/assets/888f5806-2d38-4f04-b387-a2b90513b0d5" />


#### Hierarchical Synthesis

By default, Yosys performs a **hierarchical synthesis**. It preserves the boundaries of your sub-modules, synthesizing each one individually. When you visualize the design using `show`, you'll see boxes representing each sub-module.

<img width="1218" height="759" alt="Screenshot from 2025-09-27 20-49-45" src="https://github.com/user-attachments/assets/824e4419-1813-4e35-84d6-5ffb85684c58" />

This is often preferred because:

  * **Reusability**: If you have multiple instances of the same module (e.g., four adders), the tool synthesizes it once and reuses it.
  * **Complexity Management**: In large designs, it's easier to work on and debug one module at a time.

#### Flat Synthesis

If you want the synthesizer to see the design as a single, combined block of logic, you can use the `flatten` command. This dissolves the sub-module boundaries, allowing the tool to perform optimizations across the entire design. This can sometimes result in a more optimized gate-level netlist, as logic can be shared or simplified across what were previously separate modules.

<img width="1218" height="759" alt="Screenshot from 2025-09-27 20-53-15" src="https://github.com/user-attachments/assets/98fd0f6a-2744-44d4-81bc-71b60a9dff3a" />


-----

### Flops and Efficient Coding Styles 🧑‍💻

Combinational circuits can suffer from **glitches**—brief, unwanted signal transitions at the output. This happens because signals arrive at gates through different paths with slightly different propagation delays.

To solve this, we use sequential elements called **flip-flops (flops)**. A flop acts as a memory element that samples its input and passes it to its output only on a specific event—typically the rising (`posedge`) or falling (`negedge`) edge of a clock signal. This ensures that the output is only updated with a stable, glitch-free value after the combinational logic has had time to settle.

<img width="1218" height="759" alt="Screenshot from 2025-09-27 21-05-49" src="https://github.com/user-attachments/assets/d04b5a70-6d77-4cb3-aa98-8ec6186c3d2c" />


#### Reset Signals

Flops need to be initialized to a known state. This is done using a **reset** signal.

  * **Asynchronous Reset**: The reset happens immediately, regardless of the clock signal. It is level-sensitive. This is often preferred for putting the system into a known state instantly.
  * **Synchronous Reset**: The reset only occurs on the active clock edge. It is sampled just like any other input.

An efficient coding style for a D-flip-flop with an active-high asynchronous reset looks like this:

```verilog
always @(posedge clk or posedge reset) begin
    if (reset)
        q <= 1'b0;
    else
        q <= d;
end
```

<img width="1218" height="759" alt="Screenshot from 2025-09-27 21-08-35" src="https://github.com/user-attachments/assets/e1d4c903-a6e1-494d-aff1-0c4103b3a501" />

<img width="1218" height="759" alt="Screenshot from 2025-09-27 21-10-13" src="https://github.com/user-attachments/assets/92394052-8466-4ecc-8c70-baf2c11c75d8" />

-----

### Lab 6: Flip-Flop Synthesis

Synthesizing a design with flip-flops involves a specific step to map the generic flops to the technology library's physical cells.

The Yosys synthesis flow is:

1.  **Generic Synthesis**: The `synth` command infers high-level components like adders, multiplexers, and generic D-flip-flops.
2.  **Flop Mapping**: The `dfflibmap` command is used to map these generic flops to the specific DFF cells available in the `.lib` file (e.g., `sky130_fd_sc_hd__DFF_N_1`).
    > `dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib`
3.  **Combinational Mapping**: The `abc` command is then run to map the remaining combinational logic to standard cells and perform optimizations.
4.  **Cleanup**: The `clean` command removes any leftover intermediate cells for a tidy final netlist.

This process ensures that both the sequential and combinational parts of your design are correctly implemented using the physical cells from your chosen technology library.
<img width="1218" height="759" alt="Screenshot from 2025-09-27 21-13-16" src="https://github.com/user-attachments/assets/6f9135f5-376b-4cc2-8ff1-06c869f239b4" />
