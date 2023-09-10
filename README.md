# Physicaldesign_openlane
## Day-1


<details>
  <summary>Introduction</summary>
Physical Design or PnR (Place and Route) is the core of any IC design cycle.
From a RTL netlist to final tape-out, each phase of PnR brings it’s own challenges and surprises.
With the introduction of open-source technology for chip creation, many RTL designs and EDA Tools were made available for free.
The [SKY130 PDK] fills the gap in a whole Open source chip development.(https://skywater-pdk.readthedocs.io/en/latest/rules.html) from Skywater Technologies and Google.
There were a number of EDA Tools with distinct functions throughout the design cycle.
The design flow was not clear, and the Skywater pdk was only compatible with industrial equipment.
These problems were addressed by [OpenLane](https://github.com/The-OpenROAD-Project/OpenLane), which offered a fully automated and tidy RTL to GDSII flow. OpenLane is not a product; rather, it is a flow made up of a number of EDA tools, automation scripts, and Skywater-pdks that have been optimized for use with open-source EDA tools.
</details>
<details>
 <summary> Overall Design Flow</summary>
Register Transfer Level (RTL) is a representation of the digital circuit at the abstract level. There are two elements in digital circuits: Sequential Circuit (Flip-Flop) and Combinational Circuit (Gates), with the help of these two elements, a digital designer can implement any circuit, i.e., adder, multiplier, counter, memories, and state machines. An RTL design is created for a design specification using HDLs like Verilog or VHDL, or it can be created using high-level synthesis tools like SystemC, MATLAB HDL Coder, Bluespec, etc. A digital design engineer can represent their logic/functionality of the design in a simple text entry language.
The process of converting the RTL Netlist into a manufactured IC then starts, and is known as the Physical Design Flow. Floor planning, which entails placing preplaced cells, power planning, etc., comes first in the physical design process. The placement of logical synthesis comes next. So that the clock's skew is at a minimal or under the necessary threshold, we now perform CTS (Clock Tree Synthesis). Following CTS, all of the assembled components are routed. A process known as "Static Timing Analysis" is used between each and every step in the physical design flow, from logic synthesis through routing, to analyze the design at each stage and confirm that it is actually right. Magic is an open source application to view the layouts for every stage. You can extract a tiny netlist, run a SPICE simulation, and compare the results with the post-layout Simulation using ngspice.The digram of design flow is shown below.
![fig-1]()  
 </details>
 <details>
<summary> OpenLane Flow </summary>
 </details>
