# FIFO Memory Controller — Functional Specification

## 1. Purpose

This document defines the functional requirements, operating conditions, interfaces, and initial implementation parameters for the synchronous FIFO memory controller.

The specification serves as the reference for RTL development, simulation, verification, synthesis, and physical implementation.

## 2. Design Configuration

| Parameter | Value |
|---|---|
| FIFO type | Synchronous FIFO |
| Data width | 8 bits |
| FIFO depth | 16 entries |
| Storage capacity | 128 bits |
| Address width | 4 bits |
| Pointer width | 5 bits |
| Clock domains | 1 |
| Reset type | Synchronous, active-high |
| Read operation | Controlled by `rd_en` |
| Write operation | Controlled by `wr_en` |

## 3. Interface Definition

### Inputs

| Signal | Width | Description |
|---|---:|---|
| `clk` | 1 | System clock |
| `rst` | 1 | Active-high synchronous reset |
| `wr_en` | 1 | Write request |
| `rd_en` | 1 | Read request |
| `din` | 8 | Data input |

### Outputs

| Signal | Width | Description |
|---|---:|---|
| `dout` | 8 | Data output |
| `full` | 1 | FIFO full status |
| `empty` | 1 | FIFO empty status |

## 4. Clock Requirements

All stateful operations are synchronized to the rising edge of `clk`.

The initial implementation uses a single clock domain for:

- Pointer updates
- Memory write operations
- Read output updates
- Status flag updates
- Reset handling

No asynchronous clock crossing is included in the baseline design.

## 5. Reset Requirements

Reset is synchronous and active-high.

When `rst` is asserted at the rising clock edge:

- Write pointer shall be reset to zero.
- Read pointer shall be reset to zero.
- FIFO shall be marked empty.
- FIFO shall not be marked full.
- Output data shall be reset to zero.
- Controller shall return to its initial operating state.

The memory array contents are not required to be cleared during reset. The FIFO status must ensure that uninitialized memory is never treated as valid data.

## 6. Write Operation

A write operation is requested when `wr_en = 1`.

A write shall be accepted when:

- Reset is not asserted.
- `wr_en = 1`.
- `full = 0`.

On an accepted write:

1. `din` shall be stored at the current write address.
2. The write pointer shall advance by one.
3. The write address shall wrap around when the memory boundary is reached.

If the FIFO is full, a write request shall be rejected. Existing stored data shall not be overwritten.

## 7. Read Operation

A read operation is requested when `rd_en = 1`.

A read shall be accepted when:

- Reset is not asserted.
- `rd_en = 1`.
- `empty = 0`.

On an accepted read:

1. Data at the current read address shall be transferred to `dout`.
2. The read pointer shall advance by one.
3. The read address shall wrap around when the memory boundary is reached.

If the FIFO is empty, a read request shall be rejected. The output data shall retain its previous value unless reset or an accepted read updates it.

The baseline implementation uses a registered read output.

## 8. FIFO Full Condition

The FIFO is full when all 16 memory entries contain unread data.

The architecture uses an extra wrap bit in the read and write pointers to distinguish full and empty states.

For a depth of 16:

- Address bits: 4
- Pointer bits: 5

The full detection logic shall prevent writes when the FIFO has reached its maximum capacity.

The exact comparison logic shall be implemented consistently with the pointer architecture.

## 9. FIFO Empty Condition

The FIFO is empty when no unread data remains.

The empty condition is determined when the read pointer and write pointer represent the same circular-buffer position and wrap state.

When empty:

- Read requests shall not be accepted.
- The output shall not be updated by an invalid read.
- The empty status shall remain asserted until a valid write changes the FIFO state.

## 10. Simultaneous Read and Write

The controller shall support simultaneous read and write requests.

Operation acceptance is based on the FIFO status before the active clock edge.

### When neither full nor empty

If `rd_en = 1` and `wr_en = 1` while the FIFO is neither full nor empty:

- Read shall be accepted.
- Write shall be accepted.
- Both pointers shall advance.
- Occupancy shall remain unchanged.

### When full

If both requests are asserted while the FIFO is full:

- Read shall be accepted.
- Write shall be rejected based on the pre-edge full status.
- The existing data shall not be overwritten.
- Occupancy shall decrease by one.

### When empty

If both requests are asserted while the FIFO is empty:

