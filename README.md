# Digital VLSI SoC Design and Planning 

## DAY 1
### Introduction to QFN-48 Package, chip, pads, core, die and IPs

(image)

The diagram below illustrates a block-level architecture of a Processor or System on Chip (SoC) along with its interfacing components. It highlights the pinout layout and shows how various peripherals connect to the SoC. This is a typical example of an SoC interface, commonly used in embedded system design. It shows how communication interfaces like I2C, UART as well as memory units. It also includes power and ground connections.In technical term the processor with all components is called a PACKAGE. One kind of package, which is used in the arduino board is a QFN - 48 package (Quad Flat No-Leads).

(image)
A chip is actually inside a package, and is connected to various "PINS" or inputs/outputs.
components of chip are:
1. pads : Pads serve as the connection points between the integrated circuit and the external chip package. These small structures are responsible for transmitting electrical signals into and out of the chip. They are carefully designed and positioned to align with the package pins.
2. core : The core is where the digital logic resides. It includes all the fundamental building blocks like AND, OR, XOR gates, multiplexers (MUXes), and more. This is the heart of the chip—responsible for executing all processor functions.
3. die : A die is a single piece cut from a larger semiconductor wafer. It can function independently and often contains one or more cores. Each die represents a complete, functional unit ready to be packaged or integrated into devices.

Inside the core, there are two main types of reusable components:

Foundry IPs are building blocks provided by the chip manufacturer (foundry). These are already designed and tested. They include things like PLLs (for clock control), ADCs (for signal conversion), and SRAM (for memory).

Macros are similar to Foundry IPs but are purely digital. These are standard logic blocks that can be reused across different chip designs. Common examples include GPIO banks and SPI controllers.

Together, these components help in designing an efficient, reliable, and reusable SoC.

### Introduction to RISC-V
---
RISC-V is an open-standard Instruction Set Architecture (ISA) based on the principles of Reduced Instruction Set Computing (RISC). Unlike proprietary ISAs like ARM or x86, RISC-V is free to use, allowing individuals, companies, and universities to design and build their own processors without licensing fees. In hardware, the chip is connected to its package using bond wires.

### From Software Applications to Hardware
---
Devices are powered by chips, but apps written in high-level languages can't run directly on hardware. They must be translated into binary machine code that the chip understands. This is where system software comes into play.

System software handles the translation process in the following steps:

1. Operating Systems (OS): The OS manages hardware resources and performs essential tasks like input/output operations, memory management, and low-level system functions. It also interacts with the application software and sends code to the compiler for translation.
2. Compilers: Compilers take the high-level source code (e.g., C/C++) and convert it into lower-level machine instructions suited for the hardware. These instructions are architecture-specific and are often referred to as ISAs (Instruction Set Architectures). The output is usually an executable file (e.g., .exe) that is passed to the assembler.
3. Assemblers: Assemblers translate the compiler-generated instructions into binary machine code. This binary code is what the hardware finally executes.
   
(imagr)
In the image above shows Stopwatch App as an example. A C program is written to implement the stopwatch functionality. Here's how it's executed step by step:
1. The program is compiled by the system (e.g., Windows/Linux) into machine instructions.
2. These instructions are then translated into RISC-V assembly code.
3. The RISC-V code is converted by the assembler into binary format.
4. This binary is processed by the RTL (Register Transfer Level) design of the hardware.
5. RTL is synthesized into a netlist, which describes the circuit connections.
6. Finally, the netlist is turned into a chip layout during physical design, producing real, functioning hardware capable of running the stopwatch.

### SoC Design and OpenLANE
---
This image illustrates the open-source digital ASIC (Application-Specific Integrated Circuit) design flow, using freely available tools, design files, and manufacturing data provided by the open-source community.
At the center of this process is the ASIC, created through a combination of:
* EDA Tools (Electronic Design Automation):These tools, like QFlow, OpenROAD, and OpenLane, are used for designing, simulating, and verifying digital circuits and PCBs.
* RTL Designs (Register Transfer Level): These are digital design files written in hardware description languages such as Verilog or VHDL. They define how a circuit should function. RTLs are commonly sourced from open platforms like librecores.org, opencores.org, or GitHub.
* PDK Data (Process Design Kits): A PDK is a set of files provided by a semiconductor foundry to model the physical fabrication process. It includes: Design Rules (DRC), Device Models, Standard Cell Libraries (digital logic), I/O Libraries

