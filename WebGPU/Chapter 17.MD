WebGPU

W3C Working Draft, 6 April 2023

More details about this document

This version:

https://www.w3.org/TR/2023/WD-webgpu-20230406/

---
---
***译注：本节中有些单词存在两种翻译，备注如下：***      
pass: 管道 或者 通道。

---
---
17\. 渲染管道

17.1. GPU渲染管道编码器

```
[Exposed=(Window, DedicatedWorker), SecureContext]
interface GPURenderPassEncoder {
    undefined setViewport(float x, float y,
        float width, float height,
        float minDepth, float maxDepth);

    undefined setScissorRect(GPUIntegerCoordinate x, GPUIntegerCoordinate y,
                        GPUIntegerCoordinate width, GPUIntegerCoordinate height);

    undefined setBlendConstant(GPUColor color);
    undefined setStencilReference(GPUStencilValue reference);

    undefined beginOcclusionQuery(GPUSize32 queryIndex);
    undefined endOcclusionQuery();

    undefined executeBundles(sequence<GPURenderBundle> bundles);
    undefined end();
};
GPURenderPassEncoder includes GPUObjectBase;
GPURenderPassEncoder includes GPUCommandsMixin;
GPURenderPassEncoder includes GPUDebugCommandsMixin;
GPURenderPassEncoder includes GPUBindingCommandsMixin;
GPURenderPassEncoder includes GPURenderCommandsMixin;
```

---

17\. Render Passes

17.1. GPURenderPassEncoder

```
[Exposed=(Window, DedicatedWorker), SecureContext]
interface GPURenderPassEncoder {
    undefined setViewport(float x, float y,
        float width, float height,
        float minDepth, float maxDepth);

    undefined setScissorRect(GPUIntegerCoordinate x, GPUIntegerCoordinate y,
                        GPUIntegerCoordinate width, GPUIntegerCoordinate height);

    undefined setBlendConstant(GPUColor color);
    undefined setStencilReference(GPUStencilValue reference);

    undefined beginOcclusionQuery(GPUSize32 queryIndex);
    undefined endOcclusionQuery();

    undefined executeBundles(sequence<GPURenderBundle> bundles);
    undefined end();
};
GPURenderPassEncoder includes GPUObjectBase;
GPURenderPassEncoder includes GPUCommandsMixin;
GPURenderPassEncoder includes GPUDebugCommandsMixin;
GPURenderPassEncoder includes GPUBindingCommandsMixin;
GPURenderPassEncoder includes GPURenderCommandsMixin;
```

---
---

GPURenderPassEncoder 有以下内部插槽，用于在编码时进行验证：

**[[command_encoder]]，类型为：GPUCommandEncoder，只读**    
表示要创建实例 this 的渲染通道编码器的 GPUCommandEncoder。

**[[attachment_size]], 只读**       
设置为以下范围：        
* width, height = 该管道的渲染附件的尺寸

**[[occlusion_query_set]], 类型为： GPUQuerySet, 只读**     
用于存储管道的闭塞查询结果的 GPUQuerySet，它在管道创建时用 GPURenderPassDescriptor.occlusionQuerySet 进行初始化。

**[[occlusion_query_active]], 类型为： boolean**        
管道的 [[occlusion_query_set]] 是否被写入。

**[[endTimestampWrites]], 类型为：GPURenderPassTimestampWrites, 只读**      
在管道结束时需要执行的时间戳附件。

**[[maxDrawCount]] 类型为：GPUSize64, 只读**        
此管道中允许的最大绘制次数。

---

GPURenderPassEncoder has the following internal slots used for validation while encoding:

**[[command_encoder]], of type GPUCommandEncoder, readonly**        
The GPUCommandEncoder that created this render pass encoder.

**[[attachment_size]], readonly**       
Set to the following extents:
* width, height = the dimensions of the pass’s render attachments

**[[occlusion_query_set]], of type GPUQuerySet, readonly**      
The GPUQuerySet to store occlusion query results for the pass, which is initialized with GPURenderPassDescriptor.occlusionQuerySet at pass creation time.

**[[occlusion_query_active]], of type boolean**         
Whether the pass’s [[occlusion_query_set]] is being written.

**[[endTimestampWrites]], of type GPURenderPassTimestampWrites, readonly**          
The timestamp attachments which need to be executed when the pass ends.

**[[maxDrawCount]] of type GPUSize64, readonly**            
The maximum number of draws allowed in this pass.

---
---

当作为 GPUCommandBuffer 的一部分执行编码渲染管道命令时，一个内部的 RenderState 对象被用来跟踪渲染所需的当前状态。

RenderState 包含以下用于执行渲染命令的内部插槽：

**[[occlusionQueryIndex]]，类型为：GPUSize32**      
[[occlusion_query_set]] 中的索引，用来存储闭塞查询结果。

**[[viewport]]**        
当前视口的矩形和深度范围， 初始设置为以下值：
* x, y = 0.0, 0.0
* width, height = 管道渲染目标的尺寸
* minDepth, maxDepth = 0.0, 1.0

**[[scissorRect]]**     
当前的裁剪的矩形范围，初始设置为以下值：
* x, y = 0, 0
* width, height = 管道渲染目标的尺寸

**[[blendConstant]]，类型为： GPUColor**
当前的混合常量值，最初为[0, 0, 0, 0]。

**[[stencilReference]], 类型为： GPUStencilValue**      
当前的模板参考值，最初为 0。

---

When executing encoded render pass commands as part of a GPUCommandBuffer, an internal RenderState object is used to track the current state required for rendering.

RenderState contains the following internal slots used for execution of rendering commands:

**[[occlusionQueryIndex]], of type GPUSize32**          
The index into [[occlusion_query_set]] at which to store the occlusion query results.

**[[viewport]]**        
Current viewport rectangle and depth range. Initially set to the following values:
* x, y = 0.0, 0.0
* width, height = the dimensions of the pass’s render targets
* minDepth, maxDepth = 0.0, 1.0

**[[scissorRect]]**         
Current scissor rectangle. Initially set to the following values:
* x, y = 0, 0
* width, height = the dimensions of the pass’s render targets

**[[blendConstant]], of type GPUColor**         
Current blend constant value, initially [0, 0, 0, 0].

**[[stencilReference]], of type GPUStencilValue**       
Current stencil reference value, initially 0.

---
---

### 17.1.1. 渲染管道编码器的创建

```
enum GPURenderPassTimestampLocation {
    "beginning",
    "end"
};

dictionary GPURenderPassTimestampWrite {
    required GPUQuerySet querySet;
    required GPUSize32 queryIndex;
    required GPURenderPassTimestampLocation location;
};

typedef sequence<GPURenderPassTimestampWrite> GPURenderPassTimestampWrites;

dictionary GPURenderPassDescriptor : GPUObjectDescriptorBase {
    required sequence<GPURenderPassColorAttachment?> colorAttachments;
    GPURenderPassDepthStencilAttachment depthStencilAttachment;
    GPUQuerySet occlusionQuerySet;
    GPURenderPassTimestampWrites timestampWrites = [];
    GPUSize64 maxDrawCount = 50000000;
};
```

---

17.1.1. Render Pass Encoder Creation

```
enum GPURenderPassTimestampLocation {
    "beginning",
    "end"
};

dictionary GPURenderPassTimestampWrite {
    required GPUQuerySet querySet;
    required GPUSize32 queryIndex;
    required GPURenderPassTimestampLocation location;
};

typedef sequence<GPURenderPassTimestampWrite> GPURenderPassTimestampWrites;

dictionary GPURenderPassDescriptor : GPUObjectDescriptorBase {
    required sequence<GPURenderPassColorAttachment?> colorAttachments;
    GPURenderPassDepthStencilAttachment depthStencilAttachment;
    GPUQuerySet occlusionQuerySet;
    GPURenderPassTimestampWrites timestampWrites = [];
    GPUSize64 maxDrawCount = 50000000;
};
```

---
---

**colorAttachments, 类型为： sequence\<GPURenderPassColorAttachment?\>.**       
此序列中的 GPURenderPassColorAttachment 值的集合，定义了当执行此渲染管道时那个颜色附件会输出。

由于使用上的兼容性的原因，任何颜色附件都不能是另一个附件的别名，或者是渲染管道内使用的任何资源的别名。

**depthStencilAttachment, 类型为： GPURenderPassDepthStencilAttachment**        
GPURenderPassDepthStencilAttachment（GPU渲染管道深度/模板附件），定义了深度/模板附件，在执行此渲染管道时，将用于输出或进行碰撞测试。

由于使用上的兼容性的原因，任何可写的深度/模板附件都不能是其他附件的别名，或者是渲染管道内使用的任何资源的别名。

**occlusionQuerySet，类型为： GPUQuerySet**          
GPUQuerySet 的值定义了该管道的闭塞查询结果的存储位置。

**timestampWrites，类型为：GPURenderPassTimestampWrites，默认为[] 。**          
一系列 GPURenderPassTimestampWrite 值，定义了这个通道的时间戳值将被何处和何时写入。

**maxDrawCount，类型为：GPUSize64，默认为 50000000。**     
渲染过程中调用的最大数量。某些具体的实现用于确定在渲染管道之前注入的工作任务的大小。保持默认值是一个很好的默认选择，除非已知会有更多的绘制任务要调用。

---

**colorAttachments, of type sequence\<GPURenderPassColorAttachment?\>**           
The set of GPURenderPassColorAttachment values in this sequence defines which color attachments will be output to when executing this render pass.

Due to usage compatibility, no color attachment may alias another attachment or any resource used inside the render pass.

**depthStencilAttachment, of type GPURenderPassDepthStencilAttachment**         
The GPURenderPassDepthStencilAttachment value that defines the depth/stencil attachment that will be output to and tested against when executing this render pass.

Due to usage compatibility, no writable depth/stencil attachment may alias another attachment or any resource used inside the render pass.

**occlusionQuerySet, of type GPUQuerySet**          
The GPUQuerySet value defines where the occlusion query results will be stored for this pass.

**timestampWrites, of type GPURenderPassTimestampWrites, defaulting to []**     
A sequence of GPURenderPassTimestampWrite values defines where and when timestamp values will be written for this pass.

**maxDrawCount, of type GPUSize64, defaulting to 50000000**         
The maximum number of draw calls that will be done in the render pass. Used by some implementations to size work injected before the render pass. Keeping the default value is a good default, unless it is known that more draw calls will be done.

---
---

**有效的使用方法**  
实例 this 所给定的 GPUDevice 设备和 GPURenderPassDescriptor，适用以下验证规则：
1. this.colorAttachments.length 必须是小于等于 device.[[limit]].maxColorAttachments。
2. 对于 this.colorAttachments 中每个非空的 colorAttachments：
    1. colorAttachment 必须符合 GPURenderPassColorAttachment 的有效使用规则。
3. 如果提供了 this.depthStencilAttachment：
    1. this.depthStencilAttachment 必须符合 GPURenderPassDepthStencilAttachment 的有效使用规则。
4. 必须至少存在一个附件，要么：
    * this.colorAttachments 中的一个非空值，或
    * 一个 this.depthStencilAttachment。
5. 验证`GPURenderPassDescriptor的每一个样本的颜色附件字节（device，this.colorAttachments）` 要返回成功。
6. 所有视图必须在 this.colorAttachments 的非空成员中，以及如果 this.depthStencilAttachment.view 存在，则必须具有相等的样本数。
7. 对于在 this.colorAttachments 的非空成员中的每个视图和 如果存在的 this.depthStencilAttachment.view， 则 [[renderExtent]] 必须匹配。
8. 如果 this.occlusionQuerySet 不是空的：
    1. this.occlusionQuerySet.type 必须是 occlusion。
9. 验证 `timestampWrites(device, this.timestampWrites)` 必须返回 true。
10. this.timestampWrites 中没有两个条目可以有相同的[querySet, queryIndex]对。

---

**Valid Usage**         

