# WebGPU

W3C Working Draft, 22 March 2023 

More details about this document

This version:

https://www.w3.org/TR/2023/WD-webgpu-20230322/

https://www.w3.org/TR/2023/WD-webgpu-20230322/#bind-group-creation

---
---

# 8\. 资源绑定
   
## 8.1. GPUBindGroupLayout

GPUBindGroupLayout 定义了在 GPUBindGroup 中绑定的一组资源 以及 这组资源在着色器阶段的可访问性之间 的接口。

```
[Exposed=(Window, DedicatedWorker), SecureContext]
interface GPUBindGroupLayout {
};
GPUBindGroupLayout includes GPUObjectBase;
```

GPUBindGroupLayout 有以下内部插槽：

[[descriptor]]，类型为：  GPUBindGroupLayoutDescriptor

---

8\. Resource Binding
   
8.1. GPUBindGroupLayout

A GPUBindGroupLayout defines the interface between a set of resources bound in a GPUBindGroup and their accessibility in shader stages.

```
[Exposed=(Window, DedicatedWorker), SecureContext]
interface GPUBindGroupLayout {
};
GPUBindGroupLayout includes GPUObjectBase;
```

GPUBindGroupLayout has the following internal slots:    
**[[descriptor]], of type GPUBindGroupLayoutDescriptor**

---
---

### 8.1.1.绑定组布局的创建

一个 GPUBindGroupLayout 是通过 GPUDevice.createBindGroupLayout() 函数来创建的。

```
dictionary GPUBindGroupLayoutDescriptor : GPUObjectDescriptorBase {
    required sequence<GPUBindGroupLayoutEntry> entries;
};
```

一个 GPUBindGroupLayoutEntry 描述了 一个包含在 GPUBindGroupLayout 中的单一着色器资源绑定。

```
dictionary GPUBindGroupLayoutEntry {
    required GPUIndex32 binding;
    required GPUShaderStageFlags visibility;

    GPUBufferBindingLayout buffer;
    GPUSamplerBindingLayout sampler;
    GPUTextureBindingLayout texture;
    GPUStorageTextureBindingLayout storageTexture;
    GPUExternalTextureBindingLayout externalTexture;
};
```

---

8.1.1. Bind Group Layout Creation

A GPUBindGroupLayout is created via GPUDevice.createBindGroupLayout().

```
dictionary GPUBindGroupLayoutDescriptor : GPUObjectDescriptorBase {
    required sequence<GPUBindGroupLayoutEntry> entries;
};
```

A GPUBindGroupLayoutEntry describes a single shader resource binding to be included in a GPUBindGroupLayout.

```
dictionary GPUBindGroupLayoutEntry {
    required GPUIndex32 binding;
    required GPUShaderStageFlags visibility;

    GPUBufferBindingLayout buffer;
    GPUSamplerBindingLayout sampler;
    GPUTextureBindingLayout texture;
    GPUStorageTextureBindingLayout storageTexture;
    GPUExternalTextureBindingLayout externalTexture;
};
```

---
---

GPUBindGroupLayoutEntry 字典有以下成员：    
**binding，类型为： GPUIndex32**    
GPUBindGroupLayout 中资源绑定的唯一标识，对应于 GPUBindGroupEntry.binding 和 GPUShaderModule 中的 @binding 属性。

**visibility，类型为： GPUShaderStageFlags**    
GPUShaderStage 的成员的一个比特集。每个设置的位表示 GPUBindGroupLayoutEntry 的资源将可以从相关的着色器阶段访问。

**buffer，类型为： GPUBufferBindingLayout**    
当提供时，表示此 GPUBindGroupLayoutEntry 所绑定资源类型为 GPUBufferBinding。

**sampler，类型为： GPUSamplerBindingLayout**    
当提供时，表示此 GPUBindGroupLayoutEntry 所绑定资源类型为 GPUSampler。

**texture，类型为： GPUTextureBindingLayout**    
当提供时，表示此 GPUBindGroupLayoutEntry 所绑定资源类型为 GPUTextureView。

**storageTexture，类型为： GPUStorageTextureBindingLayout**    
当提供时，表示此 GPUBindGroupLayoutEntry 所绑定资源类型为 GPUTextureView。

**externalTexture，类型为： GPUExternalTextureBindingLayout**    
当提供时，表示此 GPUBindGroupLayoutEntry 所绑定资源类型为 GPUExternalTexture。

---

GPUBindGroupLayoutEntry dictionaries have the following members:

**binding, of type GPUIndex32**   
A unique identifier for a resource binding within the GPUBindGroupLayout, corresponding to a GPUBindGroupEntry.binding and a @binding attribute in the GPUShaderModule.

**visibility, of type GPUShaderStageFlags**   
A bitset of the members of GPUShaderStage. Each set bit indicates that a GPUBindGroupLayoutEntry's resource will be accessible from the associated shader stage.

**buffer, of type GPUBufferBindingLayout**    
When provided, indicates the binding resource type for this GPUBindGroupLayoutEntry is GPUBufferBinding.

**sampler, of type GPUSamplerBindingLayout**  
When provided, indicates the binding resource type for this GPUBindGroupLayoutEntry is GPUSampler.

**texture, of type GPUTextureBindingLayout**    
When provided, indicates the binding resource type for this GPUBindGroupLayoutEntry is GPUTextureView.

**storageTexture, of type GPUStorageTextureBindingLayout**    
When provided, indicates the binding resource type for this GPUBindGroupLayoutEntry is GPUTextureView.

**externalTexture, of type GPUExternalTextureBindingLayout**    
When provided, indicates the binding resource type for this GPUBindGroupLayoutEntry is GPUExternalTexture.

---
---

```
typedef [EnforceRange] unsigned long GPUShaderStageFlags;
[Exposed=(Window, DedicatedWorker)]
namespace GPUShaderStage {
    const GPUFlagsConstant VERTEX   = 0x1;
    const GPUFlagsConstant FRAGMENT = 0x2;
    const GPUFlagsConstant COMPUTE  = 0x4;
};
```

GPUShaderStage包含以下标志，这些标志描述了在那个着色器阶段，此 GPUBindGroupLayoutEntry 中一个对应的 GPUBindGroupEntry 将是可见的：    
**VERTEX**    
该绑定组条目将被 顶点着色器访问。

**FRAGMENT**    
该绑定组条目将被 片段着色器访问。

**COMPUTE**   
该绑定组条目将被 计算着色器访问。

GPUBindGroupLayoutEntry 的绑定成员由 GPUBindGroupLayoutEntry 定义了哪些成员而决定：缓冲区、采样器、纹理、存储纹理或外部纹理。对于任何给定的 GPUBindGroupLayoutEntry，只能定义其中一个。每个成员都有一个相关的 GPUBindingResource 类型，每个绑定类型都有一个相关的内部用法，由下表给出：

