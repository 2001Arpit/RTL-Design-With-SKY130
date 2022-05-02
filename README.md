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
  - [RTL](#rtl) 
  - [Icarus Verilog](#icarus-verilog)
  - [Yosys](#yosys)
- [Standard Cell Library](#standard-cell-library)
  - [Different Flavours of cells](#different-flavours-of-cells)
  - [PVT](#pvt)
- [Synthesis](#synthesis)
  - [Hierarchical Synthesis](#hierarchical-synthesis)
  - [Flat Synthesis](#flat-synthesis)
  - [Sub-module Level Synthesis](#sub-module-level-synthesis)
- [Flops](#flops)
  - [Glitch](#glitch)
  -  [Flop coding](#flop-coding)
  -  [Special Optimizations](#special-optimizations)
- [Optimisations](#optimisations) 
  - [Combinational Logic Optimisation](#combinational-logic-optimisation)
    - [Constant Propogation](#constant-propogation)
    - [Boolean Logic Optimisation](#boolean-logic-optimisation)
  - [Sequential Logic Optimisation](#sequential-logic-optimisation)
    - [Sequential Constant Propogation](#sequential-constant-propogation)
    - [State Optimisation](#state-optimisation)
    - [Retiming](#Retiming) 
    - [Cloning](#Cloning)
- [Gate Level Simulation](#gate-level-simulation)
  - [Incomplete sensitivity list](#Incomplete-sensitivity-list) 

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
  
- Sometimes a design contains multiple modules. These modules will have a hierarchy, with the top module being the one that calls the other modules. 
- Example: A full adder consisting 2 half adders. Here the half adder module will be instantiated within the full adder module twice.
  
- Lets consider another example where we are tryning to implement this logic:
  
![comb](https://user-images.githubusercontent.com/92947276/166233434-ecac37bf-7715-478f-96db-cbf06f30164a.PNG)

- The code is:
  
![multiple module code](https://user-images.githubusercontent.com/92947276/166233503-9c6953a0-2efd-4486-b2aa-ed9ce15e599f.PNG)

- It consists of 2 sub-modules, one for OR operation and another for AND operation. Both of them are being instantiated in the top module.
  
- We will now sythesise it using Yosys.
- First read the library file to import standard cells.
- Then read the verilog file.
- synthesise the design : `synth -top multiple modules`
  
![multiple module synth](https://user-images.githubusercontent.com/92947276/166234937-25c9105d-9703-480d-8aa0-641ca9011b90.PNG)
  
![multiple module synth2](https://user-images.githubusercontent.com/92947276/166235040-367130ed-9664-4a2f-bd95-67abe198e261.PNG)
  
- This will show you all the submodules in your design along with your design hierarchy. In addition, we can observe that the report mentions which cells will be used in the sub-modules.
 
- We will now map the standard cells to the design using `abc -liberty`.
- We are now ready to view the logic diagram:
  
![multiple modules show](https://user-images.githubusercontent.com/92947276/166235629-cd497e98-fdb6-4241-98cb-684cc1712698.PNG)
  
- Compared to the logical circuit we designed earlier, we can observe that instead of AND and OR cells directly, Yosys has decided to __preserve the hierarchy__.  
- When we write the netlist we can see that the hierarchy is preserved again.
- One interesting thing that can be obeserved from the netlist is that instead of using a OR gate, Yosys decided to use a NAND gate with inverted inputs.
  
![multiple module write verilog2](https://user-images.githubusercontent.com/92947276/166236851-49f47c71-f76e-48b0-bf6c-12518b96e52e.PNG)
  
- The reason for this is __NOT__ that NAND gates have considerably less area and power consumption as seen in the .lib file since the area of inverters combined with that of NAND gates is greater than that of an OR gate.
  
![comparisiom2](https://user-images.githubusercontent.com/92947276/166238457-c438d09a-3f45-4563-ba1a-42bacb9d8381.PNG)

- The real reason to choose NAND gates over OR gates is due to the layout of PMOS inside the gates. In practical applications, we do not prefer PMOS in series. This is because the W/L ratio of PMOS is more than NMOS and the mobility of holes in PMOS is also less than the mobility of electrons in NMOS; this causes an increase in resistance and delay in switching. Therefore we prefer to keep PMOS in parallel.
  
- OR gate:
- ![or-gate-cmos](https://user-images.githubusercontent.com/92947276/166240882-911a9524-73f7-4f16-b0b7-fcad54dfde63.jpg)
  
- NAND gate:
- ![nand](https://user-images.githubusercontent.com/92947276/166240910-a56aafc9-dabd-40fc-8488-3224f17d0e02.jpg)

## Flat Synthesis
- In flat synthesis hierarchy is not present (preserved).
- To do flat synthesis simply type `flatten` after mapping standard cells.
- View the logic diagram:
  
![multiple modules flat show](https://user-images.githubusercontent.com/92947276/166241938-075f8261-d847-4050-980d-f3fcf57a6903.PNG)

- We can compare from the previous diagram that the sub-modules are no longer present and have been replaced by standard cells.

## Sub-module Level Synthesis
- Sometimes we prefer sub-module level synthesis when we have too many instantiations of the same module or when we have a massive design with a large number of modules. In this case, we prefer to synthesise modules portion by portion so that the tool can write a more optimised netlist.
- We can do this in Yosys by simply using the command: `synth -top submodule1`.
- This will synthesise only the specified module and not the entire design.

# Flops
## Glitch
- A glitch is a small spike that happens at the output. It occurs due to the delays in gates.
- I have simulated the same combinational circuit discussed above, ie (a&b)|c, in modelsim with gates initialised with delays, here is the result:

![glitch code](https://user-images.githubusercontent.com/92947276/166246967-766f4067-6bce-4360-9e45-e49577137b3f.PNG)
![glitch](https://user-images.githubusercontent.com/92947276/166246994-68b80dd4-1ff0-4b90-85f6-6fbf242fc265.PNG)

- It can be seen that the output goes low even though the logic is equal to 1.
- This can be prevented by inserting a D-flip flop in between the combinational circuit as such:

![shield](https://user-images.githubusercontent.com/92947276/166247862-06a77a23-452c-40d4-b8a8-6b3e2c7c2033.PNG)

- This keeps the output at the flop stable, keeping the rest of the circuit stable. This process is called shielding. Q is shielded from changes in d due to clk.
  
## Flop coding
- We need to initialise the flop before use, or it will use garbage value.
- Initialising can be done by resetting it. There are two types of reset:
  - Asynchronous reset
  - Synchronous reset
- A flop can have either one or both types of reset, as shown in the code:
  
![all reset](https://user-images.githubusercontent.com/92947276/166249167-b370a658-6809-46cd-b9b1-92a057c567d7.PNG)
  
- A flop with Asynchronous reset can be reset at any time irrespevctive of clock edge, that is why we have put `(posedge clk, posedge reset)` in the sensitivity list.
- A __sensitivity list__ has all the inputs at which the always block will activate.

![reset async](https://user-images.githubusercontent.com/92947276/166250000-00170a66-7976-47dc-b2b3-add5a1830992.PNG)

- After simulating the design using Icarus Verilog, the following results were observed:
  
![reset async in action](https://user-images.githubusercontent.com/92947276/166250338-506beea0-4dc1-47c7-a41b-ed06fab66218.PNG)

- We can see that the reset is activating irrespective of the clock edge.
- For synthesising a designs with flops we need to map it using `dfflibmap -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib` followed by mapping with abc command.
- After synthesis we get the following logic diagram:
  
![reset async show](https://user-images.githubusercontent.com/92947276/166250663-a3568460-af72-46ca-992a-48a46cd7dfc2.PNG)

- A flop with synchronous reset only resets the flop when the reset is active at the clock edge. That is why the always block will only activate at the positive clock edge.
  
![reset sync](https://user-images.githubusercontent.com/92947276/166251481-3d0a9f23-244f-4a51-b872-95f762a9f839.PNG)

- In the above logic circuit we can see that the mux output will go low at reset = 1. But this reset will only be get in effect when the next positive clock edge comes.
- After simulation using Icarus Verilog we observe the following:
  
![reset sync in action](https://user-images.githubusercontent.com/92947276/166254675-9ce4915a-910a-411b-b88d-97c4855b43dd.PNG)

- We can see that the effect of reset is only reflected in the output at the next positive clock edge.
- A flop with both asynchronous and synchronous resets will reset at both the conditions.
  
![reset sync async](https://user-images.githubusercontent.com/92947276/166256307-500db418-5931-4ba5-8587-fb7753f384b9.PNG)

# Special Optimizations
- We will work with the following codes for this section:
  
![mult code](https://user-images.githubusercontent.com/92947276/166258722-843afe82-b3d0-4601-8c45-019be7292df1.PNG)

- If we look at the first code, it is simply taking a 3 bit input and multiplying it with 2 to get a 4 bit output. The truth table is as followed:

![TRUTH](https://user-images.githubusercontent.com/92947276/166260750-ad1eac61-a2d2-4b20-9e8e-ab5daa120fb7.PNG)

- We can observe that the output has shifted to the left by one bit. This simplifies the entire logic circuits to just a few wires.
- After `synth -top mult_2` command in Yosys we can observe that there are no cells:
  
![mult2 synth](https://user-images.githubusercontent.com/92947276/166261288-fe134456-c8cd-4692-8247-dd8acb493a89.PNG)
  
- After synthesis we get the following logic diagram and net list:
  
![mult2 show](https://user-images.githubusercontent.com/92947276/166261400-0a4dd02a-5d75-454e-b5df-0973de122a0f.PNG)

![mult2 net](https://user-images.githubusercontent.com/92947276/166261464-af589dbe-9532-47fb-ae5b-c570cc8892e1.PNG)

- In the second code we are multiplying the input with 9. This a similar case were we dont have to use any logic components for its synthesis.
- It is equivalent to:
  
![mult9](https://user-images.githubusercontent.com/92947276/166262854-18023687-7a85-4b6e-9b73-38caf26780d9.PNG)

- After synthesis we get the following logic diagram and netlist:
  
![mult9 show](https://user-images.githubusercontent.com/92947276/166263027-3f688503-3557-48d9-a0bb-18325718b659.PNG)
  
![mult9 net](https://user-images.githubusercontent.com/92947276/166263056-da34b199-e4dc-40e2-8c67-f7f0631acb0b.PNG)
  
- We can see that the required components have been reduced to just some wires again. 
  
# Optimisations
- Under optimisation, we try to reduce the components/ size of the logic circuit as much as possible to get the most optimised design
- This results in area and power consumption reduction.
- There are two types of optimisations:
  - Combinational logic optimisation
  - Sequential logic optimisation
  
## Combinational Logic Optimisation
- Some techniques used to optimise a combinational circuits are:
  - Constant Propogation
  - Boolean Logic Optimisation
  
### Constant Propogation:
- Consider the following logic: Y = ((AB)+C)', where A is grounded.
- The logic circuit of the following can be reduced as follows:
  
![gates opt eg](https://user-images.githubusercontent.com/92947276/166268426-fc905586-9f44-47a3-b804-4f8c1493ac89.PNG)
 
-This reduces the numbers of transistors in our design from 6 to 2 MOSFETs.
  
- Cosider the following code:
  
![check3 code](https://user-images.githubusercontent.com/92947276/166291937-3f53decf-aded-47c9-b539-ab72084accca.PNG)

- It is a code for a multiplexer circuit, lets see how to optimize it.
- After `synth -top opt_check3` use the command `opt_clean -purge` to optimize your design
- According to Yosys documentation:
  - opt_clean : This pass identifies wires and cells that are unused and removes them. Other
passes often remove cells but leave the wires in the design or reconnect the
wires but leave the old cells in the design. This pass can be used to clean up
after the passes that do the actual work.
  - -purge : also remove internal nets if they have a public name
- Here is a comparison between a non-optimised netlist and an optimised netlist:
  
![check3 no_opt netlist](https://user-images.githubusercontent.com/92947276/166292945-643c4a5d-2135-4bca-b784-da23ef1381d3.PNG)
![check3 opt netlist](https://user-images.githubusercontent.com/92947276/166292977-9461d141-499b-405a-bedd-1be32d07f6c4.PNG)


### Boolean Logic Optimisation
- Consider the following boolean logic: `a?(b?c:(c?a:0)):(!c)`
- It is one of many ways to design a multiplexer circuit in Verilog.

![mux comb opt](https://user-images.githubusercontent.com/92947276/166269532-21e1ce69-5e4f-44a3-b9fe-3c7fc12b014e.PNG)

- As you can see that the above circuit can be reduced to simple XOR gate.

## Sequential Logic Optimisation
- Some Techniques used to optimise Sequential logic are:
  - Sequential constant propogation
  - State optimisation
  - Retiming
  - Clonning
  
### Sequential Constant Propogation
- Consider the following circuit:
  
![seq](https://user-images.githubusercontent.com/92947276/166276517-220d57f8-4127-436c-847d-5047d51d3a6f.PNG)

- We can observe that no matter what value of reset or clock is given, the output will always be 1. This makes our entire circuit redundant. 

### State Optimisation
- It refers to the optimisation of unused states. We try to implement a state machine with the least number of states possible.
  
### Retiming
- It is the technique of shifting logic around to improve your maximum operational frequency.
  
### Cloning
- Consider the following circuit:

![state opt](https://user-images.githubusercontent.com/92947276/166289082-f75c1867-e5d3-47e5-8998-0d012601280e.PNG)

- Assume we have ample positive slack at A, which means that data can reach A a little later, and flop C does not meet setup time due to the large distance from A. To solve this issue, we can duplicate (clone) A to meet the timing of C as shown below:
  
![state opt2](https://user-images.githubusercontent.com/92947276/166289815-69c1c793-483c-4ee2-84ae-f2dbd9d8300b.PNG)

# Gate Level Simulation
- Gate level simulation or GLS is the simulation of the gate-level netlist obtained from Yosys. This simulation is done using iverilog.
- In addition to the netlist, we will also pass the standard cell libraries and testbench to iverilog.
- By doing this, we are making sure we don't have any Synthesis-Simulation mismatch, i.e. our RTL simulation should give the same results as our GLS.
- Synthesis-Simulation mismatch can happen due to the following reasons:
  - Incomplete sensitivity list
  - Blocking and non-blocking statements
  
## Incomplete sensitivity list
- We will explore this by taking the following example:
  
![mux codes](https://user-images.githubusercontent.com/92947276/166302494-6fbe933d-a87b-44a0-9437-13d8d1b1490c.PNG)
 
- In the code bad_mux.v, the sensitivity list only consists of `sel`. It means that the output will only change after `sel` has changed.
- This behavior can be observed in the RTL simulation:

![bad mux rtl simulation](https://user-images.githubusercontent.com/92947276/166303893-d6659698-99fc-44b7-8bfa-ea1508fdd58a.PNG)

- We can observe that the output is not changing when the input changes. This is incorrect behavior.
- Now lets compare these results to GLS.
- First write a net-list using Yosys.
- We will now use iverilog to conduct GLS. Type the command:
- `iverilog ../my_lib/verilog_model/primitives.v ../my_lib/verilog_model/sky130_fd_sc_hd.v bad_mux_net.v tb_ bad_mux.v`
- From here, the procedure for gtkwave is the same. It will produce the following result:
  
![bad mux gls simulation](https://user-images.githubusercontent.com/92947276/166304752-9d6ef78a-5943-4aa8-b7a4-5d4893ed95ae.PNG)

- We can observe that this is the correct behaviour of a 2:1 mux.
This is correct because the gate-level netlist is not concerned with the sensitivity list but only with the interconnections of various components.
- In the other two codes, we will not get any synthesis-simulation mismatch.
- Here is a comparision between the netlist of good and bad mux:

![good mux net](https://user-images.githubusercontent.com/92947276/166305853-584dca28-27b0-4674-8d5d-4ec6d702b9e4.PNG)
![bad mux net](https://user-images.githubusercontent.com/92947276/166305865-576870f5-9223-4314-8c20-8ec9c1eca43a.PNG)
  
- We can see that they are the same, thus proving that the netlist does not care about an incomplete sensitivity list.
- It should also be noted that Yosys will try to warn you if your design has an incomplete sensitivity list.

## Blocking and Non-Blocking Statements
- Blocking: represented by `=`. They execute statements in the order they are written.
- Non-Blocking: represented by `<=`. They execute all the RHS when always block is entered and assigns to LHS.
- These come into picture when using always block.
-Let us consider the following code to understand the significance of blocking and non-blocking statements:
                                    
![Screenshot (2)](https://user-images.githubusercontent.com/92947276/166312659-af13a87f-f7c1-4d78-aab9-8d3cfedc9139.png)
  
- Here is the logical circuit:
                                    
![blocking caveat diagram](https://user-images.githubusercontent.com/92947276/166312857-5d733608-536e-496c-af54-39d74aca24d5.PNG)
  
- We can see that the `x&c` is assigned to `y` before the value of `x` is resolved. Thus it will look at the previous value of `x`, which is the behaviour of a latch. This is undesirable in combinational circuits.
Let us look at the simulation result:
                                    
![blocking caveat gtk](https://user-images.githubusercontent.com/92947276/166313579-109ff9e9-e7d3-4efa-99fd-b2e2bf12717c.PNG)
  
- We can see the output is considering the past value of `x` to get its result.
- This gets resolved when we conduct GLS:
                                    
![blocking caveat gls gtk](https://user-images.githubusercontent.com/92947276/166314090-f379ab9b-2b76-4147-9ddd-ffbec0092dbc.PNG)
                                    
- The results are now appearing properly.                                   
                                      
# If Statements
- If statements in Verilog work similarly to if statements in any other language. They have priority, with the top one having the highest priority.
- If statements in Verilog must be placed inside always block.
- If statements are often used to realise multiplexer circuits. But we must be careful as incomplete if statements give rise to __infered__ latches.
- Take the following code to understand this problem:
                                    
![incomplete if code](https://user-images.githubusercontent.com/92947276/166316115-6c898931-57eb-4452-a195-2ea66e20ce8c.PNG)
                                   
- The select line defines only one input (when select is 1) in the first code. This leaves the other input (when select is 0) undefined. Therefore the synthesis tool considers it as preserving the previous value of output.
- When simulated we see the following result:
                                    
![incomplete if gtk](https://user-images.githubusercontent.com/92947276/166317658-5fe36db9-b105-450f-91fd-5a0f46d7572a.PNG)

- It is observed that when the select line (i0) goes low, the output latches to its previous value.
- Even the synthesis report lets us know that there is a latch present in the design:
                                    
![incomplete if synth](https://user-images.githubusercontent.com/92947276/166317997-25ef2f8d-e0e7-4d62-89f3-a488bfa282f1.PNG)
                                  
- A latch is also observed in the logic diagram:
                                    
![incomplete if show](https://user-images.githubusercontent.com/92947276/166318335-2fbade58-45b2-47b2-b80e-48eddf520c8e.PNG)
                                                                        
- In the second code, a similar occurence takes place. Here is what the code is trying to implement:                                    
  
![incomplete if2 circuit](https://user-images.githubusercontent.com/92947276/166318681-d265fe99-ff16-41c8-a295-4e2c887f5254.PNG)
  
- The simulation results are:
                                    
![incomplete if2 gtk latching](https://user-images.githubusercontent.com/92947276/166318820-331e9019-e97c-4dad-8ec1-48b9599adc90.PNG)
  
- It is observed that when both select lines go low, the output will latch to its previous value.
- After synthesis we get this logic diagram:
                                    
![incomplete if2 show](https://user-images.githubusercontent.com/92947276/166319287-2a52c0d3-89af-4f6a-b20e-e3c871794b33.PNG)
  
- For better understanding, here is the logic gate diagram of the same:

![incomplete if2 show circuit](https://user-images.githubusercontent.com/92947276/166319451-33e62e05-96ff-495e-be23-7192194b4813.PNG)
                                    
# Case Statements
- Case statements in Verilog are placed inside always block and must have an output register type variable.                                     
- One must be cautious when using case statements:
  - Incomplete cases cause inferred latches.
  - Partial assignments also cause inferred latches.
  - Overlapping cases give rise to unpredictable behaviour.                                   
  - Default statement does not guarantee the absence of inferred latches.        

## Incomplete Case                                    
- Consider the following codes:
                                    
![cases cose](https://user-images.githubusercontent.com/92947276/166322458-65beeb8c-f819-488f-b629-ff02c42f3067.PNG)
                                    
- In the first code, there is an attempt at creating a mux, But the code failed to define all possible values of `sel`.
- From the simulation it is observed that the output latches at sel = 10 (undefined):

![incomplete case gtk latching](https://user-images.githubusercontent.com/92947276/166323157-8c400bb1-af38-4bac-bcce-c75a662652c9.PNG)
                                    
- Synthesis result also show a latch:
                                    
![incomplete case show](https://user-images.githubusercontent.com/92947276/166323515-fdc4ab47-8d31-45fb-9fe9-dae53e5336cf.PNG)
                                    
- Here is a simplified version of the same:                                    
                                    
![incomplete case circuit](https://user-images.githubusercontent.com/92947276/166323596-01cadf43-cfbe-4e8f-b89f-b83b5f3bf0e5.PNG)
                                    
- In the second code we have fixed this issue by simply adding a default statement, which will handle any undefined case.
- Proper operation is verified from the simulation:                                    
                                    
![complete case gtk no latching](https://user-images.githubusercontent.com/92947276/166323941-3e756378-17e9-439f-bdbe-5174105e9bb6.PNG)
                                    
- At sel = 10, the output follows `i2` as defined by the default case.                                     
- Synthesis results in the following:                                    
                                    
![complete case show](https://user-images.githubusercontent.com/92947276/166324258-8c33f014-dc57-43cb-8523-f2435aad396e.PNG)
                                    
- Here is a simplified version of the same:                                    
                                    
![complete case circuit](https://user-images.githubusercontent.com/92947276/166324315-976b5885-1cec-43a9-9234-33f53bc39ccf.PNG)
                                    
## Partial Assignment
- The following code contains partial assignment:                                    
                                    
![partial case code](https://user-images.githubusercontent.com/92947276/166325032-a3328362-e16f-42dd-a0fe-5ca7cfddd30c.PNG)
                                    
- In case sel = 01, it has failed to give value to `x`.
- The simulation results are similar to incomplete case, here only `x` gets latched to its previous output at sel = 01. This is an example of 'default case do not prevent inferred latches'.                                 
- The synthesis produces the following:                                    
                                    
![partial case synth](https://user-images.githubusercontent.com/92947276/166325595-63a115d3-ece8-405f-94de-5aa7c277a772.PNG)
![partial case show](https://user-images.githubusercontent.com/92947276/166325646-be546234-7b34-4071-a4f8-c6bb02cf5fba.PNG)
                                                                       
- We can see in the reports that a latch has been generated.                                    
- Here is a simplified version of the logic diagram:                                    
                                    
![partial case circuit](https://user-images.githubusercontent.com/92947276/166326114-a03a8ab4-8638-4d01-8259-4f922ce31482.PNG)

## Overlapping cases                                   
- Consider the following code:                                    
                                    
![bad case code](https://user-images.githubusercontent.com/92947276/166326424-4adefdc6-b3a2-4856-93ad-dc3431000946.PNG)
                                    
- Here at sel = 10, it satisfies the 3rd and 4th cases. This overlap is undesirable and causes unpredictable behaviour as shown:
                                    
![bad case gtk](https://user-images.githubusercontent.com/92947276/166326630-356259be-b2c0-4ba5-8870-0e6e08bf4d95.PNG)
                                    
- Note that this is not the behaviour of an inferred latch as shown in the synthesis report (zero latches):                                     
                                    
![bad case synth](https://user-images.githubusercontent.com/92947276/166326959-8acee3ee-f253-47b1-bd1d-276d4ba6b55e.PNG)
                                    
- This is fixed by conducting a GLS:                                    
                                    
![bad case gls gtk](https://user-images.githubusercontent.com/92947276/166327067-7ac7d65c-e13b-4a3a-9233-73f65760cc05.PNG)
                                    
                                    
                                    
                                    
