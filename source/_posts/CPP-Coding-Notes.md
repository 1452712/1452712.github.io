---
title: C++ Coding Notes
date: 2018-12-01 13:00:37
categories:
- Language
tags:
- C++
updated:
---
Some notes and issues I took from work.
<!-- more -->

## Define Functions with Macro

- Macro Definition Example

```cpp
// Windows SDK (winnt)
#ifndef DECLSPEC_NOTHROW
#if (_MSC_VER >= 1200) && !defined(MIDL_PASS)
#define DECLSPEC_NOTHROW   __declspec(nothrow)
#else
#define DECLSPEC_NOTHROW
#endif
#endif
// ......
#ifdef _68K_
#define STDMETHODCALLTYPE       __cdecl
#else
#define STDMETHODCALLTYPE       __stdcall
#endif

// Windows SDK (combaseapi)
#ifdef COM_STDMETHOD_CAN_THROW
#define COM_DECLSPEC_NOTHROW
#else
#define COM_DECLSPEC_NOTHROW DECLSPEC_NOTHROW
#endif
// ......
#define STDMETHOD(method)        virtual COM_DECLSPEC_NOTHROW HRESULT STDMETHODCALLTYPE method
```

- Usage Example

```cpp
STDMETHOD(get_Count)(_Out_ long* pcount)
{
    if (pcount == NULL)
        return E_POINTER;
    ATLASSUME(m_coll.size()<=LONG_MAX);

    *pcount = (long)m_coll.size();

    return S_OK;
}
```

## Operator Overloading

> https://en.cppreference.com/w/cpp/language/operators

```cpp
struct Sum
{
    int sum;
    Sum() : sum(0) { }
    void operator()(int n) { sum += n; }
};

Sum s = std::for_each(v.begin(), v.end(), Sum());
```

| Expression | As member function | As non-member function | Example |
|: ---- :|: ---- :|: ---- :|: ---- :|
| @a | (a).operator@ ( ) | operator@ (a) | `!std::cin calls std::cin.operator!()` |
| a@b | (a).operator@ (b) | operator@ (a, b) | `std::cout << 42 calls std::cout.operator<<(42)` |
| a=b | (a).operator= (b) | cannot be non-member | `std::string s; s = "abc"; calls s.operator=("abc")` |
| a(b...) | (a).operator()(b...) | cannot be non-member | `std::random_device r; auto n = r(); calls r.operator()()` |
| a[b] | (a).operator[](b) | cannot be non-member | `std::map<int, int> m; m[1] = 2; calls m.operator[](1)` |
| a-> | (a).operator-> ( ) | cannot be non-member | `auto p = std::make_unique<S>(); p->bar() calls p.operator->()` |
| a@ | (a).operator@ (0) | operator@ (a, 0) | `std::vector<int>::iterator i = v.begin(); i++ calls i.operator++(0)` |

In this table, @ is a placeholder representing all matching operators: all prefix operators in @a, all postfix operators other than -> in a@, all infix operators other than = in a@b

**Summplement:**

Within the namespace (e.g. in other member functions of the same class), we can use in this way;

```cpp
    this->operator()(item);
```

## Notice the differences among `size_t`, `UINT`, `UINT64`, etc.

```cpp
// Defined by compiler
#ifdef _WIN64
    typedef unsigned __int64 size_t;
    typedef __int64          ptrdiff_t;
    typedef __int64          intptr_t;
#else
    typedef unsigned int     size_t;
    typedef int              ptrdiff_t;
    typedef int              intptr_t;
#endif
```

Since `memcpy`, `memcpy_s`, `operator new`, `operator new[]` etc are defined as `size_t`, using `UINT64` will cause data lose error.

```cpp
// Defined by Windows SDK

// (basetsd.h)
// Type definitions for the basic sized types.
typedef signed char         INT8, *PINT8;
typedef signed short        INT16, *PINT16;
typedef signed int          INT32, *PINT32;
typedef signed __int64      INT64, *PINT64;
typedef unsigned char       UINT8, *PUINT8;
typedef unsigned short      UINT16, *PUINT16;
typedef unsigned int        UINT32, *PUINT32;
typedef unsigned __int64    UINT64, *PUINT64;

// (minwindef.h)
// Basic Windows Type Definitions for minwin partition
typedef int                 INT;
typedef unsigned int        UINT;
typedef unsigned int        *PUINT;
```