---

```
typedef [EnforceRange] unsigned long GPUShaderStageFlags;
[Exposed=(Window, DedicatedWorker)]
namespace GPUShaderStage {
    const GPUFlagsConstant VERTEX   = 0x1;
    const GPUFlagsConstant FRAGMENT = 0x2;
    const GPUFlagsConstant COMPUTE  = 0x4;
};
```

GPUShaderStage contains the following flags, which describe which shader stages a corresponding GPUBindGroupEntry for this GPUBindGroupLayoutEntry will be visible to:

**VERTEX**    
The bind group entry will be accessible to vertex shaders.

**FRAGMENT**    
The bind group entry will be accessible to fragment shaders.

**COMPUTE**   
The bind group entry will be accessible to compute shaders.

The binding member of a GPUBindGroupLayoutEntry is determined by which member of the GPUBindGroupLayoutEntry is defined: buffer, sampler, texture, storageTexture, or externalTexture. Only one may be defined for any given GPUBindGroupLayoutEntry. Each member has an associated GPUBindingResource type and each binding type has an associated internal usage, given by this table:

---
---
备注： 此表数据不作过多翻译

|绑定成员 | 资源类型 | 绑定类型 | 绑定用途 |
| --- | --- | --- | --- |
| buffer | GPUBufferBinding | "uniform" | constant |
|  |  | "storage" | storage |
|  |  | "read-only-storage" | storage-read |
| --- | --- | --- | --- |
| sampler | GPUSampler | "filtering" | constant |
|  |  | "non-filtering" | constant |
|  |  | "comparison" | constant |
| --- | --- | --- | --- |
| texture | GPUTextureView | "float" | constant |
|  |  | "unfilterable-float" | constant |
|  |  | "depth" | constant |
|  |  | "sint" | constant |
|  |  | "uint" | constant |
| --- | --- | --- | --- |
| storageTexture | GPUTextureView | "write-only" | storage |
| --- | --- | --- | --- |
| externalTexture | GPUExternalTexture |  | constant |

***表格结束***

---

| Binding member | Resource type | Binding type | Binding usage |
| --- | --- | --- | --- |
| buffer | GPUBufferBinding | "uniform" | constant |
|  |  | "storage" | storage |
|  |  | "read-only-storage" | storage-read |
| --- | --- | --- | --- |
| sampler | GPUSampler | "filtering" | constant |
|  |  | "non-filtering" | constant |
|  |  | "comparison" | constant |
| --- | --- | --- | --- |
| texture | GPUTextureView | "float" | constant |
|  |  | "unfilterable-float" | constant |
|  |  | "depth" | constant |
|  |  | "sint" | constant |
|  |  | "uint" | constant |
| --- | --- | --- | --- |
| storageTexture | GPUTextureView | "write-only" | storage |
| --- | --- | --- | --- |
| externalTexture | GPUExternalTexture |  | constant |

---
---

如果向某一限制使用的插槽的数量超过限制中所支持值，则 GPUBindGroupLayoutEntry 的条目列表值将会超过所支持的限制中的绑定插槽的限制。每个条目可能可以针对多个限制值而使用多个插槽。
1. 对于条目列表中的每个条目，如果：   
**entry.buffer?.type 是 "uniform"， 并且 entry.buffer?.hasDynamicOffset 是 true**
考虑使用 1个 maxDynamicUniformBuffersPerPipelineLayout 插槽。   
**entry.buffer?.type 是 "storage"，并且 entry.buffer?.hasDynamicOffset 是 true**    
考虑使用 1个 maxDynamicStorageBuffersPerPipelineLayout 插槽。

--- 
The list of GPUBindGroupLayoutEntry values entries exceeds the binding slot limits of supported limits limits if the number of slots used toward a limit exceeds the supported value in limits. Each entry may use multiple slots toward multiple limits.

1. For each entry in entries, if:   
    **entry.buffer?.type is "uniform" and entry.buffer?.hasDynamicOffset is true**    
    Consider 1 maxDynamicUniformBuffersPerPipelineLayout slot to be used.
    **entry.buffer?.type is "storage" and entry.buffer?.hasDynamicOffset is true**    
    Consider 1 maxDynamicStorageBuffersPerPipelineLayout slot to be used.

---
---

2. 针对于每个着色器阶段的 "VERTEX, FRAGMENT, COMPUTE"：
    1. 针对条目中的每个条目，如果 entry.visibility 包含 stage 如下的话：
        **entry.buffer?.type是 "uniform"**    
        考虑使用1个 maxUniformBuffersPerShaderStage 插槽。    
        **entry.buffer?.type是 "storage "或 "read-only-storage"**   
        考虑使用1个 maxStorageBuffersPerShaderStage 插槽。    
        **entry.sampler被提供**     
        考虑使用1个 maxSamplersPerShaderStage 插槽。      
        **entry.texture被提供**    
        考虑使用1个 maxSampledTexturesPerShaderStage 插槽。
        **entry.storageTexture被提供**    
        考虑使用1个 maxStorageTexturesPerShaderStage 插槽。
        **entry.externalTexture被提供**   
        考虑使用4个 maxSampledTexturesPerShaderStage 插槽，1个 maxSamplersPerShaderStage 插槽，以及1个maxUniformBuffersPerShaderStage 插槽。

---
2. For each shader stage stage in « VERTEX, FRAGMENT, COMPUTE »:
    1. For each entry in entries for which entry.visibility contains stage, if:       
        **entry.buffer?.type is "uniform"**   
        Consider 1 maxUniformBuffersPerShaderStage slot to be used.         
        **entry.buffer?.type is "storage" or "read-only-storage"**    
        Consider 1 maxStorageBuffersPerShaderStage slot to be used.    
        **entry.sampler is provided**   
        Consider 1 maxSamplersPerShaderStage slot to be used.   
        **entry.texture is provided**   
        Consider 1 maxSampledTexturesPerShaderStage slot to be used.    
        **entry.storageTexture is provided**    
        Consider 1 maxStorageTexturesPerShaderStage slot to be used.    
        **entry.externalTexture is provided**   
        Consider 4 maxSampledTexturesPerShaderStage slot, 1 maxSamplersPerShaderStage slot, and 1 maxUniformBuffersPerShaderStage slot to be used.

---
---

```
enum GPUBufferBindingType {
    "uniform",
    "storage",
    "read-only-storage"
};

dictionary GPUBufferBindingLayout {
    GPUBufferBindingType type = "uniform";
    boolean hasDynamicOffset = false;
    GPUSize64 minBindingSize = 0;
};
```

