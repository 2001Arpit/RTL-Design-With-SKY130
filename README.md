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

- We can verify the counter operation using this waveform. You can also change the colour of individual signals for convenience.

## Yosys
- We will be using Yosys for the synthesis of our Verilog designs. It will convert our RTL design to a gate-level netlist.
- Yosys uses a synthesis script to read a design from a Verilog file, synthesizes it to a gate-level netlist using the cell library and writes the synthesized results as a Verilog netlist. The synthesis script will be written by the user on the terminal.
- We will consider the same example as mentioned above.
- To start Yosys just type `yosys` in the terminal.
- We will start by reading our library files; this lets the synthesizer know what standard cells we are using: 
- 'read_liberty -lib ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib`

![read_liberty](https://user-images.githubusercontent.com/92947276/166224775-728c491c-3cd6-4166-9114-1b7023a02e80.PNG)
  
- According to Yosys documentation: 
  - read_liberty: Read cells from liberty file as modules into current design.
  - lib: Only create empty blackbox modules.
- Note that the address to the .lib file may differ.
- Now we will read our verilog file 
- `read_verilog good_counter.v`
- After reading the Verilog file, it will let you know if there are any errors or warnings.
- We will know synthesize the design:
- `synth -top good_counter`
- According to Yosys documentation:
  - synth: This command runs the default synthesis script. This command does not operate
on partly selected designs.
  - -top: Use the specified module as top module.
-Now we will map the design to a specified cell library (in our case sky130 library).
- `abc -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib`
- According to Yosys documentation:
  - abc: This pass uses the ABC tool for technology mapping of yosys's internal gate
library to a target architecture.
  - -liberty: generate netlists for the specified cell library (using the liberty
        file format).
- To view the resulting design we will simply type `show`.

![counter show](https://user-images.githubusercontent.com/92947276/166219871-a41b296f-e677-440a-9f00-bd5af0affb04.PNG)

- The various blocks present in the design are nothing but standard cells which are specified in the .lib file.
- One observation made during this workshop is that buffers are inserted in the design while synthesising combinational circuits, but that was not the case with sequential circuits. Here is an example of a mux:
  
![mux show](https://user-images.githubusercontent.com/92947276/166220568-2af6ca6d-1c36-4aef-909e-f693504c46ed.PNG)
  
- I believe that this is due to the .lib file we are using.

- Now we can write the netlist for the design. A netlist is a gate-level description of your design that specifies the components and their interconnections.
- `write_verilog -noattr good_counter_net.v`
- Use the -noattr to make the netlist easy to read. Here is a comparision between using and not using -noattr.
  
![mux write verilog noattr](https://user-images.githubusercontent.com/92947276/166221205-5e757817-dff0-4d21-b7b0-d3e50f4eb3b3.PNG) ![mux write verilog](https://user-images.githubusercontent.com/92947276/166221185-0c4e9f4c-0586-4cf4-b2e7-195bbe3eb1b3.PNG) 

# Standard cell Library
  
- sky130_fd_sc_hd__tt_025C_1v80.lib:
  -sky130: 130nm technology
  - tt   : typical
  - 025C : 25°C (operating temperature)
  - 1v80 : operating voltage
  
## Different Flavours of cells
- The .lib file is a collection of logical modules. It contains different 'flavours' of gates:
  - slow
  - fast 
- They refer to the logic gate's speed (propagation delay). We utilize different gates depending on our setup and hold requirements. For meeting setup time, we require faster cells, but we need slower cells to meet hold time.
- Setup and hold time creates a window around the clock edge where the data cannot be toggled, or it can cause metastability (uncertainty). 
  
![image](https://user-images.githubusercontent.com/92947276/166223384-4d4967e4-f105-4ba0-9a34-3509861a4aa7.png)
  
- In practical world setup analysis is used to improve performance and hold analysis is used to prevent data corruption. 

- What makes these cells fast and slow are:
  - Charging/ Discharging of capacitance.
  - Faster cells are wider, whereas slower cells are narrower.
  - This also makes the faster cells occupy more area and consume more power.
- We might also need to use slow cells due to the maximum operating frequency of the design. The maximum frequency is limited by the critical path (path whose propagation delay is the highest). 

## PVT
  - PVT stands for Process, Voltage, Temperature. It is a very common to see these three things once you open your library file.
  
  ![pvt](https://user-images.githubusercontent.com/92947276/166227047-64d166ba-5e0b-4198-8aaa-a0dcc8c5edaa.PNG)
  
- You can look foe specific cells in the file by: `:/<keyword>`
- Example: lets look for a a2111o cell. The function of this cell is ((a&b)|c|d|e).
- `:/a2111o`

![a2111o](https://user-images.githubusercontent.com/92947276/166228884-09b6cc07-f9b4-496a-8920-620f3eeffc75.PNG)

- It will specify leakage power, value and delay for all 32 combinations.
  
- We will now compare the different flavours of the same gate:
  
![comparisiom](https://user-images.githubusercontent.com/92947276/166231056-e30eb288-89d9-4053-9059-191737720721.PNG)
  
- We can observe that the leakage power and area increase (left to right). They are all and2 gates, but they all use a different width of transistors. As the width of the transistors increases, so does the area and power of the cell.
  
# Synthesis
## Hierarchical Synthesis
  