Given a GPUDevice device and GPURenderPassDescriptor this, the following validation rules apply:
1. this.colorAttachments.length must be ≤ device.[[limits]].maxColorAttachments
2. For each non-null colorAttachment in this.colorAttachments:
    1. colorAttachment must meet the GPURenderPassColorAttachment Valid Usage rules.
3. If this.depthStencilAttachment is provided:
    1. this.depthStencilAttachment must meet the GPURenderPassDepthStencilAttachment Valid Usage rules.
4. There must exist at least one attachment, either:
    * A non-null value in this.colorAttachments, or
    * A this.depthStencilAttachment.
5. Validating GPURenderPassDescriptor’s color attachment bytes per sample(device, this.colorAttachments) succeeds.
6. All views in non-null members of this.colorAttachments, and this.depthStencilAttachment.view if present, must have equal sampleCounts.
7. For each view in non-null members of this.colorAttachments and this.depthStencilAttachment.view, if present, the [[renderExtent]] must match.
8. If this.occlusionQuerySet is not null:
    1. this.occlusionQuerySet.type must be occlusion.
9. Validate timestampWrites(device, this.timestampWrites) must return true.
10. No two entries in this.timestampWrites may have the same [querySet, queryIndex] pair.

---
---

**验证 `GPURenderPassDescriptor 的每个样本的颜色附件字节(GPUDevice device, sequence\<GPURenderPassColorAttachment? \> colorAttachments)`**             
1. 让 formats 成为一个空的 \<GPUTextureFormat?\> 类型的列表。
2. 对于 colorAttachments 中的每个 colorAttachment：
    1. 如果 colorAttachment 是未定义的，继续。
    2. 将 colorAttachment.view.[[descriptor]].format 追加到 formats。
3. 计算每个样本（格式）的颜色附件字节数必须 ≤ device.[[limit]].maxColorAttachmentBytesPerSample。

---

**Validating GPURenderPassDescriptor’s color attachment bytes per sample(GPUDevice device, sequence\<GPURenderPassColorAttachment?\> colorAttachments)**

1. Let formats be an empty list\<GPUTextureFormat?\>
2. For each colorAttachment in colorAttachments:
    1. If colorAttachment is undefined, continue.
    2. Append colorAttachment.view.[[descriptor]].format to formats.
3. Calculating color attachment bytes per sample(formats) must be ≤ device.[[limits]].maxColorAttachmentBytesPerSample.

---
---

#### 17.1.1.1. 颜色附件

```
dictionary GPURenderPassColorAttachment {
    required GPUTextureView view;
    GPUTextureView resolveTarget;

    GPUColor clearValue;
    required GPULoadOp loadOp;
    required GPUStoreOp storeOp;
};
```

---

17.1.1.1. Color Attachments

```
dictionary GPURenderPassColorAttachment {
    required GPUTextureView view;
    GPUTextureView resolveTarget;

    GPUColor clearValue;
    required GPULoadOp loadOp;
    required GPUStoreOp storeOp;
};
```

---
---

**view，类型为: GPUTextureView**        
一个描述纹理子资源的 GPUTextureView ，该纹理子资源将被输出到这个颜色附件。

**resolveTarget，类型为: GPUTextureView。**
一个描述纹理子资源的 GPUTextureView，如果视图是多采样的，将接收此颜色附件的解析输出。

**clearValue，类型为: GPUColor**
表示在执行渲染过程之前用于清除视图的值。如果没有提供，默认值为 {r：0, g: 0, b: 0, a: 0} 。如果loadOp 不是 "clear"，则会被忽略。

clearValue 的分量都是双倍的值。它们会被转换为符合渲染附件的纹理格式的 texel 值。如果转换失败，将会产生一个验证错误。

**loadOp，类型为： GPULoadOp**
表示在执行渲染过程之前要对视图进行的加载操作。

**注意：** 建议优先选择清除，详情请看 "清除" 操作。

**storeOp，类型为： GPUStoreOp**        
在执行完渲染过程之后对视图进行的存储操作。

---

**view, of type GPUTextureView**        
A GPUTextureView describing the texture subresource that will be output to for this color attachment.

**resolveTarget, of type GPUTextureView**       
A GPUTextureView describing the texture subresource that will receive the resolved output for this color attachment if view is multisampled.

**clearValue, of type GPUColor**        
Indicates the value to clear view to prior to executing the render pass. If not provided, defaults to {r: 0, g: 0, b: 0, a: 0}. Ignored if loadOp is not "clear".

The components of clearValue are all double values. They are converted to a texel value of texture format matching the render attachment. If conversion fails, a validation error is generated.

**loadOp, of type GPULoadOp**       
Indicates the load operation to perform on view prior to executing the render pass.

**NOTE: ** It is recommended to prefer clearing; see "clear" for details.

**storeOp, of type GPUStoreOp**     
The store operation to perform on view after executing the render pass.

---
---

GPURenderPassColorAttachment 有效用法
实例 this 给定一个GPURenderPassColorAttachment：        
1. 让 renderViewDescriptor 成为 this.view.[[descriptor]]。
2. 让 resolveViewDescriptor 成为 this.resolveTarget.[[descriptor]]。
3. 让 renderTexture 成为 this.view.[[texture]]。
4. 让 resolveTexture 成为 this.resolveTarget.[[texture]]。

下列验证规则适用：      
* renderViewDescriptor.format 必须是一个可渲染的颜色格式。
* this.view 必须是一个可渲染的纹理视图。
* 如果 this.loadOp 是 "clear"：
    * 将 IDL 值 this.clearValue 转换为纹理格式为 renderViewDescriptor.format 的 texel 值时，不得抛出TypeError。         
    **注意：** 如果该值，对于格式来说是超出范围的，但对于相应的 WGSL 原始类型（f32、i32或u32）来说是在范围的，则不会抛出错误。
* 如果提供了 this.resolveTarget：
    * renderTexture.sampleCount 必须大于1。
    * resolveTexture.sampleCount 必须是1。
    * this.resolveTarget 必须是一个可渲染的纹理视图。
    * this.resolveTarget 和 this.view 所访问的子资源的大小必须是匹配的。
    * resolveViewDescriptor.format 必须等于 renderViewDescriptor.format。
    * resolveTexture.format 必须等于 renderTexture.format。
    * resolveViewDescriptor.format 必须支持 根据 §26.1.1 普通颜色格式 要求进行解析。

---

**GPURenderPassColorAttachment Valid Usage**        

Given a GPURenderPassColorAttachment this:      
1. Let renderViewDescriptor be this.view.[[descriptor]].
2. Let resolveViewDescriptor be this.resolveTarget.[[descriptor]].
3. Let renderTexture be this.view.[[texture]].
4. Let resolveTexture be this.resolveTarget.[[texture]].

The following validation rules apply:       
* renderViewDescriptor.format must be a color renderable format.
* this.view must be a renderable texture view.
* If this.loadOp is "clear":
    * Converting the IDL value this.clearValue to a texel value of texture format renderViewDescriptor.format must not throw a TypeError.       
    **NOTE:** An error is not thrown if the value is out-of-range for the format but in-range for the corresponding WGSL primitive type (f32, i32, or u32).
* If this.resolveTarget is provided:
    * renderTexture.sampleCount must be > 1.
    * resolveTexture.sampleCount must be 1.
    * this.resolveTarget must be a renderable texture view.
    * The sizes of the subresources seen by this.resolveTarget and this.view must match.
    * resolveViewDescriptor.format must equal renderViewDescriptor.format.
    * resolveTexture.format must equal renderTexture.format.
    * resolveViewDescriptor.format must support resolve according to § 26.1.1 Plain color formats.

---
---

如果满足以下要求，则 GPUTextureView 视图就是一个可渲染的纹理视图：
* view.[[texture]].usage 必须包含 RENDER_ATTACHMENT。
* descriptor.dimension 必须是 "2d"。
* descriptor.mipLevelCount 必须是1。
* descriptor.arrayLayerCount 必须是1。
* descriptor.aspect 必须引用的是 view.[[texture]] 的所有方面。

其中 descriptor 是 view.[[descriptor]]。

---

A GPUTextureView view is a renderable texture view if the following requirements are met:
* view.[[texture]].usage must contain RENDER_ATTACHMENT.
* descriptor.dimension must be "2d".
* descriptor.mipLevelCount must be 1.
* descriptor.arrayLayerCount must be 1.
* descriptor.aspect must refer to all aspects of view.[[texture]].

where descriptor is view.[[descriptor]].

---
---

**计算每个样本的颜色附件字节数(formats)**       

**参数：**  
* sequence\<GPUTextureFormat?\> formats

**返回值：** GPUSize32

1. 让总数 total 为 0。
2. 对于格式中的每个非空的格式： 
    1. 断言：格式是一个可渲染的颜色格式。
    2. 让 renderTargetPixelByteCost 成为格式的 渲染目标像素字节成本数。
    3. 让 renderTargetComponentAlignment 成为格式的 渲染目标分量的对齐方式。
    4. 将总数四舍五入到 renderTargetComponentAlignment 的最小倍数，大于或等于总数。
    5. 把 renderTargetPixelByteCost 加到总数中。
3. 返回总数 total。

---

**Calculating color attachment bytes per sample(formats)**

**Arguments:**  
* sequence\<GPUTextureFormat?\> formats

**Returns:** GPUSize32
1. Let total be 0.
2. For each non-null format in formats
    1. Assert: format is a color renderable format.
    2. Let renderTargetPixelByteCost be the render target pixel byte cost of format.
    3. Let renderTargetComponentAlignment be the render target component alignment of format.
    4. Round total up to the smallest multiple of renderTargetComponentAlignment greater than or equal to total.
    5. Add renderTargetPixelByteCost to total.
3. Return total.

---
---

#### 17.1.1.2. 深度/模板 附件

```
dictionary GPURenderPassDepthStencilAttachment {
    required GPUTextureView view;

    float depthClearValue;
    GPULoadOp depthLoadOp;
    GPUStoreOp depthStoreOp;
    boolean depthReadOnly = false;

    GPUStencilValue stencilClearValue = 0;
    GPULoadOp stencilLoadOp;
    GPUStoreOp stencilStoreOp;
    boolean stencilReadOnly = false;
};
```
---

17.1.1.2. Depth/Stencil Attachments

```
dictionary GPURenderPassDepthStencilAttachment {
    required GPUTextureView view;

    float depthClearValue;
    GPULoadOp depthLoadOp;
    GPUStoreOp depthStoreOp;
    boolean depthReadOnly = false;

    GPUStencilValue stencilClearValue = 0;
    GPULoadOp stencilLoadOp;
    GPUStoreOp stencilStoreOp;
    boolean stencilReadOnly = false;
};
```

---
---

**view，类型为： GPUTextureView**       
描述纹理子资源的 GPUTextureView，该纹理子资源将被输出到此深度/模板附件中，并且也从中读取数据。

**depthClearValue，类型为： float**     
表示在执行渲染过程之前要清除视图的深度分量的值。如果 depthLoadOp 操作不是 "clear"，则会被忽略。其值必须在 0.0 和 1.0 之间，且包括边界值。

**depthLoadOp，类型为： GPULoadOp。**
表示在执行渲染过程之前对视图的深度组件进行的加载操作。

**注意：** 建议优先选择清除操作，详情请参见 "clear" 操作。

**depthStoreOp，类型为： GPUStoreOp**
表示在执行渲染过程之后对视图的深度组件进行的存储操作。

**depthReadOnly，类型为： 布尔值，默认为false。**
表示视图的深度组件是只读的。

**stencilClearValue，类型为： GPUStencilValue，默认为0。**      
表示在执行渲染过程之前要清除视图的模板组件的值。如果 stencilLoadOp 不是 "clear"，则忽略。

该值将被转换为视图模板方面的类型，通过将 LSBs 的数量，设置为在模板方面的视图中一个 texel 块的比特数，即 此 texel 块的比特数 和 LSBs 的数量是相同的。

