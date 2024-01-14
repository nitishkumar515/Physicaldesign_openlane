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
magic -T /home/nitish/OpenLane/vsdstdcelldesign/libs/sky130A.tech lef read tmp/merged.nom.lef def read results/floorplan/picorv32a.def &

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
magic -T /home/nitish/OpenLane/vsdstdcelldesign/libs/sky130A.tech lef read tmp/merged.nom.lef def read results/floorplan/picorv32a.def &

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
  - PnR is a iterative flow and hence, we can make changes to the environment variables in the fly to observe the changes in our design. 
 - Let us say If I want to change my pin configuration along the core from equvi distance randomly placed to someother placement, we just set that IO mode variable on command prompt as shown below
 ```
 set ::env(FP_IO_MODE) 2
```
### SPICE Deck Creation and Simulation for CMOS inverter
- Before performing a SPICE simulation we need to create SPICE Deck
SPICE Deck provides information about the following:
- Component connectivity - Connectivity of the Vdd, Vss,Vin, substrate. Substrate tunes the threshold voltage of the MOS.
- component values - values of PMOS and NMOS, Output load, Input Gate Voltage, supply voltage.
- Node Identification and naming - Nodes are required to define the SPICE Netlist
     For example ```M1 out in vdd vdd pmos w = 0.375u L = 0.25u``` , ```cload out 0 10f```
