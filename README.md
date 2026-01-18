RISC-V RV32I Pipelined CPU

This is a 5-stage pipelined implementation of the RISC-V RV32I base integer ISA written in SystemVerilog. The goal of the project was mainly to understand how the ISA maps onto a classic pipeline, and what needs to be done to make hazards, branches, and memory operations behave correctly cycle-by-cycle.

Pipeline Overview

The CPU follows the standard 5-stage design:

IF → ID → EX → MEM → WB


A quick summary of what happens in each stage:

IF: fetch instruction at PC, compute next PC (default PC+4)

ID: decode instruction, read register file, generate immediates + control signals

EX: ALU ops, branch comparisons, compute jump/branch targets

MEM: load/store access

WB: write result back to register file

Instruction Support

Implements the integer base ISA (RV32I):

Arithmetic / Logical: ADD, SUB, XOR, OR, AND, SLT, SLTU, SLL, SRL, SRA

Immediate versions: ADDI, ORI, ANDI, SLTI, SLLI, etc.

Memory: LW, SW

Control Flow: BEQ, BNE, BLT, BGE, JAL, JALR

Upper: LUI, AUIPC

This set was enough to run various small test programs and exercise control + data paths.

Hazard Handling

Pipeline hazards were the trickiest part of the project. The design handles:

Data hazards

Forwarding paths are implemented for common ALU-to-ALU dependencies:

EX → EX
MEM → EX


Load-use hazards still require a 1-cycle stall. Example:

LW   x1, 0(x2)
ADD  x3, x1, x4   // stall here

Control hazards

Branches and jumps are resolved in the EX stage. On a taken branch or jump, IF and ID are flushed and PC is redirected. No prediction is used (always assume not taken).

Penalty depends on the instruction ordering, usually 1–2 cycles.

Memory Model

The design uses separate instruction and data memories. These can be replaced with external modules or wrapped to simulate an AXI/AHB interface. For testing, simple behavioral memories were enough.

Testing

The CPU was tested using hand-written RISC-V assembly programs to verify:

basic arithmetic

loads/stores

branches/jumps

forwarding cases

load-use stall behavior

Waveforms were inspected to confirm that forwarding, stalls, and flushes happened at the right time.

Why Build This

The main motivation for the project was to get a concrete feel for:

how ISAs map into microarchitectural pipelines

why hazards exist and how to deal with them

how ALU/branch/memory stages interact

how control flow affects timing

It’s also a useful stepping stone toward features like caches, speculation, and eventually out-of-order execution.

Possible Extensions

If continued, the next steps would be:

static/dynamic branch prediction

instruction/data caches

CSR + privilege

multiplier/divider or the full M extension

MMU + SV32

bus interface instead of simple memories
