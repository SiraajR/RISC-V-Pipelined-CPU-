# RISC‑V RV32I — 5‑Stage Pipelined CPU (SystemVerilog)

A compact, educational implementation of the RISC‑V RV32I base integer ISA using a classic 5‑stage pipeline in SystemVerilog. The project was built to experiment with how instructions map to pipeline stages and to explore hazard mitigation techniques such as forwarding, stalls and flushes.

---

## Table of Contents

- [Overview](#overview)
- [Pipeline overview](#pipeline-overview)
- [Instruction support](#instruction-support)
- [Hazard handling](#hazard-handling)
  - [Data hazards](#data-hazards)
  - [Control hazards](#control-hazards)
- [Memory model](#memory-model)
- [Testing](#testing)
- [Why build this](#why-build-this)
- [Possible extensions](#possible-extensions)
- [Contributing](#contributing)
- [License](#license)

---

## Overview

This repository contains a 5‑stage pipelined CPU that implements the RISC‑V RV32I base integer instruction set. It was implemented primarily for learning and experimentation: to understand the interactions across pipeline stages and the practical handling of hazards.

The pipeline stages are:

IF → ID → EX → MEM → WB

A brief summary:

- IF: Instruction fetch, default next PC = PC + 4
- ID: Instruction decode, register-file reads, immediate generation, control generation
- EX: ALU operations, branch comparisons, compute branch/jump targets
- MEM: Data memory access for loads/stores
- WB: Write results back to the register file

---

## Pipeline overview

The design follows the canonical 5‑stage pipeline with separate registers between stages (IF/ID, ID/EX, EX/MEM, MEM/WB). The main control and datapath elements include:

- Register file (read/write in ID / WB)
- ALU in EX
- Forwarding unit to reduce data‑hazard stalls
- Hazard detection unit to insert stalls on load‑use cases
- Branch/jump resolution in EX (no prediction)

Example instruction flow:

```text
IF: fetch instruction at PC
ID: decode + read registers + generate immediates
EX: perform ALU op or compare for branch
MEM: perform load/store
WB: write back to register file
```

---

## Instruction support

Implemented RV32I instructions (sufficient for small test programs and datapath verification):

- Arithmetic / Logical: ADD, SUB, XOR, OR, AND, SLT, SLTU, SLL, SRL, SRA
- Immediate variants: ADDI, ORI, ANDI, SLTI, SLLI, SRLI, SRAI, etc.
- Memory: LW, SW
- Control flow: BEQ, BNE, BLT, BGE, JAL, JALR
- Upper immediates: LUI, AUIPC

This instruction set allows basic computational workloads, branching, and memory access to exercise forwarding, stalls, and flushes.

---

## Hazard handling

Hazards are handled via a combination of forwarding, stalling, and flushing.

### Data hazards

- Forwarding implemented for common ALU‑to‑ALU dependencies:
  - EX → EX
  - MEM → EX
- Load‑use hazards require a 1‑cycle stall:
  - Example:
    ```
    LW   x1, 0(x2)
    ADD  x3, x1, x4   // requires one stall cycle
    ```

### Control hazards

- Branches and jumps are resolved in the EX stage.
- On a taken branch or jump, IF and ID stages are flushed and the PC is redirected.
- No dynamic branch prediction is implemented (static not‑taken assumption).
- Typical penalty: 1–2 cycles depending on instruction ordering and pipeline timing.

---

## Memory model

- Separate instruction and data memories (Harvard style) in the reference design.
- For unit testing the project uses simple behavioral memory modules that are easy to replace.
- These memories can be wrapped or replaced with an AXI/AHB or other bus interface if integrating with a larger system.

---

## Testing

Testing was performed with hand‑written RISC‑V assembly test programs that exercise:

- Basic arithmetic and logical operations
- Load/store correctness
- Branch and jump behavior
- Forwarding and data‑hazard cases
- Load‑use stall behavior

Waveforms were inspected to verify that forwarding, stalls, and flushes occur at the correct cycles. You can test the design using your preferred SystemVerilog simulator (Icarus/Verilator/Questa/ModelSim) with simple testbenches and behavioral memories.

Suggested quick checks (examples — adapt to your toolchain and testbench layout):

```bash
# Example (pseudo-commands — update for your simulator)
# Compile
sv-simulator compile top.sv cpu.sv mem.sv tb.sv

# Run simulation and open waveform
sv-simulator run --wave=wave.vcd
gtkwave wave.vcd
```

---

## Why build this

This project provides hands‑on experience with:

- How ISAs map onto pipeline microarchitecture
- Why hazards exist and practical methods to mitigate them
- Interactions between ALU, branch logic, and memory stages
- How control flow affects timing and pipeline behavior

It is a good stepping stone to more advanced topics such as caches, speculation, and out‑of‑order execution.

---

## Possible extensions

If extended, the next steps could include:

- Static or dynamic branch prediction
- Instruction and data caches
- Control and Status Registers (CSR) and privilege modes
- Multiplier/divider unit or full M extension
- MMU + Sv32 virtual memory support
- Replace behavioral memories with a bus interface (AXI/AHB)

---

## Contributing

Contributions and improvements are welcome. Possible contributions:

- Add testbenches with automated test vectors
- Implement extensions listed above
- Improve documentation and usage examples for simulators

Feel free to open issues or PRs describing changes or proposals.

---

## License

Specify your chosen license here (e.g., MIT, Apache 2.0). If you don't have one yet, consider adding a LICENSE file.

---