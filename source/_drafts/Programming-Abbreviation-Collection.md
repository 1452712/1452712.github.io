---
title: Programming Abbreviation Collection
date: 2018-08-20 23:19:47
categories:
- Programming
tags:
updated:
mathjax: true
---

A collection of programming abbreviations.

<!-- more -->

## Abbreviation

| Abbreviation | Full Name | Related Link|
| :------: | :------ | :------ |
| ACES | Academic Color Encoding System | |
| AGP | Accelerated Graphics Port | |
| AWS | Amazon Web Service | |
| BSP | Binary Space Partition| [wiki](https://en.wikipedia.org/wiki/Binary_space_partitioning) |
| CB Test | Certification Body Test | [IECEE](https://www.iecee.org/certification/certificates/) |
| CICD | Continuous Integration & Continuous Delivery| [Part of Deployment Pipeline](https://en.wikipedia.org/wiki/Continuous_delivery); [CICD in the Wild](https://medium.com/@edzob/ci-and-cd-in-the-wild-b5ca8f71fa28) |
| COM | Components Object Model | |
| CSO | Compiled Shader Object | |
| CTAD | Class Template Argument Deduction | |
| DAG | Directed Acyclic Graph (graph theory) | [wiki](https://en.wikipedia.org/wiki/Directed_acyclic_graph) |
| DLL | Dynamic-Link Library | |
| DMA | Direct Memory Access | |
| DVI | Digital Visual Interface | |
| DXGI | DirectX Graphics Infrastructure | |
| FBX | Filmbox (a 3D Asset exchange format) | [Autodesk](https://www.autodesk.com/products/fbx/overview) |
| FMAC | Floating Multiply-ACcumulate | |
| GART | Graphics Address Remapping Table | |
| GDI | Windows Graphics Device Interface | |
| GUID | Global Unique Identifier; (**5 Types:** date-time & MAC address; DCE Security; MD5 hash & namespace; random; SHA-1 hash & namespace) | [quick guide](https://betterexplained.com/articles/the-quick-guide-to-guids/);  [msdn](https://msdn.microsoft.com/en-us/library/system.guid%28v=vs.110%29.aspx); [guid](http://guid.one/)|
| HDMI | High-Definition Multimedia Interface | |
| HDR | High Dynamic Range | HDR vs SDR |
| IOMMU | Input-Output Memory Management Unit | Refer to Picture 1 |
| IaaS | Infrastructure as a Service | e.g. [AWS](https://aws.amazon.com) |
| KPI | Key Performance Indicator | |
| MBPS | megabit per second | |
| MIPS | millions of instructions per second | |
| PCH | Platform Controller Hub | |
| PCIE | Peripheral Component Interconnect Express | Refer to Picture 2 |
| PaaS | Platform as a Service | e.g. [Google App Engine](https://cloud.google.com/appengine/) |
| RPC | Remote Procedure Call | [wiki](https://en.wikipedia.org/wiki/Remote_procedure_call) |
| RPM | Revolutions Per Minute | |
| SDR | Software-Defined Radio (v.s. HDR) | |
| SoC | System-on-a-Chip | |
| SWD | Serial Wire Debug | An electrical interface | |
| TBB | Threading Building Blocks (Intel) | |
| TD | Test Data | |
| TLS | Transport Layer Security | [wiki](https://en.wikipedia.org/wiki/Transport_Layer_Security); [IETF](https://tools.ietf.org/html/rfc8446) |
| UMD | User-Mode Graphics Driver | |
| VGA | Video Graphics Array connector (15-pin) | |
| WDDM | Windows Display Driver Model | [design guide](https://docs.microsoft.com/en-us/windows-hardware/drivers/display/windows-vista-display-driver-model-design-guide); [workflow](https://docs.microsoft.com/en-us/windows-hardware/drivers/display/windows-vista-and-later-display-driver-model-operation-flow) |
| WRL | Windows Runtime Library | [WRL](https://docs.microsoft.com/en-us/cpp/windows/windows-runtime-cpp-template-library-wrl); [WinRT](https://docs.microsoft.com/en-us/windows/uwp/cpp-and-winrt-apis/index)|
| XFB | Binary Device Interface File Format Header | [AXway File Broker](https://docs.axway.com/bundle/SecureTransport_536_AdministratorGuide_allOS_en_HTML5/page/Content/AdministratorsGuide/setup/c_st_aboutXFB_TO.htm) |
* * *

> [wiki: Glossary of computer graphics](https://en.wikipedia.org/wiki/Glossary_of_computer_graphics)

## Useful Tools

- QT: https://resources.qt.io/resources-by-content-type-videos-demos-tutorials
- Openframeworks: https://openframeworks.cc/
- Jenkins
- Visual Leak Detector
- Bullseye
- WinUSB
- [Unreal](https://www.unrealengine.com/en-US/ue4-on-github)
- Arnold

## Common "Slang" in Computer Graphics

- AO: Ambient Occlusion (A shading and rendering technique used to calculate how exposed each point in a scene is to ambient lighting.)
- AZDO: Approaching Zero Driver Overhead; [OpenGL Efficiency](https://www.khronos.org/assets/uploads/developers/library/2014-gdc/Khronos-OpenGL-Efficiency-GDC-Mar14.pdf)
- BSDF = BRDF + BTDF
  - BSDF: Bidirectional Scattering Distribution Function
  - BRDF: Bidirectional Reflectance Distribution Function
  - BTDF: Bidirectional Transmittance Distribution Function
  - Incident Light Beam => Specular Reflection + Specular Transmition + BSDF
- BSP: Binary Space Partitioning (A spatical data structure)
- BVH: Bounding Volume Hierarchy
- CSG: Constructive Solid Geometry
- DXR: DirectX Raytracing (A feature of Microsoft's DirectX that allows for HW real-time raytracing). [Announcement](https://blogs.msdn.microsoft.com/directx/2018/03/19/announcing-microsoft-directx-raytracing/).
- GDDR: Graphics Double Data Rate
- GI: Global Illumination
- GPGPU: General Purpose computing on GPUs.
  Performs non-specialized calculations which are typically be conducted by the CPU. Started from NVIDIA GeForce 3, CUDA allowed programmers to ignore the underlying graphical concepts in favor of more common high-performance computing concepts. Microsoft's DirectCompute and Apple/Khronos Group's OpenCL followed up later, which means that modern GPGPU pipelines can leverage the speed of a GPU without requiring full and explicit conversion of the data to a graphical form.
- GTI: Graphics Technology Interface.
- Mip: Multim in Parvo (Latin)(i.e. much in a small space)
- PBR: Physically Based Rendering (A rendering modes opposite to Classic mode)
- TBR/TBDR: Tiled Based Deferrd Architecture, The tech used to balance GPU power & performance on mobiles.
- TCB control: tension, continuity and bias control. A technique to tweak the shape of the curve.
- TIN: Triangular Irregular Network
- TLB: Translation Lookaside Buffer
- UV Mapping: Suface Parameterization/Texture Coordinate Fundation
- VFX: Visual Effects, including:
  - Special effects
    - Phsical effects
  - Digital effects (short to FX or digital FX)
    - Matte paintings and stills
    - Motion capture (Mo-Cap)
    - Modelling
    - Animation
    - Compositing
    - Simulation

## Appending Picture

### IOMMU

![IOMMU](/contents/images/Programming-Abbreviation-Collection/MMU_and_IOMMU.png)

### PCI Express

![PCI Express](/contents/images/Programming-Abbreviation-Collection/PCI_Express.png)