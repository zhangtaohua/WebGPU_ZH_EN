WebGPU

W3C Working Draft, 6 April 2023 

More details about this document

This version:

https://www.w3.org/TR/2023/WD-webgpu-20230406/

---
---

# 19\. 队列

19.1. GPUQueueDescriptor

GPUQueueDescriptor 描述了一个队列请求。

```
dictionary GPUQueueDescriptor : GPUObjectDescriptorBase {
};
```

---

19\. Queues

19.1. GPUQueueDescriptor

GPUQueueDescriptor describes a queue request.

```
dictionary GPUQueueDescriptor : GPUObjectDescriptorBase {
};
```

---
---

19.2. GPU 队列

```
[Exposed=(Window, DedicatedWorker), SecureContext]
interface GPUQueue {
    undefined submit(sequence<GPUCommandBuffer> commandBuffers);

    Promise<undefined> onSubmittedWorkDone();

    undefined writeBuffer(
        GPUBuffer buffer,
        GPUSize64 bufferOffset,
        [AllowShared] BufferSource data,
        optional GPUSize64 dataOffset = 0,
        optional GPUSize64 size);

    undefined writeTexture(
        GPUImageCopyTexture destination,
        [AllowShared] BufferSource data,
        GPUImageDataLayout dataLayout,
        GPUExtent3D size);

    undefined copyExternalImageToTexture(
        GPUImageCopyExternalImage source,
        GPUImageCopyTextureTagged destination,
        GPUExtent3D copySize);
};
GPUQueue includes GPUObjectBase;
```

---

19.2. GPUQueue

```
[Exposed=(Window, DedicatedWorker), SecureContext]
interface GPUQueue {
    undefined submit(sequence<GPUCommandBuffer> commandBuffers);

    Promise<undefined> onSubmittedWorkDone();

    undefined writeBuffer(
        GPUBuffer buffer,
        GPUSize64 bufferOffset,
        [AllowShared] BufferSource data,
        optional GPUSize64 dataOffset = 0,
        optional GPUSize64 size);

    undefined writeTexture(
        GPUImageCopyTexture destination,
        [AllowShared] BufferSource data,
        GPUImageDataLayout dataLayout,
        GPUExtent3D size);

    undefined copyExternalImageToTexture(
        GPUImageCopyExternalImage source,
        GPUImageCopyTextureTagged destination,
        GPUExtent3D copySize);
};
GPUQueue includes GPUObjectBase;
```

---
---

GPUQueue 有以下方法：

**writeBuffer(buffer, bufferOffset, data, dataOffset, size)**       

对提供的数据进行写入 GPUBuffer 的操作。

**调用于：** GPUQueue 实例自身。

**参数：**      

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| buffer | GPUBuffer | ✘ | ✘ | 要写入的缓冲区。 |
| bufferOffset | GPUSize64 | ✘ | ✘ | 从缓冲区的那个字节偏移量开始写入。 |
| data | BufferSource | ✘ | ✘ | 要写进缓冲区的数据。 |
| dataOffset | GPUSize64 | ✘ | ✔ | 要从数据中的那个偏移量开始写入。如果数据是 TypedArray，则以元素为单位，否则以字节为单位。 |
| size | GPUSize64 | ✘ | ✔ | 从数据写入缓冲区的内容大小。如果数据是 TypedArray，以元素为单位，否则以字节为单位。 |

**返回值：** undefined

---

GPUQueue has the following methods:

**writeBuffer(buffer, bufferOffset, data, dataOffset, size)**       

Issues a write operation of the provided data into a GPUBuffer.

**Called on:** GPUQueue this.

**Arguments:**

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| buffer | GPUBuffer | ✘ | ✘ | The buffer to write to. |
| bufferOffset | GPUSize64 | ✘ | ✘ | Offset in bytes into buffer to begin writing at. |
| data | BufferSource | ✘ | ✘ | Data to write into buffer. |
| dataOffset | GPUSize64 | ✘ | ✔ | Offset in into data to begin writing from. Given in elements if data is a TypedArray and bytes otherwise. |
| size | GPUSize64 | ✘ | ✔ | Size of content to write from data to buffer. Given in elements if data is a TypedArray and bytes otherwise. |

Returns: undefined

---
---