- Write shall be accepted.
- Read shall be rejected based on the pre-edge empty status.
- No fall-through bypass shall occur.
- Occupancy shall increase by one.

This baseline uses pre-edge status evaluation. Same-cycle bypass and full-state write replacement are not included.

## 11. Pointer Operation

Each pointer is 5 bits wide.

The lower 4 bits select the memory address. The upper bit tracks wraparound.

On an accepted operation, the corresponding pointer increments by one.

The pointer shall wrap naturally according to its 5-bit width. The memory address shall wrap after the 16th entry.

Pointer updates shall only occur on accepted operations.

## 12. Error Prevention

The controller shall prevent the following invalid operations:

- Writing when full.
- Reading when empty.
- Overwriting unread data.
- Advancing pointers on rejected operations.
- Treating uninitialized memory as valid FIFO data.

Overflow and underflow event outputs are not part of the baseline interface. Their detection may be explored in future revisions.

## 13. Output Behavior

`dout` is a registered output.

- It updates when a read operation is accepted.
- It retains its previous value when no valid read is accepted.
- It resets to zero when synchronous reset is asserted.

The output is not required to be zero when the FIFO is empty except immediately after reset. Its value may represent the last successfully read data.

## 14. Functional Verification Requirements

The implementation shall be tested for:

| Test | Expected Result |
|---|---|
| Reset | Pointers reset, empty asserted |
| Write one entry | Entry stored, empty deasserted |
| Read one entry | Correct data retrieved |
| Write multiple entries | Data stored in order |
| Read multiple entries | Data retrieved in order |
| Fill all 16 entries | Full asserted |
| Read from full FIFO | Full deasserts after state update |
| Read empty FIFO | Operation rejected |
| Write full FIFO | Operation rejected |
| Pointer wraparound | Correct circular addressing |
| Simultaneous read/write | Correct acceptance behavior |
| Reset during operation | FIFO returns to initial state |

## 15. Acceptance Criteria

The baseline FIFO shall be considered functionally complete when:

- All accepted writes store the correct data.
- All accepted reads return the correct data in FIFO order.
- The full flag accurately indicates capacity reached.
- The empty flag accurately indicates no valid data.
- No accepted write occurs while full.
- No accepted read occurs while empty.
- Pointer wraparound operates correctly.
- Simultaneous read/write behavior matches the specification.
- All directed test cases pass in simulation.
- No unexplained simulation errors remain.

## 16. Scope Limitations

The baseline design does not include:

- Asynchronous FIFO operation.
- Multiple clock domains.
- AXI or APB bus interfaces.
- Programmable thresholds.
- Error event counters.
- ECC or parity.
- Memory initialization file support.
- Fall-through or combinational bypass.
- SRAM macro integration in the first RTL version.

These may be considered future enhancements.

---

**Document Status:** Initial Specification  
**Version:** 1.0  
**RTL Status:** Not yet implemented
</WritingBlock}

---

# Step 5 — `docs/design_decisions.md`

This is where we document why we chose a particular architecture or implementation. This is important in VLSI because two designs can function identically but have different timing, area, power, and physical-layout characteristics.

Create `docs/design_decisions.md`:

<WritingBlock id="64021" variant="document"># FIFO Memory Controller — Design Decisions

## 1. Purpose

This document records the major architectural and implementation decisions made during the development of the FIFO Memory Controller.

It provides the engineering rationale behind design choices and maintains a record of trade-offs considered throughout the RTL-to-GDS flow.

Decisions may be revised as implementation, verification, synthesis, or physical-design results become available.

## 2. Decision Log

### D-001 — Synchronous FIFO Architecture

**Decision:** Use a synchronous FIFO architecture.

**Rationale:**
- Simpler control logic.
- Single clock domain.
- Easier simulation and verification.
- Suitable for introductory ASIC implementation.
- Avoids asynchronous clock-domain crossing complexity.

**Alternatives considered:**
- Asynchronous FIFO with independent read and write clocks.

**Status:** Accepted

---

### D-002 — FIFO Depth of 16

**Decision:** Use 16 storage entries.

**Rationale:**
- Provides enough capacity to demonstrate circular-buffer operation.
- Uses a power-of-two depth.
- Allows simple address wraparound.
- Requires only a 4-bit memory address.

**Alternatives considered:**
- Depth 8.
- Depth 32.
- Parameterized depth.