For more information of fundamental types of C++, refer to [msdn](https://msdn.microsoft.com/en-us/library/cc953fe1.aspx).

## _TODO:_ C++17 Features

> https://www.codingame.com/playgrounds/2205/7-features-of-c17-that-will-simplify-your-code/introduction

### Fold Expressions

```cpp
#include <iostream>
#include <string>
using namespace std;

template<typename ... Args> auto sum(Args ...args) [
    return (args + ...);
]

int main(){
    cout << (1 + 2 + 3 + 5 + 8) << endl;
    return 0;
}
```

| EXPRESSION | EXPANSION |
|:--------|:--------|
| (... op pack) | ((pack1 op pack2) op ...) op packN |
| (init op ... op pack) | (((init op pack1) op pack2) op ...) op packN |
| (pack op ...) | pack1 op (... op (packN-1 op packN)) |
| (pack op ... op init) | pack1 op (... op (packN-1 op (packN op init))) |

Default values of empty parameter packs: `&&`: `true`; `||`: `false`; `,`: `void()`.

e.g.

```cpp
template<typename ...Args>
void FoldPrint(Args&&... args){
    cout << ... << forward<Args>args << endl;
}

int main(){
    FoldPrint("Hello" , ", ", "C++", 17, "!");
    FoldPrint();

    return 0;
}
```

## tbb Multithread lock

```cpp
if(multiThreading)
{
    tbb::spin_mutex::scoped_lock lck(mutex);

    // ......
}
```

The lock will be released when it goes out of the bracket.

## P4V force sync removed files

```cmd
p4 diff -sd <path-to-resource>...|p4 -x- sync -f
```

## Other Tips

1. All the tabs in code should be replaced by spaces.
2. Must add comments and make them understanded.
3. For excetions, we can return `false`, `NULL`, etc. However, we should ensure that the whole program would not crash.
4. Notice some functions, especially related to assert, exceptions, etc. would be skipped under OPTIMIZE compile.
5. To debug bugs releated GPU, we must enable GPU Debug Layer. And for many times, `Removing Device` or `Device Hunging` is caused by incorrectly pipeline settings/bindings/etc.
6. To avoid pow computing, using exponents of a power of 2 (e.g. 2, 4, 8, ...), so that it can be implement by shifting.

## Mark some useful tools

- [Handle](https://docs.microsoft.com/en-us/sysinternals/downloads/handle)
- [Performance Monitor](https://www.windowscentral.com/how-use-performance-monitor-windows-10)
- [Code Graph](https://marketplace.visualstudio.com/items?itemName=YaobinOuyang.CodeAtlas)
- Pix
- Process Explorer
- SunLogin

## Using `.inl` files

> https://stackoverflow.com/questions/1208028/significance-of-a-inl-file-in-c

`.inl` files are never mandatory and have no special significance to the compiler. It's just a way of structuring your code that provides a hint to the humans that might read it.

I use .inl files in two cases:

- For definitions of inline functions.
- For definitions of function templates.

In both cases, I put the declarations of the functions in a header file, which is included by other files, then I #include the .inl file at the bottom of the header file.

It separates the interface from the implementation and makes the header file a little easier to read. If you care about the implementation details, you can open the .inl file and read it. If you don't, you don't have to.

## Signals and Slots

> An simple introduction could be found in the [blog by simmesimme](http://simmesimme.github.io/tutorials/2015/09/20/signal-slot).

A pattern pre-defined in [Qt](https://doc.qt.io/archives/qt-4.8/signalsandslots.html) which makes it easier to implement the Observer pattern. The class based on `QObject` can tell the outside world that its state has changed by emitting a signal, `valueChanged()`, and it has a slot which other objects can send signals to.

```cpp
#include <QObject>

class Counter : public QObject
{
    Q_OBJECT

public:
    Counter() { m_value = 0; }

    int value() const { return m_value; }

public slots:
    void setValue(int value);

signals:
    void valueChanged(int newValue);

private:
    int m_value;
};
```

**NOTICE:** Have no relationship with `std::signal`! 

## Memory Leak

1. VS Diagnostic Tool
   > [Analyze CPU and Memory while Debugging](https://devblogs.microsoft.com/visualstudio/analyze-cpu-memory-while-debugging/)
   > [Measure app performance in Visual Studio](https://docs.microsoft.com/en-us/visualstudio/profiling/index?view=vs-2019)
2. CRT Library
   > [Find memory leaks with the CRT library](https://docs.microsoft.com/en-us/visualstudio/debugger/finding-memory-leaks-using-the-crt-library?view=vs-2019)

Possible reasons:

- Multithread
- Mem Pool
- Temp resource
- Destructor

## Include Guards

**Preferred**: use `#pragma once` in header files instead of `#ifndef ...`.
Advantages: [wiki](https://en.wikipedia.org/wiki/Pragma_once). For more about pre-processor, refer to this [link](https://en.wikipedia.org/wiki/C_preprocessor).

