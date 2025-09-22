# Week 0: Tool Setup and Familiarization

This directory contains all the work done in Week 0. The focus of this week is to set up the environment and get familiar with the tools that will be used throughout this project.

## Tasks: Environment Setup

* **Objective:** To install and configure all the necessary open-source EDA tools.
* **Tools Used:**
    * Yosys
    * Iverilog
    * GTKWave
    * ngspice
    * MAGIC
* **Steps:**
    1.  Tried working with Virtual Box to setup environment but it failed, possibly due to prior dual booting instances.
    2.  As a result, setup WSL environment and got Ubuntu working.
    3.  Began Yosys Installation\
    **Yosys** → Translates hardware description code (RTL) into a circuit made of logic gates (a netlist).\
    Working:
<img width="1123" height="641" alt="image" src="https://github.com/user-attachments/assets/0b32df3d-acea-4b17-8d4e-1de6aac4d259" />

    5. Began Iverilog Installation\
    **Iverilog** → Simulates RTL code to check if it behaves correctly according to its logic.\
    Working:
<img width="937" height="353" alt="image" src="https://github.com/user-attachments/assets/ca0f56f8-37cd-4715-b681-607cfc9a652f" />

    6. Began GTKWave Installation\
    **GTKWave** → A waveform viewer that visually displays the simulation results from Iverilog for debugging.\
    Working:
<img width="1387" height="621" alt="image" src="https://github.com/user-attachments/assets/120809cb-cf08-4c24-a39f-586d4f6ca6d4" />

    7. Began ngspice Installation\
    **Ngspice** → Simulates the electrical characteristics of final circuit layout to analyze timing, voltage, and power.\
    Working:
<img width="729" height="354" alt="image" src="https://github.com/user-attachments/assets/028cec95-82d8-4df1-a498-75e37fefea24" />

    9. Began Magic Installation\
    **Magic VLSI** → A tool for drawing and editing the physical polygon-based layout of integrated circuit.\
    Working:
<img width="1514" height="742" alt="image" src="https://github.com/user-attachments/assets/0c9f622a-9852-4206-baa2-4ed20e17a3d9" />




---
