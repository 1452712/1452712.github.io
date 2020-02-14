---
title: 'API Research: TBB, GCD & STL'
categories:
- Programming
tags:
- C++
- Concurrency
updated:
---

<!-- more -->

## Overview

Based on the current OGS usages, the following chart shows differences of required features between TBB, GCD & STL(C++11):

| Lib | Platform Support | Thread-Safe Container | Parallel Algorithms |
|:---:| ---:| ---:| ---:|
| TBB | | | |
| STL | | | |
| GCD | | | |

Here are some suggestions:
- For current OGS:
  - Replace duplicated or deprecated tbb modules/APIs with STL;
  - Keep TBB's concurrency container;
  - Simplify TBB parallel algorithms with STL;
  - // TODO: Demo to check Cache Miss Performance Impact.

- For GSF:
  - //TODO

Key ideas for multithreading:
//

********

## TBB
// Intro...

### Platform Support

| Category | Description |
| :--- | :--- |
| **Processors** | **Optimized for all compatible Intel® processors including Intel Atom®, Intel® Core™, Intel® Xeon®, and Intel® Xeon Phi™ processors.** |
| Language | C++ |
| Portability and Compatibility | Open source under an Apache* license <br> Compatible with multiple compilers (compiler agnostic) |
| Operating Systems	| Windows*, Linux*, macOS*, Android* additional with open source |

So basically, TBB can only benefit Intel CPUs. However, the latest market-share chart shows that, even though Intel still occupies more than 60% of the CPU market, AMD is increasing obviously:

![IntelAMDMarketShare](/contents/images/API-Research-TBB-GCD-STL/Intel-AMD-Market-Share.png)

And we should notice the potentiality in their new products and techniques. Take the comparison between two recent parts as an example:

|  | Base speed	| Max speed	| TDP | Price |
| :--- | :---: | :---: | :---: | :---: |
| Ryzen 7 3800X	| 3.9GHz | 4.5GHz | 105 watts | $399 |
| Core i9-9900K | 3.6GHz | 5.0GHz | 95 watts | $488 |

> With Intel Turbo Boost Technology, the Intel chip can hit the 5.0GHz ceiling using two cores. The boost number drops to 4.8GHz using four cores and 4.7GHz using eight cores. Meanwhile, AMD’s Precision Boost 2 technology increases the speed of any number of cores . The increase is based on an analysis of the current environment involving thermal, electrical, and headroom utilization. It’s what AMD calls the “reliability triangle.”<br>
> That said, the AMD chip has a base speed advantage while the Intel chip has a higher turbo speed ceiling.
### Duplicated Modules in STL

Here are some duplicated functionalities in TBB that OGS often uses.

| TBB | STL |
| :--- | :--- |
| tbb::atomic | std::atomic |
| tbb::tbb_thread | std::thread |
| tbb::mutex | std::mutex |
| tbb::recursive_mutex | std::recursive_mutex |
| tbb::reader_writer_lock | std::shared_mutex |
| tbb::hash & tbb:hasher | std::hash |

### // ...

### Summary
// Pros & Cons

********

## STL
// Intro...
// Related Libs: thread, locks, condition variable

![STLThreadLevel](/contents/images/API-Research-TBB-GCD-STL/STL-Thread-Level.png)

### Compiler Support

Currently, OGS are using C++11, which are almost fully supported by all the main compilers including GCC, Clang, MSVC, Apple Clang, EDG eccp, Intel C++, etc. 

Even though we can gain more features if we upgrade to C++14, which is also capable deployed on our build machines, the motivation won't come until we gains more then pains. As for C++17 or even C++2a, there are still some remained tasks for compilers to do. For example, here are some left C++17 library features:

| C++17 feature | GCC libstdc++ | Clang libc++ | MSVC Standard Library |
| :--- | :---: | :---: | :---: |
| Hardware interference size | - | - | 19.11 |
| Elementary string conversions | 8 (no FP) | 7 (no FP) | 19.24 |
| std::shared_ptr and std::weak_ptr with array support | 7 | - | 19.12 |


### Thread-Safe Container

### Parallel Algorithms
// Thread Library: future, async, ...
// std::sort (ParallelSTL)

### // ...

### Summary
// Pros & Cons

********

## GCD
// Intro...

### Platform Support
// Performance & Stability on Windows!!!

### Thread-Safe Container

### Parallel Algorithms
// Basic Usage
// Demo: simply compare performance with TBB & STL

### // ...

### Summary
// Pros & Cons

********

## Reference
- [C++ compiler support](https://en.cppreference.com/w/cpp/compiler_support)
- [AMD vs Intel: Which is better for 2019 and beyond?](https://www.androidauthority.com/amd-vs-intel-994185/)
- [AMD vs Intel Market Share](https://www.cpubenchmark.net/market_share.html)
