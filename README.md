# VSD SoC Design & Planning Workshop

## RTL to GDSII | OpenLANE | Sky130

This repository contains my notes, experiments, screenshots, and learning from the **SoC Design and Planning Workshop** focused on the complete **RTL-to-GDSII physical design flow**.

The workshop gave me an opportunity to move beyond just understanding VLSI concepts theoretically and actually work with open-source EDA tools. Throughout the sessions, I worked with the `picorv32a` RISC-V design and explored how a design moves from RTL to a physical chip layout.

---

## What I Explored

During the workshop, I worked through the major stages of a digital ASIC flow:

* RISC-V and SoC fundamentals
* Open-source EDA flow
* RTL synthesis
* Floorplanning
* Standard-cell placement
* Custom standard-cell design
* SPICE simulation and characterization
* Static Timing Analysis (STA)
* Clock Tree Synthesis (CTS)
* Power Distribution Network (PDN)
* Global and detailed routing
* Design Rule Checking (DRC)
* Post-route timing analysis
* RTL-to-GDSII implementation

The overall flow can be thought of as:

**RTL → Synthesis → Floorplan → Placement → CTS → Routing → Verification → GDSII**

---

# Day 1 — Getting Started with OpenLANE and RISC-V

The first day was mainly about understanding what actually happens between a software program and a physical chip.

I started with the basic structure of a chip package, including the **die, core, pads and IP blocks**. I also learned how the core contains the actual digital logic while the pads provide communication with the outside world.

### RISC-V and the Software-to-Hardware Journey

One of the interesting parts was understanding how software eventually becomes hardware operations.

A simplified flow is:

**C Program → Assembly → Machine Code → RTL → Synthesis → Place & Route → GDSII**

I also got introduced to **RISC-V**, its ISA, and its role in connecting software instructions with hardware implementation.

### Open-Source ASIC Flow

I learned that an open-source digital ASIC flow mainly depends on:

* RTL design
* EDA tools
* PDK
* Standard-cell libraries

The workshop used the **Sky130 PDK** along with several open-source tools.

### OpenLANE Flow

OpenLANE brings several tools together to automate the RTL-to-GDSII process.

| Stage           | Tool                   |
| --------------- | ---------------------- |
| Synthesis       | Yosys, ABC             |
| Floorplanning   | OpenROAD               |
| Placement       | OpenROAD               |
| CTS             | TritonCTS              |
| Routing         | FastRoute, TritonRoute |
| Timing Analysis | OpenSTA                |
| Layout / DRC    | Magic                  |
| LVS             | Netgen                 |
| GDS Generation  | Magic / KLayout        |

### First OpenLANE Run

I worked with the `picorv32a` design and explored OpenLANE in interactive mode.

```bash
./flow.tcl -interactive
```

The design was then prepared using:

```tcl
prep -design picorv32a
```

After preparation, synthesis was performed using:

```tcl
run_synthesis
```

Synthesis converts the RTL into a gate-level representation using cells from the available standard-cell library.

### Flop Ratio

As part of analysing the synthesis result, I calculated the flop ratio:

```text
Flop Ratio = Number of D Flip-Flops / Total Number of Cells

             = 1613 / 15762
             ≈ 10.23%
```

This helped me understand how much sequential logic was present in the synthesized design.

---

# Day 2 — Floorplanning, Placement and Library Cells

The second day shifted from RTL to the physical side of chip design.

### Floorplanning

Floorplanning decides the basic physical organization of the chip.

Some important concepts I explored were:

* Core area
* Utilization factor
* Aspect ratio
* I/O pin placement
* Pre-placed cells
* Placement blockages
* Power planning
* Decoupling capacitors

The utilization factor can be represented as:

```text
Utilization = Netlist Area / Core Area
```

The aspect ratio is:

```text
Aspect Ratio = Height / Width
```

These parameters have a direct impact on routing, congestion and the overall physical implementation.

### Running Floorplan

```tcl
run_floorplan
```

I then explored the generated DEF files and used **Magic** to visualize the physical floorplan.

### Placement

After floorplanning, standard cells were placed inside the available core area.

```tcl
run_placement
```

Placement tries to achieve a good balance between:

* Wire length
* Timing
* Congestion
* Cell connectivity

This was one of the stages where the difference between logical design and physical implementation became much clearer to me.

---

# Day 3 — CMOS Inverter and Standard-Cell Characterization

Day 3 focused more on the transistor and standard-cell level.

### CMOS Inverter

I worked with a CMOS inverter containing:

* PMOS
* NMOS
* Power supply
* Input
* Output

A SPICE representation was created and simulated to understand the electrical behaviour of the inverter.

### Important Timing Parameters

From the simulation waveform, I studied:

* Rise time
* Fall time
* Propagation delay

For example:

```text
Rise Time = Time at 80% − Time at 20%
```

and propagation delay is measured around the 50% voltage points.

### Magic and SPICE

I also worked with Magic to inspect the standard-cell layout and extract a SPICE netlist.

The extracted circuit was then simulated using **ngspice**:

```bash
ngspice sky130_inv.spice
```

