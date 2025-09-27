# Digital Design Flow with Iverilog, GTKWave, and Yosys

## Introduction to Iverilog

The major components of a Verilog project are the **design** and the **testbench**. Stimulus inputs are applied to the design, and the outputs are monitored to check for validity. The output only changes when there is a corresponding change in the input. These changes are stored at the end in a **Value Change Dump (`.vcd`)** file. We can view these changes using the **`gtkwave`** tool, where they will be displayed in the form of waveforms.

---

## Lab 1: Environment Setup

First, we clone into the **VSDFlow** directory to gain access to the required tools. We repeat this process to get the **sky130 libraries**. These contain all the required lab files and standard Verilog cells for various digital designs.
<img width="877" height="522" alt="image" src="https://github.com/user-attachments/assets/5d17a2fd-2bab-49af-b494-c91d462aa784" />

---

## Lab 2: Intro to Iverilog and GTKWave

The syntax for **Iverilog** is:
`iverilog <design.v> <tb.v>`

This command generates an `a.out` file, which is then run as `./a.out`. This execution generates a `.vcd` file. Opening the `.vcd` file using `gtkwave` allows us to observe the waveforms, which can be dragged and dropped onto the timescale for analysis.

<img width="1092" height="634" alt="image" src="https://github.com/user-attachments/assets/0a3ad584-905c-43f4-acb3-2678bbf8ac3e" />

We can use `gvim` to observe the files containing the code for the design and the testbench. They contain the logic for the design, and the testbench contains commands for dumping the files to get the `.vcd` file and a clock to generate stimulus. In this specific case, it contains no inputs.

---

## Introduction to Yosys

A **synthesiser** is used to convert an RTL (Register Transfer Level) design into a **netlist**, which is a representation of the design using standard cells defined in a library (`.lib`) file. **Yosys** is an open-source synthesis tool.

The basic Yosys flow involves:
1.  `read_verilog` to read the design.
2.  `read_liberty` to read the standard cell library (`.lib`).
3.  `write_verilog` to output the synthesized netlist file.

To verify the synthesis, we use the generated netlist along with the original testbench in **Iverilog** and observe the output waveforms in **GTKWave**.

### The Standard Cell Library (`.lib`)

A `.lib` file is a collection of basic logical modules like logic gates. It also contains variations of the same cell with different inputs or speeds. This is crucial for translating any kind of RTL code into an accurate netlist. The choice between cell types involves trade-offs:
* **Fast cells** are needed for higher frequency and performance.
* **Slow cells** can be used to avoid hold timing issues.

To make cells faster, transistors must be wider to source more current and charge/discharge capacitance quickly. This, however, results in **greater area and power consumption**. In order to guide the synthesiser on making these choices, we use **constraints**.

---

## Lab 3: Yosys Synthesis Example

1.  Open **Yosys**.
2.  Read the libraries:
    > `read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib`
3.  Read the Verilog design file (e.g., a multiplexer):
    > `read_verilog my_mux.v`
4.  Set the top module and synthesize it to generic gates:
    > `synth -top my_mux`
5.  Map the generic gates to the standard cells from the library:
    > `abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib`
6.  Use the `show` command to see a graphical version of the logic that Yosys has realised.

<img width="1210" height="769" alt="Screenshot from 2025-09-27 20-01-42" src="https://github.com/user-attachments/assets/e9b63901-0d36-494f-baf8-58208118d73d" />


7.  Using `write_verilog` allows us to create the netlist file. We can use `vim` to open it, and it will show us all the connections in the form of text.
8.  Using the `-noattr` flag along with `write_verilog` will create a more compact netlist showing only the cells and their net connections.