A notable example is the 130nm open-source PDK released by SkyWater and Google, which made open-source ASIC design truly possible starting in June 2020.

### Simplified RTL2GDS flow
---
The RTL to GDSII flow is the process that transforms a high-level hardware design (usually written in Verilog or VHDL) into a physical layout file (GDSII) ready for chip manufacturing. This flow involves multiple steps, each crucial for ensuring that the final design is functionally correct, physically manufacturable, and electrically reliable.
1. Synthesis : Converts RTL (Register Transfer Level) code into a gate-level netlist using logic gates from a Standard Cell Library (SCL).
* Each standard cell has predefined models for behavior and layout.
* This step translates high-level descriptions into real, placeable logic components.
2. Floorplanning & Power Planning (FP + PP): Floorplanning involves dividing the die into regions for various functional blocks and placing I/O pads.
* Power Planning ensures a proper power distribution network (VDD and GND) throughout the chip.
* A good floorplan improves performance and makes routing easier later.
3. Placement : Standard cells from the netlist are placed onto the chip layout in rows aligned with a predefined grid (called "sites").Done in two stages:
* Global Placement: Estimates initial positions for cells (may allow overlaps).
* Detailed Placement: Refines positions from the global stage with no overlaps, optimizing for timing and area.
4. Clock Tree Synthesis (CTS): Builds a network to distribute the clock signal to all sequential elements (like flip-flops). Uses special buffers and inverters to ensure:Minimal clock skew (difference in arrival time) and Minimal latency
* A well-designed clock tree ensures correct and synchronized operation of the circuit.
5. Routing : Connects all placed cells using metal wires over multiple layers. Done in two phases:
    * Global Routing: Plans rough wire paths to avoid congestion.
    * Detailed Routing: Final wire paths are drawn with exact width and spacing.

Includes signal routing and power routing, following design rules to avoid issues like crosstalk and timing violations.
6. Sign-off : The final stage before fabrication, ensuring the design is correct and manufacturable. Includes:
* Timing analysis (setup/hold checks)
* Power analysis
* DRC (Design Rule Check)
* LVS (Layout vs Schematic)
* Signal integrity checks

Only after passing sign-off is the GDSII layout file generated, which is sent to the foundry for chip fabrication.

Throughout this entire flow, the PDK (Process Design Kit) provides critical technology-specific data, such as: Standard cell layout and electrical models, Design rules for manufacturing, Metal layer definitions, I/O cell libraries.

For open-source ASIC projects, the SkyWater 130nm PDK enables designers to go from RTL to GDSII using fully open-source tools

### Introduction to OpenLane and detailed ASIC design FLOW
---
OpenLane is an open-source digital ASIC design flow built on top of tools like Yosys, OpenROAD, Magic, and Netgen. It streamlines the RTL-to-GDSII process and adds utilities for design exploration, testing, and verification. Key stages:
1. Synthesis Exploration
   * it generates a delay vs area report
   * Helps identify how circuit complexity impacts speed and size.
   * Uses Yosys and ABC for logic synthesis
2. Design Exploration
   * Sweeps through multiple design configurations (such as utilization or clock period).
   * Identifies the best configuration for performance, power, or area.
   * Generates reports to support data-driven decision.
3. OpenLANE Regression Testing
   * Compares results to past baselines.
   * Ensures that code or tool changes haven’t degraded performance or quality.
4. Design for Test (also k/a DFT)
   * Adds features to the chip to allow post-fabrication testing. Includes: Scan insertion, Automatic Test Pattern Generation (ATPG), Fault simulation, Pattern compaction, Fault coverage analysis
   * Ensures defects can be identified in real silicon.