The generated waveform was used to understand the timing behaviour of the inverter.

### CMOS Fabrication Concepts

I also got an overview of the CMOS fabrication sequence, including:

* Active region formation
* N-well / P-well formation
* Gate formation
* Source and drain formation
* Contacts
* Metal layers
* Passivation

This helped connect the layout I was seeing in Magic with the actual physical fabrication process.

---

# Day 4 — Custom Cell Integration, STA and CTS

Day 4 was focused heavily on timing and clock distribution.

### LEF and Standard-Cell Integration

I learned how a custom standard cell can be represented using a **LEF file** so that physical-design tools can understand its:

* Cell boundary
* Pins
* Routing information
* Physical dimensions

The custom inverter cell was integrated into the `picorv32a` flow.

### Static Timing Analysis

STA was one of the most important concepts covered during the workshop.

I learned about:

* Setup time
* Setup slack
* Hold time
* Hold slack
* Clock uncertainty
* OCV
* CRPR
* Critical paths

Setup slack can be represented as:

```text
Setup Slack = Data Required Time − Data Arrival Time
```

A setup violation occurs when the data arrives too late.

### OpenSTA

I used **OpenSTA** to perform pre-CTS timing analysis and examine timing reports.

```bash
sta pre_sta.conf
```

This gave me a practical understanding of how timing constraints and standard-cell libraries are used during STA.

### Clock Tree Synthesis

After timing analysis, I explored **Clock Tree Synthesis (CTS)**.

```tcl
run_cts
```

CTS builds a clock distribution network using buffers so that the clock can reach different sequential elements with controlled skew and delay.

I also learned why timing needs to be checked again after CTS because the clock network becomes physically realistic after buffer insertion.

---

# Day 5 — Power Distribution, Routing and Final Implementation

The final stage focused on power delivery and routing.

### Power Distribution Network

A Power Distribution Network (PDN) provides the required:

* VDD
* VSS

connections throughout the design.

The PDN was generated using:

```tcl
gen_pdn
```

I learned how power moves from the larger chip-level network down to the standard-cell power rails.

### Global and Detailed Routing

Routing takes place in two major stages.

**Global Routing**

Finds approximate paths and creates routing guides while considering congestion.

**Detailed Routing**

Converts those guides into actual metal and via connections while satisfying design rules.

The routing stage was executed using:

```tcl
run_routing
```

### Routing and DRC

I also learned about common physical-design problems such as:

* Minimum spacing violations
* Antenna violations
* Connectivity issues
* Design-rule violations

DRC is important because the final physical layout must follow the rules of the selected fabrication technology.

---

# Tools Used

| Tool        | What I Used It For                                   |
| ----------- | ---------------------------------------------------- |
| OpenLANE    | Overall RTL-to-GDSII flow                            |
| Yosys       | RTL synthesis                                        |
| OpenROAD    | Floorplanning, placement and physical implementation |
| Magic       | Layout viewing and DRC                               |
| OpenSTA     | Static timing analysis                               |
| ngspice     | Circuit simulation                                   |
| TritonCTS   | Clock Tree Synthesis                                 |
| TritonRoute | Detailed routing                                     |
| Netgen      | LVS                                                  |
| Sky130 PDK  | Technology and process information                   |

These tools together provided the open-source environment used throughout the workshop.

---

# My Key Takeaways

This workshop gave me a much clearer picture of what happens inside a digital chip before it can actually be manufactured.

The main things I learned were:

* How software instructions eventually connect to hardware implementation
* How RISC-V fits into an SoC
* How RTL is converted into a gate-level netlist
* How floorplanning and placement affect physical design
* How standard cells are designed and characterized
* How SPICE helps analyse transistor-level behaviour
* How setup and hold timing are checked using STA
* Why CTS is necessary for clock distribution
* How power is distributed across a chip
* How global and detailed routing work
* Why DRC and timing verification are essential before sign-off

Most importantly, I got hands-on exposure to the **complete RTL-to-GDSII flow**, rather than looking at each topic only from a theoretical point of view.

---

# Workshop Outcome

The workshop helped me connect several concepts that I had previously studied separately.

Starting from a RISC-V based RTL design, I was able to follow the design through **synthesis, floorplanning, placement, standard-cell integration, timing analysis, CTS, power planning and routing**.

The biggest takeaway for me was understanding that designing a chip is not just about writing RTL. A working design also has to satisfy physical, timing, power and manufacturing constraints before it can become real silicon.

---

## Acknowledgements

I would like to thank **VLSI System Design (VSD)**, **NASSCOM**, and the instructors and contributors involved in the workshop for providing such a practical learning experience.

Special thanks to **Kunal Ghosh** and **Nickson Jose** for the guidance, tools and hands-on exercises that made the workshop much more useful than a purely theoretical course.

---

## References

* VSD SoC Design and Planning Workshop
* OpenLANE / OpenROAD documentation
* Sky130 PDK documentation
* VSD Standard Cell Design repository

---

### Final Note

This repository represents my learning and hands-on work during the workshop. The screenshots, commands, observations and notes are included to document my progress through the RTL-to-GDSII flow.
# openlane
