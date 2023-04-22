WebGPU

W3C Working Draft, 13 April 2023


More details about this document

This version:

https://www.w3.org/TR/2023/WD-webgpu-20230413/

---
---

# 23\.  操作细节

本节介绍了各种 GPU 操作的细节。

> ISSUE 34 本节内容并不完整。

---

23\. Detailed Operations

This section describes the details of various GPU operations.

> **ISSUE 34** This section is incomplete.

---
---

## 23.1. 转移

**编辑：** 在高层次上描述转移.

---

23.1. Transfer

**Editorial:** describe the transfers at the high level

---
---

## 23.2. 计算

计算操作提供了对 GPU 可编程硬件的直接访问。计算着色器没有着色器阶段的输入或输出，其结果是以副作用的方式将数据写入到绑定为 GPUBufferBindingType."storage" 和 GPUStorageTextureBindingLayout 的存储绑定中。这些操作在 GPUComputePassEncoder 中被编码为：
* dispatchWorkgroups()
* dispatchWorkgroupsIndirect()

**编辑：** 描述计算算法。

如果着色器的执行没有在用户代理确定的合理时间点内结束，那么设备可能会丢失。

---

23.2. Computing

Computing operations provide direct access to GPU’s programmable hardware. Compute shaders do not have shader stage inputs or outputs, their results are side effects from writing data into storage bindings bound as GPUBufferBindingType."storage" and GPUStorageTextureBindingLayout. These operations are encoded within GPUComputePassEncoder as:
* dispatchWorkgroups()
* dispatchWorkgroupsIndirect()

> **Editorial:** describe the computing algorithm

The device may become lost if shader execution does not end in a reasonable amount of time, as determined by the user agent.

---
---

## 23.3. 渲染

渲染是由一组 GPU 操作完成的，这些操作在 GPURenderPassEncoder 中执行，其结果是纹理数据的修改，且由渲染通道附件查看。这些操作由以下函数进行编码：
* draw()
* drawIndexed()
* drawIndirect()
* drawIndexedIndirect()。

注意：渲染是 GPU 的传统用途，并且硬件中有多个固定功能模块做支持。

---

23.3. Rendering

Rendering is done by a set of GPU operations that are executed within GPURenderPassEncoder, and result in modifications of the texture data, viewed by the render pass attachments. These operations are encoded with:
* draw()
* drawIndexed()
* drawIndirect()
* drawIndexedIndirect().

**NOTE:** rendering is the traditional use of GPUs, and is supported by multiple fixed-function blocks in hardware.

---
---

主要的渲染算法如下：

**render(descriptor, drawCall, state)**

**参数：**
* descriptor：当前 GPURenderPipeline 的描述符。
* drawCall：绘制调用参数。
* state：发出绘制调用的 GPURenderCommandsMixin 的 RenderState。

1. 解析索引。参见 § 23.3.1 索引解析算法。       
   让 vertexList 成为 `resolve indices(drawCall, state)` 的结果。
2. 处理顶点。参见 § 23.3.2 顶点处理。           
   执行 `process vertices(vertexList, drawCall, descriptor.vertex, state)` 。
3. 组装基元。参见 § 23.3.3 基元装配。
    执行 `assemble primitives(vertexList, drawCall, descriptor.primitive)` 。
4. 剪切基元。参见 § 23.3.4 基元裁剪。
    让 primitiveList 成为这一阶段的结果。
5. 栅格化。参见 § 23.3.5 栅格化。
    让 rasterizationList 成为 `rasterize(primitiveList, state)` 的结果。
6. 处理片段。参见 § 23.3.6 片段处理。
    收集一个片段列表，该列表是对 rasterizationList 中的每个 rasterPoint 执行 `process fragment(rasterPoint, descriptor.fragment, state)` 的结果。
7. 处理 深度/模板
    **编辑：** 使用片段填写该部分内容。
8. 输出像素。
    **编辑：** 填写该部分内容。

---

The main rendering algorithm:

**render(descriptor, drawCall, state)**

**Arguments:**
* descriptor: Description of the current GPURenderPipeline.
* drawCall: The draw call parameters.
* state: RenderState of the GPURenderCommandsMixin where the draw call is issued.

1. Resolve indices. See § 23.3.1 Index Resolution.    
    Let vertexList be the result of resolve indices(drawCall, state).

2. Process vertices. See § 23.3.2 Vertex Processing.               
    Execute process vertices(vertexList, drawCall, descriptor.vertex, state).

3. Assemble primitives. See § 23.3.3 Primitive Assembly.            
    Execute assemble primitives(vertexList, drawCall, descriptor.primitive).

4. Clip primitives. See § 23.3.4 Primitive Clipping.            
    Let primitiveList be the result of this stage.

5. Rasterize. See § 23.3.5 Rasterization.               
    Let rasterizationList be the result of rasterize(primitiveList, state).

6. Process fragments. See § 23.3.6 Fragment Processing.         
    Gather a list of fragments, resulting from executing process fragment(rasterPoint, descriptor.fragment, state) for each rasterPoint in rasterizationList.

7. Process depth/stencil.       
    > **Editorial:** fill out the section, using fragments
8. Write pixels.        
    > **Editorial:** fill out the section

---
---

### 23.3.1. 索引解析

在渲染的第一阶段，管道为每个实例建立了一个要处理的顶点列表。

`解析索引(drawCall, state)`

**参数：** 
* drawCall：绘制调用参数。
* state：在绘制调用时 GPURenderCommandsMixin 状态的快照。

**返回值：** 整数索引的列表。

---

23.3.1. Index Resolution

At the first stage of rendering, the pipeline builds a list of vertices to process for each instance.

**resolve indices**(drawCall, state)

**Arguments:**
* drawCall: The draw call parameters.
* state: The snapshot of the GPURenderCommandsMixin state at the time of the draw call.

**Returns:** list of integer indices.

---
---

1. 让 vertexIndexList 成为一个空的索引列表。
2. 如果 drawCall 是一个有索引的绘图调用：
    1. 用 drawCall.indexCount 的整数初始化 vertexIndexList。
    2. 对于 i 的范围 0 ... drawCall.indexCount（不包括）：
        1. 让 relativeVertexIndex 成为 `fetch index（i + drawCall.firstIndex, state.[[index_buffer]]）`。
        2. 如果 relativeVertexIndex 有特殊的值 "out of bounds"，则停止并返回空列表。        
        **注意：** 具体实现可以选择在这种情况发生时时，显示一个警告，特别是在很容易检测到的情况下（比如在非直接索引的绘制调用中）。
        3. 将 drawCall.baseVertex + relativeVertexIndex 追加到 vertexIndexList 中。
3. 否则：
    1. 用 drawCall.vertexCount 的整数初始化 vertexIndexList。
    2. 将每个顶点索引列表项 i 设置为 drawCall.firstVertex+i 的值。
