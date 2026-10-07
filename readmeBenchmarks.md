# Benchmarks & Metrics

## How Fast Is Fast?

When evaluating a processor, simply saying that one CPU is "faster" than another is not enough.

A processor may perform very well on integer calculations but poorly on floating-point workloads. Another processor may have a lower clock frequency but execute more useful work per cycle.

This is why processor performance is measured using **benchmarks** and **metrics**.

A benchmark is a program or workload used to measure a particular aspect of system performance.

The major benchmarks discussed here are:

- MIPS
- Dhrystone
- CoreMark
- Embench
- SPEC

The most important lesson is:

> **A benchmark number has meaning only when we know what was measured, how it was measured, and under what conditions.**

---

# Table of Contents

1. [What Is a Benchmark?](#1-what-is-a-benchmark)
2. [What Does "Fast" Mean?](#2-what-does-fast-mean)
3. [Important Performance Metrics](#3-important-performance-metrics)
4. [MIPS](#4-mips)
5. [Dhrystone](#5-dhrystone)
6. [CoreMark](#6-coremark)
7. [Embench](#7-embench)
8. [SPEC](#8-spec)
9. [Comparing the Benchmarks](#9-comparing-the-benchmarks)
10. [Why One Benchmark Is Not Enough](#10-why-one-benchmark-is-not-enough)
11. [How to Read Benchmark Numbers Honestly](#11-how-to-read-benchmark-numbers-honestly)
12. [Benchmarking a RISC-V Core](#12-benchmarking-a-risc-v-core)
13. [Example: Comparing Two RISC-V Cores](#13-example-comparing-two-risc-v-cores)
14. [Benchmarking and Microarchitecture](#14-benchmarking-and-microarchitecture)
15. [Common Benchmarking Mistakes](#15-common-benchmarking-mistakes)
16. [Key Takeaways](#16-key-takeaways)

---

# 1. What Is a Benchmark?

A **benchmark** is a standardized program or workload used to measure the performance of a computer system.

Instead of simply saying:

> CPU A is faster than CPU B.

we run the same workload on both processors.

For example:

    Same Benchmark
          |
    +-----+-----+
    |           |
   CPU A       CPU B
    |           |
  100 ms      150 ms
    |           |
    +-----+-----+
          |
     CPU A is faster

The benchmark provides a common workload for comparison.

However, the result only tells us how the processors behave **for that particular workload**.

A processor that performs well on one benchmark may not perform equally well on another.

---

# 2. What Does "Fast" Mean?

There is no single definition of processor speed.

Performance depends on many factors:

    Processor Performance
             |
    +--------+--------+
    |        |        |
  Clock     IPC     Memory
 Frequency          System
    |        |        |
   GHz    Instructions Cache
          per Cycle  Latency

Important factors include:

- Clock frequency
- Instructions Per Cycle (IPC)
- Cycles Per Instruction (CPI)
- Number of cores
- Pipeline design
- Cache
- Memory system
- Branch handling
- Instruction set
- Compiler
- Workload

A processor running at 3 GHz is not automatically faster than one running at 2 GHz.

The 2 GHz processor may perform more useful work in each clock cycle.

## 2.1 CPU Time

A simplified relationship is:

    CPU Time = Instruction Count × CPI / Clock Frequency

where:

- **Instruction Count** = number of instructions executed
- **CPI** = average cycles per instruction
- **Clock Frequency** = cycles per second

Since approximately:

    IPC = 1 / CPI

we can also think of performance in terms of:

    Performance ∝ Clock Frequency × IPC

assuming the workload and other conditions remain comparable.

---

# 3. Important Performance Metrics

Different benchmarks report performance in different ways.

| Metric | Meaning |
|---|---|
| Execution Time | Time required to complete a workload |
| Throughput | Amount of work completed per unit time |
| MIPS | Million Instructions Per Second |
| DMIPS | Dhrystone MIPS |
| CoreMark | Performance score from CoreMark |
| CoreMark/MHz | CoreMark performance normalized by frequency |
| Embench Score | Performance across embedded workloads |
| SPEC Score | Performance on standardized application workloads |
| CPI | Average cycles per instruction |
| IPC | Average instructions per cycle |

The metric should always be interpreted together with the benchmark.

For example:

    1000 MIPS

does not automatically mean:

    1000 times faster than 1 MIPS

because the instruction sets, instruction counts, workloads and measurement conditions may be different.

---

# 4. MIPS

## 4.1 What Is MIPS?

**MIPS** stands for:

> Million Instructions Per Second

It represents approximately how many million machine instructions a processor executes per second.

The formula is:

    MIPS = Instructions Executed / (Execution Time × 10^6)

### Example

Suppose a processor executes:

    500 million instructions

in:

    2 seconds

Then:

    MIPS = 500 / 2
         = 250 MIPS

So the processor executes approximately:

    250 million instructions per second

---

## 4.2 MIPS and CPI

A simplified relationship is:

    MIPS = Clock Frequency / (CPI × 10^6)

Suppose:

    Clock Frequency = 100 MHz
    CPI = 2

Then:

    MIPS = 100 / 2
         = 50 MIPS

If CPI becomes:

    CPI = 1

then:

    MIPS = 100 / 1
         = 100 MIPS

Therefore, reducing CPI can increase MIPS without increasing the clock frequency.

---

## 4.3 Why MIPS Can Be Misleading

The major problem with MIPS is:

> **Not all instructions represent the same amount of useful work.**

Imagine:

    CPU A → 1 billion instructions
    CPU B → 500 million instructions

The two CPUs may perform the same high-level task using different numbers of instructions.

Therefore, comparing MIPS alone can give a misleading impression.

Different ISAs may also require different numbers of instructions to perform the same operation.

So:

    Higher MIPS
         ≠
    Automatically higher real-world performance

---

## 4.4 MIPS vs MHz

These are different concepts.

    MHz / GHz
        ↓
    Number of clock cycles per second

    MIPS
        ↓
    Number of million instructions executed per second

Clock frequency describes the operating rate of the processor.

MIPS is a performance metric derived from instruction execution.

---

# 5. Dhrystone

## 5.1 What Is Dhrystone?

**Dhrystone** is a classic synthetic benchmark designed to measure general integer and system performance.

It was introduced in 1984 and became widely used for evaluating processors, especially embedded processors.

Dhrystone contains operations involving:

- Integer calculations
- Function calls
- Pointers
- Arrays
- Structures
- Strings
- Control flow

It is primarily an integer-oriented benchmark.

---

## 5.2 DMIPS

Dhrystone results are commonly reported as:

> **DMIPS — Dhrystone MIPS**

A traditional normalization uses a reference performance of 1757 Dhrystones per second:

    DMIPS = Dhrystones per second / 1757

For example, if a system achieves:

    17570 Dhrystones/second

then:

    DMIPS = 17570 / 1757
          = 10 DMIPS

DMIPS is therefore a normalized Dhrystone result.

It should not be interpreted as general-purpose "real MIPS."

---

## 5.3 Why Dhrystone Became Popular

Dhrystone became popular because it was:

- Small
- Easy to run
- Easy to port
- Suitable for embedded systems
- Relatively inexpensive
- Simple to understand

This made it convenient for comparing processors with limited resources.

---

## 5.4 Limitations of Dhrystone

Dhrystone is an old benchmark and does not represent every type of modern workload.

Possible limitations include:

- Small working set
- Limited representation of modern applications
- Compiler optimization effects
- Benchmark-specific optimization
- Weak representation of memory-intensive workloads
- Limited floating-point coverage

Therefore:

> Dhrystone is useful for simple and historical comparisons, but it should not be treated as a complete measurement of processor performance.

---

# 6. CoreMark

## 6.1 What Is CoreMark?

**CoreMark** is a benchmark designed to measure the performance of CPU cores, particularly in embedded systems.

It was developed by **EEMBC**.

CoreMark includes workloads involving:

- Linked-list processing
- Matrix operations
- State-machine processing
- CRC calculations

This provides a more modern embedded-oriented workload than older benchmarks such as Dhrystone.

---

## 6.2 CoreMark Score

A CoreMark result can be reported as:

    CoreMark

or normalized as:

    CoreMark/MHz

Suppose:

    CoreMark = 250
    Clock Frequency = 100 MHz

Then:

    CoreMark/MHz = 250 / 100
                 = 2.5

---

## 6.3 Why CoreMark/MHz Is Useful

Consider:

    CPU A:
    CoreMark = 400
    Frequency = 200 MHz

    CPU B:
    CoreMark = 450
    Frequency = 300 MHz

Raw CoreMark:

    CPU B > CPU A

But normalized performance:

    CPU A = 400 / 200
          = 2.0 CoreMark/MHz

    CPU B = 450 / 300
          = 1.5 CoreMark/MHz

Therefore:

    CPU B:
    Higher total performance

    CPU A:
    Higher performance per MHz

These answer two different questions.

---

## 6.4 What CoreMark Does Not Tell Us

CoreMark is not a universal measure of CPU performance.

It does not directly tell us how a processor will perform on:

- Operating systems
- Web browsers
- Databases
- AI workloads
- Large scientific applications
- Heavy floating-point workloads

It is primarily useful for evaluating embedded CPU/core performance.

---

# 7. Embench

## 7.1 What Is Embench?

**Embench** is a benchmark suite designed for deeply embedded systems.

Instead of depending on one small workload, Embench contains multiple benchmarks intended to represent different types of embedded computation.

Conceptually:

                         Embench
                            |
          +-----------------+-----------------+
          |                 |                 |
       Workload A        Workload B        Workload C
          |                 |                 |
          +-----------------+-----------------+
                            |
                    Overall Performance

---

## 7.2 Why Embench Is Useful

A single benchmark can favor a particular architecture.

Using multiple workloads gives us a broader view of performance.

Embench is useful for evaluating:

- Microcontrollers
- Small RISC-V cores
- FPGA processors
- Low-power processors
- Embedded systems

---

## 7.3 Embench and RISC-V

Embench is especially relevant to RISC-V because many RISC-V implementations target:

    Small
      ↓
    Low-power
      ↓
    Resource-constrained
      ↓
    Embedded

systems.

A small RISC-V core may not have enough resources to run large desktop/server benchmarks.

Embench provides workloads that are much more suitable for this class of processor.

---

## 7.4 Important Point About Embench Scores

Embench results depend on:

- Benchmark version
- Compiler
- Compiler options
- Processor frequency
- Memory system
- Hardware implementation

Therefore, two Embench scores should not be compared blindly.

Always identify the conditions under which the score was obtained.

---

# 8. SPEC

## 8.1 What Is SPEC?

**SPEC** stands for:

> Standard Performance Evaluation Corporation

SPEC develops standardized benchmark suites for evaluating computer systems.

SPEC benchmarks are generally much larger and more demanding than small embedded benchmarks.

---

## 8.2 SPEC CPU

One of the major SPEC benchmark families is:

> **SPEC CPU**

SPEC CPU contains CPU-intensive workloads designed to evaluate processor and memory-system performance.

It includes:

- Integer workloads
- Floating-point workloads

This makes it considerably broader than benchmarks such as Dhrystone.

---

## 8.3 SPECint and SPECfp

Historically, SPEC results have commonly been discussed in terms of:

    SPECint

for integer-oriented workloads and:

    SPECfp

for floating-point workloads.

Modern SPEC CPU suites use specific benchmark sets and reporting rules, so the exact benchmark suite/version should always be mentioned.

---

## 8.4 SPEC Ratio

SPEC commonly uses a ratio-based approach.

Conceptually:

    SPEC Ratio = Reference Time / Test Time

Suppose:

    Reference Time = 100 seconds
    CPU A Time = 50 seconds

Then:

    Ratio = 100 / 50
          = 2

Another processor:

    Reference Time = 100 seconds
    CPU B Time = 25 seconds

Then:

    Ratio = 100 / 25
          = 4

CPU B completes the workload in half the time of CPU A, so its ratio is twice as high.

---

## 8.5 Why SPEC Is Different

SPEC uses a collection of substantial workloads rather than one tiny synthetic workload.

Conceptually:

                         SPEC
                           |
          +----------------+----------------+
          |                                 |
       Integer                       Floating Point
          |                                 |
          +----------------+----------------+
                           |
                  Multiple Workloads
                           |
                    Broader Evaluation

This makes SPEC useful for larger CPUs and systems.

---

## 8.6 Why SPEC Is Not Ideal for Tiny Cores

A tiny embedded RISC-V core may not have:

- Enough memory
- Operating-system support
- Required runtime environment
- Floating-point hardware
- Sufficient performance

to run large SPEC workloads efficiently.

Therefore, for small RISC-V cores, benchmarks such as:

    Dhrystone
    CoreMark
    Embench

are often more practical.

---

# 9. Comparing the Benchmarks

| Benchmark | Main Measurement | Typical Target | Main Advantage |
|---|---|---|---|
| MIPS | Instructions per second | General processors | Very simple metric |
| Dhrystone | Integer/system performance | Embedded processors | Small and easy to run |
| CoreMark | CPU core performance | Embedded systems | Modern embedded benchmark |
| Embench | Multiple embedded workloads | Deeply embedded systems | Broader embedded evaluation |
| SPEC CPU | CPU-intensive application workloads | Powerful CPUs/systems | Large and diverse workloads |

The important point is:

> **There is no single benchmark that is best for every processor.**

The correct benchmark depends on what we want to measure.

---

# 10. Why One Benchmark Is Not Enough

Consider two processors:

| Benchmark | CPU A | CPU B |
|---|---:|---:|
| Dhrystone | 100 | 90 |
| CoreMark | 150 | 180 |
| Embench | 80 | 95 |

Using only Dhrystone:

    CPU A appears faster.

Using CoreMark:

    CPU B appears faster.

Using Embench:

    CPU B appears faster.

This does not necessarily mean that one benchmark is wrong.

It means the processors behave differently across different workloads.

Therefore:

> **A good processor evaluation should use a suitable collection of benchmarks rather than relying on one number.**

---

# 11. How to Read Benchmark Numbers Honestly

This is one of the most important parts of benchmarking.

A benchmark result should never be presented without its context.

Instead of saying:

> "CPU A has a CoreMark score of 500, so it is faster."

we should ask:

    Which CoreMark version?
    Which compiler?
    Which compiler flags?
    Which clock frequency?
    Which memory system?
    Which hardware?
    Which operating conditions?
    Which optimization settings?
    Which reference system?
    Was it real hardware, FPGA or simulation?

---

## 11.1 Benchmark Version

Always identify the benchmark version.

For example:

    CoreMark version = ______

or:

    SPEC CPU version = ______

Different versions can use different workloads or methodologies.

---

## 11.2 Compiler

The compiler can have a major effect on performance.

Consider:

    Compiler A
        ↓
    Execution Time = 1.0 s

    Compiler B
        ↓
    Execution Time = 0.8 s

The processor did not change.

The generated machine code changed.

Therefore:

> Benchmark performance is partly a measurement of the processor and partly a measurement of the software toolchain.

---

## 11.3 Compiler Optimization

Compiler optimization levels can significantly change the generated code.

Common examples include:

    -O0
    -O1
    -O2
    -O3

A benchmark compiled with aggressive optimization may run much faster than the same benchmark compiled without optimization.

Therefore, comparisons should use documented and consistent compiler settings.

---

## 11.4 Clock Frequency

Raw performance generally increases with clock frequency if the architecture and workload remain comparable.

For example:

    CPU A = 100 MHz
    CPU B = 200 MHz

CPU B has twice as many clock cycles available per second.

However:

    Higher frequency
         ≠
    Better architecture

This is why normalized metrics such as:

    CoreMark/MHz

can be useful.

---

## 11.5 Memory System

The CPU core is not the only component affecting benchmark performance.

A simplified system looks like:

    CPU Core
       |
       ↓
     Cache
       |
       ↓
    Memory Bus
       |
       ↓
      RAM

Performance can depend on:

- Cache size
- Cache latency
- Memory latency
- Memory bandwidth
- Bus architecture
- Memory wait states

Therefore, a benchmark result may measure the entire system rather than only the CPU execution units.

---

## 11.6 Workload

Different programs stress different parts of a processor.

For example:

    Integer-heavy workload
            ↓
        Integer ALU

    Floating-point workload
            ↓
             FPU

    Memory-heavy workload
            ↓
       Cache + Memory

    Branch-heavy workload
            ↓
      Branch Handling

Therefore:

> **Benchmark performance is workload-dependent.**

---

# 12. Benchmarking a RISC-V Core

Suppose we have designed or implemented a RISC-V core.

We want to answer:

> How fast is our core?

A basic benchmarking process is:

                 RISC-V Core
                      |
                      ↓
               Select Benchmark
                      |
                      ↓
             Compile for RISC-V
                      |
                      ↓
             Generate Machine Code
                      |
                      ↓
              Run on the Core
                      |
                      ↓
             Measure Execution
                      |
                      ↓
             Calculate Metrics

---

## 12.1 What Should Be Recorded?

A proper benchmark report should record:

    Processor/Core:
    RISC-V ISA:
    XLEN:
    Clock Frequency:
    Pipeline:
    Cache:
    Memory:
    Benchmark:
    Benchmark Version:
    Compiler:
    Compiler Version:
    Compiler Flags:
    Execution Time:
    Instruction Count:
    Cycle Count:
    Final Score:
    Platform:

This makes the result easier to reproduce and compare.

---

# 13. Example: Comparing Two RISC-V Cores

Suppose we have two RISC-V cores.

## Core A

    Clock = 100 MHz
    CoreMark = 200

## Core B

    Clock = 200 MHz
    CoreMark = 300

Looking only at the raw score:

    Core B > Core A

because:

    300 > 200

But normalize by frequency:

    Core A:

    200 / 100
    = 2.0 CoreMark/MHz

    Core B:

    300 / 200
    = 1.5 CoreMark/MHz

Therefore:

    Core B:
    Higher total performance

    Core A:
    Higher performance per MHz

These are different conclusions.

---

# 14. Benchmarking and Microarchitecture

Benchmarks are particularly useful when studying **microarchitecture**.

A processor's microarchitecture includes things such as:

- Pipeline depth
- ALU organization
- Register file
- Branch handling
- Cache organization
- Memory interface
- Hazard handling
- Instruction scheduling
- Out-of-order execution
- Superscalar execution

Changing the microarchitecture can change benchmark performance.

For example:

                    Same ISA
                       |
              +--------+--------+
              |                 |
            Core A            Core B
         3-stage pipeline   5-stage pipeline
              |                 |
              +--------+--------+
                       |
                   Benchmark
                       |
                 Compare Results

Both processors can implement the same RISC-V ISA while having very different performance.

This demonstrates an important distinction:

    ISA
     ↓
    What instructions the processor supports

    Microarchitecture
     ↓
    How those instructions are actually executed

---

# 15. Benchmarking a Tiny RISC-V Core

For a tiny RISC-V core, the goal is usually not to compete directly with a desktop CPU.

Instead, we may care about:

    Performance
         +
       Area
         +
       Power
         +
     Frequency
         +
    Memory Requirements

A useful evaluation might therefore look like:

                       RISC-V Core
                            |
             +--------------+--------------+
             |              |              |
         Performance       Area           Power
             |              |              |
          CoreMark       FPGA LUTs       Energy
          Embench        Registers       Consumption
          Dhrystone

This is especially important for embedded processors.

A core that is slightly slower but dramatically smaller and lower-power may be the better design.

---

# 16. Performance Per MHz vs Total Performance

These two metrics should not be confused.

## Total Performance

Answers:

> How much work does the processor complete?

Example:

    CoreMark = 500

---

## Performance Per MHz

Answers:

> How efficiently does the architecture perform at a given clock frequency?

Example:

    CoreMark/MHz = 2.5

A processor can therefore have:

    Higher total performance

but:

    Lower performance per MHz

if it simply operates at a much higher frequency.

---

# 17. Performance Per Watt

For embedded systems, performance alone is not enough.

We may also care about:

    Performance / Power

For example:

    Core A:
    Performance = 100
    Power = 2 W

    Core B:
    Performance = 120
    Power = 5 W

Core B is faster.

But:

    Core A = 100 / 2
           = 50 performance/W

    Core B = 120 / 5
           = 24 performance/W

Core A is more efficient in this simplified example.

Therefore:

> The fastest processor is not always the most efficient processor.

---

# 18. Common Benchmarking Mistakes

## Mistake 1: Comparing Different Benchmarks

Bad comparison:

    CPU A = 500 CoreMark
    CPU B = 300 DMIPS

These are different metrics from different benchmarks.

The numbers cannot simply be compared directly.

---

## Mistake 2: Ignoring Frequency

Comparing:

    Core A = 100 MHz
    Core B = 500 MHz

using only raw benchmark scores may hide architectural efficiency.

---

## Mistake 3: Ignoring the Compiler

Different compiler versions and optimization flags can produce different results.

---

## Mistake 4: Treating MIPS as Universal Performance

MIPS measures instruction throughput.

It does not directly measure how much useful application work is completed.

---

## Mistake 5: Using Only One Benchmark

One benchmark can favor a particular architecture.

Multiple workloads give a more reliable picture.

---

## Mistake 6: Ignoring Memory

A benchmark may be limited by memory rather than the ALU or pipeline.

---

## Mistake 7: Ignoring Benchmark Version

A score without the benchmark version is difficult to reproduce or compare reliably.

---

## Mistake 8: Comparing Simulation and Real Hardware Without Care

A benchmark running in:

    RTL simulation

may behave very differently from the same benchmark running on:

    FPGA

or:

    ASIC

Simulation is useful for functional verification and architectural study, but its timing performance should not automatically be treated as real hardware performance.

---

# 19. A Better Way to Report a Benchmark

Instead of:

    Our RISC-V CPU achieved 500.

report something like:

    CoreMark Score: 500
    Clock Frequency: 100 MHz
    CoreMark/MHz: 5.0
    Compiler: ______
    Compiler Version: ______
    Optimization: -O2
    ISA: RV32I
    Memory Configuration: ______
    Benchmark Version: ______
    Platform: FPGA / RTL Simulation / ASIC

This makes the result much more meaningful.

---

# 20. Benchmark Hierarchy

Different benchmarks answer different questions.

                         Benchmarks
                              |
          +-------------------+-------------------+
          |                   |                   |
       Simple              Embedded            Large
       Metrics             Workloads           Workloads
          |                   |                   |
        MIPS          +-------+-------+          SPEC
                      |       |       |
                  Dhrystone CoreMark Embench

This is not a strict hierarchy of "better" and "worse."

It is a hierarchy of **different purposes**.

---

# 21. What Should We Use for RISC-V?

The appropriate benchmark depends on the RISC-V implementation.

## Tiny Embedded Core

Good choices:

    Dhrystone
    CoreMark
    Embench

## Microcontroller-Class RISC-V

Useful choices:

    CoreMark
    Embench
    Dhrystone

## Larger RISC-V Processor

More extensive workloads may be appropriate:

    SPEC CPU

The exact benchmark should always match the processor's intended application.

---

# 22. Benchmarking Formula Cheat Sheet

## MIPS

    MIPS = Instructions / (Time × 10^6)

## CPU Time

    CPU Time = Instruction Count × CPI / Clock Frequency

## IPC

    IPC ≈ 1 / CPI

## CoreMark/MHz

    CoreMark/MHz = CoreMark Score / Clock Frequency in MHz

## Dhrystone DMIPS

Traditional normalization:

    DMIPS = Dhrystones per Second / 1757

## SPEC Ratio

Conceptually:

    SPEC Ratio = Reference Time / Test Time

The exact reporting and aggregation rules depend on the specific benchmark suite and version.

---

# 23. The Most Important Idea

A benchmark number is **not an absolute property of a processor**.

It is the result of an experiment involving:

    Processor
        +
    Microarchitecture
        +
    Clock Frequency
        +
    Memory System
        +
    Compiler
        +
    Compiler Flags
        +
    Benchmark
        +
    Benchmark Version
        +
    Software Environment
        =
    Measured Result

Therefore:

> **A benchmark score should always be read together with the conditions under which it was obtained.**

---

# 24. Key Takeaways

### 1. MIPS

Measures:

    Million Instructions Per Second

It is simple, but not a reliable universal measure of useful performance.

---

### 2. Dhrystone

A classic integer-oriented benchmark.

Useful for:

    Simple embedded comparisons

but limited as a representation of modern workloads.

---

### 3. CoreMark

A popular embedded CPU benchmark.

Useful metrics include:

    CoreMark
    CoreMark/MHz

---

### 4. Embench

A collection of embedded workloads.

Useful for:

    Microcontrollers
    Small RISC-V cores
    FPGA processors
    Deeply embedded systems

---

### 5. SPEC

A large benchmark family designed for more substantial computing systems.

It provides a broader evaluation than small synthetic benchmarks.

---

### 6. No Single Number Tells the Whole Story

A processor can win one benchmark and lose another.

Therefore:

    One benchmark
         ↓
    One perspective

    Multiple suitable benchmarks
         ↓
    Better understanding

---

### 7. Always Check the Conditions

When comparing two benchmark results, ask:

    Same benchmark?
    Same version?
    Same compiler?
    Same optimization?
    Same frequency?
    Same workload?
    Same memory conditions?
    Same measurement method?

If not, the comparison may not be fair.

---

# Final Summary

The question:

> "How fast is this processor?"

is incomplete.

A better question is:

> **"How fast is this processor for this workload, under these conditions, compared with what reference?"**

MIPS, Dhrystone, CoreMark, Embench and SPEC each provide different ways of answering that question.

For RISC-V, especially small embedded cores, benchmarks such as **Dhrystone, CoreMark and Embench** can provide useful measurements, while larger processors may benefit from more comprehensive suites such as **SPEC CPU**.

The goal of benchmarking is not to find a single magical number.

The goal is to understand:

    WHAT was measured
          +
    HOW it was measured
          +
    UNDER WHICH CONDITIONS
          +
    WHAT the result actually means

That is how benchmark numbers should be read honestly.
