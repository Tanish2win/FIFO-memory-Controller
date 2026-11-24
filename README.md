# FIFO Memory Controller — RTL to GDS

> **A digital design project exploring FIFO memory architecture, RTL design, functional verification, logic synthesis, floorplanning, physical implementation, and GDSII generation using open-source VLSI tools.**

---

## 1. Project Overview

This project aims to design and implement a **Synchronous FIFO (First-In, First-Out) Memory Controller** from architectural specification to physical layout.

The project follows a complete ASIC design workflow, covering both the **Front-End Design** and **Back-End Physical Design** stages. The objective is not merely to write functional RTL, but to understand how a digital memory controller is designed, verified, synthesized, physically implemented, and prepared for fabrication.

The design will be developed incrementally, with each stage documented and its results committed to this repository.

## 2. Objectives

- Design a synchronous FIFO memory controller using Verilog/SystemVerilog.
- Understand FIFO memory organization and circular pointer architecture.
- Implement read and write control logic.
- Develop full and empty detection mechanisms.
- Verify functionality through simulation and testbenches.
- Perform RTL synthesis and analyze the synthesized netlist.
- Explore ASIC floorplanning and physical constraints.
- Perform placement, clock-tree synthesis, and routing.
- Analyze timing, area, and physical-design reports.
- Generate the final layout in GDSII format using an open-source RTL-to-GDS flow, subject to tool and design compatibility.

## 3. Initial Design Specifications

| Parameter | Specification |
|---|---|
| Design | Synchronous FIFO Memory Controller |
| FIFO Depth | 16 entries |
| Data Width | 8 bits |
| Total Storage | 128 bits |
| Address Width | 4 bits |
| Pointer Width | 5 bits |
| Clock | Single clock domain |
| Reset | Synchronous, active-high |
| Write Control | `wr_en` |
| Read Control | `rd_en` |
| Input Data | `din[7:0]` |
| Output Data | `dout[7:0]` |
| Status Flags | `full`, `empty` |
| RTL Language | Verilog/SystemVerilog |
| Target PDK | SkyWater SKY130 (planned) |
| Physical Design Flow | OpenLane-based flow (planned) |

*These are the initial baseline specifications and may be revised during the architecture and implementation stages.*

## 4. FIFO Architecture

The FIFO follows the First-In, First-Out principle. Data is written into memory sequentially and retrieved in the same order in which it was stored.

### Functional Blocks

| Block | Description |
|---|---|
| FIFO Memory Array | Stores incoming data entries |
| Write Pointer | Tracks the next write location |
| Read Pointer | Tracks the next read location |
| Control Logic | Controls write/read operations |
| Status Logic | Generates full and empty flags |
| Reset Logic | Initializes controller state |

### Conceptual Operation

**Write operation**
1. The controller receives input data.
2. A write request is asserted.
3. If the FIFO is not full, data is stored at the current write address.
4. The write pointer advances.

**Read operation**
1. A read request is asserted.
2. If the FIFO is not empty, stored data is retrieved.
3. The read pointer advances.

**Status operation**
- `full` indicates that the FIFO cannot accept another write.
- `empty` indicates that no valid data is available to read.

## 5. Interface Signals

| Signal | Direction | Width | Description |
|---|---|---:|---|
| `clk` | Input | 1 | System clock |
| `rst` | Input | 1 | Active-high synchronous reset |
| `wr_en` | Input | 1 | Write request |
| `rd_en` | Input | 1 | Read request |
| `din` | Input | 8 | Input data |
| `dout` | Output | 8 | Output data |
| `full` | Output | 1 | FIFO full indicator |
| `empty` | Output | 1 | FIFO empty indicator |

## 6. Design Flow

The project will follow a structured RTL-to-GDS implementation flow.

```text
Design Specification
        │
        ▼
Architecture & Block Diagram
        │
        ▼
RTL Development
        │
        ▼
Testbench Development
        │
        ▼
Functional Simulation
        │
        ▼
Debugging & Verification
        │
        ▼
RTL Synthesis
        │
        ▼
Gate-Level Netlist
        │
        ▼
Floorplanning
        │
        ▼
Placement & Optimization
        │
        ▼
Clock Tree Synthesis
        │
        ▼
Routing
        │
        ▼
Physical Verification
        │
        ▼
GDSII Generation
```

## 7. Tools & Technologies

The project intends to use open-source tools wherever practical.

