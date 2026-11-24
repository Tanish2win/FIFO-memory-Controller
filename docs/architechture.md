# FIFO Memory Controller — Architecture

## 1. Overview

The FIFO Memory Controller is a synchronous digital memory system designed to temporarily store data and retrieve it in the order in which it was written.

The architecture consists of a memory array, read and write pointers, control logic, status flag generation, and reset logic.

The design uses a single clock domain to coordinate read and write operations.

## 2. Architectural Goals

The architecture is designed to:

- Maintain First-In, First-Out data ordering.
- Support independent read and write requests.
- Prevent writes when the FIFO is full.
- Prevent reads when the FIFO is empty.
- Efficiently manage circular memory addressing.
- Provide predictable synchronous operation.
- Support simulation, synthesis, and ASIC physical implementation.

## 3. Top-Level Architecture

The FIFO controller consists of the following major functional blocks:

1. FIFO Storage Array
2. Write Pointer Logic
3. Read Pointer Logic
4. Write Control Logic
5. Read Control Logic
6. Full/Empty Detection Logic
7. Reset Logic

## 4. Functional Block Descriptions

### 4.1 FIFO Storage Array

The storage array holds data entries temporarily.

Initial configuration:

- Depth: 16 entries
- Data width: 8 bits
- Total capacity: 128 bits

The memory is addressed using a 4-bit address field.

The memory array will initially be modeled using RTL constructs suitable for synthesis. The actual mapping into flip-flops, SRAM macros, or other storage structures depends on the synthesis tool and target implementation.

### 4.2 Write Pointer

The write pointer identifies the next memory location available for writing.

The pointer advances when a write request is accepted.

The address portion of the pointer selects one of the 16 memory locations. An additional pointer bit is used to distinguish circular-buffer wraparound states.

### 4.3 Read Pointer

The read pointer identifies the next memory location available for reading.

The pointer advances when a read request is accepted.

The read and write pointers operate independently, allowing the FIFO to manage incoming and outgoing data.

### 4.4 Write Control Logic

The write control block evaluates the write request and FIFO status.

A write is accepted only when:

- `wr_en` is asserted.
- The FIFO is not full.

When a write is accepted:

1. Input data is stored at the write address.
2. The write pointer advances.
3. FIFO occupancy is updated through the pointer relationship.

### 4.5 Read Control Logic

The read control block evaluates the read request and FIFO status.

A read is accepted only when:

- `rd_en` is asserted.
- The FIFO is not empty.

When a read is accepted:

1. Data is retrieved from the current read address.
2. The read pointer advances.
3. FIFO occupancy is updated through the pointer relationship.

### 4.6 Status Flag Logic

The controller generates two primary status signals.

**FULL**

Indicates that all FIFO entries are occupied and no additional write can be accepted.

**EMPTY**

Indicates that no valid data is available for reading.

These flags are generated using the read and write pointer states. The final logic will be defined and verified during RTL implementation.

### 4.7 Reset Logic

Reset initializes the controller to an empty state.

During reset:

- Write pointer is initialized.
- Read pointer is initialized.
- `empty` is asserted.
- `full` is deasserted.
- Control state is initialized.

The memory contents do not necessarily need to be reset because the empty flag prevents invalid stored values from being treated as valid data.

## 5. Data Flow

### Write Data Flow

1. External logic provides data through `din`.
2. External logic asserts `wr_en`.
3. Write control checks the `full` condition.
4. If the FIFO is not full, data is written into the memory array.
5. Write pointer advances to the next position.

### Read Data Flow

1. External logic asserts `rd_en`.
2. Read control checks the `empty` condition.
3. If the FIFO is not empty, data is retrieved from the current read address.
4. Read pointer advances to the next position.
5. The output presents the retrieved data according to the implemented read timing.

## 6. Circular Buffer Operation

The FIFO memory is organized as a circular buffer.

The read and write pointers move through the memory address space. When the address reaches the final memory location, it wraps around to the beginning.

The pointer contains an additional wrap bit beyond the address bits. This allows the architecture to distinguish pointer equality associated with empty and full states.

The detailed pointer comparison logic will be finalized in the RTL design specification.

## 7. Synchronous Operation

All controller state transitions are synchronized to the active clock edge.

The initial design uses:

- One clock domain.
- Synchronous reset.
- Clock-edge-triggered pointer updates.
- Clock-edge-triggered memory writes.
- Synchronous control-state updates.

The design does not include asynchronous clock-domain crossing logic.

## 8. Simultaneous Read and Write

The architecture will support simultaneous read and write requests when the FIFO state permits the operations.

The acceptance of each operation will be determined by the status conditions and controller logic.

The behavior when the FIFO is empty or full and both requests are asserted will be explicitly defined and tested during RTL development.

## 9. Implementation Considerations

The RTL architecture will be written to support synthesis and physical implementation.

Considerations include:

- Synthesizable coding style.
- Efficient pointer logic.
- Avoiding unnecessary combinational logic.
- Clear separation of storage and control.
- Predictable reset behavior.
- Verification of boundary conditions.
- Awareness of memory inference limitations.
- Evaluation of timing and area after synthesis.

## 10. Future Improvements

Potential extensions beyond the initial implementation include:

- Parameterized FIFO depth.
- Parameterized data width.
- Programmable almost-full and almost-empty thresholds.
- Occupancy counter.
- Overflow and underflow event indicators.
- Memory macro integration.
- Dual-clock asynchronous FIFO architecture.
- Improved throughput and timing optimization.

These extensions will be considered only after the baseline FIFO implementation has been verified.

---

**Document Status:** Initial Architecture  
**Implementation Status:** Not yet implemented
