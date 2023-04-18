# WebGPU

W3C Working Draft, 28 March 2023

More details about this document

This version:

https://www.w3.org/TR/2023/WD-webgpu-20230328/

---
---

# 13\.命令编码

## 13.1. GPUCommandsMixin
GPUCommandsMixin 定义了对命令进行编码的所有接口的共同状态，它没有任何方法。

```
interface mixin GPUCommandsMixin {
};
```

GPUCommandsMixin 为包含它的接口添加了以下内部槽：

[[state]], 类型为：  encoder state
表示编码器的当前状态，初始化为 "open"。

[[commands]]，类型为：list<GPU command>。
表示当包含这些命令的 GPUCommandBuffer 被提交时，将在队列时间线上执行的 GPU命令列表。

---

13\. Command Encoding

13.1. GPUCommandsMixin

GPUCommandsMixin defines state common to all interfaces which encode commands. It has no methods.

```
interface mixin GPUCommandsMixin {
};
```

GPUCommandsMixin adds the following internal slots to interfaces which include it:

**[[state]], of type encoder state**      
The current state of the encoder, initially set to "open".

**[[commands]], of type list<GPU command>**     
A list of GPU commands to be executed on the Queue timeline when a GPUCommandBuffer containing these commands is submitted.

---
---

编码器状态可以是以下之一：

**"open"**      
编码器可以对新的命令进行编码。

**"locked"**      
编码器不能使用，因为它被一个子编码器锁定：它是一个 GPUCommandEncoder，而GPURenderPassEncoder 或 GPUComputePassEncoder 处于活动状态。当管道执行结束时，该编码器会再次变成 "open" 状态。

在这种状态下发出的任何命令都会使编码器失效。

**"ended"**      
编码器已被结束，新的命令不能再被编码。

在此状态下发出的任何命令都会产生一个验证错误。

---

The encoder state may be one of the following:

**"open"**      
The encoder is available to encode new commands.

**"locked"**      
The encoder cannot be used, because it is locked by a child encoder: it is a GPUCommandEncoder, and a GPURenderPassEncoder or GPUComputePassEncoder is active. The encoder becomes "open" again when the pass is ended.

Any command issued in this state makes the encoder invalid.

**"ended"**     
The encoder has been ended and new commands can no longer be encoded.

Any command issued in this state will generate a validation error.

---
---

验证 GPUCommandsMixin 编码器的编码状态：
如果 encoder.[[state]] 是：

**"open"**      
则返回true。

**"locked"**      
使编码器无效，并返回 false。

**"ended"**      
产生一个验证错误，并返回 false。

要在 GPUCommandsMixin 编码器上排队等候命令，该编码器需发出GPU命令的命令执行步骤：
1. 将命令附加到 encoder.[[commands]]。
2. 当命令被作为 GPUCommandBuffer 的一部分被执行时：
    1. 发布命令的执行步骤。

---

To Validate the encoder state of GPUCommandsMixin encoder:
If encoder.[[state]] is:

**"open"**      
Return true.

**"locked"**      
Make encoder invalid, and return false.

**"ended"**   
Generate a validation error, and return false.

To Enqueue a command on GPUCommandsMixin encoder which issues the steps of a GPU Command command:
1. Append command to encoder.[[commands]].
2. When command is executed as part of a GPUCommandBuffer:
    1. Issue the steps of command.

---
---
### 13.2. GPU命令编码器

```
[Exposed=(Window, DedicatedWorker), SecureContext]
interface GPUCommandEncoder {
    GPURenderPassEncoder beginRenderPass(GPURenderPassDescriptor descriptor);
    GPUComputePassEncoder beginComputePass(optional GPUComputePassDescriptor descriptor = {});

    undefined copyBufferToBuffer(
        GPUBuffer source,
        GPUSize64 sourceOffset,
        GPUBuffer destination,
        GPUSize64 destinationOffset,
        GPUSize64 size);

    undefined copyBufferToTexture(
        GPUImageCopyBuffer source,
        GPUImageCopyTexture destination,
        GPUExtent3D copySize);

    undefined copyTextureToBuffer(
        GPUImageCopyTexture source,
        GPUImageCopyBuffer destination,
        GPUExtent3D copySize);

    undefined copyTextureToTexture(
        GPUImageCopyTexture source,
        GPUImageCopyTexture destination,
        GPUExtent3D copySize);

    undefined clearBuffer(
        GPUBuffer buffer,
        optional GPUSize64 offset = 0,
        optional GPUSize64 size);

    undefined writeTimestamp(GPUQuerySet querySet, GPUSize32 queryIndex);

    undefined resolveQuerySet(
        GPUQuerySet querySet,
        GPUSize32 firstQuery,
        GPUSize32 queryCount,
        GPUBuffer destination,
        GPUSize64 destinationOffset);

    GPUCommandBuffer finish(optional GPUCommandBufferDescriptor descriptor = {});
};
GPUCommandEncoder includes GPUObjectBase;
GPUCommandEncoder includes GPUCommandsMixin;
GPUCommandEncoder includes GPUDebugCommandsMixin;
```

---

13.2. GPUCommandEncoder

```
[Exposed=(Window, DedicatedWorker), SecureContext]
interface GPUCommandEncoder {
    GPURenderPassEncoder beginRenderPass(GPURenderPassDescriptor descriptor);
    GPUComputePassEncoder beginComputePass(optional GPUComputePassDescriptor descriptor = {});

    undefined copyBufferToBuffer(
        GPUBuffer source,
        GPUSize64 sourceOffset,
        GPUBuffer destination,
        GPUSize64 destinationOffset,
        GPUSize64 size);

    undefined copyBufferToTexture(
        GPUImageCopyBuffer source,
        GPUImageCopyTexture destination,
        GPUExtent3D copySize);

    undefined copyTextureToBuffer(
        GPUImageCopyTexture source,
        GPUImageCopyBuffer destination,
        GPUExtent3D copySize);

    undefined copyTextureToTexture(
        GPUImageCopyTexture source,
        GPUImageCopyTexture destination,
        GPUExtent3D copySize);

    undefined clearBuffer(
        GPUBuffer buffer,
        optional GPUSize64 offset = 0,
        optional GPUSize64 size);

    undefined writeTimestamp(GPUQuerySet querySet, GPUSize32 queryIndex);

    undefined resolveQuerySet(
        GPUQuerySet querySet,
        GPUSize32 firstQuery,
        GPUSize32 queryCount,
        GPUBuffer destination,
        GPUSize64 destinationOffset);

    GPUCommandBuffer finish(optional GPUCommandBufferDescriptor descriptor = {});
};
GPUCommandEncoder includes GPUObjectBase;
GPUCommandEncoder includes GPUCommandsMixin;
GPUCommandEncoder includes GPUDebugCommandsMixin;
```