After DFT, physical implementation is done using the OpenROAD tools. The major steps include floor and power planning, adding decoupling capacitors and tap cells, global and detailed placement of components, post-placement optimization, clock tree synthesis (CTS), and global and detailed routing. These steps physically arrange and connect the circuit on the chip. 

Since CTS and post placement optimization change the netlist, it is important to verify that the design's logic remains the same. This is done using the LCE tool in Yosys, which performs formal verification to ensure that the functional behavior has not changed due to netlist modifications.

5. Logic Equivalence Check (LEC)
   * checks that the physical implementation and the netlist have the same logic.
   * It is performed each time netlist is modified and checks that changing netlist did not change function.
6. Dealing with Antenna Rules violations
   * Verifies that metal interconnects don’t accumulate dangerous charge during fabrication.
   * If violations exist, antenna diodes or rerouting are applied
   * OpenLANE uses a preventive method by placing fake antenna diode cells next to every cell input after placement. If the antenna checker detects a real violation, these fake diodes are replaced with actual diode cells.
7. Timing Analysis (STA)
   * Uses the synthesized netlist and other physical data to verify timing constraints.
   * Ensures setup and hold times are met for all paths.
   * This involves extracting resistance and capacitance information from the layout using DEF2SPEF, then running STA with the OpenSTA tool.
8. Physical verification (DRC & LVS)
   * Design Rule Checking (DRC) ensures the layout follows manufacturing rules, which is done using Magic.
   *  Layout Versus Schematic (LVS) checks that physical layout matches the circuit logic. This is done using Magic and Netgen, and a SPICE netlist is also extracted from the layout for further analysis.
  
### Get Familiar to Opensource EDA tools
Open the virtual machine and then open terminal on it. Now, First we try to get into the working directory on which we have to work to access the OpenLANE flow.

## LAB 1

**1. Run 'picorv32a' design synthesis using OpenLANE flow**

The Path for the current working directory:
```
vsduser@vsdsquadron:~/Desktop/work/tools/openlane_workshop_dir/openlane$
```
To open Openlane, we can use the docker command using interactive. After invoking the docker command, the prompt changes to bash-4.2$

```
docker 
```
```tcl
# to access all the tools available in the openLANE
bash-4.2$ flow.tcl -interactive

# Now that OpenLANE flow is open we have to input the required packages for proper functionality of the OpenLANE flow
package require openlane 0.9

#Now the OpenLANE flow is ready to run any design and initially we have to prep the design creating some necessary files and directories for running a specific design which in our case is 'picorv32a'
prep -design picorv32a

#Now that the design is prepped and ready, we can run synthesis using following command
run_synthesis

# Exit OpenLANE
exit
```
**2. Calculate the flop ratio**

Calculation of Flop Ratio and DFF % from synthesis statistics report file

*Flop Ratio = 1613 / 14876 = 0.10842*

*Percentage of DFF's = 0.10842 * 100 = 10.84296 %*

--------
## DAY 2
### Good floorplan vs bad floorplan and introduction to library cells

Netlist describes the connectivity between all the Electronic components. Below, represents the netlisting of the various Flip-Flops and logic gates.These needs to be converted to specified physical dimensions for placing inside the core. [2.1] 


Consider both standard cells and FF has 1 unit height and width so there area is 1 sq unit.Next calculate area occupied by the netlist on a silicon wafer. Thus since 4 gates/flip-flops here, the total size of the silicon wafer will 4 sq. units.

Core is defined as the section of the chip where the fundamental logic of the design is placed. Die consists of core, it is a small semiconductor material specimen on which the fundamental circuit is fabricated.

Utilisation factor is the Area occupied by netlist / Total area of core. If the logical cells occupy the core fully, it is known as 100% utilisation. Aspect Ratio is the ratio between height and width. If the chip is square - it is 1, else the chip is rectangular in shape.
For eg. in the fig above:

Utilisation factor = 50 %

Aspect Ratio = 2 : 4 = 1 : 2 = .5

### Preplaced cells

