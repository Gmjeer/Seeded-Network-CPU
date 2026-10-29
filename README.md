# Seeded Network CPU

**A small seeded-program CPU prototype built from connected logic and state.**

This repository explores a computer architecture in which connected logic fabric performs the program's computation. A seed initializes the program; the fabric then executes arithmetic, memory updates, conditional branching, loops, and halt behavior.

## Current paper

[Read the v0.1 prototype paper](NETWORK_CPU_SEEDED_LOGIC_FABRIC_PROTOTYPE_PAPER_v0_1.md)

## What has been tested

- A minimal 8-bit CPU is described in SystemVerilog.
- The Icarus Verilog testbench loads a seven-word program and verifies its final state.
- The program completed in 15 instruction cycles with the expected accumulator, memory value, and halt signal.
- Yosys synthesis and nextpnr-ice40 place-and-route report 332 of 1,280 iCE40HX1K logic cells used and a 103.75 MHz maximum clock estimate against a 100 MHz target.

## Evidence boundary

The testbench passed, and the FPGA tools produced a timing estimate. **No physical FPGA board was programmed or measured.** The current design has a centralized program counter and decoder; it does not yet prove that autonomous network nodes can collectively control execution. USB boot, USB-backed memory, Linux, public-Internet execution, and competitive CPU performance are future work, not demonstrated capabilities.

## Next milestone

Build a multi-tile design with local state and transition rules, bounded links and queues, and no global instruction dispatcher. Compare it with this baseline using the same program and report correctness, latency, throughput, area, and measured hardware timing.

**Status:** Early research prototype. Not peer reviewed.
