# WebGPU

W3C Working Draft, 28 March 2023

More details about this document

This version:

https://www.w3.org/TR/2023/WD-webgpu-20230328/

---
---

# 14\. 可编程管道

```
interface mixin GPUBindingCommandsMixin {
    undefined setBindGroup(GPUIndex32 index, GPUBindGroup bindGroup,
        optional sequence<GPUBufferDynamicOffset> dynamicOffsets = []);

    undefined setBindGroup(GPUIndex32 index, GPUBindGroup bindGroup,
        Uint32Array dynamicOffsetsData,
        GPUSize64 dynamicOffsetsDataStart,
        GPUSize32 dynamicOffsetsDataLength);
};
```

GPUBindingCommandsMixin 假设 GPUObjectBase 和 GPUCommandsMixin 成员存在于同一个对象上。它必须也仅仅只被 包含这些混合器的接口 所包含。

GPUBindingCommandsMixin 有以下内部槽：
[[bind_groups]], 类型为： ordered map<GPUIndex32, GPUBindGroup>。
每个索引的当前 GPUBindGroup，最初为空。

[[dynamic_offsets]], 类型为： ordered map<GPUIndex32, sequence>GPUBufferDynamicOffset>>。
每个 [[bind_groups]] 条目的当前动态偏移量，最初为空。

---

14\. Programmable Passes

```
interface mixin GPUBindingCommandsMixin {
    undefined setBindGroup(GPUIndex32 index, GPUBindGroup bindGroup,
        optional sequence<GPUBufferDynamicOffset> dynamicOffsets = []);

    undefined setBindGroup(GPUIndex32 index, GPUBindGroup bindGroup,
        Uint32Array dynamicOffsetsData,
        GPUSize64 dynamicOffsetsDataStart,
        GPUSize32 dynamicOffsetsDataLength);
};
```

GPUBindingCommandsMixin assumes the presence of GPUObjectBase and GPUCommandsMixin members on the same object. It must only be included by interfaces which also include those mixins.

GPUBindingCommandsMixin has the following internal slots:

**[[bind_groups]], of type ordered map<GPUIndex32, GPUBindGroup>**    
The current GPUBindGroup for each index, initially empty.

**[[dynamic_offsets]], of type ordered map<GPUIndex32, sequence>GPUBufferDynamicOffset>>**    
The current dynamic offsets for each [[bind_groups]] entry, initially empty.

---
---

## 14.1. 绑定组

**setBindGroup()** 有两个重载函数：

**setBindGroup(index, bindGroup, dynamicOffsets)**    

表示为给定的索引设置这当前的 GPUBindGroup。

调用于：GPUBindingCommandsMixin 实例自身。

**参数：**  

**index，类型为： GPUIndex32，不可为空，必传。**    
表示要设置的绑定组的索引。

**bindGroup，类型为： GPUBindGroup，不可为空，必传。**      
表示用于后续渲染或计算命令的绑定组。

**dynamicOffsets, 类型为： sequence<GPUBufferDynamicOffset>, 不可为空, 默认值为 [] 。**   
包含 bindGroup 中每个被标记为 buffer.hasDynamicOffset 的条目的缓冲区偏移量的 数组，单位为字节。

返回：undefined

**注意：** dynamicOffsets[i] 用于绑定组中的第 i 个动态缓冲区绑定，当绑定是由GPUBindGroupLayoutEntry.binding 排序时。以不同的方式说，dynamicOffsets 的顺序与动态缓冲区绑定的 GPUBindGroupLayoutEntry.binding 相同。

**上下文时间轴步骤：**      
1. 在 this.[[device]] 的设备时间线上发起后续步骤。

---

## 14.1. Bind Groups

setBindGroup() has two overloads:

**setBindGroup(index, bindGroup, dynamicOffsets)**    

Sets the current GPUBindGroup for the given index.

**Called on:** GPUBindingCommandsMixin this.

**Arguments:**      
**index, of type GPUIndex32, non-nullable, required**     
The index to set the bind group at.

**bindGroup, of type GPUBindGroup, non-nullable, required**     
Bind group to use for subsequent render or compute commands.