4. 返回 vertexIndexList。

**注意：** 在间接绘制调用的情况下，drawCall 的 indexCount、vertexCount和其他属性将从间接缓冲区读取而不是绘制命令本身。

**编辑：** 更好地指定间接命令。

---

1. Let vertexIndexList be an empty list of indices.
2. If drawCall is an indexed draw call:
    1. Initialize the vertexIndexList with drawCall.indexCount integers.
    2. For i in range 0 .. drawCall.indexCount (non-inclusive):
        1. Let relativeVertexIndex be fetch index(i + drawCall.firstIndex, state.[[index_buffer]]).
        2. If relativeVertexIndex has the special value "out of bounds", stop and return the empty list.            
        **NOTE:** Implementations may choose to display a warning when this occurs, especially when it is easy to detect (like in non-indirect indexed draw calls).
        3. Append drawCall.baseVertex + relativeVertexIndex to the vertexIndexList.
3. Otherwise:
    1. Initialize the vertexIndexList with drawCall.vertexCount integers.
    2. Set each vertexIndexList item i to the value drawCall.firstVertex + i.
4. Return vertexIndexList.

**NOTE:** in case of indirect draw calls, the indexCount, vertexCount, and other properties of drawCall are read from the indirect buffer instead of the draw command itself.

**Editorial:** specify indirect commands better.

---
---

**fetch index(i, buffer, offset, format)**

**参数：**
1. i：要获取的顶点索引的索引。
2. state：绘制调用时的 GPURenderCommandsMixin 状态的快照。

**返回值：** 无符号整数或"出界"(out of bounds)

1. 让 indexSize 由 state.[[index_format]] 决定。：
    **"uint16"**        
    2

    **"uint32"**        
    4

2. 如果 state.[[index_buffer_offset]] + li + 1| × indexSize > state.[[index_buffer_size]]，则返回特殊值 "Out of bounds"。
3. 解释 state.[[index_buffer]] 中的数据，从偏移量 state.[[index_buffer_offset]]+ i × indexSize 开始， 大小为 indexSize 字节，作为一个无符号整数并返回。

---

**fetch index**(i, buffer, offset, format)

**Arguments:**
* i: Index of a vertex index to fetch.
* state: The snapshot of the GPURenderCommandsMixin state at the time of the draw call.

**Returns:** unsigned integer or "out of bounds"

1. Let indexSize be defined by the state.[[index_format]]:      
    **"uint16"**        
    2

    **"uint32"**        
    4
2. If state.[[index_buffer_offset]] + li + 1| × indexSize > state.[[index_buffer_size]], return the special value "out of bounds".

3. Interpret the data in state.[[index_buffer]], starting at offset state.[[index_buffer_offset]] + i × indexSize, of size indexSize bytes, as an unsigned integer and return it.

---
---

### 23.3.2. 顶点处理

顶点处理阶段是渲染管道的一个可编程阶段，用于处理顶点的属性数据，并为 §23.3.4 基元剪切产生剪切空间位置，以及为 §23.3.6 片段处理生成其他数据。

`处理顶点(vertexIndexList, drawCall, desc, state)`

**参数：**
* vertexIndexList：要处理的顶点索引列表（可变，通过引用传递）。
* drawCall：绘制调用的参数。
* desc：GPUVertexState 类型的描述符。
* state：在绘制调用时 GPURenderCommandsMixin 状态的快照。

vertexIndexList 中的每个顶点的 vertexIndex，在索引 rawInstanceIndex 中的每个实例中，都是独立处理的。rawInstanceIndex 的范围是 0 到 drawCall.instanceCount - 1，包括此办界值在内。这种处理是平行进行的，且任何副作用，例如写入 GPUBufferBindingType."storage" 绑定，且都可能以任何顺序发生。

---

23.3.2. Vertex Processing

Vertex processing stage is a programmable stage of the render pipeline that processes the vertex attribute data, and produces clip space positions for § 23.3.4 Primitive Clipping, as well as other data for the § 23.3.6 Fragment Processing.

**process vertices**(vertexIndexList, drawCall, desc, state)

**Arguments:**
* vertexIndexList: List of vertex indices to process (mutable, passed by reference).
* drawCall: The draw call parameters.
* desc: The descriptor of type GPUVertexState.
* state: The snapshot of the GPURenderCommandsMixin state at the time of the draw call.

Each vertex vertexIndex in the vertexIndexList, in each instance of index rawInstanceIndex, is processed independently. The rawInstanceIndex is in range from 0 to drawCall.instanceCount - 1, inclusive. This processing happens in parallel, and any side effects, such as writes into GPUBufferBindingType."storage" bindings, may happen in any order.

---
---

1. 让 instanceIndex 成为 rawInstanceIndex + drawCall.firstInstance。
2. 对于 desc.buffers 列表中的每个非空的 vertexBufferLayout：
    1. 让 i 为 这个列表中的缓冲区布局的索引。
    2. 让 vertexBuffer、vertexBufferOffset 和 vertexBufferBindingSize 是状态中第 i 个插槽 state.[[vertex_buffers]] 的 缓冲区、偏移量和大小。
    3. 让vertexElementIndex依赖于vertexBufferLayout.stepMode：
    **"vertex" 顶点**         
    vertexIndex         
    **"instance" 实例**      
    instanceIndex                  
    4. 对于 vertexBufferLayout.attributions 中的每个 attributeDesc：
        1. 让 attributeOffset 成为 vertexBufferOffset + vertexElementIndex * vertexBufferLayout.arrayStride + attributeDesc.offset。
        2. 从顶点缓冲区加载格式为 attributeDesc.format 的属性数据，从偏移量 attributeOffset 开始。各分量按照 x、y、z、w 的顺序从缓冲区内存进行加载。            
        如果此过程中产生了越界访问，则根据 WGSL 的无效内存引用行为来确定最终的结果值。
        3. 可选的（由具体的实现定义的）：如果 attributeOffset + sizeof(attributeDesc.format) > vertexBufferOffset + vertexBufferBindingSize，则清空vertexIndexList 并停止，取消绘制调用。       

        **注意：** 这允许具体的实现在发出绘制调用之前检测索引缓冲区中的越界值，而不是使用无效的内存引用行为。
        4. 根据通道格式规则，将数据转换成着色器可见的格式。         
        ```
        例子 32 

        一个类型为"snorm8x2"、字节值为[0x70, 0xD0]的属性值将被转换为 WGSL 中的 vec2<f32>(0.88, -0.38)。
        ```

        5. 根据着色器类型调整数据大小：
            * 如果两者都是标量，或者都是相同维度的向量，则不需要调整。
            * 如果数据是矢量，但着色器类型是标量，那么只提取第一个分量。
            * 如果两者都是向量，并且数据有更高的维度，那么额外的分量将被删除。
            ```
            例子 33

            一个类型为 "float32x3" 且值为 vec3<f32> (1.0, 2.0, 3.0) 的属性值，如果预期是一个2分量的向量，那么将作为 vec2<f32>(1.0, 2.0) 暴露给着色器。
            ```
            * 如果shader的类型是一个更高维度的向量，或者数据是一个标量，那么缺失的分量将由 vec4<*>(0, 0, 0, 1) 值填充。
            ```
            例子 34

            如果一个类型为 "sint32" 且值为 5 的属性被转换为一个 4分量的向量，那么它将作为 vec4<i32>(5, 0, 0, 1) 暴露给着色器。
            ```
        6. 将数据绑定到顶点着色器的输入位置 attributeDesc.shaderLocation。