- Simulation commands
- Model file - information of parameters related to transistors
Simulation of CMOS using different width and lengths. From the waveform, irrespective of switching the shape of it are almost same.
![Screenshot from 2023-09-18 09-53-18](https://github.com/nitishkumar515/Physicaldesign_openlane/assets/140998638/c79e51d9-0c4d-4850-8f2e-d8969c4d615f)

From the waveform we can see the characteristics are maintained  across all sizes of CMOS. So CMOS as a circuit is a robust device hence use in designing of logic gates. Parameters that define the robustness of the CMOS are

### Switching Threshold Vm

- The Switching Threshold of a CMOS inverter is the point where the Vin = Vout on the DC Transfer characreristics. 
- At this point, both the transistors are in saturation region, means both are turned on and have high chances of current flowing driectly from VDD to Ground called Leakage current.

![Screenshot from 2023-09-18 09-54-07](https://github.com/nitishkumar515/Physicaldesign_openlane/assets/140998638/3eab8885-134f-4fc4-a504-08bcf2f7028b)
Through transient analysis, we calculate the rise and fall delays of the CMOS by SPICE Simulation. As we know delays are calculated at 50% of the final values.


### Lab steps to git clone vsdstdcelldesign

- First, clone the required mag files and spicemodels of inverter,pmos and nmos sky130. The command to clone files from github link is:
```
git clone https://github.com/nickson-jose/vsdstdcelldesign.git
```
once I run this command, it will create ``vsdstdcelldesign`` folder in openlane directory.

Inorder to open the mag file and run magic go to the directory

For layout we run magic command

``` magic -T sky130A.tech sky130_inv.mag & ```

Ampersand at the end makes the next prompt line free, otherwise magic keeps the prompt line busy. Once we run the magic command we get the layout of the inverter in the magic window
![Screenshot from 2023-09-18 09-58-19](https://github.com/nitishkumar515/Physicaldesign_openlane/assets/140998638/6bc30570-1c08-4585-831a-60c1077b41dc)

</details>

<details>
  <summary>Inception of Layout and CMOS Fabrication Process
</summary>
  
### Mask CMOS Fabrication

The 16-mask CMOS (Complementary Metal-Oxide-Semiconductor) fabrication process involves several crucial steps for creating integrated circuits. Let's break it down with some jargon:

1. **Substrate Selection**:
   - In the initial phase, the appropriate semiconductor substrate is chosen.

2. **Active Region Creation**:
   - To isolate the active regions for transistors, the process begins with the deposition of SiO2 and Si3N4 layers, followed by photolithography and silicon nitride etching.
   - This is known as LOCOS (Local Oxidation of Silicon), where oxide is grown in certain regions.
   - Subsequently, Si3N4 is removed using hot phosphoric acid.
   
   3. **N-Well and P-Well Formation**:
   - The N-well and P-well regions are created separately.
   - P-well formation involves photolithography and ion implantation of p-type Boron material into the p-substrate.
   - N-well is formed similarly with n-type Phosphorus material.
   - High-temperature furnace processes drive-in diffusion to establish well depths, known as the tub process.

4. **Gate Formation**:
   - The gate is a pivotal CMOS transistor terminal that controls threshold voltages for transistor switching.
   - A polysilicon layer is deposited and photolithography techniques are applied to create NMOS and PMOS gates.
   - Important parameters for gate formation include oxide capacitance and doping concentration.

5. **Lightly Doped Drain (LDD) Formation**:
   - LDD is created to mitigate hot electron and short channel effects.

6. **Source & Drain Formation**:
   - Thin oxide layers are added to avoid channel effects during ion implantation.
   - N+ and P+ implants are performed using Arsenic implantation and high-temperature annealing.

7. **Local Interconnect Formation**:
   - Thin screen oxide is removed through etching in HF solution.
   - Titanium deposition through sputtering is initiated.
   - Heat treatment results in chemical reactions, producing low-resistant titanium silicon dioxide for interconnect contacts and titanium nitride for top-level connections, enabling local communication.

8. **Higher Level Metal Formation**:
  - To achieve suitable metal interconnects, non-planar surface topography is addressed.
   - Chemical Mechanical Polishing (CMP) is utilized by doping silicon oxide with Boron or Phosphorus to achieve surface planarization.
   - TiN and blanket Tungsten layers are deposited and subjected to CMP.
   - An aluminum (Al) layer is added and subjected to photolithography and CMP.
   - This constitutes the first level of interconnects, and additional interconnect layers are added to reach higher-level metal layers.

9. **Dielectric Layer Addition**:
   - Finally, a dielectric layer, typically Si3N4, is applied to safeguard the chip.

This complex process results in the creation of advanced integrated circuits with multiple layers of interconnects, essential for modern electronic devices.

![Screenshot from 2023-09-18 10-02-59](https://github.com/nitishkumar515/Physicaldesign_openlane/assets/140998638/3031b63e-63ba-4de2-ae61-3895d1f73379)


### SKY130 basic layer layout and LEF using inverter

- From Layout, we see the layers which are required for CMOS inverter. Inverter is, PMOS and NMOS connected together.
- Gates of both PMOS and NMOS are connected together and fed to input(here ,A), NMOS source connected to ground(here, VGND), PMOS source is connected to VDD(here, VPWR), Drains of PMOS and NMOS are connected together and fed to output(here, Y). 
The First layer in skywater130 is ``localinterconnect layer(locali)`` , above that metal 1 is purple color and metal 2 is pink color.
If you want to see connections between two different parts, place the cursor over that area and press S one times. The tkson window gives the component name.
![Screenshot from 2023-09-18 10-03-53](https://github.com/nitishkumar515/Physicaldesign_openlane/assets/140998638/b2dbf43b-db87-4c52-89d9-188113e984f4)

### Library exchange format (.lef)

- The layout of a design is defined in a specific file called LEF.
-  It includes design rules (tech LEF) and abstract information about the cells. 
    -  ```Tech LEF``` -  Technology LEF file contains information about the Metal layer, Via Definition and DRCs.
    -  ```Macro LEF``` -  Contains physical information of the cell such as its Size, Pin, their direction.
 
### Designing standard cell and SPICE extraction in MAGIC 

-  First we need to provide bounding box width and height in tkson window. lets say that width of BBOX is 1.38u and height is 2.72u. The command to give these values to magic is
   ``` property Fixed BBOX (0 0 1.32 2.72)  ```
- After this, Vdd, GND segments which are in metal 1 layer, their respective contacts and atlast logic gates layout is defined
Inorder to know the logical functioning of the inverter, we extract the spice and then we do simulation on the spice. To extract it on spice we open TKCON window, the steps are
- Know the present directory - ``pwd ``
- create an extration file -  the command is  `` extract all `` and  ``sky130_inv.ext`` files has been created
          
- create spice file using .ext file to be used with our ngspice tool  - the commands are  
      ``` ext2spice cthresh 0 rthresh 0 ``` - extracts parasatic capcitances also since these are actual layers - nothing is created in the folder
      ``` ext2spice ``` - a file ```sky130_inv.spice``` has been created.
![Screenshot from 2023-09-18 10-05-57](https://github.com/nitishkumar515/Physicaldesign_openlane/assets/140998638/bf01c0c6-0576-4382-a1b7-65fa17bb1096)
</details>

<details>
  <summary> SKY130 Tech File Labs </summary>
  
### Create Final SPICE Deck

let us see what is inside the spice Deck
In the spice file subcircuit(subckt), pmos and nmos node connections are defined
   
For NMOS  ``` XO Y A VGND VGND sky130_fd_pr_nfet_01v8 ``` . The order is  ``` Cell_name Drain Gate Source Substrate model_name ``` .
For PMOS  ``` X1 Y A VPWR VPWR sky130_fd_pr_pfet_01v8 ``` . The order is   ``` cell_name Drain Gate Source Substrate model_name ```.
   
For transient anaylsis, we would like to define these following connections and extra nodes for these in spice file
  - VGND to VSS
  - Supply voltage from VPWR to Ground - extra nodes here will be 0 and VDD with a value of 3.3v 
  - sweep in/pulse between A pin and VGND (0)
Before, editing the file, make sure scaling is proper, we measure the value of the gride size from the magic layout and define using `` .option scale=0.01u`` in the Deck file.

Now keeping the connection in mind, define the required commands in the file. Along with this we need to include libs for nmos ``nshort.lib`` and pmos ``pshort.lib`` and define transient analysis commands too. We comment the subckt since we are trying to input the controls and transient analysis also. Model names are changed to ``nshort_model.0`` and ``pshort_model.0`` according to the libs of nmos and pmos.
  
These voltage sources and simulation commands are defined in the Deck file.

   ``
.include ./libs/pshort.lib
.include ./libs/nshort.lib
   VDD VPWR 0 3.3V
   VSS VGND 0 0V
   Va A VGND PULSE(0V 3.3V 0 0.1ns 0.1ns 2ns 4ns)
   .tran 1n 20n
   .control
   run
   .endc
   .end
   ``
   ![fig]()

   ### Using ngspice for spice simulation
  
Spice Deck is done and now to run spice simulation invoke ngspice in the tool and pass the source file. 
 
  ``` ngspice sky130_inv.spice ```
  
On the prompt you can see the values the ngspice has taken. To see the plot, use
   
   ``` plot y vs time a ```
   ![figure]()

   ### Standard cell characterization of CMOS Iinverter 
 
characterization of the inverter standard cell depends on Four timing parameters
 
 **Rise Transition**: Time taken for the output to rise from 20% to 80% of max value
 **Fall Transition**: Time taken for the output to fall from 80% to 20% of max value
 **Cell Rise delay**: difference in time(50% output rise) to time(50% input fall)
 **Cell Fall delay**: difference in time(50% output fall) to time(50% input rise)
 
 The above timing parameters can be computed by noting down various values from the ngspice waveform.
 
 ``` Rise Transition : 2.25421 - 2.18636 = 0.006785 ns / 67.85ps ```
 ``` Fall Transitio : 4.09605 - 4.05554 = 0.04051ns/40.51ps ```
 ```Cell Rise Delay : 2.21701 - 2.14989 = 0.06689ns/66.89ps ```
 ```Cell Fall Delay : 4.07816 - 4.05011 = 0.02805ns/28.05ps ```

 ### LAB exercise and DRC Challenges

### Intrdocution of Magic and Skywater DRC's

  - In-depth overview of Magic's DRC engine
  - Introduction to Google/Skywater DRC rules
  - Lab : Warm-up exercise : Fixing a simple rule error
  - Lab : Main exercie : Fixing or create a complex error

 ### Sky130s pdk intro and Steps to download labs
  
  - setup to view the layouts
  - For extracting and generating views, Google/skywater repo files were built with Magic
  - Technology file dependency is more for any layout. hence, this file is created first.
  - Since, Pdk is still under development, there are some unfinished tech files and these are packaged for magic along with lab exercise layout and bunch of stuff into the tar ball

 
We can download the packaged files from web using ``wget `` command. wget stands for web get, a non-interactive file downloader command.
  
  ``` wget http://opencircuitdesign.com/open_pdks/archive/drc_tests.tgz```
  

  ![fig]()

once extraction is done, drc_tests file is created and you will have all the information about magic layout for this lab exercise

Now run MAGIC

For better graphics use command ``magic -d XR ``

Now, lets see an example of simple failing set of rules of metal 1 layer.  you can either run this by magic command line `` magic -d XR met1.mag `` or from the magic console window, `` menu - file - open -load file9here, met1.mag) ``

![fig]()

We use following commands to see metal cut as shown.
```
cif see VIA2

```
![fig]()

## Load Sky130 tech rules for drc challenges 

First load the poly file by ``load poly.mag`` on tkcon window.

Finding the error by mouse cursor and find the box area, Poly.9 is violated due to spacing between polyres and poly.

![fig]()

We find that distance between regular polysilicon & poly resistor should be 22um but it is showing 17um and still no errors . We should go to sky130A.tech file and modify as follows to detect this error.

In line

```
spacing npres *nsd 480 touching_illegal \
	"poly.resistor spacing to N-tap < %d (poly.9)"
```
change to

```
spacing npres allpolynonres 480 touching_illegal \
	"poly.resistor spacing to N-tap < %d (poly.9)"
```
Also,
```
spacing xhrpoly,uhrpoly,xpc alldiff 480 touching_illegal \

	"xhrpoly/uhrpoly resistor spacing to diffusion < %d (poly.9)"
```

change to 

```
spacing xhrpoly,uhrpoly,xpc allpolynonres 480 touching_illegal \

	"xhrpoly/uhrpoly resistor spacing to diffusion < %d (poly.9)"

```
![fig]()


</details>
   
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

![Screenshot from 2023-09-17 21-55-47](https://github.com/nitishkumar515/Physicaldesign_openlane/assets/140998638/521671d2-9b35-441e-a9e2-759efd267275)

To ensure that ports lie on the intersection point, the grid spacing in Magic (tkcon) must be changed to the li1 X and li1 Y values. After providing the command, we have following:

```
grid 0.46um 0.34um 0.23um 0.17um

```
![Screenshot from 2023-09-17 21-52-47](https://github.com/nitishkumar515/Physicaldesign_openlane/assets/140998638/7e627b59-5b2d-41c1-8d1d-2bf4c5f149a2)

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

![Screenshot from 2023-09-19 01-27-36](https://github.com/nitishkumar515/Physicaldesign_openlane/assets/140998638/f0bda061-b22d-483c-b0ad-b5e3fc745f5e)


After placement, we check for legality & To check the layout invoke magic from the results/placement directory:

```
magic -T home/nitish/OpenLane/vsdstdcelldesign/libs/sky130A.tech lef read tmp/merged.nom.lef def read results/floorplan/picorv32a.def &

```
![Screenshot from 2023-09-18 23-03-40](https://github.com/nitishkumar515/Physicaldesign_openlane/assets/140998638/0d7ba018-a80d-44c6-aa0d-d3a2ef687cce)



</details>

<details>
	<summary> Post-synthesis timing analysis Using OpenSTA </summary>

### Clock Tree Synthesis using Tritoncts

- Clock tree synthesis (CTS) can be implemented in various ways, and the choice of the specific technique depends on the design requirements, constraints, and goals. Here are some different types or approaches to clock tree synthesis:

- Balanced Tree CTS: In a balanced tree CTS, the clock signal is distributed in a balanced manner, often resembling a binary tree structure. This approach aims to provide roughly equal path lengths to all clock sinks (flip-flops) to minimize clock skew. It's relatively straightforward to implement and analyze but may not be the most power-efficient solution.

- H-tree CTS: An H-tree CTS uses a hierarchical tree structure, resembling the letter "H." It is particularly effective for distributing clock signals across large chip areas. The hierarchical structure can help reduce clock skew and optimize power consumption.

- Star CTS: In a star CTS, the clock signal is distributed from a single central point (like a star) to all the flip-flops. This approach simplifies clock distribution and minimizes clock skew but may require a higher number of buffers near the source.

- Global-Local CTS: Global-Local CTS is a hybrid approach that combines elements of both star and tree topologies. The global clock tree distributes the clock signal to major clock domains, while local trees within each domain further distribute the clock. This approach balances between global and local optimization, addressing both chip-wide and domain-specific clocking requirements.

- Mesh CTS: In a mesh CTS, clock wires are arranged in a mesh-like grid pattern, and each flip-flop is connected to the nearest available clock wire. It is often used in highly regular and structured designs, such as memory arrays. Mesh CTS can offer a balance between simplicity and skew minimization.

- Adaptive CTS: Adaptive CTS techniques adjust the clock tree structure dynamically based on the timing and congestion constraints of the design. This approach allows for greater flexibility and adaptability in meeting design goals but may be more complex to implement.

- crosstalk in VLSI:
Impact: Crosstalk is a significant concern in VLSI design due to the high integration density of components on a chip. Uncontrolled crosstalk can lead to data corruption, timing violations, and increased power consumption. Mitigation: VLSI designers employ various techniques to mitigate crosstalk, such as optimizing layout and routing, using appropriate shielding, implementing proper clock distribution strategies, and utilizing clock gating to reduce dynamic power consumption when logic is idle

Clock Net Shielding in VLSI:
Purpose: In VLSI circuits, the clock distribution network is crucial for synchronous operation. Clock signals must reach all parts of the chip while minimizing skew and maintaining signal integrity. Shielding Techniques: VLSI designers may use shielding techniques to isolate the clock network from other signals, reducing the risk of interference. This can include dedicated clock routing layers, clock tree synthesis algorithms, and buffer in- sertion to manage clock distribution more effectively. Clock Domain Isolation: VLSI designs often have multiple clock domains. Shielding and proper clock gating help ensure that clock signals do not propagate between domains, avoiding metastability issues and maintaining synchronization.



    
</details>

## Day 5

<details>
	<summary>Final steps in RTL to GDS</summary>
	
 ## Maze Routing (Lee's algorithm)

 Routing is the process of finding best possible route to connect source and target or between two points. Maze algorithm is desinged for routing which is developed by Lee.
The Lee algorithm starts with two designated points, the source and target, and leverages the routing grid to identify the shortest or optimal route between them. As shown in figures below.

![Screenshot from 2023-09-18 14-52-33](https://github.com/nitishkumar515/Physicaldesign_openlane/assets/140998638/b50ee03e-14ec-49a0-acab-04b7c3b71aec)
![Screenshot from 2023-09-18 14-51-16](https://github.com/nitishkumar515/Physicaldesign_openlane/assets/140998638/3ac6d573-cd7e-492a-b4db-91f0c563c6f4)


In first figure path between source to target has two bend and in second figure has one bend so least bend path is preferable in routing.

## Design Rule Check (DRC)

DRC checking is an essential part of the physical design flow and ensures the design meets manufacturing requirements and will not result in a chip failure. It defines the Quality of chip. They are so many DRCs, let us see few of them.
DRC verifies whether a design meets the predefined process technology rules given by the foundry for its manufacturing.
typical design rule for physical wires
- Minimun width of the wires
- Minimun spacing between wires
  To solve signal short violation ``` Minimum width of the wire ``` ``` Minimun spacing between wires ``` should be maintained as per Photolithography rules or requirement.
  
  ![Screenshot from 2023-09-18 15-42-09](https://github.com/nitishkumar515/Physicaldesign_openlane/assets/140998638/6874bfda-d02f-484d-bdd6-1fbdf8dd32e8)

The upper metal layer  should be wider than the lower metal layer to solve signal short violations as shown in figure below.

![Screenshot from 2023-09-18 15-44-21](https://github.com/nitishkumar515/Physicaldesign_openlane/assets/140998638/bde428b0-2b13-4851-8285-d21c2f32de71)

</details>
<details>
	<summary>Power Distribution Network generation</summary>
 - Unlike the general ASIC flow, Power Distribution Network generation is not a part of floorplan run in OpenLANE. PDN must be generated after CTS and post-CTS STA analyses:
- We can check whether PDN has been created or no by check the current def environment variable: 
```echo	$::env(CURRENT_DEF)
```
```
gen_pdn
```
- gen_pdn Generates the power distribution network.

- The power distribution network has to take the design_cts.def as the input def file.

- Power rings,strapes and rails are created by PDN.

- From VDD and VSS pads, power is drawn to power rings.

- Next, the horizontal and vertical strapes connected to rings draw the power from strapes.

- Stapes are connected to rings and these rings are connected to std cells. So, standard cells get power from rails.

- Here are definitions for the straps and the rails. In this design, straps are at metal layer 4 and 5 and the standard cell rails are at the metal layer 1. Vias connect accross the layers as required.

![Screenshot from 2023-09-19 00-44-09](https://github.com/nitishkumar515/Physicaldesign_openlane/assets/140998638/a222b798-23d3-425f-9f8a-9780cd42ab8e)

</details>
<details>
	<summary>Routing</summary>
	
Routing is divided into two distinct stages
- Global Routing : Global routing first partitions the routing region into tiles and decides tile-to-tile paths for all nets while attempting to optimize some given objective function. This task is accomplished by the "FASTE ROUTE" engine.
- Detailed Routing : The objective of detailed routing is to assign route segments of signal nets to specific routing tracks, vias, and metal layers in a manner consistent with given global routes of those nets.  This task is accomplished by the "tritonRoute" engine.

### Key Features of TritonRoute

- Initial Detail Routing
- Adherence to Pre-Processed Route Guides
- Adherence to Pre-Processed Route Guides
- Guide Splitting
- Guide Merging
- Guide Bridging

### TritonRoute problem statement

```
Inputs : LEF, DEF, Preprocessed route guides
Output : Detailed routing solution with optimized wire length and via count
Constraints : Route guide honoring, connectivity constraints and design rules.
```
The space where the detailed route takes place has been defined. Now TritonRoute handles the connectivity in two ways.
- Access Point(AP)
- Access Point Cluster(APC)
### TritonRoute run for routing
Make sure the CURRENT_DEF is set to pdn.def.
- Start routing by using

``` run_routing
```
![image](https://github.com/nitishkumar515/Physicaldesign_openlane/assets/140998638/27fdebe3-d375-4f8c-8587-f6cc8cd7ab28)

 
 </details>
 
## Word of Thanks
I sciencerly thank **Mr. Kunal Ghosh**(Founder/**VSD**) for helping me out to complete this flow smoothly.

## Acknowledgement
- Kunal Ghosh, VSD Corp. Pvt. Ltd.
- Alwin Shaju, IIITB
- Chatgpt
- Kanish R,Colleague,IIIT B
- Priyansh Pandya, IIITB

## Reference 
- https://www.vsdiat.com
- https://github.com/KanishR1
- https://github.com/bhargav-vlsi/Physical_design_open_lane