**dynamicOffsets, of type sequence<GPUBufferDynamicOffset>, non-nullable, defaulting to []**      
Array containing buffer offsets in bytes for each entry in bindGroup marked as buffer.hasDynamicOffset.

**Returns:** undefined

**NOTE:** dynamicOffsets[i] is used for the i-th dynamic buffer binding in the bind group, when bindings are ordered by GPUBindGroupLayoutEntry.binding. Said differently dynamicOffsets are in the same order as dynamic buffer binding’s GPUBindGroupLayoutEntry.binding.

**Content timeline steps:**     
1. Issue the subsequent steps on the Device timeline of this.[[device]].

---
---

**设备时间轴步骤：**      
1. 验证实例的编码器状态，如果它返回值为false，则停止。
2. 如果以下任何条件不满足，使之无效并停止。
    * 实例的 this 所指向的 bindGroup是有效的。
    * index < this.[[device]].[[limit]].maxBindGroups。
    * dynamicOffsets.length 是 bindGroup.[[layout]].[dynamicOffsetCount]]。
    * 遍历 bindGroup 中的每个动态绑定偏移量，并对每个 bufferBinding、bufferLayout 和dynamicOffsetIndex 运行以下步骤：
        * 让 bufferDynamicOffset 成为 dynamicOffsets[dynamicOffsetIndex]。
        * bufferBinding.offset + bufferDynamicOffset + bufferLayout.minBindingSize ≤ bufferBinding.buffer.size。
        * 如果 bufferLayout.type 是 "uniform"：
            * dynamicOffset 是 minUniformBufferOffsetAlignment 的倍数。
        * 如果 bufferLayout.type 是 "storage" 或 "read-only-storage"：
            * dynamicOffset 是 minStorageBufferOffsetAlignment 的倍数。
3. 设置 this.[[bind_groups]][index] 为 bindGroup。
4. 设置 this.[[dynamic_offsets]][index] 为 dynamicOffsets 的副本。

---

**Device timeline steps:**
1. Validate the encoder state of this. If it returns false, stop.
2. If any of the following conditions are unsatisfied, make this invalid and stop.
    * bindGroup is valid to use with this.
    * index < this.[[device]].[[limits]].maxBindGroups.
    * dynamicOffsets.length is bindGroup.[[layout]].[[dynamicOffsetCount]].
    * Iterate over each dynamic binding offset in bindGroup and run the following steps for each bufferBinding, bufferLayout, and dynamicOffsetIndex:
        * Let bufferDynamicOffset be dynamicOffsets[dynamicOffsetIndex].
        * bufferBinding.offset + bufferDynamicOffset + bufferLayout.minBindingSize ≤ bufferBinding.buffer.size.
        * if bufferLayout.type is "uniform":
            * dynamicOffset is a multiple of minUniformBufferOffsetAlignment.
        * if bufferLayout.type is "storage" or "read-only-storage":
            * dynamicOffset is a multiple of minStorageBufferOffsetAlignment.
3. Set this.[[bind_groups]][index] to be bindGroup.
4. Set this.[[dynamic_offsets]][index] to be a copy of dynamicOffsets.

---
---

**setBindGroup(index, bindGroup, dynamicOffsetsData, dynamicOffsetsDataStart, dynamicOffsetsDataLength)**   

为给定的索引设置为当前的 GPUBindGroup，将动态偏移量指定为 Uint32Array 的子集。

调用于：GPUBindingCommandsMixin 实例自身。

参数：

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| index | GPUIndex32 | ✘ | ✘ | 要设置绑定组的索引。 |
| bindGroup | GPUBindGroup | ✘ | ✘ | 用于后续的渲染或计算命令的绑定组。 |
| dynamicOffsetsData | Uint32Array | ✘ | ✘ | 在bindGroup中每个被标记为 buffer.hasDynamicOffset 的条目的缓冲区偏移量的数组。 |
| dynamicOffsetsDataStart | GPUSize64 | ✘ | ✘ | 从缓冲区偏移数据开始的元素在 dynamicOffsetsData 中的偏移。 |
| dynamicOffsetsDataLength | GPUSize32 | ✘ | ✘ | 要从 dynamicOffsetsData 中读取的缓冲区偏移量的数量。 |