---
---

### 13.2.1. 命令编码器的创建

```
dictionary GPUCommandEncoderDescriptor : GPUObjectDescriptorBase {
};
```

**createCommandEncoder(descriptor)**

用于创建一个 GPUCommandEncoder。

调用于：GPUDevice 实例自身。

参数：

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| `descriptor` | `GPUCommandEncoderDescriptor` | ✘ | ✔ | 要创建的 GPUCommandEncoder的描述符。 |

返回：GPUCommandEncoder

---

13.2.1. Command Encoder Creation

```
dictionary GPUCommandEncoderDescriptor : GPUObjectDescriptorBase {
};
```

**createCommandEncoder(descriptor)**    
Creates a GPUCommandEncoder.

**Called on:** GPUDevice this.

**Arguments:**    

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| `descriptor` | `GPUCommandEncoderDescriptor` | ✘ | ✔ | Description of the GPUCommandEncoder to create. |

**Returns:** GPUCommandEncoder

---
---

**上下文时间线上步骤：**    
1. 让 e 是一个新的 GPUCommandEncoder 对象。
2. 在实例的设备时间线上发起初始化步骤。
3. 返回 e。

**设备时间轴初始化步骤：**  
1. 如果以下任何条件不满足，产生一个验证错误，使 e 无效，并停止。
    * 实例的this指针要是有效的。

>> **ISSUE 20** 描述剩余的 createCommandEncoder() 验证和算法步骤。

```
// 例子 25
// 创建一个 GPUCommandEncoder，对一个命令进行编码以便清除一个缓冲区，结束编码后得到一个GPUCommandBuffer，然后提交给 GPUQueue。
// EXAMPLE 25
// Creating a GPUCommandEncoder, encoding a command to clear a buffer, finishing the encoder to get a GPUCommandBuffer, then submitting it to the GPUQueue.

const commandEncoder = gpuDevice.createCommandEncoder();
commandEncoder.clearBuffer(buffer);
const commandBuffer = commandEncoder.finish();
gpuDevice.queue.submit([commandBuffer]);

```

---

**Content timeline steps:**   
1. Let e be a new GPUCommandEncoder object.
2. Issue the initialization steps on the Device timeline of this.
3. Return e.

**Device timeline initialization steps:**   
1. If any of the following conditions are unsatisfied generate a validation error, make e invalid, and stop.
    * this is valid.

>> **ISSUE 20** Describe remaining createCommandEncoder() validation and algorithm steps.


```
// EXAMPLE 25
// Creating a GPUCommandEncoder, encoding a command to clear a buffer, finishing the encoder to get a GPUCommandBuffer, then submitting it to the GPUQueue.

const commandEncoder = gpuDevice.createCommandEncoder();
commandEncoder.clearBuffer(buffer);
const commandBuffer = commandEncoder.finish();
gpuDevice.queue.submit([commandBuffer]);

```

---
---

## 13.3. 通行证编码

**beginRenderPass(descriptor)**     
开始对描述符所描述的渲染通道进行编码。

调用于：GPUCommandEncoder 实例自身

参数：

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| `descriptor` | `GPURenderPassDescriptor` | ✘ | ✘ | 要创建的 GPURenderPassEncoder 的描述符。 |

返回：GPURenderPassEncoder

---
13.3. Pass Encoding

**beginRenderPass(descriptor)**   
Begins encoding a render pass described by descriptor.

**Called on:** GPUCommandEncoder this.

**Arguments:**

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| `descriptor` | `GPURenderPassDescriptor` | ✘ | ✘ | Description of the GPURenderPassEncoder to create. |

**Returns:** GPURenderPassEncoder

---
---


**上下文时间线步骤：**        
1. 对于 descriptor.colorAttachments 中每个非空的 colorAttachment：
    1. 如果 colorAttachment.clearValue 不是为null。
        1. ? 验证 GPUColor shape（colorAttachment.clearValue）。
2. 让 pass 成为一个新的GPURenderPassEncoder对象。
3. 在实例的设备时间线上发起初始化步骤。
4. 返回 pass。

---

**Content timeline steps:**
1. For each non-null colorAttachment in descriptor.colorAttachments:
    1. If colorAttachment.clearValue is not null.
        1. ? validate GPUColor shape(colorAttachment.clearValue).
2. Let pass be a new GPURenderPassEncoder object.
3. Issue the initialization steps on the Device timeline of this.
4. Return pass.

---
---

**设备时间轴初始化步骤：**      
1. 验证这个的编码器状态。如果它返回false，使 pass 变得无效并返回。
2. 将this.[[state]] 设置为 "locked"。
3. 如果不满足以下任何要求，则使 pass 无效并返回。
    * 描述符必须满足由设备的this.[[device]] 给出的有效使用规则。
4. 对于 descriptor.colorAttachments 中每个非空的 colorAttachment：
    1. colorAttachment.view所访问的纹理子资源被认为是在渲染过程中作为附件使用的。
