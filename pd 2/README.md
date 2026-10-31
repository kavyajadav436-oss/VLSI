# PD MODULE-2
<p>
  <img src="https://img.shields.io/badge/Language-Verilog-9cf" alt="Verilog">
  <img src="https://img.shields.io/badge/Tool-Icarus%20Verilog-blue" alt="Icarus Verilog">
  <img src="https://img.shields.io/badge/Tool-GTKWave-orange" alt="GTKWave">
  <img src="https://img.shields.io/badge/Tool-Yosys-green" alt="Yosys">
  <img src="https://img.shields.io/badge/PDK-SKY130-red" alt="SKY130">
</p>

## Chip Floorplanning and Power Integrity

## 📌 Project Overview

This project is part of the Physical Design (PD) learning journey and focuses on the fundamental concepts of **ASIC Chip Floorplanning and Power Integrity**.

The module explores how a logical netlist is transformed into a physical representation by defining the chip core and die dimensions, calculating cell area, setting utilization and aspect ratio, and organizing standard cells and pre-placed IP blocks within the floorplan.

The project also covers important power integrity concepts such as **switching current, IR drop, inductive voltage drop, noise margin, and decoupling capacitors**. In addition, practical aspects of the ASIC physical design flow are explored using the **OpenLane flow**, **OpenROAD-based layout tools**, and the **SKY130 technology and standard-cell library**.

The later stages of the module demonstrate **floorplanning, power planning, PDN creation, standard-cell placement, placement blockages, tap cells, LEF and technology files, timing constraints, and binding logical netlist cells with physical library cells**.

Overall, this project provides a step-by-step understanding of how an ASIC design progresses from a logical netlist to a structured physical layout.

---

## 🎯 Objectives

The main objectives of this project are:

- Understand the fundamentals of ASIC physical design and chip floorplanning.
- Analyze a logical netlist and convert logical cells into physical representations.
- Calculate the area occupied by standard cells and flip-flops.
- Understand the relationship between **core area, die area, utilization factor, and aspect ratio**.
- Study the role and placement of **pre-placed cells and IP blocks**.
- Understand how floorplanning affects routing, congestion, timing, and overall chip performance.
- Analyze switching current and its impact on power integrity.
- Understand **IR drop** and inductive voltage variations in the power distribution network.
- Study **noise margin** and the effect of supply noise on digital circuits.
- Learn the purpose and placement of **decoupling capacitors**.
- Explore **power planning and Power Distribution Network (PDN)** concepts.
- Understand the OpenLane physical design configuration flow.
- Study the **SKY130 technology, LEF files, and standard-cell libraries**.
- Understand standard-cell placement, placement blockages, and tap-cell insertion.
- Learn how logical netlist cells are bound with physical library cells.
- Visualize the transition from **Netlist → Floorplan → Placement → Physical Layout**.

---

## 🛠️ Tools and Technologies

| Tool / Technology | Purpose |
|---|---|
| **OpenLane** | Automated open-source RTL-to-GDSII ASIC design flow |
| **OpenROAD** | Physical design implementation and layout visualization |
| **SKY130 PDK** | Open-source 130 nm process design kit |
| **sky130_fd_sc_hd** | High-density standard-cell library |
| **LEF Files** | Physical abstraction of standard cells, pins, layers, and obstructions |
| **Tcl** | Configuration and automation of the OpenLane flow |
| **Verilog** | Hardware description of the design |
| **SDC** | Timing and clock constraint definition |
| **KLayout / Layout Viewer** | Visualization and inspection of physical layouts |
| **Linux / Ubuntu** | Development and execution environment |
| **GitHub** | Project documentation and version control |

---

## 📚 Table of Contents

