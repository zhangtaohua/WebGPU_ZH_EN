# WebGPU

W3C Working Draft, 24 March 2023

More details about this document

This version:

https://www.w3.org/TR/2023/WD-webgpu-20230324/


# 9\. 着色器模块

## 9.1. GPUShaderModule

```
[Exposed=(Window, DedicatedWorker), SecureContext]
interface GPUShaderModule {
    Promise<GPUCompilationInfo> getCompilationInfo();
};
GPUShaderModule includes GPUObjectBase;
```

GPUShaderModule 是对一个内部着色器模块对象的引用。

---

9\. Shader Modules

9.1. GPUShaderModule

```
[Exposed=(Window, DedicatedWorker), SecureContext]
interface GPUShaderModule {
    Promise<GPUCompilationInfo> getCompilationInfo();
};
GPUShaderModule includes GPUObjectBase;
```

GPUShaderModule is a reference to an internal shader module object.

---
---

### 9.1.1. 着色器模块创建

```
dictionary GPUShaderModuleDescriptor : GPUObjectDescriptorBase {
    required USVString code;
    object sourceMap;
    record<USVString, GPUShaderModuleCompilationHint> hints;
};
```

代码，类型为：USVString
着色器模块的WGSL源代码。

sourceMap，类型为： object
如果定义了，可能会被解释为 source-map-v3 格式。

源码图是可选的，但作为一种标准化的方式来支持开发工具的集成，如源语言调试[SourceMap]。源地图中的WGSL名称（标识符）遵循WGSL标识符中定义的规则。

---

9.1.1. Shader Module Creation

```
dictionary GPUShaderModuleDescriptor : GPUObjectDescriptorBase {
    required USVString code;
    object sourceMap;
    record<USVString, GPUShaderModuleCompilationHint> hints;
};
```

**code, of type USVString**   
The WGSL source code for the shader module.

**sourceMap, of type object**     
If defined MAY be interpreted as a source-map-v3 format.

Source maps are optional, but serve as a standardized way to support dev-tool integration such as source-language debugging [SourceMap]. WGSL names (identifiers) in source maps follow the rules defined in WGSL identifier comparison.

---
---

hints，类型为： record\<USVString, GPUShaderModuleCompilationHint\>。
如果定义了，则将着色器的入口点的名称映射到 一个 GPUShaderModuleCompilationHint。这些GPUShaderModuleCompilationHint 都不会被验证。具体的实现应该使用 GPUShaderModuleCompilationHint 中的任何信息，在 createShaderModule() 中尽可能地进行编译工作。入口点名称命名遵循 WGSL 标识符中定义的规则。

**注意：** 除了性能之外，在提示中提供信息并没有任何可观察到的效果。因为一个着色器模块可以容纳多个入口点，并且可以从一个着色器模块中创建多个管道，对于一个具体的实现来说，在 createShaderModule() 中可以一次完成尽可能多的编译工作，这样可能会更有效，而不是多次调用 createComputePipeline() / createRenderPipeline()。

***译注，因为WGSL写的代码是文本，是要通过调用此函数编译后再给GPU执行的，这一WebGL的着色器语言的工作方式类似，所以这里是完成尽可能多的编译工作***

---

**hints, of type record\<USVString, GPUShaderModuleCompilationHint\>**    
If defined maps an entry point name from the shader to a GPUShaderModuleCompilationHint. No validation is performed with any of these GPUShaderModuleCompilationHint. Implementations should use any information present in the GPUShaderModuleCompilationHint to perform as much compilation as is possible within createShaderModule(). Entry point names follow the rules defined in WGSL identifier comparison.

**NOTE:** Supplying information in hints does not have any observable effect, other than performance. Because a single shader module can hold multiple entry points, and multiple pipelines can be created from a single shader module, it can be more performant for an implementation to do as much compilation as possible once in createShaderModule() rather than multiple times in the multiple calls to createComputePipeline() / createRenderPipeline().

---
---

**createShaderModule(descriptor)** 函数
用于创建一个 GPUShaderModule。

调用于： GPUDevice this 实例自身。

参数：

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| `descriptor` | `GPUShaderModuleDescriptor` | ✘ | ✔ |  创建 GPUShaderModule 的描述符. |

返回值： GPUShaderModule。

---
**createShaderModule(descriptor)**

Creates a GPUShaderModule.


Called on: GPUDevice this.