**上下文时间轴步骤：**
1. 如果数据是一个 ArrayBuffer 或 DataView，让元素类型为 "byte"。否则，数据是一个TypedArray；让元素类型是 TypedArray 的类型。
2. 让 dataSize 为数据的大小，以元素为单位。
3. 如果大小缺失，让 contentsSize 为 dataSize - dataOffset。否则，让 contentsSize 为size。
4. 如果以下任何条件不能满足，抛出 OperationError 并停止。
    * contentsSize ≥ 0。
    * dataOffset + contentsSize ≤ dataSize。
    * contentsSize，转换为字节，是 4 字节的倍数。
5. 让 dataContents 成为源缓冲区所持有的字节的一个副本。
6. 让 contents 是 dataContents 中数据的 contentsSize 个元素，位置从 dataOffset 偏移处开始。
7. 在实例 this 的设备时间轴上发起后续步骤。

---

**Content timeline steps:**
1. If data is an ArrayBuffer or DataView, let the element type be "byte". Otherwise, data is a TypedArray; let the element type be the type of the TypedArray.
2. Let dataSize be the size of data, in elements.
3. If size is missing, let contentsSize be dataSize − dataOffset. Otherwise, let contentsSize be size.
4. If any of the following conditions are unsatisfied, throw OperationError and stop.
    * contentsSize ≥ 0.
    * dataOffset + contentsSize ≤ dataSize.
    * contentsSize, converted to bytes, is a multiple of 4 bytes.
5. Let dataContents be a copy of the bytes held by the buffer source.
6. Let contents be the contentsSize elements of dataContents starting at an offset of dataOffset elements.
7. Issue the subsequent steps on the Device timeline of this.

---
---

**设备时间轴步骤：**
1. 如果以下任何条件未能得到满足，产生一个验证错误并停止。
    * 实例 this 所指向的缓冲区是有效的。
    * buffer.[[internals]].state 是 "available"。
    * buffer.usage 包括 COPY_DST。
    * bufferOffset，转换为字节数，要是 4 的倍数。
    * bufferOffset + contentsSize，转换为字节数，要小于等于 buffer.size 字节数。
2. 将内容写入缓冲区，从 bufferOffset 的位置开始。

---

**Device timeline steps:**

1. If any of the following conditions are unsatisfied, generate a validation error and stop.
    * buffer is valid to use with this.
    * buffer.[[internals]].state is "available".
    * buffer.usage includes COPY_DST.
    * bufferOffset, converted to bytes, is a multiple of 4 bytes.
    * bufferOffset + contentsSize, converted to bytes, ≤ buffer.size bytes.
2. Write contents into buffer starting at bufferOffset.

---
---

writeTexture(destination, data, dataLayout, size)
对提供的数据进行写入 GPUTexture 的操作。

**调用于：** GPUQueue 实例自身。

**参数：**  

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| destination | GPUImageCopyTexture | ✘ | ✘ | 源数据要写入的纹理子资源。 |
| data | BufferSource | ✘ | ✘ | 要写进目的地的数据。 |
| dataLayout | GPUImageDataLayout | ✘ | ✘ | 数据中内容的布局。 |
| size | GPUExtent3D | ✘ | ✘ | 要从数据写到目的地的内容的范围。 |

**返回值：** undefined

---

**writeTexture(destination, data, dataLayout, size)**       

Issues a write operation of the provided data into a GPUTexture.

**Called on:** GPUQueue this. 

**Arguments:**

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| destination | GPUImageCopyTexture | ✘ | ✘ | The texture subresource and origin to write to. |
| data | BufferSource | ✘ | ✘ | Data to write into destination. |
| dataLayout | GPUImageDataLayout | ✘ | ✘ | Layout of the content in data. |
| size | GPUExtent3D | ✘ | ✘ | Extents of the content to write from data to destination. |

Returns: undefined

---
---

**上下文时间轴步骤：**      
1. ? 验证 `GPUOrigin3D shape(destination.origin)`。
2. ? 验证 `GPUExtent3D shape(size)`。
3. 让 dataBytes 成为缓冲区源数据所持有的数据字节的副本。
4. 在实例 this 的设备时间轴上发起后续步骤。

---

**Content timeline steps:**
1. ? validate GPUOrigin3D shape(destination.origin).
2. ? validate GPUExtent3D shape(size).
3. Let dataBytes be a copy of the bytes held by the buffer source data.
4. Issue the subsequent steps on the Device timeline of this.

---
---

