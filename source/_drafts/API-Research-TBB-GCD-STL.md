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

The following chart shows differences of required features between TBB, GCD & STL(C++11):

| Lib | Platform Support | Thread-Safe Container | Parallel Algorithms |
|:---:| ---:| ---:| ---:|
| TBB | Windows, Linux, macOS | YES | `tbb::parallel_do` |
| STL | Windows, Linux, macOS| NO (alternative: boost) | `stl::async` |
| GCD | macOS | NO | `dispatch_async` |

### Parallel Computing Applications in Graphics / Games

| Parallel Computing | Description | Application |
| :--- | :--- | :--- |
| Sparse Linear Algebra (e.g., SpMV, OSKI, or SuperLU) | Data sets include many zero values. Data is usually stored in compressed matrices to reduce the storage and bandwidth requirements to access all of the nonzero values. One example is block compressed sparse row (BCSR). Because of the compressed formats, data is generally accessed with indexed loads and stores. | Reverse kinematics; Spring models |
| Spectral Methods (e.g., FFT) | Data are in the frequency domain, as opposed to time or spatial domains. Typically, spectral methods use multiple butterfly stages, which combine multiply-add operations and a specific pattern of data permutation, with all-to-all communication for some stages and strictly local for others. | Texture maps |
| Structured Grids (e.g., Cactus or Lattice- Boltzmann Magneto-hydrodynamics) | Represented by a regular grid; points on grid are conceptually updated together. It has high spatial locality. Updates may be in place or between 2 versions of the grid. The grid may be subdivided into finer grids in areas of interest (“Adaptive Mesh Refinement”); and the transition between granularities may happen dynamically. |Smoothing; interpolation |
| Graph Traversal  (e.g., Quicksort) | Visits many nodes in a graph by following successive edges. These applications typically involve many levels of indirection, and a relatively small amount of computation. | Reverse kinematics, collision detection, depth sorting, hidden surface removal |
| Finite State Machine | A system whose behavior is defined by states, transitions defined by inputs and the current state, and events associated with transitions or states. | Response to collisions |

********

## STL

> The Standard Template Library (STL) is a software library for the C++ programming language that influenced many parts of the C++ Standard Library. It provides four components called *algorithms*, *containers*, *functions*, and *iterators*.

Main related Libs: **Thread support library**, **Atomic operations library**, **Algorithms library**.

### Compiler Support

C++11 are almost fully supported by all the main compilers including GCC, Clang, MSVC, Apple Clang, EDG eccp, Intel C++, etc. 

Even though we can gain more features if we upgrade to C++14, which is also capable deployed on our build machines, the motivation won't come until we gains more then pains. As for C++17 or even C++2a, there are still some remained tasks for compilers to do. For example, here are some left C++17 library features:

| C++17 feature | GCC libstdc++ | Clang libc++ | MSVC Standard Library |
| :--- | :---: | :---: | :---: |
| Hardware interference size | - | - | 19.11 |
| Elementary string conversions | 8 (no FP) | 7 (no FP) | 19.24 |
| std::shared_ptr and std::weak_ptr with array support | 7 | - | 19.12 |

### Thread-Safe Container

> Thread safety
>1. All container functions can be called concurrently by different threads on different containers. (i.e. it is safe to use two different std::vector instances on two different threads)
>2. All const member functions can be called concurrently by different threads on the same container.
>3. Different elements in the same container can be modified concurrently by different threads, except for the elements of `std::vector<bool>` (for example, a vector of `std::future` objects can be receiving values from multiple threads).
>4. Container operations that invalidate any iterators modify the container and cannot be executed concurrently with any operations on existing iterators even if those iterators are not invalidated.
>5. Elements of the same container can be modified concurrently with those member functions that are not specified to access these elements.
>In any case, container operations (as well as algorithms, or any other C++ standard library functions) may be parallelized internally as long as this does not change the user-visible results (e.g. `std::transform` may be parallelized, but not `std::for_each` which is specified to visit each element of a sequence in order)