- 1 - Define Width and Height of Core and Die
- 2 - Convert Netlist Symbols into Physical Dimensions
- 3 - Calculate Area Occupied by the Netlist
- 4 - Utilization Factor and Aspect Ratio
- 5 - Core and Die Dimension Example
- 6 - Define Locations of Pre-placed Cells
- 7 - Placement of Pre-placed Cells
- 8 - IP Blocks and Floorplanning
- 9 - Surrounded Pre-Placed Cells with Decoupling Capacitors
- 10 - Switching Current and Voltage Drop
- 11 - Noise Margin
- 12 - Solution:Add Decoupling Capacitors
- 13 - Decoupling Capacitor Placement Around Blocks
- 14 - Decoupling Capacitor Placement in the Floorplan
- 15 - Power Network,Driver,Load and 16-bit Bus
- 16 - Floorplanning
- 17 - Power Planning
- 18 - Power Distribution Network
- 19 - Picorv32a ASIC Design Flow using OpenLane
- 20 - OpenLane Physical Design Configuration
- 21 - SkyWater PDK LEF File Configuration
- 22 - OpenLane Floorplanning Configuration
- 23 - Standard Cell Placement
- 24 - Decoupling Capacitors
- 25 - Logical Cell Placement Blockage
- 26 - Tap Cells
- 27 - LEF & Technology Files
- 28 - OpenLane Configuration
- 29 - Timing Constraints
- 30 - OpenROAD / Layout View
- 31 - Bind Netlist with Physical Library Cells
- 32 - Placement



## 1. Define Width and Height of Core and Die

The first step in physical design is to understand the netlist and convert the logical representation of the design into physical dimensions.

A netlist describes the connectivity between different components of an electronic design.

The example netlist contains:

* Flip-Flops (FF)
* AND gate
* OR gate
* Clock connection
* Data connections

The standard cells and flip-flops in the netlist are later converted into physical dimensions during floorplanning.
<img width="1920" height="1080" alt="Screenshot (117)" src="https://github.com/user-attachments/assets/99014082-f6ff-4e35-88ee-ec74ea45e9ad" />

---

## 2. Convert Netlist Symbols into Physical Dimensions

After understanding the netlist, the logical components are represented as physical standard cells.

The highlighted elements include:

* Flip-Flops
* Standard cells
* AND/OR logic cells

Each logical cell occupies a certain physical area on the silicon.

Therefore, the total area occupied by all cells must be calculated before determining the core dimensions.
<img width="1920" height="1080" alt="Screenshot (118)" src="https://github.com/user-attachments/assets/7f6fdf41-93b2-4a8a-a023-0f82d896f96b" />

---

## 3. Calculate Area Occupied by the Netlist

For the given example, each standard cell and flip-flop is represented as a unit square.

For example:

```text
Width  = 1 unit
Height = 1 unit

Area = Width × Height
     = 1 × 1
     = 1 sq. unit
```
The total area occupied by the netlist is calculated by adding the area of all standard cells and flip-flops.
This gives the total cell area required inside the core.
<img width="1920" height="1080" alt="Screenshot (119)" src="https://github.com/user-attachments/assets/d657d5b0-e0b9-4d39-a04f-93521dcb6056" />


## 4. Utilization Factor and Aspect Ratio

Two important parameters used to define the core dimensions are:
Utilization Factor
```
Utilization Factor =
Area Occupied by Netlist
-------------------------
Total Area of Core
```
<img width="1920" height="1080" alt="Screenshot (120)" src="https://github.com/user-attachments/assets/211c4821-e0ff-4a35-88b0-c22edb6ab923" />

The utilization factor indicates how much of the core area is occupied by the placed cells.
Aspect Ratio
Aspect Ratio = Height / Width
For the example shown, the core and die dimensions are selected based on the required utilization factor and aspect ratio.
The diagram illustrates a core of approximately 4 units × 2 units with the die surrounding the core.


## 5. Core and Die Dimension Example
Another example demonstrates how the dimensions of the core and die change according to the utilization factor.
The physical dimensions are selected so that sufficient space is available for:
Standard-cell placement
Routing
Power distribution
Decoupling cells
Other physical-design requirements
A lower utilization factor provides more free area inside the core, which can help in reducing placement and routing congestion.
<img width="1920" height="1080" alt="Screenshot (122)" src="https://github.com/user-attachments/assets/f4c04f0c-9bda-410b-8c49-5ebc565f7a0e" />

## 6. Define Locations of Pre-placed Cells
Some cells or blocks cannot be freely placed by the automated placement tool.
These cells are known as pre-placed cells.
Examples include:
Memory
Clock-gating cells
Comparators
Multiplexers
Other large IP blocks
These blocks have user-defined locations and are placed before automated placement and routing.
<img width="1838" height="580" alt="Screenshot (124)" src="https://github.com/user-attachments/assets/7f1ef943-2c7e-4b80-936b-75a91e54e2b6" />

