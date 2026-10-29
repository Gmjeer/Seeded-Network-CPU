# A Seeded Network CPU: A Small Logic-Fabric Prototype and Timing Study

**Author:** Shane A. Fortner  
**Draft:** v0.1 — 28 October 2026  
**Status:** Early prototype paper; not peer reviewed

## Abstract

This paper describes an early test of a computer whose processing is implemented by a connected network of logic and state elements. A USB seed is intended to initialize the program and startup state; the network fabric performs the subsequent computation. To test whether a small program can be represented and executed in such a fabric, a minimal 8-bit CPU was written in SystemVerilog, simulated, synthesized, and placed and routed for an iCE40HX1K FPGA target. Its test program initializes a memory location to three, repeatedly decrements it, branches until it reaches zero, and halts. The HDL simulation passed in 15 instruction cycles. FPGA tools reported a maximum clock frequency of 103.75 MHz for the selected target, with 332 of 1,280 logic cells and one embedded RAM block used. This is a design-tool timing estimate, not a measurement from a physical board. The prototype establishes that a small seeded program can be expressed as connected logic and state in an FPGA fabric. It does not yet demonstrate the proposed fully distributed controller, endpoint-free execution over the public Internet, a USB bootloader, Linux, or performance competitive with a modern general-purpose CPU.

## 1. Motivation and scope

The proposed network CPU treats the connected fabric as the processor. Nodes are intended to hold state, transform data, make local decisions, and forward results over links. The seed starts the computation; it is not an ongoing execution engine. This differs from using the Internet as a passive transport between conventional computers, where each computer's ordinary CPU still performs the work.

There are two related targets: a software model of programmable network-native nodes, and a closed-system chip whose connected logic fabric carries out computation. The work reported here addresses a narrower first chip-level question: can a seeded program with arithmetic, memory updates, a conditional loop, and halt behavior be represented in an FPGA's connected logic and state fabric?

The prototype is intentionally small. It is an executable engineering baseline, not a claim that a complete network-native CPU or Linux machine has been completed.

## 2. Prototype architecture

The SystemVerilog design contains a 4-bit program counter, an 8-bit accumulator, a 16-word program store, 16 bytes of data memory, instruction decode, arithmetic, branch selection, and halt state. The program store accepts words over a seed-write interface. The testbench drives that interface with a seven-word program.

The program uses six instruction types: load immediate, add immediate, load, store, jump-if-nonzero, and halt. Its flow is:

1. Load 3 into the accumulator and store it at data-memory address 0.
2. Load the value from address 0, add 255 (decrement modulo 256), and store the result.
3. Branch back while the accumulator is nonzero.
4. Halt when the value reaches zero.

The arithmetic and control are described as logic and state in RTL. FPGA synthesis maps those connections to the selected device's lookup tables, carry resources, flip-flops, embedded memory, and routing fabric.

### Seed boundary

The design exposes a parallel seed-write port. The testbench uses that port to load program words before setting `seed_done`. It does not include a USB host controller, USB mass-storage driver, or bootloader. A practical USB implementation would need a seed-loading subsystem that reads the USB image and presents verified words to this interface.

## 3. Verification and implementation method

The testbench was compiled and run with Icarus Verilog. It loaded the seed words, clocked the CPU, and checked the final program counter, accumulator, RAM value, and halt signal. The program completed in 15 instruction cycles and met all assertions.

The RTL was synthesized with Yosys 0.33 using the iCE40 mapping flow, then placed and routed with nextpnr-ice40 0.6. The selected target was an iCE40HX1K in the TQ144 package. A 100 MHz timing target was supplied. No package pin-constraint file or physical FPGA board was available; I/O pins were automatically placed by the tool.

## 4. Results

| Measure | Result |
|---|---:|
| HDL program result | PASS |
| Instruction cycles to halt | 15 |
| Final program counter | 7 |
| Final accumulator | 0 |
| Final data memory at address 0 | 0 |
| Tool-reported maximum clock | 103.75 MHz |
| Timing target | 100 MHz, PASS |
| iCE40 logic cells | 332 / 1,280 (25%) |
| Embedded RAM blocks | 1 / 16 (6%) |
| I/O cells | 45 / 112 (40%) |
| Yosys mapped cells | 349 total: 185 LUT4, 9 carry cells, 154 flip-flop variants, 1 RAM block |

The one-instruction-per-cycle implementation has a nominal rate near 104 million instructions per second at the reported clock estimate. This number describes only the prototype's very small custom instruction set. It is not a standard benchmark result and should not be compared directly with general-purpose CPU MIPS figures.

The timing result comes from synthesis and place-and-route models for a specific FPGA family and package. It does not include measured board behavior, power, thermal conditions, USB activity, or board-level timing. Automatic I/O placement also means the result is not a complete board design signoff.

## 5. What the result establishes

The test establishes that the RTL implementation can execute a program involving persistent state, arithmetic, a conditional branch, repetition, and halt. Synthesis and placement tools map that design into a specific FPGA logic fabric and report that the chosen 100 MHz target closes timing in their model.

This is useful feasibility evidence for the chip-level path: the program's execution is represented by connected hardware logic and state, rather than by the Python gate simulator used in the earlier model. The simulation and FPGA implementation both produce the same final program state.

## 6. What remains unproven

The present design has a centralized program counter and instruction decoder inside one small CPU block. Although these are themselves implemented by connected logic, this is not yet the proposed distributed state machine in which many autonomous network nodes jointly control execution without a central controller. It proves gate-fabric execution of a tiny program, not the full network-native architecture.

The prototype also does not establish:

- execution on ordinary Internet routers or on a public network without programmable compute nodes;
- the performance of long-distance network links as CPU interconnects;
- a real USB boot path or USB-backed RAM;
- an operating-system interface, a C compiler, or Linux boot;
- wide-data node operations, multiple concurrent programs, or distributed memory consistency;
- board-measured clock rate, power consumption, or advantage over conventional processors.

## 7. Next experiments

The next work should keep this tested processor as a baseline and build a second design that tests the defining distributed-control claim. That design should use multiple explicit compute/state tiles, local transition rules, bounded links and queues, and no global instruction dispatcher. A common program should be run on both designs, with correctness, latency, throughput, area, and timing compared.

The practical chip path is to add a real USB seed loader, constrain pins for a selected board, program an inexpensive FPGA, and measure clock, power, and I/O behavior. The architecture path is to define a wide-data node, distributed branch/loop semantics, and memory ownership before increasing the processor width or attempting an operating system. Linux would require a substantially more complete instruction set, memory model, interrupts, timers, and device interfaces.

## Conclusion

A small seeded program has been simulated and mapped into an FPGA logic fabric. The testbench passes, and place-and-route reports 103.75 MHz for the selected iCE40 target while meeting a 100 MHz constraint. This provides a concrete starting point for the network CPU's chip-level investigation. The next decisive test is not a larger clock claim; it is an FPGA implementation where multiple local state/compute nodes collectively perform control and execution without a central instruction dispatcher.