5. 让 depthStencilAttachment 成为 descriptor.depthStencilAttachment，如果没有提供，则为 null。
6. 如果 depthStencilAttachment 不是空的：
    1. 让 depthStencilView 成为 depthStencilAttachment.view。
    2. 如果 depthStencilAttachment.depthReadOnly 和 stencilReadOnly 为真：
        1. depthStencilView 的子资源 就会被认为是在渲染过程中作为附件只读使用的。
    3. 否则，depthStencilView 所访问的纹理子资源就会被认为是在渲染过程中作为附件使用的。
    4. 设置 pass.[[depthReadOnly]] 为 depthStencilAttachment.depthReadOnly。
    6. 设置 pass.[[stencilReadOnly]] 为 depthStencilAttachment.stencilReadOnly。
7. 设置 pass.[[layout]] 为 从 pass(descriptor) 获得的渲染目标布局。
8. 对于 descriptor.timestampWrites 中的每个 timestampWrite：
    1. 如果 timestampWrite.location 是 "beginning"，则在 this.[[commands]] 中附加一条GPU命令，此命令将 GPU的时间戳值写入 timestampWrite.querySet 中 timestampWrite.queryIndexth索引值所对应位置处。
    2. 否则，如果timestampWrite.location 是 "end"，将 timestampWrite 追加到pass.[[endTimestampWrites]]。
9. 设置 pass.[[drawCount]] 为 0。
10. 设置 pass.[[maxDrawCount]] 为 descriptor.maxDrawCount。
11. 在此实例的队列中入队一个命令，当此命令在队列时间线上执行时，将发起后续步骤。

---

**Device timeline initialization steps:**   
1. Validate the encoder state of this. If it returns false, make pass invalid and return.
2. Set this.[[state]] to "locked".
3. If any of the following requirements are unmet, make pass invalid and return.
    * descriptor must meet the Valid Usage rules given device this.[[device]].
4. For each non-null colorAttachment in descriptor.colorAttachments:
    1. The texture subresource seen by colorAttachment.view is considered to be used as attachment for the duration of the render pass.
5. Let depthStencilAttachment be descriptor.depthStencilAttachment, or null if not provided.
6. If depthStencilAttachment is not null:
    1. Let depthStencilView be depthStencilAttachment.view.
    2. If depthStencilAttachment.depthReadOnly and stencilReadOnly are true:
        1. The subresources of depthStencilView are considered to be used as attachment-read for the duration of the render pass.
    3. Else, the texture subresource seen by depthStencilView is considered to be used as attachment for the duration of the render pass.
    4. Set pass.[[depthReadOnly]] to depthStencilAttachment.depthReadOnly.
    5. Set pass.[[stencilReadOnly]] to depthStencilAttachment.stencilReadOnly.
7. Set pass.[[layout]] to derive render targets layout from pass(descriptor).
8. For each timestampWrite in descriptor.timestampWrites,
    1. If timestampWrite.location is "beginning", append a GPU command to this.[[commands]] that writes the GPU’s timestamp value into the timestampWrite.queryIndexth index in timestampWrite.querySet.
    2. Otherwise, if timestampWrite.location is "end", Append timestampWrite to pass.[[endTimestampWrites]].
9. Set pass.[[drawCount]] to 0.
10. Set pass.[[maxDrawCount]] to descriptor.maxDrawCount.
11. Enqueue a command on this which issues the subsequent steps on the Queue timeline when executed.

---
---

**队列时间轴步骤：**        
1. 让当前执行的 GPUCommandBuffer 的[[renderState]] 成为一个新的 RenderState。
2. **ISSUE 21** 执行附件 加载/清除 操作。

**ISSUE 22** 定义只读 深度/模板的行为

---

**Queue timeline steps:**
1. Let the [[renderState]] of the currently executing GPUCommandBuffer be a new RenderState.
2. **ISSUE 21**  Perform attachment loads/clears.

>> **ISSUE 22** specify the behavior of read-only depth/stencil


---
---

**beginComputePass(descriptor)**        

开始对描述符所描述的计算通道进行编码。

调用于：GPUCommandEncoder 实例自身。

**参数：**  

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| `descriptor` | `GPUComputePassDescriptor` | ✘ | ✔ |  |

返回：GPUComputePassEncoder

**上下文时间线步骤：**         
1. 让 pass 成为一个新的 GPUComputePassEncoder 对象。
2. 在实例的设备时间线上发起初始化步骤。
3. 返回 pass。

---

**beginComputePass(descriptor)**      

Begins encoding a compute pass described by descriptor.

**Called on:** GPUCommandEncoder this.

**Arguments:**

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| `descriptor` | `GPUComputePassDescriptor` | ✘ | ✔ |  |

**Returns:** GPUComputePassEncoder

**Content timeline steps:**
1. Let pass be a new GPUComputePassEncoder object.
2. Issue the initialization steps on the Device timeline of this.
3. Return pass.

---
---

**设备时间轴初始化步骤：**      
1. 验证实例的编码器状态。如果它返回false，使 pass 无效并返回。
2. 将 this.[[state]] 设置为 "locked"。
3. 如果不满足以下任何要求，则使 pass 无效并返回。
    * 验证 timestampWrites(this.[[device]], descriptor.timestampWrites) 函数必须返回true。
4. 对于 descriptor.timestampWrites 中的每个 timestampWrite: 
    1. 如果 timestampWrite.location 是 "beginning"，则在 this.[[commands]] 中附加一条GPU命令，此命令将 GPU的时间戳值写入 timestampWrite.querySet 中 timestampWrite.queryIndexth索引值所对应位置处。
    2. 否则，如果timestampWrite.location 是 "end"，将 timestampWrite 追加到pass.[[endTimestampWrites]]。

---

**Device timeline initialization steps:**       
1. Validate the encoder state of this. If it returns false, make pass invalid and return.
2. Set this.[[state]] to "locked".
3. If any of the following requirements are unmet, make pass invalid and return.
    * Validate timestampWrites(this.[[device]], descriptor.timestampWrites) must return true.
4. For each timestampWrite in descriptor.timestampWrites,
    1. If timestampWrite.location is "beginning", append a GPU command to this.[[commands]] that writes the GPU’s timestamp value into the timestampWrite.queryIndexth index in timestampWrite.querySet.
    2. Otherwise, if timestampWrite.location is "end", Append timestampWrite to pass.[[endTimestampWrites]].