返回： undefined

---

**setBindGroup(index, bindGroup, dynamicOffsetsData, dynamicOffsetsDataStart, dynamicOffsetsDataLength)**

Sets the current GPUBindGroup for the given index, specifying dynamic offsets as a subset of a Uint32Array.

**Called on:** GPUBindingCommandsMixin this.

**Arguments:**

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| index | GPUIndex32 | ✘ | ✘ | The index to set the bind group at. |
| bindGroup | GPUBindGroup | ✘ | ✘ | Bind group to use for subsequent render or compute commands. |
| dynamicOffsetsData | Uint32Array | ✘ | ✘ | Array containing buffer offsets in bytes for each entry in bindGroup marked as buffer.hasDynamicOffset. |
| dynamicOffsetsDataStart | GPUSize64 | ✘ | ✘ | Offset in elements into dynamicOffsetsData where the buffer offset data begins. |
| dynamicOffsetsDataLength | GPUSize32 | ✘ | ✘ | Number of buffer offsets to read from dynamicOffsetsData. |

**Returns:** undefined

---
---

**上下文时间轴步骤：**    
1. 如果以下任何要求未得到满足，则抛出一个 RangeError 并停止。
    * dynamicOffsetsDataStart 必须大于等于 0。
    * dynamicOffsetsDataStart + dynamicOffsetsDataLength 必须小于等于 dynamicOffsetsData.length。
2. 让 dynamicOffsets 是一个列表，且包含从索引dynamicOffsetsDataStart开始，长度为 OffsetsDataLength 的 dynamicOffsetsData 元素的一个副本范围。
3. 调用 `this.setBindGroup（index, bindGroup, dynamicOffsets）`。

---

**Content timeline steps:**
1. If any of the following requirements are unmet, throw a RangeError and stop.
    * dynamicOffsetsDataStart must be ≥ 0.
    * dynamicOffsetsDataStart + dynamicOffsetsDataLength must be ≤ dynamicOffsetsData.length.
2. Let dynamicOffsets be a list containing the range, starting at index dynamicOffsetsDataStart, of dynamicOffsetsDataLength elements of a copy of dynamicOffsetsData.
3. Call this.setBindGroup(index, bindGroup, dynamicOffsets).

---
---

为了遍历给定的 GPUBindGroup 的 bindGroup 中的每个动态绑定偏移量，给定如下每个动态偏移量要执行的步骤列表：
1. 让 dynamicOffsetIndex 为 0。
2. 让 layout 成为 bindGroup.[[layout]]。
3. 对于 bindGroup.[[entry]] 中的每个 GPUBindGroupEntry 条目，按照 entry.binding 的值进行递增排序：
    1. 让 bindingDescriptor 成为 layout.[[entryMap]][entry.binding] 中的GPUBindGroupLayoutEntry。
    2. 如果 bindingDescriptor.buffer?.hasDynamicOffset 为真：
        1. 让 bufferBinding 成为 entry.resource.
        2. 让 bufferLayout 成为 bindingDescriptor.buffer。
        3. 用 bufferBinding、bufferLayout 和 dynamicOffsetIndex 调用步骤。
        4. 让 dynamicOffsetIndex 成为 dynamicOffsetIndex + 1

---

To Iterate over each dynamic binding offset in a given GPUBindGroup bindGroup with a given list of steps to be executed for each dynamic offset:
1. Let dynamicOffsetIndex be 0.
2. Let layout be bindGroup.[[layout]].
3. For each GPUBindGroupEntry entry in bindGroup.[[entries]] ordered in increasing values of entry.binding:
    1. Let bindingDescriptor be the GPUBindGroupLayoutEntry at layout.[[entryMap]][entry.binding]:
    2. If bindingDescriptor.buffer?.hasDynamicOffset is true:
        1. Let bufferBinding be entry.resource.
        2. Let bufferLayout be bindingDescriptor.buffer.
        3. Call steps with bufferBinding, bufferLayout, and dynamicOffsetIndex.
        4. Let dynamicOffsetIndex be dynamicOffsetIndex + 1