---

1. Let instanceIndex be rawInstanceIndex + drawCall.firstInstance.
2. For each non-null vertexBufferLayout in the list of desc.buffers:
    1. Let i be the index of the buffer layout in this list.
    2. Let vertexBuffer, vertexBufferOffset, and vertexBufferBindingSize be the buffer, offset, and size at slot i of state.[[vertex_buffers]].
    3. Let vertexElementIndex be dependent on vertexBufferLayout.stepMode:      
    **"vertex"**        
    vertexIndex         
    **"instance"**      
    instanceIndex                   
    4. For each attributeDesc in vertexBufferLayout.attributes:
        1. Let attributeOffset be vertexBufferOffset + vertexElementIndex * vertexBufferLayout.arrayStride + attributeDesc.offset.
        2. Load the attribute data of format attributeDesc.format from vertexBuffer starting at offset attributeOffset. The components are loaded in the order x, y, z, w from buffer memory.                   
        If this results in an out-of-bounds access, the resulting value is determined according to WGSL’s invalid memory reference behavior.    
        3. Optionally (implementation-defined): If attributeOffset + sizeof(attributeDesc.format) > vertexBufferOffset + vertexBufferBindingSize, empty vertexIndexList and stop, cancelling the draw call.             
        **NOTE:** This allows implementations to detect out-of-bounds values in the index buffer before issuing a draw call, instead of using invalid memory reference behavior.
        4. Convert the data into a shader-visible format, according to channel formats rules.   
        ```
        // EXAMPLE 32
        An attribute of type "snorm8x2" and byte values of [0x70, 0xD0] will be converted to vec2<f32>(0.88, -0.38) in WGSL.
        ```
        5. Adjust the data size to the shader type:
            * if both are scalar, or both are vectors of the same dimensionality, no adjustment is needed.
            * if data is vector but the shader type is scalar, then only the first component is extracted.
            * if both are vectors, and data has a higher dimension, the extra components are dropped.
            ```
            // EXAMPLE 33
            An attribute of type "float32x3" and value vec3<f32>(1.0, 2.0, 3.0) will exposed to the shader as vec2<f32>(1.0, 2.0) if a 2-component vector is expected.
            ```
            * if the shader type is a vector of higher dimensionality, or the data is a scalar, then the missing components are filled from vec4\<*\>(0, 0, 0, 1) value.
            ```
            // EXAMPLE 33
            An attribute of type "sint32" and value 5 will be exposed to the shader as vec4<i32>(5, 0, 0, 1) if a 4-component vector is expected.
            ```
        6. Bind the data to vertex shader input location attributeDesc.shaderLocation.

---
---

3. 对于 state.[[bind_groups]] 中 index 处的每个 GPUBindGroup组：
    1. 对于绑定组中的每个资源 GPUBindingResource：
        1. 让 entry 成为该资源对应的 GPUBindGroupLayoutEntry。
        2. 如果 entry.visibility 包括 VERTEX：
            * 将该资源绑定到组索引下的着色器，并绑定 GPUBindGroupLayoutEntry.binding。
4. 设置着色器的绑定：
    * 将 vertex_index 内建程序， 如果有的话， 设置为 vertexIndex。
    * 将 instance_index 内置程序， 如果有的话， 设置为 instanceIndex。
5. 调用 desc 描述的顶点着色器入口点函数。       
    **注意：** 目标平台会对顶点着色器的调用结果进行缓存。不能保证任何重复超过一次的顶点索引会导致多次调用。同样地，也不能保证一个顶点索引只被处理一次。

    如果着色器的执行没有在用户代理确定的合理时间点内结束，那么设备可能会丢失。

---
3. For each GPUBindGroup group at index in state.[[bind_groups]]:
    1. For each resource GPUBindingResource in the bind group:
        1. Let entry be the corresponding GPUBindGroupLayoutEntry for this resource.
        2. If entry.visibility includes VERTEX:
            * Bind the resource to the shader under group index and binding GPUBindGroupLayoutEntry.binding.
4. Set the shader builtins:
    * Set the vertex_index builtin, if any, to vertexIndex.
    * Set the instance_index builtin, if any, to instanceIndex.
5. Invoke the vertex shader entry point described by desc.

**NOTE:** The target platform caches the results of vertex shader invocations. There is no guarantee that any vertexIndex that repeats more than once will result in multiple invocations. Similarly, there is no guarantee that a single vertexIndex will only be processed once.

The device may become lost if shader execution does not end in a reasonable amount of time, as determined by the user agent.

---
---

### 23.3.3. 基元组装

基元是由 GPU 的一个固定功能阶段而组装的。

**assemble primitives(vertexIndexList, drawCall, desc)**

**参数：**
* vertexIndexList：要处理的顶点索引的列表。
* drawCall：绘制调用的参数。
* desc：GPUPrimitiveState 类型的描述符。

对于每个实例，基元会基于 vertexIndexList，利用已经被着色器处理过的顶点来进行组装。

1. 首先，如果基元拓扑结构是条状的（这意味着 desc.stripIndexFormat 不是未定义的），并且drawCall 是索引的，那么顶点索引列表就会被分割成子列表，使用 desc.stripIndexFormat 的最大值作为分隔符。        

例如：一个具有 "uint16" 类型，值为 [1, 2, 65535, 4, 5, 6] 的顶点索引列表将被分割成子列表[1, 2] 和 [4, 5, 6]。 

---

23.3.3. Primitive Assembly

Primitives are assembled by a fixed-function stage of GPUs.

**assemble primitives**(vertexIndexList, drawCall, desc)

**Arguments:**
* vertexIndexList: List of vertex indices to process.
* drawCall: The draw call parameters.
* desc: The descriptor of type GPUPrimitiveState.

For each instance, the primitives get assembled from the vertices that have been processed by the shaders, based on the vertexIndexList.