---
```
enum GPUBufferBindingType {
    "uniform",
    "storage",
    "read-only-storage"
};

dictionary GPUBufferBindingLayout {
    GPUBufferBindingType type = "uniform";
    boolean hasDynamicOffset = false;
    GPUSize64 minBindingSize = 0;
};
```

---
---

GPUBufferBindingLayout 字典有以下成员：   
**type，类型为： GPUBufferBindingType，默认为 "uniform"**  
表示绑定到此绑定的缓冲区所需的类型。    
**hasDynamicOffset，类型为： boolean，默认为false**
表示此绑定是否需要动态偏移。    
**minBindingSize，类型为： GPUSize64，默认为0**
表示与此绑定点一起使用的缓冲区绑定的最小尺寸。     
在  createBindGroup() 中，绑定总是根据这个size大小进行验证。      
如果这个值不是0，管道创建会额外验证这个值 ≥ 该变量的最小缓冲区绑定大小。    
如果这个值是0，管道创建会忽略它，相反，绘图/调度命令会验证 GPUBindGroup 中的每个绑定是否满足此变量值的 最小缓冲区绑定大小。

**注意：**在执行时间过程中，类似的验证理论上是可能的存在的，例如用于验证早期的其他绑定相关字段，如 sampleType 和 format，目前只能在流水线创建时进行验证。然而，这样的执行时验证可能是费时的或不必要的、且复杂的，所以它只适用于 minBindingSize，这样以便最少的对软件工程学设计产生不必要的影响。

---
GPUBufferBindingLayout dictionaries have the following members:   

**type, of type GPUBufferBindingType, defaulting to "uniform"**   
  Indicates the type required for buffers bound to this bindings.   

**hasDynamicOffset, of type boolean, defaulting to false**    
    Indicates whether this binding requires a dynamic offset.    

**minBindingSize, of type GPUSize64, defaulting to 0**    
Indicates the minimum size of a buffer binding used with this bind point.   

Bindings are always validated against this size in createBindGroup().

If this is not 0, pipeline creation additionally validates that this value ≥ the minimum buffer binding size of the variable.

If this is 0, it is ignored by pipeline creation, and instead draw/dispatch commands validate that each binding in the GPUBindGroup satisfies the minimum buffer binding size of the variable.

**NOTE:** Similar execution-time validation is theoretically possible for other binding-related fields specified for early validation, like sampleType and format, which currently can only be validated in pipeline creation. However, such execution-time validation could be costly or unnecessarily complex, so it is available only for minBindingSize which is expected to have the most ergonomic impact.

---
---

```
enum GPUSamplerBindingType {
    "filtering",
    "non-filtering",
    "comparison"
};

dictionary GPUSamplerBindingLayout {
    GPUSamplerBindingType type = "filtering";
};
```

GPUSamplerBindingLayout 字典有以下成员：    
**type，属于GPUSamplerBindingType类型，默认为 "过滤"**    
表示与此绑定的采样器的必要类型。

---

```
enum GPUSamplerBindingType {
    "filtering",
    "non-filtering",
    "comparison"
};

dictionary GPUSamplerBindingLayout {
    GPUSamplerBindingType type = "filtering";
};
```

GPUSamplerBindingLayout dictionaries have the following members:

**type, of type GPUSamplerBindingType, defaulting to "filtering"**    
Indicates the required type of a sampler bound to this bindings.    

---
---

```
enum GPUTextureSampleType {
    "float",
    "unfilterable-float",
    "depth",
    "sint",
    "uint"
};

dictionary GPUTextureBindingLayout {
    GPUTextureSampleType sampleType = "float";
    GPUTextureViewDimension viewDimension = "2d";
    boolean multisampled = false;
};
```

---

```
enum GPUTextureSampleType {
    "float",
    "unfilterable-float",
    "depth",
    "sint",
    "uint"
};

dictionary GPUTextureBindingLayout {
    GPUTextureSampleType sampleType = "float";
    GPUTextureViewDimension viewDimension = "2d";
    boolean multisampled = false;
};
```

---
---
GPUTextureBindingLayout 字典有以下成员：    
**sampleType，类型为： GPUTextureSampleType，默认为 "float"**    
表示与此绑定的纹理视图所需的类型。    
**viewDimension，类型为： GPUTextureViewDimension，默认为 "2d"**   
表示与此绑定的纹理视图所需的尺寸。    
**multisampled，类型为： boolean，默认为false。**    
表示与此绑定的纹理视图是否必须是多采样的。    

---

GPUTextureBindingLayout dictionaries have the following members:

**sampleType, of type GPUTextureSampleType, defaulting to "float"**   
Indicates the type required for texture views bound to this binding.

**viewDimension, of type GPUTextureViewDimension, defaulting to "2d"**    
Indicates the required dimension for texture views bound to this binding.

**multisampled, of type boolean, defaulting to false**    
Indicates whether or not texture views bound to this binding must be multisampled.

---
---

```
enum GPUStorageTextureAccess {
    "write-only"
};

dictionary GPUStorageTextureBindingLayout {
    GPUStorageTextureAccess access = "write-only";
    required GPUTextureFormat format;
    GPUTextureViewDimension viewDimension = "2d";
};

```

---

```
enum GPUStorageTextureAccess {
    "write-only"
};

dictionary GPUStorageTextureBindingLayout {
    GPUStorageTextureAccess access = "write-only";
    required GPUTextureFormat format;
    GPUTextureViewDimension viewDimension = "2d";
};
```

---
---

GPUStorageTextureBindingLayout  字典有以下成员：    
**access，类型为： GPUStorageTextureAccess，默认为 "write-only"。**    
表示这个绑定的访问模式，表示可读和可写。    
**注意：**目前只有一种访问模式，即 "write-only"，但这在将来会进行扩展。   

**format，类型为： GPUTextureFormat**    
表示与此绑定的纹理视图的必要格式。    

**viewDimension，类型为： GPUTextureViewDimension，默认为 "2d"。**   
表示与此绑定的纹理视图所需的尺寸。    

---

GPUStorageTextureBindingLayout dictionaries have the following members:

**access, of type GPUStorageTextureAccess, defaulting to "write-only"**   
The access mode for this binding, indicating readability and writability.

NOTE: There is currently only one access mode, "write-only", but this will expand in the future.

**format, of type GPUTextureFormat**    
The required format of texture views bound to this binding.

**viewDimension, of type GPUTextureViewDimension, defaulting to "2d"**    
Indicates the required dimension for texture views bound to this binding.

---
---