Pre-Placed cells are complex logic blocks that can be reused. They are already implemented and cannot be touched by Auto Place and Route tools - and hence are required to be very well designed.

For this first we divide the logic into blocks - while preserving the connectivity of the logic. By extending IO pins and making connections we can convert the logic into two parts that can be used as needed elsewhere in the design if needed. The various preplaced blocks available include memory, clock-gating cell, comparator, MUX.

Thus Floorplanning is the arrangement of these IPs onto the chip.

### Decoupling Capacitor

Decoupling capacitors are placed locally around the pre-placed cells. The decoupling capacitor is a large capacitor completely full of charge whose voltage is equivalent to the power supply. When many gates switch at the same time, the power supply may not respond quickly enough, leading to voltage drops or noise on the power lines. This unstable power can cause incorrect logic behavior or timing issues in the chip. During switching activity the decoupling capacitor decouples the circuit from the main supply and provides the necessary voltage required by the pre placed cells. 

### Power Planning

As all coupling capacitors present in the circuit demands the power suppy simultaneosly a single power supply cannot adhere to the demands which reult in noise in the circuit cause due to voltage groop or ground bounce. Hence, power planning is very important part of floor planning. During this stage mulltiple power supply are placed in a chip for proper fuctioning. One structure for the power planning is the mseh structure where multiple power and groud lines are arrange in horizontal abd vertical manner as shown in figure below.

### Pin placement

For proper pin placement the connectivity information coded using verilog/vhdl language called netlist is considered. The location of input and output pin depensd upon the conncetivity requirement and the designer. However, the basic trend is to select a location which results in reduce connectivity length. Optimal pin placement is done taking care about the less buffering and less amount of power consumption. The size of the clock pins are wider as compared to the other data pins. The locgiical cell blockage is done to make sure that the automated PnR does not used the are for placement of cells.

Thus the complete deign is:

## LAB DAY 2

**1. Run floorplan**
```tcl
# to access all the tools available in the openLANE
bash-4.2$ flow.tcl -interactive

package require openlane 0.9
'
prep -design picorv32a

#Now that the design is prepped and ready, we can run synthesis 
run_synthesis

# Now we can run floorplan
run_floorplan
```

**2. Find the die area from the floorplan def**

Area of die in microns = Die width in microns * die height in microns 1000 Unit Distance = 1 Micron Die width in unit Distance = 660685 - 0 = 660685 Die height in unit Distance =671405 - 0 = 671405 Distance in microns = Value in unit Distance / 1000 Die width in microns = 660685 / 1000 = 660.685 Microns Die height in microns = 671405 / 1000 = 671.405 Microns Area of Die in microns = 660.685 × 671.405 = 443587.212425 Square Microns

**3.Load generated floorplan def in magic tool**

```tcl
# Change directory to path containing generated floorplan def
cd Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/17-03_12-06/results/floorplan/

# Command to load the floorplan def in magic tool
magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.floorplan.def &
```
**4.Run 'picorv32a' design congestion aware placement using OpenLANE flow and generate necessary outputs**

```tcl
# Congestion aware placement by default
run_placement
```

**5.Load generated placement def in magic tool and explore the placement.**

```tcl
# Change directory to path containing generated placement def
cd Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/17-03_12-06/results/placement/

# Command to load the placement def in magic tool
magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.placement.def &
```

### Cell design and Characterization flow

Thus, After final placement and routing for the above netlist the final die will be represented as shown.
It consists of standard cells that are stored in the Library. The library consists of various varities of std. cells of different sizes and different functionality.

Steps of Characterisation Flow:-

1. Reading of SPICE module files
2. Reading of netlist extracted by SPICE
3. Recognising buffer behaviour
4. Reading subcircuits
5. Attaching neccessary power sources
6. Applying stimulus
7. Provide neccessary output capacitance
8. Provide simulation command like .tran for transient simulation or .dc for DC simulation.
   
These steps are given to atool called GUNA in the form of a configuration file, which runs simulations and will generate timing, noise and power models in the form of .libs files.

## DAY 3
### Design Library Cell Using Magic Layout and NGSPICE characterisation












