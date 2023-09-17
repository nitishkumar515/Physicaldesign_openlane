# Physicaldesign_openlane
## Day-1

<details>
  <summary>Introduction</summary> 
  
* Physical Design or PnR (Place and Route) is the core of any IC design cycle.
* From a RTL netlist to final tape-out, each phase of PnR brings it’s own challenges and surprises.
* With the introduction of open-source technology for chip creation, many RTL designs and EDA Tools were made available for free.
- The [SKY130 PDK] fills the gap in a whole Open source chip development. from Skywater Technologies and Google.
- There were a number of EDA Tools with distinct functions throughout the design cycle.
- The design flow was not clear, and the Skywater pdk was only compatible with industrial equipment.
- These problems were addressed by [OpenLane](https://github.com/The-OpenROAD-Project/OpenLane), which offered a fully automated and tidy RTL to GDSII flow.
- OpenLane is not a product; rather, it is a flow made up of a number of EDA tools, automation scripts, and Skywater-pdks that have been optimized for use with open-source EDA tools.
</details>
<details>
 <summary> Overall Design Flow</summary>
  
- Register Transfer Level (RTL) is a representation of the digital circuit at the abstract level.
- There are two elements in digital circuits: Sequential Circuit (Flip-Flop) and Combinational Circuit (Gates), with the help of these two elements, a digital designer can implement any circuit, i.e., adder, multiplier, counter, memories, and state machines. An RTL design is created for a design specification using HDLs like Verilog or VHDL, or it can be created using high-level synthesis tools like SystemC, MATLAB HDL Coder, Bluespec, etc. A digital design engineer can represent their logic/functionality of the design in a simple text entry language.
- The process of converting the RTL Netlist into a manufactured IC then starts, and is known as the Physical Design Flow.
- Floor planning, which entails placing preplaced cells, power planning, etc., comes first in the physical design process.
- The placement of logical synthesis comes next. So that the clock's skew is at a minimal or under the necessary threshold, we now perform CTS (Clock Tree Synthesis). Following CTS, all of the assembled components are routed.
- A process known as "Static Timing Analysis" is used between each and every step in the physical design flow, from logic synthesis through routing, to analyze the design at each stage and confirm that it is actually right.
- Magic is an open source application to view the layouts for every stage. You can extract a tiny netlist, run a SPICE simulation, and compare the results with the post-layout Simulation using ngspice.The digram of design flow is shown below.
![fig-1](https://github.com/nitishkumar515/Physicaldesign_openlane/blob/main/fig100.png)  
 </details>
 <details>
<summary> OpenLane Flow </summary>
   
The block digram of openlane flow is shown below
![fig-2](https://github.com/nitishkumar515/Physicaldesign_openlane/blob/main/fig101.png)   

### 1.  Synthesis  
RTL synthesizer primary responsibility is to convert the code into the gate-level netlist. This is a automated process; a tool has all the standard libraries definitions that can manipulate the respective gate-level netlist, which is an equivalent of your design in RTL. Standard cells have regular layout each have different views/models. We use Yosys which is an Open Source Logic Synthesizer. Yosys takes the RTL design and timing .libs and verilog models of standard cells and converts  into  a  RTL Netlist. abc does the tehnology mapping to the required skywater-pdk variants 
### 1.1 Goals of Synthesis
To get a gate-level netlist.
Inserting clock gates.
Logic optimazations.
etc
### 1.2 Deign Exploration Utility 
This is used to suit the design configuration and generate reports with different metrics to select the best. This is also used for regression testing
### 1.3 Design For Test - DFT Insertion
It is used to test the design. Digital system verification and testing are progressively more important, as they become major contributors to the
manufacturing cost of a new IC product. 
###  2. Floor Planning and Power Planning
Floor planing is the starting step in ASIC physical design. Floor plan determines the size of the design cell (or die), creates the boundary and core area, and creates wire tracks for placement of standard cells. It is also a process of positioning blocks or macros on the die. This is done by OpenROAD flow. 
The following parameters are decided in the floor planning stage.
Die size, core size of the chip (rectangular or rectilinear)
I/O pad’s location
Plan for power
Row configuration
### 3. Placement
There are two types of placement.  The other required logic is placed optimally.
Placement is of two steps
- Global Placement- finds the optimal position for each cells. These positions are not necessarly correct, cells may overlap
- Detialed Placement - After Global placement is done minimal alterations are done to correct the issues
### 4. Clock Tree Synthesis 
To ensure minimum skew the Clock is routed optimally through the circuit using different algorithms. This is done in the OpenROAD flow. This is done by TritonCTS.
### 5. Fake Antenna and diode swapping
Long wires acts as antennas and cause accumulation of charges during the fabrication process damaging the transistor. To avoid this bridging is used to pass the wire through different layers or an antenna diode cell is added to leak away the charges

* OpenLane approach - Insert Fake Diode to every cell input during placement. This matches the footprint of the library of the antenna diode. The Antenna Checker is run to check for violations, if there are violations then the fake diode is swapped with a real one.
* OpenROAD approach - In the global route step, the antenna violation is addressed automatically by inserting an antenan diode OpenLane allows the user to chose either of the above approaches.
### 6. Routing
Implement the interconnect using the available metal layers. skywater130 PDK define 6 routing layers. There are two steps.
* Global Routing - This is done inside the OpenROAD flow (FastRoute)
* Detailed Routing - This is performed using TritonRoute outside the OpenROAD flow after the global routing. Before performing this step the Logic Equivalence Check is performed by Yosys, since OpenROAD does some optimisations the circuit.

### 7. RC Extraction
From the .def file, the parasitic extraction is done to generate the .spef file (Standard Prasitic Exchange Format) which produces an accurate analog model of the circuit by including the parasitic effects due to wires, parasitic capacitances, etc.
### 8. STA
At this stage again OpenSTA is used to perform the Static Timing Analysis.
### 9. Sign-off Steps
* Physical verifications
* Design Rule Check (DRC) is performed by Magic.
* Layout Versus Schematic (LVS) is performed by Netgen.
### 10. GDSII Extraction
The routed .def file is used my Magic to generate the GDSII file
## OpenLane Installation and Environment Setup
Refer to [Kanish R1 GIthub](https://github.com/KanishR1/Physical-Design-Using-Openlane) or [OpenLane build Script by Nikson Jose] for OpenLane installation and environment setup.If the installation is carried out on a Virtual Machine/Linux, the following repository can be used from reference **(https://github.com/nickson-jose/openlane_build_script)**

## Working with OpenLane

### Start Openlane
```
make mount
```
The terminal changes into the docker instance. Open the OpenLane in interactive mode.
```
./flow.tcl -interactive
```
Set the package required by OpenLane

```pakage require openlane 0.9```

![fig103](https://github.com/nitishkumar515/Physicaldesign_openlane/assets/140998638/2f1edda4-b21c-4bc5-a9c8-6bca4256513e)

## Synthesis

Run the synthesis
```run_synthesis```

OpenLane invokes the following

- `Yosys` - RTL Synthesis and maps to yosys generic cells
- `abc` - Technology mapping with the Skywater130 PDK. Here `sky130_fd_sc_hd` Skywater Foundry produced High density standard cells are used.
- `OpenSTA` - This does the Static Timing Analysis on the netlist generated after synthesis and generated the timing reports
  
View the synthesis statistics


![fig104](https://github.com/nitishkumar515/Physicaldesign_openlane/assets/140998638/a2a35d77-0d62-48be-977b-1bbf8b7f3bd5)

### Key concepts

#### Flops ratio 

- The flop ratio is defined as the ratio of the number of flops to the total number of cells
- Here flop ratio is **1596/10104 = 0.1579** (i.e: 15.8%) [From the synthesis statistics]
   
 </details>

### Day-2
<details>
  <summary>Chip Floor Planning Consideration</summary>
  
#### Utilisation Factor

- The ratio of area occupied by the cells in the netlist to the total area of the core
- Best practice is to set the utilisation factor less than 50% so that there will be space for optimisations, routing, inserting buffers etc.,

### Aspect Ratio

- Aspect ratio is the ratio of height to the width of the die.
- Aspect Ratio of 1 indicates that the die is a square die

## Floorplanning

Floorplanning involves the following stages

### Pre-Placed cells

- Whenever there is a complex logic which is repeated multiple times or a design given by a third-party it can be perceived as abstract black box with input and output ports, clocks etc ., 
- These modules can be either macros or IP
    - Macro  - It is a module such as CPU Core which are developed by the entity fabicating the chip
    - IP - It is an "Intellectual Propertly" which the entity fabricating the chip gets as a package from a third party or even packaged Hard IPs developed by the same entity. Common examples of IPs are SRAM, PLL, Protocol Converters etc.,

- These Macros and IPs are placed in the core at first before placing the standard cells  and power planning
- These are optimally such that the cells which are more connected to each other are placed nearby and oriented for input and ouputs

### Decoupling Capacitors to the pre placed cells

- The power lines can have some RLC component causing the voltage to drop at the node where it enters the Blocks or the ground of the cell can be at a higher potential than ideally 0V
- When this happens, there is a chance such that the logic transitions are not to the upper or lower noise margins but to the forbidden state causing the circuit to misbehave
- This is prevented by adding a capacitor in parallel with the power and ground node of the block such that the capacitor decouples the block from the power source whenever there is a logic transition

### Power Planning

- When there are several cells or blocks drawing power from the same power rail and sinking power to the same ground pin the following effects are observed
    - Whenever there is alogic transition from 1 to 0 in a large number of cells then there is a Voltage Droop in the power lines as Voltage Drops from Vdd
    - Whener there is a logic transition from 0 to 1 in a large number of cells simultaneously causes the ground potential to raise above 0V calles as Ground Bump
    - These effects pose a risk of driving the logic state out of the specified noise margin.
    - To avoid this the Vdd and Gnd are placed as a grid of horizontal and vertical tracks and the cell nearer to an intersection can tap power or sink power to the Vdd or Gnd intersection respectively

### Pin Placement
 - The input, output and Clock pins are placed optimally such that there is less complication in routing or optimised delay
 - There are different styles of pin placement in openlane like `random pin placement` , `uniformly spaced` etc.,

  </details>

  <details>

<summary>Floorplan run on OpenLANE & review layout in Magic</summary>

**Floorplan envrionment variables or switches:**
1. ```FP_CORE_UTIL``` - core utilization percentage
2. ```FP_ASPECT_RATIO``` - the cores aspect ratio
3. ```FP_CORE_MARGIN``` - The length of the margin surrounding the core area
4. ```FP_IO_MODE``` - defines pin configurations around the core(1 = randomly equidistant/0 = not equidistant)
5. ```FP_CORE_VMETAL``` - vertical metal layer where I/O pins are placed
6. ```FP_CORE_HMETAL``` - horizontal metal layer where I/O pins are placed
 
***Note: Usually, the parameter values for vertical metal layer and horizontal metal layer will be 1 more than that specified in the files***

**Importance files in increasing priority order:**
1. ```floorplan.tcl``` - System default settings
2. ```conifg.tcl```
3. ```sky130A_sky130_fd_sc_hd_config.tcl```
 
 To run the picorv32a floorplan in openLANE:
 
 ```
 run_floorplan
 
 ```
![runfloorplan](https://github.com/nitishkumar515/Physicaldesign_openlane/assets/140998638/33bdd81c-da77-4a8f-be26-349a2fdaf8eb)

Post the floorplan run, a .def file will have been created within the ```results/floorplan``` directory. We may review floorplan files by checking the ```floorplan.tcl.``` The system defaults will have been overriden by switches set in conifg.tcl and further overriden by switches set in ```sky130A_sky130_fd_sc_hd_config.tcl.```

To view the floorplan, Magic is invoked after moving to the results/floorplan directory:

![floorplandictonary](https://github.com/nitishkumar515/Physicaldesign_openlane/assets/140998638/22d7b055-3900-4c24-8741-c2ba7e932d92)


```
magic -T /home/parallels/OpenLane/vsdstdcelldesign/libs/sky130A.tech lef read tmp/merged.nom.lef def read results/floorplan/picorv32a.def &

```
![Screenshot from 2023-09-15 23-18-25](https://github.com/nitishkumar515/Physicaldesign_openlane/assets/140998638/eb11ff27-d562-4dd9-98f6-bc2b1a287794)

One can zoom into Magic layout by selecting an area with left and right mouse click followed by pressing "z" key.

Various components can be identified by using the what command in tkcon window after making a selection on the component.

Zooming in also provides a view of decaps present in picorv32a chip.

The standard cell can be found at the bottom left corner.

You can clearly see I/O pins, Decap cells and Tap cells. Tap cells are placed in a zig zag manner or you can say diagonally
</details>
<details>
  <summary>
    Library Binding and Placement
  </summary>
  
  ## Netlist Binding and initial place design

First we need to bind the netlist with physical cells. We have shapes for OR, AND and every cell for pratice purpose. But in reality we dont have such shapes, we have give an physical dimensions like rectangles or squares weight and width. This information is given in libs and lefs. Now we place these cells in our design by initilaising it. 

## Optimize Placement

The next step is placement. Once we initial the design, the logic cells in netlist in its physical dimisoins is placed on the floorplan. Placement is perfomed in 2 stages:

Global Placement: Cells will be placed randomly in optimal positions which may not be legal and cells may overlap. Optimization is done through reduction of half parameter wire length.
Detailed Placement: It alters the position of cells post global placement so as to legalise them.
Legalisation of cells is important from timing point of view.

Optimization is stage where we estimate the lenght and capictance, based on that we add buffers. Ideally, Optimization is done for better timing.

![Screenshot from 2023-09-15 23-33-01](https://github.com/nitishkumar515/Physicaldesign_openlane/assets/140998638/d9616758-71b0-4079-b213-2b44f985fa38)


## Congestion aware Placement 

Post placement, the design can be viewed on magic within results/placement directory:

```
magic -T /home/parallels/OpenLane/vsdstdcelldesign/libs/sky130A.tech lef read tmp/merged.nom.lef def read results/floorplan/picorv32a.def &

```
![Screenshot from 2023-09-15 23-34-29](https://github.com/nitishkumar515/Physicaldesign_openlane/assets/140998638/4f341418-9f36-4538-a5f7-33f243922b86)


**Note: Power distribution network generation is usually a part of the floorplan step. However, in the openLANE flow, floorplan does not generate PDN.  It is created after post CTS. The steps are - floorplan, placement, CTS, Post CTS and then PDN**

## Need for libraries and characterization

As we know, From logic synthesis to routing and STA, each and evry stage has one thing in common i.e., logic gates/ logic cells. In order for the tool understand these gates are and their timing, we need to characterize these cells. 

## CELL DESIGN AND CHARACETRIZATION FLOWS
Library is a place where we get information about every cell. It has differents cells with different size, functionality,threshold voltages. There is a typical cell design flow steps.
1. Inputs : PDKS(process design kit) : DRC & LVS, SPICE Models, library & user-defined specs.
2. Design Steps :Circuit design, Layout design (Art of layout Euler's path and stick diagram), Extraction of parasitics, Characterization (timing, noise, power).
3. Outputs: CDL (circuit description language), LEF, GDSII, extracted SPICE netlist (.cir), timing, noise and power .lib files

### Standard Cell Characterization Flow

A typical standard cell characterization flow that is followed in the industry includes the following steps:

1. Read in the models and tech files
2. Read extracted spice Netlist
3. Recognise behavior of the cells
4. Read the subcircuits
5. Attach power sources
6. Apply stimulus to characterization setup
7. Provide neccesary output capacitance loads
8. Provide neccesary simulation commands

Now all these 8 steps are fed in together as a configuration file to a characterization software called GUNA. This software generates timing, noise, power models.
These .libs are classified as Timing characterization, power characterization and noise characterization.

![Screenshot from 2023-09-15 23-35-45](https://github.com/nitishkumar515/Physicaldesign_openlane/assets/140998638/a6ba224d-08e3-4c7a-a7e5-5331484f0671)


### TIMING CHARACTERIZATION
In standard cell characterisation, One of the classification of libs is timing characterisation.

### Timing threshold definitions 
Timing defintion |	Value
-------------- | --------------
slew_low_rise_thr	| 20% value
slew_high_rise_thr | 80% value
slew_low_fall_thr |	20% value
slew_high_fall_thr |	80% value
in_rise_thr	| 50% value
in_fall_thr |	50% value
out_rise_thr |	50% value
out_fall_thr | 50% value

### Propagation Delay and Transition Time 

**Propagation Delay** 
The time difference between when the transitional input reaches 50% of its final value and when the output reaches 50% of its final value. Poor choice of threshold values lead to negative delay values. Even thought you have taken good threshold values, sometimes depending upon how good or bad the slew, the dealy might be still +ve or -ve.

```
Propagation delay = time(out_thr) - time(in_thr)
```
**Transition Time**

The time it takes the signal to move between states is the transition time , where the time is measured between 10% and 90% or 20% to 80% of the signal levels.

```
Rise transition time = time(slew_high_rise_thr) - time (slew_low_rise_thr)

Low transition time = time(slew_high_fall_thr) - time (slew_low_fall_thr)
```

</details>

### DAY3 Design Library Cell using ngspice simulations

<details>
  <summary>CMOS inverter ngspice simulations </summary>
  ``ngspice``  is opesoure engine where simulations are done.

  ### IO Placer revision

</details>

## DAY 4 

<details>

<summary> Timing Analysis and Clock Tree Synthesis (CTS) </summary>

### Standard Cell LEF generation
During Placement, entire mag information is not necessary. Only the PR boundary, I/O ports, Power and ground rails of the cell is required. This information is defined in LEF file.
The main objective is to extract lef from the mag file and plug into our design flow.

# Grid into Track info

 **Track** :A path or a line on which metal layers are drawn for routing. Track is used to define the height of the standard cell. 

To implement our own stdcell, few guidelines must be followed 
 - I/O ports must lie on the intersection on Horizontal and vertical tracks
 - Width and Height of standard cell are odd mutliples of Horizontal track pitch and Vertical track pitch

This information is defined in ``tracks.info``. 

```
li1 X 0.23 0.46 
li1 Y 0.17 0.34
```

before grid on:

![Screenshot from 2023-09-17 21-52-47](https://github.com/nitishkumar515/Physicaldesign_openlane/assets/140998638/7e627b59-5b2d-41c1-8d1d-2bf4c5f149a2)

To ensure that ports lie on the intersection point, the grid spacing in Magic (tkcon) must be changed to the li1 X and li1 Y values. After providing the command, we have following:

```
grid 0.46um 0.34um 0.23um 0.17um

```
![Screenshot from 2023-09-17 21-55-47](https://github.com/nitishkumar515/Physicaldesign_openlane/assets/140998638/521671d2-9b35-441e-a9e2-759efd267275)

### Create Port Definition: 

However, certain properties and definitions need to be set to the pins of the cell. For LEF files, a cell that contains ports is written as a macro cell, and the ports are the declared as PINs of the macro.

The way to define a port is through Magic console and following are the steps:
- In Magic Layout window, first source the .mag file for the design (here inverter). Then Edit >> Text which opens up a dialogue box.
- When you double press S at the I/O lables, the text automatically takes the string name and size. Ensure the Port enable checkbox is checked and default checkbox is unchecked as shown in the figure:

![Screenshot from 2023-09-17 21-59-22](https://github.com/nitishkumar515/Physicaldesign_openlane/assets/140998638/123afff8-44aa-4157-b433-a045994f2c04)

- In the above figure, The number in the textarea near enable checkbox defines the order in which the ports will be written in LEF file (0 being the first).

-  For power and ground layers, the definition could be same or different than the signal layer. Here, ground and power connectivity are taken from metal1

### Set port class and port use attributes for layout 
fter defining ports, the next step is setting port class and port use attributes.

Select port A in magic:
```
port class input
port use signal
```
Select Y area
```
port class output
port use signal
```
Select VPWR area
```
port class inout
port use power
```
Select VGND area
```
port class inout
port use ground

```
### Custom cell naming and lef extraction.

Name the custom cell through tkcon window as ```sky130_vsdinv.mag```.

We generate lef file by command:

```
lef write

```
This generates sky130_vsdinv.lef file.
![Screenshot from 2023-09-17 22-03-02](https://github.com/nitishkumar515/Physicaldesign_openlane/assets/140998638/160bc150-144c-49d3-9738-fb3e7fd85655)
### Steps to include custom cell in ASIC design

We have created a custom standard cell in previous steps of an inverter. Copy lef file, sky130_fd_sc_hd_typical.lib, sky130_fd_sc_hd_slow.lib & sky130_fd_sc_hd_fast.lib to src folder of picorv32a from libs folder vsdstdcelldesign. Then modify the config.tcl as follows.

```

# Design
set ::env(DESIGN_NAME) "picorv32a"

set ::env(VERILOG_FILES) "$::env(DESIGN_DIR)/src/picorv32a.v"

set ::env(CLOCK_PORT) "clk"
set ::env(CLOCK_NET) $::env(CLOCK_PORT)

set ::env(GLB_RESIZER_TIMING_OPTIMIZATIONS) {1}

set ::env(LIB_SYNTH) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__typical.lib"
set ::env(LIB_SLOWEST) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__slow.lib"
set ::env(LIB_FASTEST) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__fast.lib"
set ::env(LIB_TYPICAL) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__typical.lib"

set ::env(EXTRA_LEFS) [glob $::env(OPENLANE_ROOT)/designs/$::env(DESIGN_NAME)/src/*.lef]

set filename $::env(DESIGN_DIR)/$::env(PDK)_$::env(STD_CELL_LIBRARY)_config.tcl
if { [file exists $filename] == 1} {
	source $filename
}

```
To integrate standard cell in openlane flow after `` make mount `` , perform following commands:

```
prep -design picorv32a -tag RUN_2023.09.09_20.37.18 -overwrite 
set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
add_lefs -src $lefs
run_synthesis

```
synthesis report :
![Screenshot from 2023-09-17 22-53-56](https://github.com/nitishkumar515/Physicaldesign_openlane/assets/140998638/42498c87-fcf2-4ac1-8597-0054a13de2b3)

sta report:

![Screenshot from 2023-09-17 22-57-26](https://github.com/nitishkumar515/Physicaldesign_openlane/assets/140998638/300ea7c5-2086-4afd-a08f-e20fd42e2c4c)








### Delay Tables

Basically, Delay is a parameter that has huge impact on our cells in the design. Delay decides each and every other factor in timing. 
For a cell with different size, threshold voltages, delay model table is created where we can it as timing table.
```Delay of a cell depends on input transition and out load```. 
Lets say two scenarios, 
we have long wire and the cell(X1) is sitting at the end of the wire : the delay of this cell will be different because of the bad transition that caused due to the resistance and capcitances on the long wire.
we have the same cell sitting at the end of the short wire: the delay of this will be different since the tarn is not that bad comapred to the earlier scenario.
Eventhough both are same cells, depending upon the input tran, the delay got chaned. Same goes with o/p load also.

VLSI engineers have identified specific constraints when inserting buffers to preserve signal integrity. They've noticed that each buffer level must maintain consistent sizing, but their delays can vary depending on the load they drive. To address this, they introduced the concept of "delay tables," which essentially consist of 2D arrays containing values for input slew and load capacitance, each associated with different buffer sizes. These tables serve as timing models for the design.

When the algorithm works with these delay tables, it utilizes the provided input slew and load capacitance values to compute the corresponding delay values for the buffers. In cases where the precise delay data is not readily available, the algorithm employs a technique of interpolation to determine the closest available data points and extrapolates from them to estimate the required delay values.

![Screenshot from 2023-09-17 22-13-36](https://github.com/nitishkumar515/Physicaldesign_openlane/assets/140998638/a8593786-ca47-494f-9b97-d588683e65dd)

### Openlane steps with custom standard cell

We perform synthesis and found that it has positive slack and met timing constraints.

During Floorplan,``` 504 endcaps, 6731 tapcells ``` got placed. Design has 275 original rows

Now ``` run_placement```

After placement, we check for legality &To check the layout invoke magic from the results/placement directory:

```
magic -T /home/parallels/OpenLane/vsdstdcelldesign/libs/sky130A.tech lef read tmp/merged.nom.lef def read results/floorplan/picorv32a.def &

```
![Screenshot from 2023-09-17 23-32-38](https://github.com/nitishkumar515/Physicaldesign_openlane/assets/140998638/5a41c5cd-1fba-4a69-871e-06322307db6e)


</details>

<details>
	<summary> Post-synthesis timing analysis Using OpenSTA </summary>

Timing analysis is carried out outside the openLANE flow using OpenSTA tool. For this, ```pre_sta.conf``` is required to carry out the STA analysis. Invoke OpenSTA outside the openLANE flow as follows:
 
```
sta pre_sta.conf
```

sdc file for OpenSTA is modified like this:



    








 
 </details>