**Status:** Accepted for baseline implementation

---

### D-003 — Data Width of 8 Bits

**Decision:** Use an 8-bit data width.

**Rationale:**
- Simple data representation.
- Suitable for initial simulation.
- Keeps the baseline design small.
- Allows easy waveform inspection.

**Alternatives considered:**
- 16-bit data width.
- 32-bit data width.
- Parameterized data width.

**Status:** Accepted for baseline implementation

---

### D-004 — Extra Pointer Wrap Bit

**Decision:** Use a 5-bit read pointer and 5-bit write pointer for a FIFO with 16 entries.

**Rationale:**

The lower 4 bits identify the memory address. The additional bit tracks wraparound and allows full and empty conditions to be distinguished.

This avoids ambiguity when the read and write address bits are equal.

**Status:** Accepted

---

### D-005 — Synchronous Active-High Reset

**Decision:** Use synchronous active-high reset.

**Rationale:**
- Reset behavior is easy to model.
- State changes occur at the active clock edge.
- Provides a straightforward baseline for simulation and synthesis.

**Alternatives considered:**
- Asynchronous active-low reset.
- Asynchronous active-high reset.

**Status:** Accepted for baseline implementation

---

### D-006 — Registered Read Output

**Decision:** Use a registered read output.

**Rationale:**
- Provides predictable read timing.
- Allows output data to remain stable between valid reads.
- Avoids requiring a combinational output path from the memory array.
- Provides a clear clocked behavior for verification.

**Trade-offs:**
- Introduces read latency associated with the registered read.
- May affect memory inference depending on the RTL coding style and target technology.

**Status:** Accepted for baseline implementation

---

### D-007 — Prevent Invalid Reads and Writes

**Decision:** Reject writes when full and reject reads when empty.

**Rationale:**
- Prevents unread data from being overwritten.
- Prevents invalid memory reads from being treated as valid data.
- Preserves FIFO ordering and data integrity.

**Status:** Accepted

---

### D-008 — Pre-Edge Status Evaluation for Simultaneous Operations

**Decision:** Determine read and write acceptance using the current FIFO status before the active clock edge.

**Rationale:**
- Simplifies the baseline control logic.
- Makes the behavior deterministic and easier to verify.
- Avoids introducing bypass logic in the first implementation.

**Trade-offs:**
- A write is not accepted in the same cycle that a full FIFO is being read.
- A read is not accepted in the same cycle that an empty FIFO is being written.

**Status:** Accepted for baseline implementation

---

### D-009 — Open-Source RTL-to-GDS Toolchain

**Decision:** Explore an open-source ASIC implementation flow.

**Planned tools:**
- Icarus Verilog for simulation.
- GTKWave for waveform analysis.
- Yosys for synthesis.
- OpenLane for physical implementation.
- SkyWater SKY130 as the target PDK.

**Rationale:**
- Enables reproducible learning.
- Provides access to the complete design flow.
- Supports documentation and analysis without relying exclusively on proprietary tools.

**Status:** Planned; tool compatibility to be confirmed

---

### D-010 — GitHub-Based Engineering Documentation

**Decision:** Use GitHub as the primary project repository and engineering log.

**Rationale:**
- Tracks source changes.
- Maintains project history.
- Provides a reproducible record of development.
- Enables collaboration and review.
- Allows documentation, reports, diagrams, and RTL to be maintained together.

**Status:** Accepted

## 3. Future Decision Topics

The following topics will be evaluated as the design progresses:

- Memory implementation: flip-flop array versus SRAM macro.
- Pointer-based full/empty logic versus occupancy-counter logic.
- Parameterization strategy.
- Synthesis optimization and area.
- Timing constraints and target clock frequency.
- Floorplan utilization.
- Pin placement strategy.
- Clock-tree architecture.
- Routing congestion.
- Power estimation and optimization.
- Physical verification methodology.

## 4. Change Management

When a design decision is revised:

1. Record the original decision ID.
2. Explain the reason for the change.
3. Describe the new approach.
4. Identify the impact on RTL, testbench, constraints, or physical design.
5. Update the relevant documentation.
6. Commit the revision to the repository.

Design decisions should be based on technical reasoning, simulation evidence, synthesis results, or physical-design observations rather than assumptions.

---

**Document Status:** Initial Decision Log  
**Version:** 1.0