## 7. Placement of Pre-placed Cells
The location of pre-placed cells is important because their position affects the rest of the physical design.
The example shows two blocks containing multiple logic cells.
The cells and their connections are arranged so that the required input/output connections can be maintained.
I/O pins can be extended to make the connectivity between blocks clear.
Proper placement helps reduce:
Routing congestion
Wire length
Timing problems
Unnecessary routing detours
<img width="869" height="734" alt="Screenshot (125)" src="https://github.com/user-attachments/assets/c3672a74-a831-4d00-9375-848b05389ac1" />


## 8. IP Blocks and Floorplanning
Modern ASIC designs may contain several pre-designed IP blocks.
Examples include:
Memory
Clock-gating cells
Comparator
Multiplexer
The arrangement of these IPs or blocks inside the chip is called floorplanning.
These IPs have user-defined locations and are placed in the chip before automated placement and routing.
Therefore, floorplanning determines the physical organization of major blocks inside the chip.

<img width="1175" height="581" alt="Screenshot (127)" src="https://github.com/user-attachments/assets/90606d97-76e4-4046-aea0-fc6383f393e2" />


## 9. Surround Pre-placed Cells with Decoupling Capacitors
Pre-placed blocks may experience high switching activity and can require a large instantaneous current.
To improve power integrity, decoupling capacitors can be placed around the pre-placed cells.
The floorplan contains:
Core
Die
Block A
Block B
Block C
Decoupling capacitor regions
The decoupling capacitors provide a local source of charge near the blocks.
This helps reduce supply voltage fluctuations during switching.

<img width="976" height="707" alt="Screenshot (128)" src="https://github.com/user-attachments/assets/649a14fb-7099-47ea-a629-a24f177578f1" />

## 10. Switching Current and Voltage Drop
During switching operation, a complex digital circuit may demand a large amount of instantaneous current.
This is called peak switching current.
The power supply network contains parasitic resistance and inductance.
When switching current flows through these elements, a voltage drop occurs.
The resistive voltage drop is:
```
V = I × R
```
The inductive voltage variation is:
```
V = L × di/dt
```
Therefore, because of the resistance and inductance of the power network, the voltage available at the circuit can become lower than the ideal supply voltage.
<img width="1309" height="700" alt="Screenshot (129)" src="https://github.com/user-attachments/assets/4c5a373c-cb90-42e4-bc8f-f3ad0b2d2792" />

## 11. Noise Margin 
Noise margin represents the ability of a digital circuit to tolerate unwanted voltage disturbances without changing the interpreted logic value.
The two important noise margins are:
Noise Margin High
```
NMH = VOH(min) − VIH(min)
```
Noise Margin Low
```
NML = VIL(max) − VOL(max)
```
The diagram shows different noise-induced bumps and their relationship with the noise-margin levels.
A small noise bump remains within the acceptable region and does not cause a logic error.
If the noise exceeds the available noise margin, it may be interpreted as an unwanted logic transition.

<img width="1084" height="584" alt="Screenshot (130)" src="https://github.com/user-attachments/assets/eb0fa1ce-b52d-4bd0-b78b-58f30ebdd24e" />


## 12. Solution: Add Decoupling Capacitors
One solution for reducing the effect of switching-current demand is to add a decoupling capacitor.
The decoupling capacitor is connected in parallel with the circuit.
When the circuit switches and requires a large current, the capacitor can provide charge locally.
The power network then replenishes the charge stored in the capacitor.
```
Switching occurs
       ↓
Circuit demands current
       ↓
Decoupling capacitor supplies charge
       ↓
Power network replenishes the charge
```
This reduces the effect of sudden current demand on the supply voltage.

<img width="1291" height="698" alt="Screenshot (133)" src="https://github.com/user-attachments/assets/45ff5865-832c-46ff-bea8-be3e88695fa6" />


## 13. Decoupling Capacitor Placement Around Blocks
Decoupling capacitors can be placed around important pre-placed blocks.
For example:
+--------------------------------+
|                                |
|        DECAP1                  |
|        Block A   Block B       |
|        DECAP2                  |
|        Block C                 |
|        DECAP3                  |
|                                |
+--------------------------------+
The purpose is to keep the decoupling capacitors close to the blocks that require additional instantaneous current.
This provides a local current source and improves power integrity.
<img width="1342" height="583" alt="Screenshot (134)" src="https://github.com/user-attachments/assets/37256ea8-fe0c-4be8-b6c9-793212230bb4" />