---
---

验证 `编码器绑定组(encoder, pipeline)`

参数：

GPU绑定命令混合编码器(GPUBindingCommandsMixin encoder)
编码器，其绑定组正在被验证。

GPU基础管道的管道 (GPUPipelineBase pipeline)
用于验证编码器绑定组是否兼容的管道。

1. 如果以下任何条件未能得到满足，则返回 false：
    * pipeline 不能是空的。
    * 管道所使用的所有绑定组必须被设置并与管道布局兼容：对于 pipiline.[[layout]].[[bindGroupLayouts]] 中的每一对（GPUIndex32 index，GPUBindGroupLayout bindGroupLayout）：
        * 让 bindGroup 成为encoder.[[bind_groups]][index]。
        * bindGroup 不能为空。
        * bindGroup.[[layout]] 必须与 bindGroupLayout 的组对等。
    * 对于没有用 minBindingSize 预设的缓冲区绑定，绑定范围必须足够大，以满足最小缓冲区绑定大小。    
    >> ISSUE 26 将这个检查正式化。
    * 编码器绑定组 别名的一个 `可写资源(encoder, pipeline)` 必须是假的。      
否则返回 true。

---

**Validate encoder bind groups(encoder, pipeline)**   

**Arguments:**

**GPUBindingCommandsMixin encoder**       
Encoder whose bind groups are being validated.

**GPUPipelineBase pipeline**      
Pipeline to validate encoders bind groups are compatible with.

1. If any of the following conditions are unsatisfied, return false:
    * pipeline must not be null.
    * All bind groups used by the pipeline must be set and compatible with the pipeline layout: For each pair of (GPUIndex32 index, GPUBindGroupLayout bindGroupLayout) in pipeline.[[layout]].[[bindGroupLayouts]]:
        * Let bindGroup be encoder.[[bind_groups]][index].
        * bindGroup must not be null.
        * bindGroup.[[layout]] must be group-equivalent with bindGroupLayout.
    * For buffer bindings that weren’t prevalidated with minBindingSize, the binding ranges must be large enough for the minimum buffer binding size.
    >> **ISSUE 26** Formalize this check.
    * Encoder bind groups alias a writable resource(encoder, pipeline) must be false.
  
Otherwise return true.


---
---

如果任何可写缓冲区的绑定范围与同一缓冲区的任何其他绑定范围重叠，或者任何可写纹理绑定在纹理子资源中与任何其他纹理绑定（可能使用相同或不同的GPUTextureView对象）重叠，则 **编码器绑定组可别名为一个`可写资源(encoder, pipeline)`**。

参数：

**GPUBindingCommandsMixin 类型的编码器**       
编码器，其绑定组正在被验证。

**GPUPipelineBase 类型的管道**      
用于验证编码器绑定组是否兼容的管道。

---

**Encoder bind groups alias a writable resource**(encoder, pipeline) if any writable buffer binding range overlaps with any other binding range of the same buffer, or any writable texture binding overlaps in texture subresources with any other texture binding (which may use the same or a different GPUTextureView object).

**Arguments:**

**GPUBindingCommandsMixin encoder**         
Encoder whose bind groups are being validated.

**GPUPipelineBase pipeline**        
Pipeline to validate encoders bind groups are compatible with.


---
---

