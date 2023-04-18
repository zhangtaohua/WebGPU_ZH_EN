WebGPU

W3C Working Draft, 13 April 2023


More details about this document

This version:

https://www.w3.org/TR/2023/WD-webgpu-20230413/

---
---

# 22\. 错误和调度

## 22.1. 致命错误

```
enum GPUDeviceLostReason {
    "unknown",
    "destroyed"
};

[Exposed=(Window, DedicatedWorker), SecureContext]
interface GPUDeviceLostInfo {
    readonly attribute GPUDeviceLostReason reason;
    readonly attribute DOMString message;
};

partial interface GPUDevice {
    readonly attribute Promise<GPUDeviceLostInfo> lost;
};
```

---

22\. Errors & Debugging

22.1. Fatal Errors

```
enum GPUDeviceLostReason {
    "unknown",
    "destroyed"
};

[Exposed=(Window, DedicatedWorker), SecureContext]
interface GPUDeviceLostInfo {
    readonly attribute GPUDeviceLostReason reason;
    readonly attribute DOMString message;
};

partial interface GPUDevice {
    readonly attribute Promise<GPUDeviceLostInfo> lost;
};
```

---
---

GPUDevice 有以下附加属性：

**lost，类型为： Promise<GPUDeviceLostInfo>, 只读**         
一个有后备插槽属性(attribute)，持有一个与设备一起创建的Promise，在设备的生命周期内保持 pending 状态，然后在设备丢失时被设置为 resolved 状态。

在初始化时，它被设置为一个新的 Promise。

---

GPUDevice has the following additional attributes:

**lost, of type Promise\<GPUDeviceLostInfo\>, readonly**    
A slot-backed attribute holding a promise which is created with the device, remains pending for the lifetime of the device, then resolves when the device is lost.

Upon initialization, it is set to a new promise.

---
---

22.2. GPU 错误

```
[Exposed=(Window, DedicatedWorker), SecureContext]
interface GPUError {
    readonly attribute DOMString message;
};
```

---

22.2. GPUError

```
[Exposed=(Window, DedicatedWorker), SecureContext]
interface GPUError {
    readonly attribute DOMString message;
};
```

---
---

GPUError 是从 popErrorScope() 和 uncapturederror 事件浮现的所有错误的基础接口。

错误必须只因为操作而产生，这此操作明确说明了在其各自的算法中可能产生错误的条件，以及所产生的错误的子类型。

**注意：** GPUError 在本规范的未来版本中可能会定义新的子类型。应用程序应该处理这种可能性，尽可能的只使用错误的消息，并专门使用 instanceof。当需要对错误进行序列化处理时，请使用 error.constructor.name（例如，将其转化为JSON，用于调试报告）。

---

GPUError is the base interface for all errors surfaced from popErrorScope() and the uncapturederror event.

Errors must only be generated for operations that explicitly state the conditions one may be generated under in their respective algorithms, and the subtype of error that is generated.

**NOTE:** GPUError may gain new subtypes in future versions of this spec. Applications should handle this possibility, using only the error’s message when possible, and specializing using instanceof. Use error.constructor.name when it’s necessary to serialize an error (e.g. into JSON, for a debug report).

---
---

GPUError有以下属性(attribute)：

**message，类型为： DOMString，只读**       
一个人类可读的、可本地化的文本消息，提供了发生此错误的信息。

**注意：** 这个消息通常是为应用程序开发人员调试他们的应用程序和捕捉调试报告的信息而准备的，而不是浮现给最终用户。

**注意：** 用户代理不应该在这个消息中包括潜在的机器可理解的细节，如"内存不足"的可用系统内存情况或其他关于内存耗尽的条件的细节。

**注意：** 该消息应遵循语言和方向性信息的最佳实践。这包括利用未来可能出现的关于报告性的字符串语言和方向性元数据的任何标准。

**编辑：** 在写这篇文章的时候，没有任何 语言/方向性的建议 可以提供与传统 API 的兼容性和一致性，但是当有的时候，要正式采用。

---

GPUError has the following attributes:

**message, of type DOMString, readonly**        
A human-readable, localizable text message providing information about the error that occurred.

**NOTE:** This message is generally intended for application developers to debug their applications and capture information for debug reports, not to be surfaced to end-users.

**NOTE:** User agents should not include potentially machine-parsable details in this message, such as free system memory on "out-of-memory" or other details about the conditions under which memory was exhausted.