## 14. Decoupling Capacitor Placement in the Floorplan
The floorplan can be organized using different regions for blocks and decoupling capacitors.
```
The example shows:
DECAP1
Block A
Block B
DECAP2
Block C
DECAP3
```
The decoupling capacitors are strategically placed around the pre-placed cells.
Proper placement helps reduce the distance between the capacitor and the switching circuit.
A shorter current path helps reduce the impact of parasitic resistance and inductance.
<img width="973" height="573" alt="Screenshot (135)" src="https://github.com/user-attachments/assets/b7a7290e-28e2-416d-be5e-697e99b83ac1" />


## 15. Power Network, Driver, Load and 16-bit Bus
The final example illustrates the power network connecting multiple driver and load circuits.
The power distribution network contains resistance and inductance.
Each circuit has associated capacitance and switching requirements.
The diagram also illustrates a signal path representing a multi-bit bus.
For the given example, the blue path represents a 16-bit bus.
The key concept is that switching activity across multiple signals can create a large instantaneous current demand.
Therefore, proper power-network design and decoupling are required to maintain stable supply voltage and reliable circuit operation.
<img width="1144" height="600" alt="Screenshot (136)" src="https://github.com/user-attachments/assets/20c93dc0-0302-476e-855b-faa3aee40e1c" />


## 📐 16. Floorplanning
Floorplanning is one of the first major physical-design stages.
The floorplan determines:
```
Die dimensions
Core dimensions
Core utilization
Aspect ratio
Standard-cell placement region
I/O placement region
Power distribution requirements
Important Floorplan Parameters
```
The configuration shown in the project includes parameters related to:
```
FP_CORE_UTIL
FP_ASPECT_RATIO
FP_SIZING
DIE_AREA
FP_IO_HMETAL
FP_IO_VMETAL
FP_IO_MODE
```
Core Utilization
```
FP_CORE_UTIL
```
 determines the approximate percentage of the core area occupied by standard cells.
A suitable utilization value is important because excessive utilization can make routing difficult, while very low utilization can increase chip area.

<img width="1229" height="583" alt="Screenshot (137)" src="https://github.com/user-attachments/assets/50da70e4-0414-4910-86fd-9266ffe07301" />


## ⚡17. Power Planning
Power planning creates the power-distribution network (PDN) required to deliver stable supply voltages throughout the chip.
The screenshots show a power grid consisting of horizontal and vertical metal structures.
Typical power connections include:
```
VDD
│
├── Horizontal Power Rails
│
├── Vertical Power Rails
│
└── Standard Cell Power Connections

VSS
│
├── Horizontal Ground Rails
│
├── Vertical Ground Rails
│
└── Standard Cell Ground Connections
```

The power grid helps reduce voltage drop and provides reliable power delivery to the cells distributed across the core.

<img width="977" height="592" alt="Screenshot (138)" src="https://github.com/user-attachments/assets/6c65e942-8ee9-48a6-9e1b-b0d39fb58b5b" />

## 🔌18. Power Distribution Network (PDN)
The project examines the physical organization of power structures across the chip core.
The PDN consists of:
Horizontal power straps
Vertical power straps
Standard-cell power rails
VDD connections
VSS connections
Power grid intersections
A well-designed PDN is essential for:
Reducing IR drop
Improving power integrity
Providing uniform supply voltage
Supporting reliable standard-cell operation

<img width="1035" height="582" alt="Screenshot (142)" src="https://github.com/user-attachments/assets/d20df03f-6735-4feb-9788-f91c82fa8c63" />

## 19. Picorv32a ASIC Design Flow using OpenLane
OpenLane is an automated, open-source RTL-to-GDSII hardware design framework. It automatically transforms human-readable hardware description code (Verilog RTL) into the final physical layout blueprint (GDSII) required to manufacture a physical silicon microchip.The picorv32a is an optimized RISC-V CPU core used as a design benchmark in this automated flow.The config.tcl file acts as the configuration hub for this process. It defines critical hardware parameters—such as target layout names, input file paths, and the required clock speeds—to guide the software engines through synthesis, placement, and routing without human intervention.

