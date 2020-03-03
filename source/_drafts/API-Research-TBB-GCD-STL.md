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
| GCD | macOS | | |

Here are my personal suggestions:
- For current OGS:
  - Replace duplicated or deprecated tbb modules/APIs with STL;
  - Keep TBB's concurrency container;
  - Balance tbb operator work;
  - Seperate data processing with pure cache data-structure operations;
  - // TODO: Demo to check Cache Miss Performance Impact.

- For GSF:
  - For cross-platform compatibility, avoid using either GCD or TBB and limit within STL or boost; As for platform-specific optimization, we could consider WinRT or GCD.
  - Simplify current TBB parallel algorithm usages with STL.
  - Redesign OGS containers so that users can clearly choose the thread-safe or the non-thread-safe version.

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

### Thread-Safe Container

### Parallel Algorithms

A Sample Usage:
//...

Key Implementation: (Way to distribute tasks)
//...

So a serious problem is that we have dependencies between Nodes because of data structures (BSP-Tree) & usages (Callbacks, SwapAgent, MissingChanel, etc.)

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
Problem: Thread-local Cache
Optional Solution:
1. Refactor Cache Mechanism: Keep Useful Data into Cache and release all the thread-local cache when ending tasks.
// std::sort (ParallelSTL)

### // ...

### Summary
// Pros & Cons

********

## GCD
// Intro...
> Grand Central Dispatch (GCD or libdispatch) provides comprehensive support for concurrent code execution on multicore hardware. 

### Platform Support

Originally, the GCD only supports Darwin officially. Around 2011, as these [mails](https://lists.macosforge.org/pipermail/libdispatch-dev/2011-April/000513.html) and [blogs](https://blog.quiscalusmexicanus.org/2011/04/25/grand-central-dispatch-for-win32-things-still-to-do/) showed, DrPizza tried to port it into Windows but failed. Legacy code is still available on [github](https://github.com/DrPizza/libdispatch).

Following are detailed information:

> libdispatch is currently available on all **Darwin** platforms.

Site from wikipedia:
> Darwin is an open-source Unix-like operating system first released by Apple Inc.

![UnixTimeline](/contents/images/API-Research-TBB-GCD-STL/Unix_timeline.en.svg)

We might be capable to use it by [Objective-C++](https://medium.com/@husain.amri/objective-c-deliver-us-from-swift-3a44d3ac00e7). Definitely, however, it will probably impact the performance a lot, let alone the heavy maintenance workload in the future.

A new version of GCD called [swift-corelibs-libdispatch](https://apple.github.io/swift-corelibs-libdispatch/) aims to make a modern version of libdispatch available on all other **Swift** platforms. Nevertheless, there are no official supporting neither for Windows nor the Unix. Non-official projects like [XDispatch](http://opensource.mlba-team.de/xdispatch/docs/current/index.html) or even personal projects like [swift-build](https://github.com/compnerd/swift-build) could involve potential risks. An implementation for [FreeBSD](https://wiki.freebsd.org/action/show/GrandCentralDispatch?action=show&redirect=GCD) didn't update since 2009.

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