**NOTE:** The message should follow the best practices for language and direction information. This includes making use of any future standards which may emerge regarding the reporting of string language and direction metadata.

Editorial: At the time of this writing, no language/direction recommendation is available that provides compatibility and consistency with legacy APIs, but when there is, adopt it formally.

---
---

```
[Exposed=(Window, DedicatedWorker), SecureContext]
interface GPUValidationError : GPUError {
    constructor(DOMString message);
};
```

GPUValidationError 是 GPUError 的一个子类型，表示一个操作没有满足所有验证要求。验证错误总是指示应用程序的错误，假设使用相同的[[功能]]和[[限制]]，也希望在所有设备上都以同样的方式失败。

要为 GPUDevice 设备生成一个验证错误，请运行以下步骤：

**内容时间轴步骤：**        
1. 让 error 成为一个新的 GPUValidationError，并带有适当的错误信息。
2. 将 error 错误 派发给设备。

---

```
[Exposed=(Window, DedicatedWorker), SecureContext]
interface GPUValidationError : GPUError {
    constructor(DOMString message);
};
```

GPUValidationError is a subtype of GPUError which indicates that an operation did not satisfy all validation requirements. Validation errors are always indicative of an application error, and is expected to fail the same way across all devices assuming the same [[features]] and [[limits]] are in use.

**To generate a validation error for GPUDevice device, run the following steps:**   
Content timeline steps:
1. Let error be a new GPUValidationError with an appropriate error message.
2. Dispatch error error to device.

---
---

```
[Exposed=(Window, DedicatedWorker), SecureContext]
interface GPUOutOfMemoryError : GPUError {
    constructor(DOMString message);
};
```

GPUOutOfMemoryError 是 GPUError 的一个子类型，表示没有足够的可用内存来完成请求的操作。如果以较低的内存需求（如使用较小的纹理尺寸）再次尝试，或者其他资源使用者的内存首先被释放了，该操作可能会成功。

要为 GPUDevice 设备生成一个内存不足的错误，请运行以下步骤：       

**内容时间轴步骤：**        
1. 让 error 成为一个新的 GPUOutOfMemoryError， 并带有适当的错误信息。
2. 将 error 错误 派发给设备。

---

```
[Exposed=(Window, DedicatedWorker), SecureContext]
interface GPUOutOfMemoryError : GPUError {
    constructor(DOMString message);
};
```

GPUOutOfMemoryError is a subtype of GPUError which indicates that there was not enough free memory to complete the requested operation. The operation may succeed if attempted again with a lower memory requirement (like using smaller texture dimensions), or if memory used by other resources is released first.

**To generate an out-of-memory error for GPUDevice device, run the following steps:**
Content timeline steps:
1. Let error be a new GPUOutOfMemoryError with an appropriate error message.
2. Dispatch error error to device.

---
---

```
[Exposed=(Window, DedicatedWorker), SecureContext]
interface GPUInternalError : GPUError {
    constructor(DOMString message);
};
```

GPUInternalError 是 GPUError 的一个子类型，它表示即使所有的验证要求都得到满足，但由于系统或具体实现的特定原因而导致操作失败。例如，该操作可能超出了具体实现的能力，其操作方式不容易被支持的限制所捕获。同样的操作在其他设备或不同的情况下可能会成功。

要为GPUDevice设备生成一个内部错误，请运行以下步骤：

**内容时间轴步骤：**        
1. 让 error 成为一个新的 GPUInternalError， 并带有适当的错误信息。
2. 将 error 错误 派发给设备。

---

```
[Exposed=(Window, DedicatedWorker), SecureContext]
interface GPUInternalError : GPUError {
    constructor(DOMString message);
};
```

GPUInternalError is a subtype of GPUError which indicates than an operation failed for a system or implementation-specific reason even when all validation requirements have been satisfied. For example, the operation may exceed the capabilities of the implementation in a way not easily captured by the supported limits. The same operation may succeed on other devices or under difference circumstances.

**To generate an internal error for GPUDevice device, run the following steps:**
Content timeline steps:
1. Let error be a new GPUInternalError with an appropriate error message.
2. Dispatch error error to device.
---
---

## 22.3. 错误作用域

GPU 错误作用域捕获了 GPUErrprs 产生时当前的GPU 错误作用域。错误作用域用于隔离一组在 WebGPU 调用中发生的错误，通常用于调试目的或使操作更具容错性。

GPU错误作用域有以下内部插槽：

