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

## Lec1: Intro

### Notes
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
- Monte Carlo 
- Sparse Linear Algebra
- Spectral Methods
- Particle Methods
- Structured Grids
- Unstructured Grids
- _Sorting_
- _Graphs_
- _Hashing_

Performance models: Roofline, α-β (latency/bandwidth), LogP

Cross-cutting: Communication avoiding, load balancing, hierarchical algorithms, autotuning, Moore’s Law, Amdahl’s Law, Little’s Law.





