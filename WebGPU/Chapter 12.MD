# WebGPU

W3C Working Draft, 28 March 2023

More details about this document

This version:

https://www.w3.org/TR/2023/WD-webgpu-20230328/

---
---

# 12\. 命令缓冲区

命令缓冲区是 预先记录的GPU命令列表，可以提交给 GPUQueue 来执行。每条 GPU 命令都代表了要在 GPU 上执行的任务，如设置状态、绘图、复制资源等。

一个 GPUCommandBuffer 只能被提交一次，在一次提交之后它就会失效。要在多次提交中重复使用渲染命令，请使用 GPURenderBundle。

---

12\. Command Buffers

Command buffers are pre-recorded lists of GPU commands that can be submitted to a GPUQueue for execution. Each GPU command represents a task to be performed on the GPU, such as setting state, drawing, copying resources, etc.

A GPUCommandBuffer can only be submitted once, at which point it becomes invalid. To reuse rendering commands across multiple submissions, use GPURenderBundle.

---
---

## 12.1. GPUCommandBuffer

```

[Exposed=(Window, DedicatedWorker), SecureContext]
interface GPUCommandBuffer {
};

// GPUCommandBuffer 包括 GPUObjectBase；
GPUCommandBuffer includes GPUObjectBase;
```

GPUCommandBuffer 有以下内部插槽：

**[[command_list]]， 类型为： list<GPU command>。**     
表示 GPU 命令的列表，当这个命令缓冲区被提交时，这个列表将在队列时间线上执行。     

**[[renderState]]， 类型为： RenderState**        
正在执行的任何渲染通道命令所使用的当前状态，初始值为null。

### 12.1.1. 命令缓冲区的创建

dictionary GPUCommandBufferDescriptor : GPUObjectDescriptorBase {
};

---

12.1. GPUCommandBuffer

```
[Exposed=(Window, DedicatedWorker), SecureContext]
interface GPUCommandBuffer {
};
GPUCommandBuffer includes GPUObjectBase;
```

GPUCommandBuffer has the following internal slots:

**[[command_list]], of type list<GPU command>**         
A list of GPU commands to be executed on the Queue timeline when this command buffer is submitted.

**[[renderState]], of type RenderState**            
The current state used by any render pass commands being executed, initially null.

12.1.1. Command Buffer Creation

```
dictionary GPUCommandBufferDescriptor : GPUObjectDescriptorBase {
};
```
