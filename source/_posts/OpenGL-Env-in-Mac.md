---
title: OpenGL Env in Mac
date: 2018-07-15 01:06:55
categories:
- Graphics
tags:
- OpenGL
- macOS
updated:
---
> Refer to https://blog.csdn.net/sinat_14891561/article/details/74941333

## Basic Env:

- Platform: macOS High Sierra
- Xcode: 9.4.1
- OpenGL: 4.1

## Suggestion

Even though we can compile and install GLFW&GLEW manually, I still strongly recommend installation by _home-brew_, especially for those who have already installed brew.

## **Issue**

_Header Search Paths_ as well as _Library Search Paths_ in Search Paths (Buildings) should be declared specifically. `/usr/local/include/**` may cause ERRORs like `#include nested too deeply`, because other libraries might exist in this folder.

The solution is to use `usr/local/include/glad` instead. However, another ERROR will be caused in _glad.c_, indicating that _<glad/glad.h>_ cannot be found. Modify it into _<glad.h>_. _<KHR/khrplatform.h>_, _<GL/glew.h>_ and _<GLFW/glfw.h>_ are also the same.