```
dictionary GPUExternalTextureBindingLayout {
};
```
一个 GPUBindGroupLayout 对象有以下内部插槽：    
**[[entryMap]]，类型为： 有序图\<GPUSize32, GPUBindGroupLayoutEntry\>。**    
表示指向 GPUBindGroupLayoutEntrys 的绑定索引图，即这个 GPUBindGroupLayout 所描述的就是这些索引。        

**[[dynamicOffsetCount]], 类型为： GPUSize32**    
表示此 GPUBindGroupLayout 中具有动态偏移量的缓冲区绑定的数量。

**[[exclusivePipeline]], 类型为：GPUPipelineBase?, 初始值为 null**    
如果它是作为默认管线布局的一部分创建的，则表示创建此 GPUBindGroupLayout 的管线。如果它不为空，用此 GPUBindGroupLayout 创建的 GPUBindGroups 只能用于指定的 GPUPipelineBase。

---

```
dictionary GPUExternalTextureBindingLayout {
};
```

A GPUBindGroupLayout object has the following internal slots:

**[[entryMap]], of type ordered map\<GPUSize32, GPUBindGroupLayoutEntry\>**   
The map of binding indices pointing to the GPUBindGroupLayoutEntrys, which this GPUBindGroupLayout describes.

**[[dynamicOffsetCount]], of type GPUSize32**   
The number of buffer bindings with dynamic offsets in this GPUBindGroupLayout.

**[[exclusivePipeline]], of type GPUPipelineBase?, initially null**   
The pipeline that created this GPUBindGroupLayout, if it was created as part of a default pipeline layout. If not null, GPUBindGroups created with this GPUBindGroupLayout can only be used with the specified GPUPipelineBase.

---
---

**createBindGroupLayout(descriptor)**

用于创建一个 GPUBindGroupLayout。

调用于： GPUDevice 实例自身。

参数： 

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| `descriptor` | `GPUBindGroupLayoutDescriptor` | ✘ | ✘ | 创建 GPUBindGroupLayout 的描述符。 |

返回值： GPUBindGroupLayout

---
**createBindGroupLayout(descriptor)**

Creates a GPUBindGroupLayout.

Called on: GPUDevice this.

Arguments:

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| `descriptor` | `GPUBindGroupLayoutDescriptor` | ✘ | ✘ | Description of the GPUBindGroupLayout to create. |

Returns: GPUBindGroupLayout

---
---

上下文时间线步骤：    
1. 对于 descriptor.entries 中的每个 GPUBindGroupLayoutEntry 条目：
    1. 如果提供了 entry.storageTexture：
        1. 在this.[[device]]上，用 entry.storageTexture.format 所要求特征 进行纹理格式验证。
2. 让 layout 成为一个新的 GPUBindGroupLayout 对象。
3. 在this.[[device]]的设备时间线上发起后续初始化步骤。
4. 返回 layout。

---

Content timeline steps:
1. For each GPUBindGroupLayoutEntry entry in descriptor.entries:
    1. If entry.storageTexture is provided:
        1. ? Validate texture format required features for entry.storageTexture.format with this.[[device]].
2. Let layout be a new GPUBindGroupLayout object.
3. Issue the initialization steps on the Device timeline of this.
4. Return layout.

---
---

设备时间轴上初始化步骤：    
1. 如果以下任何条件不满足，则产生一个验证错误，使 layout 无效，并停止:    
    * this.device 的 this 是有效的。
    * 让限制(limits) 成为this.[[device]].[[limits]]。
    * 描述符中每个条目的绑定值是唯一的。
    * 描述符中的每个条目的绑定值必须 < limits.maxBindingsPerBindGroup。
    * descriptor.entries 不得超过绑定插槽限制的限制值。
    * 对于 descriptor.entries 中的每个 GPUBindGroupLayoutEntry 条目：
        * 必须精确提供 entry.buffer、 entry.sampler、 entry.texture 和 entry.storageTexture 中的一个。
        * entry.visibility 只包含 GPUShaderStage 中定义的位。
        * 如果 entry.visibility 包括 VERTEX：
            * entry.buffer?.type 就不能是 "storage"。
            * entry.storageTexture?.access不能是 "write-only"。
        * 如果 entry.texture?.multisampled 是 true：
            * entry.texture.viewDimension 是 "2d"。
            * entry.texture.sampleType 不是 "float"。
        * 如果 entry.storageTexture 被提供：
            * entry.storageTexture.viewDimension 不是 "cube "或 "cube-array"。
            * entry.storageTexture.format 必须是一个可以支持存储使用的格式。
2. 将 layout.[[descriptor]] 设置为 descriptor。
3. 将 layout.[[dynamicOffsetCount]] 设置为 提供了缓冲区，且 buffer.hasDynamicOffset 为 true 的描述符中条目的数量。
4. 对于 descriptor.entries 中的每个 GPUBindGroupLayoutEntry 条目：
    1. 将条目插入 layout.[[entryMap]] 中，其键名称为： entry.binding。

---

Device timeline initialization steps:
1. If any of the following conditions are unsatisfied generate a validation error, make layout invalid, and stop.
    * this is valid.
    * Let limits be this.[[device]].[[limits]].
    * The binding of each entry in descriptor is unique.
    * The binding of each entry in descriptor must be < limits.maxBindingsPerBindGroup.
    * descriptor.entries must not exceed the binding slot limits of limits.
    * For each GPUBindGroupLayoutEntry entry in descriptor.entries:
        * Exactly one of entry.buffer, entry.sampler, entry.texture, and entry.storageTexture is provided.
        * entry.visibility contains only bits defined in GPUShaderStage.
        * If entry.visibility includes VERTEX:
            * entry.buffer?.type must not be "storage".
            * entry.storageTexture?.access must not be "write-only".
        * If entry.texture?.multisampled is true:
            * entry.texture.viewDimension is "2d".
            * entry.texture.sampleType is not "float".
        * If entry.storageTexture is provided:
            * entry.storageTexture.viewDimension is not "cube" or "cube-array".
            * entry.storageTexture.format must be a format which can support storage usage.
2. Set layout.[[descriptor]] to descriptor.
3. Set layout.[[dynamicOffsetCount]] to the number of entries in descriptor where buffer is provided and buffer.hasDynamicOffset is true.
4. For each GPUBindGroupLayoutEntry entry in descriptor.entries:
    1. Insert entry into layout.[[entryMap]] with the key of entry.binding.

---
---

### 8.1.2.兼容性

两个 GPUBindGroupLayout 对象 a 和 b 要被认为是 组内等价的，当且仅当以下所有条件被满足：
* a.[[exclusivePipeline]] == b.[[exclusivePipeline]]。
* 对于任何绑定数量的绑定，满足以下条件之一：
    * a.[[entryMap]] 和 b.[[entryMap]] 都没有。
    * a.[[entryMap]][binding] == b.[[entryMap]][binding]。