1. First, if the primitive topology is a strip, (which means that desc.stripIndexFormat is not undefined) and the drawCall is indexed, the vertexIndexList is split into sub-lists using the maximum value of desc.stripIndexFormat as a separator.

    Example: a vertexIndexList with values [1, 2, 65535, 4, 5, 6] of type "uint16" will be split in sub-lists [1, 2] and [4, 5, 6].

---
---

2. 对于每个子列表 vl，根据 desc.topology 进行 基元生成：

**"line-list" "线-列表"**             
此线条基元由（vl.0，vl.1）组成，然后是（vl.2，vl.3），然后是（vl.4 到 vl.5），等等。每个后续基元都需要2个顶点。

**"line-strip" "线-条"**        
此线条基元由（vl.0, vl.1）组成，然后是（vl.1, vl.2），然后是（vl.2, vl.3），等等。每个后续基元都需要一个顶点。

**"triangle-list" "三角形-列表"**     
此三角形基元由（vl.0, vl.1, vl.2）组成，然后是（vl.3, vl.4, vl.5），然后是（vl.6, vl.7, vl.8），等等。每个后续的基元都需要3个顶点。

**"triangle-strip" "三角形基元"**        
此三角形基元由(vl.0, vl.1, vl.2)，然后(vl.2, vl.1, vl.3)，然后(vl.2, vl.3, vl.4)，然后(vl.4, vl.3, vl.5)等组成。每个后续的基元都需要1个顶点。

**编辑：** 是否应该更正式地定义这个问题？

任何不完整的基元都会被丢弃。

---

1. For each of the sub-lists vl, primitive generation is done according to the desc.topology:

**"line-list"**             
Line primitives are composed from (vl.0, vl.1), then (vl.2, vl.3), then (vl.4 to vl.5), etc. Each subsequent primitive takes 2 vertices.

**"line-strip"**        
Line primitives are composed from (vl.0, vl.1), then (vl.1, vl.2), then (vl.2, vl.3), etc. Each subsequent primitive takes 1 vertex.

**"triangle-list"**     
Triangle primitives are composed from (vl.0, vl.1, vl.2), then (vl.3, vl.4, vl.5), then (vl.6, vl.7, vl.8), etc. Each subsequent primitive takes 3 vertices.

**"triangle-strip"**        
Triangle primitives are composed from (vl.0, vl.1, vl.2), then (vl.2, vl.1, vl.3), then (vl.2, vl.3, vl.4), then (vl.4, vl.3, vl.5), etc. Each subsequent primitive takes 1 vertices.

**Editorial:** should this be defined more formally?

Any incomplete primitives are dropped.

---
---

### 23.3.4. 基元剪裁

顶点着色器必须产生一个内置的 "位置"（类型为vec4<f32>），它表示一个顶点的剪辑位置。

**编辑：** 链接到 WGSL 的内置程序

基元被剪辑到剪辑容积量，对于基元内的任何剪辑位置 p，它由以下不等式定义：
* -p.w ≤ p.x ≤ p.w
* -p.w≤p.y≤p.w
* 0 ≤ p.z ≤ p.w (深度剪裁)

---

23.3.4. Primitive Clipping

Vertex shaders have to produce a built-in "position" (of type vec4\<f32\>), which denotes the clip position of a vertex.

**Editorial:** link to WGSL built-ins

Primitives are clipped to the clip volume, which, for any clip position p inside a primitive, is defined by the following inequalities:
* −p.w ≤ p.x ≤ p.w
* −p.w ≤ p.y ≤ p.w
* 0 ≤ p.z ≤ p.w (depth clipping)

---
---

如果 descriptor.primitive.unclippedDepth 为真，则不应用深度剪裁：剪裁的体积在 z 维度上不受限制。

如果一个基元的每一条边缘都完全位于裁剪体积内，那么该基元就会毫无变化地通过这一阶段。如果基元的边与裁剪体积的边界相交，则相交的边会被沿裁剪体积的边界的新边重新连接起来。对于三角形基元（即 descriptor.primitive.topology 是 "triganle-list" 或 "triangle-strip"），这种重新连接可能会导致在内部将新的顶点引入到多边形中。

---

If descriptor.primitive.unclippedDepth is true, depth clipping is not applied: the clip volume is not bounded in the z dimension.

A primitive passes through this stage unchanged if every one of its edges lie entirely inside the clip volume. If the edges of a primitives intersect the boundary of the clip volume, the intersecting edges are reconnected by new edges that lie along the boundary of the clip volume. For triangular primitives (descriptor.primitive.topology is "triangle-list" or "triangle-strip"), this reconnection may result in introduction of new vertices into the polygon, internally.

---
---

如果一个基元与裁剪体积的边界边缘相交，裁剪的多边形必须包括此边界边缘上的一个点。

如果顶点着色器输出其他浮点值（标量和向量），并符合 "透视" 插值条件，它们也会被剪切。与位于剪切体积内的顶点相关的输出值不受剪切的影响。但是，如果一个基元被剪切，分配给由剪切产生的顶点的输出值也会被剪切。

考虑到顶点 a 和 b 之间的一条边被剪切，产生了顶点c，让我们定义 t 为边上顶点之间的比率：c.p = t × a.p + (1 - t) × b.p，其中 x.p 是一个顶点 x 的输出 裁剪位置。

---

If a primitive intersects an edge of the clip volume’s boundary, the clipped polygon must include a point on this boundary edge.

If the vertex shader outputs other floating-point values (scalars and vectors), qualified with "perspective" interpolation, they also get clipped. The output values associated with a vertex that lies within the clip volume are unaffected by clipping. If a primitive is clipped, however, the output values assigned to vertices produced by clipping are clipped.

Considering an edge between vertices a and b that got clipped, resulting in the vertex c, let’s define t to be the ratio between the edge vertices: c.p = t × a.p + (1 − t) × b.p, where x.p is the output clip position of a vertex x.

---
---

对于每个顶点输出值 "v" 与相应的片段输入，a.v 和 b.v 将分别是 a 和 b 顶点的输出。剪切着色器的输出 c.v 是根据插值限定符产生的：

**"flat" "平面"**           
平坦插值是不受影响，它基于 provoking 激发顶点，也就是基元中的第一个顶点。整个基元的输出值是相同的，并且与激发顶点的顶点输出相匹配：`c.v = provoking vertex.v`

**"linear" "线性"**
内插比例会根据裁剪位置的透视坐标进行调整，从而使内插的结果在屏幕空间中是线性的。

**编辑：** 如果可能的话，在这里提供更多的具体信息

**"perspective" "透视"**
该值在裁剪空间中被线性插值，从而产生透视校正的值：

c.v = t × a.v + (1 - t) × b.v

**编辑：** 链接到 WGSL 中的插值限定词

基元剪裁的结果是一组新的基元，这些基元包含在剪裁体积中。

---