Generally speaking, if we want to modify the container in multiple threads, we need to protect that container to prevent concurrent access. For this situation, boost provides a selection of [lock-free container alternatives](https://www.boost.org/doc/libs/1_55_0/doc/html/lockfree.html) which we can consider for multi-threaded use cases.

### Parallel Algorithms

For current multithreading usages, the _Thread support library_ offers support for threads, mutual exclusion, condition variables, and futures.

![STLThreadLevel](/contents/images/API-Research-TBB-GCD-STL/STL-Thread-Level.png)

> The function template `async` runs the function f asynchronously (potentially in a separate thread which may be part of a thread pool) and returns a `std::future` that will eventually hold the result of that function call.

Sample Code:

```cpp
    void DoSomething(char c) {
        default_random_engine dre(c);
        uniform_int_distribution<int> id(10, 1000);

        for (int i = 0; i <= 10; i++) {
            this_thread::sleep_for(chrono::milliseconds(id(dre)));
            cout.put(c).flush();
        }
    }

    int Main() {
        cout << "Starting 2 operations asynchronously" << endl;

        // MUST: pass by value / const ref no mutable
        future<void> f1 = async([] { DoSomething('.'); });
        future<void> f2 = async([] { DoSomething('+'); });
        // Another method:
        //auto f3 = async(launch::async, [] { DoSomething('_'); });

        if (f1.wait_for(chrono::seconds(0)) != future_status::deferred ||
            f2.wait_for(chrono::seconds(0)) != future_status::deferred) {

            while (f1.wait_for(chrono::seconds(0)) != future_status::ready &&
                f2.wait_for(chrono::seconds(0)) != future_status::ready) {
                // ......
                this_thread::yield();
            }
        }
        cout.put('\n').flush();

        try {
            f1.get();
            f2.get();
        }
        catch (const exception& e) {
            cout << "\nEXCEPTION: " << e.what() << endl;
        }
        cout << "\nDone" << endl;

        return 0;
    }
```

Further more, in C++17, Standard library implementations (but not the users) may define additional [execution policies](https://en.cppreference.com/w/cpp/algorithm/execution_policy_tag_t) as an extension. The semantics of parallel algorithms invoked with an execution policy object of implementation-defined type is implementation-defined.

Parallel version of algorithms (except for `std::for_each` and `std::for_each_n`) are allowed to make arbitrary copies of elements from ranges, as long as both `std::is_trivially_copy_constructible_v<T>` and `std::is_trivially_destructible_v<T>` are `true`, where `T` is the type of elements.

The optional execution policies are `std::execution::sequenced_policy` (usually specified as `std::execution::seq`), `std::execution::parallel_policy` (usually specified as `std::execution::par`), `std::execution::parallel_unsequenced_policy` (indicating that a parallel algorithm's execution may be parallelized, vectorized, or migrated across threads, such as by a parent-stealing scheduler; usually specified as `std::execution::par_unseq`), `std::execution::unsequenced_policy` (indicating that a parallel algorithm's execution may be vectorized; usually specified as `std::execution::unseq`; since c++20).

Following charts shows current [standard library algorithms for which parallelized versions are provided](https://en.cppreference.com/w/cpp/experimental/parallelism):

![ParallelizedAlgorithms](/contents/images/API-Research-TBB-GCD-STL/Parallelized-Algorithms.png)

Sample Code for sequenced policy & parallel policy:

```cpp
std::size_t THRESHOLD /* = some value */;

template <class ForwardIt>
void quicksort(ForwardIt first, ForwardIt last){
    if(first == last) return;
    std::size_t distance= std::distance(first,last);
    auto pivot = *std::next(first, distance/2);

    // Dynamic Execution Policy.
    std::parallel::execution_policy exec_pol= std::parallel::par; //default 
    if ( distance < THRESHOLD ) 
        exec_pol= std::parallel_execution::seq;

    ForwardIt middle1 = std::partition(exec_pol, first, last, 
                        [pivot](const auto& em){ return em < pivot; });
    ForwardIt middle2 = std::partition(exec_pol, middle1, last, 
                        [pivot](const auto& em){ return !(pivot < em); });
    quicksort(first, middle1);
    quicksort(middle2, last);
}
```

Sample Code for parallel policy & parallel unsequenced policy:

```cpp
std::vector<int> v = { 1, 2, 3 };

int Par(){
    int sum = 0;
    std::mutex m;
    std::for_each(std::execution::par, std::begin(v), std::end(v), [&](int i){
        // Ensure no data race.
        std::lock_guard<std::mutex> lock{m};
        sum += i*i;
    });
    return sum;
}

int ParUnseq(){
    std::atomic<int> sum{0};
    std::for_each(std::execution::par_unseq, std::begin(v), std::end(v), [&](int i){
        // Ensure no deadlock
        sum.fetch_add(i*i, std::memory_order_relaxed);
    });
    return sum;
}
```

### Summary

Pros:
1. Real Cross-platform with stable & continued support;
2. No further 3rd party;
3. Various-level functions to be adopted based on usages;
4. Exception mechanism.

Cons:
1. Too detailed & specific;
2. No support for thread-local cache.

********

## TBB

>The Intel® Threading Building Blocks (Intel® TBB) library provides software developers with a solution for enabling parallelism in C++ applications and libraries. The library includes a number of _generic parallel algorithms_, _concurrent containers_, support for dependency and data _flow graphs_, _thread local storage_, a work-stealing _task scheduler_ for task based programming, _synchronization primitives_, a scalable _memory allocator_ and more.

![TBB](/contents/images/API-Research-TBB-GCD-STL/TBB.png)

### Platform Support

| Category | Description |
| :--- | :--- |
| **Processors** | **Optimized for all compatible Intel® processors including Intel Atom®, Intel® Core™, Intel® Xeon®, and Intel® Xeon Phi™ processors.** |
| Language | C++ |
| Portability and Compatibility | Open source under an Apache* license <br> Compatible with multiple compilers (compiler agnostic) |
| Operating Systems	| Windows*, Linux*, macOS*, Android* additional with open source |

So basically, TBB can only benefit Intel CPUs. However, the latest market-share chart shows that, even though Intel still occupies more than 60% of the CPU market, AMD is increasing obviously:

![IntelAMDMarketShare](/contents/images/API-Research-TBB-GCD-STL/Intel-AMD-Market-Share.png)
<font size="2"> Site from <a href="https://www.cpubenchmark.net/market_share.html">AMD vs Intel Market Share</a> </font>

And we should notice the potentiality in their new products and techniques. Take the comparison between two recent parts as an example:

|  | Base speed	| Max speed	| TDP | Price |
| :--- | :---: | :---: | :---: | :---: |
| Ryzen 7 3800X	| 3.9GHz | 4.5GHz | 105 watts | $399 |
| Core i9-9900K | 3.6GHz | 5.0GHz | 95 watts | $488 |

> With Intel Turbo Boost Technology, the Intel chip can hit the 5.0GHz ceiling using two cores. The boost number drops to 4.8GHz using four cores and 4.7GHz using eight cores. Meanwhile, AMD’s Precision Boost 2 technology increases the speed of any number of cores . The increase is based on an analysis of the current environment involving thermal, electrical, and headroom utilization. It’s what AMD calls the “reliability triangle.”<br>
> That said, the AMD chip has a base speed advantage while the Intel chip has a higher turbo speed ceiling.

### Duplicated Modules in STL

Here are some duplicated functionalities often used in TBB.

| TBB | STL |
| :--- | :--- |
| tbb::atomic | std::atomic |
| tbb::tbb_thread | std::thread |
| tbb::mutex | std::mutex |
| tbb::recursive_mutex | std::recursive_mutex |
| tbb::reader_writer_lock | std::shared_mutex |
| tbb::hash & tbb:hasher | std::hash |

### Thread-Safe Container

TBB currently offers the following concurrent containers:
- Unordered associative containers
  - Unordered map (including unordered multimap)
  - Unordered set (including unordered multiset)
  - Hash table
- Queue (including bounded queue and priority queue)
- Vector

Comparison of concurrent unordered associative containers:

| Class Name | Concurrent traversal and insertion | Keys have a value associated with them | Support concurrent erasure | Built-in locking | No visible locking (lock-free interface) | Identical items allowed to be inserted | [] and `at` accessors |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| tbb::concurrent_hash_map | YES | YES | YES | YES | NO | NO | NO |
| tbb::concurrent_unordered_map | YES | YES | NO | NO | YES | NO | YES |
| tbb::concurrent_unordered_multimap | YES | YES | NO | NO | YES | YES | NO |
| tbb::concurrent_unordered_set | YES | NO | NO | NO | YES | NO | NO |
| tbb::concurrent_unordered_multiset | YES | NO | NO | NO | YES | YES | NO |

### Parallel Algorithms

Since `tbb::parallel_while` has been deprecated, using `tbb::parallel_do` instead for applying a body until no items left:

```cpp
class Body {
public:
    Body() {};

    typedef Cell* argument_type;

    void operator()(Cell* c, tbb::parallel_do_feeder<Cell*>& feeder) const {
        c->update();
        // Restore ref_count in preparation for subsequent traversal.
        c->ref_count = ArityOfOp[c->op];
        for(size_t k=0; k<c->successor.size(); ++k) {
            Cell* successor = c->successor[k];
            // ref_count is used for inter-task synchronization.
            // Correctness checking tools might not take this into account, and report
            // data races between different tasks, that are actually synchronized.
            if(0 == --(successor->ref_count)) {
                feeder.add( successor );
            }
        }
    }
};

void ParallelPreorderTraversal(const std::vector<Cell*>& root_set) {
    tbb::parallel_do(root_set.begin(), root_set.end(),Body());
}
```

Another choice seems to stream items through a series of filters by `tbb::parallel_pipeline`:

```cpp
void ParallelPipeline(int numTokens, std::ofstream &caseBeforeFile, std::ofstream &caseAfterFile){
    tbb::parallel_pipeline(
        numTokens,
        // Get Filter
        tbb::make_filter<void, CaseStringPtr>(
            tbb::filter::serial_in_order, // filter node
            [&](tbb::flow_control &fc) -> CaseStringPtr { // filter body
                CaseStringPtr sPtr = getCaseString(caseBeforeFile)
                if(!sPtr)
                    fc.stop();
                return s_ptr;
            }) & // concatenation operation
        // Change Case Filter
        tbb::make_filer<CaseStringPtr, CaseStringPtr>(
            tbb::filter::parallel, // filter node
            [](CaseStringPtr sPtr) -> CaseStringPtr { // filter body
                std::transform(sPtr->begin(), sPtr->end(), sPtr->begin(),
                    [](char c) -> char{
                        if(std::islower(c))
                            return std::toupper(c);
                        else if(std::isupper(c))
                            return std::tolower(c);
                        else
                            return c;
                    });
                return sPtr;
            }) & // concatenation operation
        // Writer Filter
        tbb::make_filter<CaseStringPtr, void>(
            tbb::filter::serial_in_order, // filter node
            [&](CaseStringPtr sPtr) -> void { // filter body
                writeCaseString(caseAfterFile, sPtr);
            })
    );
}
```

Overall, the key implementations (i.e. way to distribute tasks) are similar:

```cpp
// tbb/parallel_do.h
namespace internal {
    template<typename Body> class do_group_task;

    //......
    template<typename Iterator, typename Body, typename Item>
    void run_parallel_do( Iterator first, Iterator last, const Body& body
#if __TBB_TASK_GROUP_CONTEXT
        , task_group_context& context
#endif
        )
    {
        typedef do_task_iter<Iterator, Body, Item> root_iteration_task;
#if __TBB_TASK_GROUP_CONTEXT
        parallel_do_feeder_impl<Body, Item> feeder(context);
#else
        parallel_do_feeder_impl<Body, Item> feeder;
#endif
        feeder.my_body = &body;

        root_iteration_task &t = *new( feeder.my_barrier->allocate_child() ) root_iteration_task(first, last, feeder);

        feeder.my_barrier->set_ref_count(2);
        feeder.my_barrier->spawn_and_wait_for_all(t);
    }

    //......

    template<typename Iterator, typename Body, typename Item>
    class do_task_iter: public task
    {
        //......

        task* run_for_input_iterator() {
            typedef do_group_task_input<Body, Item> block_type;

            block_type& t = *new( allocate_additional_child_of(*my_feeder.my_barrier) ) block_type(my_feeder);
            size_t k=0;
            while( !(my_first == my_last) ) {
                // Move semantics are automatically used when supported by the iterator
                new (t.my_arg.begin() + k) Item(*my_first);
                ++my_first;
                if( ++k==block_type::max_arg_size ) {
                    /* static template<class Body, class Item> const size_t tbb::interface9::internal::do_group_task_input<...>::max_arg_size = 4UL */
                    if ( !(my_first == my_last) )
                        recycle_to_reexecute(); //! task to be rescheduled.
                    break;
                }
            }
            if( k==0 ) {
                destroy(t);
                return NULL;
            } else {
                t.my_size = k;
                return &t;
            }
        }

        //......
    }

    //......
}
```

So a serious problem is that we have dependencies between Nodes because of data structures (BSP-Tree) & usages (Callbacks, SwapAgent, MissingChanel, etc.)

### The Best Way to Control Thread Number

There are multiple ways to control the number of threads used for execution. However, in our situation, the best way might be using multiple arenas with different numbers of slots to influence where TBB places its worker threads:

```cpp
void Execution() {
    tbb::task_arena a;

    std::thread t([=]() {
        tbb::parallel_for(0, N, [](int) {
            //......
        });
    }); 

    a.execute([=]() { 
        tbb::parallel_for(0, N, [](int) {
            //......
        });
    });

    tbb::parallel_for(0, N, [](int) {
        //......
    });
    t.join()
}
```

The figure show on possible execution order:

![TaskArena](/contents/images/API-Research-TBB-GCD-STL/TaskArena.PNG)

### Flow Graph

Implementations multithreading of dependency graph and data flow algorithms:

```cpp
void FlowGraph() {
    // step 1: construct the graph
    tbb::flow::graph g;

    // step 2: make the nodes
    tbb::flow::function_nodes<int, std::string> myFirstNode(
        g,
        tbb::flow::unlimited,
        [](const int &in) -> std::string {
            std::cout << "first node received: " << in << std::endl;
            return std::to_string(in);
        }
    );

    tbb::flow::function_node<std::string> mySecondNode(
        g,
        tbb::flow:unlimited,
        [](const std::string &in) {
            std::cout<< "second node received: " << in << std::endl;
        }
    );

    // step 3: add edges
    tbb::flow::make_edge(myFirstNode, mySecondNode);

    // step 4: send messages
    myFirstNode.try_put(10);

    // step 5: wait for graph to complete
    g.wait_for_all();
}
```

Following shows the flow graph node types supported:

![FlowGraph](/contents/images/API-Research-TBB-GCD-STL/Flow-Graph.PNG)

### Task Based Programming

> These interfaces can be used when no generic algorithm is provided that exactly matches your needs. The high-level interface, class `task_group`, lets you easily create groups of potentially parallel tasks from functors or lambda expressions.  The low-level interface, class task, permits more detailed control, such as control over exception propagation and affinity, but is not as user-friendly.

It is particularly appropriate for algorithms in which a problem can be recursively divided into smaller subproblems following a tree-like decomposition. Take Fibonacci sequence as an example:

#### High-Level Approach: `parallel_invoke`

```cpp
long parallel_fib1(long n){
    if(n < 2){
        return n;
    }
    else{
        long x = y = 0;
        tbb::parallel_invoke([&]{x = parallel_fib(n - 1);},
                             [&]{y = parallel_fib(n - 2);});
        return x + y;
    }
}
```

#### Lower Approach: `task_group`

```cpp
long parallel_fib2(long n){
    if(n < CUTOFF){
        // Sequency running for better performance
        return fib(n);
    }
    else{
        long x = y = 0;
        tbb::task_group g;
        // Spawn 2 tasks
        g.run([&]{x = parallel_fib(n - 1);});
        g.run([&]{y = parallel_fib(n - 2);});

        // Wait for both tasks to complete
        g.wait();

        return x + y;
    }
}
```

#### Low-Level Approach: `task` (Task Blocking, Task Continuation & Task Recycling)

Recycle FibTask (FibTask(8, &sum) in the figure) as a child of FibCont:

![RecyclingFibTask](/contents/images/API-Research-TBB-GCD-STL/RecyclingFibTask.PNG)

Source Code:

```cpp
// Task Continuation
class FibCont : public tbb::task {
public:
    long* const sum;
    long x, y;

    FibCont(long *sum_): sum(sum_) {}

    tbb::task* execute(){
        *sum = x + y;
        return nullptr;
    }
};

// Task Blocking
class FibTask : public tbb::task {
public:
    long n;
    long *sum;

    FibTask(long n_, long *sum_) : n(n_), sum(sum_) {}

    // Override virtual function task::execute
    tbb::task* execute(){
        if(n < CUTOFF){
            *sum = fib(n);
            return nullptr;
        }
        else{
            FibCont &c = *new(allocate_continuation()) FibCont(sum);
            FibTask &b = *new(c.allocate_child()) FibTask(n - 2, &c.y);

            // Task Recycling
            recycle_as_child_of(c);
            this->n -= 1;
            this->sum = &c.x;

            // Set ref_count to "Two Children"
            c.set_ref_count(2);
            tbb:task::spawn(b);

            return this;
        }
    }
};

long parallel_fib3(long n){
    long sum = 0L;
    FibTask &a = *new(tbb::task::allocate_root()) FibTask(n, &sum);
    tbb::task::spawn_root_and_wait(a);
    return sum;
}
```

The generic templates for parallel algorithms in STL;
Is it somewhat similar to GCD???

### Summary

Pros:
  - Schedule thread work in the whole products across components;
  - Detailed (but uncontrollable) memory management.

Cons:
  - For general parallel algorithms, it is too general that hard to customize usages;
  - For Flow Graph or Task Based Programming, detailed and careful design are required;
  - Intel CPU only.

Remaining Tasks:
  - Demo to check Cache Miss Performance Impact.

********

## GCD

> Grand Central Dispatch (GCD or libdispatch) provides comprehensive support for concurrent code execution on multicore hardware.
> GCD, operating at the system level, can better accommodate the needs of all running applications, matching them to the available system resources in a balanced fashion.

### Platform Support

Originally, the GCD only supports Darwin officially. Around 2011, as these [mails](https://lists.macosforge.org/pipermail/libdispatch-dev/2011-April/000513.html) and [blogs](https://blog.quiscalusmexicanus.org/2011/04/25/grand-central-dispatch-for-win32-things-still-to-do/) showed, DrPizza tried to port it into Windows but failed. Legacy code is still available on [github](https://github.com/DrPizza/libdispatch).

Following are detailed information:

> libdispatch is currently available on all **Darwin** platforms.

Site from wikipedia:
> Darwin is an open-source Unix-like operating system first released by Apple Inc.

![UnixTimeline](/contents/images/API-Research-TBB-GCD-STL/Unix_timeline.en.svg)

We might be capable to use it by [Objective-C++](https://medium.com/@husain.amri/objective-c-deliver-us-from-swift-3a44d3ac00e7). Definitely, however, it will probably impact the performance a lot, let alone the heavy maintenance workload in the future.

A new version of GCD called [swift-corelibs-libdispatch](https://apple.github.io/swift-corelibs-libdispatch/) aims to make a modern version of libdispatch available on all other **Swift** platforms. Nevertheless, there are no official supporting neither for Windows nor the Unix. Non-official projects like [XDispatch](http://opensource.mlba-team.de/xdispatch/docs/current/index.html) or even personal projects like [swift-build](https://github.com/compnerd/swift-build) could involve potential risks. An implementation for [FreeBSD](https://wiki.freebsd.org/action/show/GrandCentralDispatch?action=show&redirect=GCD) didn't update since 2009.

### High-Level Usages

#### Parallelism with GCD

Sample Usage (parallel for-loop):
```swift
DispatchQueue.concurrentPerform(1000) { i in /* iteration i */ } 
dispatch_apply(DISPATCH_APPLY_AUTO, 1000, ^(size_t i){ /* iteration i */ })
```

To balance CPU core's work, we need to choose an appropriate iteration count (i.e. Dynamic Resource Availablility). For example, we assign 3 in the following situation, that is `DispatchQueue.concurrentPerform(3) { i in /* iteration i */ }`, where the bubble will waste CPU resources:
![IterationCount3](/contents/images/API-Research-TBB-GCD-STL/Iteration-Count-3.png)

A better choice might be 100:
![IterationCount100](/contents/images/API-Research-TBB-GCD-STL/Iteration-Count-100.png)

#### Concurrency for Independently Executed Tasks

![ContextSwitch](/contents/images/API-Research-TBB-GCD-STL/Context-Switch.png)

The context switch will result in:
- Repeatedly waiting for exclusive access to contended resources.
    Solution: Optimizing Lock Contention.
- Repeatedly switching between independent operations.
    Solution: One Queue Hierarchy per Subsystem.
    ![QueueHierarchy](/contents/images/API-Research-TBB-GCD-STL/Queue-Hierarchy.png)
- Repeatedly bouncing an operation between threads.
    Solution: Leveraging Ownership and Unified Identity.
    ![Leverage1](/contents/images/API-Research-TBB-GCD-STL/Leverage1.png)
    ![Leverage2](/contents/images/API-Research-TBB-GCD-STL/Leverage2.png)

### Core Ideas: Queue

`Queue`, intended for flow control, is list of Blocks and obey FIFO. The Blocks here are encapsulate units of work. Originally, Blocks are a language-level feature added to C, Objective-C and C++, which allow users to create distinct segments of code that can be passed around to methods or functions as if they were values. Here is an example for Blocks:

```objectivec
bool rev = arg;
__block int cnt = 0;

sort(array, ^(int a, int b){
    cnt++;
    if (rev)
        return b - a;
    else
        return a - b;
});

log("Count: %d", cnt);
```

2 Execution Mode:
- Serialization Queue
  - Serial queues execute blocks one at a time.
- Concurrency Queue
  - Execute multiple blocks at the same time;
  - May complete out of order;
  - Queues execute concurrently with respect to other queues;
  - Can be suspended and resumed like serial queues.

Types:
- Main queue (1 serial queue, application’s main thread, UI)
    `dispatch_get_main_queue();`
- Global dispatch queues (4 system-defined global concurrent queue: bg, low, default, high)
    `dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);`
- Customized serial / concurrent queue 
    `dispatch_queue_create(“com.company.app.task”, DISPATCH_QUEUE_SERIAL);`

This figure shows the architecture of queues and thread pool:

![GCDPool](/contents/images/API-Research-TBB-GCD-STL/gcd-pool.png)

### Basic Usage

- Submitting blocks to queues
    `dispatch_async(queue, ^{ /* Block */ });`
    `dispatch_sync(queue, ^{ /* Block */ });`
- Submitting blocks later (Recommend: Main queue)
    `dispatch_after(when, queue, ^{ /* Block */ });`
- Concurrently executing one block many times (data-level parallelism)
    `dispatch_apply(iterations, queue, ^(size_t i){ /* Block */ })`
- Suspending and resuming execution
    `dispatch_suspend(queue);`
    `dispatch_resume(queue);`
- Managing queue lifetime
    `dispatch_retain(queue);`
    `dispatch_release(queue);`
- Force-synchronizing by Barrier Queues (Concurrency Queue only)
    `dispatch_barrier_async(concurrent_queue, ^{ /* Barrier */ });`
    `dispatch_barrier_sync(concurrent_queue, ^{ /* Barrier */ });`

#### Use Global Dispatch Queue to Accelerate Loading:

```objectivec
    //......
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_HIGH, 0), ^{ // Dispatch block asynchronously
        UIImage *overlayImage = [self faceOverlayImageFromImage:_image];
        dispatch_async(dispatch_get_main_queue(), ^{ // Dispatch block to main thread
            [self fadeInNewImage:overlayImage]; // Update UI
        });
    });
    //......
```

#### Atomic Operation in Singletons:

```objectivec
+ (instancetype)sharedManager
{
    static PhotoManager *sharedPhotoManager = nil;

    // Ensure initialization once
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        sharedPhotoManager = [[PhotoManager alloc] init];
        sharedPhotoManager->_photosArray = [NSMutableArray array];
    });
    return sharedPhotoManager;
}
```

The logical figure shows the dispatch flow of this usage:

![DispatchOnce](/contents/images/API-Research-TBB-GCD-STL/Dispatch-Once.png)

#### Customized Queue for Readers-Writers Problem:

```objectivec
// create concurrent queue
dispatch_queue_t queue = dispatch_queue_create("com.gcdTest.queue", DISPATCH_QUEUE_CONCURRENT);

// read block
dispatch_async(queue, ^{
    array = [NSArray arrayWithArray:_photosArray];
});

// barrier block for write
dispatch_barrier_async(queue, ^{
    // write
    [_photosArray addObject:photo];

    // post a notification
    dispatch_async(dispatch_get_main_queue(), ^{
        [self postContentAddedNotification]; 
    });
});

// another read block
dispatch_async(queue, ^{
    //......
});
```

The logical figure shows the dispatch flow of this usage:

![BarrierBlock](/contents/images/API-Research-TBB-GCD-STL/Barrier-Block.png)

### Advanced Usage

#### Semaphore

Sample Code:

```objectivec
- (void)downloadImageURLWithString:(NSString *)URLString
{
    // Create semaphore with initialized value
    dispatch_semaphore_t semaphore = dispatch_semaphore_create(0);
 
    NSURL *url = [NSURL URLWithString:URLString];
    __unused Photo *photo = [[Photo alloc]
                             initwithURL:url
                             withCompletionBlock:^(UIImage *image, NSError *error) {
                                 if (error) {
                                     XCTFail(@"%@ failed. %@", URLString, error);
                                 }
 
                                 // Increment semaphore and post a notification
                                 dispatch_semaphore_signal(semaphore);
                             }];
 
    // Wait for semaphore
    dispatch_time_t timeoutTime = dispatch_time(DISPATCH_TIME_NOW, kDefaultTimeoutLengthInNanoSeconds);
    if (dispatch_semaphore_wait(semaphore, timeoutTime)) {
        XCTFail(@"%@ timed out", URLString);
    }
}
```

#### Dispatch Group (Recommend: Global Dispatch Queue)

Use `dispatch_group_enter` and `dispatch_group_leave` to notify or callback a group of blocks. Sample Code:

```objectivec
- (void)downloadPhotosWithCompletionBlock:(BatchPhotoDownloadingCompletionBlock)completionBlock
{
    __block NSError *error;
    dispatch_group_t downloadGroup = dispatch_group_create();
 
    dispatch_apply(3, dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_HIGH, 0), ^(size_t i) {
 
        NSURL *url;
        switch (i) {
            case 0:
                url = [NSURL URLWithString:kOverlyAttachedGirlfriendURLString];
                break;
            case 1:
                url = [NSURL URLWithString:kSuccessKidURLString];
                break;
            case 2:
                url = [NSURL URLWithString:kLotsOfFacesURLString];
                break;
            default:
                break;
        }
 
        dispatch_group_enter(downloadGroup); // Must invoke before dispatch_group_leave
        Photo *photo = [[Photo alloc] initwithURL:url
                              withCompletionBlock:^(UIImage *image, NSError *_error) {
                                  if (_error) {
                                      error = _error;
                                  }
                                  dispatch_group_leave(downloadGroup);
                              }];
 
        [[PhotoManager sharedManager] addPhoto:photo];
    });
 
    dispatch_group_notify(downloadGroup, dispatch_get_main_queue(), ^{
        if (completionBlock) {
            completionBlock(error);
        }
    });
}
```

#### Dispatch Source

Usually apply for timer. Sample Code:

```objectivec
- (void)viewDidLoad
{
    [super viewDidLoad];

    dispatch_queue_t queue = dispatch_get_main_queue(); 
    // Access Dispatch Source outside
    static dispatch_source_t source = nil;  
    // Avoid Retain Cycle leading to memory leaks
    __typeof(self) __weak weakSelf = self;

    // Ensure initialization once
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        source = dispatch_source_create(DISPATCH_SOURCE_TYPE_SIGNAL, SIGSTOP, 0, queue);    
        if (source)
        {
            // Catch the notification
            dispatch_source_set_event_handler(source, ^{
                // Logic Handler
                NSLog(@"Hi, I am: %@", weakSelf);
            });

            // Recover source from pause (default) to active
            dispatch_resume(source);
        }
    });
 
  //......
}
```

### Summary

Pros:
- Simplified design;
- Improve workload balance in lower level.

Cons:
- Limited platform support;
- Can be covered by tbb???

For platform support, one option is that abstracting and implementing a cross-platform version of GCD.

********

## Reference

### Overview

- Asanovic, K., Bodik, R., Catanzaro, B. C., Gebis, J. J., Husbands, P., Keutzer, K., ... & Yelick, K. A. (2006). _The landscape of parallel computing research: A view from berkeley._

### STL

- [C++ compiler support](https://en.cppreference.com/w/cpp/compiler_support)
- [The Parallelism TS Should be Standardized](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/p0024r2.html#algorithms.parallel.exec)
- [Using C++17 Parallel Algorithms for Better Performance](https://devblogs.microsoft.com/cppblog/using-c17-parallel-algorithms-for-better-performance/)
- Josuttis, N. M. (2012). _The C++ standard library: a tutorial and reference._ Addison-Wesley.
- Holzner, Steven (2001). _C++ : Black Book._ Scottsdale, Ariz.: Coriolis Group.

### TBB

- [AMD vs Intel: Which is better for 2019 and beyond?](https://www.androidauthority.com/amd-vs-intel-994185/)
- [intel/tbb/examples](https://github.com/intel/tbb/tree/tbb_2020/examples)
- [TBB Getting Started Tutorial](https://www.threadingbuildingblocks.org/intel-tbb-tutorial)
- Voss, M., Asenjo, R., & Reinders, J. (2019). _Pro TBB: C++ Parallel Programming with Threading Building Blocks._ Apress.

### GCD

- [Framework Dispatch (Objective-C)](https://developer.apple.com/documentation/dispatch?language=objc)
- [Mastering Grand Central Dispatch](https://developer.apple.com/videos/play/wwdc2011/210/)
- [Blocks and Grand Central Dispatch in Practice](https://developer.apple.com/videos/play/wwdc2011/308/)
- [Modernizing Grand Central Dispatch Usage](https://developer.apple.com/videos/play/wwdc2017/706/)
- [Apple Concurrency Programming Guide](https://developer.apple.com/library/archive/documentation/General/Conceptual/ConcurrencyProgrammingGuide/Introduction/Introduction.html#//apple_ref/doc/uid/TP40008091-CH1-SW1)
- [Apple Multithreading Programming Guide](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/Multithreading/Introduction/Introduction.html#//apple_ref/doc/uid/10000057i)
- [Working with Blocks](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/WorkingwithBlocks/WorkingwithBlocks.html)
- [Multithreading and Grand Central Dispatch on iOS for Beginners Tutorial](https://www.raywenderlich.com/3049-multithreading-and-grand-central-dispatch-on-ios-for-beginners-tutorial)
- [Grand Central Dispatch Tutorial for Swift 4: Part 1/2](https://www.raywenderlich.com/5370-grand-central-dispatch-tutorial-for-swift-4-part-1-2)
- [Grand Central Dispatch Tutorial for Swift 4: Part 2/2](https://www.raywenderlich.com/5371-grand-central-dispatch-tutorial-for-swift-4-part-2-2)
- [Cocoa_chen's articles on GCD](http://cocoa-chen.github.io/tags/#GCD)
- [Mike Ash's articles on Grand Central Dispatch](https://www.mikeash.com/pyblog/?tag=gcd)
