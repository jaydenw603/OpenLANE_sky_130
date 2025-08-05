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

1. The following code is pasted to clone the repository into OpenLANE:

```git clone https://github.com/nickson-jose/vsdstdcelldesign.git```

2. The following code is pasted to clone the sky130A.tech file to the new "vsdstdcelldesign" directory:

```cp sky130A.tech /home/vsduser/Desktop/work/tools/openlane_working_dir/openlane/vsdstdcelldesign/```

Note: to create a box in Magic you need 2 coordinate points (llx,lly) and (urx,ury), (lower left and upper right).

3. Make an extract file by typing ```extract all``` in the tkcon terminal. After, extract the spice file by typing ```ext2spice cthresh 0 rthresh 0``` followed by ```ext2spice```.
4. Download ngspice if necessary by typing in ```sudo apt install ngspice```.
5. View the spice file by typing in ```vim sky130_inv.spice```.

![spice view](https://github.com/user-attachments/assets/a2c350d4-0b5c-478e-a807-fafaafa74596)
#### Basic Commands:
- :w - writes the changes made in the vim file to the file (saves it)
- :quit! - exits the vim file
- i - allows you to edit the file, "insert" command
- Esc - allows you to escape the insert command
  
6. Edit the file to make it look like this:

![Spice deck](https://github.com/user-attachments/assets/6e72eec6-cf66-48ae-ae19-55a94b8a0ee3)


7. Type in ```ngspice sky130_inv.spice``` in the terminal to open ngspice.
8. Then type in ```plot y vs time a``` to plot variables y and a on a Voltage vs Time graph.

![ng spice graph](https://github.com/user-attachments/assets/207f7a16-8b21-49b0-8783-ec5c4fca0142)

9. Using this graph, we can now find the slew rates and propigation delay:
#### Rise Delay & Fall Delay(The time difference between 50% of the Voltage Value, 1.65V):
My estimated Rise Delay: 4.06634ns - 4.0401ns = 0.02624ns

![rise transition](https://github.com/user-attachments/assets/ce0e3a70-ad2c-4514-9355-8f876d14b1b4)


My estimated Fall Delay: 4.05545ns - 4.04554ns = 0.00991ns

![fall transition](https://github.com/user-attachments/assets/18dcd77c-c132-4070-a01c-e86a62790e8c)


#### Rise & Fall Transition(The time difference between 20% and 80% of the Voltage Value, 0.66V and 2.64V):
My estimated Rise Transition: 2.18ns - 2.1194ns = 0.0606ns

![rise time](https://github.com/user-attachments/assets/bf0644c5-36b7-47a8-b445-e5de333005fa)

My estimated Fall Transition: 4.06634ns-4.0401ns = 0.02624ns

![fall delay](https://github.com/user-attachments/assets/97e1e105-ac83-495a-bb37-986ca3c3afa4)

## Magic DRC Lab
- A website that lists all Magic commands and some tutorials on how to use the Magic tool: http://opencircuitdesign.com/magic/
- A website that lists specific information about the technology, like electrical connectivity, design rules, rules for netlists, etc: http://opencircuitdesign.com/magic/techref/maint2.html
- A website that lists specific DRC rules and syntax: https://skywater-pdk.readthedocs.io/en/main/rules/periphery.html#rules-periphery--page-root
1. Type in ```wget http://opencircuitdesign.com/open_pdks/archive/drc/tests.tgz``` to download the files needed for the lab and then extract the tarball.
2. Create an example of the DRC rules by making a box of m3contact and then typing ```cif see VIA2``` into the tkcon terminal. It should auto-format via2 onto the box of m3contact as outlined by the design rules, always leaving the required amount of space between the via2 and edge of m3contact.

![painted metal3](https://github.com/user-attachments/assets/c6100f39-e340-401e-91e3-dc9a44177843)

3. Here is an example of a DRC issue where it isn't detecting an issue despite the polyspacing being less than .21um.

![drc no issues](https://github.com/user-attachments/assets/a2f5a2fe-5c88-4455-a667-2299fadcae76)

4. This can be fixed by altering the sky130A.tech file as such. You can find keywords in vim by typing `/` followed by the keyword.

![skytech change](https://github.com/user-attachments/assets/fa024add-76b8-4639-879f-5fcc4b3b28c7)![skytech change 2](https://github.com/user-attachments/assets/d5eb0948-8da3-4e0e-b11f-908e9df01ee4)

![drc issue fixed](https://github.com/user-attachments/assets/9ed411f0-4047-41c1-8ff2-d77b0505a487)

# Day 4 - Pre-layout timing analysis & the Importance of a Good Clock Tree

## Converting Grid Info to Track Info in Magic
You can find the information under the pdks in the format [name] [axis] [offset] [track pitch]: 

![grid coordinates](https://github.com/user-attachments/assets/10ca0603-5dbb-4c68-8395-0e302d853d5a)

You can then type this information into the tkcon terminal in magic to change the grid. You can toggle the grid in magic by pressing `g`. Type in ```grid 0.46um 0.34um 0.23um 0.17um``` to change the grid values to those shown in the file. When viewing the grid again, it will be changed and the pins will lie on the intersections

![new grid magic](https://github.com/user-attachments/assets/13f2a1e2-2967-4e39-aa6a-9a142c6441f7)

I personally struggled to run synthesis on the new config.tcl file, but I got it after lots of debugging.

After getting errors like these for a while: ![openland issues cont](https://github.com/user-attachments/assets/1ea2534f-b269-411c-bf7d-1cb470c54051)

I learned that completely restarting OpenLane and going through the docker process fixed it. I assume the software is caching LIB_MAX values somewhere and causing it to conflict.

I then identified another error, which was due to my permissions. Originally, my `picorv32a.v` & `picorv32a.sdc` files belonged to the docker group, instead of my user, so I had to redownload those files.

![working src directory](https://github.com/user-attachments/assets/f736a7dd-3a95-49d2-86dc-6a964b583a66)

This was my working config.tcl file.

![working config tcl file](https://github.com/user-attachments/assets/48ddd111-1daa-4eb2-a5ba-808cfa12994a)

However, there is a large slack violation, which is expected and will be fixed later. 

![slack violation](https://github.com/user-attachments/assets/346b1cca-5f49-4ca0-89e8-c6c1978f93f4)

## Delay Tables

Buffers on the same level must have the same capacitance and size to ensure equal timing delay on each level. However, they can be different on differing levels. This means different levels will have varying slew rates and will thus have varying delays.

![buffer](https://github.com/user-attachments/assets/86219e63-4abf-4eab-8e89-384983963bd3)

Delay tables are used to find the timing of each cell. The main factors of delay are input slew and output capacitance. Each buffer will have its own unique delay table. You add the delay from all levels to find the latency of each block, which should be the same. 

![delay tables](https://github.com/user-attachments/assets/fe31d891-ed76-4caa-bd3e-5a8c89db07f6)

In this example, the skew is 0 across all blocks because the latency is always x9 + y15.

## Day 4 Lab Cont.

To minimize the slack error, type the following commands into the openlane terminal:

```
% echo $::env(SYNTH_STRATEGY)
% set ::env(SYNTH_STRATEGY) "DELAY 1"
% echo $::env(SYNTH_BUFFERING) 
% echo $::env(SYNTH_SIZING) 
% set ::env(SYNTH_SIZING) 1
% echo $::env(SYNTH_DRIVING_CELL)
```
Afterwards type in `init_floorplan` and then `run_placement`. I got these design stats after I ran placement.

![working placement](https://github.com/user-attachments/assets/19fa2085-2dde-4347-982f-12a9612a3491)

After opening magic in the placement directory of this run, I saw this placement: 

![placement](https://github.com/user-attachments/assets/54c5fe17-3dbc-492a-b729-c103cb5e6106)

When taking a closer look at the chip, you can see how the rails are aligned across cells.

![alligned rails](https://github.com/user-attachments/assets/df1c8dc5-fde6-4de3-ad58-6cf071e8c5c2)

## Timing Analysis 
The equation for Timing Analysis is ```Θ < T - S - SU```
Θ (Combinational Delay): The total time the signal takes to travel between two flip-flops. It's the time your circuit takes to process a signal between two sequential clocked elements.
T (Time period): The duration of one full clock cycle. This is the maximum time required for a signal to complete its journey between flip-flops.
S (Setup time): It's how early the signal needs to arrive before the clock ticks so the capture flip-flop can read it correctly.
SU (Setup Uncertainty): It’s a buffer that accounts for the unpredictable wiggles in the clock's timing caused by hardware imperfections.

## Lab Steps to Configure OpenSTA

Reduce the negative slack by setting ```set ::env(SYNTH_MAX_FANOUT) 4``` and running synthesis again. I actually removed all my negative slack.

![0 slack](https://github.com/user-attachments/assets/5b9e5291-d276-4e7e-b50d-d3ac3cadba35)

After setting up the pre_sta.conf file for OpenSTA timing debugging and running these commands I was able to reduce my negative slack from -3.02 to -1.97 (test file because my true wns is 0):
```
replace_cell 46045 sky130_fd_sc_hd__mux2_4
replace_cell 46535 sky130_fd_sc_hd__mux4_4
replace_cell 23636 sky130_fd_sc_hd__nand2_4
```
## Clock Tree Synthesis 

Clock Slew: This happens due to the resistance and capacitance of the wires carrying the clock signal, causing distortion or delay by the time it reaches its destination. To control clock slew, designers use clock buffers.
Crosstalk: Crosstalk occurs when a signal from a nearby net interferes with the clock net. Clock shielding helps prevent this by isolating the clock net from adjacent nets, reducing the coupling capacitance between them. The shield can be connected to either VDD or ground since these voltages are stable.

<img width="1389" height="732" alt="clock net sheidl" src="https://github.com/user-attachments/assets/133d9c96-2542-43e2-a004-c48f4ab4d88b" />

### Timing Analysis with real Clocks
Single Clock:
Timing analysis with real clocks accounts for the delays introduced by clock buffers, which influence both setup and hold checks. In setup analysis, the data must arrive at the capture flip-flop before the next rising clock edge to be latched correctly. A setup violation occurs when the path is too slow, often due to high combinational delay, clock buffer delay, setup time, or timing uncertainty (jitter). These factors are analyzed across two clock cycles to ensure data stability.

<img width="1365" height="766" alt="clocks thign" src="https://github.com/user-attachments/assets/82c61691-cc11-4287-a7b6-a5f367a86e88" />

To the left of the `<` is the Data Arrival Time and to the right is the Data Required Time. Arrival - Required = Slack, which should always be 0 or greater. 

Hold analysis, on the other hand, ensures that data is not captured too early. It examines whether the data remains stable after being launched, using the same rising clock edge for both launch and capture flip-flops. A hold violation happens when the path is too fast, and data arrives before the flip-flop is ready to capture it. This is influenced by combinational delay, hold time, and clock buffer delay, but not by time period or jitter. The ultimate goal is to achieve positive slack in both setup and hold checks to ensure reliable data capture.

<img width="1380" height="769" alt="holding anaylss" src="https://github.com/user-attachments/assets/678daf6c-11ba-4c3b-b39f-0ecff43f74ce" />



## Running CTS with TritonCTS
After reducing the negative slack as much as possible, and running floorplan and placement, type in `run_cts`. This will create a DEF file which can be used for CTS. 

![run_cts](https://github.com/user-attachments/assets/cce2f244-489d-4dfe-af5f-7598e2d6fe88)

After following the steps for CTS analysis, there is a large slack violation. However, this is to be expected because we are evaluating it with Min and Max corners, not just one. This means this analysis is unreliable.

![cts analysis](https://github.com/user-attachments/assets/618523bb-1715-4ed3-a012-6251a8ee9fe5)

After following these correct steps (with the db file already setup) you get the correct analysis: 
```
openroad
read_db pico_cts.db (don't have to write the db file again because nothing changed)
read_verilog /openLANE_flow/designs/picorv32a/runs/t3/results/synthesis/picorv32a.synthesis_cts.v
read_liberty $::env(LIB_SYNTH_COMPLETE)
link_design picorv32a
read_sdc /openLANE_flow/vsdstdcelldesign/extras/my_base.sdc
set_propagated_clock [all_clocks]
report_checks -format full_clock_expanded -digits 4
```

![correct cts analysis](https://github.com/user-attachments/assets/c960a392-dab6-4b38-9ef8-5a2d2ccb4a93)

![whole slack](https://github.com/user-attachments/assets/72a4bfad-e177-43e5-b4bb-4c137b57af69)

# Day 5 - Final Steps for RTL2GDS

## Routing
One routing method is called Maze Routing or Lee's Algorithm:
It looks for the shortest path between two cells(source and target) with a routing grid. There may be multiple paths, but the best path is one with the fewest bends/corners. The route can't be diagonal, often looking like an L-shape, and must not overlap an obstruction such as a macro. Some downsides to this algorithm are its high run time and high memory consumption, so it's better to use a more optimized routing algorithm. But the core principle of the shortest path with the least amount of bends will stay constant.

<img width="752" height="623" alt="routing grid" src="https://github.com/user-attachments/assets/0ee7529e-8a73-4e64-86d1-11077c424373" />

## DRC Cleaning
DRC cleaning is done to ensure the design rules were followed and that the routes can be fabricated correctly. Most DRCs are due to the constraints of the photolithography process, where the wavelength of light limits the minimum size of the trace. There are thousands of DRCs. Some examples include:
- Minimum wire width
- Minimum wire pitch (center-to-center between wires)
- Minimum wire spacing (edge-to-edge between wires)
- Signal short: Where two wires on the same layer cross over each other, disrupting the signals between cells. This can be solved by moving the route to the next layer using vias.
- Via Width
- Via Spacing

<img width="376" height="189" alt="wire pitch" src="https://github.com/user-attachments/assets/3eae2516-04ae-41df-b268-9bd9b0709464" />

<img width="788" height="336" alt="chnaged wire" src="https://github.com/user-attachments/assets/9357ba20-d6d3-48fe-87de-d0bbf5b7038e" />

## PDN (Power Distribution Network)

`gen_pdn` generates the PDN. The PDN takes in the design_cts.def as the input def file. This will create the grid for the VDD and GND. VDD and GND are organized in a ring around the core, and the straps connect them to the standard cells. The ring of VDD and GND are powered by the pads surrounding them. 

<img width="760" height="500" alt="PND diagram" src="https://github.com/user-attachments/assets/00ae3bf8-a9c4-48e9-bc73-8e12642e4b9e" />

## Routing with TritonRoute

do `run_routing` to start the routing process.

OpenLane routing is separated into two stages:
Global Routing: Routing guides (rectangular grids) are created that can route everything. This is done by using FastRoute.
Detailed Routing: Uses the grid created by global routing to connect the pins using the least amount of bends and in the shortest distance. The tool used is TritonRoute, and it constantly searches for optimisations to find the best possible path to connect the pins.

After my routing process was completed, I had 7 DRC errors that this specific software couldn't automatically fix. If I had chosen a different software, it likely could've found a solution at the expense of a higher runtime and required memory.

![routing working with errors](https://github.com/user-attachments/assets/c0c47260-3f44-443b-bdb6-c182bd53fc11)

Opening the file `120-tritonRoute.drc`, you can see that all 7 DRC errors are shorting errors.

![DRC errors](https://github.com/user-attachments/assets/72c3c2ba-31a5-4b40-a996-a438a84ac876)

After opening up magic, this is the model of the chip:

![Post routing chip](https://github.com/user-attachments/assets/d52decc3-329a-4f74-a5ad-21fabd9f957d)

This marks the end of the RTL2GDSII process. To fix the DRC errors, they would either need to be manually fixed or the design would need to be rerouted. 
