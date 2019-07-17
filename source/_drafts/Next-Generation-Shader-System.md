---
title: Next-Generation Shader System
categories:
- Graphics
tags:
- Rendering
updated:
---

///////////////////////////////////////////////////////////////
////////////////////////// ATTENTION //////////////////////////
///////////////////////// DO NOT POST /////////////////////////
///////////////////////// BACKUP ONLY /////////////////////////
///////////////////////////////////////////////////////////////

An open source engines' shader system survey to build up a new-generation effect system.

<!-- more -->

## Commons

1. 1 general shader language for all platforms;
2. Well defined compiler & optimizer (e.g. [glsl-optimizer](https://github.com/aras-p/glsl-optimizer));
3. Flexible link usage like Unity's Shader Graph;
4. Dynamic debugging tool;
5. Pre-compiled shader resource for performance.

## MiniEngine [DX12]

### Shader file

- In _AoRenderCS.hlsli_

  No matter what shader it is, all main functions are named in `main(...)`

  ```cs
  void main( uint3 Gid : SV_GroupID, uint GI : SV_GroupIndex, uint3 GTid : SV_GroupThreadID, uint3 DTid : SV_DispatchThreadID )
  {
      // ......

      // Fetch four depths and store them in LDS
  #ifdef INTERLEAVE_RESULT
      float4 depths = DepthTex.Gather(LinearBorderSampler, float3(QuadCenterUV, DTid.z));
  #else
      float4 depths = DepthTex.Gather(LinearBorderSampler, QuadCenterUV);
  #endif

      // ......
  }
  ```

- In _AORender1CS.hlsl_

  ```cs
  #define INTERLEAVE_RESULT
  #include "AoRenderCS.hlsli"
  ```

### Compilation & Loading

- Build Shader Effect in .vcxproj

  - Include shader files

    ```xml
    <FxCompile Include="Shaders\BufferCopyPS.hlsl">
      <ShaderType>Pixel</ShaderType>
    </FxCompile>
    ```

  - Settings in project properties.

    ![HLSLCompilerSettings](/contents/images/Next-Generation-Shader-System/HLSLCompilerSettings.PNG)

    Notice that it also defines the output variable names of compiled buffers in `Header Variable Name` as `g_p%(Filename)`.

  - Compile with [Effect-Compiler Tool](https://docs.microsoft.com/en-us/windows/desktop/direct3dtools/fxc)
  
    cmd actually generated:
    ```bash
    /Zi /E"main" /Vn"g_p%(Filename)" /cs"_5_0" /Fh"D:\Projects\DirectX-Graphics-Samples\MiniEngine\ModelViewer\..\Build_VS15\x64\Debug\Output\ModelViewer\CompiledShaders\%(Filename).h" /nologo 
    ```

- Load when initialize root signature
  
  ```cpp
  #include "CompiledShaders/BufferCopyPS.h"

  //......
  // Set into PSO Desc as pixel shader
  s_BlendUIPSO.SetPixelShader( g_pBufferCopyPS, sizeof(g_pBufferCopyPS) );
  ```

## Unreal [Cross-platform]

> https://docs.unrealengine.com/en-us/Programming/Rendering/ShaderDevelopment

### Material Shader

- Create material shader type:

  ```cpp
  class FLightFunctionVS : public FMaterialShader
  {
      DECLARE_SHADER_TYPE(FLightFunctionVS,Material);
      // ......
      // Compilation Settings
      // Parameter
      // Serializer
  }
  ```

  General compilation settings have been defined in `FMaterialShader`.

- Instantiate shader:

  ```cpp
  IMPLEMENT_MATERIAL_SHADER_TYPE(,FLightFunctionVS,TEXT("/Engine/Private/LightFunctionVertexShader.usf"),TEXT("Main"),SF_Vertex);
  ```

> HLSL Cross Compiler
> Compiles HLSL shader source code into a high-level intermediate representation, performs device-independent optimizations, and produces OpenGL Shading Language (GLSL) compatible source code. The library is largely based on the GLSL compiler from Mesa as an extension.

### Compiler

Create a compiler by themselves and process shader compilation as jobs;
Basic functions are defined in `ShaderCompiler.h`.

Main process:

```cpp
class FGlobalShaderTypeCompiler
{
public:
	/**
	* Enqueues compilation of a shader of this type.
	*/
	ENGINE_API static class FShaderCompileJob* BeginCompileShader(...);

	/**
	* Enqueues compilation of a shader pipeline of this type.
	*/
	ENGINE_API static void BeginCompileShaderPipeline(...);

	/** Either returns an equivalent existing shader of this type, or constructs a new instance. */
	static FShader* FinishCompileShader(...);
};
```

User can also define their own versions of compiler in `ShaderType`.

Generally speaking, it is still a string processor.

## Unity [Cross-platform]

> https://docs.unity3d.com/2019.1/Documentation/Manual/SL-ShadingLanguage.html

### Surface Shaders
  Unity define a declarative language called **"ShaderLab"** with their own keywords and grammar to simplify the organization of basic shaders, properties, textures, etc. Corresponding UI supports creating shaders without coding.

### Vertex and Fragment Shaders
  Embedding HLSL “snippets” in the shader text;
  Using "#pragma geometry name" to clarify shader name (no parameter);
  Setting parameters in `Property` block;
  1 Shader compiled into all supported renderers;
  Direct GLSL is supported;
  Internal extension file type (.cginc) (helper functions & built-in variables in these files);
  Defining preprocessor macros;
  Debugging using VS internal tools/Pix.

> Shader Compilers
>- Internally, different shader compilers are used for shader program compilation:
Windows & Microsoft platforms (DX11, DX12 and Xbox One) all use Microsoft’s HLSL compiler (currently d3dcompiler_47).
>- OpenGL Core, OpenGL ES 3, OpenGL ES 2.0 and Metal use Microsoft’s HLSL followed by bytecode translation into GLSL or Metal, using [HLSLcc](https://github.com/Unity-Technologies/HLSLcc).
>- OpenGL ES 2.0 can use source level translation via [hlsl2glslfork](https://github.com/aras-p/hlsl2glslfork) and [glsl optimizer](https://github.com/aras-p/glsl-optimizer). This is enabled by adding `#pragma prefer_hlsl2glsl gles`.
>- Other console platforms use their respective compilers (e.g. PSSL on PS4).
>- Surface Shaders use Cg 2.2 and [MojoShader](https://icculus.org/mojoshader/) for code generation analysis step.

## NVIDIA [NV Hardware required]

> https://github.com/nvpro-samples/shared_sources
> https://developer.nvidia.com/rtx/raytracing/dxr/DX12-Raytracing-tutorial-Part-2
`nv_helpers_dx12::CompileShaderLibrary(L"XXX.hlsl"):`

## Forge [Cross-platform]

> https://github.com/ConfettiFX/The-Forge/tree/master/Common_3/ThirdParty/OpenSource

## Further Topics

### Shader Model 6.0

> https://docs.microsoft.com/en-us/windows/desktop/direct3dhlsl/hlsl-shader-model-6-0-features-for-direct3d-12

**Wave-level Operation:**
Multithread GPU to benefit performance.

Potential Usage:

- [GPU Scalarization](https://flashypixels.wordpress.com/2018/11/10/intro-to-gpu-scalarization-part-1/)
- stream compaction,
- reductions,
- block transpose,
- bitonic sort or Fast Fourier Transforms (FFT),
- binning,
- stream de-duplication,
- ...

Requirement: DirectX Feature Level 12.0

### Programmable Shaders

> https://docs.microsoft.com/en-us/windows/desktop/direct3d12/direct3d-12-render-passes

**Benifit performance of renderer based on Tile-Based Deferred Rendering**

> https://docs.microsoft.com/en-us/windows/desktop/direct3d12/d3d12-graphics-reference-shader-reference

**Create & manage programmable shaders**

**Improve performance with Variable Rate Shading (VRS)**

> https://devblogs.microsoft.com/directx/variable-rate-shading-a-scalpel-in-a-world-of-sledgehammers/