---
---

## 13.4. 缓冲区复制命令

**copyBufferToBuffer(source, sourceOffset, destination, destinationOffset, size)**

将一条命令编码到 GPUCommandEncoder 中，其作用是将数据从一个 GPUBuffer 的子区域复制到另一个 GPUBuffer 的子区域。

调用于：GPUCommandEncoder 实例自身。

参数：

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| `source` | `GPUBuffer` | ✘ | ✘ | 要复制的GPUBuffer。 |
| `sourceOffset` | `GPUSize64` | ✘ | ✘ | 从源buffer中开始复制的字节偏移量。 |
| `destination` | `GPUBuffer` | ✘ | ✘ | 要复制到的GPUBuffer。 |
| `destinationOffset` | `GPUSize64` | ✘ | ✘ | 目的地的字节偏移量，用于放置复制的数据。 |
| `size` | `GPUSize64` | ✘ | ✘ | 要复制的字节数。 |

返回： undefined

**上下文时间线步骤：**
1. 在this.[[device]]的设备时间轴上发起后续步骤。

---

13.4. Buffer Copy Commands

**copyBufferToBuffer(source, sourceOffset, destination, destinationOffset, size)**      
Encode a command into the GPUCommandEncoder that copies data from a sub-region of a GPUBuffer to a sub-region of another GPUBuffer.

**Called on:** GPUCommandEncoder this.

**Arguments:**

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| `source` | `GPUBuffer` | ✘ | ✘ | The GPUBuffer to copy from. |
| `sourceOffset` | `GPUSize64` | ✘ | ✘ | Offset in bytes into source to begin copying from. |
| `destination` | `GPUBuffer` | ✘ | ✘ | The GPUBuffer to copy to. |
| `destinationOffset` | `GPUSize64` | ✘ | ✘ | Offset in bytes into destination to place the copied data. |
| `size` | `GPUSize64` | ✘ | ✘ | Bytes to copy. |

**Returns:** undefined

**Content timeline steps:**
1. Issue the subsequent steps on the Device timeline of this.[[device]].

---
---

**设备时间轴步骤：**
1. 验证实例自身的编码器的状态，如果它返回false，则停止。
2. 如果以下任何条件不满足，使之无效并停止。
    * 实例的this 指向的源地址是有效。
    * 实例的this 指向的目的地是有效的。
    * source.usage 包含 COPY_SRC。
    * destination.usage 包含 COPY_DST。
    * size 是 4 的倍数。
    * sourceOffset 是 4 的倍数。
    * destinationOffset 是 4 的倍数。
    * source.size ≥（sourceOffset+size）。
    * destination.size ≥（destinationOffset+size）。
    * source 和 destination 不是同一个 GPUBuffer。
3. 在此实例的队列中入队一个命令，当此命令在队列时间线上执行时，将发起后续步骤。

**队列时间轴步骤：**
1. 从 sourceOffset 开始，将源数据的 size 大小的字节数 复制到目的地中从 destinationOffset 开始的位置。

---

**Device timeline steps:**      
1. Validate the encoder state of this. If it returns false, stop.
2. If any of the following conditions are unsatisfied make this invalid and stop.
    * source is valid to use with this.
    * destination is valid to use with this.
    * source.usage contains COPY_SRC.
    * destination.usage contains COPY_DST.
    * size is a multiple of 4.
    * sourceOffset is a multiple of 4.
    * destinationOffset is a multiple of 4.
    * source.size ≥ (sourceOffset + size).
    * destination.size ≥ (destinationOffset + size).
    * source and destination are not the same GPUBuffer.
3. Enqueue a command on this which issues the subsequent steps on the Queue timeline when executed.

**Queue timeline steps:**
1. Copy size bytes of source, beginning at sourceOffset, into destination, beginning at destinationOffset.

---
---

**clearBuffer(buffer, offset, size)**

将一个命令编码到 GPUCommandEncoder 中，作用是用零填充 GPUBuffer 的一个子区域。

调用于：GPUCommandEncoder 实例自身。

参数：

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| `buffer` | `GPUBuffer` | ✘ | ✘ | 要清除的GPUBuffer。 |
| `offset` | `GPUSize64` | ✘ | ✔ | 要清除的缓冲区中子区域的字节偏移量。 |
| `size` | `GPUSize64` | ✘ | ✔ | 要清除的子区域的字节大小数。默认为缓冲区的大小减去偏移量。 |

返回：undefined

**上下文时间线步骤：**
1. 在this.[[device]]的设备时间轴上发起后续步骤。

---

**clearBuffer(buffer, offset, size)**     
Encode a command into the GPUCommandEncoder that fills a sub-region of a GPUBuffer with zeros.

**Called on:** GPUCommandEncoder this.

**Arguments:**

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| `buffer` | `GPUBuffer` | ✘ | ✘ | The GPUBuffer to clear. |
| `offset` | `GPUSize64` | ✘ | ✔ | Offset in bytes into buffer where the sub-region to clear begins. |
| `size` | `GPUSize64` | ✘ | ✔ | Size in bytes of the sub-region to clear. Defaults to the size of the buffer minus offset. |

**Returns:** undefined

**Content timeline steps:**     
1. Issue the subsequent steps on the Device timeline of this.[[device]].

---
---

**设备时间轴步骤：**
1. 验证实例自身的编码器的状态。如果它返回false，则停止。
2. 如果大小size 缺失，将大小设置为 max(0, |buffer|.{{GPUBuffer/size}}- |offset|)。
3. 如果以下任何条件未能得到满足，则使实例自身无效并停止。
    * 实例自身 this 的指向的 buffer是有效的。
    * buffer.usage 包含 COPY_DST。
    * size 是 4 的倍数。
    * offset 是 4 的倍数。
    * buffer.size ≥（offset+size）。