**stencilLoadOp，类型为： GPULoadOp**
表示在执行渲染过程之前对视图的模板组件进行的加载操作。

**注意：** 建议优先选择清除操作，详情请参见 "clear" 操作。

**stencilStoreOp, 类型为： GPUStoreOp**
表示在执行渲染过程之后对视图的模板组件进行的存储操作。

**stencilReadOnly，类型为：布尔值，默认为 false。**
表示视图的模板组件是只读的。

---

**view, of type GPUTextureView**        
A GPUTextureView describing the texture subresource that will be output to and read from for this depth/stencil attachment.

**depthClearValue, of type float**      
Indicates the value to clear view's depth component to prior to executing the render pass. Ignored if depthLoadOp is not "clear". Must be between 0.0 and 1.0, inclusive.

**depthLoadOp, of type GPULoadOp**      
Indicates the load operation to perform on view's depth component prior to executing the render pass.

**NOTE:** It is recommended to prefer clearing; see "clear" for details.

**depthStoreOp, of type GPUStoreOp**        
The store operation to perform on view's depth component after executing the render pass.

**depthReadOnly, of type boolean, defaulting to false**     
Indicates that the depth component of view is read only.

**stencilClearValue, of type GPUStencilValue, defaulting to 0**     
Indicates the value to clear view's stencil component to prior to executing the render pass. Ignored if stencilLoadOp is not "clear".

The value will be converted to the type of the stencil aspect of view by taking the same number of LSBs as the number of bits in the stencil aspect of one texel block of view.

**stencilLoadOp, of type GPULoadOp**        
Indicates the load operation to perform on view's stencil component prior to executing the render pass.

**NOTE:** It is recommended to prefer clearing; see "clear" for details.

**stencilStoreOp, of type GPUStoreOp**          
The store operation to perform on view's stencil component after executing the render pass.

**stencilReadOnly, of type boolean, defaulting to false**           
Indicates that the stencil component of view is read only.

---
---

**GPURenderPassDepthStencilAttachment 有效用法**        
通过实例 this 给定一个 GPURenderPassDepthStencilAttachment，适用以下验证规则：          
* this.view 必须有一个深度/模板格式。
* this.view 必须是一个可渲染的纹理视图。
* 让 format 成为 this.view.[[descriptor]].format。
* 如果 this.depthLoadOp 是 "clear"，必须提供 this.depthClearValue，并且其值必须在 0.0 和 1.0 之间，包括0.0 或 1.0 边界值。
* 如果 format 是一个组合的深度/模板格式：
    * this.depthReadOnly 必须等于 this.stencilReadOnly
* 如果格式有一个深度方面，并且 this.depthReadOnly 不为真：
    * 必须提供 this.depthLoadOp。
    * 必须提供 this.depthStoreOp。          
    否则：
    * 必须不能提供 this.depthLoadOp。
    * 必须不能提供 this.depthStoreOp。
* 如果格式有一个模板方面，并且 this.stencilReadOnly 不为真：
    * 必须提供 this.stencilLoadOp。
    * 必须提供 this.stencilStoreOp。        
    否则：
    * 必须不能提供 this.stencilLoadOp。
    * 必须不能提供 this.stencilStoreOp。

---

**GPURenderPassDepthStencilAttachment Valid Usage**     

Given a GPURenderPassDepthStencilAttachment this, the following validation rules apply:

* this.view must have a depth-or-stencil format.
* this.view must be a renderable texture view.
* Let format be this.view.[[descriptor]].format.
* If this.depthLoadOp is "clear", this.depthClearValue must be provided and must be between 0.0 and 1.0, inclusive.
* If format is a combined depth-stencil format:
    * this.depthReadOnly must be equal to this.stencilReadOnly
* If format has a depth aspect and this.depthReadOnly is not true:
    * this.depthLoadOp must be provided.
    * this.depthStoreOp must be provided.               
    Otherwise:
    * this.depthLoadOp must not be provided.
    * this.depthStoreOp must not be provided.
* If format has a stencil aspect and this.stencilReadOnly is not true:
    * this.stencilLoadOp must be provided.
    * this.stencilStoreOp must be provided.             
    Otherwise:
    * this.stencilLoadOp must not be provided.
    * this.stencilStoreOp must not be provided.

---
---

#### 17.1.1.3. 加载和存储操作

```
enum GPULoadOp {
    "load",
    "clear"
};
```

**"load"**      
将此附件中现有值加载到渲染通道中。

**"clear"**     
为此附件加载一个清除值到渲染通道中。

**注意：** 在某些GPU硬件上（主要是移动设备），"clear "明显更加高效，因为它避免了将数据从主内存加载到瓦片本地内存中。而在其他GPU硬件上，则没有明显的区别。因此，在初始值不重要的情况下，建议使用 "clear" 操作而不是 "load"操作。（例如，渲染目标将使用天空盒来清除）。

---

17.1.1.3. Load & Store Operations

```
enum GPULoadOp {
    "load",
    "clear"
};
```

**"load"**      
Loads the existing value for this attachment into the render pass.

**"clear"**         
Loads a clear value for this attachment into the render pass.

**NOTE:** On some GPU hardware (primarily mobile), "clear" is significantly cheaper because it avoids loading data from main memory into tile-local memory. On other GPU hardware, there isn’t a significant difference. As a result, it is recommended to use "clear" rather than "load" in cases where the initial value doesn’t matter (e.g. the render target will be cleared using a skybox).

---
---


```
enum GPUStoreOp {
    "store",
    "discard"
};
```

**"store"**     
存储此附件的渲染通道的结果值。

**"discard"（丢弃）**       
丢弃此附件的渲染通道的结果值。

---

```
enum GPUStoreOp {
    "store",
    "discard"
};
```

**"store"**         
Stores the resulting value of the render pass for this attachment.

**"discard"**           
Discards the resulting value of the render pass for this attachment.

---
---

17.1.1.4.渲染通道布局

GPURenderPassLayout 包含了当前通道的渲染目标布局，它决定了该通道与渲染管道的兼容性。

```
dictionary GPURenderPassLayout : GPUObjectDescriptorBase {
    required sequence\<GPUTextureFormat?\> colorFormats;
    GPUTextureFormat depthStencilFormat;
    GPUSize32 sampleCount = 1;
};
```

两个 GPURenderPassLayout 值要相等的话，必须满足以下条件：
* 它们的 depthStencilFormat 和 sampleCount 相等，并且
* 忽略任何尾部的空值，它们的 colorFormats 是相等的。

---

**17.1.1.4. Render Pass Layout**            

GPURenderPassLayout contains the layout of the render targets for the current pass, which determines the compatibility of the pass with render pipelines.

```
dictionary GPURenderPassLayout : GPUObjectDescriptorBase {
    required sequence\<GPUTextureFormat?\> colorFormats;
    GPUTextureFormat depthStencilFormat;
    GPUSize32 sampleCount = 1;
};
```

**Two GPURenderPassLayout values are equal if:**        
* Their depthStencilFormat and sampleCount are equal, and
* Their colorFormats are equal ignoring any trailing nulls.

---
---

**从通道中导出渲染目标布局**

**参数：**  
* GPURenderPassDescriptor 描述符

**返回：** GPURenderPassLayout

1. 让 layout 成为一个新的 GPURenderPassLayout 对象。
2. 对于 descriptor.colorAttachments 中的每个 colorAttachment：
    1. 如果 colorAttachment 不是空的：
        1. 将 layout.sampleCount 设为 colorAttachment.view.[[texture]].sampleCount。
        2. 将 colorAttachment.view.[[descriptor]].format 追加到 layout.colorFormats。
    2. 否则：
    1. 将 null 追加到 layout.colorFormats。
3. 让 depthStencilAttachment 成为 descriptor.depthStencilAttachment ，如果没有提供，则为 null。
4. 如果 depthStencilAttachment 不是null：
    1. 让 view 成为 depthStencilAttachment.view。
    2. 将 layout.sampleCount 设为 view.[[texture]].sampleCount。
    3. 将 layout.depthStencilFormat 设为 view.[[descriptor]].format。
5. 返回 layout。

---

**derive render targets layout from pass**      
**Arguments:**
* GPURenderPassDescriptor descriptor            

**Returns:** GPURenderPassLayout

1. Let layout be a new GPURenderPassLayout object.
2. For each colorAttachment in descriptor.colorAttachments:
    1. If colorAttachment is not null:
        1. Set layout.sampleCount to colorAttachment.view.[[texture]].sampleCount.
        2. Append colorAttachment.view.[[descriptor]].format to layout.colorFormats.
    2. Otherwise:
        1. Append null to layout.colorFormats.
3. Let depthStencilAttachment be descriptor.depthStencilAttachment, or null if not provided.
4. If depthStencilAttachment is not null:
    1. Let view be depthStencilAttachment.view.
    2. Set layout.sampleCount to view.[[texture]].sampleCount.
    3. Set layout.depthStencilFormat to view.[[descriptor]].format.
5. Return layout.

---
---

**从管道中导出渲染目标布局**    

**参数：**      
* GPURenderPipelineDescriptor 描述符

**返回值：** GPURenderPassLayout

1. 让 layout 成为一个新的 GPURenderPassLayout 对象。
2. 设置 layout.sampleCount 为 descriptor.multisample.count。
3. 如果提供了 descriptor.depthStencil：
    1. 将 layout.depthStencilFormat 设置为 descriptor.depthStencil.format。
4. 如果提供了 descriptor.fragment：
    1. 对于 descriptor.fragment.target 中的每个 colorTarget：
        1. 如果 colorTarget 不是空的，则将 colorTarget.format 追加到 layout.colorFormats 中，否则追加为 null。
5. 返回 layout。

---

**derive render targets layout from pipeline**

**Arguments:**      
* GPURenderPipelineDescriptor descriptor

**Returns:**  GPURenderPassLayout

1. Let layout be a new GPURenderPassLayout object.
2. Set layout.sampleCount to descriptor.multisample.count.
3. If descriptor.depthStencil is provided:
    1. Set layout.depthStencilFormat to descriptor.depthStencil.format.
4. If descriptor.fragment is provided:
    1. For each colorTarget in descriptor.fragment.targets:
        1. Append colorTarget.format to layout.colorFormats if colorTarget is not null, or append null otherwise.
5. Return layout.

---
---

### 17.1.2. 最终化

一旦用户完成了该通道的命令记录，就可以通过调用 end() 来结束渲染通道编码器。一旦调用了 end()，就不能再使用渲染通道编码器了。

**end()**   
完成对渲染通道命令序列的记录。

**调用于：** GPURenderPassEncoder 实例自身。

**返回值：** undefined

