# FIFO Controller — Block Diagram

## 1. Top-Level Architecture

The diagram illustrates the primary functional blocks of the synchronous FIFO memory controller.

```mermaid
flowchart LR
    CLK[Clock and Reset] --> CTRL[Control Logic]

    DIN[Data Input - 8 bit] --> WM[Write Control]
    WR[Write Enable] --> WM
    WM --> WP[Write Pointer - 5 bit]
    WP --> MEM[FIFO Memory Array - 16 x 8]

    MEM --> RM[Read Control]
    RD[Read Enable] --> RM
    RM --> RP[Read Pointer - 5 bit]
    RP --> MEM

    RM --> DOUT[Data Output - 8 bit]

    WP --> STATUS[Full / Empty Detection]
    RP --> STATUS

    STATUS --> FULL[Full Flag]
    STATUS --> EMPTY[Empty Flag]

    CTRL --> WM
    CTRL --> RM

    CLK --> WP
    CLK --> RP
    CLK --> MEM
    CLK --> CTRL
```

## 2. Functional Summary

- The write control receives input data and write requests.
- The write pointer identifies the memory location used for writing.
- The memory array stores the data.
- The read pointer identifies the next data location to be read.
- The read control manages valid read requests.
- The status logic generates full and empty conditions.
- Clock and reset synchronize the controller.

## 3. Design Notes

This diagram represents the planned logical architecture. The detailed RTL may combine or separate some logic blocks based on implementation requirements.

**Status:** Initial architecture diagram  
**Revision:** 1.0
