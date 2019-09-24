---
title: Introduction to Parallel Programming
categories:
- Programming
tags:
- C++
- Concurrency
updated:
---
> [CIS 410/510](http://ipcc.cs.uoregon.edu/curriculum.html)
<!-- more -->
## Overview

### Issues

- Resource allocation
- Data access, communication, synchronization
- Performance & Scalability

### Leveraging Moore's Law

- More transistors = more parallelism opportunities
- Microprocessors
  - Implicit parallelism: pipelining; multiple functional units; superscalar
  - Explicit parallelism: SIMD instructions; long instruction works

Current obstacle: power! => Scalable Parallel Computing.

### Parallel Architecture

- Classical: Flynn's Taxonomy
  - 2 Dimensions: Instruction & Data
  - SISI, SIMD, MISD, MIMD
- Types
  - **Refer to 2.1 Architecture.Types**

### Top 500 Benchmarking Methodology

- High-Performance Computing (HPC)
- Rmax : maximal performance Linpack benchmark
  - dense linear system of equations (Ax = b)
- Rpeak : theoretical peak performance
- Nmax : problem size needed to achieve Rmax
- N1/2 : problem size needed to achieve 1/2 of Rmax

### Scalable Parallel Computing

- Scalability in parallel architecture
  - Processor numbers
  - Memory architecture
  - Interconnection network
  - Avoid critical architecture bottlenecks
- Scalability in computational problem
  - Problem size
  - Computational algorithms
    - Computation to memory access ratio
    - Computation to communication ration
- Parallel programming models and tools
- Performance scalability

## Architecture

### Types

- Uniprocessor
  - Scalar Processor
  - Vector Processors
    - Pipelined units
    - Chaining
    - Data parallel / Dataflow Arch
  - SIMD
- Shared-memory Multiprocessor (SMP)
  - Shared Memory Address Space
  - Bus-based Memory System
    - Shared Physical Memory
    - Uniform Memory Access (UMA), Cache Coherency UMA (CC-UMA): Shared Cache
    - Crossbar SMP (UMA SMP)
    - Centralized Memory, "Dance Hall" SMP & Shared Cache (Core: Network)
    - Distributed Memory (NUMA; Cache coherency): Coherent Memory System
  - Interconnection Network
- Distributed Memory Multiprocessor
  - Message Passing Between Nodes
  - Massively Parallel Processor (MPP)
- Cluster of SMPs (_Combination of SMP & DMM_)
  - Shared memory addressing within SMP node
  - Message Passing Between SMP Nodes
  - Regarded as MPP if Lots of Processors
- Multicore
  - Multicore Processor (HW multithreaded; hyperthread)
  - GPU Accelerator
  - "Fused" Processor Accelerator
- Multicore SMP+GPU Cluster (_Combination of Cluster of SMPs & Multicore_)
  - Shared memory addressing within SMP node
  - Message Passing Between SMP Nodes
  - GPU Accelerators Attached
- Multicomputer

### Hardware Parallelism

- Instruction-Level Parallelism (ILP)
  - VLIW
  - Hyperthread
- Data (Share data)
  - Memory Consistency: Write Propagation + Write Serialization.
  - States, State Transition Diagram & Actions.
  - Sequential Consistency: memory operations in program order. => Bus-based Cache-Coherent(CC) Architecture with Cache-Coherency Protocol (e.g. Snoopy).
    - Basic Snoopy: Write Invalidate, Write Broadcast, Write Serialization, Write Invalidate vs Broadcast.
    - Snooping Cache Variations

      | Protocol | Properties |
      | :----- | :-----: |
      | Basic Protocol | Exclusive, Shared, Invalid |
      | Berkeley Protocol | Owned Exclusive, Owned Shared, Shared, Invalid |
      | Illinois Protocol | Private Dirty, Private Clean, Shared, Invalid |
      | MESI Protocol | Modified (private,!=Memory), Exclusive (private,=Memory), Shared (shared,=Memory), Invalid |

  - Need Coherency System to support (or Directories as the generic scalable approach).
- Processor (Increase amount)
  - Distributed Memory Multiprocessors
- Memory System (Increase units & bandwidth)
- Communication (Increase interconnection & bandwidth)

### _TODO_





## TBB Tutorial

_Task-based Parallelism._

- Generic Parallel Algorithm
  Based on [Structured Parallel Programming](http://parallelbook.com/).
- Concurrent Containers
  - Fine-grained Locking
  - Lock-free
- Flow Graph
  A complex graph with many nodes and edges. Based on this design, running a task concurrently in a parallel loop is possible. But we must take care of the usage. Here is a GUI Tool [Flow Graph Analyzer](https://software.intel.com/en-us/advisor/features/flow-graph?_ga=2.14186295.1475735756.1557998035-1386537741.1556067475).
- Scalable Memory Allocator
  - `scalable_allocator<T> `: avoid scalability bottlenecks;
  - `cache_aligned_allocator<T>`: avoid false sharing.
- Thread Local Storage
  - `combinable`: hold per-thread sub-computations -> single result;
  - `enumerable_thread_specific`: container with one element per thread.
- Task Schedular
  - `task_group`
- Synchronization Primitives
  - Mutual Exclusion: Define max threads account in a region of code;
  - Atomic Operations: Avoid mutual exclusion but limited operations.

## _TODO:_ Demo






// Following is the course notes from [UCB-CS267: Applications of Parallel Computers](https://sites.google.com/lbl.gov/cs267-spr2019/)


//TODO: Create a new post page & Move the contents with notes below to the new.

## Lec1: Introduction

### Terms
Application: High Performance Simulation/Data Analytics
**Strong Scaling**: Speedup(P) = Time(1)/Time(P). Fixed problem size, vary number of processors.
**Weak Scaling**: Efficiency-Processors(cores). Fixed problem size per processor. Efficiency can be Flop/s, time, etc.
**Amdahl's Law**: Speedup(P) <= 1/s (P: processor number; s: sequential work fraction)
Parallelism overheads: Creation, Communication, Synchronization, Extra (redundant) Computation => A tradeoff of work unit size.
Single Processor Memory Hierarchy: the larger, the slower => Temporal Locality.

### Course contents
Parallelization Machines:
- Shared memory (OpenMP, PGAS)
- Distributed memory (MPI)
- Data parallel (SPARK, CUDA)

Parallelization Strategies:
- Dense Linear Algebra
- Monte Carlo (Embarrassing Parallelism / Map Reduce)
- Sparse Linear Algebra: Explicit (SpMV) / Implicit (Linear Sys)
- Spectral Methods (FTT)
- Particle Methods
- Structured Grids
- Unstructured Grids
- _Sorting_
- _Graphs_
- _Hashing_

Performance models: Roofline, α-β (latency/bandwidth), LogP

Cross-cutting: Communication avoiding, load balancing, hierarchical algorithms, autotuning, Moore’s Law, Amdahl’s Law, Little’s Law.


## Lec2: Single Processor Machines (HW)

### Costs in modern processors

Ideally: variables, operations, controls => balanced cost
Actually: compilers manage memory & registers => optimize code.

>Optimization: Unrolls/Fuses/Interchanges loops, Eliminates dead code, Reorder instructions (reuse registers), Strength reduction (e.g. *2 => <<), etc.

### Memory hierarchies

Latency/Bandwidth limit performance:
Latency = distance/speed limit = hours;
Bandwidth = cars/mile * miles/hour * #lanes = cars/hour (independent of dis).

Most programs have a high degree of locality:
spatial locality (nearby) v.s. temporal locality (reusing).

Memory hierarchy improves avg, with worse latency:
    Register, On-chip cache, Pages (TLB, VM), ...
Solutions:
    Temporal locality, Spatial locality;
    Parallelism: Vector operations require independent locations, Prefetching, Delayed writing (write buffer), ...
Little's Law (Ideal concurrent issues): concurrency = latency * bandwidth.

### Parallelism within single processors

Instruction Level Parallelism (ILP):
BW = loads / time.
(Not change latency; slowest stage bottleneck; Speedup <= #stages)

SIMD units:
Processing by vectors (SSE2 data types: 16 bytes);
Data dependencies limit parallelism (RAW, WAR, WAW).

Special Instructions:
FMA (Fused Multiply-Add): `x = round(c * z + y)`. (same rate as +/*; matrix).

### Case study: matrix multiplication

Naive Matrix Multiply:

```
//implements C = C + A * B
for i = 1 to n 
    //read row i of A into fast mem
    for j = 1 to n
        //read C(i, j) into fast mem
        //read column j of B into fast mem
        for k = 1 to n
            C(i, j) = C(i, j) + A(i, k) * B(k, j);
        //write C(i, j) back to slow mem
```

Blocked (Tiled) Matrix Multiply (tiling for registers/caches):
(Similarly: Recursive Matrix Multiplication)

```
//A, B, C: n-n matrices => N-N matrices of b-b subblocks
//block size: b = n / N
//3 nested loops inside
for i = 1 to N 
    for j = 1 to N
        //read block C(i, j) into fast mem
        for k = 1 to N
            //read blocks A(i, k), B(k, j) into fast mem
            //multiply on blocks
            //block size = loop bounds
            C(i, j) = C(i, j) + A(i, k) * B(k, j);
        //write block C(i, j) back to slow mem
```

To accelerate computing, make **b as large as possible (~square root of cache size)**.
For multi-level mem: minimize communication; appropriate block size; cache oblivious algorithms.

Theorem (Hong & Kung, 1981): Communication lower bounds for matmul
Any reorganization of matmul (using only associativity) has computational intensity $$q = O( (M_fast)^1/2 )$$, so:
   $$ #words moved between fast/slow memory = Ω (n^3 / (M_fast)^1/2 )$$

Refer to [BLAS (Basic Linear Algebra Subroutines)](www.netlib.org/blas) for more fast matmul algo.

### Optimization

- Tilling for registers: loop unrolling, use of named "register" variables;
- Tilling for multiple levels of cache and TLB;
- Exploiting fine-grained parallelism in processor: rm dependencies, SIMD;
- Complicated compiler interactions (flags);
- Minimize pointer updates (using strided memory addressing).

> Automatic performance tuning (Berkeley centric): 
[ASPIRE](aspire.eecs.berkeley.edu)
[BeBOP](bebop.cs.berkeley.edu) 
[PHiPAC](www.icsi.berkeley.edu/~bilmes/phipac)
          in particular tr-98-035.ps.gz
[ATLAS](www.netlib.org/atlas)
[LAPACK (Linear Algebra PACKage)](http://www.netlib.org/lapack/): a standard linear algebra library optimized to use the BLAS effectively on uniprocessors and shared memory machines (software, documentation and reports)
[ScaLAPACK (Scalable LAPACK)](http://www.netlib.org/scalapack/): A parallel version of LAPACK for distributed memory machines (software, documentation and reports).


## Lec3&4: Sources of Parallelism and Locality in Simulation

### Pre-info

6 parallel hardwares, with corresponding programming models:
Shared, Shared address space, Message passing, Data parallel, Cluster, Grid.

Parallelism and locality in simulation:
Performance; Real world problems (e.g. obj opt independency, nearby dependency, simplify distant dependency, etc.); Scientific models; Parallelism at multi-levels.

Simulation basic category _=> Main content of this lecture_.
> A given phenomenon can be modeled at multiple levels.
> Many simulations combine more than one of these techniques.

Common problems: load balancing; locality; tradeoff.

Sharks and Fish #1-6.

### Discrete Event System

Models: discrete time & space.

Elements: finite set of variables, "state", "transition function".

Approaches:
1. Decompose domain: **Graph partitioning** assigns subgraphs to processors, i.e. determines parallelism and locality.
  Principle: a) Evenly distribute (load balance); b) Minimize edge crossings (minimize communication). (NP-hard => approximate)
2. Run each components ahead using.
   - Synchronous (**State Machine**): change at every clock tick (time driven);
     Synchronous Circuit Simulation: communicate at end of each time step.
   - Asynchronous (**Event Driven Simulation**): change only if inputs changes (event driven); no global time steps, but individual time stamp.
     Scheduling Asynchronous Circuit Simulation: communicate on-demand. 
     Conservative (wait for inputs: deadlock detection) v.s. Speculative/Optimistic (assume no inputs: backup & roll-back).
     More efficient, harder to parallelize and local. (e.g. distributed mem: MPI)

### Particle System

Models: continuous time & space (usually combined with discrete events).

Elements: finite number of particles, Newton's Laws.

> $$force = external_force + nearby_force + far_field_force$$

- External Force: force on each particle is independent.
  "Embarrassingly Parallel / Map Reduce"
- Nearby Force: depend on other nearby particles, thus require interaction and communication.
  Domain decomposition: O(n/p) particles/processor.
  Challenge:
    1) Interactions near boundary => minimize ghost zone;
    2) Load imbalance  => divide space unevenly.
- Far-Field Force: force depends on all other particles, thus all-in-all communication. Worst O(n^2).
  - Particle-Mesh Methods: PDEs, O(n log n) or O(n).
    Approximation:
      1) Move particles to nearby mesh points (scatter);
      2) Solve mesh problem (e.g. FFT, multigrid);
      3) Interpolate forces from mesh points (gather).
  - Tree Decomposition: group up particles, O(n log n) or O(n).
    Optional algorithms:
      1) Barnes-Hut;
      2) FMM (fast multiple method) of Greengard/Rohkin;
      3) Anderson's method.

### Lumped System (ODEs)

Models: lumped variables on continuous parameters (usually time, thus differentiate to time).
Variant: ODEs with some constraints (i.e. DAEs: Differential Algebraic Eq.)

Applications:
- Approximate as graph;
- Compute endpoints only;
- Laws depend on field:
  EE: Ohm's Law, Kirchoff's Laws, etc.
  ME: F=ma, Hook's Law, etc.

Solving:
1. Value at time t:
  ODE: x'(t) = f(x) = A * x(t), A is a sparse matrix, compute x(i * dt) = x[i].
  - Explicit methods.
    x'(i * dt) = slope => x[i + 1] = x[i] + dt * slope(i)
    e.g. (Forward) Euler's method: approximate x'(t) = A * x(t) by (x[i + 1] - x[i]) / dt = A * x[i], thus x[i + 1] = x[i] + dt * A * x[i].
    Simple algo for sparse-matrix Vector Multiplication (SpMV: **Graph partitioning** => matrix reordering => parallel); may need update frequently.
    _SpMV in Compressed Sparse Row (CSR) format:_
    _y = y + A * x, store/arithmetic on nonzero only._
  - Implicit methods.
    x'((i + 1) * dt) = slope => x[i + 1] = x + dt * slope(i + 1)
    e.g. Backward Euler solve: similar to the above.
    Larger timestep; need solve linear sys each step.
  - Direct methods (Gaussian elimination): LU Decomposition.
  - Iterative solvers: Jacobi, SOR, CG, Multigrid, ...
2. Modes of vibration.
  - Eigenvalue problems.
    Step1: Solve x(t) = sin(ω * t) * x0 => d^2x(t)/dt^2 = A * x(t), x0 "mode shape".
    Step2: Plug in to get -ω^2 * x0 = A * x0, thus -ω^2 is eigenvalue, x0 is eigenvector of A.
    Step3: Solution schemes reduce to sparse-matrix multiplications or sparse linear systems.

### Partial Differential Equations (PDEs)

Models: continuous variables depending on continuous parameters.

Categories (can be mixed):
Elliptic pro: steady state, global space dependence (e.g. Potential);
Hyperbolic pro: time dependent, local space (e.g. Pressure);
Parabolic pro: time dependent, global space (e.g. Temp, Concentration).
_*global: many communication or tiny timesteps;_
_*local: finite wave speed thus limited communication._

Solutions are similar to the ODEs (explicit, implicit, ...).
General Algorithms:
Dense LU, Band LU, Jacobi, Explicit Inverse, Conjugate Gradient, Red-Black SOR, Sparse LU, FFT, Multigrid, Lower Bound.
> [Source Code](www.cs.berkeley.edu/~demmel/ma221)

Practical meshes (irregular; mesh => matrix => reordering):
- Composite meshes;
- Unstructured meshes;
- Adaptive meshes (AMR: Adaptive Mesh Refinement).