**[[errors]]，类型为： list\<GPUError\>，最初为[] 。**              
在当前 GPU 错误作用域观察到的 GPUErrors（如果有的话）。

[[filter]], 类型为： GPUErrorFilter
决定这个 GPU 错误作用域所能观察到的 GPUError 的类型。

---

22.3. Error Scopes

A GPU error scope captures GPUErrors that were generated while the GPU error scope was current. Error scopes are used to isolate errors that occur within a set of WebGPU calls, typically for debugging purposes or to make an operation more fault tolerant.

GPU error scope has the following internal slots:

**[[errors]], of type list\<GPUError\>, initially []**      
The GPUErrors, if any, observed while the GPU error scope was current.

**[[filter]], of type GPUErrorFilter**      
Determines what type of GPUError this GPU error scope observes.

---
---

```
enum GPUErrorFilter {
    "validation",
    "out-of-memory",
    "internal"
};

partial interface GPUDevice {
    undefined pushErrorScope(GPUErrorFilter filter);
    Promise<GPUError?> popErrorScope();
};
```
---

```
enum GPUErrorFilter {
    "validation",
    "out-of-memory",
    "internal"
};

partial interface GPUDevice {
    undefined pushErrorScope(GPUErrorFilter filter);
    Promise<GPUError?> popErrorScope();
};
```

---
---

GPUErrorFilter 定义了在调用 pushErrorScope() 时应该捕获的错误类型：

**"validation"**            
表示该错误作用域将捕获 GPUValidationError 类型的错误。

**"Out-of-memory"**         
表示该错误作用域将捕获 GPUOutOfMemoryError 类型的错误。

**"internal"**          
表示该错误作用域将捕获 GPUInternalError 类型的错误。

GPUDevice 有以下内部插槽：

**[[errorScopeStack]]，类型为： stack<GPU错误作用域>。**                
已推送到 GPUDevice 的 GPU错误作用域的堆栈。

---

GPUErrorFilter defines the type of errors that should be caught when calling pushErrorScope():

**"validation"**        
Indicates that the error scope will catch a GPUValidationError.

**"out-of-memory"**       
Indicates that the error scope will catch a GPUOutOfMemoryError.

**"internal"**      
Indicates that the error scope will catch a GPUInternalError.

GPUDevice has the following internal slots:

**[[errorScopeStack]], of type stack\<GPU error scope\>**   
A stack of GPU error scopes that have been pushed to the GPUDevice.

---
---

GPUError 错误 和 GPUDevice 设备的当前错误作用域是通过向 设备的 设备时间线发起以下步骤来确定的：

**设备时间线的步骤：**      
1. 如果错误是以下类型的实例：       
    **GPUValidationError**      
    让类型为 "validation"。       
    **GPUOutOfMemoryError**          
    让类型为 "out-of-memory"。          
    **GPUInternalError**                 
    让类型为 "internal"。              
2. 让 scope 是 device.[[errorScopeStack]] 的最后一项。
3. 当 scope 不是未定义时：
    1. 如果 scope.[[filter]] 是包含类型的，则返回 scope。
    2. 将 scope 设置为 device.[[errorScopeStack]] 的前一项。
4. 返回 undefined。

---

The current error scope for a GPUError error and GPUDevice device is determined by issuing the following steps to the Device timeline of device:

Device timeline steps:
1. If error is an instance of:      
    **GPUValidationError**      
    Let type be "validation".     
    **GPUOutOfMemoryError**     
    Let type be "out-of-memory".        
    **GPUInternalError**      
    Let type be "internal".
2. Let scope be the last item of device.[[errorScopeStack]].
3. While scope is not undefined:
    1. If scope.[[filter]] is type, return scope.
    2. Set scope to the previous item of device.[[errorScopeStack]].
4. Return undefined.

---
---

要在GPUDevice 设备上 调度一个错误的 GPUError 错误，请在设备的时间线上运行以下步骤：

**设备时间线的步骤：**
1. 让 scope 是 错误和设备的 当前错误作用域。
2. 如果 scope 不是未定义的：
    1. 将 错误附加到 scope.[[errors]]。
    2. 返回。
3. 否则向内容时间线发起以下步骤：

**内容时间线的步骤：**  
1. 如果用户代理选择，为 GPUDevice 设备 排入一个全局任务，步骤如下：
    1. 在 设备 上发起一个名为 "uncapturederror" 的 GPUUncapturedErrorEvent，其错误为当前错误信息。