<img width="958" height="934" alt="less config tcl 4th" src="https://github.com/user-attachments/assets/6290e875-280f-4602-b43a-8b427850888a" />


## 20. Openlane Physical Design Configuration (sky130_fd_sc_hd)
In the OpenLane ASIC design flow, the hardware description language (HDL) code is transformed into a physical layout. This process relies heavily on configuration files (.tcl) to define constraints and optimization goals for the synthesis, floorplanning, placement, and routing stages.The configuration snippet specifically targets the sky130_fd_sc_hd standard cell library (SkyWater 130nm High Density) and establishes several foundational parameters:Synthesis & Timing Control: Variables like SYNTH_MAX_FANOUT define the maximum number of digital inputs that a single logic gate output can drive, balancing signal integrity and delay. The CLOCK_PERIOD sets the targeted clock cycle time in nanoseconds, defining the performance constraint for static timing analysis (STA).Floorplanning & Density: The utilization variables specify how much of the core area will be occupied by standard cells. The core utilization (FP_CORE_UTIL) sets the initial budget, while PL_TARGET_DENSITY dynamically calculates the targeted placement density, ensuring cells are optimally packed without causing unroutable congestion during the physical implementation stage.

<img width="958" height="934" alt="less sky130A_sky130_fd_sc_hd_config tcl 5th image" src="https://github.com/user-attachments/assets/9421bc1b-2684-44ce-87c5-92d51a2d3396" />



## 21.SkyWater PDK LEF File Configuration
This section provides an overview of the structural and physical parameters defined in the merged_upadded.lef file for the SkyWater PDK open-source digital design flow.📝 Theory & File OverviewA LEF (Library Exchange Format) file contains the abstract physical layout information of a cell library. It provides the place-and-route tools with necessary physical data—such as cell boundaries, pin locations, and metal layers—without exposing the internal silicon-level geometric details.The file snippet shown defines foundational technology attributes, manufacturing grids, and macro site definitions required for structural placement:Technology Units: Establishes standard scaling units for the design data, mapping database units to physical physical metrics (e.g., 1000 database microns per millimeter, standard resistance in Ohms, time in nanoseconds, and capacitance in picofarads).Manufacturing Grid: Sets the geometric granularity (0.005 microns) for placing geometry features. All layouts and wiring components must align precisely with this grid.Site Definitions: Outlines the placement row templates for cells. It characterizes standard properties like symmetry, layer types, and core dimensions for both high-density single-height (unithd) and double-height (unithddbl) library cells.
<img width="958" height="934" alt="less merged_upadded lef 6th image" src="https://github.com/user-attachments/assets/7a9cdff1-bfda-46ea-b364-87ba12d0fb88" />



## 📜 22.OpenLane Floorplanning Configuration
This section outlines the primary configuration variables required to control and customize the floorplanning stage within the OpenLane automated RTL-to-GDSII flow.
### 🔍 Core Configuration Parameters
### 🏗️ Die & Core Area Definition
FP_CORE_UTIL: The core utilization percentage. It defines how densely the standard cells are packed within the core area. (Default: 50 percent).

FP_ASPECT_RATIO: Controls the physical shape of the core by setting the ratio of its height to its width. (Default: 1).

FP_SIZING: Dictates the strategy for sizing the core and die area. It determines whether to use a relative scaling method based on core utilization (FP_CORE_UTIL) or an absolute size definition. It accepts both "relative" and "absolute" configurations.

