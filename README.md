# OpenLANE_sky_130
This repository contains a comprehensive list of notes from the 5 day OpenLANE sky 130 workshop. 
# Table of Contents
- [Day 1 - Intro to open source EDA, SKY130 PDK, and OpenLane](#day-1---intro-to-open-source-eda-sky130-pdk-and-openlane)
  - i.

# Day 1 - Intro to open source EDA, SKY130 PDK, and OpenLane
## Introductory IC vocabulary
- Package: Material that surrounds and protects the IC from physical damage. It also allows for the mounting of the IC to the PCB. A package that many people are familiar with is the black epoxy or plastic cases surrounding common ICs. 
- Die: The piece of semiconducting material, usually silicon, that the chip is manufactured on.
- PADS: The metallic points of contact where signals enter and exit the die.
- Core: The functional area of the IC (like its brain). This is where the digital circuitry exists. It excludes all of the IO ring around it and PADS.
- Macro: A pre-designed functional block that may require analog parts
- Foundry IPs: Blocks that contain special techniques to manufacture, and as a result, can only be made at foundries.

## Simplified RTL to GDSII Design Flow
- Synthesis: RTL is converted into a circuit of gate logic (from the standard cell library) described by an HDL
- Floor & Power Planning: Depends on whether you are implementing this with a macro or the entire chip. The power network is constructed through a parallel structure of VDD and VSS strips.
- Placement: This is done in two steps. Global placement tries to find the optimal position of each cell, but it may not be in a legal position. Detailed placement slightly alters their positions to make them legal.
- Clock tree synthesis: must create a distribution network such that the clock signals arrive at the same time across all components.
- Routing: Horizontal and vertical wires connect the cells. The PDK finds various information for each metal layer such as width, height, or thickness. Sky 130 has 6 routing layers.
- Sign Off: Various design and timing tests (Design Rules Checking & Static Timing Analysis) to verify that design rules were followed and timing constraints were met.

## OpenLANE ASIC Design Flow
Open Lane is an open-source RTL to GDSII flow that references various other open-source projects and is meant specifically for the Sky-130 PDK.
- DFT (Design for Test): Optional step that uses the open-source project "Fault" to make the chip testable
-  All physical implementation is done by the OpenROAD app (Floor & Power Planning, Placement, Clock Tree synthesis, Routing, etc)
-  Antenna Rules Violation: Long wire segments can act as an antenna and accumulate charge. This might damage the connected transistor gates. The solutions are to either use bridging, where the wire is connected to a higher level intermediary, or insert an antenna diode to leak away the charges.
-  The process also includes the Sign Off process as described above

## OpenLANE Setup

# Day 2 - Floorplans & Introduction to Library Cells

## Floorplanning
1. Define the Width and Hieght of the Core & Die: You need to start by finding the area of the netlist, which is essentially the space that all the standard cells(Examples: AND, OR), take up. You can use this to find the "Utilization Factor" which is (Area of the Netlist)/(Area of the Core). Generally the utilization factor is kept at roughly 60% to allow for extra space in the core for non-ideal wires. Aspect ratio shows the ratio between hieght and width. 
2. Define the Location of Preplaced Cells: These are resuable logicblocks or macros that are already implemented and need to be placed by the user. The automated tools won't change the position of these blocks, so it's important to get these right.
3. Surround Preplaced Cells with Decoupling Capacitors: The macros often switch lots of current, which can lead to voltages drops, excess noise, and various other glitches. By placing decoupling capacitors near the logic block, the capacitors will send enough current needed for the logicblock to switch within the noise margin range.
4. Power Planning: Ideally, there will be multiple powersource taps for many reasons. It reduces voltage drops, and allows enough current to be deliver, because one set of VDD & VSS pins aren't enough and will get overloaded. Aditionally, it prevents ground bounce( a brief jump in GND potential). Power issues could lead to timing issues, so it's important to have multiple powersources. This can be done through a "power mesh," a network of horizontal and vertical bands of VDD and VSS.
5. Pin Placement: The input and output ports are located in the region between the core and the edge of the die, located on the PADS. The placements of the ports depend on the arrangement of the cells on the cores and the preference of the designer. The clock ports are thicker, lower in resistance, compared to the data ports since these ports need to supply the whole chip.
6. Cell Placement Blockage: Fills in all the area unused by the ports to prevent the automated tool from placing cells there.

7. 