**注意：** 如果（也只有在如果）没有注册 uncapturederror 处理程序的情况下，用户代理应将未捕获的错误浮现给开发者，例如在浏览器的开发者控制台中发出警告。

**注意：** 用户代理可以选择节制或限制 GPUDevice 可以引发的 GPUUncapturedErrorEvents 的数量，以防止过多的错误处理或记录，从而影响到性能。

---

To dispatch an error GPUError error on GPUDevice device, run the following steps on the Device timeline of device:

Device timeline steps:
1. Let scope be the current error scope for error and device.
2. If scope is not undefined:
    1. Append error to scope.[[errors]].
    2. Return.
3. Otherwise issue the following steps to the Content timeline:

Content timeline steps:
1. If the user agent chooses, queue a global task for GPUDevice device with the following steps:
    1. Fire a GPUUncapturedErrorEvent named "uncapturederror" on device, with an error of error.        

**NOTE:** If (and only if) there are no uncapturederror handlers are registered, user agents should surface uncaptured errors to developers, for example as warnings in the browser’s developer console.

**NOTE:** The user agent may choose to throttle or limit the number of GPUUncapturedErrorEvents that a GPUDevice can raise to prevent an excessive amount of error handling or logging from impacting performance.

---
---

pushErrorScope(filter)

将一个新的 GPU错误作用域推送到实例的this 的 [[errorScopeStack]]上。

**调用于：**GPUDevice 实例自身。

**参数：**

| Parameter | Type | Nullable | Optional | Description |
| --- | --- |  --- |  --- |  --- | 
| filter | GPUErrorFilter | ✘ | ✘ | 观察此错误作用域内的哪一类错误。 |

**返回值：** undefined

**上下文时间线步骤：**
1. 在实例 this 的设备时间轴上发起后续的步骤。

---
**pushErrorScope(filter)**          

Pushes a new GPU error scope onto the [[errorScopeStack]] for this.

**Called on:** GPUDevice this.

**Arguments:**

| Parameter | Type | Nullable | Optional | Description |
| --- | --- |  --- |  --- |  --- | 
| filter | GPUErrorFilter | ✘ | ✘ | Which class of errors this error scope observes. |

**Returns:** undefined

Content timeline steps:
1. Issue the subsequent steps on the Device timeline of this.

---
---

**设备的时间轴步骤：**
1. 让 scope 是一个新的 GPU错误作用域。
2. 将 scope.[[filter]] 设为 filter。
3. 把 scope 推到 this.[[errorScopeStack]]。

---

Device timeline steps:
1. Let scope be a new GPU error scope.
2. Set scope.[[filter]] to filter.
3. Push scope onto this.[[errorScopeStack]].

---
---

popErrorScope()

从 [[errorScopeStack]] 中弹出一个GPU错误作用域，并解析为此错误作用域所观察到的任何GPUError，如果没有，则为空。

我们不保证 Promise 所解析的顺序。

**调用于：** GPUDevice 实例自身。

**返回值：** Promise\<GPUError?\>。

**上下文时间线步骤：**
1. 让 contentTimeline 成为当前的上下文时间线。
2. 让 promise 是一个新的 Promise 对象。
3. 在实例 this 的设备时间线上发起检查步骤。
4. 返回 promise。

---

**popErrorScope()**
Pops a GPU error scope off the [[errorScopeStack]] for this and resolves to any GPUError observed by the error scope, or null if none.

There is no guarantee of the ordering of promise resolution.

**Called on:** GPUDevice this.

**Returns:** Promise\<GPUError?\>

Content timeline steps:
1. Let contentTimeline be the current Content timeline.
2. Let promise be a new promise.
3. Issue the check steps on the Device timeline of this.
4. Return promise.

---
---

**设备时间轴检查步骤：**
1. 如果以下任何一项要求没有得到满足：
    * 实例 this 必须是有效的。
    * this.[[errorScopeStack]].size 必须大于 0。            
  
    然后，在 contentTimeline 上发起以下步骤并返回：

    **上下文时间线的步骤：**            
    1. 以 OperationError 将 promise 设置为 refected 状态。

2. 让 scope 成为从 this.[[errorScopeStack]] 弹出的一个项目的结果。
3. 让 error 是 scope.[[error]] 中的任何一个项目错误，如果没有，则为 null。      
    对于列表中的存在任何两个错误 E1 和 E2 ，如果 E2 是由 E1 引起的，E2 不应该是被选中的那个。

    **注意：** 例如，如果 E1 来自 t = createTexture()，而 E2 来自 t.createView()，因为 t是无效的，E1 应该是首选，因为它对开发人员来说更容易理解出错的原因。由于这两个都是GPUValidationErrors，唯一的区别在于消息字段(message)，而这一字段无论如何是能由人类阅读。