**设备的时间线步骤：**  
1. 让 texture 成为 destination.texture。
2. 如果以下任何条件不满足，产生一个验证错误并停止。
    * 验证 `GPUImageCopyTexture(destination, size)` 并返回 true。
    * texture.usage 包括 COPY_DST。
    * texture.sampleCount 为 1。
    * 验证 texture copy range(destination, size) 返回 true。
    * destination.aspect 必须引用的是 texture.format 的一个方面。
    * 根据 §26.1.2 深度/模板格式 描述，该方面必须是有效的图像复制目的地。
    * 让 aspectSpecificFormat = texture.format.
    * 如果 texture.format 是一种 深度或模板格式：
        * 根据 §26.1.2深度/模板格式 描述要求，将 aspectSpecificFormat 设置为 texture.format 的一个特定方面格式。
    * 验证`线性纹理数据（dataLayout, dataBytes.length, aspectSpecificFormat, size）`  要返回成功。      
    **注意：** 与 GPUCommandEncoder.copyBufferToTexture() 不同，对 dataLayout.bytesPerRow 或 dataLayout.offset 都没有对齐要求。
3. 让 contents 成为通过查看带有 dataLayout 和 size 的 dataBytes 所访问的图像内容。
    >> **ISSUE 29** 更加正式地指定 。       
    **注意：** 这被描述为将所有的数据复制到设备时间轴上，但在实践中，数据可能比需要的大很多。具体的实现应该通过只复制必要的数据字节来进行优化。
4. 在实例 this 的队列时间轴上发起后续步骤。

**队列时间轴的步骤：**
1. 将内容写入目的地。
   >> **ISSUE 30** 定义副本，包括对 snorm 的规定。

---

**Device timeline steps:**

1. Let texture be destination.texture.
2. If any of the following conditions are unsatisfied, generate a validation error and stop.
    * validating GPUImageCopyTexture(destination, size) returns true.
    * texture.usage includes COPY_DST.
    * texture.sampleCount is 1.
    * validating texture copy range(destination, size) return true.
    * destination.aspect must refer to a single aspect of texture.format.
    * That aspect must be a valid image copy destination according to § 26.1.2 Depth-stencil formats.
    * Let aspectSpecificFormat = texture.format.
    * If texture.format is a depth-or-stencil format:
        * Set aspectSpecificFormat to the aspect-specific format of texture.format according to § 26.1.2 Depth-stencil formats.
    * validating linear texture data(dataLayout, dataBytes.length, aspectSpecificFormat, size) succeeds.            
    **NOTE:** unlike GPUCommandEncoder.copyBufferToTexture(), there is no alignment requirement on either dataLayout.bytesPerRow or dataLayout.offset.
3. Let contents be the contents of the images seen by viewing dataBytes with dataLayout and size.           
**ISSUE 29** Specify more formally.     
**NOTE: ** This is described as copying all of data to the device timeline, but in practice data could be much larger than necessary. Implementations should optimize by copying only the necessary bytes.
1. Issue the subsequent steps on the Queue timeline of this.

**Queue timeline steps:**           
1. Write contents into destination.
    **ISSUE 30** Define copy, including provision for snorm.

---
---

**copyExternalImageToTexture(source, destination, copySize)**

发起一个将平台图像/画布的内容复制到目标纹理中的操作。

此操作根据 GPUImageCopyTextureTagged 的参数，将源目标的颜色编码转换到目标颜色编码中。

复制到带“-srgb” 后缀的纹理的结果是与复制到相应的不带 “-srgb”格式的纹理的结果，其间要复制的字节数是相同，但最后不同的是相应的解码值。因此，在复制操作之后，根据目标纹理的格式是否为带有“-srgb”，对其进行采样会有不同的结果，其他方面都没有改变。

---

**copyExternalImageToTexture(source, destination, copySize)**           

Issues a copy operation of the contents of a platform image/canvas into the destination texture.

This operation performs color encoding into the destination encoding according to the parameters of GPUImageCopyTextureTagged.

Copying into a -srgb texture results in the same texture bytes, not the same decoded values, as copying into the corresponding non--srgb format. Thus, after a copy operation, sampling the destination texture has different results depending on whether its format is -srgb, all else unchanged.

---
---

**调用于：** GPUQueue 实例自身。

**参数：**

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| source | GPUImageCopyExternalImage | ✘ | ✘ | 要复制到目的地的源图像和原目标。 |
| destination | GPUImageCopyTextureTagged | ✘ | ✘ | 要写入的纹理子资源和原点，以及其编码元数据。 |
| copySize | GPUExtent3D | ✘ | ✘ | 从源目标写到目的地的内容的范围。 |

**返回值：** undefined

---