如果绑定组的布局是组内等价的，它们可以在所有上下文中进行互换使用。

---

8.1.2. Compatibility
Two GPUBindGroupLayout objects a and b are considered group-equivalent if and only if all of the following conditions are satisfied:
* a.[[exclusivePipeline]] == b.[[exclusivePipeline]].
* for any binding number binding, one of the following conditions is satisfied:
    * it’s missing from both a.[[entryMap]] and b.[[entryMap]].
    * a.[[entryMap]][binding] == b.[[entryMap]][binding]

If bind groups layouts are group-equivalent they can be interchangeably used in all contents.

---
---

## 8.2. GPU组绑定(GPUBindGroup)

GPUBindGroup 定义了一个组内要绑定在一起的资源，以及在着色器阶段如何使用这些资源。

```
[Exposed=(Window, DedicatedWorker), SecureContext]
interface GPUBindGroup {
};
GPUBindGroup includes GPUObjectBase;
```

一个 GPUBindGroup 对象有以下内部插槽：        
**[[layout]]，类型为: GPUBindGroupLayout，只读**        
表示与此 GPUBindGroup相关的 GPUBindGroupLayout。

**[[entries]], 类型为: sequence\<GPUBindGroupEntry\>, 只读**    
表示此 GPUBindGroup 描述的 GPUBindGroupEntrys 的集合。

**[[usedResources]], 类型为有序地图\<subresource, list\<internal usage\>\>, 只读**    
表示此绑定组使用的缓冲区和纹理子资源的集合，与内部使用标志的列表有关。

---

8.2. GPUBindGroup
A GPUBindGroup defines a set of resources to be bound together in a group and how the resources are used in shader stages.

```
[Exposed=(Window, DedicatedWorker), SecureContext]
interface GPUBindGroup {
};
GPUBindGroup includes GPUObjectBase;
```

A GPUBindGroup object has the following internal slots:   
**[[layout]], of type GPUBindGroupLayout, readonly**    
The GPUBindGroupLayout associated with this GPUBindGroup.

**[[entries]], of type sequence\<GPUBindGroupEntry\>, readonly**    
The set of GPUBindGroupEntrys this GPUBindGroup describes.

**[[usedResources]], of type ordered map\<subresource, list\<internal usage\>\>, readonly**   
The set of buffer and texture subresources used by this bind group, associated with lists of the internal usage flags.

---
---

### 8.2.1. 绑定组创建

一个 GPUBindGroup 是通过 GPUDevice.createBindGroup() 创建的。

```
dictionary GPUBindGroupDescriptor : GPUObjectDescriptorBase {
    required GPUBindGroupLayout layout;
    required sequence<GPUBindGroupEntry> entries;
};
```

GPUBindGroupDescriptor 字典 有以下成员：    
**layout，类型为： GPUBindGroupLayout**    
这个绑定组要创建 GPUBindGroupLayout 的条目。    

**entries，类型为： sequence\<GPUBindGroupEntry\>。**   
一个条目列表，是由布局所描述的，描述了每个绑定要向着色器公开的资源。

---

8.2.1. Bind Group Creation

A GPUBindGroup is created via GPUDevice.createBindGroup().

```
dictionary GPUBindGroupDescriptor : GPUObjectDescriptorBase {
    required GPUBindGroupLayout layout;
    required sequence<GPUBindGroupEntry> entries;
};
```

GPUBindGroupDescriptor dictionaries have the following members:

**layout, of type GPUBindGroupLayout**    
The GPUBindGroupLayout the entries of this bind group will conform to.

**entries, of type sequence\<GPUBindGroupEntry\>**    
A list of entries describing the resources to expose to the shader for each binding described by the layout.

```
typedef (GPUSampler or GPUTextureView or GPUBufferBinding or GPUExternalTexture) GPUBindingResource;

dictionary GPUBindGroupEntry {
    required GPUIndex32 binding;
    required GPUBindingResource resource;
};
```

---
---

```
typedef (GPUSampler or GPUTextureView or GPUBufferBinding or GPUExternalTexture) GPUBindingResource;

dictionary GPUBindGroupEntry {
    required GPUIndex32 binding;
    required GPUBindingResource resource;
};
```

一个 GPUBindGroupEntry 描述了一个要在 GPUBindGroup 中绑定的单一资源，并有以下成员：

**binding，类型为： GPUIndex32**   
GPUBindGroup 中资源绑定的唯一标识，对应于 GPUBindGroupLayoutEntry.binding 和 GPUShaderModule 中的 @binding 属性。
**resource，类型为： GPUBindingResource**   
要绑定的资源，可以是 GPUSampler、GPUTextureView、GPUExternalTexture、或 GPUBufferBinding。

---

A GPUBindGroupEntry describes a single resource to be bound in a GPUBindGroup, and has the following members:

**binding, of type GPUIndex32**   
A unique identifier for a resource binding within the GPUBindGroup, corresponding to a GPUBindGroupLayoutEntry.binding and a @binding attribute in the GPUShaderModule.

**resource, of type GPUBindingResource**    
The resource to bind, which may be a GPUSampler, GPUTextureView, GPUExternalTexture, or GPUBufferBinding.

---
---

```
dictionary GPUBufferBinding {
    required GPUBuffer buffer;
    GPUSize64 offset = 0;
    GPUSize64 size;
};
```

一个 GPUBufferBinding 描述了一个缓冲区和可选的范围，并且作为一个资源进行绑定，并有以下成员：

**buffer，类型为： GPUBuffer**
要绑定的 GPUBuffer。

**offset，类型为： GPUSize64，默认为0**    
从缓冲区开始，到暴露给着色器后缓冲区绑定的范围开始处的偏移量，单位是字节。

**size，类型为：GPUSize64**   
缓冲区绑定的大小，单位是字节。如果没有提供，则指定为从偏移开始到缓冲区结束的范围。

---

```
dictionary GPUBufferBinding {
    required GPUBuffer buffer;
    GPUSize64 offset = 0;
    GPUSize64 size;
};
```

A GPUBufferBinding describes a buffer and optional range to bind as a resource, and has the following members:

**buffer, of type GPUBuffer**   
The GPUBuffer to bind.

**offset, of type GPUSize64, defaulting to 0**    
The offset, in bytes, from the beginning of buffer to the beginning of the range exposed to the shader by the buffer binding.

**size, of type GPUSize64**   
The size, in bytes, of the buffer binding. If not provided, specifies the range starting at offset and ending at the end of buffer.

---
---