For each vertex output value "v" with a corresponding fragment input, a.v and b.v would be the outputs for a and b vertices respectively. The clipped shader output c.v is produced based on the interpolation qualifier:

**"flat"**          
Flat interpolation is unaffected, and is based on provoking vertex, which is the first vertex in the primitive. The output value is the same for the whole primitive, and matches the vertex output of the provoking vertex: c.v = provoking vertex.v

**"linear"**            
The interpolation ratio gets adjusted against the perspective coordinates of the clip positions, so that the result of interpolation is linear in screen space.

**Editorial:** provide more specifics here, if possible

**"perspective"**           
The value is linearly interpolated in clip space, producing perspective-correct values:

c.v = t × a.v + (1 − t) × b.v

**Editorial:** link to interpolation qualifiers in WGSL

The result of primitive clipping is a new set of primitives, which are contained within the clip volume.

---
---

### 23.3.5. 栅格化

栅格化是一个硬件处理阶段，它将生成的基元映射到帧缓冲区的二维渲染区域，即当前 GPURenderPassEncoder 中的渲染附件集。这个渲染区域被分割成一个均匀的像素网格。

栅格化决定了受一个基元影响的像素集合。在多重采样的情况下，每个像素被进一步分割成 descriptor.multisample.count 个样本。样本中每个像素的位置都是一样的，但本规范中没有定义。

> ISSUE 35 我们是否要在 Vulkan 中强制启用 "标准采样位置"？

---

23.3.5. Rasterization

Rasterization is the hardware processing stage that maps the generated primitives to the 2-dimensional rendering area of the framebuffer - the set of render attachments in the current GPURenderPassEncoder. This rendering area is split into an even grid of pixels.

Rasterization determines the set of pixels affected by a primitive. In case of multi-sampling, each pixel is further split into descriptor.multisample.count samples. The locations of samples are the same for each pixel, but not defined in this spec.

> **ISSUE 35** do we want to force-enable the "Standard sample locations" in Vulkan?


---
---

帧缓冲区的坐标从渲染目标的左上角开始。每个单位完全对应于一个像素。更多信息请参见 § 3.3 坐标系。

让我们定义一个 FragmentDestination 来包含：

**position 位置**                     
帧缓冲空间中的 2D 像素位置

**sampleIndex 采样索引**            
如果 §23.3.10 采样频率着色器被激活，则是一个整数，否则为空

我们还将使用 NDC 的概念--规范化设备坐标。在这个坐标系中，视口的边界在 X 和 Y 上的范围是 -1 到1 ，在 Z 上的范围是 0 到 1 。

---

The framebuffer coordinates start from the top-left corner of the render targets. Each unit corresponds exactly to a pixel. See § 3.3 Coordinate Systems for more information.

Let’s define a FragmentDestination to contain:

**position**        
the 2D pixel position in framebuffer space

**sampleIndex**         
an integer in case § 23.3.10 Sample frequency shading is active, or null otherwise

We’ll also use a notion of NDC - normalized device coordinates. In this coordinate system, the viewport bounds range in X and Y from -1 to 1, and in Z from 0 to 1.

---
---



栅格化产生一个栅格化点（RasterizationPoints）的列表，每个点都包含以下数据：

**destination 目的地**      
指的是 FragmentDestination

**coverageMask 覆盖掩码**           
指的是多样本的覆盖掩码（见 §23.3.11 样本掩码）。

**frontFacing 正面**            
如果它是一个基元的正面上的点，则为真。

**perspectiveDivisor 透视因子**          
指的是在基元内部 内插的 1.0÷W 因子。

**depth 深度**      
指的是 NDC 中的深度

**primitiveVertices 基元顶点**      
指的是构成基元的顶点输出列表

**barycentricCoordinates 重心坐标**         
指的是 § 23.3.5.3 节的 重心坐标

**编辑：** 定义深度计算算法

---

Rasterization produces a list of RasterizationPoints, each containing the following data:

**destination**         
refers to FragmentDestination

**coverageMask**        
refers to multisample coverage mask (see § 23.3.11 Sample Masking)

**frontFacing**         
is true if it’s a point on the front face of a primitive

**perspectiveDivisor**          
refers to interpolated 1.0 ÷ W across the primitive

**depth**       
refers to the depth in NDC

**primitiveVertices**           
refers to the list of vertex outputs forming the primitive

**barycentricCoordinates**      
refers to § 23.3.5.3 Barycentric coordinates

**Editorial:** define the depth computation algorithm

---
---

**rasterize(primitiveList, state)。**

**参数：**  
* primitiveList：要栅格化的基元的列表。
* state：活动的 RenderState。

**返回值：** RasterizationPoint 的列表。

primitiveList 中的每个基元都被独立处理。但是，基元的顺序会影响后面的阶段，例如 深度/模板操作和像素写入。

---

**rasterize**(primitiveList, state)

**Arguments:**
* primitiveList: List of primitives to rasterize.
* state: The active RenderState.

**Returns: **list of RasterizationPoint.

Each primitive in primitiveList is processed independently. However, the order of primitives affects later stages, such as depth/stencil operations and pixel writes.

---
---

1. 首先，被裁剪的顶点被转换为 NDC -- 即归一化设备坐标。假如给定输出位置 p ，NDC 坐标的计算方法是：

`divisor(p) = 1.0 ÷ p.w`

`ndc(p) = vector(p.x÷p.w, p.y÷p.w, p.z÷p.w)`

2. 让 vp 是 state[[viewport]]。然后根据渲染目标的大小，将 NDC 坐标 n 转换为帧缓冲区坐标：

`framebufferCoords(n) = vector(vp.x + 0.5 × (n.x + 1) × vp.width, vp.y + . 5 × (n.y + 1) × vp.height)`

> **编辑：** 同时指定深度转换到视口的位置。

---

1. First, the clipped vertices are transformed into NDC - normalized device coordinates. Given the output position p, the NDC coordinates are computed as:

    divisor(p) = 1.0 ÷ p.w

    ndc(p) = vector(p.x ÷ p.w, p.y ÷ p.w, p.z ÷ p.w)

2. Let vp be state.[[viewport]]. Then the NDC coordinates n are converted into framebuffer coordinates, based on the size of the render targets:

    framebufferCoords(n) = vector(vp.x + 0.5 × (n.x + 1) × vp.width, vp.y + .5 × (n.y + 1) × vp.height)

    **Editorial:** specify the depth translation into viewport as well

---
---

3. 让 rasterizationPoints 成为一个空列表。

**编辑：** 定义如何为每个栅格化点分配一个插值因子 `divisor(p)` 和 `framebufferCoords(n)`，以及其他属性。

4. 根据 primitive.topology，以特定的栅格化算法进行：

