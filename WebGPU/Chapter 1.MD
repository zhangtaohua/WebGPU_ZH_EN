WebGPU

W3C Working Draft, 2 February 2023

More details about this document

This version:

https://www.w3.org/TR/2023/WD-webgpu-20230202/

Latest published version:

https://www.w3.org/TR/webgpu/

Editor's Draft:

https://gpuweb.github.io/gpuweb/

History:

https://www.w3.org/standards/history/webgpu

---
---

# 1\. 介绍
> 1. Introduction

这一节是非规范性的。

> This section is non-normative.

[图像处理单元](https://en.wikipedia.org/wiki/Graphics_processing_unit)，简称GPU，已经成为人个电脑中用于渲染和计算应用不可或缺的硬件单元。WebGPU是一系列为Web应用暴露GPU硬件能力的API接口。这组AP从头开始设计，以便有效的映射到本地GPU的API接口（从2014年以后）， 同时，WebGPU不和[WebGL](https://www.khronos.org/webgl/)相关，也没有明确采用OpenGL ES的设计思想。

> Graphics Processing Units, or GPUs for short, have been essential in enabling rich rendering and computational applications in personal computing. WebGPU is an API that exposes the capabilities of GPU hardware for the Web. The API is designed from the ground up to efficiently map to (post-2014) native GPU APIs. WebGPU is not related to WebGL and does not explicitly target OpenGL ES.

WebGPU将物理GPU硬件单元视为[GPU适配器(GPUAdapters)](https://www.w3.org/TR/webgpu/#gpuadapter)，它通过管理资源的[GPUDevice](https://www.w3.org/TR/webgpu/#gpudevice) 和 执行指令的设备[GPUQueues](https://www.w3.org/TR/webgpu/#gpuqueue) 提供与适配器的连接。GPUDevice可能拥有自己的内存单元，用于高速的访问处理单元。[GPUBuffer](https://www.w3.org/TR/webgpu/#gpubuffer) 和 [GPUTexture](https://www.w3.org/TR/webgpu/#gputexture) 是由GPU内存支持的物理资源。[GPUCommandBuffer](https://www.w3.org/TR/webgpu/#gpucommandbuffer) 和 [GPURenderBundle](https://www.w3.org/TR/webgpu/#gpurenderbundle) 是用户记录指令的容器。 [GPUShaderModule](https://www.w3.org/TR/webgpu/#gpushadermodule)包含[着色器代码](https://www.w3.org/TR/webgpu/#shaders)。其他资源，诸如[GPUSampler](https://www.w3.org/TR/webgpu/#gpusampler) 和 [GPUBindGroup](https://www.w3.org/TR/webgpu/#gpubindgroup) 则配置了GPU如何使用[物理资源](https://www.w3.org/TR/webgpu/#physical-resources)的方式。

> WebGPU sees physical GPU hardware as GPUAdapters. It provides a connection to an adapter via GPUDevice, which manages resources, and the device’s GPUQueues, which execute commands. GPUDevice may have its own memory with high-speed access to the processing units. GPUBuffer and GPUTexture are the physical resources backed by GPU memory. GPUCommandBuffer and GPURenderBundle are containers for user-recorded commands. GPUShaderModule contains shader code. The other resources, such as GPUSampler or GPUBindGroup, configure the way physical resources are used by the GPU.

GPUs通过管道(pipeline)传输数据给[CPUCommandBuffers](https://www.w3.org/TR/webgpu/#gpucommandbuffer)中已编码的命令来执行，[管道(pipeline)](https://www.w3.org/TR/webgpu/#pipeline)是一组固定功能和可编程阶段的混各体。可编程阶段用于执行着色器，它是一系统专门设计用于在GPU硬件上运行的特殊程序。一个管道中大部分阶段都是由[GPURenderPipeline](https://www.w3.org/TR/webgpu/#gpurenderpipeline)或者 [GPUComputePipeline](https://www.w3.org/TR/webgpu/#gpucomputepipeline) 对象定义好的。这些阶段不包括管道对象中在编码过程中通过命令而进行的设置，例如调用 [beginRenderPass()](https://www.w3.org/TR/webgpu/#dom-gpucommandencoder-beginrenderpass) 或者 [setBlendCOnstant()](https://www.w3.org/TR/webgpu/#dom-gpurenderpassencoder-setblendconstant)函数等。

> GPUs execute commands encoded in GPUCommandBuffers by feeding data through a pipeline, which is a mix of fixed-function and programmable stages. Programmable stages execute shaders, which are special programs designed to run on GPU hardware. Most of the state of a pipeline is defined by a GPURenderPipeline or a GPUComputePipeline object. The state not included in these pipeline objects is set during encoding with commands, such as beginRenderPass() or setBlendConstant().