**createBindGroup(descriptor) 函数**   

用于创建一个 GPUBindGroup。

调用于： GPUDevice 实例自身。

参数：

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| `descriptor` | `GPUBindGroupDescriptor` | ✘ | ✘ | 创建 GPUBindGroup 的描述符。 |

返回值： GPUBindGroup。

---

**createBindGroup(descriptor)**   

Creates a GPUBindGroup.

Called on: GPUDevice this.

Arguments:

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| `descriptor` | `GPUBindGroupDescriptor` | ✘ | ✘ | Description of the GPUBindGroup to create. |

Returns: GPUBindGroup

---
---

上下文时间轴步骤：
1. 让 bindGroup 成为一个新的 GPUBindGroup 对象。
2. 在这个的设备时间线上发起后续初始化步骤。
3. 返回 bindGroup。

---

**Content timeline steps:** 
1. Let bindGroup be a new GPUBindGroup object.
2. Issue the initialization steps on the Device timeline of this.
3. Return bindGroup.

---
---

设备时间轴初始化步骤：
1. 让limit成为this.[[device]].[[limit]]。
2. 如果以下任何条件不满足，产生一个验证错误，使 bindGroup 无效，并停止。
  * descriptor.layout 使用 this 来访问是有效的。
  * descriptor.layout的条目数量 正好等于 descriptor.entry 的数量。
  
    对于 descriptor.entries 中的每个 GPUBindGroupEntry bindingDescriptor：

  * 让 资源 成为 bindingDescriptor.resource。
  * 在 descriptor.layout.entries 中 正好有一个 GPUBindGroupLayoutEntry layoutBinding，使 layoutBinding.binding 等同于 bindingDescriptor.binding。

---

**Device timeline initialization steps:**   
1. Let limits be this.[[device]].[[limits]].
2. If any of the following conditions are unsatisfied generate a validation error, make bindGroup invalid, and stop.
    * descriptor.layout is valid to use with this.
    * The number of entries of descriptor.layout is exactly equal to the number of descriptor.entries.    
  
    For each GPUBindGroupEntry bindingDescriptor in descriptor.entries:  

    * Let resource be bindingDescriptor.resource.
    * There is exactly one GPUBindGroupLayoutEntry layoutBinding in descriptor.layout.entries such that layoutBinding.binding equals to bindingDescriptor.binding.
  
  ---
  ---

  ***接上面，还在是验证的条件***

* 如果 layoutBinding 所定义的 绑定成员是：    
**sampler**   
  * 资源就是一个 GPUSampler。
  * 资源用this来访问是有效的。
  * 如果 layoutBinding.sampler.type 是：    
    "filtering"   
    resource.[[isComparison]] 为 false。    

    "non-filtering"   
    resource.[[isFiltering]] 为 false。 resource.[[isComparison]] 为 false。

    "comparison"
    resource.[[isComparison]] 为 true。
  
  * 
  **texture**
  * 资源是一个 GPUTextureView。
  * 资源用this来访问是有效的。
  * 让 texture 成为 resource.[[texture]]。
  * layoutBinding.texture.viewDimension 等于资源的尺寸。
  * layoutBinding.texture.sampleType 与资源的格式兼容。
  * texture 的用法包括 TEXTURE_BINDING。
  * 如果 layoutBinding.texture.multisampled 为 true，texture的 sampleCount 大于1，否则 texture的sampleCount 为 1。

  ---

  * If the defined binding member for layoutBinding is    
        **sampler**   
        * resource is a GPUSampler.
        * resource is valid to use with this.
        * If layoutBinding.sampler.type is:   
        **"filtering"**   
          * resource.[[isComparison]] is false.     
        **"non-filtering"**   
          * resource.[[isFiltering]] is false. resource.[[isComparison]] is false.    
        **"comparison"**
          * resource.[[isComparison]] is true.

        **texture**   
        * resource is a GPUTextureView.
        * resource is valid to use with this.
        * Let texture be resource.[[texture]].
        * layoutBinding.texture.viewDimension is equal to resource’s dimension.
        * layoutBinding.texture.sampleType is compatible with resource’s format.
        * texture’s usage includes TEXTURE_BINDING.
        * If layoutBinding.texture.multisampled is true, texture’s sampleCount > 1, Otherwise texture’s sampleCount is 1.

---
---

**storageTexture** 
* 资源是一个 GPUTextureView。
* 资源用this来访问是有效的。
* 让texture 成为 resource.[[texture]]。
* layoutBinding.storageTexture.viewDimension 等于资源的尺寸。
* layoutBinding.storageTexture.format 等于 resource.[[descriptor]].format。
* 纹理的用法包括 STORAGE_BINDING。
* resource.[[descriptor]].mipLevelCount 必须为 1。

---
        **storageTexture**    
        * resource is a GPUTextureView.
        * resource is valid to use with this.
        * Let texture be resource.[[texture]].
        * layoutBinding.storageTexture.viewDimension is equal to resource’s dimension.
        * layoutBinding.storageTexture.format is equal to resource.[[descriptor]].format.
        * texture’s usage includes STORAGE_BINDING.
        * resource.[[descriptor]].mipLevelCount must be 1.

---
---

**buffer**
* 资源是一个GPUBufferBinding。
* 资源用this来访问是有效的。
* 由 resource.offset 和 resource.size 指定的绑定部分驻留在缓冲区内，并且其大小非零。
* 有效的缓冲区绑定大小（资源）≥ layoutBinding.buffer.minBindingSize。
* 如果 layoutBinding.buffer.type 是：   
    "uniform"    
    * resource.buffer.usage 包括 UNIFORM。
    * 有效缓冲区绑定大小（资源）≤ limits.maxUniformBufferBindingSize。
    * resource.offset 是 limit.minUniformBufferOffsetAlignment 的倍数。
    
    "storage" 或 "read-only-storage"
    * resource.buffer.usage 包括 STORAGE。
    * 有效缓冲区绑定大小（资源）≤ limits.maxStorageBufferBindingSize。
    * 有效的缓冲区绑定大小（资源）是4的倍数。
    * resource.offset 是 limit.minStorageBufferOffsetAlignment 的倍数。

**externalTexture**
* 资源是一个 GPUExternalTexture。
* 资源用this来访问是有效的。