**"point-list"**         
**"点-列表"**             
渲染为点，如果没有被 §23.3.4 基元剪裁过滤，则进入 §23.3.5.1 点栅格化。

**"line-list" or "line-strip"**         
**"线-列表" 或 "线-条"**           
被 §23.3.4 基元剪裁后的线条进入 §23.3.5.2 线栅格化。

**"triangle-list" or "triangle-strip"**             
**"三角形列表" 或 "三角形条带"**            
在 §23.3.4 基无剪切中产生的多边形将进入 §23.3.5.4 多边形栅格化。

5. 从栅格化 Points 中移除所有 rp.destination.position 在 state.[[scissorRect]] 之外的点。
6. 返回 rasterizationPoints。

---

3. Let rasterizationPoints be an empty list.

    **Editorial:** specify that each rasterization point gets assigned an interpolated divisor(p) and framebufferCoords(n), as well as the other attributes.

4. Proceed with a specific rasterization algorithm, depending on primitive.topology:

**"point-list"**        
The point, if not filtered by § 23.3.4 Primitive Clipping, goes into § 23.3.5.1 Point Rasterization.

**"line-list" or "line-strip"**         
The line cut by § 23.3.4 Primitive Clipping goes into § 23.3.5.2 Line Rasterization.

**"triangle-list" or "triangle-strip"**     
The polygon produced in § 23.3.4 Primitive Clipping goes into § 23.3.5.4 Polygon Rasterization.

5. Remove all the points rp from rasterizationPoints that have rp.destination.position outside of state.[[scissorRect]].

6. Return rasterizationPoints.

---
---

#### 23.3.5.1. 点栅格化
在包含点的帧缓冲坐标的像素中 选择一个单一的 FragmentDestination。

覆盖掩码取决于多重取样模式：

**sample-frequency**    
**采样频率**      
coverageMask = 1 ≪ sampleIndex

**pixel-frequency multi-sampling**      
**像素频率的多重取样**      
coverageMask = 1 ≪ descriptor.multisample.count - 1

**no multi-sampling**       
**不进行多重取样**      
coverMask = 1

---

23.3.5.1. Point Rasterization

A single FragmentDestination is selected within the pixel containing the framebuffer coordinates of the point.

The coverage mask depends on multi-sampling mode:

**sample-frequency**    
coverageMask = 1 ≪ sampleIndex

**pixel-frequency multi-sampling**      
coverageMask = 1 ≪ descriptor.multisample.count − 1

**no multi-sampling**       
coverageMask = 1

---
---

#### 23.3.5.2.线条栅格化

**编辑：** 填写本节。

---

23.3.5.2. Line Rasterization

**Editorial:** fill out this section

---
---

#### 23.3.5.3. 重性坐标

**Barycentric coordinates** Barycentric坐标 是一个由 n 个数字 bi 组成的列表，它是为帧缓冲空间中具有 n 个顶点 vi 所组成的凸多边形内的一个点 p 定义的。每个 bi 的范围是 0 到 1，包括边界值在内，并且代表与顶点 vi 的接近程度。它们的总和始终是常数：

∑（bi）= 1

这些坐标唯一地指定了多边形内（或其边界上）的任何点 p：

p = ∑ (bi × pi)

对于一个有 3 个顶点的多边形 -- 一个三角形，任何点 p 的 arycentric 坐标可按以下方式计算：

***公式格式请参见原文本***

Apolygon = A(v1, v2, v3) b1 = A(p, b2, b3) ÷ Apolygon b2 = A(b1, p, b3) ÷ Apolygon b3 = A(b1, b2, p) ÷ Apolygon

其中A（点的列表）是给定的顶点集所构成的多边形的面积。

对于有 3 个以上顶点的多边形，确切的算法取决于具体的实现。其中一个可能的实现方式是对多边形进行三角形化计算，并根据一个点所处的三角形来计算该点的偏心值。

---

23.3.5.3. Barycentric coordinates

Barycentric coordinates is a list of n numbers bi, defined for a point p inside a convex polygon with n vertices vi in framebuffer space. Each bi is in range 0 to 1, inclusive, and represents the proximity to vertex vi. Their sum is always constant:

∑ (bi) = 1

These coordinates uniquely specify any point p within the polygon (or on its boundary) as:

p = ∑ (bi × pi)

For a polygon with 3 vertices - a triangle, barycentric coordinates of any point p can be computed as follows:

Apolygon = A(v1, v2, v3) b1 = A(p, b2, b3) ÷ Apolygon b2 = A(b1, p, b3) ÷ Apolygon b3 = A(b1, b2, p) ÷ Apolygon

Where A(list of points) is the area of the polygon with the given set of vertices.

For polygons with more than 3 vertices, the exact algorithm is implementation-dependent. One of the possible implementations is to triangulate the polygon and compute the barycentrics of a point based on the triangle it falls into.

---
---

#### 23.3.5.4. 多边形栅格化
如果一个多边形的方向是朝向投影，那么它就是正向的，否则，多边形是面向后的。

`栅格化多边形()`

**参数：**

**返回值：** RasterizationPoint 的列表。

---

23.3.5.4. Polygon Rasterization

A polygon is front-facing if it’s oriented towards the projection. Otherwise, the polygon is back-facing.

**rasterize polygon()**

**Arguments:**

**Returns:** list of RasterizationPoint.

---
---

1. 让 rasterizationPoints 成为一个空列表。
2. 让 v(i) 为 n 个顶点的栅格化多边形中的 第 i 号顶点（从1开始）的帧缓冲坐标。

**注意：** 本节使用术语 "多边形" 而不是 "三角形"，因为 §23.3.4 基元剪裁 阶段可能引入了额外的顶点。这一点是应用程序无法观察到的。

3. 确定多边形是否是正面的，这取决于多边形在帧缓冲区坐标中所占面积的符号：

