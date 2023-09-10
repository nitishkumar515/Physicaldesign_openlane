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
![fig-1]()  
 </details>
 <details>
<summary> OpenLane Flow </summary>
   
The block digram of openlane flow is shown below
![fig-2]()   

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

## Synthesis

Run the synthesis
```run_synthesis```

OpenLane invokes the following

- `Yosys` - RTL Synthesis and maps to yosys generic cells
- `abc` - Technology mapping with the Skywater130 PDK. Here `sky130_fd_sc_hd` Skywater Foundry produced High density standard cells are used.
- `OpenSTA` - This does the Static Timing Analysis on the netlist generated after synthesis and generated the timing reports 

View the synthesis statistics

![fig-3]()


### Key concepts

#### Flops ratio 

- The flop ratio is defined as the ratio of the number of flops to the total number of cells
- Here flop ratio is **1596/10104 = 0.1579** (i.e: 15.8%) [From the synthesis statistics]
   
 </details>

### Day-2
<details>
  <summary>Chip Floor Planning Consideration</summary>
  
#### Utilisation Factor
    
 </details>
