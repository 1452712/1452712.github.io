---
title: 2019 GDC Collection
categories:
- Graphics
tags:
- Rendering
updated:
mathjax: true
---

Technologies taken from 2019 GDC from a programmer's perspective with some demos.

<!-- more -->

## Unity GDC Keynote

> https://unity.com/events/gdc-2019

// TODO:
// Explore details of these techniques
// Explore and compare with other technique released on GTC/GDC.
// Demo (try to use).
// DX12 RTR Samples. Compare.

//////////////// Note taken from Video ////////////////
MAU (Monthly Activate User)
Slides in dark: white letters in black background
////////////////////////////////////////////////////////

### Pipeline

- SRP (Scriptable Render Pipeline)
- Next-generation -- LWRP (LightWeight Render Pipeline)
  - **Shader Graph** (Environment effects & visuals?????)

    Nested Shader Sub-Graphs (*Customized*)

    [Shader Graph Resources](https://forum.unity.com/threads/shader-graph-resources.656809/)

  - Dynamic lights with particle effects
  - Full scene dynamic shadows
- HDRP (High Definition Render Pipeline)
  >Cinematic look based on integrated post-processing stack.
  - Real-time Ray Tracing
  
    Via NVIDIA RTX platform.
    Demo will be released on Github at April 4.

  - Lightmapper
    >https://blogs.unity3d.com/2019/05/20/gpu-lightmapper-a-technical-deep-dive

### DOTS (Data-Oriented Technology Stack)

> unity.com/dots

- Burst Compiler (C#)

  Highly optimized machine code.

- Demo: Megacity

  Runtime-ready Data Format

- Physics: Havok
  - [Demo](https://blogs.unity3d.com/2019/03/19/announcing-unity-and-havok-physics-for-dots/?utm_source=twitter&utm_medium=social&utm_campaign=engine-global-generalpromo-2019-03-19&utm_content=blog_havok)

### AR Foundation

> https://unity.com/solutions/mobile-ar

mass of entities.

### Possible New Features for NextGen Pipeline

1. RTR with new pipeline
2. Shader system (e.g. Shader Graph)
3. Vulkan with new Technique
4. VR & AR

### 2019.1 Release Info

1. Long-Term Support stream (LTS): for users who wish to continue to develop and ship their games/content and stay on a stable version for an extended period
