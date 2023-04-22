# WebGPU

W3C Working Draft, 24 March 2023

More details about this document

This version:

https://www.w3.org/TR/2023/WD-webgpu-20230324/

---
---

# 10. 管道

一个管道，无论是 GPUComputePipeline 还是GPURenderPipeline，都代表了由 GPU硬件、驱动程序 和 用户代理等组合所完成的完整功能，它可以处理 着色器中绑定的 和 顶点缓冲区中 的输入数据，并产生一些输出，如输出渲染目标的颜色。

从结构上看，管道由一连串的可编程阶段（着色器）和 固定功能状态组成的，固定功能如混合模式。

**注意：** 在内部，根据不同的目标平台，驱动程序可以将一些固定功能状态转换为着色器代码，并将其与用户提供的着色器链接起来。这种链接是对象可作为一个整体被创建的原因之一。

这种组合状态被创建为一个单一的对象 (一个 GPUComputePipeline 或 GPURenderPipeline ），并且可以使用一个命令进行切换（分别是 GPUComputePassEncoder.setPipeline() 或 GPURenderCommandsMixin.setPipeline()）。

---

10. Pipelines
A pipeline, be it GPUComputePipeline or GPURenderPipeline, represents the complete function done by a combination of the GPU hardware, the driver, and the user agent, that process the input data in the shape of bindings and vertex buffers, and produces some output, like the colors in the output render targets.

Structurally, the pipeline consists of a sequence of programmable stages (shaders) and fixed-function states, such as the blending modes.

NOTE: Internally, depending on the target platform, the driver may convert some of the fixed-function states into shader code, and link it together with the shaders provided by the user. This linking is one of the reason the object is created as a whole.

This combination state is created as a single object (a GPUComputePipeline or GPURenderPipeline) and switched using one command (GPUComputePassEncoder.setPipeline() or GPURenderCommandsMixin.setPipeline() respectively).

---
---

有两种方法来创建管道：

方法一： 立即创建管道   
createComputePipeline() 和 createRenderPipeline() 函数返回一个管道对象，可以立即在一个运行编码器中使用。

当这失败时，管道对象将是无效的，调用将产生一个验证错误或一个内部错误。

**注意：** 一个句柄对象会立即返回，但实际的管道创建不是同步的。如果管道创建需要很长的时间，这可能会在创建调用 和 执行首次使用管道的submit()函数调用之间的某个时间点引起设备时间线的停顿。这个时间点并不明确，但最有可能的是：在创建时，在 setPipeline() 中第一次使用管道时，或者，在该 GPUCommandEncoder 或 GPURenderBundleEncoder 的相应 finish()函数调用时，或者，在该 GPUCommandBuffer 的submit() 函数调用时。

方法二： 异步管线创建   
createComputePipelineAsync() 和 createRenderPipelineAsync() 返回一个 Promise，该 Promise 对象在管道创建完成后会成功的返回一个管道对象。

当这一过程失败时，Promise 会被设置为 rejected, 其值为：GPUPipelineError。

---

There are two ways to create pipelines:

immediate pipeline creation
createComputePipeline() and createRenderPipeline() return a pipeline object which can be used immediately in a pass encoder.

When this fails, the pipeline object will be invalid and the call will generate either a validation error or an internal error.

NOTE: A handle object is returned immediately, but actual pipeline creation is not synchronous. If pipeline creation takes a long time, this can incur a stall in the device timeline at some point between the creation call and execution of the submit() in which it is first used. The point is unspecified, but most likely to be one of: at creation, at the first usage of the pipeline in setPipeline(), at the corresponding finish() of that GPUCommandEncoder or GPURenderBundleEncoder, or at submit() of that GPUCommandBuffer.

async pipeline creation
createComputePipelineAsync() and createRenderPipelineAsync() return a Promise which resolves to a pipeline object when creation of the pipeline has completed.

When this fails, the Promise rejects with a GPUPipelineError.

---
---

GPUPipelineError 描述一个管道创建失败的错误。

```
[Exposed=(Window, DedicatedWorker), SecureContext, Serializable]
interface GPUPipelineError : DOMException {
    constructor((DOMString or undefined) message, GPUPipelineErrorInit options);
    readonly attribute GPUPipelineErrorReason reason;
};

dictionary GPUPipelineErrorInit {
    required GPUPipelineErrorReason reason;
};

enum GPUPipelineErrorReason {
    "validation",
    "internal"
};
```

---

GPUPipelineError describes a pipeline creation failure.
```
[Exposed=(Window, DedicatedWorker), SecureContext, Serializable]
interface GPUPipelineError : DOMException {
    constructor((DOMString or undefined) message, GPUPipelineErrorInit options);
    readonly attribute GPUPipelineErrorReason reason;
};

dictionary GPUPipelineErrorInit {
    required GPUPipelineErrorReason reason;
};

enum GPUPipelineErrorReason {
    "validation",
    "internal"
};
```

---
---

GPUPipelineError 构造函数：

constructor()

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| `message` | `(DOMString or undefined)` | ✘ | ✘ | 基于 DOMException 的错误信息。 |
| `options` | `GPUPipelineErrorInit` | ✘ | ✘ | 专门针对 GPUPipelineError 的选项。 |

1. 设置 this.name 为 "GPUPipelineError"。
2. 设置 this.message 为 message ?？ ""。
3. 设置 this.reason 为 options.reason。

---

GPUPipelineError constructor:

constructor()

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| `message` | `(DOMString or undefined)` | ✘ | ✘ | Error message of the base DOMException. |
| `options` | `GPUPipelineErrorInit` | ✘ | ✘ | Options specific to GPUPipelineError. |

1. Set this.name to "GPUPipelineError".
2. Set this.message to message ?? "".
3. Set this.reason to options.reason.

---
---

GPUPipelineError 有以下属性：

**reason，类型为： GPUPipelineErrorReason，只读**    
一个只读的备用插槽支持的属性，暴露了在管道创建过程中发生的错误，其类型为 GPUPipelineErrorReason：
* "validation"： 表明是一个验证错误。
* "internal"： 表明是一个内部错误。

GPUPipelineError 对象是可序列化的对象。

针对给定值和序列化数据的 序列化步骤是：
1. 针对给定值和序列化数据 在 DOMException 上运行序列化步骤。

针对给定值和序列化数据的 反序列化步骤是：
1. 针对给定值和序列化数据 在 DOMException 上运行反序列化步骤。

---

GPUPipelineError has the following attributes:

**reason, of type GPUPipelineErrorReason, readonly**      
A read-only slot-backed attribute exposing the type of error encountered in pipeline creation as a GPUPipelineErrorReason:       
   
  * "validation": A validation error.
  * "internal": An internal error.

GPUPipelineError objects are serializable objects.

Their serialization steps, given value and serialized, are:
1. Run the DOMException serialization steps given value and serialized.

Their deserialization steps, given value and serialized, are:
1. Run the DOMException deserialization steps given value and serialized.

---
---

10.1. 基础管道

```
enum GPUAutoLayoutMode {
    "auto"
};

dictionary GPUPipelineDescriptorBase : GPUObjectDescriptorBase {
    required (GPUPipelineLayout or GPUAutoLayoutMode) layout;
};

interface mixin GPUPipelineBase {
    [NewObject] GPUBindGroupLayout getBindGroupLayout(unsigned long index);
};
```

---

10.1. Base pipelines

```
enum GPUAutoLayoutMode {
    "auto"
};

dictionary GPUPipelineDescriptorBase : GPUObjectDescriptorBase {
    required (GPUPipelineLayout or GPUAutoLayoutMode) layout;
};

interface mixin GPUPipelineBase {
    [NewObject] GPUBindGroupLayout getBindGroupLayout(unsigned long index);
};
```

---
---

GPUPipelineBase 有以下内部插槽：

**[[layout]]，类型为GPUPipelineLayout**   
用于描述针对此管道的资源布局的定义。

---

GPUPipelineBase has the following internal slots:

**[[layout]], of type GPUPipelineLayout**   
The definition of the layout of resources which can be used with this.

---
---

GPUPipelineBase 有以下方法：

**getBindGroupLayout(index)**   
用于获取一个 与 GPUPipelineBase 中位置为 index 处的 GPUBindGroupLayout 相兼容的 GPUBindGroupLayout。

调用于： GPUPipelineBase 实例对象自身。

参数： 

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| `index` | `unsigned long` | ✘ | ✘ | 管道布局的[[bindGroupLayouts]]序列的 index。 |

返回值： GPUBindGroupLayout。

---

GPUPipelineBase has the following methods:

**getBindGroupLayout(index)**     
Gets a GPUBindGroupLayout that is compatible with the GPUPipelineBase's GPUBindGroupLayout at index.

Called on: GPUPipelineBase this

Arguments:

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| `index` | `unsigned long` | ✘ | ✘ | Index into the pipeline layout’s [[bindGroupLayouts]] sequence. |

Returns: GPUBindGroupLayout

---
---

上下文时间轴步骤：
1. 让 layout 成为一个新的 GPUBindGroupLayout 对象。
2. 在这个实例的设备时间线上发起后续的初始化步骤。
3. 返回 layout。

---

Content timeline steps:
1. Let layout be a new GPUBindGroupLayout object.
2. Issue the initialization steps on the Device timeline of this.
3. Return layout.

---
---

设备时间轴初始化步骤：    
1. 如果以下任何条件不满足，产生一个验证错误，使 layout 无效，并停止。
    * this 指针是有效的。
    * index < this.[[layout]].[[bindGroupLayouts]] 的大小。

2. 初始化layout，因此它是 this.[[layout]].[[bindGroupLayouts]][index] 的副本。

**注意：** GPUBindGroupLayout 只能通过值使用，而不是通过引用使用，所以这相当于在一个新的包装器中返回相同的内部对象。每次都会返回一个新的 GPUBindGroupLayout 包装器，以避免上下文时间线和设备时间线之间的往返交替。

---

Device timeline initialization steps:   
1. If any of the following conditions are unsatisfied generate a validation error, make layout invalid, and stop.
    * this is valid.
    * index < the size of this.[[layout]].[[bindGroupLayouts]]

2. Initialize layout so it is a copy of this.[[layout]].[[bindGroupLayouts]][index].

**NOTE:** GPUBindGroupLayout is only ever used by-value, not by-reference, so this is equivalent to returning the same internal object in a new wrapper. A new GPUBindGroupLayout wrapper is returned each time to avoid a round-trip between the Content timeline and the Device timeline.

---
---

### 10.1.1.默认的管道布局

在创建一个 GPUPipelineBase 对象时，如果参数 layout 是设置为 "auto" 时，将会创建 并使用 一个默认 layout。

**注意：** 默认布局是为简单管道而提供的便利机制，但在大多数情况下建议使用显式布局。从默认布局创建的绑定组不能用于其他管道，而且默认布局的结构在改变着色器时可能会发生变化，会导致意外的绑定组创建错误。

---

10.1.1. Default pipeline layout
A GPUPipelineBase object that was created with a layout set to "auto" has a default layout created and used instead.

NOTE: Default layouts are provided as a convenience for simple pipelines, but use of explicit layouts is recommended in most cases. Bind groups created from default layouts cannot be used with other pipelines, and the structure of the default layout may change when altering shaders, causing unexpected bind group creation errors.

---
---

要为 GPUPipelineBase 管道创建一个默认的管道布局，请运行以下步骤：
1. 让 groupCount 为 0。
2. 让 groupDescs 是基于新的 GPUBindGroupLayoutDescriptor 对象而创建的 device.[[limit]].maxBindGroups 系列。
3. 对于 groupDescs 中的每个 groupDesc：
    1. 将 groupDesc.entries 设置为一个 空序列。

---

To create a default pipeline layout for GPUPipelineBase pipeline, run the following steps:

1. Let groupCount be 0.
2. Let groupDescs be a sequence of device.[[limits]].maxBindGroups new GPUBindGroupLayoutDescriptor objects.
3. For each groupDesc in groupDescs:
    1. Set groupDesc.entries to an empty sequence.

---
---

***接上 第4个复杂的步骤***

4. 对于用于创建管道的描述符中的每个 GPUProgrammableStage stageDesc 进行以下操作：
    1. 让 shaderStage 成为 stageDesc.module 中 stageDesc.entryPoint 的 GPUShaderStageFlags。
    2. 对于 stageDesc 静态使用的每个资源的资源：
        1. 让 group 成为资源的 "组"装饰。
        2. 让 binding 成为资源的 "绑定"装饰。
        3. 让 entry 成为一个新的 GPUBindGroupLayoutEntry。
        4. 将 entry.binding 设为 binding。
        5. 将 entry.visibility 设置为 shaderStage。
        6. 如果资源是为采样器绑定的：
            1. 让 samplerLayout 成为一个新的 GPUSamplerBindingLayout。
            2. 将 entry.sampler 设置为 samplerLayout。
        7. 如果资源是用于 比较采样器绑定：
            1. 让 samplerLayout 成为一个新的 GPUSamplerBindingLayout。
            2. 设置 samplerLayout.type 为 "comparison"。
            3. 将 entry.sampler 设置为 samplerLayout。
        8. 如果资源是用于 缓冲区绑定：
            1. 让 bufferLayout 成为一个新的 GPUBufferBindingLayout。
            2. 设置 bufferLayout.minBindingSize 为资源的最小缓冲区绑定大小。
            3. 如果资源是用于只读存储的缓冲区：
                1. 将 bufferLayout.type 设置为 "read-only-storage"。
            4. 如果资源是用于存储缓冲区：
                1. 将 bufferLayout.type 设为 "storage"。
            5. 将 entry.buffer 设置为 bufferLayout。
        9. 如果资源是用于 采样的纹理绑定：
            1. 让 textureLayout 成为一个新的 GPUTextureBindingLayout。
            2. 如果资源是一个深度纹理绑定：
                * 设置 textureLayout.sampleType 为 "depth"。

                否则，如果资源的采样类型是：

                **f32，并且存在一个静态使用的资源，其中有一个textureSample*的内建值**     
                将 textureLayout.sampleType 设置为 "float"。

                **f32，否则**   
                将 textureLayout.sampleType 设置为 "unfilterable-float"。

                **i32**   
                将 textureLayout.sampleType 设置为 "sint"。

                **u32**   
                将 textureLayout.sampleType 设置为 "uint"。

            3. 设置 textureLayout.viewDimension 为资源的尺寸。
            4. 如果资源是用于多采样的纹理：
                1. 将 textureLayout.multisampled 设置为 true。
            5. 将 entry.texture 设置为 textureLayout。

        10. 如果资源是用于存储纹理绑定：
            1. 让 storageTextureLayout 成为一个新的 GPUStorageTextureBindingLayout。
            2. 将 storageTextureLayout.format 设为资源的格式。
            3. 将 storageTextureLayout.viewDimension 设置为资源的尺寸。
            4. 如果资源是用于只写的存储纹理：
                1. 将 storageTextureLayout.access 设置为 "write-only"。
            5. 将 entry.storageTexture 设置为 storageTextureLayout。  
        11. 将 groupCount 设置为 max(groupCount, group + 1)。
        12. 如果 groupDescs[group] 有一个先前的条目 previousEntry，并且先前条目的binding等于此时的binding：
            1. 如果条目的可见性 与 previousEntry 不同：
                1. 将 entry.visibility 中设置的位添加先前到 previousEntry.visibility 中。
            2. 如果资源是用于缓冲区绑定的，并且条目的 buffer.minBindingSize 比 先前的 previousEntry 大：
                1. 将 previousEntry.buffer.minBindingSize 设置为 entry.buffer.minBindingSize。
            3. 如果资源是采样纹理绑定，并且条目的 texture.sampleType 与先前的 previousEntry 不同，并且条目和先前的 previousEntry 的 texture.sampleType 都是 "float" 或 "unfilterable-float"：
                1. 将 previousEntry.texture.sampleType 设置为 "float"。
            4. 如果 entry 和 先前的 previousEntry 之间有任何其他属性不相等：
               1. 返回 null（这将会导致管道的创建失败）。
        13. 否则
            1.  将条目追加到 groupDescs[group]。

---

4. For each GPUProgrammableStage stageDesc in the descriptor used to create pipeline:
    1. Let shaderStage be the GPUShaderStageFlags for stageDesc.entryPoint in stageDesc.module.
    2. For each resource resource statically used by stageDesc:
        1. Let group be resource’s "group" decoration.
        2. Let binding be resource’s "binding" decoration.
        3. Let entry be a new GPUBindGroupLayoutEntry.
        4. Set entry.binding to binding.
        5. Set entry.visibility to shaderStage.
        6. If resource is for a sampler binding:
            1. Let samplerLayout be a new GPUSamplerBindingLayout.
            2. Set entry.sampler to samplerLayout.
        7. If resource is for a comparison sampler binding:
            1. Let samplerLayout be a new GPUSamplerBindingLayout.
            2. Set samplerLayout.type to "comparison".
            3. Set entry.sampler to samplerLayout.
        8. If resource is for a buffer binding:
            1. Let bufferLayout be a new GPUBufferBindingLayout.
            2. Set bufferLayout.minBindingSize to resource’s minimum buffer binding size.
            3. If resource is for a read-only storage buffer:
                1. Set bufferLayout.type to "read-only-storage".
            4. If resource is for a storage buffer:
               1. Set bufferLayout.type to "storage".
            5. Set entry.buffer to bufferLayout.
         9. If resource is for a sampled texture binding:
            1.  Let textureLayout be a new GPUTextureBindingLayout.
            2.  If resource is a depth texture binding:
                * Set textureLayout.sampleType to "depth"
                
                Else if the sampled type of resource is:
                
                * **f32 and there exists a static use of resource with a textureSample* builtin in**    
                Set textureLayout.sampleType to "float"

                * **f32 otherwise**   
                Set textureLayout.sampleType to "unfilterable-float"

                * **i32**   
                Set textureLayout.sampleType to "sint"

                * **u32**   
                Set textureLayout.sampleType to "uint"

              3. Set textureLayout.viewDimension to resource’s dimension.
              4. If resource is for a multisampled texture:
                 1. Set textureLayout.multisampled to true.
              5. Set entry.texture to textureLayout.
          10. If resource is for a storage texture binding:
              1. Let storageTextureLayout be a new GPUStorageTextureBindingLayout.
              2. Set storageTextureLayout.format to resource’s format.
              3. Set storageTextureLayout.viewDimension to resource’s dimension.
              4. If resource is for a write-only storage texture:
                 1. Set storageTextureLayout.access to "write-only".
              5. Set entry.storageTexture to storageTextureLayout.
          11. Set groupCount to max(groupCount, group + 1).
          12. If groupDescs[group] has an entry previousEntry with binding equal to binding:
              1.  If entry has different visibility than previousEntry:
                  1.  Add the bits set in entry.visibility into previousEntry.visibility
              2.  If resource is for a buffer binding and entry has greater buffer.minBindingSize than previousEntry:
                  1.  Set previousEntry.buffer.minBindingSize to entry.buffer.minBindingSize.
              3.  If resource is a sampled texture binding and entry has different texture.sampleType than previousEntry and both entry and previousEntry have texture.sampleType of either "float" or "unfilterable-float":
                  1. Set previousEntry.texture.sampleType to "float".
              4. If any other property is unequal between entry and previousEntry:
                 1. Return null (which will cause the creation of the pipeline to fail).
           
          13.  Else
                  1. Append entry to groupDescs[group].

---
---

5. 让 groupLayouts 成为一个新的列表。
6. 对于从 0到 groupCount - 1 的每个 i 值，包括在边界在内：
    1. 让 groupDesc 成为 groupDescs[i]。
    2. 让 bindGroupLayout 成为调用 device.createBindGroupLayout()（groupDesc）的结果。
    3. 设置 bindGroupLayout.[[exclusivePipeline]] 为 pipeline。
    4. 将 bindGroupLayout 追加到 groupLayouts。
7. 让 desc 成为一个新的 GPUPipelineLayoutDescriptor。
8. 将 desc.bindGroupLayouts 设置为 groupLayouts。
9. 返回 device.createPipelineLayout()(desc)。

---

5. Let groupLayouts be a new list.
6. For each i from 0 to groupCount - 1, inclusive:
   1. Let groupDesc be groupDescs[i].
   2. Let bindGroupLayout be the result of calling device.createBindGroupLayout()(groupDesc).
   3. Set bindGroupLayout.[[exclusivePipeline]] to pipeline.
   4. Append bindGroupLayout to groupLayouts.
7. Let desc be a new GPUPipelineLayoutDescriptor.
8. Set desc.bindGroupLayouts to groupLayouts.
9. Return device.createPipelineLayout()(desc).

---
---

### 10.1.2.GPUProgrammableStage

一个 GPUProgrammableStage 描述了用户提供的 GPUShaderModule 中的入口点函数，该入口点函数控制流水线的一个可编程阶段。入口点名称遵循 WGSL 标识符比较中定义的规则。

```
dictionary GPUProgrammableStage {
    required GPUShaderModule module;
    required USVString entryPoint;
    record<USVString, GPUPipelineConstantValue> constants;
};

// 可以是 WGSL 中的 bool f32 i32 u32 类型，如果 f16 使能也可以。
// May represent WGSL’s bool, f32, i32, u32, and f16 if enabled.
typedef double GPUPipelineConstantValue; 

```

---

10.1.2. GPUProgrammableStage

A GPUProgrammableStage describes the entry point in the user-provided GPUShaderModule that controls one of the programmable stages of a pipeline. Entry point names follow the rules defined in WGSL identifier comparison.

```
dictionary GPUProgrammableStage {
    required GPUShaderModule module;
    required USVString entryPoint;
    record<USVString, GPUPipelineConstantValue> constants;
};

// May represent WGSL’s bool, f32, i32, u32, and f16 if enabled.
typedef double GPUPipelineConstantValue; 
```

---
---

GPUProgrammableStage 有以下成员：

**module，类型为： GPUShaderModule**        
GPUShaderModule 包含 相应代码将要执行的可编程阶段。 

**entryPoint，类型为: USVString**       
模块中的函数名称，表示着色器在该阶段将使用此入口点函数来执行其工作。

**constants，类型为: record\<USVString, GPUPipelineConstantValue\>。**      
指定着色器模块的模块中的管道可重写常量的值。

每个这样的管道可重写常量都由一个管道可重写常量的标识符字符串唯一标识（如果指定了常量的数字ID，则代表该常量的数字ID，否则就是常量的标识符名称）。在源代码maps 中的 WGSL名称（标识符）遵循 WGSL标识符对比章节中定义的规则。

每个 键-值对的键必须等于一个这样的常量的标识符字符串。当管道被执行时，该常量将具有指定的值。

值被指定为 GPUPipelineConstantValue ，它是一个 double 类型值。它们将被转换为管道可重写常量的 WGSL 类型（bool/i32/u32/f32/f16）。如果转换失败，将产生一个验证错误。

---

GPUProgrammableStage has the following members:

**module, of type GPUShaderModule**     
The GPUShaderModule containing the code that this programmable stage will execute.

**entryPoint, of type USVString**       
The name of the function in module that this stage will use to perform its work.

**constants, of type record\<USVString, GPUPipelineConstantValue\>**  
Specifies the values of pipeline-overridable constants in the shader module module.

Each such pipeline-overridable constant is uniquely identified by a single pipeline-overridable constant identifier string (representing the numeric ID of the constant, if one is specified, and otherwise the constant’s identifier name). WGSL names (identifiers) in source maps follow the rules defined in WGSL identifier comparison.

The key of each key-value pair must equal the identifier string of one such constant. When the pipeline is executed, that constant will have the specified value.

Values are specified as GPUPipelineConstantValue, which is a double. They are converted to WGSL type of the pipeline-overridable constant (bool/i32/u32/f32/f16). If conversion fails, a validation error is generated.

```
EXAMPLE 21

// 定义在 WGSL 中的可重写管道的常量。
// Pipeline-overridable constants defined in WGSL:

// 算数控制点
@id(0)      override has_point_light: bool = true;  // Algorithmic control.

// 数字控制点
@id(1200)   override specular_param: f32 = 2.3;     // Numeric control.

// 必须被重写
@id(1300)   override gain: f32;                     // Must be overridden.

            //  用 "width" 名称定义在 API 级别。
            override width: f32 = 0.0;              // Specifed at the API level
                                                    //   using the name "width".

            //  用 "depth" 名称定义在 API 级别。
            override depth: f32;                    // Specifed at the API level
                                                    //   using the name "depth".
                                                    //   Must be overridden.
            // 必须被重写
            // 如果不是在 API 级别设置。
            // 则依赖于另一个被重写的常量。
            override height = 2 * depth;            // The default value
                                                    // (if not set at the API level),
                                                    // depends on another
                                                    // overridable constant.

// 相应的JavaScript代码，只提供需要的重写（没有默认值）：
// Corresponding JavaScript code, providing only the overrides which are required (have no defaults):

{
    // ...
    constants: {
        1300: 2.0,  // "gain"
        depth: -1,  // "depth"
    }
}

// 相应的JavaScript代码，重写所有的常量：
Corresponding JavaScript code, overriding all constants:

{
    // ...
    constants: {
        0: false,   // "has_point_light"
        1200: 3.0,  // "specular_param"
        1300: 2.0,  // "gain"
        width: 20,  // "width"
        depth: -1,  // "depth"
        height: 15, // "height"
    }
}
```

---
---

**validating GPUProgrammableStage(stage, descriptor, layout) 函数**     

**参数：**      
* GPUShaderStage 类型的阶段
* GPUProgrammableStage 类型的描述符
* GPUPipelineLayout 类型的布局

如果满足以下所有条件，则返回true：
* descriptor.module 必须是一个有效的 GPUShaderModule。
* descriptor.module 必须包含一个入口点，用于着色器阶段的阶段，名为 descriptor.entryPoint。
* 对于每一个被 descriptor 静态使用的 binding ：
    * validating shader binding(binding, layout) 函数必须返回 true， 即验证shade绑定的函数结果必须返回true。
* 对于每一个被 descriptor 的纹理采样调用中一起静态使用的每个纹理和采样器：
    1. 让 texture 成为该调用中与采样纹理相对应的 GPUBindGroupLayoutEntry。
    2. 让 sampler 成为该调用中使用的采样器相对应的 GPUBindGroupLayoutEntry。
    3. 如果 sampler.type 是 "filtering"，那么 texture.sampleType 必须是 "float"。
* 对于 descriptor.constants 中的每一对 key→value：
    1. key 必须等于管道可重写常量的标识符字符串， 此字符串对应是在 shader 模块中定义的一些 管道可重写的常量，管道的 descriptor.module 是按照 WGSL标识符对照规则定义的。假设该常量的类型为 T 。
    2。 将 IDL值的值 转换为 WGSL 的类型 T 时，不得抛出 TypeError。
* 对于每一个被 descriptor 静态使用的管道可重写的常量标识符字符串 key：
    * 如果 key 所标识的管道可重写常量没有一个默认值，那么 descriptor.constants 必须包含 key。

如果一个管道创建错误，返回值为 false。

---

**validating GPUProgrammableStage(stage, descriptor, layout)**     

**Arguments:**      
* GPUShaderStage stage
* GPUProgrammableStage descriptor
* GPUPipelineLayout layout

Return true if all of the following conditions are met:
* descriptor.module must be a valid GPUShaderModule.
* descriptor.module must contain an entry point, for shader stage stage, named descriptor.entryPoint.
* For each binding that is statically used by descriptor:
    * validating shader binding(binding, layout) must return true.
* For each texture and sampler statically used together in texture sampling call in descriptor:
    1. Let texture be the GPUBindGroupLayoutEntry corresponding to the sampled texture in the call.
    2. Let sampler be the GPUBindGroupLayoutEntry corresponding to the used sampler in the call.
    3. If sampler.type is "filtering", then texture.sampleType must be "float".
* For each key → value in descriptor.constants:
    1. key must equal the pipeline-overridable constant identifier string of some pipeline-overridable constant defined in the shader module descriptor.module by the rules defined in WGSL identifier comparison. Let the type of that constant be T.
    2. Converting the IDL value value to WGSL type T must not throw a TypeError.
* For each pipeline-overridable constant identifier string key which is statically used by descriptor:
    * If the pipeline-overridable constant identified by key does not have a default value, descriptor.constants must contain key.

A return value of false corresponds to a pipeline-creation error.

---
---

**validating shader binding(binding, layout)**             
验证着色器绑定函数。    

**参数：**
* shader 绑定声明变量，从 shader 模块 映射出来的一个模块作用域范围内的变量声明。
* GPUPipelineLayout 布局

让 bindGroup 成为绑定组索引，而 bindIndex 则是绑定索引，其都是着色器绑定声明变量

---

**validating shader binding(binding, layout)**      

**Arguments:**  
* shader binding declaration variable, a module-scope variable declaration reflected from a shader module
* GPUPipelineLayout layout

Let bindGroup be the bind group index, and bindIndex be the binding index, of the shader binding declaration variable.

---
---

如果满足以下所有条件，则返回true：

* layout.[[bindGroupLayouts]][bindGroup] 包含一个 GPUBindGroupLayoutEntry条目，其 entry.binding == bindIndex。

* 如果入口点函数条目定义的绑定成员是：      
**buffer**        
    如果entry.buffer.type是：   
    **"uniform"**           
    变量被声明为 uniform 的地址空间。              
    **"storage"**       
    变量被声明为 storage 的地址空间 且其访问模式为 read_write。       
    **"read-only-storage"**     
    变量被声明为 storage 的地址空间，且访问模式为 read。

如果 entry.buffer.minBindingSize 不是0，那么它必须 且至少是着色器中相关缓冲区绑定变量的最小缓冲区绑定大小。

---

Return true if all of the following conditions are satisfied:
* layout.[[bindGroupLayouts]][bindGroup] contains a GPUBindGroupLayoutEntry entry whose entry.binding == bindIndex.
* If the defined binding member for entry is:       
**buffer**      
If entry.buffer.type is:        
    **"uniform"**       
    variable is declared with address space uniform.        
    **"storage"**       
    variable is declared with address space storage and access mode read_write.        
    **"read-only-storage"**     
    variable is declared with address space storage and access mode read.

If entry.buffer.minBindingSize is not 0, then it must be at least the minimum buffer binding size for the associated buffer binding variable in the shader.  

---
---

**sampler**         
如果 entry.sampler.type 是：        
    **"filtering" or "non-filtering"**           
    "过滤 "或 "非过滤"， 变量的类型为 sampler。     
    **"comparison"**        
    "比较"： 变量的类型为 sampler_comparison。

---

**sampler**         
If entry.sampler.type is:           
    **"filtering" or "non-filtering"**      
    variable has type sampler.      
    **"comparison"**        
    variable has type sampler_comparison.       

---
---

**texture**     

当且仅当 entry.texture.multisampled 为 true 时，变量的类型为 texture_multisampled_2d<T> 或 texture_depth_multisampled_2d<T>。

如果 entry.texture.sampleType 是：      
**"float", "unfilterable-float", "sint" or "uint"**     
变量有以下类型之一：        
  * texture_1d<T>     
  * texture_2d<T>     
  * texture_2d_array<T>       
  * texture_cube<T>       
  * texture_cube_array<T>     
  * texture_3d<T>     
  * texture_multisampled_2d<T>        

如果 entry.texture.sampleType 是：      
    **"float" or "unfilterable-float"**         
    则采样类型 T 为 f32。       
    **"sint"**      
    则采样类型 T 为 i32。      
    **"uint"**      
    则采样类型 T 为 u32。

---

**texture**     
If, and only if, entry.texture.multisampled is true, variable has type texture_multisampled_2d<T> or texture_depth_multisampled_2d<T>.      
If entry.texture.sampleType is:     
**"float", "unfilterable-float", "sint" or "uint"**     
variable has one of the types:      
  * texture_1d<T>     
  * texture_2d<T>     
  * texture_2d_array<T>       
  * texture_cube<T>       
  * texture_cube_array<T>     
  * texture_3d<T>     
  * texture_multisampled_2d<T>        

If entry.texture.sampleType is:         
    **"float" or "unfilterable-float"**         
    The sampled type T is f32.      
    **"sint"**      
    The sampled type T is i32.      
    **"uint"**      
    The sampled type T is u32.

---
---

**"depth"**     
变量具有以下类型之一：      
* texture_2d<T>
* texture_2d_array<T>
* texture_cube<T>
* texture_cube_array<T>
* texture_multisampled_2d<T>
* texture_depth_2d
* texture_depth_2d_array
* texture_depth_cube
* texture_depth_cube_array
* texture_depth_multisampled_2d

其中 采样类型 T 为f32。

---

**"depth"**     
variable has one of the types:      
* texture_2d<T>
* texture_2d_array<T>
* texture_cube<T>
* texture_cube_array<T>
* texture_multisampled_2d<T>
* texture_depth_2d
* texture_depth_2d_array
* texture_depth_cube
* texture_depth_cube_array
* texture_depth_multisampled_2d

where the sampled type T is f32.        

---
---

如果 entry.texture.viewDimension 是：
**"1d"**            
变量的类型是 texture_1d<T>.        
**"2d"**             
变量的类型是 texture_2d<T> or texture_multisampled_2d<T>.      
**"2d-array"**          
变量的类型是 texture_2d_array<T>.      
**"cube"**      
变量的类型是 texture_cube<T>.      
**"cube-array"**        
变量的类型是 texture_cube_array<T>.        
**"3d"**        
变量的类型是 texture_3d<T>.        

---

If entry.texture.viewDimension is:      
**"1d"**            
variable has type texture_1d<T>.        
**"2d"**                
variable has type texture_2d<T> or texture_multisampled_2d<T>.      
**"2d-array"**      
variable has type texture_2d_array<T>.      
**"cube"**      
variable has type texture_cube<T>.      
**"cube-array"**        
variable has type texture_cube_array<T>.        
**"3d"**        
variable has type texture_3d<T>.        

---
---

**storageTexture**      
如果 entry.storageTexture.viewDimension 是：        
**"1d"**        
变量的类型是 texture_storage_1d<T, A>.     
**"2d"**        
变量的类型是 texture_storage_2d<T, A>.     
**"2d-array"**      
变量的类型是 texture_storage_2d_array<T, A>.       
**"3d"**        
变量的类型是 texture_storage_3d<T, A>.      

如果 entry.storageTexture.access 是：       
**"write-only"**        
则访问模式 A 就是只写的。

最后，文本格式 T 等于 entry.storageTexture.format。

---

**storageTexture**      
If entry.storageTexture.viewDimension is:           
**"1d"**        
variable has type texture_storage_1d<T, A>.     
**"2d"**        
variable has type texture_storage_2d<T, A>.     
**"2d-array"**      
variable has type texture_storage_2d_array<T, A>.       
**"3d"**        
variable has type texture_storage_3d<T, A>.     

If entry.storageTexture.access is:      
**"write-only"**        
The access mode A is write.         

The texel format T equals entry.storageTexture.format.      

---
---

一个缓冲区绑定变量 var 的最小缓冲区绑定尺寸的计算方法如下：
1. 设 T 是变量 var 的存储类型。
2. 如果 T 是一个运行时分配大小的数组，或者包含一个运行时分配大小的数组，用 array<E, 1> 替换该 array<E>。        
    **注意：** 这可以确保总是有足够的内存去容纳一个元素，这允许数组索引被钳制在一个内存中的可访问数组的长度上。
3. 返回 SizeOf(T)。

**注意：** 强制执行这个下限操作，可以确保通过缓冲区变量的读写只能访问缓冲区边界区域内的内存位置。

如果在指定的着色器模块中，且指定入口点函数的着色器阶段接口中存在着 资源绑定、管道可重写常数、着色器阶段输入 或 着色器阶段输出，则这些资源绑定、或管道可重写常数、或着色器阶段输入，或着色器阶段输出 被认为是被 GPUProgrammableStage 静态使用的。

---

The minimum buffer binding size for a buffer binding variable var is computed as follows:
1. Let T be the store type of var.
2. If T is a runtime-sized array, or contains a runtime-sized array, replace that array<E> with array<E, 1>.        
**NOTE:** This ensures there’s always enough memory for one element, which allows array indices to be clamped to the length of the array resulting in an in-memory access.
3. Return SizeOf(T).

**NOTE:** Enforcing this lower bound ensures reads and writes via the buffer variable only access memory locations within the bound region of the buffer.

A resource binding, pipeline-overridable constant, shader stage input, or shader stage output is considered to be statically used by a GPUProgrammableStage if it is present in the interface of the shader stage of the specified entryPoint, in the specified shader module.

---
---

## 10.2. GPU计算管道 (GPUComputePipeline)

GPUComputePipeline 是一种控制计算着色器阶段的流水线，可以在 GPUComputePassEncoder 中使用。

根据给定的 GPUPipelineLayout，计算输入和输出都包含在绑定中。且其输出对应于缓冲器绑定的类型为 "storage" 或者"storageTexture" 的绑定的类型为 ”write-only“

计算管道的阶段：
1. 计算着色器

```
[Exposed=(Window, DedicatedWorker), SecureContext]
interface GPUComputePipeline {
};
GPUComputePipeline includes GPUObjectBase;
GPUComputePipeline includes GPUPipelineBase;
```

---

10.2. GPUComputePipeline

A GPUComputePipeline is a kind of pipeline that controls the compute shader stage, and can be used in GPUComputePassEncoder.

Compute inputs and outputs are all contained in the bindings, according to the given GPUPipelineLayout. The outputs correspond to buffer bindings with a type of "storage" and storageTexture bindings with a type of "write-only".

Stages of a compute pipeline:
1. Compute shader

```
[Exposed=(Window, DedicatedWorker), SecureContext]
interface GPUComputePipeline {
};
GPUComputePipeline includes GPUObjectBase;
GPUComputePipeline includes GPUPipelineBase;
```
---
---

### 10.2.1. 计算管道的创建
一个 GPUComputePipelineDescriptor 描述了一个计算管道。更多信息详见 § 23.2 计算。

```
dictionary GPUComputePipelineDescriptor : GPUPipelineDescriptorBase {
    required GPUProgrammableStage compute;
};
```

GPUComputePipelineDescriptor 有以下成员：       
**compute，类型为: GPUProgrammableStage**       
描述 管着的 计算着色器入口点函数。

---

10.2.1. Compute Pipeline Creation
A GPUComputePipelineDescriptor describes a compute pipeline. See § 23.2 Computing for additional details.

```
dictionary GPUComputePipelineDescriptor : GPUPipelineDescriptorBase {
    required GPUProgrammableStage compute;
};
```
GPUComputePipelineDescriptor has the following members:

**compute, of type GPUProgrammableStage**   
Describes the compute shader entry point of the pipeline.

---
---

**createComputePipeline(descriptor)**       

使用 立即创建管道的方式 创建一个 GPU计算管道(GPUComputePipeline)。

**调用于：** GPUDevice 实例自身

**参数：**

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| `descriptor` | `GPUComputePipelineDescriptor` | ✘ | ✘ | 创建 GPUComputePipeline 的描述。 |
 
**返回：** GPUComputePipeline

**上下文时间线步骤：**    
1. 让管道成为一个新的 GPUComputePipeline 对象。
2. 在GPUDevice 实例的 设备时间线上发起后续初始化步骤。
3. 返回 pipeline。

---
    
**createComputePipeline(descriptor)**       
Creates a GPUComputePipeline using immediate pipeline creation.

**Called on:** GPUDevice this.      

**Arguments:**

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| `descriptor` | `GPUComputePipelineDescriptor` | ✘ | ✘ | Description of the GPUComputePipeline to create. |

Returns: GPUComputePipeline

**Content timeline steps:**     
1. Let pipeline be a new GPUComputePipeline object.
2. Issue the initialization steps on the Device timeline of this.
3. Return pipeline.

---
---

**设备时间线初始化步骤：**      
1. 如果 descriptor.layout 的值是 "auto"，让 layout 成为管道的一个新默认布局，否则让 descriptor.layout 成为管道的一个新默认布局。
2. 如果以下任何条件不满足，产生一个验证错误，使管道无效，并停止。
    * 通过实例自身的this指针使用的 layout 必须是有效的。
    * 验证 GPUProgrammableStage(COMPUTE, descriptor.compute, layout) 必须成功。     
    * 让 workgroupStorageUsed 成为 descriptor.compute 在 "workgroup" 地址空间中静态使用的所有变量的，每种类型 T 的 roundUp(16, SizeOf(T))之和。     
    workgroupStorageUsed 必须小于等于 device.limit.maxComputeWorkgroupStorageSize。     
    >> **ISSUE 14** 这是否需要考虑到填充？[Issue #gpuweb/gpuweb#3485] 。

    * descriptor.compute 必须使用小于等于每个工作组的 device.limit.maxComputeInvocationsPerWorkgroup 。
    * descriptor.compute 的 workgroup_size 属性的每个组件必须 小于等于 [device.limit.maxComputeWorkgroupSizeX, device.limit.maxComputeWorkgroupSizeY, device.limit.maxComputeWorkgroupSizeZ]中的相对应组件。
3. 设置pipeline.[[layout]] 为 layout。

---

**Device timeline initialization steps:**   
1. Let layout be a new default pipeline layout for pipeline if descriptor.layout is "auto", and descriptor.layout otherwise.
2. If any of the following conditions are unsatisfied generate a validation error, make pipeline invalid, and stop.
    * layout must be valid to use with this.
    * validating GPUProgrammableStage(COMPUTE, descriptor.compute, layout) must succeed.
    * Let workgroupStorageUsed be the sum of roundUp(16, SizeOf(T)) over each type T of all variables with address space "workgroup" statically used by descriptor.compute.     
    workgroupStorageUsed must be ≤ device.limits.maxComputeWorkgroupStorageSize.

    **ISSUE 14** Does this need to account for padding? [Issue #gpuweb/gpuweb#3485]

    * descriptor.compute must use ≤ device.limits.maxComputeInvocationsPerWorkgroup per workgroup.

    * Each component of descriptor.compute's workgroup_size attribute must be ≤ the corresponding component in [device.limits.maxComputeWorkgroupSizeX, device.limits.maxComputeWorkgroupSizeY, device.limits.maxComputeWorkgroupSizeZ].

3. Set pipeline.[[layout]] to layout.

---
---

**createComputePipelineAsync(descriptor)**      

使用异步创建管道的方式创建一个 GPUComputePipeline。当创建的管道可以使用时，返回的 Promise 对象就会成功，没有额外的延迟。

如果管道创建失败，返回的 Promise 对象将以 GPUPipelineError 拒绝。

**注意：** 在可能的情况下，最好使用该方法，因为它可以防止阻塞在管道编译工作中队列时间线。

---

**createComputePipelineAsync(descriptor)**      

Creates a GPUComputePipeline using async pipeline creation. The returned Promise resolves when the created pipeline is ready to be used without additional delay.

If pipeline creation fails, the returned Promise rejects with an GPUPipelineError.

**NOTE:** Use of this method is preferred whenever possible, as it prevents blocking the queue timeline work on pipeline compilation.

---
---

**调用于：** GPUDevice 实例自身。

**参数：**

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| `descriptor` | `GPUComputePipelineDescriptor` | ✘ | ✘ | 创建 GPUComputePipeline 的描述。 |

**返回：** Promise<GPUComputePipeline>。

**上下文时间线步骤：**    
1. 让 contentTimeline 成为当前的上下文时间线。
2. 让 promise 是一个新的 Promise 对象。
3. 在 GPUDevice 实例的 设备时间线上发起后续的初始化步骤。
4. 返回 promise。

---
**Called on:** GPUDevice this.

**Arguments:**

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| `descriptor` | `GPUComputePipelineDescriptor` | ✘ | ✘ | Description of the GPUComputePipeline to create. |

**Returns:** Promise<GPUComputePipeline>

**Content timeline steps:**
1. Let contentTimeline be the current Content timeline.
2. Let promise be a new promise.
3. Issue the initialization steps on the Device timeline of this.
4. Return promise.

---
---

**设备时间线初始化步骤：**  
1. 让 pipeline 成为一个新的 GPUComputePipeline，就像用描述符调用 this.createComputePipeline()那样创建；
2. 当 pipeline 准备好被使用或已经失效时，在 contentTimeline 上发出后续步骤。

**上下文时间线的步骤：**    
1. 如果 pipeline ...

    **有效的 valid**        
    让 Promie 是成功的，其值为 pipeline。

    **因内部错误而无效**        
    让 Promie 是失败的，其值是 GPUPipelineError，GPUPipelineError 的原因是 "internal"。

    **由于验证错误而无效**      
    让 Promie 是失败的，其值是 GPUPipelineError，GPUPipelineError 的原因是 "validation"。


---

**Device timeline initialization steps:**   
1. Let pipeline be a new GPUComputePipeline created as if this.createComputePipeline() was called with descriptor;
2. When pipeline is ready to be used or has been made invalid, issue the subsequent steps on contentTimeline.

**Content timeline steps:**     
1. If pipeline...       

    **valid**       
    Resolve promise with pipeline.

    **invalid due to an internal error**        
    Reject promise with a GPUPipelineError with reason "internal".

    **invalid due to an validation error**      
    Reject promise with a GPUPipelineError with reason "validation".

```
// 举例 22
EXAMPLE 22 

// 创建一个简单的 GPUComputePipeline：
// Creating a simple GPUComputePipeline:

const computePipeline = gpuDevice.createComputePipeline({
    layout: pipelineLayout,
    compute: {
        module: computeShaderModule,
        entryPoint: 'computeMain',
    }
});
```

---
---

## 10.3. GPU渲染管道 (GPURenderPipeline)

GPURenderPipeline 是一种 控制顶点和片段着色器阶段的管道，可以在 GPURenderPassEncoder 像在 GPURenderBundleEncoder 中那样使用。

**渲染管线的输入有以下几种：**      
* 绑定(bindings)，根据给定的 GPUPipelineLayout
* 顶点和索引缓冲器，由 GPUVertexState 描述
* 颜色附件，由 GPUColorTargetState 描述
* 可选地，深度/模板附件，由 GPUDepthStencilState 描述

---

10.3. GPURenderPipeline

A GPURenderPipeline is a kind of pipeline that controls the vertex and fragment shader stages, and can be used in GPURenderPassEncoder as well as GPURenderBundleEncoder.

Render pipeline inputs are:
* bindings, according to the given GPUPipelineLayout
* vertex and index buffers, described by GPUVertexState
* the color attachments, described by GPUColorTargetState
* optionally, the depth-stencil attachment, described by GPUDepthStencilState

---
---

渲染管道的输出是：
* 类型为 "storage" 的缓冲区绑定。
* storageTexture绑定，访问权限为 "只写"。
* 颜色附件，由 GPUColorTargetState 描述。
* 可选地，深度/模板附件，由 GPUDepthStencilState 描述。

渲染管线由以下渲染阶段组成：
1. 顶点获取，由 GPUVertexState.buffers 控制。
2. 顶点着色器，由 GPUVertexState 控制。
3. 基元组装，由 GPUPrimitiveState 控制。
4. 栅格化，由 GPUPrimitiveState、GPUDepthStencilState和GPUMultisampleState 控制。
5. 片段着色器，由 GPUFragmentState 控制。
6. 模板测试和操作，由 GPUDepthStencilState 控制。
7. 深度测试和写入，由 GPUDepthStencilState 控制。
8. 输出合并，由 GPUFragmentState.target 控制。

---
Render pipeline outputs are:
* buffer bindings with a type of "storage"
* storageTexture bindings with a access of "write-only"
* the color attachments, described by GPUColorTargetState
* optionally, depth-stencil attachment, described by GPUDepthStencilState

A render pipeline is comprised of the following render stages:
1. Vertex fetch, controlled by GPUVertexState.buffers
2. Vertex shader, controlled by GPUVertexState
3. Primitive assembly, controlled by GPUPrimitiveState
4. Rasterization, controlled by GPUPrimitiveState, GPUDepthStencilState, and GPUMultisampleState
5. Fragment shader, controlled by GPUFragmentState
6. Stencil test and operation, controlled by GPUDepthStencilState
7. Depth test and write, controlled by GPUDepthStencilState
8. Output merging, controlled by GPUFragmentState.targets

---
---

```
[Exposed=(Window, DedicatedWorker), SecureContext]
interface GPURenderPipeline {
};
GPURenderPipeline includes GPUObjectBase;
GPURenderPipeline includes GPUPipelineBase;
```

GPURenderPipeline 有以下内部插槽：
**[[descriptor]], 类型为：GPURenderPipelineDescriptor**   
描述此管道的 GPURenderPipelineDescriptor。

所有 GPURenderPipelineDescriptor 的可选字段都被定义。

**[[writesDepth]], 类型为： boolean**       
如果该管道写到深度/模板附件的深度分量，则为真。

**[[writesStencil]], 类型为： boolean**       
如果该管道写到深度/模板附件的分量，则为真。

---

```
[Exposed=(Window, DedicatedWorker), SecureContext]
interface GPURenderPipeline {
};
GPURenderPipeline includes GPUObjectBase;
GPURenderPipeline includes GPUPipelineBase;
```

GPURenderPipeline has the following internal slots:

**[[descriptor]], of type GPURenderPipelineDescriptor**     
The GPURenderPipelineDescriptor describing this pipeline.

All optional fields of GPURenderPipelineDescriptor are defined.

**[[writesDepth]], of type boolean**      
True if the pipeline writes to the depth component of the depth/stencil attachment

**[[writesStencil]], of type boolean**        
True if the pipeline writes to the stencil component of the depth/stencil attachment

---
---

### 10.3.1. 渲染管线的创建

一个 GPURenderPipelineDescriptor 通过配置每个渲染阶段来描述一个渲染管道。更多细节请参见 § 23.3 渲染。

```
dictionary GPURenderPipelineDescriptor : GPUPipelineDescriptorBase {
    需要GPUVertexState vertex；
    GPUPrimitiveState primitive = {}；
    GPUDepthStencilState depthStencil；
    GPUMultisampleState multisample = {}；
    GPUFragmentState fragment；
};
```

GPURenderPipelineDescriptor有以下成员：     

**vertex，类型为： GPUVertexState**      
描述管道的顶点着色器入口点及其输入缓冲区的布局。

**primitive，类型为： GPUPrimitiveState，默认为{}。**        
描述管道的基元相关属性。

**depthStencil，类型为： GPUDepthStencilState**      
描述可选的深度/模板结构属性，包括测试、操作和偏差。

**multisample，类型为： GPUMultisampleState，默认为{}。**        
描述管道的多重采样属性。

**fragment, 类型为： GPUFragmentState**        
描述管道的片段着色器入口点及其输出颜色。如果不提供，则启用 §23.3.8 所描述的无颜色输出模式。

---

10.3.1. Render Pipeline Creation

A GPURenderPipelineDescriptor describes a render pipeline by configuring each of the render stages. See § 23.3 Rendering for additional details.

```
dictionary GPURenderPipelineDescriptor : GPUPipelineDescriptorBase {
    required GPUVertexState vertex;
    GPUPrimitiveState primitive = {};
    GPUDepthStencilState depthStencil;
    GPUMultisampleState multisample = {};
    GPUFragmentState fragment;
};
```

GPURenderPipelineDescriptor has the following members:      
**vertex, of type GPUVertexState**      
Describes the vertex shader entry point of the pipeline and its input buffer layouts.

**primitive, of type GPUPrimitiveState, defaulting to {}**      
Describes the primitive-related properties of the pipeline.

**depthStencil, of type GPUDepthStencilState**      
Describes the optional depth-stencil properties, including the testing, operations, and bias.

**multisample, of type GPUMultisampleState, defaulting to {}**      
Describes the multi-sampling properties of the pipeline.

**fragment, of type GPUFragmentState**      
Describes the fragment shader entry point of the pipeline and its output colors. If not provided, the § 23.3.8 No Color Output mode is enabled.

---
---

createRenderPipeline(descriptor)

使用立即创建管道的方式创建一个 GPURenderPipeline。

**调用于：** GPUDevice 实例自身。

**参数：**

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| `descriptor` | `GPURenderPipelineDescriptor` | ✘ | ✘ | 要创建的 GPURenderPipeline 的描述。 |
 
**返回：** GPURenderPipeline

**内容时间线步骤：**    

1. 如果提供了 descriptor.fragment：
    1. 对于 descriptor.fragment.target 的每个非空的colorState：
        1. ?用 this.[[device]] 验证 colorState.format 的纹理的纹理格式要求特征。
2. 如果提供了 descriptor.depthStencil：
    1. ?用 this.[[device]] 验证 descriptor.depthStencil.format 的纹理格式要求的特征。
3. 让 pipeline 成为一个新的 GPURenderPipeline 对象。
4. 在 this.[[device]] 的设备时间线上发起初始化步骤。
5. 返回 pipeline。

---

**createRenderPipeline(descriptor)**      

Creates a GPURenderPipeline using immediate pipeline creation.

**Called on:** GPUDevice this.

**Arguments:**

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| `descriptor` | `GPURenderPipelineDescriptor` | ✘ | ✘ | Description of the GPURenderPipeline to create. |

**Returns:** GPURenderPipeline

**Content timeline steps:**
1. If descriptor.fragment is provided:
    1. For each non-null colorState of descriptor.fragment.targets:
        1. ? Validate texture format required features of colorState.format with this.[[device]].
2. If descriptor.depthStencil is provided:
    1. ? Validate texture format required features of descriptor.depthStencil.format with this.[[device]].
3. Let pipeline be a new GPURenderPipeline object.
4. Issue the initialization steps on the Device timeline of this.
5. Return pipeline.

---
---

**设备时间线初始化步骤：**      
1. 如果 descriptor.layout 是 "auto"，让 layout 成为管道的一个新默认布局，否则让layout 为为 descriptor.layout 所描述的布局。
2. 如果以下任何一个条件不满足：产生一个验证错误，使管道无效，并停止。
    * this.device上的布局是有效的。
    * 验证 GPURenderPipelineDescriptor(descriptor, layout, this) 函数是成功。
3. 将 pipeline.[[descriptor]] 设置为 descriptor。
4. 将 pipeline.[[writesDepth]] 设置为 false。
5. 设置pipeline.[[writesStencil]] 设置为 false。
6. 让 depthStencil 成为 descriptor.depthStencil。
7. 如果 depthStencil 不是空的：
    1. 设置 pipeline.[[writesDepth]] 为 depthStencil.depthWriteEnabled。
    2. 如果 depthStencil.stencilWriteMask 不是 0：
        1. 让 stencilFront 成为 depthStencil.stencilFront。
        2. 让 stencilBack 成为 depthStencil.stencilBack。
        3. 让 cullMode 成为 descriptor.primitive.cullMode。
        4. 如果 cullMode 不是 "front"，并且 stencilFront.passOp、stencilFront.depthFailOp 或 stencilFront.failOp 中的任何一个不是 "keep"：
            1. 将 pipeline.[[writesStencil]] 设置为 true。
        5. 如果 cullMode 不是 "back"，并且 stencilBack.passOp、stencilBack.depthFailOp 或 stencilBack.failOp 中的任何一个不是 "keep"：
            1. 将 pipeline.[writesStencil]] 设置为 true。
8. 将 pipeline.[[layout]] 设置为 layout。

>> ISSUE 15 渲染状态是否需要描述。

---

Device timeline initialization steps:

1. Let layout be a new default pipeline layout for pipeline if descriptor.layout is "auto", and descriptor.layout otherwise.
2. If any of the following conditions are unsatisfied: generate a validation error, make pipeline invalid, and stop.
    * layout is valid to use with this.
    * validating GPURenderPipelineDescriptor(descriptor, layout, this) succeeds.
3. Set pipeline.[[descriptor]] to descriptor.
4. Set pipeline.[[writesDepth]] to false.
5. Set pipeline.[[writesStencil]] to false.
6. Let depthStencil be descriptor.depthStencil.
7. If depthStencil is not null:
    1. Set pipeline.[[writesDepth]] to depthStencil.depthWriteEnabled.
    2. If depthStencil.stencilWriteMask is not 0:
        1. Let stencilFront be depthStencil.stencilFront.
        2. Let stencilBack be depthStencil.stencilBack.
        3. Let cullMode be descriptor.primitive.cullMode.
        4. If cullMode is not "front", and any of stencilFront.passOp, stencilFront.depthFailOp, or stencilFront.failOp is not "keep":
            1. Set pipeline.[[writesStencil]] to true.
        5. If cullMode is not "back", and any of stencilBack.passOp, stencilBack.depthFailOp, or stencilBack.failOp is not "keep":
            1. Set pipeline.[[writesStencil]] to true.
8. Set pipeline.[[layout]] to layout.

>> ISSUE 15 need description of the render states.

 ---
 ---

**createRenderPipelineAsync(descriptor)**       

使用异步创建管道的方式创建一个 GPURenderPipeline。创建的管道可以使用时，返回的 Promise 对象就会成功，没有额外的延迟。

如果管道创建失败，返回的 Promise 将以 GPUPipelineError 为值设置为拒绝状态。

**注意：** 在可能的情况下，最好使用该方法，因为它可以防止阻塞在管道编译工作中队列时间线。

**调用于：** GPUDevice 实例自身。

**参数：**

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| `descriptor` | `GPURenderPipelineDescriptor` | ✘ | ✘ | 要创建的GPURenderPipeline的描述。 |
 
**返回：** Promise<GPURenderPipeline>。

---

**createRenderPipelineAsync(descriptor)**     

Creates a GPURenderPipeline using async pipeline creation. The returned Promise resolves when the created pipeline is ready to be used without additional delay.

If pipeline creation fails, the returned Promise rejects with an GPUPipelineError.

**NOTE:** Use of this method is preferred whenever possible, as it prevents blocking the queue timeline work on pipeline compilation.

**Called on:** GPUDevice this.

**Arguments:**

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| `descriptor` | `GPURenderPipelineDescriptor` | ✘ | ✘ | Description of the GPURenderPipeline to create. |

**Returns:** Promise<GPURenderPipeline>

---
---

上下文时间线步骤：
1. 让 contentTimeline 成为当前的上下文时间线。
2. 让 promise 是一个新的 Promise。
3. 在this.device的设备时间线上发起初始化步骤。
4. 返回 promise。

---

Content timeline steps:
1. Let contentTimeline be the current Content timeline.
2. Let promise be a new promise.
3. Issue the initialization steps on the Device timeline of this.
4. Return promise.

---
---

设备时间线初始化步骤：
1. 让 pipeline 成为一个新的 GPURenderPipeline，就像使用描述符为参数来调用 this.createRenderPipeline() 一样；
2. 当 pipeline 准备好被使用或已经失效时，在 contentTimeline 上发起后续步骤。

---

Device timeline initialization steps:
1. Let pipeline be a new GPURenderPipeline created as if this.createRenderPipeline() was called with descriptor;
2. When pipeline is ready to be used or has been made invalid, issue the subsequent steps on contentTimeline.

---
---

上下文时间线的步骤：
1. 如果pipeline是...        
**有效**        
promise 状态被设置为成功，其值为： pipeline。

**由于内部错误而无效**      
promise 状态被设置为失败，其值为： 原因为 “internal” 的 GPUPipelineError

**由于验证错误而无效**      
promise 状态被设置为失败，其值为： 原因为 validation 的 GPUPipelineError

---

Content timeline steps:
1. If pipeline is...        
**valid**           
Resolve promise with pipeline.      
**invalid due to an internal error**        
Reject promise with a GPUPipelineError with reason "internal".      
**invalid due to an validation error**      
Reject promise with a GPUPipelineError with reason "validation".

---
---

**验证函数 GPURenderPipelineDescriptor(descriptor, layout, device)。**      

**参数：**      
* GPURenderPipelineDescriptor 描述符
* GPUPipelineLayout 布局
* GPUDevice 设备

如果满足以下所有条件，则返回true：
* 验证 GPUProgrammableStage(VERTEX, descriptor.vertex, layout) 返回成功。
* 验证 GPUVertexState(device, descriptor.vertex, descriptor.vertex) 返回成功。
* 如果提供了 descriptor.fragment：
    * 验证 GPUProgrammableStage(FRAGMENT, descriptor.fragment, layout) 返回成功。
    * 验证 GPUFragmentState(device, descriptor.fragment) 返回成功。
    * 如果 "sample_mask" 内建程序 是 descriptor.fragment 的一个着色器阶段输出：
        * descriptor.multisample.alphaToCoverageEnabled 设置为 false。
* 验证 GPUPrimitiveState(descriptor.primitive, device) 返回成功。
* 如果提供了 descriptor.depthStencil：
    * 验证 GPUDepthStencilState(descriptor.depthStencil) 返回成功。
* 验证 GPUMultisampleState(descriptor.multisample) 返回成功。
* 验证 inter-stage interfaces(device, descriptor) 返回true。

---

**validating GPURenderPipelineDescriptor(descriptor, layout, device)**      

**Arguments:**
* GPURenderPipelineDescriptor descriptor
* GPUPipelineLayout layout
* GPUDevice device

Return true if all of the following conditions are satisfied:
* validating GPUProgrammableStage(VERTEX, descriptor.vertex, layout) succeeds.
* validating GPUVertexState(device, descriptor.vertex, descriptor.vertex) succeeds.
* If descriptor.fragment is provided:
    * validating GPUProgrammableStage(FRAGMENT, descriptor.fragment, layout) succeeds.
    * validating GPUFragmentState(device, descriptor.fragment) succeeds.
    * If the "sample_mask" builtin is a shader stage output of descriptor.fragment:
        * descriptor.multisample.alphaToCoverageEnabled is false.
* validating GPUPrimitiveState(descriptor.primitive, device) succeeds.
* If descriptor.depthStencil is provided:
    * validating GPUDepthStencilState(descriptor.depthStencil) succeeds.
* validating GPUMultisampleState(descriptor.multisample) succeeds.
* validating inter-stage interfaces(device, descriptor) returns true.

---
---

**验证 inter-stage interfaces(device, descriptor)**

**参数：**
* GPUDevice 设备
* GPURenderPipelineDescriptor 描述符

**返回：** 布尔值

1. 让 maxVertexShaderOutputComponents 成为 device.limit.maxInterStageShaderComponents。
    1. 如果 descriptor.primitive.topology 是 "point-list"：
        1. 将 maxVertexShaderOutputComponents 减去 1。
2. 如果未满足以下任何要求，则返回 false：
    * 在 descriptor.vertex 的所有用户定义的输出中，不得有超过 maxVertexShaderOutputComponents 的标量分量。(例如，f32 的输出 使用 1个 分量，而 vec3<f32> 的输出使用 3个 分量)。
    * descriptor.vertex 的每个用户定义的输出的位置必须小于 device.limit.maxInterStageShaderVariables。

---

**validating inter-stage interfaces(device, descriptor)**

**Arguments:**
* GPUDevice device
* GPURenderPipelineDescriptor descriptor

**Returns:** boolean
1. Let maxVertexShaderOutputComponents be device.limits.maxInterStageShaderComponents.
    1. If descriptor.primitive.topology is "point-list":
        1. Decrement maxVertexShaderOutputComponents by 1.
2. Return false if any of the following requirements are unmet:
    * There must be no more than maxVertexShaderOutputComponents scalar components across all user-defined outputs for descriptor.vertex. (For example, a f32 output uses 1 component, and a vec3<f32> output uses 3 components.)
    * The location of each user-defined output of descriptor.vertex must be < device.limits.maxInterStageShaderVariables.

---
---

3. 如果提供了 descriptor.fragment：
    1. 让 maxFragmentShaderInputComponents 成为 device.limit.maxInterStageShaderComponents。
        1. 如果 front_facing 内置程序 是 descriptor.fragment 的一个输入：
            1. 将 maxFragmentShaderInputComponents 减少 1。
        2. 如果 sample_index 内置程序 是 descriptor.fragment 的一个输入：
            1. 将 maxFragmentShaderInputComponents 减少 1。
        3. 如果 sample_mask 内建程序 是 descriptor.fragment 的一个输入：
            1. 将 maxFragmentShaderInputComponents 减少 1。
    2. 如果未能满足以下任何要求，则返回 false：
        * 在 descriptor.fragment 的所有用户定义的输入中，不得有超过 maxFragmentShaderInputComponents 的标量分量。
        * 对于 descriptor.fragment 的每个用户定义的输入，必须有一个针对地址、类型和输入插值的 descriptor.vertex 的用户自定义的输出。        
        **注意：** 在仅有顶点的管道可以在顶点阶段有用户自定义的输出；其值将被丢弃。
    3. 断言 descriptor.fragment 的每个用户定义的输入的位置小于 device.limit.maxInterStageShaderVariables（此值由上述规则产生）。
4. 返回 true。

---
1. If descriptor.fragment is provided:
    1. Let maxFragmentShaderInputComponents be device.limits.maxInterStageShaderComponents.
        1. If the front_facing builtin is an input of descriptor.fragment:
            1. Decrement maxFragmentShaderInputComponents by 1.
        2. If the sample_index builtin is an input of descriptor.fragment:
           1. Decrement maxFragmentShaderInputComponents by 1.
        3. If the sample_mask builtin is an input of descriptor.fragment:
            1. Decrement maxFragmentShaderInputComponents by 1.
    2. Return false if any of the following requirements are unmet:
        * There must be no more than maxFragmentShaderInputComponents scalar components across all user-defined inputs for descriptor.fragment.
        * For each user-defined input of descriptor.fragment there must be a user-defined output of descriptor.vertex that location, type, and interpolation of the input.
        **NOTE:** Vertex-only pipelines can have user-defined outputs in the vertex stage; their values will be discarded.
    3. Assert that the location of each user-defined input of descriptor.fragment is less than device.limits.maxInterStageShaderVariables (resulting from the above rules).
2. Return true.


```
// 例子 23
// EXAMPLE 23 

// 创建一个简单的 GPURenderPipeline：
// Creating a simple GPURenderPipeline:

const renderPipeline = gpuDevice.createRenderPipeline({
    layout: pipelineLayout,
    vertex: {
        module: shaderModule,
        entryPoint: 'vertexMain'
    },
    fragment: {
        module: shaderModule,
        entryPoint: 'fragmentMain',
        targets: [{
            format: 'bgra8unorm',
        }],
    }
});

```

---
---

### 10.3.2. 原生状态

```
dictionary GPUPrimitiveState {
    GPUPrimitiveTopology topology = "triangle-list";
    GPUIndexFormat stripIndexFormat;
    GPUFrontFace frontFace = "ccw";
    GPUCullMode cullMode = "none";

    // 需要 "深度-裁剪-控制" 功能。
    // Requires "depth-clip-control" feature.
    boolean unclippedDepth = false;
};
```

---

10.3.2. Primitive State

```
dictionary GPUPrimitiveState {
    GPUPrimitiveTopology topology = "triangle-list";
    GPUIndexFormat stripIndexFormat;
    GPUFrontFace frontFace = "ccw";
    GPUCullMode cullMode = "none";

    // Requires "depth-clip-control" feature.
    boolean unclippedDepth = false;
};
```

---
---

GPUPrimitiveState 有以下成员，它们描述了 GPURenderPipeline 如何从其顶点输入构建和栅格化基元：

**topoloty, 拓扑结构，类型为： GPUPrimitiveTopology ，默认为 "triangle-list" [三角形列表]**     
表示从顶点输入构建的基元的类型。

**stripIndexFormat，类型为： GPUIndexFormat**       
对于具有条状拓扑结构的的管道（"line-strip "或 "triangle-strip"），这决定了索引缓冲区格式和基元重启值（"uint16"/0xFFFF 或 "uint32"/0xFFFFFFFF）。在具有 非条状拓扑结构的管道上不允许这样做。

**注意：** 有些具体的实现需要知道基元重启值来编译管道状态对象。

要使用带状拓扑结构的管道去调用索引绘制函数 (drawIndexed()或drawIndexedIndirect()），则必须设置这个值，而且必须与绘制调用使用的索引缓冲区格式相匹配（在setIndexBuffer()中设置）。

更多细节参见 § 23.3.3 原生封装。

---

GPUPrimitiveState has the following members, which describe how a GPURenderPipeline constructs and rasterizes primitives from its vertex inputs:

**topology, of type GPUPrimitiveTopology, defaulting to "triangle-list"**       
The type of primitive to be constructed from the vertex inputs.

**stripIndexFormat, of type GPUIndexFormat**        
For pipelines with strip topologies ("line-strip" or "triangle-strip"), this determines the index buffer format and primitive restart value ("uint16"/0xFFFF or "uint32"/0xFFFFFFFF). It is not allowed on pipelines with non-strip topologies.

NOTE: Some implementations require knowledge of the primitive restart value to compile pipeline state objects.

To use a strip-topology pipeline with an indexed draw call (drawIndexed() or drawIndexedIndirect()), this must be set, and it must match the index buffer format used with the draw call (set in setIndexBuffer()).

See § 23.3.3 Primitive Assembly for additional details.

---
---

frontFace，类型为： GPUFrontFace，默认为 "ccw"
定义哪些多边形被认为是朝前的。

cullMode，类型为： GPUCullMode，默认为 "none"
定义哪个多边形方向将被剔除，如果有的话。

unclippedDepth，类型为： boolean，默认为 false。
如果为真，表示深度剪裁被禁用。

需要启用 "depth-clip-control" 功能。

**验证 GPUPrimitiveState(descriptor, device)**   
**参数：**  
* GPUPrimitiveState 描述符
* GPUDevice 设备

如果满足以下所有条件，返回true：
* 如果 descriptor.topology 不是 "line-strip" 或 "triangle-strip"：
    * 则不能提供 descriptor.stripIndexFormat。
* 如果 descriptor.unclippedDepth 为真：
    * 设备必须启用 "depth-clip-control" 功能。

---

**frontFace, of type GPUFrontFace, defaulting to "ccw"**        
Defines which polygons are considered front-facing.

**cullMode, of type GPUCullMode, defaulting to "none"**     
Defines which polygon orientation will be culled, if any.

**unclippedDepth, of type boolean, defaulting to false**        
If true, indicates that depth clipping is disabled.

Requires the "depth-clip-control" feature to be enabled.

**validating GPUPrimitiveState(descriptor, device)**        
**Arguments:**           
* GPUPrimitiveState descriptor
* GPUDevice device

Return true if all of the following conditions are satisfied:
* If descriptor.topology is not "line-strip" or "triangle-strip":
    * descriptor.stripIndexFormat must not be provided.
* If descriptor.unclippedDepth is true:
    * "depth-clip-control" must be enabled for device.

---
---

```
enum GPUPrimitiveTopology {
    "point-list",
    "line-list",
    "line-strip",
    "triangle-list",
    "triangle-strip"
};
```

GPUPrimitiveTopology 定义了使用 GPURenderPipeline 进行的绘图调用时将使用的原始类型。更多细节请参见 § 23.3.5 栅格化：

**"point-list"**    
每个顶点数据都定义了一个点基元。        
**"line-list"**     
连续的两个顶点对都定义了一个线段基元。          
**"line-strip"**    
第一个顶点之后的每个顶点都和它前一个顶点之间定义了一个线段基元。            
**"triangle-list"**         
每连续的三个顶点的三联体都定义了一个三角形基元。            
**"triangle-strip"**        
前两个顶点之后的每个顶点都和它之前两个顶点之间定义了一个三角形基元。        

---

```
enum GPUPrimitiveTopology {
    "point-list",
    "line-list",
    "line-strip",
    "triangle-list",
    "triangle-strip"
};
```

GPUPrimitiveTopology defines the primitive type draw calls made with a GPURenderPipeline will use. See § 23.3.5 Rasterization for additional details:

**"point-list"**    
Each vertex defines a point primitive.

**"line-list"**     
Each consecutive pair of two vertices defines a line primitive.

**"line-strip"**        
Each vertex after the first defines a line primitive between it and the previous vertex.

**"triangle-list"**     
Each consecutive triplet of three vertices defines a triangle primitive.

**"triangle-strip"**        
Each vertex after the first two defines a triangle primitive between it and the previous two vertices.

---
---

```
enum GPUFrontFace {
    "ccw",
    "cw"
};
```

GPUFrontFace 定义了哪些多边形会被 GPURenderPipeline 认为是正面的。更多细节详见 § 23.3.5.4 多边形光栅化：

**"ccw"**       
顶点的帧缓冲区坐标如果是以 逆时针 的顺序给出的多边形被认为是朝前的，正面的。

**"cw"**    
顶点的帧缓冲区坐标如果是以 顺时针 的顺序给出的多边形被认为是朝前的，正面的。

---

```
enum GPUFrontFace {
    "ccw",
    "cw"
};
```

GPUFrontFace defines which polygons are considered front-facing by a GPURenderPipeline. See § 23.3.5.4 Polygon Rasterization for additional details:

**"ccw"**       
Polygons with vertices whose framebuffer coordinates are given in counter-clockwise order are considered front-facing.

**"cw"**        
Polygons with vertices whose framebuffer coordinates are given in clockwise order are considered front-facing.

---
---

```
enum GPUCullMode {
    "none",
    "front",
    "back"
};
```

GPUCullMode 定义了 GPURenderPipeline 的绘制调用模式会将哪些多边形进行剔除。更多细节详见 § 23.3.5.4 多边形光栅化：

**"none"**      
没有多边形被丢弃。      
**"front"**     
朝前的多边形被丢弃。            
"back"      
背面的多边形被丢弃。        

**注意：** GPUFrontFace 和 GPUCullMode 对 "point-list"、"line-list" 或 "line-strip" 拓扑结构没有影响。

---

```
enum GPUCullMode {
    "none",
    "front",
    "back"
};
```

GPUPrimitiveTopology defines which polygons will be culled by draw calls made with a GPURenderPipeline. See § 23.3.5.4 Polygon Rasterization for additional details:

**"none"**      
No polygons are discarded.

**"front"**     
Front-facing polygons are discarded.

**"back"**      
Back-facing polygons are discarded.

**NOTE:** GPUFrontFace and GPUCullMode have no effect on "point-list", "line-list", or "line-strip" topologies.

---
---

### 10.3.3. 多采样状态

```
dictionary GPUMultisampleState {
    GPUSize32 count = 1;
    GPUSampleMask mask = 0xFFFFFFFF;
    boolean alphaToCoverageEnabled = false;
};
```

GPUMultisampleState 有以下成员，它们描述了 GPURenderPipeline 如何与一个渲染通道的多采样附件进行交互。

**count，类型为： GPUSize32，默认为 1**     
每个像素的采样数量。这个 GPURenderPipeline 将只与具有匹配的样本数的附件纹理（ colorAttachments 和 depthStencilAttachment ）相兼容。

**mask，类型为： GPUSampleMask，默认为 0xFFFFFFFF**     
决定了哪些样本被写入的掩码。

**alphaToCoverageEnabled，类型为： boolean，默认为 false**   
当为真时，表示一个片段的 alpha 通道应该被用来生成一个样本覆盖掩码。

---

10.3.3. Multisample State

```
dictionary GPUMultisampleState {
    GPUSize32 count = 1;
    GPUSampleMask mask = 0xFFFFFFFF;
    boolean alphaToCoverageEnabled = false;
};
```

GPUMultisampleState has the following members, which describe how a GPURenderPipeline interacts with a render pass’s multisampled attachments.

**count, of type GPUSize32, defaulting to 1**       
Number of samples per pixel. This GPURenderPipeline will be compatible only with attachment textures (colorAttachments and depthStencilAttachment) with matching sampleCounts.

**mask, of type GPUSampleMask, defaulting to 0xFFFFFFFF**       
Mask determining which samples are written to.

**alphaToCoverageEnabled, of type boolean, defaulting to false**        
When true indicates that a fragment’s alpha channel should be used to generate a sample coverage mask.

---
---

**验证 GPUMultisampleState(descriptor)：**      

**参数：**
* GPUMultisampleState 描述符

如果满足以下所有条件，则返回 true：
* 如果 descriptor.alphaToCoverageEnabled 为真：
    * descriptor.count > 1.

---

**validating GPUMultisampleState(descriptor) Arguments:**       
* GPUMultisampleState descriptor

Return true if all of the following conditions are satisfied:
* If descriptor.alphaToCoverageEnabled is true:
    * descriptor.count > 1.

---
---

### 10.3.4. 片段状态

```
dictionary GPUFragmentState : GPUProgrammableStage {
    required sequence<GPUColorTargetState?> targets;
};
```

**验证 GPUFragmentState(GPUDevice device, GPUFragmentState descriptor)**        

如果满足以下所有要求，则返回 true：
* descriptor.targets.length 必须小于等于 device.[[limit]].maxColorAttachments。
* 片段输出资源的总数必须小于等于 device.[[limit]].maxFragmentCombinedOutputResources。
* 对于 descriptor.target 的每个索引包含一个非空 colorState 值：
    * colorState.format 必须是 §26.1.1 表中列出的具有 RENDER_ATTACHMENT 功能的普通颜色格式。
    * 如果提供了 colorState.blend：
        * colorState.format 必须是可混合的。
        * colorState.blend.color 必须是一个有效的 GPUBlendComponent。
        * colorState.blend.alpha 必须是一个有效的 GPUBlendComponent。
    * colorState.writeMask 必须是小于 16。
    * 如果 descriptor.entryPoint 有一个着色器阶段的输出值输出，且其地址属性等于索引：
        * 对于 colorState.format 中的每个分量，输出中必须有一个相应的分量。(也就是说，RGBA 需要 vec4，RGB 需要vec3 或 vec4 ，RG 需要 vec2 或 vec3 或 vec4。)
        * 如果提供了colorState.blend，并且 colorState.blend.color.srcFactor 或 colorState.blend.color.dstFactor 使用了 alpha 源（即 "src-alpha"、"one-minus-src-alpha" 或 "src-alpha-saturated" 中的任何一个），那么：
            * 输出必须包含一个alpha通道（也就是说，它必须是vec4）。     
            否则，由于该附件没有着色器输出：
            * colorState.writeMask 必须为 0。
* 验证 GPUFragmentState 的颜色附件字节数的每一个样本(device, descriptor.targets) 的函数返回成功。

---

10.3.4. Fragment State

```
dictionary GPUFragmentState : GPUProgrammableStage {
    required sequence<GPUColorTargetState?> targets;
};
```

**validating GPUFragmentState(GPUDevice device, GPUFragmentState descriptor)**      

Return true if all of the following requirements are met:
* descriptor.targets.length must be ≤ device.[[limits]].maxColorAttachments.
* The total number of fragment output resources must be ≤ device.[[limits]].maxFragmentCombinedOutputResources.
* For each index of the indices of descriptor.targets containing a non-null value colorState:
    * colorState.format must be listed in § 26.1.1 Plain color formats with RENDER_ATTACHMENT capability.
    * If colorState.blend is provided:
        * The colorState.format must be blendable.
        * colorState.blend.color must be a valid GPUBlendComponent.
        * colorState.blend.alpha must be a valid GPUBlendComponent.
    * colorState.writeMask must be < 16.
    * If descriptor.entryPoint has a shader stage output value output with location attribute equal to index:
        * For each component in colorState.format, there must be a corresponding component in output. (That is, RGBA requires vec4, RGB requires vec3 or vec4, RG requires vec2 or vec3 or vec4.)
        * If colorState.blend is provided and colorState.blend.color.srcFactor or .dstFactor uses the source alpha (is any of "src-alpha", "one-minus-src-alpha", or "src-alpha-saturated"), then:
            * output must have an alpha channel (that is, it must be a vec4).
        * Otherwise, since there is no shader output for the attachment:
            * colorState.writeMask must be 0.
* Validating GPUFragmentState’s color attachment bytes per sample(device, descriptor.targets) succeeds.


---
---

一个 GPUFragmentState 描述符的片段输出资源总数是以下各项之和：
* descriptor.target 中非空元素的数量。
* 描述符静态使用的 存储缓冲区 和 存储纹理（任何访问模式）的数量

---

The total number of fragment output resources for a GPUFragmentState descriptor is the sum of:
* the number of non-null elements in descriptor.targets
* the number of storage buffers and storage textures (of any access mode) statically used by descriptor

---
---

验证 GPUFragmentState 的每个样本的颜色附件字节数(GPUDevice device, sequence<GPUColorTargetState?> targets)
1. 让格式成为一个值类型为<GPUTextureFormat?> 的 空列表。
2. 对于 targets 中的每个target：
    1. 如果目标是未定义的，继续。
    2. 将 target.format 追加到格式列表中。
3. 计算每个样本的颜色附件字节数(格式)必须小于等于 device.[[limit]].maxColorAttachmentBytesPerSample。

---

Validating GPUFragmentState’s color attachment bytes per sample(GPUDevice device, sequence<GPUColorTargetState?> targets)

1. Let formats be an empty list<GPUTextureFormat?>
2. For each target in targets:
    1. If target is undefined, continue.
    2. Append target.format to formats.
3. Calculating color attachment bytes per sample(formats) must be ≤ device.[[limits]].maxColorAttachmentBytesPerSample.

---
---

**注意：** 片段着色器可能会输出比管道所使用的更多的值。如果是这种情况，这些值会被忽略。

如果符合以下要求，则组件是有效的 GPUBlendComponent：
* 如果 component.operations 是 "min" 或 "max"：
    * component.srcFactor 和 component.dstFactor 必须都是 "one"。

---
**NOTE:** The fragment shader may output more values than what the pipeline uses. If that is the case the values are ignored.

component is a valid GPUBlendComponent if it meets the following requirements:
* If component.operation is "min" or "max":
    * component.srcFactor and component.dstFactor must both be "one".

---
---

10.3.5. 目标颜色状态

```
dictionary GPUColorTargetState {
    required GPUTextureFormat format;

    GPUBlendState blend;
    GPUColorWriteFlags writeMask = 0xF;  // GPUColorWrite.ALL
};
```

```
dictionary GPUBlendState {
    required GPUBlendComponent color;
    required GPUBlendComponent alpha;
};
```

```
typedef [EnforceRange] unsigned long GPUColorWriteFlags;
[Exposed=(Window, DedicatedWorker), SecureContext]
namespace GPUColorWrite {
    const GPUFlagsConstant RED   = 0x1;
    const GPUFlagsConstant GREEN = 0x2;
    const GPUFlagsConstant BLUE  = 0x4;
    const GPUFlagsConstant ALPHA = 0x8;
    const GPUFlagsConstant ALL   = 0xF;
};
```

---

10.3.5. Color Target State

```
dictionary GPUColorTargetState {
    required GPUTextureFormat format;

    GPUBlendState blend;
    GPUColorWriteFlags writeMask = 0xF;  // GPUColorWrite.ALL
};
```

```
dictionary GPUBlendState {
    required GPUBlendComponent color;
    required GPUBlendComponent alpha;
};
```

```
typedef [EnforceRange] unsigned long GPUColorWriteFlags;
[Exposed=(Window, DedicatedWorker), SecureContext]
namespace GPUColorWrite {
    const GPUFlagsConstant RED   = 0x1;
    const GPUFlagsConstant GREEN = 0x2;
    const GPUFlagsConstant BLUE  = 0x4;
    const GPUFlagsConstant ALPHA = 0x8;
    const GPUFlagsConstant ALL   = 0xF;
};
```

---
---

#### 10.3.5.1. 融合状态

```
dictionary GPUBlendComponent {
    GPUBlendOperation operation = "add";
    GPUBlendFactor srcFactor = "one";
    GPUBlendFactor dstFactor = "zero";
};
```

GPUBlendComponent 有以下成员，它们描述了片段的颜色或透明度成分是如何融合的：

**operation，类型为： GPUBlendOperation，默认为 "add"**     
定义了用于计算写入目标附件组件的值的 GPUBlendOperation。

**srcFactor，类型为： GPUBlendFactor，默认为 "1"**      
定义了要对来自片段着色器的值进行的 GPUBlendFactor 操作。

**dstFactor，类型为： GPUBlendFactor，默认为 "0"**      
定义对目标附件的值进行的 GPUBlendFactor 操作。

---

10.3.5.1. Blend State

```
dictionary GPUBlendComponent {
    GPUBlendOperation operation = "add";
    GPUBlendFactor srcFactor = "one";
    GPUBlendFactor dstFactor = "zero";
};
```

GPUBlendComponent has the following members, which describe how the color or alpha components of a fragment are blended:

**operation, of type GPUBlendOperation, defaulting to "add"**       
Defines the GPUBlendOperation used to calculate the values written to the target attachment components.

**srcFactor, of type GPUBlendFactor, defaulting to "one"**          
Defines the GPUBlendFactor operation to be performed on values from the fragment shader.

**dstFactor, of type GPUBlendFactor, defaulting to "zero"**         
Defines the GPUBlendFactor operation to be performed on values from the target attachment.

---
---

下表使用这种符号来描述特定片段地址的颜色分量：

|  |  |
| --- | --- |
| RGBA<sub>src</sub> | 片段着色器为颜色附件输出的颜色。如果着色器没有返回 alpha 通道，就不能使用 src-alpha 融合因子。|
| RGBA<sub>dst</sub> | 颜色附件中当前的颜色。缺少的绿色/蓝色/alpha通道默认分别为0、0、1。 |
| RGBA<sub>const</sub> | 当前的[[blendConstant]]。 |
| RGBA<sub>srcFactor</sub> | 源融合因子分量，由 srcFactor 定义。 |
| RGBA<sub>dstFactor</sub> | 目标融合因子分量，由dstFactor定义。 |

---

The following tables use this notation to describe color components for a given fragment location:

|  |  |
| --- | --- |
| RGBA<sub>src</sub> | Color output by the fragment shader for the color attachment. If the shader doesn’t return an alpha channel, src-alpha blend factors cannot be used. |
| RGBA<sub>dst</sub> | Color currently in the color attachment. Missing green/blue/alpha channels default to 0, 0, 1, respectively. |
| RGBA<sub>const</sub> | The current [[blendConstant]]. |
| RGBA<sub>srcFactor</sub> | The source blend factor components, as defined by srcFactor. |
| RGBA<sub>dstFactor</sub> | The destination blend factor components, as defined by dstFactor. |

---
---

```
enum GPUBlendFactor {
    "zero",
    "one",
    "src",
    "one-minus-src",
    "src-alpha",
    "one-minus-src-alpha",
    "dst",
    "one-minus-dst",
    "dst-alpha",
    "one-minus-dst-alpha",
    "src-alpha-saturated",
    "constant",
    "one-minus-constant"
};
```

GPUBlendFactor 定义了如何计算 源或目标的融合系数：

| GPUBlendFactor | 融合因子 RGBA 分量 |
| --- | --- |
| "zero" | (0, 0, 0, 0) |
| "one" | (1, 1, 1, 1) |
| "src" | (R<sub>src</sub>, G<sub>src</sub>, B<sub>src</sub>, A<sub>src</sub>) |
| "one-minus-src" | (1 - R<sub>src</sub>, 1 - G<sub>src</sub>, 1 - B<sub>src</sub>, 1 - A<sub>src</sub>) |
| "src-alpha" | (A<sub>src</sub>, A<sub>src</sub>, A<sub>src</sub>, A<sub>src</sub>) |
| "one-minus-src-alpha" | (1 - A<sub>src</sub>, 1 - A<sub>src</sub>, 1 - A<sub>src</sub>, 1 - A<sub>src</sub>) |
| "dst" | (R<sub>dst</sub>, G<sub>dst</sub>, B<sub>dst</sub>, A<sub>dst</sub>) |
| "one-minus-dst" | (1 - R<sub>dst</sub>, 1 - G<sub>dst</sub>, 1 - B<sub>dst</sub>, 1 - A<sub>dst</sub>) |
| "dst-alpha" | (A<sub>dst</sub>, A<sub>dst</sub>, A<sub>dst</sub>, A<sub>dst</sub>) |
| "one-minus-dst-alpha" | (1 - A<sub>dst</sub>, 1 - A<sub>dst</sub>, 1 - A<sub>dst</sub>, 1 - A<sub>dst</sub>) |
| "src-alpha-saturated" | (min(A<sub>src</sub>, 1 - A<sub>dst</sub>), min(A<sub>src</sub>, 1 - A<sub>dst</sub>), min(A<sub>src</sub>, 1 - A<sub>dst</sub>), 1) |
| "constant" | (R<sub>const</sub>, G<sub>const</sub>, B<sub>const</sub>, A<sub>const</sub>) |
| "one-minus-constant" | (1 - R<sub>const</sub>, 1 - G<sub>const</sub>, 1 - B<sub>const</sub>, 1 - A<sub>const</sub>) |

---

```
enum GPUBlendFactor {
    "zero",
    "one",
    "src",
    "one-minus-src",
    "src-alpha",
    "one-minus-src-alpha",
    "dst",
    "one-minus-dst",
    "dst-alpha",
    "one-minus-dst-alpha",
    "src-alpha-saturated",
    "constant",
    "one-minus-constant"
};
```

GPUBlendFactor defines how either a source or destination blend factors is calculated:

| GPUBlendFactor | Blend factor RGBA components |
| --- | --- |
| "zero" | (0, 0, 0, 0) |
| "one" | (1, 1, 1, 1) |
| "src" | (R<sub>src</sub>, G<sub>src</sub>, B<sub>src</sub>, A<sub>src</sub>) |
| "one-minus-src" | (1 - R<sub>src</sub>, 1 - G<sub>src</sub>, 1 - B<sub>src</sub>, 1 - A<sub>src</sub>) |
| "src-alpha" | (A<sub>src</sub>, A<sub>src</sub>, A<sub>src</sub>, A<sub>src</sub>) |
| "one-minus-src-alpha" | (1 - A<sub>src</sub>, 1 - A<sub>src</sub>, 1 - A<sub>src</sub>, 1 - A<sub>src</sub>) |
| "dst" | (R<sub>dst</sub>, G<sub>dst</sub>, B<sub>dst</sub>, A<sub>dst</sub>) |
| "one-minus-dst" | (1 - R<sub>dst</sub>, 1 - G<sub>dst</sub>, 1 - B<sub>dst</sub>, 1 - A<sub>dst</sub>) |
| "dst-alpha" | (A<sub>dst</sub>, A<sub>dst</sub>, A<sub>dst</sub>, A<sub>dst</sub>) |
| "one-minus-dst-alpha" | (1 - A<sub>dst</sub>, 1 - A<sub>dst</sub>, 1 - A<sub>dst</sub>, 1 - A<sub>dst</sub>) |
| "src-alpha-saturated" | (min(A<sub>src</sub>, 1 - A<sub>dst</sub>), min(A<sub>src</sub>, 1 - A<sub>dst</sub>), min(A<sub>src</sub>, 1 - A<sub>dst</sub>), 1) |
| "constant" | (R<sub>const</sub>, G<sub>const</sub>, B<sub>const</sub>, A<sub>const</sub>) |
| "one-minus-constant" | (1 - R<sub>const</sub>, 1 - G<sub>const</sub>, 1 - B<sub>const</sub>, 1 - A<sub>const</sub>) |

---
---

```
enum GPUBlendOperation {
    "add",
    "subtract",
    "reverse-subtract",
    "min",
    "max"
};
```

GPUBlendOperation 定义了用于融合源和目标融合因子的算法：

| GPUBlendOperation | RGBA 分量 |
| --- | --- |
| "add" | RGBA<sub>src</sub> × RGBA<sub>srcFactor</sub> + RGBA<sub>dst</sub> × RGBA<sub>dstFactor</sub> |
| "subtract" | RGBA<sub>src</sub> × RGBA<sub>srcFactor</sub> - RGBA<sub>dst</sub> × RGBA<sub>dstFactor</sub> |
| "reverse-subtract" | RGBA<sub>dst</sub> × RGBA<sub>dstFactor</sub> - RGBA<sub>src</sub> × RGBA<sub>srcFactor</sub> |
| "min" | min(RGBA<sub>src</sub>, RGBA<sub>dst</sub>) |
| "max" | max(RGBA<sub>src</sub>, RGBA<sub>dst</sub>) |

---

```
enum GPUBlendOperation {
    "add",
    "subtract",
    "reverse-subtract",
    "min",
    "max"
};
```

GPUBlendOperation defines the algorithm used to combine source and destination blend factors:

| GPUBlendOperation | RGBA Components |
| --- | --- |
| "add" | RGBA<sub>src</sub> × RGBA<sub>srcFactor</sub> + RGBA<sub>dst</sub> × RGBA<sub>dstFactor</sub> |
| "subtract" | RGBA<sub>src</sub> × RGBA<sub>srcFactor</sub> - RGBA<sub>dst</sub> × RGBA<sub>dstFactor</sub> |
| "reverse-subtract" | RGBA<sub>dst</sub> × RGBA<sub>dstFactor</sub> - RGBA<sub>src</sub> × RGBA<sub>srcFactor</sub> |
| "min" | min(RGBA<sub>src</sub>, RGBA<sub>dst</sub>) |
| "max" | max(RGBA<sub>src</sub>, RGBA<sub>dst</sub>) |

---
---

### 10.3.6. 深度/模板 状态

```
dictionary GPUDepthStencilState {
    required GPUTextureFormat format;

    required boolean depthWriteEnabled;
    required GPUCompareFunction depthCompare;

    GPUStencilFaceState stencilFront = {};
    GPUStencilFaceState stencilBack = {};

    GPUStencilValue stencilReadMask = 0xFFFFFFFF;
    GPUStencilValue stencilWriteMask = 0xFFFFFFFF;

    GPUDepthBias depthBias = 0;
    float depthBiasSlopeScale = 0;
    float depthBiasClamp = 0;
};
```

GPUDepthStencilState 有以下成员，它们描述了 GPURenderPipeline 将如何影响一个渲染通道的 depthStencilAttachment ：

**format，类型为： GPUTextureFormat**      
表示与此 GPURenderPipeline 将兼容的 depthStencilAttachment 的格式。     

**depthWriteEnabled，类型为： boolean**          
表示此 GPURenderPipeline 是否可以修改 depthStencilAttachment 的深度值。

**depthCompare, 类型为： GPUCompareFunction**        
表示用来测试片段深度 与 depthStencilAttachment深度值的比较操作。

**stencilFront，类型为： GPUStencilFaceState，默认为 {}。**     
定义如何对前面的基元进行模板比较和操作。

**stencilBack，类型为：GPUStencilFaceState，默认为 {}。**  
定义如何对面向后方的基元进行模板比较和操作。

**stencilReadMask，类型为： GPUStencilValue，默认为 0xFFFFFFFF**        
Bitmask，控制在执行模板比较测试时读取哪些 depthStencilAttachment 的模板值位。

**stencilWriteMask，类型为： GPUStencilValue，默认为 0xFFFFFFFF。**     
Bitmask，控制在执行模板操作时，哪些 depthStencilAttachment 模板值位被写入。

**depthBias，类型为： GPUDepthBias，默认为 0**      
添加到每个片段的恒定深度偏置值。详见偏置的片段深度。

**depthBiasSlopeScale，类型为： float，默认为0。**           
与片段的斜率成比例的深度偏差。详见偏置的片段深度。

**depthBiasClamp，类型为： 浮点数，默认为0。**      
片段的最大深度偏差。详见偏置的片段深度。

---

10.3.6. Depth/Stencil State

```
dictionary GPUDepthStencilState {
    required GPUTextureFormat format;

    required boolean depthWriteEnabled;
    required GPUCompareFunction depthCompare;

    GPUStencilFaceState stencilFront = {};
    GPUStencilFaceState stencilBack = {};

    GPUStencilValue stencilReadMask = 0xFFFFFFFF;
    GPUStencilValue stencilWriteMask = 0xFFFFFFFF;

    GPUDepthBias depthBias = 0;
    float depthBiasSlopeScale = 0;
    float depthBiasClamp = 0;
};
```

GPUDepthStencilState has the following members, which describe how a GPURenderPipeline will affect a render pass’s depthStencilAttachment:

**format, of type GPUTextureFormat**        
The format of depthStencilAttachment this GPURenderPipeline will be compatible with.

**depthWriteEnabled, of type boolean**      
Indicates if this GPURenderPipeline can modify depthStencilAttachment depth values.

**depthCompare, of type GPUCompareFunction**        
The comparison operation used to test fragment depths against depthStencilAttachment depth values.

**stencilFront, of type GPUStencilFaceState, defaulting to {}**     
Defines how stencil comparisons and operations are performed for front-facing primitives.

**stencilBack, of type GPUStencilFaceState, defaulting to {}**      
Defines how stencil comparisons and operations are performed for back-facing primitives.

**stencilReadMask, of type GPUStencilValue, defaulting to 0xFFFFFFFF**      
Bitmask controlling which depthStencilAttachment stencil value bits are read when performing stencil comparison tests.

**stencilWriteMask, of type GPUStencilValue, defaulting to 0xFFFFFFFF**     
Bitmask controlling which depthStencilAttachment stencil value bits are written to when performing stencil operations.

**depthBias, of type GPUDepthBias, defaulting to 0**        
Constant depth bias added to each fragment. See biased fragment depth for details.

**depthBiasSlopeScale, of type float, defaulting to 0**     
Depth bias that scales with the fragment’s slope. See biased fragment depth for details.

**depthBiasClamp, of type float, defaulting to 0**      
The maximum depth bias of a fragment. See biased fragment depth for details.

---
---

在使用 GPUDepthStencilState 状态进行绘图时，被写入 depthStencilAttachment 附件的偏置片段深度值，是通过运行以下步骤计算的：      
1. 让格式为 attachment.view.format。
2. 让 r 表示最小正数的值 且要大于 0 在32位浮点数格式中的转换值。
3. 让 maxDepthSlope 是片段的深度值中，水平方向和垂直方向斜率的最大值。
4. 如果格式是一个 unorm 格式：
    1. 让 bias 成为 (float) state.depthBias * r + state.depthBiasSlopeScale * maxDepthSlope。
5. 否则，如果格式是一个浮点格式：
    1. 让 bias 成为 (float)state.depthBias * 2^(exp(max depth in primitive) - r) + state.depthBiasSlopeScale * maxDepthSlope。
6. 如果 state.depthBiasClamp > 0：
    1. 将 bias 设置为 min(state.depthBiasClamp, bias)。
7. 否则，如果 state.depthBiasClamp < 0：
    1. 将 bias 设置为 max(state.depthBiasClamp, bias)。
8. 如果 state.depthBias ≠ 0 或者 state.depthBiasSlopeScale ≠ 0：
    1. 将片段深度值设置为 片段深度值 + bias

---

The biased fragment depth for a fragment being written to depthStencilAttachment attachment when drawing using GPUDepthStencilState state is calculated by running the following steps:
1. Let format be attachment.view.format.
2. Let r be the minimum positive representable value > 0 in the format converted to a 32-bit float.
3. Let maxDepthSlope be the maximum of the horizontal and vertical slopes of the fragment’s depth value.
4. If format is a unorm format:
   1. Let bias be (float)state.depthBias * r + state.depthBiasSlopeScale * maxDepthSlope.
5. Otherwise, if format is a float format:
    1. Let bias be (float)state.depthBias * 2^(exp(max depth in primitive) - r) + state.depthBiasSlopeScale * maxDepthSlope.
6. If state.depthBiasClamp > 0:
    1. Set bias to min(state.depthBiasClamp, bias).
7. Otherwise if state.depthBiasClamp < 0:
    1. Set bias to max(state.depthBiasClamp, bias).
8. If state.depthBias ≠ 0 or state.depthBiasSlopeScale ≠ 0:
    1. Set the fragment depth value to fragment depth value + bias

---
---

**验证GPUDepthStencilState(descriptor)**   

**参数：**
* GPUDepthStencilState描述符

当且仅当以下所有条件都得到满足时，返回true：
* descriptor.format 是一种 深度/模板格式。
* 如果 descriptor.depthWriteEnabled 为真或 descriptor.depthCompare 不是 "always"：
    * descriptor.format 必须有一个深度分量。
* 如果 descriptor.stencilFront 或 descriptor.stencilBack 不是默认值：
    * descriptor.format必须有一个 stencil 分量。

>> ISSUE 16 该算法如何支持在扩展中添加的深度/模板格式？

---

**validating GPUDepthStencilState(descriptor)**     

**Arguments:**

* GPUDepthStencilState descriptor

Return true if, and only if, all of the following conditions are satisfied:
* descriptor.format is a depth-or-stencil format.
* If descriptor.depthWriteEnabled is true or descriptor.depthCompare is not "always":
    * descriptor.format must have a depth component.
* If descriptor.stencilFront or descriptor.stencilBack are not the default values:
    * descriptor.format must have a stencil component.

>> ISSUE 16 how can this algorithm support depth/stencil formats that are added in extensions?

---
---

```
dictionary GPUStencilFaceState {
    GPUCompareFunction compare = "always";
    GPUStencilOperation failOp = "keep";
    GPUStencilOperation depthFailOp = "keep";
    GPUStencilOperation passOp = "keep";
};
```

GPUStencilFaceState 有以下成员，它们描述了如何进行模板比较和操作：            
**compare，类型为: GPUCompareFunction，默认为 "always"**        
当测试片段时，与 depthStencilAttachment 的模板值进行比较时，使用的GPUCompareFunction。

**failOp，类型为： GPUStencilOperation，默认为 "keep"**     
如果由比较所描述的片段模板比较测试失败了，则执行 GPUStencilOperation 函数。

**depthFailOp，类型为： GPUStencilOperation，默认为 "keep"**     
如果由 depthCompare所描述的片段深度比较失败了，则执行 GPUStencilOperation 函数。

**passOp，类型为keep： GPUStencilOperation，默认为 "keep"**     
如果由 compare 所描述的片段模板比较测试通过了，则执行 GPUStencilOperation。

---

```
dictionary GPUStencilFaceState {
    GPUCompareFunction compare = "always";
    GPUStencilOperation failOp = "keep";
    GPUStencilOperation depthFailOp = "keep";
    GPUStencilOperation passOp = "keep";
};
```

GPUStencilFaceState has the following members, which describe how stencil comparisons and operations are performed:

**compare, of type GPUCompareFunction, defaulting to "always"**     
The GPUCompareFunction used when testing fragments against depthStencilAttachment stencil values.

**failOp, of type GPUStencilOperation, defaulting to "keep"**       
The GPUStencilOperation performed if the fragment stencil comparison test described by compare fails.

**depthFailOp, of type GPUStencilOperation, defaulting to "keep"**      
The GPUStencilOperation performed if the fragment depth comparison described by depthCompare fails.

**passOp, of type GPUStencilOperation, defaulting to "keep"**       
The GPUStencilOperation performed if the fragment stencil comparison test described by compare passes.

---
---

```
enum GPUStencilOperation {
    "keep",
    "zero",
    "replace",
    "invert",
    "increment-clamp",
    "decrement-clamp",
    "increment-wrap",
    "decrement-wrap"
};
```

GPUStencilOperation 定义了以下操作：        
**"keep"**      
保持当前的模板值。

**"zero"**        
将模板值设为0。

**"replace"**      
将模析值设置为 [[stencilReference]]。

**"invert"**      
对当前模板值进行比特位反转化。

**"increment-clamp"**     
递增当前的模板值，超过的值将被钳制到 depthStencilAttachment 的模板方面的最大可表示值。

**"decrement-clamp"**     
递减当前的模板值，超过的值将被钳制到 0。

**"increment-wrap"**     
递增当前的模板值，如果该值超过 depthStencilAttachment 的模板方面的最大可表示值，则包裹为 零。

**"decrement-wrap"**     
递减当前的模板值，如果值低于0，则包裹到 depthStencilAttachment 的模板方面的最大可表示值。

---

```
enum GPUStencilOperation {
    "keep",
    "zero",
    "replace",
    "invert",
    "increment-clamp",
    "decrement-clamp",
    "increment-wrap",
    "decrement-wrap"
};
```

GPUStencilOperation defines the following operations:

**"keep"**      
Keep the current stencil value.

**"zero"**      
Set the stencil value to 0.

**"replace"**       
Set the stencil value to [[stencilReference]].

**"invert"**        
Bitwise-invert the current stencil value.

**"increment-clamp"**       
Increments the current stencil value, clamping to the maximum representable value of the depthStencilAttachment's stencil aspect.

**"decrement-clamp"**       
Decrement the current stencil value, clamping to 0.

**"increment-wrap"**        
Increments the current stencil value, wrapping to zero if the value exceeds the maximum representable value of the depthStencilAttachment's stencil aspect.

**"decrement-wrap"**        
Decrement the current stencil value, wrapping to the maximum representable value of the depthStencilAttachment's stencil aspect if the value goes below 0.

---
---

### 10.3.7. 顶点状态

```
enum GPUIndexFormat {
    "uint16",
    "uint32"
};
```

索引格式决定了缓冲区内索引值的数据类型，与条状基元拓扑结构（"line-strip "或 "triangle-strip"）一起使用时，还指定了基元重启值。基元重启值指示哪个索引值表示应开始一个新的基元，而不是继续用先前的索引顶点构建三角形条带。

指定带状基元拓扑结构的 GPUPrimitiveStates 如果用于索引绘制，则必须指定 stripIndexFormat，以便在管道创建时知道将被使用的基元重启值。GPUPrimitiveStates 指定了基元拓扑结构的列表， 在进行索引绘制时将列表中的某个索引格式传递给  setIndexBuffer() 使用。

| 索引格式 | 字节大小 | 基元重启值 |
| --- | --- | --- |
| "uint16" | 2 | 0xFFFF |
| "uint32" | 4 | 0xFFFFFFFF |

---

10.3.7. Vertex State

```
enum GPUIndexFormat {
    "uint16",
    "uint32"
};
```

The index format determines both the data type of index values in a buffer and, when used with strip primitive topologies ("line-strip" or "triangle-strip") also specifies the primitive restart value. The primitive restart value indicates which index value indicates that a new primitive should be started rather than continuing to construct the triangle strip with the prior indexed vertices.

GPUPrimitiveStates that specify a strip primitive topology must specify a stripIndexFormat if they are used for indexed draws so that the primitive restart value that will be used is known at pipeline creation time. GPUPrimitiveStates that specify a list primitive topology will use the index format passed to setIndexBuffer() when doing indexed rendering.

| Index format | Byte size | Primitive restart value |
| --- | --- | --- |
| "uint16" | 2 | 0xFFFF |
| "uint32" | 4 | 0xFFFFFFFF |

---
---

#### 10.3.7.1. 顶点格式
顶点属性 GPUVertexFormat 表明来自顶点缓冲区的数据将如何被解释并暴露给着色器。格式的名称指定了分量的顺序、每个分量的比特值，以及顶点数据分量的类型。

***译注： 这里分量指的数据的一部分，比如表示颜色的 RGBA 数据，其中的 R、G、B、A 就分别是一个红色、绿色、蓝色、透明度分量。***

每个顶点数据类型可以映射到相同基本类型的任何 WGSL 标量类型，而不考虑每个分量的位数：

| 顶点格式前缀 | 顶点数据类型 | 兼容的 WGSL 类型 |
| --- | --- | --- |
| uint | unsigned int | u32 |
| sint | signed int | i32 |
| unorm | unsigned normalized | f16, f32 |
| snorm | signed normalized | f16, f32 |
| float | floating point | f16, f32 |

多分量格式指定了 "x"之后的分量数量。顶点格式和着色器类型之间的分量数量不匹配是允许的，分量要么被删除，要么用默认值填充以进行补偿。

---

10.3.7.1. Vertex Formats

The GPUVertexFormat of a vertex attribute indicates how data from a vertex buffer will be interpreted and exposed to the shader. The name of the format specifies the order of components, bits per component, and vertex data type for the component.

Each vertex data type can map to any WGSL scalar type of the same base type, regardless of the bits per component:

| Vertex format prefix | Vertex data type | Compatible WGSL types |
| --- | --- | --- |
| uint | unsigned int | u32 |
| sint | signed int | i32 |
| unorm | unsigned normalized | f16, f32 |
| snorm | signed normalized | f16, f32 |
| float | floating point | f16, f32 |

The multi-component formats specify the number of components after "x". Mismatches in the number of components between the vertex format and shader type are allowed, with components being either dropped or filled with default values to compensate.

---
---

例子 24 

一个顶点的格式为 "unorm8x2"，其字节值为[0x7F, 0xFF]的属性，可以在着色器中以下列类型访问：

| 着色器类型 | 着色器值 | 
| --- | --- |
| f16 | 0.5h | 
| f32 | 0.5f | 
| vec2<f16> | vec2(0.5h, 1.0h) | 
| vec2<f32> | vec2(0.5f, 1.0f) | 
| vec3<f16> | vec2(0.5h, 1.0h, 0.0h) | 
| vec3<f32> | vec2(0.5f, 1.0f, 0.0f) | 
| vec4<f16> | vec2(0.5h, 1.0h, 0.0h, 1.0h) | 
| vec4<f32> | vec2(0.5f, 1.0f, 0.0f, 1.0f) | 

---

EXAMPLE 24 

A vertex attribute with a format of "unorm8x2" and byte values [0x7F, 0xFF] can be accessed in the shader with the following types:

| Shader type | Shader value | 
| --- | --- |
| f16 | 0.5h | 
| f32 | 0.5f | 
| vec2<f16> | vec2(0.5h, 1.0h) | 
| vec2<f32> | vec2(0.5f, 1.0f) | 
| vec3<f16> | vec2(0.5h, 1.0h, 0.0h) | 
| vec3<f32> | vec2(0.5f, 1.0f, 0.0f) | 
| vec4<f16> | vec2(0.5h, 1.0h, 0.0h, 1.0h) | 
| vec4<f32> | vec2(0.5f, 1.0f, 0.0f, 1.0f) | 

---
---

关于顶点格式如何在着色器中暴露的其他信息，请参见 § 23.3.2 顶点处理章节

```
enum GPUVertexFormat {
    "uint8x2",
    "uint8x4",
    "sint8x2",
    "sint8x4",
    "unorm8x2",
    "unorm8x4",
    "snorm8x2",
    "snorm8x4",
    "uint16x2",
    "uint16x4",
    "sint16x2",
    "sint16x4",
    "unorm16x2",
    "unorm16x4",
    "snorm16x2",
    "snorm16x4",
    "float16x2",
    "float16x4",
    "float32",
    "float32x2",
    "float32x3",
    "float32x4",
    "uint32",
    "uint32x2",
    "uint32x3",
    "uint32x4",
    "sint32",
    "sint32x2",
    "sint32x3",
    "sint32x4"
};

```

| 顶点格式 | 数据类型 | 分量数 | 字节大小 |  WGSL 中举例类型 |
| --- | --- | --- | --- | --- |
| "uint8x2" | unsigned int | 2 | 2 | vec2<u32> |
| "uint8x4" | unsigned int | 4 | 4 | vec4<u32> |
| "sint8x2" | signed int | 2 | 2 | vec2<i32> |
| "sint8x4" | signed int | 4 | 4 | vec4<i32> |
| "unorm8x2" | unsigned normalized | 2 | 2 | vec2<f32> |
| "unorm8x4" | unsigned normalized | 4 | 4 | vec4<f32> |
| "snorm8x2" | signed normalized | 2 | 2 | vec2<f32> |
| "snorm8x4" | signed normalized | 4 | 4 | vec4<f32> |
| "uint16x2" | unsigned int | 2 | 4 | vec2<u32> |
| "uint16x4" | unsigned int | 4 | 8 | vec4<u32> |
| "sint16x2" | signed int | 2 | 4 | vec2<i32> |
| "sint16x4" | signed int | 4 | 8 | vec4<i32> |
| "unorm16x2" | unsigned normalized | 2 | 4 | vec2<f32> |
| "unorm16x4" | unsigned normalized | 4 | 8 | vec4<f32> |
| "snorm16x2" | signed normalized | 2 | 4 | vec2<f32> |
| "snorm16x4" | signed normalized | 4 | 8 | vec4<f32> |
| "float16x2" | float | 2 | 4 | vec2<f16> |
| "float16x4" | float | 4 | 8 | vec4<f16> |
| "float32" | float | 1 | 4 | f32 |
| "float32x2" | float | 2 | 8 | vec2<f32> |
| "float32x3" | float | 3 | 12 | vec3<f32> |
| "float32x4" | float | 4 | 16 | vec4<f32> |
| "uint32" | unsigned int | 1 | 4 | u32 |
| "uint32x2" | unsigned int | 2 | 8 | vec2<u32> |
| "uint32x3" | unsigned int | 3 | 12 | vec3<u32> |
| "uint32x4" | unsigned int | 4 | 16 | vec4<u32> |
| "sint32" | signed int | 1 | 4 | i32 |
| "sint32x2" | signed int | 2 | 8 | vec2<i32> |
| "sint32x3" | signed int | 3 | 12 | vec3<i32> |
| "sint32x4" | signed int | 4 | 16 | vec4<i32> |

---

See § 23.3.2 Vertex Processing for additional information about how vertex formats are exposed in the shader.

```
enum GPUVertexFormat {
    "uint8x2",
    "uint8x4",
    "sint8x2",
    "sint8x4",
    "unorm8x2",
    "unorm8x4",
    "snorm8x2",
    "snorm8x4",
    "uint16x2",
    "uint16x4",
    "sint16x2",
    "sint16x4",
    "unorm16x2",
    "unorm16x4",
    "snorm16x2",
    "snorm16x4",
    "float16x2",
    "float16x4",
    "float32",
    "float32x2",
    "float32x3",
    "float32x4",
    "uint32",
    "uint32x2",
    "uint32x3",
    "uint32x4",
    "sint32",
    "sint32x2",
    "sint32x3",
    "sint32x4"
};

```

| Vertex format | Data type | Components | Byte size | Example WGSL type |
| --- | --- | --- | --- | --- |
| "uint8x2" | unsigned int | 2 | 2 | vec2<u32> |
| "uint8x4" | unsigned int | 4 | 4 | vec4<u32> |
| "sint8x2" | signed int | 2 | 2 | vec2<i32> |
| "sint8x4" | signed int | 4 | 4 | vec4<i32> |
| "unorm8x2" | unsigned normalized | 2 | 2 | vec2<f32> |
| "unorm8x4" | unsigned normalized | 4 | 4 | vec4<f32> |
| "snorm8x2" | signed normalized | 2 | 2 | vec2<f32> |
| "snorm8x4" | signed normalized | 4 | 4 | vec4<f32> |
| "uint16x2" | unsigned int | 2 | 4 | vec2<u32> |
| "uint16x4" | unsigned int | 4 | 8 | vec4<u32> |
| "sint16x2" | signed int | 2 | 4 | vec2<i32> |
| "sint16x4" | signed int | 4 | 8 | vec4<i32> |
| "unorm16x2" | unsigned normalized | 2 | 4 | vec2<f32> |
| "unorm16x4" | unsigned normalized | 4 | 8 | vec4<f32> |
| "snorm16x2" | signed normalized | 2 | 4 | vec2<f32> |
| "snorm16x4" | signed normalized | 4 | 8 | vec4<f32> |
| "float16x2" | float | 2 | 4 | vec2<f16> |
| "float16x4" | float | 4 | 8 | vec4<f16> |
| "float32" | float | 1 | 4 | f32 |
| "float32x2" | float | 2 | 8 | vec2<f32> |
| "float32x3" | float | 3 | 12 | vec3<f32> |
| "float32x4" | float | 4 | 16 | vec4<f32> |
| "uint32" | unsigned int | 1 | 4 | u32 |
| "uint32x2" | unsigned int | 2 | 8 | vec2<u32> |
| "uint32x3" | unsigned int | 3 | 12 | vec3<u32> |
| "uint32x4" | unsigned int | 4 | 16 | vec4<u32> |
| "sint32" | signed int | 1 | 4 | i32 |
| "sint32x2" | signed int | 2 | 8 | vec2<i32> |
| "sint32x3" | signed int | 3 | 12 | vec3<i32> |
| "sint32x4" | signed int | 4 | 16 | vec4<i32> |

---
---

```
enum GPUVertexStepMode {
    "vertex",
    "instance"
};
```

步幅模式配置了如何根据当前的顶点或实例索引来计算顶点缓冲区数据的地址：

**"vertex"**        
每一个顶点的地址都以 arrayStride 的方式向前推进，并在实例之间重置。

**"instance"**      
每一个实例的地址都以 arrayStride 的方式向前推进。

---

```
enum GPUVertexStepMode {
    "vertex",
    "instance"
};
```

The step mode configures how an address for vertex buffer data is computed, based on the current vertex or instance index:

**"vertex"**        
The address is advanced by arrayStride for each vertex, and reset between instances.

**"instance"**      
The address is advanced by arrayStride for each instance.

---
---

```
dictionary GPUVertexState : GPUProgrammableStage {
    sequence<GPUVertexBufferLayout?> buffers = [];
};
```

从概念上讲，顶点缓冲区就是缓冲区内存作为一个结构体数组的视图，arrayStride 是该数组元素之间的跨度，单位是字节。顶点缓冲区的每个元素就像一个结构，其内存布局由其属性定义，描述了结构体的成员。

每个 GPUVertexAttribute 描述了每个成员的格式 和它在结构中的偏移量，以字节为单位。

每个属性在顶点着色器中被作为一个单独的输入出现，每个属性都由一个数字地址约束，该地址由 shaderLocation 指定。每个地址在 GPUVertexState 中都必须是唯一的。

---

```
dictionary GPUVertexState : GPUProgrammableStage {
    sequence<GPUVertexBufferLayout?> buffers = [];
};
```

A vertex buffer is, conceptually, a view into buffer memory as an array of structures. arrayStride is the stride, in bytes, between elements of that array. Each element of a vertex buffer is like a structure with a memory layout defined by its attributes, which describe the members of the structure.

Each GPUVertexAttribute describes its format and its offset, in bytes, within the structure.

Each attribute appears as a separate input in a vertex shader, each bound by a numeric location, which is specified by shaderLocation. Every location must be unique within the GPUVertexState.


---
---

```
dictionary GPUVertexBufferLayout {
    required GPUSize64 arrayStride;
    GPUVertexStepMode stepMode = "vertex";
    required sequence<GPUVertexAttribute> attributes;
};
```

**arrayStride，类型为： GPUSize64**     
该数组元素之间的跨度，单位是字节。

**stepMode，类型为： GPUVertexStepMode，默认为 "vertex"**     
这个数组的每个元素是代表每个顶点数据还是每个实例数据

**attributes，类型为： sequence\<GPUVertexAttribute\>。**       
一个数组，其定义在每个元素中的顶点属性的布局。

---

```
dictionary GPUVertexBufferLayout {
    required GPUSize64 arrayStride;
    GPUVertexStepMode stepMode = "vertex";
    required sequence<GPUVertexAttribute> attributes;
};
```

**arrayStride, of type GPUSize64**      
The stride, in bytes, between elements of this array.

**stepMode, of type GPUVertexStepMode, defaulting to "vertex"**     
Whether each element of this array represents per-vertex data or per-instance data

**attributes, of type sequence\<GPUVertexAttribute\>**        
An array defining the layout of the vertex attributes within each element.

---
---

```
dictionary GPUVertexAttribute {
    required GPUVertexFormat format;
    required GPUSize64 offset;

    required GPUIndex32 shaderLocation;
};
```

**format，类型为： GPUVertexFormat**        
表示该属性的 GPUVertexFormat。

**offset，类型为： GPUSize64**      
表示从元素开始地址，到属性的数据的地址的偏移量，单位是字节。

**shaderLocation，类型为： GPUIndex32**     
表示与该属性相关的数字地址，它将对应于在 vertex.module 中声明的 "@location" 属性。

---

```
dictionary GPUVertexAttribute {
    required GPUVertexFormat format;
    required GPUSize64 offset;

    required GPUIndex32 shaderLocation;
};
```

**format, of type GPUVertexFormat**     
The GPUVertexFormat of the attribute.

**offset, of type GPUSize64**       
The offset, in bytes, from the beginning of the element to the data for the attribute.

**shaderLocation, of type GPUIndex32**      
The numeric location associated with this attribute, which will correspond with a "@location" attribute declared in the vertex.module.


---
---

**验证 GPUVertexBufferLayout(device, descriptor, vertexStage)**

**参数：**      
* GPUDevice 设备
* GPUVertexBufferLayout 描述符
* GPUProgrammableStage vertexStage

当且仅当以下所有条件都得到满足时，返回 true：
* descriptor.arrayStride 小于等于 device.[[device]].[[limit]].maxVertexBufferArrayStride。
* descriptor.arrayStride 是 4 的倍数。
* 对于列表中 descriptor.attributes 中的每个属性 attrib：
    * 如果 descriptor.arrayStride 为 零：
        * attrib.offset + sizeof(attrib.format) ≤ device.[[device]].[[limits]].maxVertexBufferArrayStride。         
    否则：
        * attrib.offset + sizeof(attrib.format) ≤ descriptor.arrayStride。
    * attrib.offset 是 4 和 sizeof(attrib.format) 中最小值的倍数。
    * attrib.shaderLocation 是小于 device.[[device]].[[limit]].maxVertexAttributes。
* 对于 vertexStage 静态使用的每一个顶点属性 var，在 descriptor.attributes 中都有一个相应的 attrib 元素，对其而言，以下所有条件都是真的：
    * var的类型 T 与 attrib.format 的顶点数据类型要相兼容：
        **"unorm", "snorm", 或 "float"**        
        则 T 必须是 f32 或 vecN<f32>。
        **"uint"**      
        则 T 必须是 u32 或 vecN<u32>。
        **"sint"**      
        则 T 必须是 i32 或 vecN<i32>。
    * 着色器的地址是 attrib.shaderLocation。

---

**validating GPUVertexBufferLayout(device, descriptor, vertexStage)**       

**Arguments:**
* GPUDevice device
* GPUVertexBufferLayout descriptor
* GPUProgrammableStage vertexStage

Return true, if and only if, all of the following conditions are satisfied:
* descriptor.arrayStride ≤ device.[[device]].[[limits]].maxVertexBufferArrayStride.
* descriptor.arrayStride is a multiple of 4.
* For each attribute attrib in the list descriptor.attributes:
    * If descriptor.arrayStride is zero:
        * attrib.offset + sizeof(attrib.format) ≤ device.[[device]].[[limits]].maxVertexBufferArrayStride.      
        Otherwise:
        * attrib.offset + sizeof(attrib.format) ≤ descriptor.arrayStride.
    * attrib.offset is a multiple of the minimum of 4 and sizeof(attrib.format).
    * attrib.shaderLocation is < device.[[device]].[[limits]].maxVertexAttributes.
* For every vertex attribute var statically used by vertexStage, there is a corresponding attrib element of descriptor.attributes for which all of the following are true:
    * The type T of var is compatible with attrib.format's vertex data type:
    **"unorm", "snorm", or "float"**        
    T must be f32 or vecN<f32>.     
    **"uint"**      
    T must be u32 or vecN<u32>.     
    **"sint"**      
    T must be i32 or vecN<i32>.     
    * The shader location is attrib.shaderLocation.

---
---

**验证 GPUVertexState(device, descriptor)**         

**参数：**      
* GPUDevice 设备
* GPUVertexState 描述符

当且仅当以下所有条件都得到满足时，返回 true：
* descriptor.buffers.length 是小于等于 device.[[device]].[[limit]].maxVertexBuffers.
* 列表中 descriptor.buffers 中的每个顶点缓冲区布局描述符都通过了 GPUVertexBufferLayout(device, vertexBuffer, descriptor) 的验证。
* 在 descriptor.buffers 中的每个顶点缓冲区上，vertexBuffer.attributions.length 的总和是小于等于 device.[[device]].[[limit]].maxVertexAttributes。
* 在 descriptor.buffers 中的所有 GPUVertexAttribute 的合集中，每个 attrib 都有一个独立的 attrib.shaderLocation 值。

---
**validating GPUVertexState(device, descriptor)**

**Arguments:**      
* GPUDevice device
* GPUVertexState descriptor

Return true, if and only if, all of the following conditions are satisfied:
* descriptor.buffers.length is ≤ device.[[device]].[[limits]].maxVertexBuffers.
* Each vertexBuffer layout descriptor in the list descriptor.buffers passes validating GPUVertexBufferLayout(device, vertexBuffer, descriptor)
* The sum of vertexBuffer.attributes.length, over every vertexBuffer in descriptor.buffers, is ≤ device.[[device]].[[limits]].maxVertexAttributes.
* Each attrib in the union of all GPUVertexAttribute across descriptor.buffers has a distinct attrib.shaderLocation value.