---

        **buffer**    
        * resource is a GPUBufferBinding.
        * resource.buffer is valid to use with this.
        * The bound part designated by resource.offset and resource.size resides inside the buffer and has non-zero size.
        * effective buffer binding size(resource) ≥ layoutBinding.buffer.minBindingSize.
        * If layoutBinding.buffer.type is     
        **"uniform"**       
          * resource.buffer.usage includes UNIFORM.
          * effective buffer binding size(resource) ≤ limits.maxUniformBufferBindingSize.
          * resource.offset is a multiple of limits.minUniformBufferOffsetAlignment.
        **"storage" or "read-only-storage"**        
          * resource.buffer.usage includes STORAGE.
          * effective buffer binding size(resource) ≤ limits.maxStorageBufferBindingSize.
          * effective buffer binding size(resource) is a multiple of 4.
          * resource.offset is a multiple of limits.minStorageBufferOffsetAlignment.

    **externalTexture**    
    * resource is a GPUExternalTexture.
    * resource is valid to use with this.

---
---

3. 让 bindGroup.[[layout]] = descriptor.layout。
4. 让 bindGroup.[[entry]] = descriptor.entry。
5. 让 bindGroup.[[usedResources]] = {}.
6. 对于 descriptor.entries 中的每个 GPUBindGroupEntry bindingDescriptor：
    1. 让 internalUsage 成为 layoutBinding 的绑定用法。
    2. 资源能访问的 每个子资源 都被添加到 [[usedResources]]，作为内部用法(internalUsage)。

---
3. Let bindGroup.[[layout]] = descriptor.layout.
4. Let bindGroup.[[entries]] = descriptor.entries.
5. Let bindGroup.[[usedResources]] = {}.
6. For each GPUBindGroupEntry bindingDescriptor in descriptor.entries:
    1. Let internalUsage be the binding usage for layoutBinding.
    2. Each subresource seen by resource is added to [[usedResources]] as internalUsage.

---
---

**effective buffer binding size(binding)**    
即 计算有效的绑定缓冲区大小函数：

1. 如果没有提供 binding.size：
    1. 返回 max(0, binding.buffer.size - binding.offset)；
2. 返回 binding.size。

两个 GPUBufferBinding 对象 a 和 b 被认为是 buffer-binding-aliasing，当且仅当以下所有情况为真时：
* a.buffer == b.buffer
* 由 a.offset 和 a.size 所形成的范围 与 b.offset 和 b.size 所形成的范围相交。

>> ISSUE 11 当大小不确定时，一个范围是如何由 偏移量/大小 定义形成的。

---

**effective buffer binding size(binding)**    
1. If binding.size is not provided:
    1. Return max(0, binding.buffer.size - binding.offset);
2. Return binding.size.

Two GPUBufferBinding objects a and b are considered buffer-binding-aliasing if and only if all of the following are true:
* a.buffer == b.buffer
* The range formed by a.offset and a.size intersects the range formed by b.offset and b.size.

>> ISSUE 11 Define how a range is formed by offset/size when size can be undefined.

---
---

## 8.3. GPU 管道布局(GPUPipelineLayout)

GPUPipelineLayout 定义了，在 setBindGroup() 进行命令编码时，设置的所有 GPUBindGroup 对象的资源 与 由 GPURenderCommandsMixin.setPipeline 或 GPUComputePassEncoder.setPipeline 设置的管道的着色器集合 之间的映射关系。

一个资源的完整绑定地址可以定义为三段式：
1. shader stage mask，定义那个资源对其是可见的
2. 绑定组索引
3. 绑定编号

这个地址的构成也可以被看作是管道的绑定空间。一个 GPUBindGroup（与相应的 GPUBindGroupLayout）涵盖了该空间的一个固定的绑定组索引。其所包含的绑定需要是着色器在这个绑定组索引上使用的资源的超集。

```
[Exposed=(Window, DedicatedWorker), SecureContext]
interface GPUPipelineLayout {
};
GPUPipelineLayout includes GPUObjectBase;
```

---

8.3. GPUPipelineLayout

A GPUPipelineLayout defines the mapping between resources of all GPUBindGroup objects set up during command encoding in setBindGroup(), and the shaders of the pipeline set by GPURenderCommandsMixin.setPipeline or GPUComputePassEncoder.setPipeline.

The full binding address of a resource can be defined as a trio of:
1. shader stage mask, to which the resource is visible
2. bind group index
3. binding number

The components of this address can also be seen as the binding space of a pipeline. A GPUBindGroup (with the corresponding GPUBindGroupLayout) covers that space for a fixed bind group index. The contained bindings need to be a superset of the resources used by the shader at this bind group index.

```
[Exposed=(Window, DedicatedWorker), SecureContext]
interface GPUPipelineLayout {
};
GPUPipelineLayout includes GPUObjectBase;
```

---
---

GPUPipelineLayout 有以下内部插槽：

**[[bindGroupLayouts]], 类型为： list\<GPUBindGroupLayout\>.**
在创建时，GPUPipelineLayoutDescriptor.bindGroupLayouts 中提供的 GPUBindGroupLayout 对象。

**注意：** 为许多 GPURenderPipeline 或 GPUComputePipeline 管道使用相同的 GPUPipelineLayout，可以保证当这些管道之间有切换时，用户代理不需要在内部重新绑定任何资源。

---

GPUPipelineLayout has the following internal slots:

**[[bindGroupLayouts]], of type list\<GPUBindGroupLayout\>**    
The GPUBindGroupLayout objects provided at creation in GPUPipelineLayoutDescriptor.bindGroupLayouts.

**NOTE:** using the same GPUPipelineLayout for many GPURenderPipeline or GPUComputePipeline pipelines guarantees that the user agent doesn’t need to rebind any resources internally when there is a switch between these pipelines.


```
EXAMPLE 18 

// 例子 18

// GPUComputePipeline 对象 X 是用 GPUPipelineLayout.bindGroupLayouts A, B, C创建的。
// GPUComputePipeline 对象 Y 是用 GPUPipelineLayout.bindGroupLayouts A, D, C创建的。
// 假设命令编码序列是有两个派发：

// GPUComputePipeline object X was created with GPUPipelineLayout.bindGroupLayouts A, B, C. GPUComputePipeline object Y was created with GPUPipelineLayout.bindGroupLayouts A, D, C. Supposing the command encoding sequence has two dispatches:

1. setBindGroup(0, ...)
2. setBindGroup(1, ...)
3. setBindGroup(2, ...)
4. setPipeline(X)
5. dispatchWorkgroups()
6. setBindGroup(1, ...)
7. setPipeline(Y)
8. dispatchWorkgroups()

// 在这种情况下，尽管 GPUPipelineLayout.bindGroupLayouts 的索引2处的 GPUBindGroupLayout 或 插槽2处的GPUBindGroup 都没有改变，但用户代理必须为第二次调度重新绑定组插槽2。

In this scenario, the user agent would have to re-bind the group slot 2 for the second dispatch, even though neither the GPUBindGroupLayout at index 2 of GPUPipelineLayout.bindGroupLayouts, or the GPUBindGroup at slot 2, change.
```
---
---

