# Real RISC-V Cores — Understanding Microarchitecture

> A study of real RISC-V processor implementations, from tiny cores to pipelined, out-of-order, generated, and complete SoC-based designs.

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [RISC-V: ISA vs Microarchitecture](#2-risc-v-isa-vs-microarchitecture)
3. [Why RISC-V is a Perfect Specimen](#3-why-risc-v-is-a-perfect-specimen)
4. [Basic CPU Microarchitecture](#4-basic-cpu-microarchitecture)
5. [Tiny Cores: SERV and PicoRV32](#5-tiny-cores-serv-and-picorv32)

   * [SERV](#51-serv)
   * [PicoRV32](#52-picorv32)
   * [SERV vs PicoRV32](#53-serv-vs-picorv32)
6. [In-Order Pipelines](#6-in-order-pipelines)

   * [Pipelining](#61-pipelining)
   * [In-Order Execution](#62-in-order-execution)
   * [Pipeline Hazards](#63-pipeline-hazards)
   * [Rocket](#64-rocket)
   * [CVA6](#65-cva6)
7. [Out-of-Order Cores](#7-out-of-order-cores)

   * [Why Out-of-Order?](#71-why-out-of-order)
   * [Register Renaming](#72-register-renaming)
   * [Issue Queue](#73-issue-queue)
   * [Reorder Buffer](#74-reorder-buffer)
   * [Speculative Execution](#75-speculative-execution)
   * [BOOM](#76-boom)
   * [XiangShan](#77-xiangshan)
8. [Generators](#8-generators)

   * [Hardware Generators](#81-hardware-generators)
   * [Chisel](#82-chisel)
   * [Rocket Chip](#83-rocket-chip)
   * [Chipyard](#84-chipyard)
9. [From Core to SoC](#9-from-core-to-soc)
10. [Comparing the Real Cores](#10-comparing-the-real-cores)
11. [The Complete Microarchitecture Journey](#11-the-complete-microarchitecture-journey)
12. [Key Concepts](#12-key-concepts)
13. [Questions to Test My Understanding](#13-questions-to-test-my-understanding)
14. [Conclusion](#14-conclusion)
15. [References](#15-references)

---

# 1. Introduction

RISC-V is an **Instruction Set Architecture (ISA)** rather than one particular processor.

An ISA defines what instructions a processor understands and what those instructions are supposed to do. It does not prescribe the exact internal hardware used to execute them.

This allows many completely different processors to implement the same RISC-V ISA.

```text
                         RISC-V ISA
                              |
             +----------------+----------------+
             |                |                |
             ↓                ↓                ↓
           SERV             Rocket            BOOM
             |                |                |
        Bit-serial        In-order          Out-of-order
             |             pipeline             |
             +----------------+----------------+
                              |
                       Same ISA visible
                        to the software
```

This makes RISC-V an excellent platform for studying **microarchitecture**.

This repository follows the progression:

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

The objective is not simply to memorize processor names, but to understand **why their architectures become more complex and what problem each new technique solves**.

---

# 2. RISC-V: ISA vs Microarchitecture

## What is an ISA?

An **Instruction Set Architecture** is the interface between software and hardware.

For example:

```assembly
add x3, x1, x2
```

The RISC-V ISA defines that this instruction performs:

```text
x3 = x1 + x2
```

The ISA defines things such as:

* instructions
* registers
* instruction formats
* memory operations
* branches
* exceptions
* privilege modes
* architectural behavior

It answers:

> **What should the processor do?**

---

## What is Microarchitecture?

Microarchitecture describes the internal hardware organization used to implement the ISA.

For:

```assembly
add x3, x1, x2
```

the microarchitecture determines:

```text
How are x1 and x2 read?
        ↓
Which hardware performs the addition?
        ↓
When is the addition performed?
        ↓
Where is the result stored?
        ↓
When does x3 become available?
```

It answers:

> **How does the processor do it?**

---

## Simple Analogy

Think of the ISA as a **recipe**.

The recipe tells you what food should be produced.

Microarchitecture is the actual kitchen:

```text
Recipe → What must be produced

Kitchen → How it is produced
```

Two kitchens can follow the same recipe while having completely different equipment.

Similarly:

```text
Same RISC-V ISA
       ↓
Different microarchitectures
```

---

# 3. Why RISC-V is a Perfect Specimen

RISC-V is especially useful for studying microarchitecture because the ISA is open, modular, and not tied to one particular implementation style.

## 3.1 Open ISA

RISC-V is an open standard.

This has encouraged researchers, universities, companies, and hardware developers to create their own implementations.

As a result, many real RISC-V cores are available for study.

---

## 3.2 Same ISA, Completely Different Hardware

Consider:

```text
SERV
PicoRV32
Rocket
CVA6
BOOM
XiangShan
```

All are associated with RISC-V, but their internal designs are very different.

```text
RISC-V
   |
   +-- SERV
   |     Bit-serial
   |
   +-- PicoRV32
   |     Small processor
   |
   +-- Rocket
   |     In-order pipeline
   |
   +-- CVA6
   |     In-order application-class core
   |
   +-- BOOM
   |     Out-of-order
   |
   +-- XiangShan
         High-performance out-of-order
```

This gives us a controlled way to study microarchitecture:

> **Keep the ISA relatively constant and change the implementation.**

---

## 3.3 Modular ISA

RISC-V has a base ISA and optional extensions.

For example:

```text
RV32I
 |
 +-- M → Multiply / Divide
 +-- A → Atomic operations
 +-- F → Single-precision floating point
 +-- D → Double-precision floating point
 +-- C → Compressed instructions
 +-- V → Vector operations
```

A tiny processor can implement a limited set of features, while a high-performance processor can implement many extensions.

---

## 3.4 Different Design Goals

Processor design is about trade-offs.

A processor may prioritize:

```text
Small area
Low power
Low cost
High performance
High frequency
Configurability
```

Therefore, there is no single "best" microarchitecture.

A tiny embedded processor and a high-performance desktop-class processor can both be perfectly valid RISC-V implementations because they have different goals.

---

# 4. Basic CPU Microarchitecture

Before studying real processors, it is useful to understand what a CPU does at a high level.

Suppose the processor receives:

```assembly
add x3, x1, x2
```

A simplified execution flow is:

```text
Instruction Memory
       ↓
     Fetch
       ↓
     Decode
       ↓
 Register File
    ↙     ↘
   x1     x2
    \     /
      ALU
       ↓
      x3
```

More generally:

```text
Fetch
  ↓
Decode
  ↓
Read Registers
  ↓
Execute
  ↓
Memory Access
  ↓
Write Back
```

Different processors divide and organize these operations differently.

---

## Datapath

The **datapath** contains hardware through which data moves and is processed.

Typical components include:

```text
Register File
ALU
MUXes
Memory interfaces
Pipeline registers
```

---

## Control Unit

The control unit generates signals that tell the datapath what to do.

For:

```assembly
add x3, x1, x2
```

the control logic conceptually determines:

```text
Read rs1 = x1
Read rs2 = x2
ALU operation = ADD
Write rd = x3
Register write = ENABLE
```

Therefore:

```text
CPU
 |
 +-- Datapath → processes data
 |
 +-- Control  → controls datapath
```

---

# 5. Tiny Cores: SERV and PicoRV32

Tiny cores demonstrate the simplest end of the RISC-V microarchitecture spectrum.

The main question is:

> **How little hardware can we use to implement a useful RISC-V processor?**

---

# 5.1 SERV

SERV stands for:

> **SErial RISC-V**

SERV is a **bit-serial RISC-V processor**.

The important idea is that it uses a very small datapath and processes data serially rather than using a conventional wide datapath.

---

## Why Bit-Serial?

Suppose we want to add two 32-bit numbers.

A conventional 32-bit datapath can operate on the complete width using a 32-bit ALU.

Conceptually:

```text
A = 101101...
B = 001011...
```

SERV processes the operation one bit at a time.

```text
Cycle 1 → bit 0
Cycle 2 → bit 1
Cycle 3 → bit 2
...
Cycle 32 → bit 31
```

For addition, a carry moves from one bit position to the next:

```text
A0 + B0 + Carry
        ↓
      Result0
        ↓
      Carry

A1 + B1 + Carry
        ↓
      Result1
```

The exact internal implementation is more sophisticated, but this illustrates the fundamental idea.

---

## Why Make a CPU This Way?

A smaller datapath means:

```text
Less hardware
   ↓
Less area
   ↓
Potentially lower hardware cost
```

But:

```text
Less parallelism
   ↓
More cycles
   ↓
Lower performance
```

SERV therefore demonstrates a fundamental microarchitecture trade-off:

> **Hardware efficiency can be more important than raw performance.**

---

# 5.2 PicoRV32

PicoRV32 is another small RISC-V processor core.

It is designed to provide a practical, configurable, size-optimized RISC-V implementation for FPGA and ASIC applications.

Compared with SERV:

```text
SERV
↓
Extremely small + bit-serial

PicoRV32
↓
Small + practical + configurable
```

PicoRV32 supports multiple RISC-V configurations and provides interfaces for connecting the processor to a larger system.

---

# 5.3 SERV vs PicoRV32

| Feature     | SERV                   | PicoRV32            |
| ----------- | ---------------------- | ------------------- |
| Main idea   | Bit-serial CPU         | Small practical CPU |
| Hardware    | Extremely small        | Small               |
| Parallelism | Very low               | Higher              |
| Performance | Very low               | Higher              |
| Main lesson | Minimum implementation | Practical small CPU |

The conceptual progression is:

```text
SERV
 ↓
"What is the smallest useful implementation?"

PicoRV32
 ↓
"How can I make a small but practical implementation?"
```

---

# 6. In-Order Pipelines

After tiny processors, we move toward processors that use **pipelining** to improve throughput.

---

# 6.1 Pipelining

A pipeline divides instruction execution into stages.

A simplified five-stage pipeline is:

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

Without pipelining:

```text
Instruction 1:
IF → ID → EX → MEM → WB

Instruction 2:
             IF → ID → EX → MEM → WB
```

With pipelining:

```text
Cycle     1    2    3    4    5

I1       IF   ID   EX   MEM   WB
I2            IF   ID   EX    MEM
I3                 IF   ID    EX
I4                      IF    ID
```

Multiple instructions are therefore being processed simultaneously.

---

## Important Point

Pipelining mainly improves **throughput**.

It does not mean that one instruction suddenly requires only one stage.

Instead:

> **Different instructions occupy different stages at the same time.**

A useful analogy is an assembly line.

```text
Stage 1 → Stage 2 → Stage 3 → Stage 4
```

While one product is in Stage 3, another product can be in Stage 2.

---

# 6.2 In-Order Execution

"In-order" means that instructions are handled according to their program order.

For example:

```assembly
1. add x3, x1, x2
2. sub x5, x4, x6
3. and x7, x8, x9
```

The processor maintains the ordering:

```text
1 → 2 → 3
```

This does not mean that only one instruction exists inside the pipeline at a time.

Instead:

```text
I1 → pipeline
I2 → pipeline
I3 → pipeline
```

can all be present simultaneously, while their ordering remains controlled.

In-order processors are generally simpler than out-of-order processors.

---

# 6.3 Pipeline Hazards

Once instructions overlap, problems can occur.

These are called **pipeline hazards**.

## Data Hazard

Example:

```assembly
add x3, x1, x2
sub x4, x3, x5
```

The second instruction needs the result of the first.

```text
ADD
 ↓
produces x3

SUB
 ↓
needs x3
```

Possible solutions include:

```text
Forwarding
Stalling
```

---

## Control Hazard

Example:

```assembly
beq x1, x2, target
```

The processor may not immediately know which instruction should be fetched next.

This is why processors use mechanisms such as:

```text
Branch prediction
Branch target prediction
Pipeline flushing
```

---

## Structural Hazard

This happens when two instructions need the same hardware resource at the same time.

```text
Instruction A → needs resource X
Instruction B → also needs resource X
```

The processor must resolve the conflict.

---

# 6.4 Rocket

Rocket is a well-known RISC-V **in-order scalar processor core and generator**.

A simplified view of its pipeline is:

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

Rocket is significantly more sophisticated than tiny cores.

It can include:

* caches
* branch prediction
* virtual memory
* privilege support
* configurable ISA extensions
* memory interfaces

Rocket demonstrates an important design point:

> **A processor can achieve useful performance through pipelining without requiring out-of-order execution.**

---

# 6.5 CVA6

CVA6 is another important RISC-V processor.

It is:

```text
6-stage
Single-issue
In-order
```

A simplified conceptual pipeline is:

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

CVA6 also includes more advanced features such as:

* caches
* branch prediction
* TLBs
* virtual memory
* privilege support

The key point is:

> **CVA6 is a sophisticated in-order processor, not an out-of-order processor.**

This makes it useful for comparing increasingly complex in-order designs with later out-of-order designs.

---

# 7. Out-of-Order Cores

Out-of-order execution introduces a major change.

The processor can execute an instruction **when its operands and resources are ready**, rather than strictly waiting for earlier instructions.

---

# 7.1 Why Out-of-Order?

Consider:

```assembly
1. load x1, 0(x2)
2. add  x3, x1, x4
3. add  x5, x6, x7
```

There is a dependency:

```text
1 → 2
```

Instruction 3 is independent.

Suppose instruction 1 takes a long time because of a cache miss:

```text
I1 → waiting
I2 → waiting for I1
I3 → ready
```

An out-of-order processor can execute:

```text
I1 → waiting
I2 → waiting
I3 → execute
```

So:

```text
Program order:
1 → 2 → 3

Possible execution order:
1 → 3 → 2
```

The processor is exploiting **Instruction-Level Parallelism (ILP)**.

---
<img width="1479" height="596" alt="Screenshot 2026-10-08 020148" src="https://github.com/user-attachments/assets/ecfbb803-89ed-4614-874d-3cbdd7a03b6e" />


# 7.2 Register Renaming

Consider:

```assembly
add x1, x2, x3
sub x1, x4, x5
```

Both instructions write to `x1`.

Internally, the processor can assign different physical registers:

```text
Instruction 1 → P7
Instruction 2 → P12
```

Conceptually:

```text
Architectural x1
       |
       +----→ P7
       |
       +----→ P12
```
- A physical register is an actual storage location inside the CPU that holds a value temporarily while instructions are executing.
- These are architectural registers
This is called **register renaming**.

It helps remove false dependencies and allows more instructions to execute independently.

---

# 7.3 Issue Queue

An out-of-order processor needs somewhere to hold instructions that are waiting.

For example:

```text
Instruction       Ready?

ADD                  YES
SUB                  NO
MUL                  YES
LOAD                 NO
```

The processor can select:

```text
ADD → Execute
MUL → Execute
```

while the others remain waiting.

Therefore:

```text
Instructions
     ↓
Issue Queue
     ↓
Find ready instructions
     ↓
Execution Units
```

This allows the processor to use available hardware more effectively.

---

# 7.4 Reorder Buffer

Out-of-order execution creates another problem.

Suppose:

```text
Program order:
I1 → I2 → I3

Execution order:
I1 → I3 → I2
```

If results were permanently committed in execution order, the architectural state could become incorrect.

The **Reorder Buffer (ROB)** tracks instructions in program order.

Therefore:

```text
Execution:
I1 → I3 → I2

Commit:
I1 → I2 → I3
```

This gives us the important principle:

> **Execution can be out of order, while architectural commitment remains in order.**

The ROB is therefore important for maintaining precise architectural state.

---

# 7.5 Speculative Execution

Branches create uncertainty.

For:

```assembly
beq x1, x2, target
```

the processor may not yet know which path will be taken.

A branch predictor makes a prediction:

```text
Branch → Taken
```

The processor can then start fetching and executing instructions from the predicted path.

If the prediction is correct:

```text
Continue normally
```

If incorrect:

```text
Discard incorrect work
        ↓
Fetch correct instructions
```

This is **speculative execution**.

It improves performance by preventing the processor from sitting idle while waiting for every branch decision.

---

# 7.6 BOOM

BOOM stands for:

> **Berkeley Out-of-Order Machine**

BOOM is an open-source RISC-V out-of-order processor core.

A simplified view is:

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

Compared with a simple in-order processor, BOOM requires substantially more hardware for:

```text
Register renaming
Issue queues
Multiple execution units
Speculation
Reorder buffer
Out-of-order scheduling
```

BOOM therefore demonstrates how increasing performance also increases microarchitectural complexity.

---

# 7.7 XiangShan

XiangShan is an open-source, high-performance RISC-V processor project.

It focuses on advanced processor microarchitecture and high performance rather than minimum hardware.

Its design involves sophisticated mechanisms for:

* instruction fetching
* branch prediction
* out-of-order execution
* instruction scheduling
* register renaming
* memory operations
* caches
* retirement

XiangShan demonstrates that RISC-V can serve as the ISA for highly sophisticated processors, not just small embedded CPUs.

---

# 8. Generators

So far we have looked at processor implementations.

Modern hardware design also uses **hardware generators**.

---

# 8.1 Hardware Generators

A traditional HDL project may describe one particular processor.

A generator instead describes a **family of possible hardware designs**.

For example:

```text
Parameters
   |
   +-- Number of cores
   +-- Cache size
   +-- ISA extensions
   +-- Memory configuration
   +-- Other options
             |
             ↓
       Hardware Generator
             |
             ↓
          RTL
             |
             ↓
       FPGA / ASIC
```

This is useful because engineers can explore different designs without manually rewriting the entire processor.

---

# 8.2 Chisel

Chisel is a hardware construction language embedded in Scala.

Conceptually:

```text
Scala + Chisel
       ↓
Hardware construction program
       ↓
Generated RTL
       ↓
Verilog
```

Chisel makes it easier to describe reusable and parameterized hardware.

This is especially useful for processor generators.

---

# 8.3 Rocket Chip

Rocket Chip is a RISC-V hardware generator ecosystem.

It can be used to generate Rocket-based systems with configurable components.

The important distinction is:

```text
Rocket core
      ≠
one completely fixed CPU design
```

Instead, the Rocket ecosystem uses parameterization and generation to create different configurations.

---

# 8.4 Chipyard

Chipyard is a framework for designing RISC-V-based SoCs.

It can combine:

```text
RISC-V cores
+
Caches
+
Memory systems
+
Interconnects
+
Peripherals
+
Accelerators
+
Simulation infrastructure
```

For example:

```text
                 Chipyard
                    |
        +-----------+-----------+
        |           |           |
      Rocket       BOOM        CVA6
        |           |           |
        +-----------+-----------+
                    |
               Interconnect
                    |
          +---------+---------+
          |                   |
        Memory            Peripherals
```

Therefore, Chipyard connects the concepts of:

```text
Core
 ↓
Generator
 ↓
System
 ↓
SoC
```

---

# 9. From Core to SoC

A CPU core is only one component of a complete computer system.

An **SoC (System-on-Chip)** combines the processor with memory, communication infrastructure and peripherals.

```text
+------------------------------------------------+
|                     SoC                        |
|                                                |
|   +---------+       +----------------------+   |
|   |   CPU   | <---> | Cache / Memory      |   |
|   +---------+       +----------------------+   |
|        |                                       |
|        ↓                                       |
|   +----------------------------------------+   |
|   |           Interconnect / Bus           |   |
|   +----------------------------------------+   |
|       |              |             |           |
|      UART            GPIO          SPI         |
|                                                |
+------------------------------------------------+
```

### CPU Core

Executes instructions.

Examples:

```text
Rocket
BOOM
CVA6
PicoRV32
```

### Memory

Stores instructions and data.

### Cache

A small, fast memory close to the processor.

```text
CPU
 ↓
L1 Cache
 ↓
L2 Cache
 ↓
Main Memory
```

### Interconnect

Allows different components to communicate.

```text
CPU ↔ Memory
CPU ↔ UART
CPU ↔ GPIO
CPU ↔ SPI
```

### Peripherals

Examples include:

```text
UART
GPIO
SPI
I2C
Timers
Interrupt Controllers
```

Therefore:

```text
CPU Core
   +
Memory
   +
Interconnect
   +
Peripherals
   =
SoC
```

---

# 10. Comparing the Real RISC-V Cores

| Core          | Type         | Main Idea             | Complexity  | Main Goal                   |
| ------------- | ------------ | --------------------- | ----------- | --------------------------- |
| **SERV**      | Tiny         | Bit-serial            | Very Low    | Minimum area                |
| **PicoRV32**  | Tiny         | Small practical CPU   | Low         | Small implementation        |
| **Rocket**    | In-order     | Pipelined scalar      | Medium      | Balanced design             |
| **CVA6**      | In-order     | 6-stage, single-issue | Medium/High | Application-class processor |
| **BOOM**      | Out-of-order | High-performance OOO  | High        | Performance                 |
| **XiangShan** | Out-of-order | Advanced OOO          | Very High   | High-performance research   |

A conceptual progression is:

```text
                         Complexity
                             ↑
                             |
                       XiangShan
                             |
                           BOOM
                             |
                     Rocket / CVA6
                             |
                        PicoRV32
                             |
                           SERV
                             +----------------→
                                      Performance
```

This is a conceptual learning map, not a benchmark ranking.

---

# 11. The Complete Microarchitecture Journey

The five professor-assigned topics can be connected into one story.

## 1. RISC-V as the Specimen

RISC-V gives us the common ISA.

```text
RISC-V ISA
"What should happen?"
```

---

## 2. Tiny Cores

SERV and PicoRV32 show how little hardware can implement the ISA.

```text
RISC-V
   ↓
SERV / PicoRV32
   ↓
Small hardware
```

---

## 3. In-Order Pipelines

Rocket and CVA6 show how pipelining improves throughput while keeping execution relatively simple.

```text
RISC-V
   ↓
Pipeline
   ↓
Multiple instructions in flight
   ↓
Higher throughput
```

---

## 4. Out-of-Order Execution

BOOM and XiangShan go further by exploiting instruction-level parallelism.

```text
RISC-V
   ↓
Out-of-order execution
   ↓
Find independent instructions
   ↓
Execute when ready
   ↓
Higher performance
```

This requires additional mechanisms:

```text
Register Renaming
Issue Queue
Speculation
Reorder Buffer
Multiple Execution Units
```

---

## 5. Generators and SoCs

Finally, generators allow configurable hardware systems to be created.

```text
Parameters
    ↓
Generator
    ↓
CPU + Memory + Peripherals
    ↓
SoC
```

This gives the complete progression:

```text
RISC-V ISA
    ↓
Tiny Core
    ↓
Pipelined In-Order Core
    ↓
Out-of-Order Core
    ↓
Generator
    ↓
SoC
```

---

# 12. Key Concepts

| Concept               | Meaning                                                    |
| --------------------- | ---------------------------------------------------------- |
| **ISA**               | Defines what instructions do                               |
| **Microarchitecture** | Defines how instructions are implemented                   |
| **Datapath**          | Hardware through which data moves                          |
| **Control Unit**      | Controls datapath operations                               |
| **ALU**               | Performs arithmetic and logical operations                 |
| **Pipeline**          | Divides execution into overlapping stages                  |
| **In-order**          | Instructions are handled according to program order        |
| **Out-of-order**      | Ready instructions can execute before earlier waiting ones |
| **ILP**               | Parallelism between independent instructions               |
| **Register Renaming** | Maps architectural registers to physical registers         |
| **Issue Queue**       | Holds instructions waiting to execute                      |
| **ROB**               | Allows OOO execution while preserving correct commit order |
| **Branch Prediction** | Predicts future control flow                               |
| **Speculation**       | Executes based on predictions                              |
| **Cache**             | Small, fast memory close to the CPU                        |
| **Generator**         | Software that produces configurable hardware               |
| **SoC**               | Complete system containing CPU and supporting hardware     |

---

# 13. Questions to Test My Understanding

## RISC-V

1. What is RISC-V?
2. What is an ISA?
3. Why is RISC-V useful for microarchitecture research?
4. Why can two RISC-V processors have completely different internal hardware?
5. What is the difference between ISA and microarchitecture?

## Tiny Cores

6. What is SERV?
7. What does bit-serial mean?
8. Why does SERV use less hardware?
9. Why is SERV slower?
10. What is PicoRV32?
11. How is PicoRV32 different from SERV?

## Pipelines

12. What is pipelining?
13. Why does pipelining improve throughput?
14. What does in-order mean?
15. What is a data hazard?
16. What is a control hazard?
17. What is a structural hazard?
18. What is forwarding?
19. Why is branch prediction needed?
20. What are Rocket and CVA6?

## Out-of-Order

21. Why do processors use out-of-order execution?
22. What is Instruction-Level Parallelism?
23. What is register renaming?
24. What is an issue queue?
25. What is a reorder buffer?
26. Why can execution be out of order but commitment must remain ordered?
27. What is speculative execution?
28. What are BOOM and XiangShan?

## Generators and SoC

29. What is a hardware generator?
30. Why are generators useful?
31. What is Chisel?
32. What is Rocket Chip?
33. What is Chipyard?
34. What is an SoC?
35. What is the difference between a CPU core and an SoC?
36. Why are caches and interconnects needed?

---

# 14. Conclusion

The central idea of this study is:

> **RISC-V defines the contract; microarchitecture defines how that contract is implemented.**

The same ISA can therefore be implemented using radically different designs:

```text
                         RISC-V ISA
                              |
                              ↓
                    "What should happen?"
                              |
                              ↓
                     Microarchitecture
                              |
        +---------------------+---------------------+
        |                     |                     |
        ↓                     ↓                     ↓
      Tiny                In-Order            Out-of-Order
        |                     |                     |
      SERV              Rocket / CVA6        BOOM / XiangShan
        |                     |                     |
        +---------------------+---------------------+
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

The progression can be remembered as:

```text
SERV
 ↓
Minimum hardware

PicoRV32
 ↓
Small practical CPU

Rocket / CVA6
 ↓
Pipelined in-order execution

BOOM / XiangShan
 ↓
Out-of-order execution and high performance

Rocket Chip / Chipyard
 ↓
Configurable hardware systems

SoC
 ↓
Complete computer system
```

The important question at each stage is:

```text
SERV
"What is the smallest implementation?"

        ↓

PicoRV32
"How can I make a small practical CPU?"

        ↓

Rocket / CVA6
"How can I improve throughput using pipelining?"

        ↓

BOOM / XiangShan
"How can I exploit instruction-level parallelism?"

        ↓

Generators
"How can I create configurable hardware?"

        ↓

SoC
"How do I turn the processor into a complete system?"