4. 在现在或将来的某个时间点，在 contentTimeline 上发起后续步骤。

    **注意：** 通过允许 popErrorScope() 调用可以以任何顺序返回，作用域观察到的任何错误，本规范允许验证不按顺序完成，只要任何状态的观察是遵守本规范所要求适当位置处进行即可。例如，这就允许具体的实现在执行着色器编译时，该编译仅依赖于非状态输入，可在后台线程上与其他设备时间轴工作并行完成，并在这之后报告此过程中产生的任何错误。

**上下文时间线步骤：**
1. 应用错误来将 promise 设置为 resolved 状态。

---

Device timeline check steps:
1. If any of the following requirements are unmet:
    * this must be valid.
    * this.[[errorScopeStack]].size must be > 0.        
    
    Then issue the following steps on contentTimeline and return:           
    
    Content timeline steps:     
        1. Reject promise with an OperationError.
2. Let scope be the result of popping an item off of this.[[errorScopeStack]].
3. Let error be any one of the items in scope.[[errors]], or null if there are none.        
    For any two errors E1 and E2 in the list, if E2 was caused by E1, E2 should not be the one selected.        

    **NOTE:** For example, if E1 comes from t = createTexture(), and E2 comes from t.createView() because t was invalid, E1 should be be preferred since it will be easier for a developer to understand what went wrong. Since both of these are GPUValidationErrors, the only difference will be in the message field, which is meant only to be read by humans anyway.
4. At an unspecified point now or in the future, issue the subsequent steps on contentTimeline.         
    **NOTE:** By allowing popErrorScope() calls to resolve in any order, with any of the errors observed by the scope, this spec allows validation to complete out of order, as long as any state observations are made at the appropriate point in adherence to this spec. For example, this allows implementations to perform shader compilation, which depends only on non-stateful inputs, to be completed on a background thread in parallel with other device-timeline work, and report any resulting errors later.

Content timeline steps:
1. Resolve promise with error.

---
---

```
// 例子 30 
// 使用 错误作用域来捕获来自 GPUDevice 操作可能失败而产生的验证错误：

gpuDevice.pushErrorScope('validation');

let sampler = gpuDevice.createSampler({
    maxAnisotropy: 0, // Invalid, maxAnisotropy must be at least 1.
});

gpuDevice.popErrorScope().then((error) => {
    if (error) {
        // There was an error creating the sampler, so discard it.
        sampler = null;
        console.error(`An error occured while creating sampler: ${error.message}`);
    }
});
```

---
```
// EXAMPLE 30 
//Using error scopes to capture validation errors from a GPUDevice operation that may fail:

gpuDevice.pushErrorScope('validation');

let sampler = gpuDevice.createSampler({
    maxAnisotropy: 0, // Invalid, maxAnisotropy must be at least 1.
});

gpuDevice.popErrorScope().then((error) => {
    if (error) {
        // There was an error creating the sampler, so discard it.
        sampler = null;
        console.error(`An error occured while creating sampler: ${error.message}`);
    }
});
```

---
---

注意：错误作用域可以根据需要包括许多命令。一个错误作用域所涵盖的命令数量通常与应用程序在发生错误时打算采取的行动相关联。        

比如说：一个错误作用域只包含一个单一资源创建时，例如创建一个纹理或一个缓冲区时，此错误作用域可以用来检测故障，如内存不足的情况，在这种情况下，应用程序可以尝试释放一些资源并尝试再次分配。

然而，错误作用域并不能识别哪个命令失败。因此，例如，将加载模型时，执行的所有命令都包裹在一个错误作用域中，就不能提供足够的粒度来确定问题是否是由于内存不足的限制而引起的。因此，释放资源通常不是对该作用域失败的有效响应。一个更合适的操作应该是允许应用程序返回到另一个不同的模型，或者产生一个警告说明模型不能被加载的原因。如果需要对内存限制做出响应，分配内存的操作可以被包裹在一个较小的嵌套错误作用域中。

---

