# OpenLANE_sky_130
This repository contains a comprehensive list of notes from the 5 day OpenLANE sky 130 workshop. 
# Table of Contents
- [Day 1 - Intro to open source EDA, SKY130 PDK, and OpenLane](#day-1---intro-to-open-source-eda-sky130-pdk-and-openlane)
  - i.

# Day 1 - Intro to open source EDA, SKY130 PDK, and OpenLane
## Introductory IC vocabulary
- Package: Material that surrounds and protects the IC from physical damage. It also allows for the mounting of the IC to the PCB. A package that many people are familiar with is the black epoxy or plastic cases surrounding common ICs. 
- Die: The piece of semiconducting material, usually silicon, that which the chip is manufactured on.
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
-  Antenna Rules Violation: Long wire segments can act as an antenna and accumulate charge. This might damage the connected transistor gates. The solutions are to either use bridging, where the wire is connected to a higher-level intermediary, or insert an antenna diode to leak away the charges.
-  The process also includes the Sign Off process as described above

## OpenLANE Setup

# Day 2 - Floorplans & Introduction to Library Cells

## Floorplanning
1. Define the Width and Height of the Core & Die: You need to start by finding the area of the netlist, which is essentially the space that all the standard cells(Examples: AND, OR) take up. You can use this to find the "Utilization Factor," which is (Area of the Netlist)/(Area of the Core). Generally, the utilization factor is kept at roughly 60% to allow for extra space in the core for non-ideal wires. Aspect ratio shows the ratio between height and width. 
2. Define the Location of Preplaced Cells: These are reusable logic blocks or macros that are already implemented and need to be placed by the user. The automated tools won't change the position of these blocks, so it's important to get these right.
3. Surround Preplaced Cells with Decoupling Capacitors: The macros often switch lots of current, which can lead to voltage drops, excess noise, and various other glitches. By placing decoupling capacitors near the logic block, the capacitors will send enough current needed for the logicblock to switch within the noise margin range.
4. Power Planning: Ideally, there will be multiple power source taps for many reasons. It reduces voltage drops, and allows enough current to be deliver, because one set of VDD & VSS pins aren't enough and will get overloaded. Aditionally, it prevents ground bounce( a brief jump in GND potential). Power issues could lead to timing issues, so it's important to have multiple powersources. This can be done through a "power mesh," a network of horizontal and vertical bands of VDD and VSS.
5. Pin Placement: The input and output ports are located in the region between the core and the edge of the die, located on the PADS. The placements of the ports depend on the arrangement of the cells on the cores and the preference of the designer. The clock ports are thicker, lower in resistance, compared to the data port,s since these ports need to supply the whole chip.
6. Cell Placement Blockage: Fills in all the areas unused by the ports to prevent the automated tool from placing cells there.

## Floorplanning with OpenLANE

## Floorplanning with Magic

### Basic Functions: 
- s: For selecting an object
- v: Can be used in combination with s to center the screen. Hover your mouse over the die and press s then v to center.
- z: For zooming in and out, repeatedly press it to zoom in more. Left click to created the bottom left boundry at which you want to zoom in on, right click to make the top right boundary, completing your square.

## Placement & Routing
1. Bind the netlist to a physical cell: Give each component in the netlist a shape, width, and height from a library(a place where you can find the information on every cell, such as height, width, delay information, and the required conditions of the cell).
2. Placement: Place the cells on the floorplan. The flip-flops should be placed as close as possible to their input and output pins to reduce delay.
3. Now Optimize the Placement: This can be done by using a buffer or repeater to replicate and boost the signal across long distances. Repeaters should be inserted on the estimated capacitance and wire length due to the distance between the block and the port.

## Running Placement on OpenLANE

## Viewing Placement on OpenLANE

## Cell Design and Characterization Flows

### Cell Design Flow:
The series of steps required to design a cell. They are separated into 3 steps (Inputs, Design Steps, Outputs)
- Inputs: PDKS (process design kits) that are given by the foundry give design rules, SPICE model parameters (Threshold, linear regions, and saturation region equations, etc). The user also gets to customize some things, such as cell height, width, supply voltage, pin location, and drawn gate length.
- Design Steps:
    - Circuit Design: Design the circuit function based on the design requirements established by the inputs. They are generally done with SPICE simulations
    - Layout Design: Using the inputs, you now must design the library cell. Arrange the design using Euler's path and a sticky diagram to produce the best layout. In OpenLANE, this can be done with the magic tool. 
    - Characterization: Helps get the timing, power, and noise information.
        1. Start by reading the model file from the foundry
        2. Read the extracted SPICE netlist
        3. Recognize the behavior of the buffers
        4. Read the subcircuit
        5. Attach the necessary power supplies
        6. Apply the stimulus(input signals to test behavior such as waveforms)
        7. Supply the necessary output capacitance
        8. Supply the necessary simulation commands
      #### Feed all of these steps into a Characterization software called GUNA, which outputs timing, power, and noise .libs.
## Timing Characterization 
- Low slew thresholds will usually be measured at 20% of the value
- High slew thresholds will usually be measured at 80% of the value
- In and out thresholds will be measured at 50% of the value, regardless of whether they are rising or falling.
### Propagation Delay
Propigation delay will always be positive. If you have a negative propigation delay, then you may have chosen your thresholds incorrectly, or designed your circuit poorly.

### Transition Time
The time it takes to transition between threshold points (always measure from high to low).

# Day 3 - Designing a Library Cell with Magic and Ngspice Characterization 

### I/O Placement:
The I/O placer will automatically set the pins at equal distances apart (mode 1). To make the pins no longer equidistant, you can switch it to mode 2 by typing ```set ::env(FP_IO_MODE) 2``` into OpenLANE. You can view the new floorplan using magic, and then change the config.tcl file by echo'ing the new variable value. 

## Designing a Library Cell
- SPICE Deck - It has all the connectivity information in a cell (inputs, taps, component values, etc).
- Values in the SPICE Deck: Must set the length and width of PMOS and NMOS transistors. The PMOS transistor should be 2-3 times wider than the NMOS transistor. The gate and supply voltages should be a multiple of the length of the transistors.
## Writing the SPICE Deck:
#### Examples:
``` 
*** MODEL Description ***
*** NETLIST Description ***
M1 out in vdd vdd pmos W=0.375u L=0.25u
[MOSFET Name] [Drain Node] [Gate Node] [Source Node] [Substrate] [Transistor Type] [Width] [Length]

cload out 0 10f
Vdd vdd 0 2.5
Vin in 0 2.5
[Name] [Node] [Node] [Value]
```
#### Other Important Commands:
- `.op` - Starts the SPICE simulation
- ```.dc Vin 0 2.5 0.05``` - Searches for the input gate voltage for values from 0V-2.5V at increments of 0.05V while also measuring the output waveform.
- 