**上下文时间轴步骤：**      
1. 在实例 this.[[device] 的设备时间线上发起后续步骤。

---

17.1.2. Finalization

The render pass encoder can be ended by calling end() once the user has finished recording commands for the pass. Once end() has been called the render pass encoder can no longer be used.

**end()**       

Completes recording of the render pass commands sequence.

**Called on:** GPURenderPassEncoder this.       
**Returns:** undefined          

**Content timeline steps:**     
1. Issue the subsequent steps on the Device timeline of this.[[device]].

---
---
**设备时间轴步骤：**        
1. 让 parentEncoder 成为 this.[[command_encoder]]。
2. 如果以下任何要求未能得到满足，产生一个验证错误并停止。
    * this.[[state]] 必须是 "open"。
    * parentEncoder.[[state]] 必须是 "locked"。
3. 将 this.[[state]] 设置为 "end"。
4. 将 parentEncoder.[[state]] 设置为 "open"。
5. 如果以下任何要求没有得到满足，使 parentEncoder 无效并停止。
    * this 必须是有效的。
    * this.[[debug_group_stack]] 必须为空。
    * this.[[occlusion_query_active]] 必须为假。
    * this.[[drawCount]] 必须 ≤ this.[[maxDrawCount]]。
6. 用 this.[[command]] 扩展 parentEncoder.[[command]]。
7. 对于 this.[[endTimestampWrites]] 中的每个 timestampWrite ：
    1. 断言：timestampWrite.location 是 "end"。
    2. 在 parentEncoder.[[commands]] 中添加一条 GPU命令，将 GPU 的时间戳值写入timestampWrite.queryIndexth索引的 timestampWrite.querySet 中。
8. 在实例的队列中入队一条渲染命令，当此渲染命令带着渲染状态执行时，在队列时间线上发起后续步骤

**队列时间轴步骤：**    
1. **ISSUE 27** 对 附件进行 存储/丢弃 操作。
2. 让 renderState 为 null。

---

**Device timeline steps:**      
1. Let parentEncoder be this.[[command_encoder]].
2. If any of the following requirements are unmet, generate a validation error and stop.
    * this.[[state]] must be "open".
    * parentEncoder.[[state]] must be "locked".
3. Set this.[[state]] to "ended".
4. Set parentEncoder.[[state]] to "open".
5. If any of the following requirements are unmet, make parentEncoder invalid and stop.
    * this must be valid.
    * this.[[debug_group_stack]] must be empty.
    * this.[[occlusion_query_active]] must be false.
    * this.[[drawCount]] must be ≤ this.[[maxDrawCount]].
6. Extend parentEncoder.[[commands]] with this.[[commands]].
7. For each timestampWrite in this.[[endTimestampWrites]]:
    1. Assert: timestampWrite.location is "end".
    2. Append a GPU command to parentEncoder.[[commands]] that writes the GPU’s timestamp value into the timestampWrite.queryIndexth index in timestampWrite.querySet.
8. Enqueue a render command on this which issues the subsequent steps on the Queue timeline with renderState when executed.

**Queue timeline steps:**           
1. **ISSUE 27** Perform the attachment stores/discards.
2. Let renderState be null.

---
---

## 17.2. GPURenderCommandsMixin

GPURenderCommandsMixin 定义了 GPURenderPassEncoder 和 GPURenderBundleEncoder 共有的渲染命令。

```
interface mixin GPURenderCommandsMixin {
    undefined setPipeline(GPURenderPipeline pipeline);

    undefined setIndexBuffer(GPUBuffer buffer, GPUIndexFormat indexFormat, optional GPUSize64 offset = 0, optional GPUSize64 size);
    undefined setVertexBuffer(GPUIndex32 slot, GPUBuffer buffer, optional GPUSize64 offset = 0, optional GPUSize64 size);

    undefined draw(GPUSize32 vertexCount, optional GPUSize32 instanceCount = 1,
        optional GPUSize32 firstVertex = 0, optional GPUSize32 firstInstance = 0);
    undefined drawIndexed(GPUSize32 indexCount, optional GPUSize32 instanceCount = 1,
        optional GPUSize32 firstIndex = 0,
        optional GPUSignedOffset32 baseVertex = 0,
        optional GPUSize32 firstInstance = 0);

    undefined drawIndirect(GPUBuffer indirectBuffer, GPUSize64 indirectOffset);
    undefined drawIndexedIndirect(GPUBuffer indirectBuffer, GPUSize64 indirectOffset);
};
```

---

17.2. GPURenderCommandsMixin

GPURenderCommandsMixin defines rendering commands common to GPURenderPassEncoder and GPURenderBundleEncoder.

```
interface mixin GPURenderCommandsMixin {
    undefined setPipeline(GPURenderPipeline pipeline);

    undefined setIndexBuffer(GPUBuffer buffer, GPUIndexFormat indexFormat, optional GPUSize64 offset = 0, optional GPUSize64 size);
    undefined setVertexBuffer(GPUIndex32 slot, GPUBuffer buffer, optional GPUSize64 offset = 0, optional GPUSize64 size);

    undefined draw(GPUSize32 vertexCount, optional GPUSize32 instanceCount = 1,
        optional GPUSize32 firstVertex = 0, optional GPUSize32 firstInstance = 0);
    undefined drawIndexed(GPUSize32 indexCount, optional GPUSize32 instanceCount = 1,
        optional GPUSize32 firstIndex = 0,
        optional GPUSignedOffset32 baseVertex = 0,
        optional GPUSize32 firstInstance = 0);

    undefined drawIndirect(GPUBuffer indirectBuffer, GPUSize64 indirectOffset);
    undefined drawIndexedIndirect(GPUBuffer indirectBuffer, GPUSize64 indirectOffset);
};
```

---
---

GPURenderCommandsMixin 假定 GPUObjectBase、GPUCommandsMixin 和GPUBindingCommandsMixin 的成员在同一对象上存在。它必须仅仅只能被 包含这些混合器的接口 所包含。

GPURenderCommandsMixin 有以下内部插槽：

**[[layout]], 类型为： GPURenderPassLayout, 只读**      
渲染通道的布局。

**[[depthReadOnly]], 类型为： 布尔值, 只读。**         
如果为true，表示不修改深度组件。

**[[stencilReadOnly]]，类型为： 布尔值，只读**
如果为真，表示网板组件不被修改。

**[[pipeline]], 类型为： GPURenderPipeline**    
当前的GPURenderPipeline，最初为空。

**[[index_buffer]], 类型为： GPUBuffer**
当前用于读取索引数据的缓冲区，最初为空。

**[[index_format]], 类型为 GPUIndexFormat**
[[index_buffer]] 中索引数据的格式。

**[[index_buffer_offset]], 类型为： GPUSize64**     
当前设置的[[index_buffer]] 片段的偏移量，单位是字节。

**[[index_buffer_size]], 类型为： GPUSize64**       
当前设置的[[index_buffer]] 片段的大小，单位是字节，最初为0。

**[[vertex_buffers]]，类型为: ordered map\<slot, GPUBuffer\>。**   
当前为每个插槽读取顶点数据的 GPUBuffers，最初为空。

**[[vertex_buffer_sizes]]，类型为: ordered map\<slot, GPUSize64\>。**        
当前为每个插槽设置的 GPUBuffer 片段的大小，单位是字节，最初为空。

**[[drawCount]], 类型为： GPUSize64**        
这个编码器中记录的绘制命令的数量。

---

GPURenderCommandsMixin assumes the presence of GPUObjectBase, GPUCommandsMixin, and GPUBindingCommandsMixin members on the same object. It must only be included by interfaces which also include those mixins.

GPURenderCommandsMixin has the following internal slots:

**[[layout]], of type GPURenderPassLayout, readonly**       
The layout of the render pass.

**[[depthReadOnly]], of type boolean, readonly**        
If true, indicates that the depth component is not modified.

**[[stencilReadOnly]], of type boolean, readonly**      
If true, indicates that the stencil component is not modified.

**[[pipeline]], of type GPURenderPipeline**     
The current GPURenderPipeline, initially null.

**[[index_buffer]], of type GPUBuffer**         
The current buffer to read index data from, initially null.

**[[index_format]], of type GPUIndexFormat**            
The format of the index data in [[index_buffer]].

**[[index_buffer_offset]], of type GPUSize64**      
The offset in bytes of the section of [[index_buffer]] currently set.

**[[index_buffer_size]], of type GPUSize64**        
The size in bytes of the section of [[index_buffer]] currently set, initially 0.

**[[vertex_buffers]], of type ordered map\<slot, GPUBuffer\>**        
The current GPUBuffers to read vertex data from for each slot, initially empty.

**[[vertex_buffer_sizes]], of type ordered map\<slot, GPUSize64\>**       
The size in bytes of the section of GPUBuffer currently set for each slot, initially empty.

**[[drawCount]], of type GPUSize64**        
The number of draw commands recorded in this encoder.

---
---

为了在 GPURenderCommandsMixin 编码器上排队一条等待渲染命令，该编码器发出的 具有 RenderState 类型渲染状态的 GPU命令 的步骤是：
1. 将命令附加到 encoder.[[commands]]。
2. 当命令作为 GPUCommandBuffer 类型的 commandBuffer 的一部分被执行时：
    1. 以 commandBuffer.[[renderState]] 作为 renderState，发起命令的步骤。

---

To Enqueue a render command on GPURenderCommandsMixin encoder which issues the steps of a GPU Command command with RenderState renderState:
1. Append command to encoder.[[commands]].
2. When command is executed as part of a GPUCommandBuffer commandBuffer:
    1. Issue the steps of command with commandBuffer.[[renderState]] as renderState.

---
---

### 17.2.1. 绘图

**setPipeline(pipeline)**       

设置当前的 GPU 渲染管道。

**调用于：** GPURenderCommandsMixin 实例自身。

**参数：**

| Parameter | Type | Nullable | Optional | Description  |
| --- | --- | --- | --- | --- |
| pipeline | GPURenderPipeline | ✘ | ✘ | 后续绘图命令要使用的渲染管道。. |

**返回值:** undefined

---

17.2.1. Drawing

**setPipeline(pipeline)**       

Sets the current GPURenderPipeline.

**Called on:**  GPURenderCommandsMixin this.

**Arguments:**

| Parameter | Type | Nullable | Optional | Description  |
| --- | --- | --- | --- | --- |
| pipeline | GPURenderPipeline | ✘ | ✘ | The render pipeline to use for subsequent drawing commands. |

**Returns:** undefined

---
---

**上下文时间轴步骤：**      
1. 在实例 this.[[device] 的设备时间线上发起后续步骤。

**设备时间轴步骤：**    
1. 验证这个实例的编码器的状态，如果它返回false，则停止。
2. 让 pipelineTargetsLayout 是从 pipeline(pipeline.[[descriptor]]) 衍生出的渲染目标的布局。
3. 如果以下任何一个条件未能得到满足，则使其无效并停止。
    1. 实例 this 所指向的 pipeline是有效的。
    2. this.[[layout]] 等于 pipelineTargetsLayout。
    3. 如果 pipeline.[[writesDepth]] 存在 ：this.[[depthReadOnly]]必须为 false。
    4. 如果 pipeline.[[writesStencil]] 存在 ： this.[[stencilReadOnly]] 必须是false。
4. 将 this.[[pipeline]] 设置为 pipeline。

---

**Content timeline steps:**
1. Issue the subsequent steps on the Device timeline of this.[[device]].

**Device timeline steps:**
1. Validate the encoder state of this. If it returns false, stop.
2. Let pipelineTargetsLayout be derive render targets layout from pipeline(pipeline.[[descriptor]]).
3. If any of the following conditions are unsatisfied, make this invalid and stop.
    * pipeline is valid to use with this.
    * this.[[layout]] equals pipelineTargetsLayout.
    * If pipeline.[[writesDepth]]: this.[[depthReadOnly]] must be false.
    * If pipeline.[[writesStencil]]: this.[[stencilReadOnly]] must be false.
4. Set this.[[pipeline]] to be pipeline.

---
---

**setIndexBuffer(buffer, indexFormat, offset, size)**       

设置当前的索引缓冲区。

**调用于：** GPURenderCommandsMixin 实例自身。

**参数：**

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| buffer | GPUBuffer | ✘ | ✘ | 包含索引数据的缓冲区，用于后续绘图命令。 |
| indexFormat | GPUIndexFormat | ✘ | ✘ | 缓冲区中包含的索引数据的格式。 |
| offset | GPUSize64 | ✘ | ✔ | 缓冲区中索引数据开始的字节偏移量，默认为0。 |
| size | GPUSize64 | ✘ | ✔ | 缓冲区中索引数据的字节大小。默认为 缓冲区的字节大小减去偏移量。|

**返回值：** undefined

---

**setIndexBuffer(buffer, indexFormat, offset, size)**       

Sets the current index buffer.

**Called on:** GPURenderCommandsMixin this.

**Arguments:**

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| buffer | GPUBuffer | ✘ | ✘ | Buffer containing index data to use for subsequent drawing commands. |
| indexFormat | GPUIndexFormat | ✘ | ✘ | Format of the index data contained in buffer. |
| offset | GPUSize64 | ✘ | ✔ | Offset in bytes into buffer where the index data begins. Defaults to 0. |
| size | GPUSize64 | ✘ | ✔ | Size in bytes of the index data in buffer. Defaults to the size of the buffer minus the offset. |

Returns: undefined

---
---

在实例 this.[[device] 的设备时间线上发起以下步骤。
1. 验证这个实例的编码器的状态，如果它返回false，则停止。
2. 如果字节大小缺失，将大小设置为 max(0, buffer.size - offset)。
3. 如果以下任何条件未能得到满足，则使其无效并停止。
    * 实例 this 所引用的 buffer 是有效的。
    * buffer.usage 包含 INDEX。
    * offset 是 indexFormat的字节大小的倍数。
    * offset + size ≤ buffer.size。
4. 将 buffer 作为输入添加到使用范围。
5. 设置 this.[[index_buffer]] 为 buffer。
6. 设置 this.[[index_format]] 为 indexFormat。
7. 设置 this.[[index_buffer_offset]] 为 offset。
8. 设置 this.[[index_buffer_size]] 为 size。

---
Issue the following steps on the Device timeline of this.[[device]]:            
1. Validate the encoder state of this. If it returns false, stop.
2. If size is missing, set size to max(0, buffer.size - offset).
3. If any of the following conditions are unsatisfied, make this invalid and stop.
    * buffer is valid to use with this.
    * buffer.usage contains INDEX.
    * offset is a multiple of indexFormat’s byte size.
    * offset + size ≤ buffer.size.
4. Add buffer to the usage scope as input.
5. Set this.[[index_buffer]] to be buffer.
6. Set this.[[index_format]] to be indexFormat.
7. Set this.[[index_buffer_offset]] to be offset.
8. Set this.[[index_buffer_size]] to be size.

---
---

**setVertexBuffer(slot, buffer, offset, size)**         

为给定插槽设置当前的顶点缓冲区。

**调用于：** GPURenderCommandsMixin 实例自身。

**参数：**

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| slot | GPUIndex32 | ✘ | ✘ | 要设置顶点缓冲区的顶点缓冲区插槽。 |
| buffer | GPUBuffer | ✘ | ✘ | 包含顶点数据的缓冲区，用于后续的绘图命令。 |
| offset | GPUSize64 | ✘ | ✔ | 缓冲区中顶点数据开始的字节偏移量，默认为0。 |
| size | GPUSize64 | ✘ | ✔ | 缓冲区中顶点数据的字节大小。默认为缓冲区的大小减去偏移量。 |

**返回值：** undefined.

---

**setVertexBuffer(slot, buffer, offset, size)**     

Sets the current vertex buffer for the given slot.

**Called on:** GPURenderCommandsMixin this.

**Arguments:**

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| slot | GPUIndex32 | ✘ | ✘ | The vertex buffer slot to set the vertex buffer for. |
| buffer | GPUBuffer | ✘ | ✘ | Buffer containing vertex data to use for subsequent drawing commands. |
| offset | GPUSize64 | ✘ | ✔ | Offset in bytes into buffer where the vertex data begins. Defaults to 0. |
| size | GPUSize64 | ✘ | ✔ | Size in bytes of the vertex data in buffer. Defaults to the size of the buffer minus the offset. |

Returns: undefined

---
---

在实例 this.[[device] 的设备时间线上发起以下步骤。
1. 验证这个实例的编码器的状态，如果它返回false，则停止。
2. 如果字节大小缺失，将大小设置为 max(0, buffer.size - offset)。
3. 如果以下任何条件未能得到满足，则使其无效并停止。
    * 实例 this 所引用的 buffer 是有效的。
    * buffer.usage 包含 VERTEX。
    * slot < this.[[device]].[[limit]].maxVertexBuffers。
    * 偏移量是 4 的倍数。
    * offset + size ≤ buffer.size。
4. 将 buffer 作为输入添加到使用范围中。
5. 设置 this.[[vertex_buffers]][slot] 为 buffer。
6. 设置 this.[[vertex_buffer_sizes]][slot] 为 size。

---

Issue the following steps on the Device timeline of this.[[device]]:
1. Validate the encoder state of this. If it returns false, stop.
2. If size is missing, set size to max(0, buffer.size - offset).
3. If any of the following conditions are unsatisfied, make this invalid and stop.
    * buffer is valid to use with this.
    * buffer.usage contains VERTEX.
    * slot < this.[[device]].[[limits]].maxVertexBuffers.
    * offset is a multiple of 4.
    * offset + size ≤ buffer.size.
4. Add buffer to the usage scope as input.
5. Set this.[[vertex_buffers]][slot] to be buffer.
6. Set this.[[vertex_buffer_sizes]][slot] to be size.

---
---

**draw(vertexCount, instanceCount, firstVertex, firstInstance)**        

绘制基元，详细说明见 § 23.3 渲染。

**调用于：** GPURenderCommandsMixin 实例自身。

**参数：**

| Parameter | Type | Nullable | Optional | Description | 
| --- | --- | --- | --- | --- |
| vertexCount | GPUSize32 | ✘ | ✘ | 要绘制的顶点数量。 | 
| instanceCount | GPUSize32 | ✘ | ✔ | 要绘制的实例数量。 | 
| firstVertex | GPUSize32 | ✘ | ✔ | 顶点缓冲区的偏移量，以顶点为单位，以此开始绘制。 | 
| firstInstance | GPUSize32 | ✘ | ✔ | 第一个要绘制的实例。 | 

返回： undefined

---

**draw(vertexCount, instanceCount, firstVertex, firstInstance)**        
Draws primitives. See § 23.3 Rendering for the detailed specification.

**Called on:** GPURenderCommandsMixin this.

**Arguments:**

| Parameter | Type | Nullable | Optional | Description | 
| --- | --- | --- | --- | --- |
| vertexCount | GPUSize32 | ✘ | ✘ | The number of vertices to draw. | 
| instanceCount | GPUSize32 | ✘ | ✔ | The number of instances to draw. | 
| firstVertex | GPUSize32 | ✘ | ✔ | Offset into the vertex buffers, in vertices, to begin drawing from. | 
| firstInstance | GPUSize32 | ✘ | ✔ | First instance to draw. | 

**Returns:** undefined

---
---

在实例 this.[[device] 的设备时间线上发起以下步骤。
1. 验证这个实例的编码器的状态，如果它返回false，则停止。
2. 如果以下任何条件未能得到满足，使其无效并停止。
    * 实例 this 所引用的 draw 是有效的。
    * 让 buffer 是 this.[[pipeline]].[[descriptor]].vertex.buffers。
    * 对于从 0到缓冲区的长度（不包括长度值）的每个 GPUIndex32 插槽：
        * 如果 buffers[slot] 为 null，继续。
        * 让 bufferSize 成为 this.[[vertex_buffer_sizes]][slot]。
        * 让 stride 成为 buffers[slot].arrayStride。
        * 让 lastStride 为 max(attribute.offset + sizeof(attribute.format))，对于buffers[slot].attributions 中的每个属性 attribute。
        * 让 strideCount 根据 buffers[slot].stepMode 来计算：       
        **"vertex"（顶点）**        
        firstVertex + vertexCount       
        **"instance"（实例）**      
        firstInstance + instanceCount
        * 如果strideCount ≠ 0
            * 确保（strideCount - 1）× stride + lastStride ≤ bufferSize。
3. 将 this.[[drawCount]] 递增 1。
4. 让 passState 成为实例 this 当前状态的一个快照。
5. 在实例的队列中入队一条渲染命令，当此渲染命令带着渲染状态执行时，在队列时间线上发起后续步骤

---

Issue the following steps on the Device timeline of this.[[device]]:
1. Validate the encoder state of this. If it returns false, stop.
2. If any of the following conditions are unsatisfied, make this invalid and stop.
    * It is valid to draw with this.
    * Let buffers be this.[[pipeline]].[[descriptor]].vertex.buffers.
    * For each GPUIndex32 slot from 0 to buffers.length (non-inclusive):
        * If buffers[slot] is null, continue.
        * Let bufferSize be this.[[vertex_buffer_sizes]][slot].
        * Let stride be buffers[slot].arrayStride.
        * Let lastStride be max(attribute.offset + sizeof(attribute.format)) for each attribute in buffers[slot].attributes.
        * Let strideCount be computed based on buffers[slot].stepMode:
            **"vertex"**        
            firstVertex + vertexCount

            **"instance"**      
            firstInstance + instanceCount
        * If strideCount ≠ 0
            * Ensure (strideCount − 1) × stride + lastStride ≤ bufferSize.
3. Increment this.[[drawCount]] by 1.
4. Let passState be a snapshot of this’s current state.
5. Enqueue a render command on this which issues the subsequent steps on the Queue timeline with renderState when executed.

---
---

**队列时间线步骤：**
1. 绘制 instanceCount 个实例， 从实例的 firstInstance 开始，绘制由 vertexCount 个顶点组成的基元，从顶点参数 firstVertex 开始，并且绘制时要使用从 passState 和 renderState 获取的状态。

---

**Queue timeline steps:**       
1. Draw instanceCount instances, starting with instance firstInstance, of primitives consisting of vertexCount verticies, starting with vertex firstVertex, with the states from passState and renderState.

---
---

**drawIndexed(indexCount, instanceCount, firstIndex, baseVertex, firstInstance)**       

绘制基于索引的基元(图元)。详细说明见 § 23.3 渲染。

**调用于：** GPURenderCommandsMixin 实例自身

**参数：**

| Parameter | Type | Nullable | Optional | Description | 
| --- | --- | --- | --- | --- |
| indexCount | GPUSize32 | ✘ | ✘ | 要绘制的索引数。 | 
| instanceCount | GPUSize32 | ✘ | ✔ | 要绘制的实例的数量。 | 
| firstIndex | GPUSize32 | ✘ | ✔ | 索引缓冲区的偏移量，以索引为单位，开始绘制基元。 | 
| baseVertex | GPUSignedOffset32 | ✘ | ✔ | 在顶点缓冲区进行索引之前，添加到每个索引上的值。 | 
| firstInstance | GPUSize32 | ✘ | ✔ | 第一个要绘制的实例。 | 

**返回值：** undefined

---

**drawIndexed(indexCount, instanceCount, firstIndex, baseVertex, firstInstance)**       

Draws indexed primitives. See § 23.3 Rendering for the detailed specification.

**Called on:** GPURenderCommandsMixin this.

**Arguments:**  

| Parameter | Type | Nullable | Optional | Description | 
| --- | --- | --- | --- | --- |
| indexCount | GPUSize32 | ✘ | ✘ | The number of indices to draw. | 
| instanceCount | GPUSize32 | ✘ | ✔ | The number of instances to draw. | 
| firstIndex | GPUSize32 | ✘ | ✔ | Offset into the index buffer, in indices, begin drawing from. | 
| baseVertex | GPUSignedOffset32 | ✘ | ✔ | Added to each index value before indexing into the vertex buffers. | 
| firstInstance | GPUSize32 | ✘ | ✔ | First instance to draw. | 

Returns: undefined

---
---

在实例 this.[[device] 的设备时间线上发起以下步骤。
1. 验证这个实例的编码器的状态，如果它返回false，则停止。
2. 如果以下任何条件未能得到满足，使其无效并停止。
    * 实例 this 所引用的 draw indexed 是有效的。
    * firstIndex + indexCount ≤ this.[[index_buffer_size]] ÷ this.[[index_format]] 的字节大小；
    * 让 buffer 成为 this.[[pipeline]].[[descriptor]].vertex.buffers。
    * 对于从 0 到 缓冲区的长度（不包括其长度值）的每个 GPUIndex32 插槽：
        * 如果 buffers[slot] 为 null ，继续。
        * 让 bufferSize 成为 this.[[vertex_buffer_sizes]][slot]。
        * 让 stride 成为 buffers[slot].arrayStride。
        * 让 lastStride 为 max(attribute.offset + sizeof(attribute.format))，对于buffers[slot].attributions 中的每个属性 attribute。
        * 让 strideCount 为 firstInstance + instanceCount。
        * 如果 buffers[slot].stepMode 是 "instance" 并且 strideCount ≠ 0：
            * 确保（strideCount - 1）× stride + lastStride ≤ bufferSize。
3. 将 this.[[drawCount]] 递增 1。
4. 让 passState 成为实例 this 当前状态的一个快照。
5. 在实例的队列中入队一条渲染命令，当此渲染命令带着渲染状态执行时，在队列时间线上发起后续步骤

---

Issue the following steps on the Device timeline of this.[[device]]:
1. Validate the encoder state of this. If it returns false, stop.
2. If any of the following conditions are unsatisfied, make this invalid and stop.
    * It is valid to draw indexed with this.
    * firstIndex + indexCount ≤ this.[[index_buffer_size]] ÷ this.[[index_format]]'s byte size;
    * Let buffers be this.[[pipeline]].[[descriptor]].vertex.buffers.
    * For each GPUIndex32 slot from 0 to buffers.length (non-inclusive):
        * If buffers[slot] is null, continue.
        * Let bufferSize be this.[[vertex_buffer_sizes]][slot].
        * Let stride be buffers[slot].arrayStride.
        * Let lastStride be max(attribute.offset + sizeof(attribute.format)) for each attribute in buffers[slot].attributes.
        * Let strideCount be firstInstance + instanceCount.
        * If buffers[slot].stepMode is "instance" and strideCount ≠ 0:
            * Ensure (strideCount − 1) × stride + lastStride ≤ bufferSize.
3. Increment this.[[drawCount]] by 1.
4. Let passState be a snapshot of this’s current state.
5. Enqueue a render command on this which issues the subsequent steps on the Queue timeline with renderState when executed.

---
---

**队列时间线步骤：**
1. 绘制 instanceCount 个实例，从实例的 firstInstance 开始，绘制由 indexCount 索引顶点组成的基元的实例，从顶点 baseVertex 的索引值为 firstIndex 的位置开始，并且绘制时要使用从 passState 和 renderState 获取的状态。

注意：一个有效的程序也不应该使用超出范围的 GPUVertexStepMode."vertex" 的顶点索引。WebGPU 的具体实现有不同的处理方式来处理此问题，因此允许有一系列的行为。要么丢弃整个绘制调用，要么对那些超出边界的属性的访问由 WGSL 的无效内存引用来描述。

---

**Queue timeline steps:**

1. Draw instanceCount instances, starting with instance firstInstance, of primitives consisting of indexCount indexed verticies, starting with index firstIndex from vertex baseVertex, with the states from passState and renderState.

**NOTE:** a valid program should also never use vertex indices with GPUVertexStepMode."vertex" that are out of bounds. WebGPU implementations have different ways of handling this, and therefore a range of behaviors is allowed. Either the whole draw call is discarded, or the access to those attributes out of bounds is described by WGSL’s invalid memory reference.

---
---

**drawIndirect(indirectBuffer, indirectOffset)**     

使用从 GPUBuffer 读取的参数来绘制基元。详细说明见§ 23.3 渲染。

缓冲区中编码的间接绘图参数必须是由四个32位无符号整数值（共16个字节）组成的紧密块，其顺序与 draw() 函数的参数相同。比如说：

```
let drawIndirectParameters = new Uint32Array(4);
drawIndirectParameters[0] = vertexCount;
drawIndirectParameters[1] = instanceCount;
drawIndirectParameters[2] = firstVertex;
drawIndirectParameters[3] = firstInstance;
```

对应于 firstInstance 的值必须是 0，除非 "间接绘制第一个实例" 功能被启用。如果 "间接绘制第一个实例" 功能没有被启用，并且 firstInstance 不是0，那么 drawIndirect() 调用将被视为无效。

---

**drawIndirect(indirectBuffer, indirectOffset)**

Draws primitives using parameters read from a GPUBuffer. See § 23.3 Rendering for the detailed specification.

The indirect draw parameters encoded in the buffer must be a tightly packed block of four 32-bit unsigned integer values (16 bytes total), given in the same order as the arguments for draw(). For example:

```
let drawIndirectParameters = new Uint32Array(4);
drawIndirectParameters[0] = vertexCount;
drawIndirectParameters[1] = instanceCount;
drawIndirectParameters[2] = firstVertex;
drawIndirectParameters[3] = firstInstance;
```

The value corresponding to firstInstance must be 0, unless the "indirect-first-instance" feature is enabled. If the "indirect-first-instance" feature is not enabled and firstInstance is not zero the drawIndirect() call will be treated as a no-op.

---
---

**调用于：** GPURenderCommandsMixin 实例自身。

**参数：**  

| Parameter | Type | Nullable | Optional | Description | 
| --- | --- | --- | --- | --- |
| indirectBuffer | GPUBuffer | ✘ | ✘ | 包含间接绘制参数的缓冲区。 | 
| indirectOffset | GPUSize64 | ✘ | ✘ | 在间接缓冲区中开始绘制数据的字节偏移量。 | 

返回： undefined

---

**Called on:** GPURenderCommandsMixin this.

**Arguments:**

| Parameter | Type | Nullable | Optional | Description | 
| --- | --- | --- | --- | --- |
| indirectBuffer | GPUBuffer | ✘ | ✘ | Buffer containing the indirect draw parameters. | 
| indirectOffset | GPUSize64 | ✘ | ✘ | Offset in bytes into indirectBuffer where the drawing data begins. | 

Returns: undefined

---
---

在实例 this.[[device] 的设备时间线上发起以下步骤。
1. 验证这个实例的编码器的状态，如果它返回false，则停止。
2. 如果以下任何条件未能得到满足，使其无效并停止。
    * 实例 this 所引用的 draw 是有效的。
    * 实例 this 所引用的 indirectBuffer 是有效的。
    * indirectBuffer.usage 包含 INDIRECT。
    * indirectOffset + sizeof(间接绘制参数) ≤ indirectBuffer.size。
    * indirectOffset 是 4 的倍数。
3. 将 indirectBuffer 作为输入添加到使用范围。
4. 将 this.[[drawCount]] 递增1。
5. 让 passState 成为实例 this 当前状态的一个快照。
6. 在实例的队列中入队一条渲染命令，当此渲染命令带着渲染状态执行时，在队列时间线上发起后续步骤。

---

Issue the following steps on the Device timeline of this.[[device]]:
1. Validate the encoder state of this. If it returns false, stop.
2. If any of the following conditions are unsatisfied, make this invalid and stop.
    * It is valid to draw with this.
    * indirectBuffer is valid to use with this.
    * indirectBuffer.usage contains INDIRECT.
    * indirectOffset + sizeof(indirect draw parameters) ≤ indirectBuffer.size.
    * indirectOffset is a multiple of 4.
3. Add indirectBuffer to the usage scope as input.
4. Increment this.[[drawCount]] by 1.
5. Let passState be a snapshot of this’s current state.
6. Enqueue a render command on this which issues the subsequent steps on the Queue timeline with renderState when executed.

---
---

**队列时间轴步骤：**
1. 让 顶点计数（vertexCount）是一个无符号的32位整数，从 indirectBuffer 中的indirectOffset 字节处读取。
2. 让 instanceCount 是一个无符号的32位整数，从 indirectBuffer 中的（indirectOffset+4）字节处读取。
3. 让 firstVertex 是一个无符号的32位整数，从 indirectBuffer 中的 (indirectOffset + 8)字节处读取。
4. 让 firstInstance 是一个无符号的32位整数，从indirectBuffer 中的 (indirectOffset + 12) 字节处读取。
5. 绘制 instanceCount 个实例，从实例的 firstInstance 开始，绘制由 vertexCount 个顶点组成的基元，从顶点 firstVertex 开始，并且绘制时要使用从 passState 和 renderState 获取的状态。

---

**Queue timeline steps:**       
1. Let vertexCount be an unsigned 32-bit integer read from indirectBuffer at indirectOffset bytes.
2. Let instanceCount be an unsigned 32-bit integer read from indirectBuffer at (indirectOffset + 4) bytes.
3. Let firstVertex be an unsigned 32-bit integer read from indirectBuffer at (indirectOffset + 8) bytes.
4. Let firstInstance be an unsigned 32-bit integer read from indirectBuffer at (indirectOffset + 12) bytes.
5. Draw instanceCount instances, starting with instance firstInstance, of primitives consisting of vertexCount verticies, starting with vertex firstVertex, with the states from passState and renderState.

---
---

**drawIndexedIndirect(indirectBuffer, indirectOffset)**

使用从 GPUBuffer 读取的参数绘制索引的基元。详细说明见§ 23.3 渲染。

缓冲区中编码的间接drawIndexed参数必须是由五个32位无符号整数值（共20个字节）组成的紧密块，其顺序与 drawIndexed() 的参数相同。比如说：

```
let drawIndexedIndirectParameters = new Uint32Array(5);
drawIndexedIndirectParameters[0] = indexCount;
drawIndexedIndirectParameters[1] = instanceCount;
drawIndexedIndirectParameters[2] = firstIndex;
drawIndexedIndirectParameters[3] = baseVertex;
drawIndexedIndirectParameters[4] = firstInstance;
```

对应于 firstInstance 的值必须是 0，除非 "间接绘制第一个实例" 功能被启用。如果 "间接绘制第一个实例" 功能没有被启用，并且 firstInstance 不是0，那么 drawIndirect() 调用将被视为无效。

---

**drawIndexedIndirect(indirectBuffer, indirectOffset)**     

Draws indexed primitives using parameters read from a GPUBuffer. See § 23.3 Rendering for the detailed specification.

The indirect drawIndexed parameters encoded in the buffer must be a tightly packed block of five 32-bit unsigned integer values (20 bytes total), given in the same order as the arguments for drawIndexed(). For example:

```
let drawIndexedIndirectParameters = new Uint32Array(5);
drawIndexedIndirectParameters[0] = indexCount;
drawIndexedIndirectParameters[1] = instanceCount;
drawIndexedIndirectParameters[2] = firstIndex;
drawIndexedIndirectParameters[3] = baseVertex;
drawIndexedIndirectParameters[4] = firstInstance;
```

The value corresponding to firstInstance must be 0, unless the "indirect-first-instance" feature is enabled. If the "indirect-first-instance" feature is not enabled and firstInstance is not zero the drawIndexedIndirect() call will be treated as a no-op.

---
---

**调用于：** GPURenderCommandsMixin 实例自身。

**参数：**  

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| indirectBuffer |GPUBuffer | ✘ | ✘ | 包含间接绘制索引参数的缓冲区。 |
| indirectOffset |GPUSize64 | ✘ | ✘ | 在间接缓冲区中开始绘制数据的字节偏移。 |

**返回：** undefined

---

**Called on:** GPURenderCommandsMixin this.

**Arguments:**

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| indirectBuffer |GPUBuffer | ✘ | ✘ | Buffer containing the indirect drawIndexed parameters. |
| indirectOffset |GPUSize64 | ✘ | ✘ | Offset in bytes into indirectBuffer where the drawing data begins. |

Returns: undefined

---
---

在实例 this.[[device] 的设备时间线上发起以下步骤。
1. 验证这个实例的编码器的状态，如果它返回false，则停止。
2. 如果以下任何条件未能得到满足，使其无效并停止。
    * 实例 this 所引用的 draw indexed 是有效的。
    * 实例 this 所引用的 indirectBuffer 是有效的。
    * indirectBuffer.usage 包含 INDIRECT。
    * indirectOffset + sizeof(间接绘制索引参数) ≤ indirectBuffer.size。
    * indirectOffset 是 4 的倍数。
3. 将 indirectBuffer 作为输入添加到使用范围。
4. 将 this.[[drawCount]] 递增1。
5. 让 passState 成为实例 this 当前状态的一个快照。
6. 在实例的队列中入队一条渲染命令，当此渲染命令带着渲染状态执行时，在队列时间线上发起后续步骤。

---

Issue the following steps on the Device timeline of this.[[device]]:
1. Validate the encoder state of this. If it returns false, stop.
2. If any of the following conditions are unsatisfied, make this invalid and stop.
    * It is valid to draw indexed with this.
    * indirectBuffer is valid to use with this.
    * indirectBuffer.usage contains INDIRECT.
    * indirectOffset + sizeof(indirect drawIndexed parameters) ≤ indirectBuffer.size.
    * indirectOffset is a multiple of 4.
3. Add indirectBuffer to the usage scope as input.
4. Increment this.[[drawCount]] by 1.
5. Let passState be a snapshot of this’s current state.
6. Enqueue a render command on this which issues the subsequent steps on the Queue timeline with renderState when executed.

---
---

**队列时间轴步骤：**
1. 让 indexCount 是一个无符号的32位整数，从 indirectBuffer 的 indirectOffset 字节处读取。
2. 让 instanceCount 是一个无符号的32位整数，从 indirectBuffer 的 (indirectOffset + 4) 字节处读取。
3. 让 firstIndex 是一个无符号的32位整数，从 indirectBuffer 的 (indirectOffset + 8) 字节处读取。
4. 让 baseVertex 是一个无符号的32位整数，从 indirectBuffer 的 (indirectOffset + 12) 字节处读取。
5. 让 firstInstance 成为一个无符号的32位整数，从 indirectBuffer 的 (indirectOffset + 16) 字节处读取。
6. 绘制 instanceCount 个实例，从实例的 firstInstance 开始，绘制由 indexCount 索引顶点组成的基元的实例，从顶点 baseVertex 的索引值为 firstIndex 的位置开始，并且绘制时要使用从 passState 和 renderState 获取的状态。

---

**Queue timeline steps:**   
1. Let indexCount be an unsigned 32-bit integer read from indirectBuffer at indirectOffset bytes.
2. Let instanceCount be an unsigned 32-bit integer read from indirectBuffer at (indirectOffset + 4) bytes.
3. Let firstIndex be an unsigned 32-bit integer read from indirectBuffer at (indirectOffset + 8) bytes.
4. Let baseVertex be an unsigned 32-bit integer read from indirectBuffer at (indirectOffset + 12) bytes.
5. Let firstInstance be an unsigned 32-bit integer read from indirectBuffer at (indirectOffset + 16) bytes.
6. Draw instanceCount instances, starting with instance firstInstance, of primitives consisting of indexCount indexed verticies, starting with index firstIndex from vertex baseVertex, with the states from passState and renderState.

---
---

要确定使用 GPURenderCommandsMixin 编码器来进行绘制是否有效，请运行以下步骤：
1. 如果以下任何条件不满足，返回 false：
    * 验证 `编码器绑定groups(encoder, encoder.[[pipeline]])` 必须为真。
    * 让 pipelineDescriptor 为 encoder.[[pipeline]].[[descriptor]]。
    * 对于每个从 0 到 pipelineDescriptor.vertex.buffers.length 的 GPUIndex32 插槽：
        * 如果 pipelineDescriptor.vertex.buffers[slot] 不是空的，encoder.[[vertex_buffers]] 必须包含 slot。
2. 否则返回 true。

---

To determine if it’s valid to draw with GPURenderCommandsMixin encoder run the following steps:
1. If any of the following conditions are unsatisfied, return false:
    * Validate encoder bind groups(encoder, encoder.[[pipeline]]) must be true.
    * Let pipelineDescriptor be encoder.[[pipeline]].[[descriptor]].
    * For each GPUIndex32 slot 0 to pipelineDescriptor.vertex.buffers.length:
        * If pipelineDescriptor.vertex.buffers[slot] is not null, encoder.[[vertex_buffers]] must contain slot.
2. Otherwise return true.

---
---

为了确定用 GPURenderCommandsMixin 编码器来进行绘制索引是否有效，请执行以下步骤：
1. 如果以下任何条件不满足，返回false：
    * 使用编码器来进行绘制必须是有效的。
    * encoder.[[index_buffer]] 必须不是空的。
    * 让 拓扑结构 成为 encoder.[[pipeline]].[[descriptor]].primitive.topology。
    * 如果 拓扑结构 是 "line-strip" 或 "triangle-strip"：
        * encoder.[[index_format]] 必须等于 encoder.[[pipeline]].[[descriptor]].primitive.stripIndexFormat。
2. 否则返回true。

---

To determine if it’s valid to draw indexed with GPURenderCommandsMixin encoder run the following steps:
1. If any of the following conditions are unsatisfied, return false:
    * It must be valid to draw with encoder.
    * encoder.[[index_buffer]] must not be null.
    * Let topology be encoder.[[pipeline]].[[descriptor]].primitive.topology.
    * If topology is "line-strip" or "triangle-strip":
        * encoder.[[index_format]] must equal encoder.[[pipeline]].[[descriptor]].primitive.stripIndexFormat.
2. Otherwise return true.

---
---

### 17.2.2. 栅格化状态

GPURenderPassEncoder 有几个方法会影响绘制命令如何进行光栅化该编码器所使用的附件。

**setViewport(x, y, width, height, minDepth, maxDepth)**        
设置在栅格化阶段所使用的视口，从归一化设备坐标线性映射到视口坐标。

**调用于：** GPURenderPassEncoder 实例自身。

**参数：**

| Parameter | Type | Nullable | Optional | Description | 
| --- | --- | --- | --- | --- |
| x | float | ✘ | ✘ | 视口的最小X值，单位是像素。 | 
| y | float | ✘ | ✘ | 视口的最小Y值，单位是像素。 | 
| width | float | ✘ | ✘ | 视口的宽度，单位是像素。| 
| height | float | ✘ | ✘ | 视口的高度，单位是像素。| 
| minDepth | float | ✘ | ✘ | 视口的最小深度值。 | 
| maxDepth | float | ✘ | ✘ | 视口的最大深度值。 | 

返回： undefined

---

17.2.2. Rasterization state

The GPURenderPassEncoder has several methods which affect how draw commands are rasterized to attachments used by this encoder.

**setViewport(x, y, width, height, minDepth, maxDepth)**        

Sets the viewport used during the rasterization stage to linearly map from normalized device coordinates to viewport coordinates.

**Called on:** GPURenderPassEncoder this.

**Arguments:**

| Parameter | Type | Nullable | Optional | Description | 
| --- | --- | --- | --- | --- |
| x | float | ✘ | ✘ | Minimum X value of the viewport in pixels. | 
| y | float | ✘ | ✘ | Minimum Y value of the viewport in pixels. | 
| width | float | ✘ | ✘ | Width of the viewport in pixels. | 
| height | float | ✘ | ✘ | Height of the viewport in pixels. | 
| minDepth | float | ✘ | ✘ | Minimum depth value of the viewport. | 
| maxDepth | float | ✘ | ✘ | Maximum depth value of the viewport. | 

Returns: undefined

---
---

**上下文时间轴步骤：**      
1. 在实例 this.[[device] 的设备时间线上发起后续步骤。

**设备时间轴步骤：**        
1. 验证实例 this 的编码器的状态，如果它返回false，则停止。
2. 如果以下任何条件不满足，使之无效并停止。
    * x ≥ 0
    * y ≥ 0
    * width ≥ 0
    * height ≥ 0
    * x + width ≤ this.[[attachment_size]].width
    * y + height ≤ this.[[attachment_size]].height
    * 0.0 ≤ minDepth ≤ 1.0
    * 0.0 ≤ maxDepth ≤ 1.0
    * minDepth < maxDepth
3. 在实例的队列中入队一条渲染命令，当此渲染命令带着渲染状态执行时，在队列时间线上发起后续步骤

**队列时间线步骤：**
1. 将x、y、width 和 height 四舍五入到某个统一的精度，不能低于四舍五入到整数的精度。
2. 将 renderState.[[viewport]] 设置为x、y、width、height、minDepth 和 maxDepth 的范围。

---

**Content timeline steps:**         
1. Issue the subsequent steps on the Device timeline of this.[[device]].

**Device timeline steps:**          
1. Validate the encoder state of this. If it returns false, stop.
2. If any of the following conditions are unsatisfied, make this invalid and stop.
    * x ≥ 0
    * y ≥ 0
    * width ≥ 0
    * height ≥ 0
    * x + width ≤ this.[[attachment_size]].width
    * y + height ≤ this.[[attachment_size]].height
    * 0.0 ≤ minDepth ≤ 1.0
    * 0.0 ≤ maxDepth ≤ 1.0
    * minDepth < maxDepth
3. Enqueue a render command on this which issues the subsequent steps on the Queue timeline with renderState when executed.

**Queue timeline steps:**       
1. Round x, y, width, and height to some uniform precision, no less precise than integer rounding.
2. Set renderState.[[viewport]] to the extents x, y, width, height, minDepth, and maxDepth.

---
---

**setScissorRect(x, y, width, height)**

设置在栅格化阶段使用的剪裁矩形， 在转化为视口坐标后，任何落在剪切矩形之外的片段都会被丢弃。

**调用于：** GPURenderPassEncoder 实例自身。

**参数：**

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| x | GPUIntegerCoordinate | ✘ | ✘ | 剪切矩形的最小X值, 单位为像素。 |
| y | GPUIntegerCoordinate | ✘ | ✘ | 剪切矩形的最小YY值, 单位为像素。 |
| width | GPUIntegerCoordinate | ✘ | ✘ | 剪切矩形的宽度, 单位为像素。 |
| height | GPUIntegerCoordinate | ✘ | ✘ | 剪切矩形的高度, 单位为像素。 |

**返回值：** undefined

---

**setScissorRect(x, y, width, height)**     

Sets the scissor rectangle used during the rasterization stage. After transformation into viewport coordinates any fragments which fall outside the scissor rectangle will be discarded.

**Called on:** GPURenderPassEncoder this.       
**Arguments:**

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| x | GPUIntegerCoordinate | ✘ | ✘ | Minimum X value of the scissor rectangle in pixels. |
| y | GPUIntegerCoordinate | ✘ | ✘ | Minimum Y value of the scissor rectangle in pixels. |
| width | GPUIntegerCoordinate | ✘ | ✘ | Width of the scissor rectangle in pixels. |
| height | GPUIntegerCoordinate | ✘ | ✘ | Height of the scissor rectangle in pixels. |

Returns: undefined

---
---

**上下文时间轴步骤：**      
1. 在实例 this.[[device] 的设备时间线上发起后续步骤。

**设备时间轴步骤：**        
1. 验证实例 this 的编码器的状态，如果它返回false，则停止。
2. 如果以下任何条件不满足，使之无效并停止。
    * x+width ≤ this.[[attachment_size]].width。
    * y+height ≤ this.[[attachment_size]].height。
3.  在实例的队列中入队一条渲染命令，当此渲染命令带着渲染状态执行时，在队列时间线上发起后续步骤

**队列时间轴步骤：**
1. 将 renderState.[[scissorRect]] 设置为 x、y、宽度和高度的范围。

---

**Content timeline steps:**     
1. Issue the subsequent steps on the Device timeline of this.[[device]].

**Device timeline steps:**      
1. Validate the encoder state of this. If it returns false, stop.
2. If any of the following conditions are unsatisfied, make this invalid and stop.
    * x+width ≤ this.[[attachment_size]].width.
    * y+height ≤ this.[[attachment_size]].height.
3. Enqueue a render command on this which issues the subsequent steps on the Queue timeline with renderState when executed.

**Queue timeline steps:**
1. Set renderState.[[scissorRect]] to the extents x, y, width, and height.

---
---

**setBlendConstant(color)**     
设置 和 "constant" 和 "one-minus-constant" 的 GPUBlendFactors 参数一起使用的 常量混合颜色和 alpha 值。

**调用于：** GPURenderPassEncoder 实例自身。

**参数：**

| Parameter | Type | Nullable | Optional | Description | 
| --- | --- | --- | --- | --- |
| color | GPUColor | ✘ | ✘ | 混合时使用的颜色。 | 

**返回值：** undefined

---
**setBlendConstant(color)**

Sets the constant blend color and alpha values used with "constant" and "one-minus-constant" GPUBlendFactors.

**Called on:** GPURenderPassEncoder this.       
**Arguments:**

| Parameter | Type | Nullable | Optional | Description | 
| --- | --- | --- | --- | --- |
| color | GPUColor | ✘ | ✘ | The color to use when blending. | 

Returns: undefined

---
---


**上下文时间轴步骤：**      
1. ？验证 `GPU颜色着色器(color)`
2. 在实例 this.[[device] 的设备时间线上发起后续步骤。

**设备时间轴步骤：**        
1. 验证实例 this 的编码器的状态，如果它返回false，则停止。
2. 在实例的队列中入队一条渲染命令，当此渲染命令带着渲染状态执行时，在队列时间线上发起后续步骤

**队列时间线步骤：**
1. 设置 renderState.[[blendConstant]] 为 color。

---

**Content timeline steps:**
1. ? validate GPUColor shape(color).
2. Issue the subsequent steps on the Device timeline of this.[[device]].

**Device timeline steps:**      
1. Validate the encoder state of this. If it returns false, stop.
2. Enqueue a render command on this which issues the subsequent steps on the Queue timeline with renderState when executed.

**Queue timeline steps:**       
3. Set renderState.[[blendConstant]] to color.

---
---

**setStencilReference(reference)** 

设置 [[stencilReference]] 的值，在使用 "replace" 的 GPUStencilOperation 进行模板测试时使用。

**调用于：** GPURenderPassEncoder 实例自身。

**参数：**

| Parameter | Type | Nullable | Optional | Description | 
| --- | --- | --- | --- | --- |
| reference | GPUStencilValue | ✘ | ✘ | 新的模板参考值。 | 

**返回值：** undefined

---

**setStencilReference(reference)**      

Sets the [[stencilReference]] value used during stencil tests with the "replace" GPUStencilOperation.

**Called on:** GPURenderPassEncoder this.       

**Arguments:**

| Parameter | Type | Nullable | Optional | Description | 
| --- | --- | --- | --- | --- |
| reference | GPUStencilValue | ✘ | ✘ | The new stencil reference value. | 

Returns: undefined

---
---


**上下文时间轴步骤：**      
1. 在实例 this.[[device] 的设备时间线上发起后续步骤。

**设备时间轴步骤：**        
1. 验证实例 this 的编码器的状态，如果它返回false，则停止。
2. 在实例的队列中入队一条渲染命令，当此渲染命令带着渲染状态执行时，在队列时间线上发起后续步骤

**队列时间轴步骤：**
1. 将 renderState.[[stencilReference]] 设置为 reference。

---

**Content timeline steps:**     
1. Issue the subsequent steps on the Device timeline of this.[[device]].

**Device timeline steps:**      
1. Validate the encoder state of this. If it returns false, stop.
2. Enqueue a render command on this which issues the subsequent steps on the Queue timeline with renderState when executed.

**Queue timeline steps:**       
1. Set renderState.[[stencilReference]] to reference.

---
---

### 17.2.3. 查询

**beginOcclusionQuery(queryIndex)** 

**调用于：** GPURenderPassEncoder 实例自身。

**参数：**

| Parameter | Type | Nullable | Optional | Description | 
| --- | --- | --- | --- | --- |
| queryIndex | GPUSize32 | ✘ | ✘ | 查询集中的查询索引。 | 

**返回值：** undefined

---

17.2.3. Queries

**beginOcclusionQuery(queryIndex)**

**Called on:** GPURenderPassEncoder this.

**Arguments:**

| Parameter | Type | Nullable | Optional | Description | 
| --- | --- | --- | --- | --- |
| queryIndex | GPUSize32 | ✘ | ✘ | The index of the query in the query set. | 

Returns: undefined

---
---

**上下文时间轴步骤：**      
1. 在实例 this.[[device] 的设备时间线上发起后续步骤。

**设备时间轴步骤：**        
1. 验证实例 this 的编码器的状态，如果它返回false，则停止。
2. 如果以下任何条件不满足，使之无效并停止。
    * this.[[occlusion_query_set]] 不是空的。
    * queryIndex < this.[[occlusion_query_set]].count。
    * 在这个通道中同一个 queryIndex 的查询， 必须没有在查询之前被写入。
    * this.[[occlusion_query_active]] 为 false。
3. 将 this.[[occlusion_query_active]] 设置为 true。
1. 在实例的队列中入队一条渲染命令，当此渲染命令带着渲染状态执行时，在队列时间线上发起后续步骤

**队列时间轴步骤：**
1. 将 renderState.[[occlusionQueryIndex]] 设置为 queryIndex。

---

**Content timeline steps:**     
1. Issue the subsequent steps on the Device timeline of this.[[device]].

**Device timeline steps:**
1. Validate the encoder state of this. If it returns false, stop.
2. If any of the following conditions are unsatisfied, make this invalid and stop.
    * this.[[occlusion_query_set]] is not null.
    * queryIndex < this.[[occlusion_query_set]].count.
    * The query at same queryIndex must not have been previously written to in this pass.
    * this.[[occlusion_query_active]] is false.
3. Set this.[[occlusion_query_active]] to true.
4. Enqueue a render command on this which issues the subsequent steps on the Queue timeline with renderState when executed.

**Queue timeline steps:**           
1. Set renderState.[[occlusionQueryIndex]] to queryIndex.

---
---

**endOcclusionQuery()**     

**调用于:** GPURenderPassEncoder 实例自身。

**返回值:** undefined

**上下文时间轴步骤：**      
1. 在实例 this.[[device] 的设备时间线上发起后续步骤。

---

**endOcclusionQuery()**     

**Called on:** GPURenderPassEncoder this.

**Returns:** undefined

**Content timeline steps:**     
1. Issue the subsequent steps on the Device timeline of this.[[device]].

---
---

**设备时间轴步骤：**        
1. 验证实例 this 的编码器的状态，如果它返回false，则停止。
2. 如果以下任何条件不满足，使之无效并停止。
    * this.[[occlusion_query_active]] 为 true。
3. 将 this.[[occlusion_query_active]] 设为 false。
4. 在实例的队列中入队一条渲染命令，当此渲染命令带着渲染状态执行时，在队列时间线上发起后续步骤

**队列时间轴步骤：**
1. 如果在执行相应的 beginOcclusionQuery() 命令后，有任何片段样本通过了所有每个片段的测试，让passingFragments 为非零值。
2. 将 passingFragments 写入 renderState.[[occlusion_query_set]] 索引处的 this.[[occlusionQueryIndex]] 中。

---

**Device timeline steps:**
1. Validate the encoder state of this. If it returns false, stop.
2. If any of the following conditions are unsatisfied, make this invalid and stop.
    * this.[[occlusion_query_active]] is true.
3. Set this.[[occlusion_query_active]] to false.
4. Enqueue a render command on this which issues the subsequent steps on the Queue timeline with renderState when executed.

**Queue timeline steps:**       
1. Let passingFragments be non-zero if any fragment samples passed all per-fragment tests since the corresponding beginOcclusionQuery() command was executed.
2. Write passingFragments into this.[[occlusion_query_set]] at index renderState.[[occlusionQueryIndex]].

---
---

### 17.2.4. 绑定

**executeBundles(bundles)**   

执行先前记录在给定的 GPURenderBundles 中的命令，作为本次渲染过程的一部分。

当一个 GPURenderBundle 被执行时，它不会继承渲染通道的管道、绑定组、顶点和索引缓冲区。在一个 GPURenderBundle 执行后，渲染通道的管道、绑定组和顶点/索引缓冲区的状态会被清除（变成初始的空值）。

**注意：** 状态是被清除的，而不是恢复到以前的状态。即使有零个 GPURenderBundles 被执行，这也会发生。

---

17.2.4. Bundles

**executeBundles(bundles)**     

Executes the commands previously recorded into the given GPURenderBundles as part of this render pass.

When a GPURenderBundle is executed, it does not inherit the render pass’s pipeline, bind groups, or vertex and index buffers. After a GPURenderBundle has executed, the render pass’s pipeline, bind group, and vertex/index buffer state is cleared (to the initial, empty values).

**NOTE:** The state is cleared, not restored to the previous state. This occurs even if zero GPURenderBundles are executed.

---
---

**调用于：** GPURenderPassEncoder 实例自身。

**参数：**

| Parameter | Type | Nullable | Optional | Description | 
| --- | --- | --- | --- | --- |
| bundles | sequence\<GPURenderBundle\> | ✘ | ✘ | 要执行的渲染绑定包的列表。 | 

**返回值：** undefined

**上下文时间轴步骤：**      
1. 在实例 this.[[device] 的设备时间线上发起后续步骤。

---

**Called on:** GPURenderPassEncoder this.

**Arguments:**

| Parameter | Type | Nullable | Optional | Description | 
| --- | --- | --- | --- | --- |
| bundles | sequence\<GPURenderBundle\> | ✘ | ✘ | List of render bundles to execute. | 

Returns: undefined

**Content timeline steps:**     
1. Issue the subsequent steps on the Device timeline of this.[[device]].

---
---

**设备时间轴步骤：**        
1. 验证实例 this 的编码器的状态，如果它返回false，则停止。
2. 如果以下任何条件不满足，使之无效并停止。
    * 对于 bundles 中的每个 bundle：
        * 实例 this 所指向的 bundle 必须是有效的。
        * this.[[layout]] 必须等于 bundle.[[layout]]。
        * 如果 this.[[depthReadOnly]] 为 true，bundle.[[depthReadOnly]] 必须为 true。
        * 如果 this.[[stencilReadOnly]] 为 true，bundle.[[stencilReadOnly]] 必须为 true。
3. 对于 bundles 中的每个 bundle：
    1. 用 bundle.[[drawCount]] 递增 this.[[drawCount]]。
4. 清除 this.[[bind_groups]]。
5. 将 this.[[pipeline]] 设置为 null。
6. 将 this.[[index_buffer]] 设置为 null 。
7. 清除 this.[[vertex_buffers]]。
8. 让 passState 成为实例 this 当前状态的一个快照。
9. 在实例的队列中入队一条渲染命令，当此渲染命令带着渲染状态执行时，在队列时间线上发起后续步骤。

---

**Device timeline steps:**
1. Validate the encoder state of this. If it returns false, stop.
2. If any of the following conditions are unsatisfied, make this invalid and stop.
    * For each bundle in bundles:
        * bundle must be valid to use with this.
        * this.[[layout]] must equal bundle.[[layout]].
        * If this.[[depthReadOnly]] is true, bundle.[[depthReadOnly]] must be true.
        * If this.[[stencilReadOnly]] is true, bundle.[[stencilReadOnly]] must be true.
3. For each bundle in bundles:
    1. Increment this.[[drawCount]] by bundle.[[drawCount]].
4. Clear this.[[bind_groups]].
5. Set this.[[pipeline]] to null.
6. Set this.[[index_buffer]] to null.
7. Clear this.[[vertex_buffers]].
8. Let passState be a snapshot of this’s current state.
9. Enqueue a render command on this which issues the subsequent steps on the Queue timeline with renderState when executed.

---
---

**队列时间轴步骤：**        
1. 对于 bundles 中的每个 bundle：
    1. 使用 passState 和  renderState 来执行 bundle.[[command_list]]中的每个命令。 
    **注意：** 渲染状态不能通过执行渲染绑定包来改变。还要注意的是，对渲染绑定包来说，没有可变的 passState 状态是可访问的。

---

**Queue timeline steps:**
1. For each bundle in bundles:
    1. Execute each command in bundle.[[command_list]] with passState and renderState.      
    **NOTE:** renderState cannot be changed by executing render bundles. Also note, no mutable passState state is visible to render bundles.