**Called on:** GPUQueue this.

**Arguments:**

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| source | GPUImageCopyExternalImage | ✘ | ✘ | source image and origin to copy to destination. |
| destination | GPUImageCopyTextureTagged | ✘ | ✘ | The texture subresource and origin to write to, and its encoding metadata. |
| copySize | GPUExtent3D | ✘ | ✘ | Extents of the content to write from source to destination. |

Returns: undefined

---
---

**上下文时间轴步骤：**
1. ?验证 `GPUOrigin2D shape(source.origin)`。
2. ?验证 `GPUOrigin3D shape(destination.origin)`。
3. ?验证 `GPUExtent3D shape(copySize)`。
4. 让 sourceImage 成为 source.source 。
5. 如果 sourceImage 不是 origin-clean，则抛出一个 SecurityError 并停止。
6. 如果以下要求没有得到满足，抛出一个操作错误并停止。
    * source.origin.x + copySize.width 必须小于等于 sourceImage的宽度。
    * source.origin.y + copySize.height 必须小于等于 sourceImage的高度。
    * source.origin.z + copySize.depthOrArrayLayers 必须小于等于 1。
7. 让可用性 usability 为  ?`检查图像参数的可用性(source)`。
8. 在实例 this 的设备时间轴上发起后续步骤。

---

**Content timeline steps:**
1. ? validate GPUOrigin2D shape(source.origin).
2. ? validate GPUOrigin3D shape(destination.origin).
3. ? validate GPUExtent3D shape(copySize).
4. Let sourceImage be source.source
5. If sourceImage is not origin-clean, throw a SecurityError and stop.
6. If any of the following requirements are unmet, throw an OperationError and stop.
    * source.origin.x + copySize.width must be ≤ the width of sourceImage.
    * source.origin.y + copySize.height must be ≤ the height of sourceImage.
    * source.origin.z + copySize.depthOrArrayLayers must be ≤ 1.
7. Let usability be ? check the usability of the image argument(source).
8. Issue the subsequent steps on the Device timeline of this.


---
---

**设备时间轴步骤：**    
1. 让纹理 texture 成为 destination.texture。
2. 如果以下任何一项要求没有得到满足，产生一个验证错误并停止。
    * 可用性 usability 必须是好的。
    * 实例 this 所指向的 destination.texture 必须是有效的。
    * 验证 `GPUImageCopyTexture(destination, copySize)` 必须返回 true。
    * 验证 `纹理复制范围（destination, copySize）` 必须返回true。
    * texture.usage 必须同时包括 RENDER_ATTACHMENT 和 COPY_DST。
    * texture.dimension 必须是 "2d"。
    * texture.sampleCount 必须为 1。
    * texture.format 必须是下列格式之一（它们都支持 RENDER_ATTACHMENT 的用法）：
        * "r8unorm"
        * "r16float"
        * "r32float"
        * "rg8unorm"
        * "rg16float"
        * "rg32float"
        * "rgba8unorm"
        * "rgba8unorm-srgb"
        * "bgra8unorm"
        * "bgra8unorm-srgb"
        * "rgb10a2unorm"
        * "rgba16float"
        * "rgba32float"
3. **ISSUE 31** 做实际的复制。

---

**Device timeline steps:**          
1. Let texture be destination.texture.
2. If any of the following requirements are unmet, generate a validation error and stop.
    * usability must be good.
    * destination.texture must be valid to use with this.
    * validating GPUImageCopyTexture(destination, copySize) must return true.
    * validating texture copy range(destination, copySize) must return true.
    * texture.usage must include both RENDER_ATTACHMENT and COPY_DST.
    * texture.dimension must be "2d".
    * texture.sampleCount must be 1.
    * texture.format must be one of the following formats (which all support RENDER_ATTACHMENT usage):
        * "r8unorm"
        * "r16float"
        * "r32float"
        * "rg8unorm"
        * "rg16float"
        * "rg32float"
        * "rgba8unorm"
        * "rgba8unorm-srgb"
        * "bgra8unorm"
        * "bgra8unorm-srgb"
        * "rgb10a2unorm"
        * "rgba16float"
        * "rgba32float"
3. **ISSUE 31** Do the actual copy.

---
---

**submit(commandBuffers)**      

安排GPU执行该队列上的命令缓冲区。

提交的命令缓冲区不能再次使用。

**调用于：** GPUQueue 实例自身。

**参数：**

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| commandBuffers | sequence\<GPUCommandBuffer\> | ✘ | ✘ |  | 