4. 在此实例的队列中入队一个命令，当此命令在队列时间线上执行时，将发起后续步骤。

**队列时间线的步骤：**
1. 从偏移量开始，将缓冲区 size 字节大小的数据设置为 0。

---

**Device timeline steps:**      
1. Validate the encoder state of this. If it returns false, stop.
2. If size is missing, set size to max(0, |buffer|.{{GPUBuffer/size}} - |offset|).
3. If any of the following conditions are unsatisfied make this invalid and stop.
    * buffer is valid to use with this.
    * buffer.usage contains COPY_DST.
    * size is a multiple of 4.
    * offset is a multiple of 4.
    * buffer.size ≥ (offset + size).
4. Enqueue a command on this which issues the subsequent steps on the Queue timeline when executed.

**Queue timeline steps:**
1. Set size bytes of buffer to 0 starting at offset.

---
---

## 13.5. 图像复制命令

**copyBufferToTexture(source, destination, copySize)**

将一条命令编码到 GPUCommandEncoder 中，其作用是将数据从 GPUBuffer 的一个子区域复制到一个或多个连续纹理子资源的一个子区域。

被调用的对象：GPUCommandEncoder 实例自身。

参数：

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| `source` | `GPUImageCopyBuffer` | ✘ | ✘ | 与copySize结合，定义了源缓冲区的区域。 |
| `destination` | `GPUImageCopyTexture` | ✘ | ✘ | 与copySize结合，定义了目标纹理子资源的区域。 |
| `copySize` | `GPUExtent3D` | ✘ | ✘ |  |

返回： undefined

上下文时间线步骤：
1. ? 验证 GPUOrigin3D shape(destination.origin)。
2. ? 验证 GPUExtent3D shape(copySize)。
3. 在 this.[[device]] 的设备时间线上发起后续步骤：

---

13.5. Image Copy Commands

**copyBufferToTexture(source, destination, copySize)**    
Encode a command into the GPUCommandEncoder that copies data from a sub-region of a GPUBuffer to a sub-region of one or multiple continuous texture subresources.

**Called on:** GPUCommandEncoder this.

**Arguments:**

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| `source` | `GPUImageCopyBuffer` | ✘ | ✘ | Combined with copySize, defines the region of the source buffer. |
| `destination` | `GPUImageCopyTexture` | ✘ | ✘ | Combined with copySize, defines the region of the destination texture subresource. |
| `copySize` | `GPUExtent3D` | ✘ | ✘ |  |

**Returns:** undefined

**Content timeline steps:**   
1. ? validate GPUOrigin3D shape(destination.origin).
2. ? validate GPUExtent3D shape(copySize).
3. Issue the subsequent steps on the Device timeline of this.[[device]]:

---
---

**设备时间轴步骤：**        
1. 验证实例自身的编码器状态。如果它返回false，则停止。
2. 如果以下任何条件不满足，使之无效并停止。
    * 让 dstTexture 成为 destination.texture。
    * 验证 GPUImageCopyBuffer(source) 函数返回 true。
    * source.buffer.usage 包含 COPY_SRC。
    * 验证 GPUImageCopyTexture(destination, copySize) 返回 true。
    * dstTexture.usage 包含 COPY_DST。
    * dstTexture.sampleCount 是 1。
    * 让 aspectSpecificFormat = dstTexture.format.
    * 如果 dstTexture.format 是一种深度/模板格式：
        * destination.aspect 必须指向 dstTexture.format 的一个单一方面。
        * 根据 § 26.1.2 深度/模板格式，该方面必须是一个有效的图像复制目的地。
        * 根据 § 26.1.2 深度/模板格式，将 aspectSpecificFormat 设置为特定方面的格式。
    * 验证 `纹理复制范围（destination, copySize）` 返回true。
    * 如果 dstTexture.format 不是深度/模板格式：
        * source.offset 是 dstTexture.format 的 texel 块复制足迹的 倍数。
    * 如果 dstTexture.format 是一种深度/模板格式：
        * source.offset 是 4 的倍数。
    * 验证 `线性纹理数据（source, source.buffer.size, aspectSpecificFormat, copySize）` 函数返回成功。
3. 在此实例的队列中入队一个命令，当此命令在队列时间线上执行时，将发起后续步骤。

**队列时间线的步骤：**

>> **ISSUE 23** 定义拷贝，包括对 snorm 的规定。

---

**Device timeline steps:**      
1. Validate the encoder state of this. If it returns false, stop.
2. If any of the following conditions are unsatisfied, make this invalid and stop.
    * Let dstTexture be destination.texture.
    * validating GPUImageCopyBuffer(source) returns true.
    * source.buffer.usage contains COPY_SRC.
    * validating GPUImageCopyTexture(destination, copySize) returns true.
    * dstTexture.usage contains COPY_DST.
    * dstTexture.sampleCount is 1.
    * Let aspectSpecificFormat = dstTexture.format.
    * If dstTexture.format is a depth-or-stencil format:
        * destination.aspect must refer to a single aspect of dstTexture.format.
        * That aspect must be a valid image copy destination according to § 26.1.2 Depth-stencil formats.
        * Set aspectSpecificFormat to the aspect-specific format according to § 26.1.2 Depth-stencil formats.
    * validating texture copy range(destination, copySize) return true.
    * If dstTexture.format is not a depth-or-stencil format:
        * source.offset is a multiple of the texel block copy footprint of dstTexture.format.
    * If dstTexture.format is a depth-or-stencil format:
        * source.offset is a multiple of 4.
    * validating linear texture data(source, source.buffer.size, aspectSpecificFormat, copySize) succeeds.
3. Enqueue a command on this which issues the subsequent steps on the Queue timeline when executed.

**Queue timeline steps:**
>> **ISSUE 23** Define copy, including provision for snorm.

---
---

**copyTextureToBuffer(source, destination, copySize)**

将一条命令编码到 GPUCommandEncoder 中，其作用是从一个或多个连续纹理子资源的子区域复制数据到 GPUBuffer 的子区域。