**注意：** GPUPipelineLayout 的预期用法是将最常见和最少经常变化的绑定组放在布局的 "底部"，即较低的绑定组槽号，如0或1。一个绑定组在绘制调用之间需要改变的频率越高，其索引就应该越高。这个一般准则允许用户代理在绘制调用之间尽量减少状态变化，从而降低CPU的开销。

---

**NOTE:** the expected usage of the GPUPipelineLayout is placing the most common and the least frequently changing bind groups at the "bottom" of the layout, meaning lower bind group slot numbers, like 0 or 1. The more frequently a bind group needs to change between draw calls, the higher its index should be. This general guideline allows the user agent to minimize state changes between draw calls, and consequently lower the CPU overhead.

---
---

### 8.3.1. 管道布局创建(Pipeline Layout Creation) 

一个 GPUPipelineLayout 是通过 GPUDevice.createPipelineLayout() 创建的。

```
dictionary GPUPipelineLayoutDescriptor : GPUObjectDescriptorBase {
    required sequence<GPUBindGroupLayout> bindGroupLayouts;
};
```

GPUPipelineLayoutDescriptor 字典定义了一个管道 所使用的所有 GPUBindGroupLayouts，并具有以下成员：

**bindGroupLayouts，类型为: sequence\<GPUBindGroupLayout\>。**      
表示管道将使用的 GPUBindGroupLayouts 的列表。每个元素对应于 GPUShaderModule 中的 @group属性，第 N 个元素对应于@group(N)。

---

8.3.1. Pipeline Layout Creation

A GPUPipelineLayout is created via GPUDevice.createPipelineLayout().

```
dictionary GPUPipelineLayoutDescriptor : GPUObjectDescriptorBase {
    required sequence<GPUBindGroupLayout> bindGroupLayouts;
};
```

GPUPipelineLayoutDescriptor dictionaries define all the GPUBindGroupLayouts used by a pipeline, and have the following members:

**bindGroupLayouts, of type sequence\<GPUBindGroupLayout\>**    
A list of GPUBindGroupLayouts the pipeline will use. Each element corresponds to a @group attribute in the GPUShaderModule, with the Nth element corresponding with @group(N).

---
---

**createPipelineLayout(descriptor)**    

用于创建一个 GPUPipelineLayout

调用于： GPUDevice 实例自身。

参数： 

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| `descriptor` | `GPUPipelineLayoutDescriptor` | ✘ | ✘ |  创建 GPUPipelineLayout 的描述符. |

返回值： GPUPipelineLayout

---

**createPipelineLayout(descriptor)**    

Creates a GPUPipelineLayout.

Called on: GPUDevice this.

Arguments:

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| `descriptor` | `GPUPipelineLayoutDescriptor` | ✘ | ✘ | Description of the GPUPipelineLayout to create. |

Returns: GPUPipelineLayout

---
---

上下文时间轴上的步骤：
1. 让 pl 成为一个新的 GPUPipelineLayout 对象。
2. 在这个的设备时间线上发起后续初始化步骤。
3. 返回pl。

---

**Content timeline steps:**   
1. Let pl be a new GPUPipelineLayout object.
2. Issue the initialization steps on the Device timeline of this.
3. Return pl.

---
---

**设备时间轴初始化步骤：**
1. 让 limits 的值为 this.[[device]].[[limits]]。
2. 让 allEntries 为 descriptor.bindGroupLayouts 中的所有bgl 的 bgl.[[descriptor]].entries 串联的结果。
3. 如果以下任何条件不满足，就会产生一个验证错误，使 pl 无效，并停止：
    * descriptor.bindGroupLayouts 中的每一个 GPUBindGroupLayout 都必须是可以通过this有效访问的，并且其 [[exclusivePipeline]] 为 null。
    * descriptor.bindGroupLayouts 的大小必须 ≤ limits.maxBindGroups。
    * allEntries 必须不超过的绑定插槽限制的限制。
4. 将 pl.[[bindGroupLayouts]] 设置为 descriptor.bindGroupLayouts。

**注意：** 如果两个 GPUPipelineLayout 对象 在任何用法上都被视为等同的条件是： 它们内部 [[bindGroupLayouts]] 序列所包含的 GPUBindGroupLayout 对象是 组等价的。

---

**Device timeline initialization steps:**     
1. Let limits be this.[[device]].[[limits]].
2. Let allEntries be the result of concatenating bgl.[[descriptor]].entries for all bgl in descriptor.bindGroupLayouts.
3. If any of the following conditions are unsatisfied generate a validation error, make pl invalid, and stop.
    * Every GPUBindGroupLayout in descriptor.bindGroupLayouts must be valid to use with this and have a [[exclusivePipeline]] of null.
    * The size of descriptor.bindGroupLayouts must be ≤ limits.maxBindGroups.
    * allEntries must not exceed the binding slot limits of limits.
4. Set the pl.[[bindGroupLayouts]] to descriptor.bindGroupLayouts.

**NOTE:** two GPUPipelineLayout objects are considered equivalent for any usage if their internal [[bindGroupLayouts]] sequences contain GPUBindGroupLayout objects that are group-equivalent.

---
---

## 8.4. 例子

***由于此节只有一个例子，不做翻译了***

---

8.4. Example

```
// EXAMPLE 19

// 创建一个 GPUBindGroupLayout，描述一个具有 一个uniform缓冲器、一个纹理和一个采样器的绑定。然后用GPUBindGroupLayout 创建一个 GPUBindGroup 和一个 GPUPipelineLayout。

// Create a GPUBindGroupLayout that describes a binding with a uniform buffer, a texture, and a sampler. Then create a GPUBindGroup and a GPUPipelineLayout using the GPUBindGroupLayout.

const bindGroupLayout = gpuDevice.createBindGroupLayout({
    entries: [{
        binding: 0,
        visibility: GPUShaderStage.VERTEX | GPUShaderStage.FRAGMENT,
        buffer: {}
    }, {
        binding: 1,
        visibility: GPUShaderStage.FRAGMENT,
        texture: {}
    }, {
        binding: 2,
        visibility: GPUShaderStage.FRAGMENT,
        sampler: {}
    }]
});

const bindGroup = gpuDevice.createBindGroup({
    layout: bindGroupLayout,
    entries: [{
        binding: 0,
        resource: { buffer: buffer },
    }, {
        binding: 1,
        resource: texture
    }, {
        binding: 2,
        resource: sampler
    }]
});

const pipelineLayout = gpuDevice.createPipelineLayout({
    bindGroupLayouts: [bindGroupLayout]
});

```