DIE_AREA: Explicitly defines the boundary coordinates of the outer die block for floorplanning. It is specified as a 4-corner rectangle layout. Units are measured in millimeters (mm) or micrometers (μm). (Note: This overrides relative utilization sizing if manually defined).
### 📌 Pin & IO Placement
FP_IO_HMETAL: Specifies the specific metal layer assigned to route the horizontal IO pins on the top and bottom edges of the die block. (Default: 4).
FP_IO_VMETAL: Specifies the specific metal layer assigned to route the vertical IO pins on the left and right sides of the die block. (Default: 3).
FP_IO_MODE: Determines the strategy for random IO pin placement. Setting it to 0 enables matching node placements, while 1 triggers random but equidistant placement along the core boundaries. (Default: 1).
FP_IO_MIN_DISTANCE: Sets the absolute minimum physical spacing required between adjacent IO pins to avoid manufacturing design rule errors.
### ⚡ Power Distribution Network (PDN) & Pitch
FP_WELLTAP_CELL / FP_ENDCAP_CELL: The specific physical layout cell names used for tap and endcap insertion to prevent latch-up conditions.
FP_PDN_VOFFSET / FP_PDN_HOFFSET: The offset measurements applied to the vertical and horizontal power stripes relative to the design origin.
FP_PDN_VPITCH / FP_PDN_HPITCH: The recurring pitch/distance between parallel vertical and horizontal power stripes across the metal stack layers.
FP_PDN_AUTO_ADJUST: A boolean switch determining if the flow should automatically scale and adjust the power grid layout to match the core boundaries when adjustments are necessary. (Default: 1 [Enabled]).
### 🧲 Taps, Tie-offs, and IO Extensions
FP_TAPCELL_DIST: Defines the horizontal distance limits between adjacent welltap columns across the layout row structures. (Default: 14).
FP_IO_VEXTEND / FP_IO_HEXTEND: Extends the routing pins slightly outside the core/die perimeter to make external macro routing simpler.
FP_IO_VLENGTH / FP_IO_HLENGTH: Dictates the absolute length of vertical and horizontal physical pins. (Default: 4).
FP_IO_VTHICKNESS_MULT / FP_IO_HTICKNESS_MULT: A multiplier value scaling the thickness of pins over the standard minimum layer widths. (Default: 2).

<img width="958" height="934" alt="less README md  openlane danlo configuration danlo undi 7th" src="https://github.com/user-attachments/assets/92a81c0e-8990-4607-b382-06a17cfa4e68" />


## 🧱 23. Standard Cell Placement
After floorplanning and power planning, logical cells are placed inside the core region.
The placement process determines the physical location of:
Combinational cells
Sequential cells
Buffers
Inverters
Logic gates
Other standard cells
The screenshots demonstrate the placement of cells in organized rows inside the defined core area.
<img width="1920" height="944" alt="layout 8th" src="https://github.com/user-attachments/assets/c04a62c5-3d6e-4d22-bc0e-28c3c796d419" />


<img width="958" height="934" alt="what command 9th image" src="https://github.com/user-attachments/assets/4944eeaf-9a68-40ab-aae3-a18f0be41b88" />
Good placement is important for achieving:
Shorter interconnects
Better timing
Lower congestion
Efficient routing
Lower power consumption
<img width="1920" height="944" alt="10th" src="https://github.com/user-attachments/assets/1ba203df-ed8d-4240-ba79-c06969145f08" />


## 🔋24. Decoupling Capacitors
The floorplan diagrams also illustrate the placement of decoupling capacitor structures.
Decap cells help stabilize the local power supply by providing charge close to switching logic.
They are particularly useful for:
Reducing local supply noise
Supporting transient current demand
Improving power integrity
The screenshots show decap-related structures distributed within the physical design.

## 🧩 25. Logical Cell Placement Blockage
Placement blockages are used to prevent standard cells from being placed in selected regions.
The project demonstrates a logical-cell placement blockage region around specific structures.
Example concept:
+--------------------------------------+
|                                      |
|   Standard Cell Placement Area       |
|                                      |
|          +----------------+          |
|          |   BLOCKAGE     |          |
|          |                |          |
|          |   DECAP /      |          |
|          |   SPECIAL      |          |
|          |   CELLS        |          |
|          +----------------+          |
|                                      |
+--------------------------------------+
Placement blockages can be useful for:
Protecting special structures
Reserving routing resources
Managing congestion
Creating space for macros or other physical structures

## 🧠26. Tap Cells
Tap cells are an important part of the physical implementation of CMOS designs.
They provide well/substrate connections required by the standard-cell technology and help prevent issues such as latch-up.
Tap-cell insertion is therefore an important physical-design step before final routing and verification.

## 📚 27. LEF & Technology Files
The project uses technology and library information associated with the SKY130A PDK.
The screenshots show investigation of technology files such as:
sky130A.tech
sky130_fd_sc_hd__...
*.lef
LEF files provide physical information about cells, including:
Cell dimensions
Pin locations
Metal layers
Routing information
Obstructions
Physical abstracts
Technology files describe the physical rules and layers required by the implementation tools.