调用于：GPUCommandEncoder 实例自身

参数：

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| `source` | `GPUImageCopyTexture` | ✘ | ✘ | 与 copySize 结合，定义了源纹理子资源的区域。 |
| `destination` | `GPUImageCopyBuffer` | ✘ | ✘ | 与 copySize 结合，定义目标缓冲区的区域。 |
| `copySize` | `GPUExtent3D` | ✘ | ✘ |  |

返回：undefined

**上下文时间线步骤：**
1. ? 验证 `GPUOrigin3D shape(source.origin)`。
2. ? 验证 `GPUExtent3D shape(copySize)`。
3. 在 this.[[device]] 的设备时间线上发布后续步骤：

---

**copyTextureToBuffer(source, destination, copySize)**    

Encode a command into the GPUCommandEncoder that copies data from a sub-region of one or multiple continuous texture subresourcesto a sub-region of a GPUBuffer.

**Called on:** GPUCommandEncoder this.

**Arguments:**

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| `source` | `GPUImageCopyTexture` | ✘ | ✘ | Combined with copySize, defines the region of the source texture subresources. |
| `destination` | `GPUImageCopyBuffer` | ✘ | ✘ | Combined with copySize, defines the region of the destination buffer. |
| `copySize` | `GPUExtent3D` | ✘ | ✘ |  |

**Returns:** undefined

**Content timeline steps:**     
1. ? validate GPUOrigin3D shape(source.origin).
2. ? validate GPUExtent3D shape(copySize).
3. Issue the subsequent steps on the Device timeline of this.[[device]]:

---
---

**设备时间轴步骤：**

1. 验证实例自身的编码器状态。如果它返回false，则停止。
2. 如果以下任何条件不满足，使之无效并停止。
    * 让 srcTexture 成为 source.texture。
    * 验证 `GPUImageCopyTexture(source, copySize)` 返回 true。
    * srcTexture.usage 包含 COPY_SRC。
    * srcTexture.sampleCount 为 1。
    * 让 aspectSpecificFormat = srcTexture.format.
    * 如果 srcTexture.format 是一个深度/模板格式的格式：
        * source.aspect 必须引用的是 srcTexture.format 的一个单一方面。
        * 根据 §26.1.2 深度/模板格式，该方面必须是有效的图像拷贝源。
        * 根据 §26.1.2 深度/模板格式，将 aspectSpecificFormat 设置为一个特定方面的格式
    * 验证 `GPUImageCopyBuffer(destination)` 返回true。
    * destination.buffer.usage 包含 COPY_DST。
    * 验证 `纹理复制范围（source, copySize）`返回true。
    * 如果 srcTexture.format 不是深度/模板格式：
        * destination.offset 是 srcTexture.format 的 texel 块复制范围的倍数。
    * 如果 srcTexture.format 是一种深度/模板格式：
        * destination.offset 是 4 的倍数。
    * 验证 `线性纹理数据（ destination, destination.buffer.size, aspectSpecificFormat, copySize）` 且成功。
3. 在此实例的队列中入队一个命令，当此命令在队列时间线上执行时，将发起后续步骤。

**队列时间轴步骤：**

>> **ISSUE 24** 定义副本，包括为snorm提供。

---

**Device timeline steps:**    
1. Validate the encoder state of this. If it returns false, stop.
2. If any of the following conditions are unsatisfied, make this invalid and stop.
    * Let srcTexture be source.texture.
    * validating GPUImageCopyTexture(source, copySize) returns true.
    * srcTexture.usage contains COPY_SRC.
    * srcTexture.sampleCount is 1.
    * Let aspectSpecificFormat = srcTexture.format.
    * If srcTexture.format is a depth-or-stencil format format:
        * source.aspect must refer to a single aspect of srcTexture.format.
        * That aspect must be a valid image copy source according to § 26.1.2 Depth-stencil formats.
        * Set aspectSpecificFormat to the aspect-specific format according to § 26.1.2 Depth-stencil formats.
    * validating GPUImageCopyBuffer(destination) returns true.
    * destination.buffer.usage contains COPY_DST.
    * validating texture copy range(source, copySize) returns true.
    * If srcTexture.format is not a depth-or-stencil format:
        * destination.offset is a multiple of the texel block copy footprint of srcTexture.format.
    * If srcTexture.format is a depth-or-stencil format:
        * destination.offset is a multiple of 4.
    * validating linear texture data(destination, destination.buffer.size, aspectSpecificFormat, copySize) succeeds.
3. Enqueue a command on this which issues the subsequent steps on the Queue timeline when executed.

**Queue timeline steps:**     
>> **ISSUE 24** Define copy, including provision for snorm.

---
---

**copyTextureToTexture(source, destination, copySize)**

将一条命令编码到 GPUCommandEncoder 中，其作用是从一个或多个连续纹理子资源的子区域复制数据到另一个或多个连续纹理子资源的子区域。

调用于：GPUCommandEncoder 实例自身

参数：

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| `source` | `GPUImageCopyTexture` | ✘ | ✘ | 与copySize结合，定义了源纹理子资源的区域。 |
| `destination` | `GPUImageCopyTexture` | ✘ | ✘ | 与copySize结合，定义目标纹理子资源的区域。 |
| `copySize` | `GPUExtent3D` | ✘ | ✘ |  |

返回：undefined

**上下文时间线步骤：**      
1. ? 验证 `GPUOrigin3D shape(source.origin)`。
2. ? 验证 `GPUOrigin3D shape(destination.origin)`。
3. ? 验证 `GPUExtent3D shape(copySize)`。
4. 在this.[[device]]的设备时间线上发起后续步骤：

---

**copyTextureToTexture(source, destination, copySize)**     

Encode a command into the GPUCommandEncoder that copies data from a sub-region of one or multiple contiguous texture subresources to another sub-region of one or multiple continuous texture subresources.

**Called on:** GPUCommandEncoder this.