| Tool | Purpose |
|---|---|
| VS Code | RTL development and repository management |
| Git & GitHub | Version control and project documentation |
| Icarus Verilog | RTL compilation and simulation |
| GTKWave | Waveform visualization |
| Yosys | Logic synthesis |
| OpenLane | RTL-to-GDS physical design flow |
| SkyWater SKY130 | Open-source process design kit |

Tool versions, installation instructions, configurations, and issues encountered will be documented as the project progresses.

## 8. Verification Strategy

The FIFO controller will be tested against normal operating conditions, boundary conditions, and invalid operation attempts.

Planned test scenarios include:

- Reset initialization
- Single write operation
- Single read operation
- Multiple consecutive writes
- Multiple consecutive reads
- FIFO filling to capacity
- FIFO empty condition
- FIFO full condition
- Read and write pointer wraparound
- Read request while empty
- Write request while full
- Simultaneous read and write
- Data ordering verification
- Randomized stimulus (planned)

Simulation results will be used to verify the correctness of the design before synthesis.

## 9. Physical Design & Floorplanning

After functional verification and synthesis, the design will proceed toward physical implementation.

The back-end phase will explore:

- Die and core dimensions
- Core utilization
- Placement rows
- Pin placement
- Power planning
- Standard-cell placement
- Timing optimization
- Clock-tree synthesis
- Signal routing
- Parasitic extraction
- Design Rule Checks (DRC)
- Layout-versus-schematic checks (where supported)
- Final GDSII generation

The physical implementation will be analyzed through reports and layout visualizations, with attention to area, timing, congestion, and routing.

## 10. Ten-Day Development Roadmap

| Day | Task | Phase |
|---|---|---|
| Day 1 | Architecture, specification, repository setup | Planning |
| Day 2 | FIFO RTL implementation | Front-End |
| Day 3 | Testbench development and simulation | Verification |
| Day 4 | Debugging and functional validation | Verification |
| Day 5 | Logic synthesis and netlist analysis | Synthesis |
| Day 6 | Floorplanning and physical constraints | Back-End |
| Day 7 | Placement and optimization | Back-End |
| Day 8 | Clock-tree synthesis and routing | Back-End |
| Day 9 | Physical validation and GDS generation | Back-End |
| Day 10 | Documentation, analysis, and final presentation | Finalization |

## 11. Repository Structure

```text
FIFO-Memory-Controller-RTL-to-GDS/
│
├── README.md
├── LICENSE
├── .gitignore
│
├── docs/
│   ├── architecture.md
│   ├── specification.md
│   ├── block_diagram/
│   └── images/
│
├── rtl/
│   └── fifo_controller/
│
├── tb/
│   └── fifo_controller/
│
├── sim/
│
├── synthesis/
│
├── constraints/
│
├── physical_design/
│   ├── floorplan/
│   ├── placement/
│   ├── routing/
│   └── reports/
│
└── results/
    ├── waveforms/
    ├── reports/
    └── screenshots/
```

## 12. Project Status

| Stage | Status |
|---|---|
| Project planning | In progress |
| Architecture definition | In progress |
| RTL development | Not started |
| Functional verification | Not started |
| Synthesis | Not started |
| Floorplanning | Not started |
| Placement | Not started |
| Clock-tree synthesis | Not started |
| Routing | Not started |
| Physical verification | Not started |
| GDSII generation | Not started |

We will update the status as each stage is completed and verified.

## 13. Engineering Documentation

This repository will document the engineering process, not only the final output.

Documentation will include:

- Design decisions and their reasoning
- Architecture diagrams
- RTL implementation details
- Testbench methodology
- Simulation waveforms
- Synthesis reports
- Floorplan screenshots
- Timing and area analysis
- Physical implementation reports
- Issues encountered and debugging solutions

## 14. Final Goal

By the end of this project, we aim to have a working FIFO memory controller, a verified RTL implementation, a synthesized netlist, a physical floorplan, and a documented RTL-to-GDS implementation flow.

The project is intended to build practical understanding of digital design, memory control, ASIC implementation, and physical design through a reproducible, hands-on engineering workflow.

---

**Project Type:** Digital Design / VLSI / ASIC Physical Design  
**Design Approach:** RTL-first, simulation-driven, synthesis and physical implementation  
**Repository:** FIFO-Memory-Controller-RTL-to-GDS  
**Development Schedule:** 10 Days  
**Target Technology:** SkyWater SKY130 (planned)
