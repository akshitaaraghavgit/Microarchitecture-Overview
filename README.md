# Real RISC-V Cores — Understanding Microarchitecture

> A detailed study of real RISC-V processor implementations, from tiny cores such as SERV and PicoRV32 to pipelined, out-of-order, generated, and complete SoC-based designs.

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [What is RISC-V?](#2-what-is-risc-v)
3. [ISA vs Microarchitecture](#3-isa-vs-microarchitecture)
4. [Why RISC-V is a Perfect Specimen for Studying Microarchitecture](#4-why-risc-v-is-a-perfect-specimen-for-studying-microarchitecture)
5. [Understanding a CPU Before Studying Real Cores](#5-understanding-a-cpu-before-studying-real-cores)

   * [5.1 Instruction Execution](#51-instruction-execution)
   * [5.2 Datapath](#52-datapath)
   * [5.3 Control Unit](#53-control-unit)
6. [The Spectrum of RISC-V Cores](#6-the-spectrum-of-risc-v-cores)
7. [Tiny Cores: SERV and PicoRV32](#7-tiny-cores-serv-and-picorv32)

   * [7.1 SERV](#71-serv)
   * [7.2 Why SERV is Bit-Serial](#72-why-serv-is-bit-serial)
   * [7.3 Advantages and Disadvantages of SERV](#73-advantages-and-disadvantages-of-serv)
   * [7.4 PicoRV32](#74-picorv32)
   * [7.5 SERV vs PicoRV32](#75-serv-vs-picorv32)
8. [In-Order Pipelined Cores](#8-in-order-pipelined-cores)

   * [8.1 What is Pipelining?](#81-what-is-pipelining)
   * [8.2 Why Do We Need Pipelines?](#82-why-do-we-need-pipelines)
   * [8.3 What Does In-Order Mean?](#83-what-does-in-order-mean)
   * [8.4 Pipeline Hazards](#84-pipeline-hazards)
   * [8.5 Rocket](#85-rocket)
   * [8.6 CVA6](#86-cva6)
9. [Out-of-Order Cores](#9-out-of-order-cores)

   * [9.1 Why Out-of-Order Execution?](#91-why-out-of-order-execution)
   * [9.2 Instruction-Level Parallelism](#92-instruction-level-parallelism)
   * [9.3 Register Renaming](#93-register-renaming)
   * [9.4 Issue Queue](#94-issue-queue)
   * [9.5 Reorder Buffer](#95-reorder-buffer)
   * [9.6 Speculative Execution](#96-speculative-execution)
   * [9.7 BOOM](#97-boom)
   * [9.8 XiangShan](#98-xiangshan)
10. [Generators](#10-generators)

    * [10.1 What is a Hardware Generator?](#101-what-is-a-hardware-generator)
    * [10.2 Chisel](#102-chisel)
    * [10.3 Rocket Chip](#103-rocket-chip)
    * [10.4 Chipyard](#104-chipyard)
11. [From CPU Core to SoC](#11-from-cpu-core-to-soc)
12. [Comparing the Real RISC-V Cores](#12-comparing-the-real-risc-v-cores)
13. [The Complete Microarchitecture Journey](#13-the-complete-microarchitecture-journey)
14. [Important Concepts](#14-important-concepts)
15. [Questions I Should Be Able to Answer](#15-questions-i-should-be-able-to-answer)
16. [Conclusion](#16-conclusion)
17. [References](#17-references)

---

# 1. Introduction

RISC-V is an **Instruction Set Architecture (ISA)**.

However, RISC-V does not describe one particular processor.

Instead, many completely different processors can implement the same RISC-V ISA.

For example:

```text
                         RISC-V ISA
                              |
            +-----------------+------------------+
            |                 |                  |
            ↓                 ↓                  ↓
          SERV             Rocket              BOOM
            |                 |                  |
       Tiny / serial      In-order          Out-of-order
            |              pipeline             |
            |                 |                  |
            +-----------------+------------------+
                              |
                       Same ISA visible
                        to the software
```

This makes RISC-V extremely useful for understanding **microarchitecture**.

We can keep the instruction set the same while changing the internal organization of the processor.

This repository studies real RISC-V cores in increasing order of complexity:

```text
Tiny Core
   ↓
Small Practical Core
   ↓
In-Order Pipelined Core
   ↓
Out-of-Order Core
   ↓
Hardware Generator
   ↓
Complete SoC
```

---

# 2. What is RISC-V?

RISC-V is an open Instruction Set Architecture based on the **Reduced Instruction Set Computer (RISC)** philosophy.

An ISA defines the interface between software and hardware.

For example:

```assembly
add x3, x1, x2
```

The RISC-V ISA tells us that the result of this instruction should be:

```text
x3 = x1 + x2
```

The ISA also defines:

* registers
* instructions
* instruction formats
* memory operations
* branch instructions
* exceptions
* privilege modes
* encoding rules

But the ISA does **not** tell us exactly how the hardware should perform the operation.

That is the job of microarchitecture.

---

# 3. ISA vs Microarchitecture

This distinction is the foundation of this entire topic.

## ISA

ISA answers:

> **What should the processor do?**

For example:

```text
ADD
LW
SW
BEQ
AND
OR
XOR
```

The ISA defines what these instructions mean.

---

## Microarchitecture

Microarchitecture answers:

> **How does the processor actually do it?**

For:

```assembly
add x3, x1, x2
```

the microarchitecture decides:

```text
How are x1 and x2 read?
        ↓
Which ALU performs the addition?
        ↓
When is the addition performed?
        ↓
Where is the result stored?
        ↓
When is x3 updated?
```

---

## Simple analogy

Think of the ISA as a **recipe**.

The recipe says:

> "Make a cake."

Microarchitecture is the actual kitchen arrangement:

```text
Oven
Mixer
Bowl
Ingredients
Chef
Timing
```

Two kitchens can follow the same recipe but use completely different equipment.

Similarly:

```text
Same ISA
   ↓
Different microarchitectures
```

---

# 4. Why RISC-V is a Perfect Specimen for Studying Microarchitecture

RISC-V is especially useful because the same ISA has been implemented in processors with very different designs.

## 4.1 Open ISA

RISC-V is an open standard.

This has encouraged:

* universities
* researchers
* startups
* companies
* hardware enthusiasts

to develop their own implementations.

Therefore, there are many publicly available RISC-V cores that we can actually study.

---

## 4.2 Same ISA, Different Hardware

Consider these processors:

```text
SERV
PicoRV32
Rocket
CVA6
BOOM
XiangShan
```

They all implement RISC-V, but they are very different internally.

```text
                 RISC-V
                    |
        +-----------+-----------+
        |           |           |
        ↓           ↓           ↓
      SERV        Rocket       BOOM
        |           |           |
    bit-serial   in-order    out-of-order
```

This lets us ask:

> If the ISA is the same, why does the hardware look so different?

The answer is **microarchitecture**.

---

## 4.3 RISC-V is Modular

RISC-V has a base ISA and extensions.

For example:

```text
RV32I
 |
 +-- M → Integer multiplication/division
 |
 +-- A → Atomic operations
 |
 +-- F → Single-precision floating point
 |
 +-- D → Double-precision floating point
 |
 +-- C → Compressed instructions
 |
 +-- V → Vector operations
```

A tiny embedded processor may implement only a small set of features.

A high-performance processor can implement many extensions.

---

## 4.4 Different Design Goals

Different processors optimize for different things.

One processor may prioritize:

```text
Minimum area
```

Another:

```text
Low power
```

Another:

```text
High frequency
```

Another:

```text
Maximum performance
```

Therefore:

```text
Microarchitecture = trade-offs
```

There is no single "best" processor design for every application.

---

# 5. Understanding a CPU Before Studying Real Cores

Before looking at SERV, Rocket or BOOM, we need a basic picture of what every processor is trying to do.

---

# 5.1 Instruction Execution

Suppose the processor receives:

```assembly
add x3, x1, x2
```

At a high level:

```text
1. Fetch instruction
        ↓
2. Decode instruction
        ↓
3. Read x1 and x2
        ↓
4. Perform addition
        ↓
5. Store result in x3
```

Conceptually:

```text
             Instruction Memory
                    |
                    ↓
                Instruction
                    |
                    ↓
                 Decoder
                    |
                    ↓
             Register File
              /          \
            x1            x2
              \          /
               \        /
                  ALU
                   |
                   ↓
                  x3
```

This is the basic datapath idea.

---

# 5.2 Datapath

The **datapath** is the hardware through which data moves and gets processed.

Typical components include:

```text
Register File
ALU
MUXes
Memory
Pipeline Registers
```

For example:

```text
Register 1 ----\
                \
                 → ALU → Result
                /
Register 2 ----/
```

---

# 5.3 Control Unit

The datapath performs operations.

The **control unit tells it what operation to perform**.

For example, when decoding:

```assembly
add x3, x1, x2
```

the control logic needs to generate signals such as:

```text
Read Register 1 = YES
Read Register 2 = YES
ALU Operation = ADD
Write Register = YES
Destination = x3
```

Therefore:

```text
CPU
 |
 +-- Datapath → moves/processes data
 |
 +-- Control  → tells datapath what to do
```

---

# 6. The Spectrum of RISC-V Cores

A useful way to study real cores is to move from simple to complex.

```text
                 Increasing complexity
                         →
                         
SERV
  ↓
PicoRV32
  ↓
Rocket
  ↓
CVA6
  ↓
BOOM
  ↓
XiangShan
```

The important thing is not memorizing this list.

The important thing is understanding **why each generation becomes more complex**.

---

# 7. Tiny Cores: SERV and PicoRV32

Tiny cores answer the question:

> **What is the minimum amount of hardware needed to build a useful RISC-V processor?**

---

# 7.1 SERV

SERV stands for:

> **SErial RISC-V**

SERV is a **bit-serial RISC-V processor**.

The word "serial" is extremely important.

A conventional processor may have a wide datapath capable of processing many bits together.

SERV uses a much smaller datapath and processes operations serially.

---

# 7.2 Why SERV is Bit-Serial

Suppose we want to add two 32-bit numbers.

A conventional 32-bit datapath can conceptually operate on the complete values:

```text
A = 101101010101...
B = 001011101010...
```

SERV instead processes the operation one bit at a time.

Conceptually:

```text
Cycle 1 → bit 0
Cycle 2 → bit 1
Cycle 3 → bit 2
...
Cycle 32 → bit 31
```

A carry from one bit can be passed to the next bit.

For example:

```text
Bit 0:
A0 + B0 + Carry
        ↓
     Result0
        ↓
     Carry

Bit 1:
A1 + B1 + Carry
        ↓
     Result1
```

And so on.

---

## Why would we do this?

Because hardware becomes much smaller.

Instead of building a large datapath:

```text
Large ALU
Large hardware
More area
```

we can reuse a very small datapath.

The trade-off is:

```text
Smaller hardware
      ↓
Less parallelism
      ↓
More cycles
      ↓
Lower performance
```

---

# 7.3 Advantages and Disadvantages of SERV

### Advantages

* Extremely small hardware
* Useful for studying minimal CPU design
* Suitable for resource-constrained applications
* Demonstrates how an ISA can be implemented with very little hardware

### Disadvantages

* Very low performance
* Many cycles are required
* Very little parallelism

The main lesson from SERV is:

> **A CPU does not have to be large or fast to implement an ISA.**

---

# 7.4 PicoRV32

PicoRV32 is another small RISC-V processor core.

It is designed as a practical, configurable, size-optimized processor for FPGA and ASIC use.

Compared with SERV:

```text
SERV
↓
Extremely small and bit-serial

PicoRV32
↓
Small practical RISC-V CPU
```

PicoRV32 supports several RISC-V configurations and provides interfaces for connecting the processor to a larger system.

---

# 7.5 SERV vs PicoRV32

| Feature     | SERV             | PicoRV32                  |
| ----------- | ---------------- | ------------------------- |
| Main idea   | Bit-serial CPU   | Small practical CPU       |
| Datapath    | Extremely small  | More conventional         |
| Performance | Very low         | Higher                    |
| Area        | Extremely small  | Small                     |
| Complexity  | Very low         | Low                       |
| Main lesson | Minimum hardware | Practical small processor |

The progression is:

```text
SERV
 ↓
"What is the smallest implementation?"

PicoRV32
 ↓
"How can I make a small but practical implementation?"
```

---

# 8. In-Order Pipelined Cores

Now we move from tiny processors to more sophisticated processors.

The main new concept is:

> **Pipelining**

---

# 8.1 What is Pipelining?

Imagine washing clothes.

Without a pipeline:

```text
Wash → Dry → Fold
```

You wait until one load finishes before starting the next.

With a pipeline:

```text
Load 1: Wash → Dry → Fold
Load 2:        Wash → Dry → Fold
Load 3:               Wash → Dry → Fold
```

Multiple loads are being processed at different stages simultaneously.

A CPU works similarly.

---

## CPU Pipeline

A simplified 5-stage pipeline:

```text
IF → ID → EX → MEM → WB
```

Where:

```text
IF  = Instruction Fetch
ID  = Instruction Decode
EX  = Execute
MEM = Memory Access
WB  = Write Back
```

---

# 8.2 Why Do We Need Pipelines?

Suppose each stage takes one clock cycle.

Without pipelining:

```text
Instruction 1:
IF → ID → EX → MEM → WB

Then:

Instruction 2:
IF → ID → EX → MEM → WB
```

With pipelining:

```text
Cycle 1:
I1 → IF

Cycle 2:
I1 → ID
I2 → IF

Cycle 3:
I1 → EX
I2 → ID
I3 → IF

Cycle 4:
I1 → MEM
I2 → EX
I3 → ID
I4 → IF
```

Now multiple instructions are being processed simultaneously.

This improves **throughput**.

---

# 8.3 What Does In-Order Mean?

In an in-order processor, instructions are handled according to their program order.

Suppose:

```assembly
1. add x3, x1, x2
2. sub x5, x4, x6
3. and x7, x8, x9
```

The processor maintains the ordering:

```text
1 → 2 → 3
```

The instructions move through the pipeline in program order.

This makes the processor easier to design than an out-of-order processor.

---

# 8.4 Pipeline Hazards

Pipelining introduces problems called **hazards**.

There are three major types.

---

## Data Hazard

Example:

```assembly
add x3, x1, x2
sub x4, x3, x5
```

The second instruction needs the result produced by the first.

```text
ADD
 ↓
produces x3

SUB
 ↓
needs x3
```

If x3 is not ready yet, the processor has a problem.

Possible solutions:

```text
Forwarding
Stalling
```

---

## Control Hazard

Consider:

```assembly
beq x1, x2, target
```

The processor may not immediately know which instruction comes next.

This is a branch problem.

Processors use:

```text
Branch prediction
Branch target prediction
Pipeline flushing
```

to reduce the performance penalty.

---

## Structural Hazard

This happens when multiple instructions need the same hardware resource.

Example:

```text
Instruction A → needs memory
Instruction B → needs same memory resource
```

The processor must resolve the conflict.

---

# 8.5 Rocket

Rocket is a classic RISC-V processor core and generator.

It is an **in-order scalar processor** with a pipelined design.

A simplified pipeline can be represented as:

```text
Fetch
  ↓
Decode
  ↓
Execute
  ↓
Memory
  ↓
Writeback
```

Rocket is much more sophisticated than tiny cores.

It can include:

* caches
* branch prediction
* virtual memory
* privilege support
* configurable ISA extensions
* memory interfaces

---

## Why Rocket is important

Rocket is a useful example of a processor that tries to balance:

```text
Performance
+
Hardware complexity
+
Configurability
```

It shows that we can build a relatively capable processor without immediately moving to out-of-order execution.

---

# 8.6 CVA6

CVA6 is another important RISC-V processor.

CVA6 is:

```text
6-stage
Single-issue
In-order
RISC-V
```

A simplified view is:

```text
Fetch
  ↓
Decode
  ↓
Issue
  ↓
Execute
  ↓
Memory
  ↓
Commit
```

CVA6 also contains advanced features such as:

* caches
* branch prediction
* virtual memory
* TLBs
* privilege support

The important point is:

> **CVA6 is still in-order even though it is considerably more sophisticated than tiny RISC-V cores.**

---

# 9. Out-of-Order Cores

Now we move to a much more advanced microarchitecture.

The key idea is:

> **The processor does not always execute instructions in program order.**

---

# 9.1 Why Out-of-Order Execution?

Consider:

```assembly
1. load x1, 0(x2)
2. add  x3, x1, x4
3. add  x5, x6, x7
```

Instruction 2 depends on instruction 1.

Suppose instruction 1 takes a long time because memory is slow.

Then:

```text
Instruction 1 → WAITING
Instruction 2 → WAITING
Instruction 3 → READY
```

An in-order processor may be restricted by the earlier instructions.

An out-of-order processor can say:

> "Instruction 3 does not depend on 1 or 2, so I can execute it now."

Therefore:

```text
Program order:

1 → 2 → 3

Execution order:

1 → 3 → 2
```

This is the central idea behind out-of-order execution.

---

# 9.2 Instruction-Level Parallelism

Modern programs often contain instructions that are independent.

For example:

```assembly
add x3, x1, x2
sub x6, x4, x5
and x9, x7, x8
```

These instructions do not depend on each other.

Therefore, a sufficiently advanced processor can execute them in parallel.

This is called:

> **Instruction-Level Parallelism (ILP)**

The goal of an out-of-order processor is to discover and exploit this parallelism.

---

# 9.3 Register Renaming

Consider:

```assembly
add x1, x2, x3
sub x1, x4, x5
```

Both instructions write to `x1`.

Architecturally this is allowed.

Internally, however, the processor can assign different physical registers:

```text
Instruction 1:
x1 → Physical Register P7

Instruction 2:
x1 → Physical Register P12
```

Conceptually:

```text
Architectural Register
        x1
         |
    +----+----+
    |         |
    ↓         ↓
   P7        P12
```

This technique is called:

> **Register Renaming**

It helps remove false dependencies and increases available parallelism.

---

# 9.4 Issue Queue

The issue queue contains instructions waiting to execute.

For example:

```text
Instruction       Operands Ready?

ADD               YES
SUB               NO
MUL               YES
LOAD              NO
```

The processor can select:

```text
ADD → Execute
MUL → Execute
```

while the others wait.

Therefore:

```text
Instruction Queue
       ↓
Find ready instructions
       ↓
Send them to execution units
```

This is one of the major differences between simple in-order and out-of-order processors.

---

# 9.5 Reorder Buffer

Now we have a problem.

Suppose:

```text
Program order:

I1 → I2 → I3
```

but:

```text
Execution order:

I1 → I3 → I2
```

We still want the processor's final architectural state to behave as if:

```text
I1 → I2 → I3
```

The **Reorder Buffer (ROB)** helps maintain this ordering.

Conceptually:

```text
Execution:
    I1 ────────┐
    I3 ────────┼──→ Results
    I2 ────────┘

Commit:
    I1 → I2 → I3
```

This gives us:

```text
Out-of-order execution
+
In-order architectural commitment
```

This is one of the most important ideas in out-of-order microarchitecture.

---

# 9.6 Speculative Execution

Consider a branch:

```assembly
beq x1, x2, target
```

The processor does not immediately know which path will be taken.

Instead, it predicts:

```text
Branch taken
```

and starts executing instructions from that path.

If the prediction is correct:

```text
Continue
```

If it is wrong:

```text
Discard incorrect work
+
Fetch correct instructions
```

This is called:

> **Speculative Execution**

Branch prediction and speculation allow the pipeline to stay busy.

---

# 9.7 BOOM

BOOM stands for:

> **Berkeley Out-of-Order Machine**

BOOM is an open-source RISC-V out-of-order processor core.

It is designed to demonstrate and research high-performance processor microarchitecture.

A simplified BOOM-style pipeline can be viewed as:

```text
Fetch
  ↓
Decode
  ↓
Rename
  ↓
Dispatch
  ↓
Issue
  ↓
Execute
  ↓
Writeback
  ↓
Commit
```

The important difference from a simple in-order processor is that the processor now has machinery for:

```text
Register renaming
Issue queues
Multiple execution units
Speculation
Reorder buffer
Out-of-order execution
```

---

# 9.8 XiangShan

XiangShan is an open-source, high-performance RISC-V processor project.

It represents another step toward sophisticated modern processor design.

Instead of focusing on minimum hardware, XiangShan focuses on:

```text
High performance
+
Advanced microarchitecture
+
Research
```

Its design includes sophisticated mechanisms for:

* instruction fetching
* branch prediction
* out-of-order execution
* instruction scheduling
* memory operations
* caches
* register renaming
* retirement

The important lesson is:

> **RISC-V is not limited to simple embedded processors. It can also be used as the ISA for highly sophisticated high-performance CPUs.**

---

# 10. Generators

So far we have been talking about processors as hardware designs.

Modern hardware development introduces another idea:

> **Hardware can be generated using software.**

---

# 10.1 What is a Hardware Generator?

Imagine writing one CPU manually.

You might create:

```text
CPU.v
ALU.v
RegisterFile.v
Cache.v
Control.v
...
```

That creates one particular design.

A generator instead allows us to describe:

```text
"Build me a CPU with these parameters."
```

For example:

```text
Number of cores = 4
L1 cache = 32 KB
L2 cache = 512 KB
RV64 = enabled
Floating point = enabled
```

The generator can produce the corresponding hardware.

Conceptually:

```text
Parameters
    ↓
Hardware Generator
    ↓
Generated RTL
    ↓
Verilog / SystemVerilog
    ↓
FPGA / ASIC
```

---

# 10.2 Why Are Generators Useful?

Imagine designing 10 processors manually.

That would be extremely repetitive.

A generator allows us to change parameters instead.

```text
                  Generator
                     |
       +-------------+-------------+
       |             |             |
       ↓             ↓             ↓
    Small CPU     Medium CPU    Large CPU
```

This provides:

* configurability
* reuse
* faster experimentation
* easier research
* parameter exploration

---

# 10.3 Chisel

Chisel is a hardware construction language embedded in Scala.

Instead of writing only traditional RTL, designers can write hardware-generating programs.

Conceptually:

```text
Scala + Chisel
      ↓
Hardware description
      ↓
Generated RTL
      ↓
Verilog
```

This makes it easier to create parameterized hardware.

---

# 10.4 Rocket Chip

Rocket Chip is a RISC-V hardware generator ecosystem.

Instead of thinking:

```text
Rocket = one fixed processor
```

it is better to think:

```text
Rocket
+
Generator
+
Configurable system
```

The generator can create different Rocket-based systems according to configuration.

---

# 10.5 Chipyard

Chipyard provides a framework for designing complete RISC-V systems.

It can bring together:

```text
CPU cores
+
Caches
+
Memory systems
+
Interconnect
+
Peripherals
+
Accelerators
+
Simulation
```

For example:

```text
                    Chipyard
                       |
        +--------------+--------------+
        |              |              |
      Rocket          BOOM           CVA6
        |              |              |
        +--------------+--------------+
                       |
                  Interconnect
                       |
             +---------+---------+
             |                   |
           Memory            Peripherals
```

This is where processor microarchitecture connects to **actual SoC design**.

---

# 11. From CPU Core to SoC

A CPU core is only one part of a complete computer system.

An **SoC (System-on-Chip)** combines the CPU with other hardware.

A simplified SoC:

```text
+--------------------------------------------------+
|                      SoC                         |
|                                                  |
|   +---------+       +-----------------------+    |
|   |   CPU   | <---> | Cache / Memory System |    |
|   +---------+       +-----------------------+    |
|        |                                         |
|        ↓                                         |
|   +------------------------------------------+   |
|   |             Interconnect / Bus           |   |
|   +------------------------------------------+   |
|       |             |             |             |
|      UART          GPIO          SPI           |
|                                                  |
+--------------------------------------------------+
```

---

## CPU

Executes instructions.

Examples:

```text
Rocket
BOOM
CVA6
PicoRV32
```

---

## Memory

Stores:

```text
Instructions
Data
Program state
```

---

## Cache

The CPU needs data quickly.

Instead of always going to slow main memory, frequently used data can be stored in a cache.

```text
CPU
 ↓
L1 Cache
 ↓
L2 Cache
 ↓
Main Memory
```

---

## Interconnect

The interconnect allows components to communicate.

```text
CPU
 |
 +---- Memory
 |
 +---- UART
 |
 +---- GPIO
 |
 +---- SPI
```

---

## Peripherals

Examples:

```text
UART
GPIO
SPI
I2C
Timers
Interrupt Controllers
```

These allow the processor to interact with the external world.

---

# 12. Comparing the Real RISC-V Cores

| Core          | Category     | Main idea            | Complexity  | Main goal                 |
| ------------- | ------------ | -------------------- | ----------- | ------------------------- |
| **SERV**      | Tiny         | Bit-serial           | Very Low    | Minimum area              |
| **PicoRV32**  | Tiny         | Small practical CPU  | Low         | Small implementation      |
| **Rocket**    | In-order     | Pipelined scalar CPU | Medium      | Balanced performance      |
| **CVA6**      | In-order     | 6-stage single-issue | Medium/High | Application-class CPU     |
| **BOOM**      | Out-of-order | High-performance OOO | High        | Performance               |
| **XiangShan** | Out-of-order | Advanced OOO CPU     | Very High   | High-performance research |

---

# 13. The Complete Microarchitecture Journey

The entire topic can now be understood as one progression.

## Step 1 — Implement the ISA

First, we need hardware capable of executing RISC-V instructions.

```text
ADD
SUB
LW
SW
BEQ
AND
OR
XOR
...
```

Example:

```text
SERV
```

---

## Step 2 — Make the CPU practical

We want:

```text
Small area
+
Reasonable performance
```

Example:

```text
PicoRV32
```

---

## Step 3 — Pipeline the processor

Instead of completing one instruction before starting another:

```text
I1 → complete
I2 → complete
I3 → complete
```

we overlap them:

```text
I1: IF → ID → EX → MEM → WB
I2:     IF → ID → EX → MEM → WB
I3:         IF → ID → EX → MEM → WB
```

Examples:

```text
Rocket
CVA6
```

---

## Step 4 — Exploit Instruction-Level Parallelism

Now we ask:

> What if some instructions are waiting while other instructions are ready?

We allow independent instructions to execute.

```text
Program order:
I1 → I2 → I3 → I4

Possible execution:
I1 → I3 → I4 → I2
```

Examples:

```text
BOOM
XiangShan
```

---

## Step 5 — Generate Hardware

Instead of manually designing one processor:

```text
Generator
    ↓
Different configurations
```

Examples:

```text
Rocket Chip
Chipyard
```

---

## Step 6 — Build a Complete SoC

Finally:

```text
CPU
+
Cache
+
Memory
+
Interconnect
+
Peripherals
+
Accelerators
=
SoC
```

---

# 14. Important Concepts

## ISA

Defines:

> **What the processor does.**

---

## Microarchitecture

Defines:

> **How the processor does it.**

---

## Datapath

Hardware through which data moves and is processed.

---

## Control Unit

Generates control signals that tell the datapath what to do.

---

## ALU

Arithmetic Logic Unit.

Performs operations such as:

```text
ADD
SUB
AND
OR
XOR
Comparison
```

---

## Register File

Contains the processor's architectural registers.

For RV32:

```text
32 registers
×
32 bits each
```

---

## Pipeline

Divides instruction execution into stages so multiple instructions can be processed simultaneously.

---

## In-Order

Instructions are processed according to program order.

Examples:

```text
Rocket
CVA6
```

---

## Out-of-Order

Instructions can execute when ready even if earlier instructions are waiting.

Examples:

```text
BOOM
XiangShan
```

---

## Instruction-Level Parallelism

The ability to execute multiple independent instructions at the same time.

---

## Register Renaming

Maps architectural registers to physical registers to reduce false dependencies.

---

## Issue Queue

Stores instructions waiting for their operands/resources to become ready.

---

## Reorder Buffer

Tracks instructions so that out-of-order execution can still produce correct architectural results.

---

## Speculative Execution

Executing instructions based on predicted future control flow.

---

## Branch Prediction

Predicting whether a branch will be taken and/or where execution will continue.

---

## Cache

Small, fast memory located close to the processor that stores frequently accessed data/instructions.

---

## Generator

Software that generates hardware according to parameters.

---

## SoC

System-on-Chip.

A complete system containing a processor and supporting components such as memory, interconnects and peripherals.

---

# 15. Questions I Should Be Able to Answer

## RISC-V

1. What is RISC-V?
2. What is an ISA?
3. Why is RISC-V called an open ISA?
4. Why is RISC-V useful for research?
5. Why can multiple completely different processors implement RISC-V?

---

## ISA vs Microarchitecture

6. What is the difference between ISA and microarchitecture?
7. If two processors use the same ISA, why can their internal hardware be different?
8. Is RISC-V itself a processor?
9. What does the ISA specify?
10. What does the microarchitecture specify?

---

## SERV

11. What is SERV?
12. What does "bit-serial" mean?
13. Why does bit-serial processing reduce hardware?
14. Why is SERV slower?
15. What trade-off does SERV demonstrate?

---

## PicoRV32

16. What is PicoRV32?
17. Why is PicoRV32 useful?
18. How is PicoRV32 different from SERV?

---

## Pipeline

19. What is a pipeline?
20. Why do processors use pipelines?
21. What are the typical pipeline stages?
22. What is instruction throughput?
23. Does pipelining necessarily reduce the latency of one instruction?
24. What is an in-order processor?

---

## Hazards

25. What is a data hazard?
26. What is a control hazard?
27. What is a structural hazard?
28. What is forwarding?
29. Why is branch prediction required?

---

## Rocket and CVA6

30. What is Rocket?
31. Why is Rocket called an in-order processor?
32. What is CVA6?
33. Is CVA6 in-order or out-of-order?
34. Why is CVA6 more sophisticated than a tiny core?

---

## Out-of-Order

35. Why do we need out-of-order execution?
36. What is Instruction-Level Parallelism?
37. What is register renaming?
38. What is an issue queue?
39. What is a reorder buffer?
40. Why does an out-of-order processor execute instructions out of order but commit them in order?
41. What is speculative execution?
42. Why is branch prediction important?

---

## BOOM and XiangShan

43. What is BOOM?
44. What makes BOOM different from Rocket?
45. What is XiangShan?
46. Why are BOOM and XiangShan considered more sophisticated processors?

---

## Generators

47. What is a hardware generator?
48. Why are hardware generators useful?
49. What is Chisel?
50. What is Rocket Chip?
51. What is Chipyard?

---

## SoC

52. What is an SoC?
53. What is the difference between a CPU core and an SoC?
54. Why does a CPU need memory?
55. Why do we need caches?
56. What does an interconnect do?
57. What are peripherals?
58. How does a CPU communicate with peripherals?

---

# 16. Conclusion

The most important idea from this study is:

> **RISC-V defines the contract, while microarchitecture defines the implementation of that contract.**

The same RISC-V ISA can be implemented using dramatically different hardware.

```text
                    RISC-V ISA
                        |
                        ↓
              "What should happen?"
                        |
                        ↓
               Microarchitecture
                        |
       +----------------+----------------+
       |                |                |
       ↓                ↓                ↓
     Tiny           In-Order        Out-of-Order
       |                |                |
     SERV         Rocket / CVA6     BOOM / XiangShan
       |                |                |
       +----------------+----------------+
                        |
                        ↓
                   Generators
                        |
                        ↓
                Rocket Chip / Chipyard
                        |
                        ↓
                       SoC
```

The progression can therefore be summarized as:

```text
SERV
 ↓
Minimum hardware
 ↓
PicoRV32
 ↓
Small practical processor
 ↓
Rocket / CVA6
 ↓
Pipelined in-order processor
 ↓
BOOM / XiangShan
 ↓
Out-of-order high-performance processor
 ↓
Rocket Chip / Chipyard
 ↓
Configurable hardware systems
 ↓
SoC
```

The fundamental question changes at every level:

```text
SERV
"What is the smallest hardware I can build?"

        ↓

PicoRV32
"How can I build a small practical CPU?"

        ↓

Rocket / CVA6
"How can I improve throughput using pipelining?"

        ↓

BOOM / XiangShan
"How can I exploit as much instruction-level
parallelism as possible?"

        ↓

Generators
"How can I generate different hardware configurations?"

        ↓

SoC
"How can I turn the CPU into a complete system?"
```

That progression is the core idea behind this study of **real RISC-V microarchitecture**.

---