**Arguments:**

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| `source` | `GPUImageCopyTexture` | ✘ | ✘ | Combined with copySize, defines the region of the source texture subresources. |
| `destination` | `GPUImageCopyTexture` | ✘ | ✘ | Combined with copySize, defines the region of the destination texture subresources. |
| `copySize` | `GPUExtent3D` | ✘ | ✘ |  |

**Returns:** undefined

**Content timeline steps:**
1. ? validate GPUOrigin3D shape(source.origin).
2. ? validate GPUOrigin3D shape(destination.origin).
3. ? validate GPUExtent3D shape(copySize).
4. Issue the subsequent steps on the Device timeline of this.[[device]]:

---
---

**设备时间轴步骤：**
1. 验证实例的编码器的状态，如果它返回false，则停止。
2. 如果以下任何条件不满足，使之无效并停止。
    * 让 srcTexture 成为 source.texture。
    * 让dstTexture 成为 destination.texture。
    * 验证 `GPUImageCopyTexture(source, copySize)` 返回 true。
    * srcTexture.usage 包含 COPY_SRC。
    * 验证 `GPUImageCopyTexture(destination, copySize)` 返回 true。
    * dstTexture.usage 包含 COPY_DST。
    * srcTexture.sampleCount 等于 dstTexture.sampleCount。
    * srcTexture.format 和 dstTexture.format 必须是复制相兼容的。
    * 如果 srcTexture.format 是一种深度/模板格式：
        * source.aspect 和 destination.aspect 必须同时分别引用 srcTexture.form 和 dstTexture.form 的所有方面。
    * 验证 `纹理复制范围（source, copySize）` 并返回 true。
    * 验证 `纹理复制范围（destination，copySize）` 并返回 true。
    * `纹理拷贝（source，copySize）` 的子资源集 和 `纹理拷贝（destination，copySize）`的子资源集是不相交的。
3. 在此实例的队列中入队一个命令，当此命令在队列时间线上执行时，将发起后续步骤。

**队列时间轴步骤：**

>> **ISSUE 25** 定义副本，包括为snorm提供。

---

**Device timeline steps:**    
1. Validate the encoder state of this. If it returns false, stop.
2. If any of the following conditions are unsatisfied, make this invalid and stop.
    * Let srcTexture be source.texture.
    * Let dstTexture be destination.texture.
    * validating GPUImageCopyTexture(source, copySize) returns true.
    * srcTexture.usage contains COPY_SRC.
    * validating GPUImageCopyTexture(destination, copySize) returns true.
    * dstTexture.usage contains COPY_DST.
    * srcTexture.sampleCount is equal to dstTexture.sampleCount.
    * srcTexture.format and dstTexture.format must be copy-compatible.
    * If srcTexture.format is a depth-stencil format:
        * source.aspect and destination.aspect must both refer to all aspects of srcTexture.format and dstTexture.format, respectively.
    * validating texture copy range(source, copySize) returns true.
    * validating texture copy range(destination, copySize) returns true.
    * The set of subresources for texture copy(source, copySize) and the set of subresources for texture copy(destination, copySize) are disjoint.
3. Enqueue a command on this which issues the subsequent steps on the Queue timeline when executed.

**Queue timeline steps:**     
>> **ISSUE 25** Define copy, including provision for snorm.

---
---

## 13.6. 查询

**writeTimestamp(querySet, queryIndex)**    

当之前的所有命令都执行完毕后，将一个时间戳值写入querySet中。

调用于：GPUCommandEncoder 实例自身

参数：

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| `querySet` | `GPUQuerySet` | ✘ | ✘ | 将存储时间戳值的查询集。 |
| `queryIndex` | `GPUSize32` | ✘ | ✘ | 查询集中的查询索引。 |

返回： undefined

**上下文时间线步骤：**
1. 如果 "timestamp-query "没有为其启用：
    1. 抛出一个 TypeError。
2. 在 this.[[device]] 的设备时间轴上发起后续步骤。

---

13.6. Queries

**writeTimestamp(querySet, queryIndex)**      

Writes a timestamp value into a querySet when all previous commands have completed executing.

**Called on:** GPUCommandEncoder this.

**Arguments:**    

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| `querySet` | `GPUQuerySet` | ✘ | ✘ | The query set that will store the timestamp values. |
| `queryIndex` | `GPUSize32` | ✘ | ✘ | The index of the query in the query set. |

**Returns:** undefined

**Content timeline steps:**
1. If "timestamp-query" is not enabled for this:
    1. Throw a TypeError.
2. Issue the subsequent steps on the Device timeline of this.[[device]]:

---
---

**设备时间轴步骤：**
1. 验证实例编码器的状态。如果它返回false，则停止。
2. 如果以下任何条件不满足，使之无效并停止。
    * 实例的 this 所指向的 querySet 是有效的。
    * querySet.type 是 "timestamp"。
    * queryIndex < querySet.count。
3. 在此实例的队列中入队一个命令，当此命令在队列时间线上执行时，将发起后续步骤。

**队列时间线的步骤：**
1. 将队列时间线的当前时间戳，以纳秒为单位，写入 querySet 的 queryIndex 所指的位置处。

---

**Device timeline steps:**
1. Validate the encoder state of this. If it returns false, stop.
2. If any of the following conditions are unsatisfied, make this invalid and stop.
    * querySet is valid to use with this.
    * querySet.type is "timestamp".
    * queryIndex < querySet.count.
3. Enqueue a command on this which issues the subsequent steps on the Queue timeline when executed.

**Queue timeline steps:**   
1. Write the current timestamp for the Queue timeline, in nanoseconds, into querySet at index queryIndex.

---
---

**resolveQuerySet(querySet, firstQuery, queryCount, destination, destinationOffset)**   

将查询结果从 GPUQuerySet 转移到 GPUBuffer 的一个范围内。

调用于：GPUCommandEncoder 实例自身‘

参数：