Arguments:

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| `descriptor` | `GPUShaderModuleDescriptor` | ✘ | ✔ | Description of the GPUShaderModule to create. |

Returns: GPUShaderModule

---
---

**上下文时间轴步骤：**
1. 让 sm 成为一个新的 GPUShaderModule 对象。
2. 在这个实例的设备时间线上发起初始化步骤。
3. 返回sm。

---
**Content timeline steps:**   
1. Let sm be a new GPUShaderModule object.
2. Issue the initialization steps on the Device timeline of this.
3. Return sm.

---
---

设备时间线初始化步骤：
1. 让结果是 用WGSL源描述符中code 字符串 编译创建的着色器模块的结果。
2. 如果以下任何要求未得到满足，则生成一个验证错误，使 sm 无效，并返回。
    * 实例this必须是有效的。
    * 着色器创建程序的结果不能是错误的。

>> ISSUE 12 这里是否应该允许内部错误（未分类的错误），还是应该强制将错误 推入到管道创建进行错误提示？[问题#gpuweb/gpuweb#2308] 。

>> ISSUE 13 描述剩余的 createShaderModule() 验证和算法步骤。

---

**Device timeline initialization steps:**   
1. Let result be the result of shader module creation with the WGSL source descriptor.code.
2. If any of the following requirements are unmet, generate a validation error, make sm invalid, and return.
    * this must be valid.
    * result must not be a shader-creation program error.

>> ISSUE 12 Should internal errors (uncategorized errors) be allowed here or should we force them to be deferred to pipeline creation? [Issue #gpuweb/gpuweb#2308]

>> ISSUE 13 Describe remaining createShaderModule() validation and algorithm steps.

---
---

> **注意：** 用户代理在这里产生的验证错误的消息文本中，不应包括详细的编译器错误信息或着色器程序文本：这些细节可以通过 getCompilationInfo() 访问。用户代理应该向开发者提供人类可读的、格式化的错误细节，以方便开发者调试错误（例如，在浏览器开发者控制台中输出警告，可扩展为显示完整的着色器源信息）。
>    
> 由于着色器编译错误在生产应用中很少见，用户代理可以选择将其呈现给开发者，而不考虑错误处理（GPU错误范围或uncapturederror事件处理程序），例如，作为一个可扩展警告。如果不是这样，他们应该提供另一种人类可读的错误细节呈现方式，以便让开发者访问，例如通过添加一个复选框来无条件地显示错误，或在将 GPUCompilationInfo 对象记录到控制台，以便显示人类可读的细节。

---
> **Note:** User agents should not include detailed compiler error messages or shader text in the message text of validation errors arising here: these details are accessible via getCompilationInfo(). User agents should surface human-readable, formatted error details to developers for easier debugging (for example as a warning in the browser developer console, expandable to show full shader source).
>   
> As shader compilation errors should be rare in production applications, user agents could choose to surface them to developers regardless of error handling (GPU error scopes or uncapturederror event handlers), e.g. as an expandable warning. If not, they should provide and document another way for developers to access human-readable error details, for example by adding a checkbox to show errors unconditionally, or by showing human-readable details when logging a GPUCompilationInfo object to the console.

```
EXAMPLE 20
// 从 WGSL 代码中创建一个 GPUShaderModule：
// Create a GPUShaderModule from WGSL code:

// 一个简单的顶点和片段着色器程序，用于将视口填充为红色。
// A simple vertex and fragment shader pair that will fill the viewport with red.

const shaderSource = `
    var<private> pos : array<vec2<f32>, 3> = array<vec2<f32>, 3>(
        vec2(-1.0, -1.0), vec2(-1.0, 3.0), vec2(3.0, -1.0));

    @vertex
    fn vertexMain(@builtin(vertex_index) vertexIndex : u32) -> @builtin(position) vec4<f32> {
        return vec4(pos[vertexIndex], 1.0, 1.0);
    }

    @fragment
    fn fragmentMain() -> @location(0) vec4<f32> {
        return vec4(1.0, 0.0, 0.0, 1.0);
    }
`;

const shaderModule = gpuDevice.createShaderModule({
    code: shaderSource,
});

```

---
---

#### 9.1.1.1.  着色器模块编译提示

着色器模块编译提示是可选的，附加信息表明一个给定的 GPUShaderModule 入口点打算在未来如何使用。对于某些具体的实现来说，这些信息可能在早期有助于编译着色器模块，从而有可能提高性能。

```
dictionary GPUShaderModuleCompilationHint {
    (GPUPipelineLayout or GPUAutoLayoutMode) layout;
};
```

---

9.1.1.1. Shader Module Compilation Hints

Shader module compilation hints are optional, additional information indicating how a given GPUShaderModule entry point is intended to be used in the future. For some implementations this information may aid in compiling the shader module earlier, potentially increasing performance.

```
dictionary GPUShaderModuleCompilationHint {
    (GPUPipelineLayout or GPUAutoLayoutMode) layout;
};
```
---
---

**layout，类型为（GPUPipelineLayout或GPUAutoLayoutMode）。**    
一个  GPUPipelineLayout，GPUShaderModule可以在未来的 createComputePipeline()或createRenderPipeline()调用中使用。如果设置为 "auto"，该布局将会成为 使用和此提示相关的入口点函数的 默认管道布局。

**注意：** 如果可能的话，作者应该向 createShaderModule() 和 createComputePipeline()/createRenderPipeline()提供相同的信息。

如果作者在调用 createShaderModule() 时无法提供提示信息，他们通常不应该延迟调用 createShaderModule()；而是应该直接省略提示中的 或者 GPUShaderModuleCompilationHint 中的未知信息。省略这些信息可能会导致编译工作被推迟到createComputePipeline()/createRenderPipeline() 函数调用。

如果作者不确定传递给 createShaderModule() 的提示信息是否会与后来传递给 createComputePipeline()/createRenderPipeline() 的相同模块的信息相匹配，他们应该避免将该信息传递给 createShaderModule()，因为将不匹配的信息传递给 createShaderModule() 可能会导致产生不必要的编译损耗。

---

**layout, of type (GPUPipelineLayout or GPUAutoLayoutMode)**    
A GPUPipelineLayout that the GPUShaderModule may be used with in a future createComputePipeline() or createRenderPipeline() call. If set to "auto" the layout will be the default pipeline layout for the entry point associated with this hint will be used.

> **Note:** If possible, authors should be supplying the same information to createShaderModule() and createComputePipeline() / createRenderPipeline().
> 
> If an author is unable to provide hint information at the time of calling createShaderModule(), they should usually not delay calling createShaderModule(); but should instead just omit the unknown information from hints or GPUShaderModuleCompilationHint. Omitting this information may cause compilation to be deferred to createComputePipeline() / createRenderPipeline().
> 
> If an author is not confident that the hint information passed to createShaderModule() will match the information later passed to createComputePipeline() / createRenderPipeline() with that same module, they should avoid passing that information to createShaderModule(), as passing mismatched information to createShaderModule() may cause unnecessary compilations to occur.

---
---
#### 9.1.2. 着色器模块编译信息

```
enum GPUCompilationMessageType {
    "error",
    "warning",
    "info"
};
```


```
[Exposed=(Window, DedicatedWorker), Serializable, SecureContext]
interface GPUCompilationMessage {
    readonly attribute DOMString message;
    readonly attribute GPUCompilationMessageType type;
    readonly attribute unsigned long long lineNum;
    readonly attribute unsigned long long linePos;
    readonly attribute unsigned long long offset;
    readonly attribute unsigned long long length;
};

[Exposed=(Window, DedicatedWorker), Serializable, SecureContext]
interface GPUCompilationInfo {
    readonly attribute FrozenArray<GPUCompilationMessage> messages;
};
```

---
9.1.2. Shader Module Compilation Information

```
enum GPUCompilationMessageType {
    "error",
    "warning",
    "info"
};
```

```
[Exposed=(Window, DedicatedWorker), Serializable, SecureContext]
interface GPUCompilationMessage {
    readonly attribute DOMString message;
    readonly attribute GPUCompilationMessageType type;
    readonly attribute unsigned long long lineNum;
    readonly attribute unsigned long long linePos;
    readonly attribute unsigned long long offset;
    readonly attribute unsigned long long length;
};

[Exposed=(Window, DedicatedWorker), Serializable, SecureContext]
interface GPUCompilationInfo {
    readonly attribute FrozenArray<GPUCompilationMessage> messages;
};
```

---
---

GPUCompilationMessage 是由 GPUShaderModule 编译器生成的信息、警告或错误信息。这些信息是人类可读的，旨在帮助开发者诊断其着色器代码的问题。每条信息可以对应着色器代码中的一个点，也可以是着色器代码的一个子串，或者根本就不对应代码中的任何特定点。

---

A GPUCompilationMessage is an informational, warning, or error message generated by the GPUShaderModule compiler. The messages are intended to be human readable to help developers diagnose issues with their shader code. Each message may correspond to either a single point in the shader code, a substring of the shader code, or may not correspond to any specific point in the code at all.

---
---

GPUCompilationMessage 具有以下属性：    
**message，类型为：DOMString，只读**
该编译消息的可读性、本地化的文本信息。

**注意：**  该消息应遵循语言和错误指示信息的最佳实践。这包括利用未来可能出现的标准中的，报告性的字符串语言信息和元数据信息。

**编辑的**：在写这篇规范时，没有任何语言/方向性的建议可以提供与传统API的兼容性和一致性，但是当有的时候，要正式采用。

**type, 类型为： GPUCompilationMessageType, 只读**   
表示消息的严重程度。

如果类型为"error"，则对应于着色器创建时的错误。

---

GPUCompilationMessage has the following attributes:

**message, of type DOMString, readonly**    
The human-readable, localizable text for this compilation message.

**NOTE:** The message should follow the best practices for language and direction information. This includes making use of any future standards which may emerge regarding the reporting of string language and direction metadata.

**Editorial:** At the time of this writing, no language/direction recommendation is available that provides compatibility and consistency with legacy APIs, but when there is, adopt it formally.

**type, of type GPUCompilationMessageType, readonly**   
The severity level of the message.

If the type is "error", it corresponds to a shader-creation error.

---
---

**lineNum，类型为： 无符号长长整型，只读**   
消息所对应的着色器代码中的行数。该值是以1为基数的，比如lineNum为1表示着色器代码的第一行。行数以换行符为界。

如果消息对应的是一个子串，则指向子串开始的那一行。如果消息不对应着色器代码中的任何特定行，则必须为0。

**linePos，类型为unsigned long long，只读。**   
表示偏移量，以 UTF-16 代码为单位，从着色器代码的 lineNum 开始， 到信息所对应的特定点 或信息所对应的子串开始的位置。该值是以 1 为基数的，如 linePos 为 1 表示该行的第一个代码单位。

如果消息对应的是一个子串，则指向该子串的第一个 UTF-16 代码单元。如果消息不对应着色器代码中的任何特定点，则必须为0。

---

**lineNum, of type unsigned long long, readonly**   
The line number in the shader code the message corresponds to. Value is one-based, such that a lineNum of 1 indicates the first line of the shader code. Lines are delimited by line breaks.

If the message corresponds to a substring this points to the line on which the substring begins. Must be 0 if the message does not correspond to any specific point in the shader code.

**linePos, of type unsigned long long, readonly**   
The offset, in UTF-16 code units, from the beginning of line lineNum of the shader code to the point or beginning of the substring that the message corresponds to. Value is one-based, such that a linePos of 1 indicates the first code unit of the line.

If message corresponds to a substring this points to the first UTF-16 code unit of the substring. Must be 0 if the message does not correspond to any specific point in the shader code.

---
---

**offset，类型为：无符号长长整形，只读**    
从着色器代码的开始位置，到该信息所对应的特定点，或者该信息所对应的子串开始的位置 计算而得的偏移量，以UTF-16代码单位计算。必须引用与 lineNum 和 linePos 相同的位置。如果消息不对应着色器代码中的任何特定点，则必须为0。

**length，类型为：无符号长长整形，只读**    
该消息对应的子串的长度，以 UTF-16 代码为单位。如果消息没有对应的子串，那么length必须是0。

**注意：** GPUCompilationMessage.lineNum 和 GPUCompilationMessage.linePos 是以 1 为基数的，因为它们最常见的用途是打印人类可读的信息，这样，可以与许多文本编辑器中显示的行和列数字相关联。

**注意：** GPUCompilationMessage.offset 和 GPUCompilationMessage.length 适合传递给 substr() 函数，以便检索信息所对应的着色器代码的子串。

---
**offset, of type unsigned long long, readonly**    
The offset from the beginning of the shader code in UTF-16 code units to the point or beginning of the substring that message corresponds to. Must reference the same position as lineNum and linePos. Must be 0 if the message does not correspond to any specific point in the shader code.

**length, of type unsigned long long, readonly**    
The number of UTF-16 code units in the substring that message corresponds to. If the message does not correspond with a substring then length must be 0.

**NOTE:** GPUCompilationMessage.lineNum and GPUCompilationMessage.linePos are one-based since the most common use for them is expected to be printing human readable messages that can be correlated with the line and column numbers shown in many text editors.

**NOTE:** GPUCompilationMessage.offset and GPUCompilationMessage.length are appropriate to pass to substr() in order to retrieve the substring of the shader code the message corresponds to.

---
---

**getCompilationInfo()**

用于返回在 GPUShaderModule 的编译过程中产生的任何信息。

消息的位置、顺序和内容都是由具体的实现定义的。特别是，信息可能不按行数顺序排列。

调用于： GPUShaderModule 实例自身

返回值 Promise\<GPUCompilationInfo\>

---
**getCompilationInfo()**

Returns any messages generated during the GPUShaderModule's compilation.

The locations, order, and contents of messages are implementation-defined. In particular, messages may not be ordered by lineNum.

**Called on:** GPUShaderModule this

**Returns:** Promise\<GPUCompilationInfo\>

---
---

上下文时间线步骤：
1. 让 contentTimeline 成为当前的上下文时间线。
2. 让 promise 是一个新的 Promise 对象。
3. 在这个的设备时间线上发起同步步骤。
4. 返回 promise。

---

**Content timeline steps:**
1. Let contentTimeline be the current Content timeline.
2. Let promise be a new promise.
3. Issue the synchronization steps on the Device timeline of this.
4. Return promise.

---
---

设备时间线的同步步骤：
1. 当设备时间线 被通知，着色器模块的创建已经完成时：
    1. 让消息成为 shader 模块创建过程中所产生的任何 错误、警告或 信息消息的列表。
    2. 在 contentTimeline 上发起后续步骤。

---
**Device timeline synchronization steps:**    
1. When the device timeline becomes informed that shader module creation has completed for this:
    1. Let messages be a list of any errors, warnings, or informational messages generated during shader module creation for this.
    2. Issue the subsequent steps on contentTimeline.

---
---

上下文时间轴上的步骤：
1. 让 info 是一个新的 GPUCompilationInfo。
2. 对于 messages 中的每一条消息：
    1. 让 m 是一个新的 GPUCompilationMessage。
    2. 设置 m.message 为消息的文本。
    3. 如果消息是一个着色器创建错误：   
        将 m.type 设为 "error"。
        
        如果消息是一个警告：    
        将 m.type 设为 "警告"。
        
        否则：  
        将 m.type 设为 "info"。
    4. 如果消息是与着色器代码中的特定子串或位置相关：   
        1. 将 m.lineNum 设置为该消息所指的代码行数，此行数是以 1 为基数的。
        2. 将 m.linePos 设置为消息所指的 m.lineNum 上，以 1  为基数，以 UTF-16代码为单位 的第一个字符数，如果消息所指的是整行，则设置为1。
        3. 将 m.offset 设为从 shader 开始，以此消息所指向的对应的子串或位置开始的数量，以 UTF-16代码点为单位。
        4. 设置 m.length 为此消息所对应的子串的长度，以 UTF-16代码点为单位，如果消息所指向是位置，则设置为0。   
        
        否则：
        1. 设置 m.lineNum 为0。
        2. 设置 m.linePos 为0。
        3. 设置 m.offset 为0。
        4. 设置 m.length 为0。

      5. 将 m 追加到 info.messages 中。
3.  将要返回的Promise 设置为 resolved，其值为：info.message。

---

**Content timeline steps:**
1. Let info be a new GPUCompilationInfo.
2. For each message in messages:
    1. Let m be a new GPUCompilationMessage.
    2. Set m.message to be the text of message.
    3.  If message is a shader-creation error:     
        Set m.type to "error"     

        If message is a warning:    
        Set m.type to "warning"   

        Otherwise:      
        Set m.type to "info"
    4. If message is associated with a specific substring or position within the shader code:
        1. Set m.lineNum to the one-based number of the first line that the message refers to.
        2. Set m.linePos to the one-based number of the first UTF-16 code units on m.lineNum that the message refers to, or 1 if the message refers to the entire line.
        3. Set m.offset to the number of UTF-16 code units from the beginning of the shader to beginning of the substring or position that message refers to.
        4. Set m.length the length of the substring in UTF-16 code units that message refers to, or 0 if message refers to a position

        Otherwise:    
        5. Set m.lineNum to 0. 
        6. Set m.linePos to 0.
        7. Set m.offset to 0.
        8. Set m.length to 0.

    5. Append m to info.messages.

3. Resolve promise with info.