## ⚙️ 28. OpenLane Configuration
The project uses Tcl-based OpenLane configuration files to define the physical-design flow.
Typical configuration variables include:
```
set ::env(DESIGN_NAME) "picorv32a"

set ::env(VERILOG_FILES) \
    "$::env(DESIGN_DIR)/src/picorv32a.v"

set ::env(SDC_FILE) \
    "$::env(DESIGN_DIR)/src/picorv32a.sdc"

set ::env(CLOCK_PERIOD) "5.000"
set ::env(CLOCK_PORT) "clk"

set ::env(FP_CORE_UTIL) 50
set ::env(FP_ASPECT_RATIO) 1
```
Note: The exact values should be updated to match the final configuration used in the repository.

## 🕒 29. Timing Constraints
The design uses an SDC file to define timing constraints.
Important timing parameters include:
Clock period
Clock port
Input delays
Output delays
Timing uncertainty
For example:
```
create_clock \
    -name clk \
    -period 5.0 \
    [get_ports clk]
```
A correct timing constraint setup is necessary for timing-driven synthesis, placement, and routing.

## 🖥️ 30. OpenROAD / Layout View
The physical layout can be inspected using OpenROAD-based tools.
The screenshots demonstrate a layout containing:
Standard-cell rows
Power structures
Cell instances
Core boundaries
I/O regions
Metal layers
The layout view allows the physical implementation to be visually inspected before final signoff.


## 31.Bind netlist with physical library cells
Logical cells such as FF1, FF2, etc. are mapped to their corresponding physical standard cells from the technology library.
<img width="445" height="540" alt="WhatsApp Image 2026-09-06 at 6 57 22 PM" src="https://github.com/user-attachments/assets/085d9aa5-fe10-43d4-9968-ef8adb81f6f7" />
<img width="1080" height="552" alt="WhatsApp Image 2026-09-06 at 6 57 22 PM (1)" src="https://github.com/user-attachments/assets/4f5f18e5-28a7-40dc-9bb7-41f2a7d38ae7" />


## 32.Placement
The bound cells are physically positioned inside the floorplan.
Placement considers connectivity between cells, available rows/sites, blockages, I/O pins, and timing/congestion.
The examples show the same logical structure represented as: Netlist → Physical placement → Physical view of logic gates.
<img width="1080" height="595" alt="WhatsApp Image 2026-09-06 at 6 57 24 PM" src="https://github.com/user-attachments/assets/3dcc51e5-06cc-414d-bce4-2ac7aba59c08" />

and the floorplan containing:
Standard-cell rows
Input/output pins such as Din1, Din2, Din3, Din4
Clock pins Clk1, Clk2
Fixed/placed cells such as FF1, FF2
Blockages/macros such as DECAP1, DECAP3, etc.
<img width="1080" height="586" alt="WhatsApp Image 2026-09-06 at 6 57 24 PM (1)" src="https://github.com/user-attachments/assets/3b1d8217-ee9f-481e-97dd-946f6c1e1fb6" />


# Key Learnings
Through this module, the following concepts were studied:
Understanding a netlist
Converting logical cells into physical dimensions
Cell area calculation
Core and die dimensions
Utilization factor
Aspect ratio
Pre-placed cells
IP blocks
Floorplanning
Placement of pre-placed cells
Switching current
Peak current
IR/voltage drop due to resistance and inductance
Noise margin
Noise-induced voltage bumps
Decoupling capacitors
Decoupling capacitor placement
Power integrity
Driver and load connectivity
Multi-bit bus considerations


# Conclusion
Chip floorplanning is a critical stage of ASIC physical design. The dimensions of the core and die must be selected based on cell area, utilization factor, and aspect ratio.
Pre-placed cells and IP blocks must be positioned carefully to achieve good connectivity and efficient routing.
During switching, large instantaneous current demand can cause voltage fluctuations because of resistance and inductance in the power network. Noise margin determines the ability of the circuit to tolerate such disturbances.
Decoupling capacitors provide local charge during high-current switching events and help maintain supply stability.
Thus, proper floorplanning, pre-placed-cell placement, power planning, noise-margin analysis, and decoupling-capacitor placement are essential for reliable physical design.

## 👤 Author

**JADHAV KAVYA**  
B.Tech – Electronics & Communication Engineering  
Anurag University  