**返回值：** undefined

---

**submit(commandBuffers)**          

Schedules the execution of the command buffers by the GPU on this queue.

Submitted command buffers cannot be used again.

**Called on:** GPUQueue this.

**Arguments:**

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| commandBuffers | sequence\<GPUCommandBuffer\> | ✘ | ✘ |  | 

Returns: undefined

---
---

**上下文时间轴步骤：**
1. 在实例 this 的设备时间轴上的发起后续步骤：

**设备时间轴步骤：**
1. 如果以下任何一项要求没有得到满足，则产生一个验证错误并停止。
    * 实例this 的 commandBuffers 中的每一个 GPUCommandBuffer 都必须是有效的。
    * 对于 commandBuffers 的任何元素中的任何命令所使用的以下每种类型的资源：
        **GPUBuffer b**     
        b.[[internals]].state 必须是 "available"。      
        **GPUTexture t**        
        t.[[destroyed]] 必须为 false。      
        **GPUExternalTexture et.**      
        [[expired]] 必须是 false。      
        **GPUQuerySet qs**          
        qs 必须处于可用状态。对于闭塞查询，beginRenderPass() 中的闭塞查询集不会被"使用"，除非它也被 beginOcclusionQuery() 使用了。
2. 对于 commandBuffers 中的每个 commandBuffer：
    1. 让 commandBuffer 无效。
3. 在实例 this 的队列时间轴上发起后续步骤：

---

**Content timeline steps:**     
1. Issue the subsequent steps on the Device timeline of this:

**Device timeline steps:**      
1. If any of the following requirements are unmet, generate a validation error and stop.
    * Every GPUCommandBuffer in commandBuffers must be valid to use with this.
    * For each of the following types of resources used by any command in any element of commandBuffers:
        * GPUBuffer b       
        b.[[internals]].state must be "available".      
        * GPUTexture t      
        t.[[destroyed]] must be false.      
        * GPUExternalTexture et             
        et.[[expired]] must be false.           
        * GPUQuerySet qs              
        qs must be in the available state. For occlusion queries, the occlusionQuerySet in beginRenderPass() is not "used" unless it is also used by beginOcclusionQuery().         
2. For each commandBuffer in commandBuffers:
    1. Make commandBuffer invalid.
3. Issue the subsequent steps on the Queue timeline of this:

---
---

**队列时间轴步骤：**
对于 commandBuffers 中的每个commandBuffer：
    1. 执行 commandBuffer.[[command_list]] 中的每个命令。

---

**Queue timeline steps:**       
1. For each commandBuffer in commandBuffers:
    1. Execute each command in commandBuffer.[[command_list]].

---
---

**onSubmittedWorkDone()**

返回一个Promise，一旦这个队列完成了到此刻为止的所有提交的工作任务的处理，将这个Promise状态设置为 resolved。

这个 Promise 的解析成功，意味着在对该队列上最后使用的 GPUBuffers 进行的调用之前的 mapAsync() 调用的也已经完成。

**调用于：** GPUQueue 实例自身。
**返回值：** Promise\<undefined\>。

---

**onSubmittedWorkDone()**       

Returns a Promise that resolves once this queue finishes processing all the work submitted up to this moment.

Resolution of this Promise implies the completion of mapAsync() calls made prior to that call, on GPUBuffers last used exclusively on that queue.

**Called on:** GPUQueue this.

**Returns:** Promise\<undefined\>

---
---

**上下文时间轴的步骤：**
1. 让 contentTimeline 成为 当前的内容时间线。
2. 让 promise 是一个新的 promise 对象。
3. 在实例 this 的设备时间线上发起同步步骤。
4. 返回 promise。

**设备时间轴的同步步骤：**
1. 当设备时间线获知当前所有排队的操作都完成时，或者 this 丢失时，或 this 变成丢失时：
    1. 在 contentTimeline 上发起后续步骤。

**上下文时间轴的步骤：**
1. 设置 promise 状态为 resolved, 值为 undefined。

---

**Content timeline steps:**     
1. Let contentTimeline be the current Content timeline.
2. Let promise be a new promise.
3. Issue the synchronization steps on the Device timeline of this.
4. Return promise.

**Device timeline synchronization steps:**          
1. When the device timeline becomes informed of the completion of all currently-enqueued operations on this, or if this is lost, or when this becomes lost:
    1. Issue the subsequent steps on contentTimeline.

**Content timeline steps:**         
1. Resolve promise.