area = 0.5 × ((v1.x × vn.y - vn.x × v1.y) + ∑ (vi+1.x × vi.y - vi.x × vi+1.y)

面积的符号是根据 primitive.frontFace 来解释的：

**"ccw"**           
面积 > 0被认为是正面的、朝前的，否则是背面的、朝后的。

**"cw"**        
面积 < 0被认为是正面的、朝前的，否则是背面的、朝后的。

---

1. Let rasterizationPoints be an empty list.
2. Let v(i) be the framebuffer coordinates for the clipped vertex number i (starting with 1) in a rasterized polygon of n vertices.

    NOTE: this section uses the term "polygon" instead of a "triangle", since § 23.3.4 Primitive Clipping stage may have introduced additional vertices. This is non-observable by the application.

3. Determine if the polygon is front-facing, which depends on the sign of the area occupied by the polygon in framebuffer coordinates:

    area = 0.5 × ((v1.x × vn.y − vn.x × v1.y) + ∑ (vi+1.x × vi.y − vi.x × vi+1.y))

    The sign of area is interpreted based on the primitive.frontFace:

    **"ccw"**       
    area > 0 is considered front-facing, otherwise back-facing

    **"cw"**        
    area < 0 is considered front-facing, otherwise back-facing

---
---

4. 基于 primitive.cullMode 进行挑选(cull)：

    **"none"**          
    **"无"**        
    所有多边形都通过此测试。            

    **"front"**             
    **"正面"**      
    正面的多边形被丢弃，在渲染管道的后期阶段不进行处理。
    
    **"back"**      
    **"背面"**      
    背面的多边形被丢弃。

---

4. Cull based on primitive.cullMode:

    **"none"**          
    All polygons pass this test.

    **"front"**             
    The front-facing polygons are discarded, and do not process in later stages of the render pipeline.

    **"back"**      
    The back-facing polygons are discarded.

---
---

5. 在帧缓冲空间中确定多边形内部的一组片段 -- 这些是为每个片段操作安排其位置。这个操作被称为 "点采样"。其逻辑是基于 descriptor.multisample 的：

    **禁用**        
    片段与像素中心有关。也就是说，所有坐标为 C 的点，其中 fract(C) = vector2(0.5, 0.5)在帧缓冲区空间中，被包围在多边形中，且都被包括在内。如果一个像素中心在多边形的边缘，其是否被包含就没有定义。

    **注意：** 这成为光栅器的一个精度问题。

    **启用**            
    每个像素都与 descriptor.multisample.count 位置相关联，这些位置是具体的实现定义的。这些位置是有顺序的，而且这个列表对帧缓冲区的每个像素都是一样的。每个位置对应于多采样帧缓冲区的一个片段。

    栅格化器建立了一个每个像素内被击中的位置的掩码，并作为 "采样掩码" 内置到片段着色器中。

---

5. Determine a set of fragments inside the polygon in framebuffer space - these are locations scheduled for the per-fragment operations. This operation is known as "point sampling". The logic is based on descriptor.multisample:

    **disabled**        
    Fragments are associated with pixel centers. That is, all the points with coordinates C, where fract(C) = vector2(0.5, 0.5) in the framebuffer space, enclosed into the polygon, are included. If a pixel center is on the edge of the polygon, whether or not it’s included is not defined.

    **NOTE:** this becomes a subject of precision for the rasterizer.

    **enabled**
    Each pixel is associated with descriptor.multisample.count locations, which are implementation-defined. The locations are ordered, and the list is the same for each pixel of the framebuffer. Each location corresponds to one fragment in the multisampled framebuffer.

    The rasterizer builds a mask of locations being hit inside each pixel and provides is as "sample-mask" built-in to the fragment shader.

---
---

6. 对于每个生成的类型为 FragmentDestination 的片段：
    1. 让 rp 成为一个新的 RasterizationPoint 对象
    2. 按照 §23.3.5.3 重心坐标，计算该片段的 arycentric坐标列表 b。将 rp.barycentricCoordinates 设为 b。
    3. 让 di 是 vi 的深度值。       
        **编辑：** 定义这个值是如何构建的。
    4. 设 rp.depth 设置为为 ∑(bi×di)
    5. 将 rp 追加到 rasterizationPoints。
7. 返回rasterizationPoints。

---

6. For each produced fragment of type FragmentDestination:
    1. Let rp be a new RasterizationPoint object
    2. Compute the list b as § 23.3.5.3 Barycentric coordinates of that fragment. Set rp.barycentricCoordinates to b.
    3. Let di be the depth value of vi.     
    **Editorial:** define how this value is constructed.
    4. Set rp.depth to ∑ (bi × di)
    5. Append rp to rasterizationPoints.

7. Return rasterizationPoints.

---
---

### 23.3.6. 片段处理

片段处理阶段是渲染管道的一个可编程阶段，它计算要写入渲染目标中的片段数据（通常是一种颜色）。

这个阶段为每个 RasterizationPoint 产生一个 Fragment：

* destination 指的是 FragmentDestination。
* coverageMask 指的是多样本覆盖掩码（见 §23.3.11 样本掩码）。
* depth 指的是 NDC 坐标中的深度。
* colors 指的是 颜色值的列表，在 colorAttachments 中，每个目标都有一个颜色值。

---

23.3.6. Fragment Processing

The fragment processing stage is a programmable stage of the render pipeline that computes the fragment data (often a color) to be written into render targets.

This stage produces a Fragment for each RasterizationPoint:
* destination refers to FragmentDestination.
* coverageMask refers to multisample coverage mask (see § 23.3.11 Sample Masking).
* depth refers to the depth in NDC coordinates.
* colors refers to the list of color values, one for each target in colorAttachments.

---
---

**Process fragment(rp, desc, state)**

**参数：**
* rp：栅格化点，由 §23.3.5 栅格化产生。
* desc：GPUFragmentState 类型的描述符。
* state：活动的 RenderState。

**返回值：** Fragment 或 null (片段或空)。

1. 让 fragment 成为一个新的 Fragment 对象。
2. 将 fragment.destination 设置为 rp.destination。
3. 将 fragment.coverageMask 设置为 rp.coverageMask。
4. 将 fragment.depth 设置为 rp.depth。

---

**process fragment(rp, desc, state)**

**Arguments:**
* rp: The RasterizationPoint, produced by § 23.3.5 Rasterization.
* desc: The descriptor of type GPUFragmentState.
* state: The active RenderState.

**Returns: **Fragment or null.

1. Let fragment be a new Fragment object.
2. Set fragment.destination to rp.destination.
3. Set fragment.coverageMask to rp.coverageMask.
4. Set fragment.depth to rp.depth.

---
---

5. 如果 desc 不是空的：
    1. 设置着色器输入的内置参数。对于入口点函数的每个非复合参数，被注释为内置参数的，根据注释设置其值：

    **position**                     
    vec4\<f32\>(rp.destination.position, rp.depth, rp.perspectiveDivisor)

    **front_facing**        
    rp.frontFacing

    **sample_index**        
    rp.destination.sampleIndex

    **sample_mask**     
    rp.coverageMask

    2. 对于用户指定的片段阶段的每个着色器阶段的输入：
        1. 让 value 成为片段输入的插值，基于 rp.barycentricCoordinates、rp.primitiveVertices 以及 输入的插值限定词。
        
            **编辑** 描述精确的方程。

        2. 将相应的片段着色器的位置输入值设置为 value。

    3. 调用 desc 所描述的片段着色器入口点函数。         
        如果着色器的执行没有在用户代理确定的合理时间点内结束，那么设备可能会丢失。

    4. 如果该片段发出丢弃操作，则返回 null。
    5. 将 fragment.colors 设置为 从着色器中输出的用户指定的着色器阶段输出值。
    6. 取出着色器输出的内置值：
        1. 如果 frag_depth 内置值 是由着色器产生的值：
            1. 让 vp 成为 state.[[viewport]]。
            2. 将 fragment.depth 设置为 `clamp(value, vp.minDepth, vp.maxDepth)`。
    7. 如果 sample_mask 的内置值 是由着色器产生的，且被做为 value：
        1. 将 fragment.coverageMask 设置为 fragment.coverageMask ∧ value。

        否则我们就处于 § 23.3.8 无颜色输出模式，并且 fragment.colors 为空(empty)。
6. 返回 fragment。

对片段的处理是平行进行的，而其产生的任何副作用，如写入 GPUBufferBindingType."storage" 的绑定，可能以任何顺序发生。

---

1. If desc is not null:
    1. Set the shader input builtins. For each non-composite argument of the entry point, annotated as a builtin, set its value based on the annotation:

        **position**        
        vec4\<f32\>(rp.destination.position, rp.depth, rp.perspectiveDivisor)

        **front_facing**        
        rp.frontFacing

        **sample_index**        
        rp.destination.sampleIndex

        **sample_mask**     
        rp.coverageMask
    2. For each user-specified shader stage input of the fragment stage:
        1. Let value be the interpolated fragment input, based on rp.barycentricCoordinates, rp.primitiveVertices, and the interpolation qualifier on the input.

            **Editorial:** describe the exact equations.

        2. Set the corresponding fragment shader location input to value.

    3. Invoke the fragment shader entry point described by desc.

        The device may become lost if shader execution does not end in a reasonable amount of time, as determined by the user agent.

    4. If the fragment issued discard, return null.   
    5. Set fragment.colors to the user-specified shader stage output values from the shader.
    6. Take the shader output builtins:
        1. If frag_depth builtin is produced by the shader as value:
            1. Let vp be state.[[viewport]].
            2. Set fragment.depth to clamp(value, vp.minDepth, vp.maxDepth).
    7. If sample_mask builtin is produced by the shader as value:
        1. Set fragment.coverageMask to fragment.coverageMask ∧ value.

        Otherwise we are in § 23.3.8 No Color Output mode, and fragment.colors is empty.
2. Return fragment.

Processing of fragments happens in parallel, while any side effects, such as writes into GPUBufferBindingType."storage" bindings, may happen in any order.

---
---

### 23.3.7. 输出合并

**编辑：** 填写本节

在这个阶段的深度输入值，如果有的话，会被钳制在当前 [[viewport]] 的深度范围内（无论片段着色器阶段是否写了 frag_depth 内置值）。

---

### 23.3.7. Output Merging

**Editorial:** fill out this section

The depth input to this stage, if any, is clamped to the current [[viewport]] depth range (regardless of whether the fragment shader stage writes the frag_depth builtin).

---
---

### 23.3.8. 无色彩输出

在无颜色输出模式下，管道不产生任何颜色附件输出。

管道仍然会执行栅格化，并根据顶点位置输出值而产生其相应的深度值。仍然可以使用深度测试和模板操作。

---

23.3.8. No Color Output

In no-color-output mode, pipeline does not produce any color attachment outputs.

The pipeline still performs rasterization and produces depth values based on the vertex position output. The depth testing and stencil operations can still be used.

---
---

### 23.3.9. 阿尔法到覆盖

在 alpha-to-coverage 模式下，基于 @location(0) 处 的片段着色器输出值的 alpha 分量，生成 MSAA 样本的额外 alpha-to-coverage 遮罩。

产生额外遮罩的算法与平台有关，对于不同的像素可能会有所不同。但它保证
* 如果 alpha ≤ 0.0，结果是 0x0
* 如果 alpha ≥ 1.0，结果是 0xFFFFFFFF。
* 如果 alpha 大于其他的 alpha1，那么产生的样本掩码 至少有与 alpha1 的掩码一样多的比特位被设置为 1

---

23.3.9. Alpha to Coverage

In alpha-to-coverage mode, an additional alpha-to-coverage mask of MSAA samples is generated based on the alpha component of the fragment shader output value at @location(0).

The algorithm of producing the extra mask is platform-dependent and can vary for different pixels. It guarantees that:
* if alpha ≤ 0.0, the result is 0x0
* if alpha ≥ 1.0, the result is 0xFFFFFFFF
* if alpha is greater than some other alpha1, then the produced sample mask has at least as many bits set to 1 as the mask for alpha1

---
---

### 23.3.10. 样本频率着色器

**编辑：** 填写该部分内容

---

23.3.10. Sample frequency shading

**Editorial:** fill out the section

---
---

### 23.3.11. 采样遮罩

一个像素的最终样本遮罩是这样计算的：光栅化遮罩 & 遮罩 & 着色器输出遮罩 (rasterization mask & mask & shader-output mask)。

只有遮罩的低计数位会被考虑。

如果最终样本遮罩的位置 N 的最小有效位的值为 "0"，那么对片段着色器的所有附件的样本颜色输出（对应于样本N）将被丢弃。同时，在深度/模板附件的相关样本上不会执行深度测试或模板操作。

**注意：** 样本 N 的颜色输出是由当前像素的 SV_SampleIndex == N 的片段着色器执行产生的。如果片段着色器没有使用这种语义，那么它对每个像素只会执行一次。

---

23.3.11. Sample Masking

The final sample mask for a pixel is computed as: rasterization mask & mask & shader-output mask.

Only the lower count bits of the mask are considered.

If the least-significant bit at position N of the final sample mask has value of "0", the sample color outputs (corresponding to sample N) to all attachments of the fragment shader are discarded. Also, no depth test or stencil operations are executed on the relevant samples of the depth-stencil attachment.

**NOTE:** the color output for sample N is produced by the fragment shader execution with SV_SampleIndex == N for the current pixel. If the fragment shader doesn’t use this semantics, it’s only executed once per pixel.

---
---

栅格化遮罩是在栅格化阶段产生，基于栅格化多边形的形状。包括在形状中的样本，其所在遮罩中的相应比特位被设置为 1。

着色器输出掩码采用片段着色器中 "sample_mask" 的内置输出值。如果内建程序没有从片段着色器中输出，并且启用了 alphaToCoverageEnabled ，那么着色器输出掩码就会变成 alpha-to-coverage 掩码，否则，它的默认值为 0xFFFFFFFF。

---

The rasterization mask is produced by the rasterization stage, based on the shape of the rasterized polygon. The samples included in the shape get the relevant bits 1 in the mask.

The shader-output mask takes the output value of "sample_mask" builtin in the fragment shader. If the builtin is not output from the fragment shader, and alphaToCoverageEnabled is enabled, the shader-output mask becomes the alpha-to-coverage mask. Otherwise, it defaults to 0xFFFFFFFF.

