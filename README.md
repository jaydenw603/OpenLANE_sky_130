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

<img width="1479" height="732" alt="OpenLANE 3 vocab" src="https://github.com/user-attachments/assets/bf936fc9-f080-42c8-a554-612d3dca712d" />

- Macro: A pre-designed functional block that may require analog parts
- Foundry IPs: Blocks that contain special techniques to manufacture, and as a result, can only be made at foundries.

<img width="1473" height="733" alt="OpenLANE foundry vs macro" src="https://github.com/user-attachments/assets/c8df7af3-43df-489b-a53e-36a6cdcec7fe" />

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

<img width="1382" height="752" alt="OpenLANE ASIC flow chart" src="https://github.com/user-attachments/assets/282a492d-ead5-4465-8034-1eba8066e1a9" />

## OpenLANE Setup
type in `docker` and then follow the commands in the picture. 

![OPENLANE SETUP UBUNTU](https://github.com/user-attachments/assets/ba25d373-a963-47ca-abd7-93413883e74c)


# Day 2 - Floorplans & Introduction to Library Cells

## Floorplanning
1. Define the Width and Height of the Core & Die: You need to start by finding the area of the netlist, which is essentially the space that all the standard cells(Examples: AND, OR) take up. You can use this to find the "Utilization Factor," which is (Area of the Netlist)/(Area of the Core). Generally, the utilization factor is kept at roughly 60% to allow for extra space in the core for non-ideal wires. Aspect ratio shows the ratio between height and width. 
2. Define the Location of Preplaced Cells: These are reusable logic blocks or macros that are already implemented and need to be placed by the user. The automated tools won't change the position of these blocks, so it's important to get these right.
3. Surround Preplaced Cells with Decoupling Capacitors: The macros often switch lots of current, which can lead to voltage drops, excess noise, and various other glitches. By placing decoupling capacitors near the logic block, the capacitors will send enough current needed for the logicblock to switch within the noise margin range.

<img width="966" height="770" alt="OpenLANE capacitor insert" src="https://github.com/user-attachments/assets/ac300f0c-e917-47fb-be55-f1b54efb70e0" />


4. Power Planning: Ideally, there will be multiple power source taps for many reasons. It reduces voltage drops, and allows enough current to be deliver, because one set of VDD & VSS pins aren't enough and will get overloaded. Aditionally, it prevents ground bounce( a brief jump in GND potential). Power issues could lead to timing issues, so it's important to have multiple powersources. This can be done through a "power mesh," a network of horizontal and vertical bands of VDD and VSS.
5. Pin Placement: The input and output ports are located in the region between the core and the edge of the die, located on the PADS. The placements of the ports depend on the arrangement of the cells on the cores and the preference of the designer. The clock ports are thicker, lower in resistance, compared to the data port,s since these ports need to supply the whole chip.
6. Cell Placement Blockage: Fills in all the areas unused by the ports to prevent the automated tool from placing cells there.

<img width="1366" height="775" alt="OpenLANE cell placement done" src="https://github.com/user-attachments/assets/9f669bea-d58d-47f4-ae0f-488ee29673cd" />


## Floorplanning with OpenLANE
In OpenLANE type in `run_floorplan`. After checking the results you should see this:

![OPENLANE pico floorplan](https://github.com/user-attachments/assets/315cbc65-548b-480e-865c-5c195980c74e)

This shows the die area of the chip

## Floorplanning with Magic

<img width="605" height="542" alt="183226953-8bf8b067-5a70-43a3-9b92-f39caaf02a4a" src="https://github.com/user-attachments/assets/ff68cea2-519e-47c0-958a-17cee5781c2f" />

### Basic Functions: 
- s: For selecting an object. Pressing it three times will select everything electrically connected to what was originally selected. 
- v: Can be used in combination with s to center the screen. Hover your mouse over the die and press s then v to center.
- z: For zooming in and out, repeatedly press it to zoom in more. Left click to created the bottom left boundry at which you want to zoom in on, right click to make the top right boundary, completing your square.

## Placement & Routing
1. Bind the netlist to a physical cell: Give each component in the netlist a shape, width, and height from a library(a place where you can find the information on every cell, such as height, width, delay information, and the required conditions of the cell).
2. Placement: Place the cells on the floorplan. The flip-flops should be placed as close as possible to their input and output pins to reduce delay.
3. Now Optimize the Placement: This can be done by using a buffer or repeater to replicate and boost the signal across long distances. Repeaters should be inserted on the estimated capacitance and wire length due to the distance between the block and the port.

<img width="1407" height="763" alt="OpenLANE Placement Optimization" src="https://github.com/user-attachments/assets/6f0adb44-4626-4b68-bec1-56a04588bbeb" />


## Running Placement on OpenLANE
Type in `run_placement`

## Viewing Placement on Magic

![OPENLANE Placement](https://github.com/user-attachments/assets/62c61942-7e54-44cb-b3ed-0c610f70eef5)

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

<img width="831" height="895" alt="timing characterization slew thr" src="https://github.com/user-attachments/assets/b304e26d-be11-466b-acbb-5610268fa82f" />

### Propagation Delay
Propigation delay will always be positive. If you have a negative propigation delay, then you may have chosen your thresholds incorrectly, or designed your circuit poorly.

<img width="1219" height="631" alt="propigation delay bad design circuit" src="https://github.com/user-attachments/assets/1f62cc53-d928-4340-b5ae-24d716790308" />


### Transition Time
The time it takes to transition between threshold points (always measure from high to low).

# Day 3 - Designing a Library Cell with Magic and Ngspice Characterization 

### I/O Placement:
The I/O placer will automatically set the pins at equal distances apart (mode 1). To make the pins no longer equidistant, you can switch it to mode 2 by typing ```set ::env(FP_IO_MODE) 2``` into OpenLANE. You can view the new floorplan using magic, and then change the config.tcl file by echo'ing the new variable value. 

## Designing a Library Cell
- SPICE Deck - It has all the connectivity information in a cell (inputs, taps, component values, etc).
- Values in the SPICE Deck: Must set the length and width of PMOS and NMOS transistors. The PMOS transistor should be 2-3 times wider than the NMOS transistor. The gate and supply voltages should be a multiple of the length of the transistors.
## Writing the SPICE Deck:
#### Example:

<img width="687" height="664" alt="openlane SPICE Example" src="https://github.com/user-attachments/assets/fc940b4b-8201-4cd6-98d5-9ebbd52aea49" />


``` 
*** MODEL Description ***
*** NETLIST Description ***
M1 out in vdd vdd pmos W=0.375u L=0.25u
[MOSFET Name] [Drain Node] [Gate Node] [Source Node] [Substrate] [Transistor Type] [Width] [Length]

cload out 0 10f
Vdd vdd 0 2.5
Vin in 0 2.5
[Name] [Node] [Node] [Value]


.LIB "tsmc_025ummodel.mod" CMOS_MODELS (This grabs the model file for all these specific types of transistors, including all the descriptions and parameters)
.end
```
#### Simulation Commands:
- `.op` - Starts the SPICE simulation
- ```.dc Vin 0 2.5 0.05``` - Searches for the input gate voltage for values from 0V-2.5V at increments of 0.05V while also measuring the output waveform.

#### SPICE Simulation Example
- Switching Threshold: The point at which Vin = Vout. This can be represented as the intersection between the curves Vm and Vout.

<img width="973" height="687" alt="OpenLANE switching threshhold" src="https://github.com/user-attachments/assets/9962e1b8-b133-4ff3-87a2-f28c2979283d" />


## CMOS Fabrication Process (16-mask process):
1. Selecting a Substrate: The material on which the IC is fabricated. The most commonly used substrate is the P-type substrate. Different types of substrates have different properties, such as resistivity and doping level. 
2. Creating an Active Region for Transistors: Grow an insulator on the substrate (SiO2) to separate the transistor regions. Then deposit Si3N4 onto the insulator. Finally, deposit photoresist onto the Si3N4.

<img width="1121" height="568" alt="pre-litphographty" src="https://github.com/user-attachments/assets/cb62cf06-e83c-4901-9bbf-b7bd1b936873" />

 At this point, you can create "pockets" by using the process of photolithography. During this process, a "mask" protects the design from the UV light, allowing for the rest of the photoresist to be removed. Then, through the process of etching, the areas of Si3N4 that no longer have photoresist on top of it get removed. After this is done, the rest of the photoresist is removed. The SiO2 is then grown by putting it inside an oxidation furnace, looking like this afterward: 

<img width="890" height="392" alt="oxidation_furnace_after" src="https://github.com/user-attachments/assets/ccce1581-d91e-4112-b0ab-dec9a24667a3" />

Finally, the rest of the Si3N4 is removed, and the "pockets" where it once was are the active regions where the transistors are grown. 
3. N-well & P-well formation: Place another layer of photoresist, and use another mask to isolate one half of the substrate. Then, implant Boron in the part of the substrate without photoresist, thus creating a P-well. 

<img width="1173" height="583" alt="boron implantation" src="https://github.com/user-attachments/assets/a576fd46-8f26-4fa8-9588-2fd47c7e137a" />

Then repeat the steps to isolate the other side and implant Phorpourus ions, creating an N-well. 

<img width="909" height="581" alt="N-well implantation" src="https://github.com/user-attachments/assets/36003b81-89ae-4955-9aba-6877bbe6b010" />

Finally, the substrate is placed inside a furnace to diffuse the P-well & N-well, expanding their depth into the substrate. 

4. Formation of the Gate: 
Some important variables that affect the Threshold Voltage for the gate include the Doping Concentration and the Oxide Capacitance:

<img width="449" height="242" alt="vars affecting threshold" src="https://github.com/user-attachments/assets/e411664d-4fe1-4308-90d1-318331d29c27" />

To start forming the gate, once again, isolate each half of the substrate. Then, once again, dope ions into each half of the substrate, but with much lower energy (Boron for P-well, Phosphorus for N-well). Then etch away the old SiO2 layer, and regrow a new one. This can be customized to change the threshold voltage (oxide capacitance). Then you grow a polysilicon layer and dope it with N-type ions to lower the resistance. Then, you create two columns of photoresist by using photolithography.

<img width="924" height="565" alt="gate formation with poly" src="https://github.com/user-attachments/assets/7f174d9e-fcaa-4d9f-96a9-866c49236991" />

Finally, etch away the rest of the polysilicon not protected by the photoresist.  

5. Forming a Lightly Doped Drain(LDD):
To start forming the gate, once again, isolate each half of the substrate. Dope ions onto each half of the substrate, but use N-type doping on the P-type side, and vice versa. This would create an N- region in the P-gate side.

<img width="954" height="595" alt="LDD" src="https://github.com/user-attachments/assets/19b8153d-bc9f-40d0-befc-aea24687b456" />

Finally, implant a layer of either SiO2 or Si3N4 and use it for plasma anisotropic etching. This will leave behind sidewalls that look like this:

<img width="1167" height="496" alt="side walls ldd" src="https://github.com/user-attachments/assets/6ecf7fbf-4024-4006-8e6b-1c22209b72e3" />

6. Source and Drain Formation:
Start by adding a thin screen of SiO2. Then, once again, isolate each half of the substrate. Then, implant another N-type ion into the N- implant side. This makes it an N+ implant.

<img width="938" height="594" alt="Screenshot 2025-07-20 170555" src="https://github.com/user-attachments/assets/b20b3d28-0d0e-4531-8078-7c7553c78929" />

Now do the same thing to the other half. Finally, put it into a high-temperature furnace to undergo high-temperature annealing. 

<img width="944" height="500" alt="N+ wells" src="https://github.com/user-attachments/assets/fc4bda4b-afaa-4082-9853-f2486506f198" />

7. Form Contacts and Interconnects: Etch the thin oxide layer using HF. Then, deposit TiSi2 on the wafer(because it has low resistivity) and heat it in an N2 ambient until a chemical reaction occurs. As a result, some of the titanium should turn into TiN.

   <img width="1070" height="529" alt="wafer heated in ambient n2" src="https://github.com/user-attachments/assets/61628366-7758-4b52-ada7-84991ce05dc3" />

The TiN is then etched away in a process called RCA Cleaning and will look something like this:

<img width="1040" height="559" alt="RCA cleaning" src="https://github.com/user-attachments/assets/92b5e15d-5751-4464-9297-388708de30f8" />

After removing the rest of the photoresist, you are left with a TiN local interconects. 

8. Higher Level Metal Formation:
Start by depositing SiO2 doped with either boron of phosphorous onto the wafer. Then polish the surface to make it a flat plane. Then use photolithography and etching to islolate the locations for the contact holes. After removing the photoresist, place a thin layer of TiN, and a blanket layer of tungsten to look like this:

<img width="1068" height="559" alt="tungsten blanket" src="https://github.com/user-attachments/assets/06b0c717-2208-4a5d-9181-59ffc5a8b30c" />

Then, polish away the top layer of tungsten, leaving a flat plane. Aluminum is then deposited and etched away, leaving a layer ontop of the contact holes. 

<img width="887" height="510" alt="first metal layer" src="https://github.com/user-attachments/assets/6e84233e-65ac-4001-9484-a3c476831af2" />

Repeat these steps to add another metal layer(thickness increasing as the metal layer increases). 

<img width="880" height="591" alt="metal layer 2" src="https://github.com/user-attachments/assets/9fd363b6-d74a-4d8e-874d-4ec94b31f369" />

Then, add a layer of Si3N4 to protect the chip. Finally, use the 16th and final mask to drill the contact holes for the chip. 

<img width="874" height="717" alt="mask 16 end" src="https://github.com/user-attachments/assets/c9d30edf-a7e3-42d1-a3e2-5b9bec6f6e82" />

## Custom Inverter Design Lab

The following code is pasted to clone the repository into OpenLANE:

```git clone https://github.com/nickson-jose/vsdstdcelldesign.git```

The following code is pasted to clone the sky130A.tech file to the new "vsdstdcelldesign" directory:

```cp sky130A.tech /home/vsduser/Desktop/work/tools/openlane_working_dir/openlane/vsdstdcelldesign/```