| Parameter | Type | Nullable | Optional | Description  |
| --- | --- | --- | --- | --- |
| querySet | GPUQuerySet | ✘ | ✘ |  |
| firstQuery | GPUSize32 | ✘ | ✘ |  |
| queryCount | GPUSize32 | ✘ | ✘ |  |
| destination | GPUBuffer | ✘ | ✘ |  |
| destinationOffset | GPUSize64 | ✘ | ✘ |  |

返回： undefined

**上下文时间线步骤：**
1. 在 this.[[device]] 的设备时间线上发起后续步骤。

---

**resolveQuerySet(querySet, firstQuery, queryCount, destination, destinationOffset)**     
Resolves query results from a GPUQuerySet out into a range of a GPUBuffer.

**Called on:** GPUCommandEncoder this.

**Arguments:**

| Parameter | Type | Nullable | Optional | Description  |
| --- | --- | --- | --- | --- |
| querySet | GPUQuerySet | ✘ | ✘ |  |
| firstQuery | GPUSize32 | ✘ | ✘ |  |
| queryCount | GPUSize32 | ✘ | ✘ |  |
| destination | GPUBuffer | ✘ | ✘ |  |
| destinationOffset | GPUSize64 | ✘ | ✘ |  |

**Returns:** undefined

**Content timeline steps:**
1. Issue the subsequent steps on the Device timeline of this.[[device]].

---
---

**设备时间轴步骤：**    
1. 验证实例的编码器状态。如果它返回false，则停止。
2. 如果以下任何条件不满足，使之无效并停止。
    * 实例 this 指向的 querySet是有效的。
    * 实例 this 指向的 destination是有效的。
    * destination.usage 包含 QUERY_RESOLVE。
    * firstQuery < querySet 中的查询数量。
    * (firstQuery + queryCount) ≤ querySet中的查询数量。
    * destinationOffset 是 256 的倍数。
    * destinationOffset + 8 × queryCount ≤ destination.size。
3. 在此实例的队列中入队一个命令，当此命令在队列时间线上执行时，将发起后续步骤。

---

**Device timeline steps:**      
1. Validate the encoder state of this. If it returns false, stop.
2. If any of the following conditions are unsatisfied, make this invalid and stop.
    * querySet is valid to use with this.
    * destination is valid to use with this.
    * destination.usage contains QUERY_RESOLVE.
    * firstQuery < the number of queries in querySet.
    * (firstQuery + queryCount) ≤ the number of queries in querySet.
    * destinationOffset is a multiple of 256.
    * destinationOffset + 8 × queryCount ≤ destination.size.
3. Enqueue a command on this which issues the subsequent steps on the Queue timeline when executed.

---
---

**队列时间轴步骤：**
1. 让 queryIndex 成为 firstQuery。
2. 让 offset 为 destinationOffset。
3. 当 queryIndex < firstQuery + queryCount 时： 
    1. 设置目的地的 8个字节，从offset开始，是 queryIndex处的 querySet 的值。
    2. 将 queryIndex 设置为 queryIndex+1。
    3. 将 offset 设置为 offset+8。

---

**Queue timeline steps:**   
1. Let queryIndex be firstQuery.
2. Let offset be destinationOffset.
3. While queryIndex < firstQuery + queryCount:
    1. Set 8 bytes of destination, beginning at offset, to be the value of querySet at queryIndex.
    2. Set queryIndex to be queryIndex + 1.
    3. Set offset to be offset + 8.

---
---

## 13.7. 最终化

通过调用 finish() 可以创建一个包含 GPUCommandEncoder 命令记录的 GPUCommandBuffer。一旦finish()被调用，命令编码器就不能再被使用。

**finish(descriptor)**

作用是完成命令序列的记录，并返回一个相应的 GPUCommandBuffer。

调用于：GPUCommandEncoder 实例自身

参数：

| Parameter | Type | Nullable | Optional | Description  |
| --- | --- | --- | --- | --- |
| descriptor | GPUCommandBufferDescriptor | ✘ | ✔ |  |

返回：GPUCommandBuffer

**上下文时间线步骤：**
1. 让 commandBuffer 成为一个新的 GPUCommandBuffer。
2. 在 this.[[device]] 的设备时间线上发起完成步骤。
3. 返回 commandBuffer。

---

13.7. Finalization

A GPUCommandBuffer containing the commands recorded by the GPUCommandEncoder can be created by calling finish(). Once finish() has been called the command encoder can no longer be used.

**finish(descriptor)**

Completes recording of the commands sequence and returns a corresponding GPUCommandBuffer.

**Called on:** GPUCommandEncoder this.

**Arguments:**

| Parameter | Type | Nullable | Optional | Description  |
| --- | --- | --- | --- | --- |
| descriptor | GPUCommandBufferDescriptor | ✘ | ✔ |  |

**Returns:** GPUCommandBuffer

**Content timeline steps:**
1. Let commandBuffer be a new GPUCommandBuffer.
2. Issue the finish steps on the Device timeline of this.[[device]].
3. Return commandBuffer.

---
---

**设备时间线完成步骤：**
1. 如果满足以下所有要求，让 validationSucceeded 为真，否则为假。
    * 实例的this 必须是有效的。
    * this.[[state]] 必须是 "open"。
    * this.[[debug_group_stack]] 必须为空。
    * This 中包含的每个使用范围都必须满足使用范围验证的要求。
2. 将 this.[[state]] 设置为 "end"。
3. 如果 validationSucceeded 是假的，那么：
    1. 产生一个验证错误。
    2. 返回一个新的无效的 GPUCommandBuffer。
4. 将 commandBuffer.[[command_list]] 设置为 this.[[commands]]。

---

**Device timeline finish steps:**     
1. Let validationSucceeded be true if all of the following requirements are met, and false otherwise.
    * this must be valid.
    * this.[[state]] must be "open".
    * this.[[debug_group_stack]] must be empty.
    * Every usage scope contained in this must satisfy the usage scope validation.
2. Set this.[[state]] to "ended".
3. If validationSucceeded is false, then:
    1. Generate a validation error.
    2. Return a new invalid GPUCommandBuffer.
4. Set commandBuffer.[[command_list]] to this.[[commands]].

