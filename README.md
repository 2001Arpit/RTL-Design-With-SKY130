# RTL-Design-With-SKY130
![image](https://user-images.githubusercontent.com/92947276/166198612-f30c8d13-e1b6-4abb-a5fe-299c2b201d88.png)

# Overview
This workshop aims to teach
- Verilog coding guidelines 
- Synthesizable Verilog codes
- Synthesis
- Optimizations
- Synthesis-Simulation mismatch
- If statements
- For loop and For generate.

We will achieve the above using iverilog for simulations and yosys for synthesis. In addition, we will be using sky130 standard cell libraries. It will tell our synthesis tool which component to use for our design. This library contains details like power consumption, delays, area, etc.

# Table of Contents
- [Introduction](#introduction)

# Introduction
## RTL (Register Transfer Level) Design
Transistor-level design connects transistors into circuits to build gates or other components. Combinational or sequential circuits whose building blocks are primarily logic gates are thus called logic-level design.
Designing circuits whose building blocks are registers and other datapath components consists of transferring data from registers through other datapath components like adders and back to registers. Such design is thus called register-transfer level design or RTL design.
## Icarus Verilog
- iverilog or Icarus Verilog is a simulation and synthesis tool. It should be noted that we will be using it as a simulation tool only and do the synthesis using yosys.
We will be viewing the simulation results using a waveform viewer known as gtkwave. It uses a .vcd file to produce the waveform. VCD stands for value change dump. It is a dump file that the gtkwave uses for simulation.
- A testbench provides these 'changes' in values. A testbench is a Verilog program that checks the functionality of our design by giving various possible inputs to the
design.  
![image](https://user-images.githubusercontent.com/92947276/166210377-a6ea5c51-9946-4413-93fc-590ca29538b0.png)
- Lets take an example of a counter which counts from 0 to 2:
![counter program and test bench](https://user-images.githubusercontent.com/92947276/166214568-e9dfe5ad-e354-46e7-9986-99eb70d7a71c.PNG)
- To run these files using iverilog, use the following command: `iverilog good_counter.v tb_good_counter.v`.
- If there were no errors, this will create a 'a.out' file in your current working directory. 'a.out' is an output file. Running this file will create .vcd file which will be used for simulation. To run the 'a.out' file use the following command: `./a.out` or `vvp a.out`.
- Note that the default .vcd file name will be tb_<module name>.vcd.
- Once the .vcd file has been generated, we can finally view the output using gtkwave: `gtkwave tb_good_counter.vcd`
![gtkwave counter](https://user-images.githubusercontent.com/92947276/166216505-5ff65b2b-64b1-4bc5-af42-8e4006b9005d.PNG)

## Yosys
- We will be using Yosys for the synthesis of our Verilog designs. It will convert our RTL design to a gate-level netlist.
- Yosys uses a synthesis script to read a design from a Verilog file, synthesizes it to a gate-level netlist using the cell library and writes the synthesized results as a Verilog netlist. The synthesis script will be written by the user on the terminal.
- 