**Note:** Error scopes can encompass as many commands as needed. The number of commands an error scope covers will generally be correlated to what sort of action the application intends to take in response to an error occuring.
For example: An error scope that only contains the creation of a single resource, such as a texture or buffer, can be used to detect failures such as out of memory conditions, in which case the application may try freeing some resources and trying the allocation again.

Error scopes do not identify which command failed, however. So, for instance, wrapping all the commands executed while loading a model in a single error scope will not offer enough granularity to determine if the issue was due to memory constraints. As a result freeing resources would usually not be a productive response to a failure of that scope. A more appropriate response would be to allow the application to fall back to a different model or produce a warning that the model could not be loaded. If responding to memory constraints is desired, the operations allocating memory can always be wrapped in a smaller nested error scope.

---
---

## 22.4.遥测

当一个 GPUError 被生成，但没有被任何 GPU错误作用域观察到时，用户代理可以使用 GPUUncapturedErrorEvent 在 GPUDevice 发起一个名为 uncapturederror 的事件。

**注意：** uncapturederror 事件 旨在用于遥测 和报告意外的错误。它们可能不会对所有未捕获的错误都进行派发（例如，可能会对要浮现的错误数量有限制），并且不应该用于处理在应用程序正常运行期间可能发生的已知错误。在这些情况下，最好使用 pushErrorScope() 和 popErrorScope() 。


---

22.4. Telemetry

When a GPUError is generated that is not observed by any GPU error scope, the user agent may fire an event named uncapturederror at a GPUDevice using GPUUncapturedErrorEvent.

**NOTE:** uncapturederror events are intended to be used for telemetry and reporting unexpected errors. They may not be dispatched for all uncaptured errors (for example, there may be a limit on the number of errors surfaced), and should not be used for handling known error cases that may occur during normal operation of an application. Prefer using pushErrorScope() and popErrorScope() in those cases.

---
---

```
[Exposed=(Window, DedicatedWorker), SecureContext]
interface GPUUncapturedErrorEvent : Event {
    constructor(
        DOMString type,
        GPUUncapturedErrorEventInit gpuUncapturedErrorEventInitDict
    );
    [SameObject] readonly attribute GPUError error;
};

dictionary GPUUncapturedErrorEventInit : EventInit {
    required GPUError error;
};
```

GPUUncapturedErrorEvent 具有以下属性(attributes)：

**error, 类型为： GPUError, 只读**          
一个后备插槽属性，持有一个代表未捕获的错误的对象。这与 popErrorScope() 返回的错误具有相同的类型。

---

```
[Exposed=(Window, DedicatedWorker), SecureContext]
interface GPUUncapturedErrorEvent : Event {
    constructor(
        DOMString type,
        GPUUncapturedErrorEventInit gpuUncapturedErrorEventInitDict
    );
    [SameObject] readonly attribute GPUError error;
};

dictionary GPUUncapturedErrorEventInit : EventInit {
    required GPUError error;
};
```

GPUUncapturedErrorEvent has the following attributes:

**error, of type GPUError, readonly**           
A slot-backed attribute holding an object representing the error that was uncaptured. This has the same type as errors returned by popErrorScope().

---
---

```
partial interface GPUDevice {
    [Exposed=(Window, DedicatedWorker)]
    attribute EventHandler onuncapturederror;
};
```

GPUDevice 有以下属性：

**onuncapturederror, 类型为： EventHandler**                
uncapturederror 事件类型的 一个事件处理程序 的 IDL属性。

---

```
partial interface GPUDevice {
    [Exposed=(Window, DedicatedWorker)]
    attribute EventHandler onuncapturederror;
};
```

GPUDevice has the following attributes:

**onuncapturederror, of type EventHandler**         
An event handler IDL attribute for the uncapturederror event type.

---
---

```
// 例子 31
// 监听来自 GPUDevice 未捕获的错误：:

gpuDevice.addEventListener('uncapturederror', (event) => {
    // Re-surface the error, because adding an event listener may silence console logs.
    console.error('A WebGPU error was not captured:', event.error);

    myEngineDebugReport.uncapturedErrors.push({
        type: event.error.constructor.name,
        message: event.error.message,
    });
});
```

---

```
// EXAMPLE 31
// Listening for uncaptured errors from a GPUDevice:
gpuDevice.addEventListener('uncapturederror', (event) => {
    // Re-surface the error, because adding an event listener may silence console logs.
    console.error('A WebGPU error was not captured:', event.error);

    myEngineDebugReport.uncapturedErrors.push({
        type: event.error.constructor.name,
        message: event.error.message,
    });
});
```