1. 对于 [VERTEX, FRAGMENT, COMPUTE] 中的每个阶段：
    1. 让 bufferBindings 成为一个 (GPUBufferBinding, boolean) 对的列表，其中后者表示该资源是否可以执行写操作。
    2. 让 textureViews 成为一个 (GPUTextureView, boolean) 对的列表，其中后者表示该资源是否可以执行写操作。
    3. 对于 pipeline.[[layout]].[bindGroupLayouts] 中的每一对（GPUIndex32 index，GPUBindGroupLayout bindGroupLayout）：
        1. 让 bindGroupEntries 成为 encoder.[[bind_groups]][index].entries。
        2. 让 bindGroupLayoutEntries 成为 bindGroupLayout.[[descriptor]].entry。
        3. 对于 bindGroupLayoutEntries 中 bindGroupLayoutEntry.visibility 包含阶段的每个 GPUBindGroupEntry bindGroupLayoutEntry：
            1. 让 bindGroupEntry 成为 bindGroupEntries 中 bindGroupEntry.binding 等价于 bindGroupLayoutEntry.binding 的 GPUBindGroupEntry。
            2. 如果 bindGroupEntry.resource 是一个 GPUBufferBinding：
                1. 让 GPUBufferBinding 的资源成为 bindGroupEntry.resource。
                2. 让 resourceWritable 为（bindGroupLayoutEntry.buffer.type == "storage"）。
                3. 对于 bufferBindings 中的每一对（GPUBufferBinding pastResource, boolean pastResourceWritable）：
                    1. 如果（resourceWritable 或 pastResourceWritable）为真，并且pastResource 和 资源是缓冲区绑定的别名，返回真。
                4. 将([resource], resourceWritable) 追加到 bufferBindings 中。        
                否则，如果 bindGroupEntry.resource 是一个 GPUTextureView：
                1. 让 GPUTextureView 资源成为 bindGroupEntry.resource。
                2. 让 resourceWritable 是（bindGroupLayoutEntry.storageTexture.access == "write-only"）。
                3. 如果 bindGroupLayoutEntry.storageTexture 没有被提供，则继续。
                4. 对于 textureViews 中的每一对（GPUTextureView pastResource, boolean pastResourceWritable）、
                    1. 如果（resourceWritable 或 pastResourceWritable）为真，并且pastResource 和 资源是纹理视图的别名，返回真。
                5. 将（[resource], resourceWritable）追加到 textureViews 中。         
              否则，继续。
2. 返回false。

**注意：** 强烈鼓励具体的实现优化这个算法。

---

1. For each stage in [VERTEX, FRAGMENT, COMPUTE]:
    1. Let bufferBindings be a list of (GPUBufferBinding, boolean) pairs, where the latter indicates whether the resource was used as writable.
    2. Let textureViews be a list of (GPUTextureView, boolean) pairs, where the latter indicates whether the resource was used as writable.
    3. For each pair of (GPUIndex32 index, GPUBindGroupLayout bindGroupLayout) in pipeline.[[layout]].[[bindGroupLayouts]]:
        1. Let bindGroupEntries be encoder.[[bind_groups]][index].entries.
        2. Let bindGroupLayoutEntries be bindGroupLayout.[[descriptor]].entries.
        3. For each GPUBindGroupEntry bindGroupLayoutEntry in bindGroupLayoutEntries for which bindGroupLayoutEntry.visibility contains stage:
            1. Let bindGroupEntry be the GPUBindGroupEntry in bindGroupEntries for which bindGroupEntry.binding is equal to bindGroupLayoutEntry.binding.
            2. If bindGroupEntry.resource is a GPUBufferBinding:
                1. Let GPUBufferBinding resource be bindGroupEntry.resource.
                2. Let resourceWritable be (bindGroupLayoutEntry.buffer.type == "storage").
                3. For each pair (GPUBufferBinding pastResource, boolean pastResourceWritable) in bufferBindings:
                    1. If (resourceWritable or pastResourceWritable) is true, and pastResource and resource are buffer-binding-aliasing, return true.
                4. Append ([resource], resourceWritable) to bufferBindings.   
                Otherwise, if bindGroupEntry.resource is a GPUTextureView:
                5. Let GPUTextureView resource be bindGroupEntry.resource.
                6. Let resourceWritable be (bindGroupLayoutEntry.storageTexture.access == "write-only").
                7. If bindGroupLayoutEntry.storageTexture is not provided, continue.
                8. For each pair (GPUTextureView pastResource, boolean pastResourceWritable) in textureViews,
                    1. If (resourceWritable or pastResourceWritable) is true, and pastResource and resource is texture-view-aliasing, return true.
                9. Append ([resource], resourceWritable) to textureViews.       
                Otherwise, continue.
2. Return false.

**NOTE:** Implementations are strongly encouraged to optimize this algorithm.
