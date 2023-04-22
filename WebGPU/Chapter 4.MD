WebGPU

W3C Working Draft, 2 February 2023

More details about this document

This version:

https://www.w3.org/TR/2023/WD-webgpu-20230202/

Latest published version:

https://www.w3.org/TR/webgpu/

Editor's Draft:

https://gpuweb.github.io/gpuweb/

History:

https://www.w3.org/standards/history/webgpu

---
---

# 4\. 初始化
> 4\. Initialization

## 4.1. navigator.gpu

> 4.1. navigator.gpu

在 Window 和 DedicatedWorkerGlobalScope 上下文中，可以分别通过 Navigator 和 WorkerNavigator 接口获得一个 GPU 对象，并且，这个GPU 对象通过 navigator.gpu 暴露出去。

> A GPU object is available in the Window and DedicatedWorkerGlobalScope contexts through the Navigator and WorkerNavigator interfaces respectively and is exposed via navigator.gpu:

```
interface mixin NavigatorGPU {
    [SameObject, SecureContext] readonly attribute GPU gpu;
};

//  Navigator 包括 NavigatorGPU。
//  WorkerNavigator 包括 NavigatorGPU。

Navigator includes NavigatorGPU;
WorkerNavigator includes NavigatorGPU;
```
## 4.2. GPU

> 4.2. GPU

GPU 是 WebGPU 的入口点

> GPU is the entry point to WebGPU.

```
[Exposed=(Window, DedicatedWorker), SecureContext]
interface GPU {
    Promise<GPUAdapter?> requestAdapter(optional GPURequestAdapterOptions options = {});
    GPUTextureFormat getPreferredCanvasFormat();
};
```

GPU有以下这些成员方法。

> GPU has the following methods:

**requestAdapter(options)**    
向用户代理请求一个适配器。用户代理选择是否返回一个适配器，如果返回一个适配器，则根据提供的参数选项进行选择性的返回。    
调用于：GPU 的实例上。   

参数。如下
| Parameter | Type | Nullable | Optional | Description |
| -------- | -------- | -------- | -------- | -------- |
| options | GPURequestAdapterOptions | ✘ | ✔ | 用于选择适配器的参数选项 |

> **`requestAdapter(options)`**    
> Requests an adapter from the user agent. The user agent chooses whether to return an adapter, and, if so, chooses according to the provided options.    
> Called on: GPU this.    
> Arguments:

| Parameter | Type | Nullable | Optional | Description |
| -------- | -------- | -------- | -------- | -------- |
| options | GPURequestAdapterOptions | ✘ | ✔ | Criteria used to select the adapter. |

返回值：Promise<GPUAdapter?>    
上下文时间线上的步骤如下：
1. 让 contentTimeline 成为当前的内容时间线。
2. 让 promise 是一个新的promise。
3. 在这个上下文时间线的设备时间线上发起 初始化步骤。    
4. 返回promise对象。

> Returns: Promise<GPUAdapter?>    
> Content timeline steps:    
> 1. Let contentTimeline be the current Content timeline.    
> 2. Let promise be a new promise.
> 3. Issue the initialization steps on the Device timeline of this.
> 4. Return promise.

设备时间线上的初始化步骤：
1. 让适配器为空。
2. 如果用户代理选择返回一个适配器，它应该：
    1. 将适配器设置为一个有效的适配器，根据 §4.2.1 适配器选择中的规则以及用户提供的参数选项标准来进行选择。    
      适配器的支持限制必须遵守 §3.6.2 限制中定义的要求。    
    2. 如果最后返回适配器符合后备适配器的标准，则将适配器.[[fallback]] 属性设为true。
3. 接着在 contentTimeline 上发起后续步骤。

> Device timeline initialization steps:
> 1. Let adapter be null.
> 2. If the user agent chooses to return an adapter, it should:
>    1. Set adapter to a valid adapter, chosen according to the rules in § 4.2.1 Adapter Selection and the criteria in options.    
>    The supported limits of the adapter must adhere to the requirements defined in § 3.6.2 Limits.
>    2.  If adapter meets the criteria of a fallback adapter set adapter.[[fallback]] to true.
> 3. Issue the subsequent steps on contentTimeline.

上下文时间线步骤。
1. 如果适配器不是空的。
    1. promise 的状态设置为成功，其值是 一个新的封装了adapter的 GPUAdapter。
2. 否则，promise 的状态设置为成功，其值是 null。

> Content timeline steps:
> 1. If adapter is not null:    
>    1. Resolve promise with a new GPUAdapter encapsulating adapter.
> 2. Otherwise, Resolve promise with null.

**getPreferredCanvasFormat()**

返回一个最佳的 GPUTextureFormat，用于在此系统上显示8位深度、标准动态范围的内容。必须只返回 "rgba8unorm" 或者 "bgra8unorm"。    
返回的值可以作为格式参数传递给在 GPUCanvasContext 上调用的 configure() 函数，以确保相关canvas 能够有效地显示其内容。    
**注意：** 不显示在屏幕上的画布可能会，也可能不会从使用这种格式中受益。    

> **getPreferredCanvasFormat()** 

> Returns an optimal GPUTextureFormat for displaying 8-bit depth, standard dynamic range content on this system. Must only return "rgba8unorm" or "bgra8unorm".

> The returned value can be passed as the format to configure() calls on a GPUCanvasContext to ensure the associated canvas is able to display its contents efficiently.

> **NOTE:** Canvases which are not displayed to the screen may or may not benefit from using this format.

调用于：GPU 的实例上.       
返回值：GPUTextureFormat    
上下文时间线上的步骤:    
返回 "rgba8unorm" 或者 "bgra8unorm"，取决于哪种格式是 在本系统上显示 WebGPU 画布的最佳可用格式。

> Called on: GPU this.    
> Returns: GPUTextureFormat    
> Content timeline steps:    
> 1. Return either "rgba8unorm" or "bgra8unorm", depending on which format is optimal for displaying WebGPU canvases on this system.


GPU有以下内部插槽。
```
[[previous_returned_adapters]], 其类型为：<adapter>的有序集合。
适配器的集合 是已经通过 requestAdapter() 返回的。他首先被使用，然后在适配器被标记过时时被清除。
```

> GPU has the following internal slots:

```
[[previously_returned_adapters]], of type ordered set<adapter>
The set of adapters that have been returned via requestAdapter(). It is used, then cleared, in mark adapters stale.
```

当系统状态的任何变化可能会影响 任何 requestAdapter() 调用的结果时，用户代理应将适配器标记为过时的，比如说：
* 一个物理适配器被添加/删除（通过插头、驱动更新、TDR等）。
* 系统的电源配置改变了（笔记本电源拔掉了，电源设置改变了，等等）。

> Upon any change in the system’s state that could affect the result of any requestAdapter() call, the user agent should mark adapters stale. For example:
> * A physical adapter is added/removed (via plug, driver update, TDR, etc.)
> * The system’s power configuration has changed (laptop unplugged, power settings changed, etc.)

此外，标记适配器过期任务，在任何时候都可以排队安排。用户代理可以选择这样做，即使在没有系统状态变化的情况下（例如，在最后一次调用 requestDevice() 几秒钟后），这对设计良好的应用程序没有影响，混淆了真正的系统状态变化，并使开发人员更清楚地认识到，在调用 requestDevice() 之前，总是再次调用 requestAdapter() 是非常必要的。


Additionally, mark adapters stale may by scheduled at any time. User agents may choose to do this often even when there has been no system state change (e.g. several seconds after the last call to requestDevice(). This has no effect on well-formed applications, obfuscates real system state changes, and makes developers more aware that calling requestAdapter() again is always necessary before calling requestDevice().

为了标记适配器是过期的：
1. 对于 navigator.gpu.[[previous_returned_adapters]] 中的每个适配器:   
  1. 使 adapter.[[adapter]] 标记为无效。
2. 清空 navigator.gpu.[[previous_returned_adapters]]。

> To mark adapters stale:
> 1. For each adapter in navigator.gpu.[[previously_returned_adapters]]:
>    1. Make adapter.[[adapter]] invalid.
> 2. Empty navigator.gpu.[[previously_returned_adapters]].

```
EXAMPLE 7
// 请求一个没有提示的GPUAdapter。
// Requesting a GPUAdapter with no hints:    

const gpuAdapter = await navigator.gpu.requestAdapter();

```

### 4.2.1.适配器选择
GPURequestAdapterOptions 向用户代理提供提示，指出什么样配置的适配器适合于应用程序的要求。

> 4.2.1. Adapter Selection

> GPURequestAdapterOptions provides hints to the user agent indicating what configuration is suitable for the application.


```
// 代码不译

dictionary GPURequestAdapterOptions {
    GPUPowerPreference powerPreference;
    boolean forceFallbackAdapter = false;
};
```
```
enum GPUPowerPreference {
    "low-power",
    "high-performance"
};
```

GPURequestAdapterOptions 有以下成员:

> GPURequestAdapterOptions has the following members:

> 第一参数是：
> **powerPreference, 类型为：GPUPowerPreference**     
> 用户可选择性的提供一个提示参数（即可提供，可不提供），用于表明应该从系统中可用适配器中选择哪一类的适配器返回用户。    
> 这个提示的值可能会影响到选择哪种适配器，但它不能影响到 是否返回适配器的问题。

>> **powerPreference, of type GPUPowerPreference**    
>> Optionally provides a hint indicating what class of adapter should be selected from the system’s available adapters.    
>> The value of this hint may influence which adapter is chosen, but it must not influence whether an adapter is returned or not.    

> 注意：这个提示的主要作用是影响在一个多GPU系统中使用哪个GPU。例如，一些笔记本电脑有一个低功率的集成GPU和一个高性能的独立GPU。该提示也可能影响所选GPU的功率配置，以匹配所要求的功率偏好。

>> NOTE: The primary utility of this hint is to influence which GPU is used in a multi-GPU system. For instance, some laptops have a low-power integrated GPU and a high-performance discrete GPU. This hint may also affect the power configuration of the selected GPU to match the requested power preference.

> 注意：根据真实的硬件配置，如电池状态和连接的显示器或可插拔的GPU，用户代理可能在相同的功率偏好下选择不同的适配器。通常情况下，鉴于相同的硬件配置和状态以及powerPreference，用户代理通常会选择相同的适配器。

>> NOTE: Depending on the exact hardware configuration, such as battery status and attached displays or removable GPUs, the user agent may select different adapters given the same power preference. Typically, given the same hardware configuration and state and powerPreference, the user agent is likely to select the same adapter.

> 它必须是以下数值之一：    
> **undefined （或者不提供）**    
> 不向用户代理提供任何提示。    
> **"low-power"**    
> 表示请求的适配器，将优先考虑是否省电，而不在意性能。    
> **注意：**一般来说，如果内容不太可能受到绘制性能的限制，就应该使用这个值；例如，如果它每秒只渲染一帧，只用简单的着色器绘制相对简单的几何图形，或者使用一个尺寸小的HTML画布元素。如果他们的内容允许，我们鼓励开发者使用这个值，因为它可能会大大改善便携式设备的电池寿命。

>> It must be one of the following values:    
>> **undefined (or not present)**    
>>      Provides no hint to the user agent.    
>> **"low-power"**    
>>     Indicates a request to prioritize power savings over performance.    
>> **NOTE:** Generally, content should use this if it is unlikely to be constrained by drawing performance; for example, if it renders only one frame per second, draws only relatively simple geometry with simple shaders, or uses a small HTML canvas element. Developers are encouraged to use this value if their content allows, since it may significantly improve battery life on portable devices.

> **"high-performance"**    
> 表示请求的适配器，将优先考虑性能，而不在意是否省电。    
> **注意：**通过选择这个值，开发者应该意识到，对于在通过此参数配置得到的适配器上而创建的设备，用户代理更有可能强迫此设备丢失，以便通过切换到低功率的适配器来节省电力。我们鼓励开发者只在他们认为绝对必要时才指定这个值，因为它可能会大大减少便携式设备的电池寿命。

>> **"high-performance"**    
>> Indicates a request to prioritize performance over power consumption.
>> NOTE: By choosing this value, developers should be aware that, for devices created on the resulting adapter, user agents are more likely to force device loss, in order to save power by switching to a lower-power adapter. Developers are encouraged to only specify this value if they believe it is absolutely necessary, since it may significantly decrease battery life on portable devices.

> 第二个参数是：
> **forceFallbackAdapter，类型为布尔值，默认为false。**    
> 当设置为"true"时，表示可能只有一个后备适配器被返回。如果用户代理不支持后备适配器，将导致 requestAdapter() 的promise 的实例值被设置为空。
> **注意:** 如果forceFallbackAdapter被设置为false，并且没有其他合适的适配器可用，或者用户代理选择返回一个后备适配器，requestAdapter() 仍然可以返回一个后备适配器。开发者如果希望阻止他们的应用程序在后备适配器上运行，那么应该在请求 GPUDevice 之前首先检查 GPUAdapter.isFallbackAdapter的属性值。

>> **forceFallbackAdapter, of type boolean, defaulting to false**    
>> When set to true indicates that only a fallback adapter may be returned. If the user agent does not support a fallback adapter, will cause requestAdapter() to resolve to null.    
>> **NOTE:** requestAdapter() may still return a fallback adapter if forceFallbackAdapter is set to false and either no other appropriate adapter is available or the user agent chooses to return a fallback adapter. Developers that wish to prevent their applications from running on fallback adapters should check the GPUAdapter.isFallbackAdapter attribute prior to requesting a GPUDevice.

```
// 获取一个 ”high-performance“ 的 GPUAdapter:
// Requesting a "high-performance" GPUAdapter:

const gpuAdapter = await navigator.gpu.requestAdapter({
    powerPreference: 'high-performance'
});
```

## 4.3 GPUAdapter

> 4.3. GPUAdapter

一个GPUAdapter封装了一个adapter，其描述了它的能力（特征和限制）。     
要获得一个GPUAdapter，请调用requestAdapter()。

> A GPUAdapter encapsulates an adapter, and describes its capabilities (features and limits).
> To get a GPUAdapter, use requestAdapter().

```
[Exposed=(Window, DedicatedWorker), SecureContext]
interface GPUAdapter {
    [SameObject] readonly attribute GPUSupportedFeatures features;
    [SameObject] readonly attribute GPUSupportedLimits limits;
    readonly attribute boolean isFallbackAdapter;

    Promise<GPUDevice> requestDevice(optional GPUDeviceDescriptor descriptor = {});
    Promise<GPUAdapterInfo> requestAdapterInfo(optional sequence<DOMString> unmaskHints = []);
};
```

> GPUAdapter 具有以下属性：    
>    
> **features 类型为GPUSupportedFeatures, 只读**    
> 其数值集合主要在：this.[[adapter]].[[features]]  
>      
> **features 类型为GPUSupportedFeatures, 只读**     
> 其限制在：this.[[adapter]].[[limits]]    
>    
> **isFallbackAdapter, 类型为 bool, 只读**    
> 返回 [[adapter]].[[fallback]] 的值。    


> GPUAdapter has the following attributes:
>> **features, of type GPUSupportedFeatures, readonly**    
>> The set of values in this.[[adapter]].[[features]].    
>>
>> **limits, of type GPUSupportedLimits, readonly**    
>> The limits in this.[[adapter]].[[limits]].    
>>
>> **isFallbackAdapter, of type boolean, readonly**    
Returns the value of [[adapter]].[[fallback]].

> GPUAdapter有以下内部插槽:    
>    
> **[[adapter]], 类型为 adapter, 只读**    
> 用于描述GPUAdapter所引用的adapter。

>> GPUAdapter has the following internal slots:    
>>   
>> [[adapter]], of type adapter, readonly    
>> The adapter to which this GPUAdapter refers.  

GPUAdapter有以下成员方法：

第一个方法是：    
**requestDevice(descriptor)**    
作用：用于向适配器(adapter)请求一个设备(device)。

> GPUAdapter has the following methods:    
> **requestDevice(descriptor)**    
>   Requests a device from the adapter.    

调用于：GPUAdapter 的实例上。   

参数，如下：
| Parameter | Type | Nullable | Optional | Description |
| -------- | -------- | -------- | -------- | -------- |
| descriptor | GPUDeviceDescriptor | ✘ | ✔ | 要请求的GPUDevice的描述参数。 |

>Called on: GPUAdapter this.    
> Arguments:

| Parameter | Type | Nullable | Optional | Description |
| -------- | -------- | -------- | -------- | -------- |
| descriptor | GPUDeviceDescriptor | ✘ | ✔ | Description of the GPUDevice to request. |


返回值： Promise<GPUDevice>。    
在上下文时间线上的步骤是：
1. 让 contentTimeline 成为当前的上下文时间线。
2. 让 promise 是一个新的 promise。
3. 让adapter 是 this.[[adapter]]。
4. 向这个适配器的设备时间线发起初始化步骤请求。
5. 返回 promise。

> Returns: Promise<GPUDevice>    
>    
> Content timeline steps:    
> 1. Let contentTimeline be the current Content timeline.    
> 1. Let promise be a new promise.   
> 1. Let adapter be this.[[adapter]].   
> 1. Issue the initialization steps to the Device timeline of this.
> 1. Return promise.

***下面就转到上文所说的设备时间线上， 以下是原文***    
设备时间线初始化步骤：      
1. 如果以下任何一项要求没有得到满足。   
    * descriptor.requiredFeatures 中的数值集合 必须是 adapter.[[features]] 中的一个子集。    
    
然后在 contentTimeline 上发起以下步骤并返回：   
> 上下文时间线上的步骤：    
> 使promise为失败状态，其错误值为： TypeError。    
>    
> **注意：** 如果浏览器根本不知道某个特性的名称（在其 GPUFeatureName 定义的名称），也会产生同样的错误。这使得浏览器不支持某项功能时的行为， 与某一特定适配器不支持某项功能时的行为 趋于相同。

> Device timeline initialization steps:    
> 1. If any of the following requirements are unmet:    
>    * The set of values in descriptor.requiredFeatures must be a subset of those in adapter.[[features]]. 
>   
> Then issue the following steps on contentTimeline and return:
>> Content timeline steps:    
>>  Reject promise with a TypeError.
>    
> NOTE: This is the same error that is produced if a feature name isn’t known by the browser at all (in its GPUFeatureName definition). This converges the behavior when the browser doesn’t support a feature with the behavior when a particular adapter doesn’t support a feature.

2. 如果以下任何一项要求没有得到满足:    
  * descriptor.requiredLimits中的每个键必须是被支持的限制的成员名字。
  * 如果要求的每个限制名称key，在其所支持的限制的键集合中，那么让此key值为 descriptor.requiredLimits[key]。
    * 值必须不高于适配器中该限制的值。
    * 如果此限制的类别是对齐的，其值必须是2的幂。    

然后对 contentTimeline 发起以下步骤请求并返回:
> 上下文时间线上的步骤：    
> 使promise为失败状态，其错误值为：OperationError    

> 2. If any of the following requirements are unmet:
>     
>    * Each key in descriptor.requiredLimits must be the name of a member of supported limits.
>    * For each limit name key in the keys of supported limits: Let value be descriptor.requiredLimits[key].
>       * value must be no better than the value of that limit in adapter.[[limits]].
>       * If the limit’s class is alignment, value must be a power of 2.
> 
> Then issue the following steps on contentTimeline and return:
>> Content timeline steps:    
>> 1. Reject promise with an OperationError.

3. 如果适配器无效，或者用户代理在其他方面不能满足请求：
    1. 让 device 是一个新的 device。
    2. 让 device(device, undefined) 为丢失状态，即设置其属性 lost 为真。  
      **注意：** 如果它不是已经准备好的话，这将使得适配器无效。    
      **注意：** 用户代理应该考虑在大多数或所有这种情况发生时，发出开发者可见的警告。应用程序应该从 requestAdapter() 开始重新执行初始化逻辑。

   否则:   
    1. 让 device 成为一个具备 描述符所要求能力的 新的 device。

> 3. If adapter is invalid, or the user agent otherwise cannot fulfill the request:
>      1. Let device be a new device.
>      2. Lose the device(device, undefined).    
>>      **NOTE:** This makes adapter invalid, if it wasn’t already.    
>>    
>>      **NOTE:**  User agents should consider issuing developer-visible warnings in most or all cases when this occurs. Applications should perform reinitialization logic starting with requestAdapter().    
>      Otherwise:
>        1. Let device be a new device with the capabilities described by descriptor.

4. 在 contentTimeline 上发起以下后续步骤。
> 上下文时间线上的步骤：     
> 1. 使promise为成功状态，其值为：一个新的 GPUDevice 对象设备。  
> **注意：** 如果因为适配器不能满足请求，导致获取的设备已经丢失了，那么获取的 device.lost 的 promise 状态会被优先设置为成功，这在整个设备的 promise 被 resolved 之前。

> 4. Issue the subsequent steps on contentTimeline.    
>  Content timeline steps:    
>    1. Resolve promise with a new GPUDevice object device.    
>    
>        **NOTE:** If the device is already lost because the adapter could not fulfill the request, device.lost has already resolved before promise resolves.


**requestAdapterInfo()**    
用于请求 GPUAdapter 的 GPUAdapterInfo。

**注意:**  适配器的信息值是通过Promise返回的，以便给用户代理一个机会，在请求未屏蔽的值时执行潜在的长时间检查较验，例如在返回请求结果前征求用户同意，然而，如果没有指定的unmaskHints，就不应该再向用户显示任何对话框。

> **requestAdapterInfo()**    
> Requests the GPUAdapterInfo for this GPUAdapter.    
> **NOTE:** Adapter info values are returned with a Promise to give user agents an opportunity to perform potentially long-running checks when requesting unmasked values, such as asking for user consent before returning. If no unmaskHints are specified, however, no dialogs should be displayed to the user.

调用于：GPUAdapter 的实例上。   
参数，如下：
| Parameter | Type | Nullable | Optional | Description |
| -------- | -------- | -------- | -------- | -------- |
| unmaskHints | sequence\<DOMString\> | ✘ | ✔ | 	是一个 GPUAdapterInfo 属性名称的列表集合，如果有的话，集合里包含用户希望请求到的未屏蔽值的属性名称。 |
返回值为： Promise\<GPUAdapterInfo\>

> Called on: GPUAdapter this.    
> Arguments:

| Parameter | Type | Nullable | Optional | Description |
| -------- | -------- | -------- | -------- | -------- |
| unmaskHints | sequence\<DOMString\> | ✘ | ✔ | A list of GPUAdapterInfo attribute names for which unmasked values are desired if available. |

> Returns: Promise\<GPUAdapterInfo\>

**其在上下文时间线上的步骤如下：**
1. 让 promise 是一个新的 promise。
2. 让 adapter 是这个实例上的adapter (this.[[adapter]])。
3. 如果此实例对象上相关全局对象具有瞬时激活功能，让 hasActivation 为真，否则为假。
4. 然后，并行的运行以下步骤：    
    1. 如果 unmaskHints.length > 0。   
        1. 如果 hasActivation 为假，则将promise状态设置为 rejected，其值为 NotAllowedError，并中止后续步骤。
        2. 如果有的话，让 unmaskedKeys 的值为 unmaskHints 中指定的字段对应的值列表，这些值是由用户代理决定的，用户代理决定那些描述信息是未加掩码，可以被获取的。
        **注意：** 用户代理可以自由地使用它认为合适的任何方法来决定哪些字段是取消屏蔽的，是可以被访问的。
        3. 将 unmaskedKeys 附加到 adapter.[[unmaskedIdentifiers]]。
    2. 将 promise 的状态设置为resolved，其值是：适配器的一个新的适配器相关信息集合。
5. 返回 promise。

> Content timeline steps:
> 1. Let promise be a new promise.
> 2. Let adapter be this.[[adapter]].
> 3. Let hasActivation be true if the relevant global object for this has transient activation, and false otherwise.
> 4. Run the following steps in parallel:
>    1. If unmaskHints.length > 0:
>       1. If hasActivation is false reject promise with a NotAllowedError and abort these steps.
>       2. Let unmaskedKeys be a list of the fields specified in unmaskHints which the user agent decides to unmask, if any.    
>       **NOTE:** The user agent is free to use any method it deems appropriate to decide which fields to unmask.
>       3. Append unmaskedKeys to adapter.[[unmaskedIdentifiers]].
>    2. Resolve promise with a new adapter info for adapter.
> 5. Return promise.

```
// 例子 9
// 使用默认的 features 和 limits 来请求获取一个 GPUDevice。

EXAMPLE 9
Requesting a GPUDevice with default features and limits:

const gpuAdapter = await navigator.gpu.requestAdapter();
const gpuDevice = await gpuAdapter.requestDevice();
```

### 4.3.1. GPUDeviceDescriptor

GPUDeviceDescriptor 用于描述一个设备请求参数。

> 4.3.1. GPUDeviceDescriptor    
> GPUDeviceDescriptor describes a device request.

```
dictionary GPUDeviceDescriptor : GPUObjectDescriptorBase {
    sequence<GPUFeatureName> requiredFeatures = [];
    record<DOMString, GPUSize64> requiredLimits = {};
    GPUQueueDescriptor defaultQueue = {};
};
```

GPUDeviceDescriptor有以下成员：

**requiredFeatures，类型为 sequence<GPUFeatureName>，默认为[] 。** 
用于指定设备请求所需的功能。如果适配器不能提供这些特性，请求将失败。    
精确指定的特征集，不能多也不能少，则可在请求到的设备上进行 API 调用的验证。

> GPUDeviceDescriptor has the following members:    
> **requiredFeatures, of type sequence<GPUFeatureName>, defaulting to []**    
>     Specifies the features that are required by the device request. The request will fail if the adapter cannot provide these features.    
>     Exactly the specified set of features, and no more or less, will be allowed in validation of API calls on the resulting device.    

**requiredLimits，类型为 record<DOMString, GPUSize64>，默认为{}。**
指定设备请求所需的限制。如果适配器不能提供这些限制，请求将失败。    
每个键必须是所支持的限制的成员名称。精确的指定限制描述，没有更好或更差，则可在请求到的设备上进行 API 调用的验证。

> **requiredLimits, of type record<DOMString, GPUSize64>, defaulting to {}**    
>      Specifies the limits that are required by the device request. The request will fail if the adapter cannot provide these limits.    
>     Each key must be the name of a member of supported limits. Exactly the specified limits, and no better or worse, will be allowed in validation of API calls on the resulting device.   

**defaultQueue，类型为 GPUQueueDescriptor，默认为{}。**
默认的 GPUQueue 的描述符。

> **defaultQueue, of type GPUQueueDescriptor, defaulting to {}**    
>     The descriptor for the default GPUQueue.

```
// 例子 10
// 如果支持的话，请求一个具备 "texture-compression-astc" 功能的 GPUDevice
EXAMPLE 10

Requesting a GPUDevice with the "texture-compression-astc" feature if supported:

const gpuAdapter = await navigator.gpu.requestAdapter();

const requiredFeatures = [];
if (gpuAdapter.features.has('texture-compression-astc')) {
    requiredFeatures.push('texture-compression-astc')
}

const gpuDevice = await gpuAdapter.requestDevice({
    requiredFeatures
});
```

#### 4.3.1.1. GPUFeatureName

每个 GPUFeatureName 都定义了一组功能，如果在 GPUFeatureName 中有的话，则允许 WebGPU 进行额外使用，否则将是无效的。

> 4.3.1.1. GPUFeatureName    
> Each GPUFeatureName identifies a set of functionality which, if available, allows additional usages of WebGPU that would have otherwise been invalid.

```
enum GPUFeatureName {
    "depth-clip-control",
    "depth32float-stencil8",
    "texture-compression-bc",
    "texture-compression-etc2",
    "texture-compression-astc",
    "timestamp-query",
    "indirect-first-instance",
    "shader-f16",
    "rg11b10ufloat-renderable"
};
```

## 4.4. GPUDevice    
一个 GPUDevice 封装了一个设备，并暴露了该设备的功能。    
GPUDevice 是顶层接口，他是通过 WebGPU接口 来创建的。    
要获得一个GPUDevice，请使用requestDevice()。    

***后面一大段接口定义代码，不译也不重复粘贴***

> 4.4. GPUDevice    
> A GPUDevice encapsulates a device and exposes the functionality of that device.    
> GPUDevice is the top-level interface through which WebGPU interfaces are created.    
> To get a GPUDevice, use requestDevice().    


```
[Exposed=(Window, DedicatedWorker), SecureContext]
interface GPUDevice : EventTarget {
    [SameObject] readonly attribute GPUSupportedFeatures features;
    [SameObject] readonly attribute GPUSupportedLimits limits;

    [SameObject] readonly attribute GPUQueue queue;

    undefined destroy();

    GPUBuffer createBuffer(GPUBufferDescriptor descriptor);
    GPUTexture createTexture(GPUTextureDescriptor descriptor);
    GPUSampler createSampler(optional GPUSamplerDescriptor descriptor = {});
    GPUExternalTexture importExternalTexture(GPUExternalTextureDescriptor descriptor);

    GPUBindGroupLayout createBindGroupLayout(GPUBindGroupLayoutDescriptor descriptor);
    GPUPipelineLayout createPipelineLayout(GPUPipelineLayoutDescriptor descriptor);
    GPUBindGroup createBindGroup(GPUBindGroupDescriptor descriptor);

    GPUShaderModule createShaderModule(GPUShaderModuleDescriptor descriptor);
    GPUComputePipeline createComputePipeline(GPUComputePipelineDescriptor descriptor);
    GPURenderPipeline createRenderPipeline(GPURenderPipelineDescriptor descriptor);
    Promise<GPUComputePipeline> createComputePipelineAsync(GPUComputePipelineDescriptor descriptor);
    Promise<GPURenderPipeline> createRenderPipelineAsync(GPURenderPipelineDescriptor descriptor);

    GPUCommandEncoder createCommandEncoder(optional GPUCommandEncoderDescriptor descriptor = {});
    GPURenderBundleEncoder createRenderBundleEncoder(GPURenderBundleEncoderDescriptor descriptor);

    GPUQuerySet createQuerySet(GPUQuerySetDescriptor descriptor);
};
GPUDevice includes GPUObjectBase;
```

GPUDevice有以下属性。    
**features, 类型为 GPUSupportedFeatures, 只读**    
一个包含该设备所支持的 GPUFeatureName 值的集合（即创建该设备时的特征）。         
**limits, 类型为 GPUSupportedLimits, 只读**    
暴露设备支持的限制（即正是创建时的限制）。    
**queue, of type GPUQueue, 只读**    
此设备原生主要的 GPUQueue。    

> GPUDevice has the following attributes:    
> **features, of type GPUSupportedFeatures, readonly**    
>     A set containing the GPUFeatureName values of the features supported by the device (i.e. the ones with which it was created).    
> **limits, of type GPUSupportedLimits, readonly**    
Exposes the limits supported by the device (which are exactly the ones with which it was created).    
> **queue, of type GPUQueue, readonly**    
The primary GPUQueue for this device.    

GPUDevice 的 [[device]] 是指 GPUDevice 所引用的 device。    
GPUDevice 拥有上面 WebIDL 定义中列出的方法。那些没有在这里定义的方法，将在本文档的其他地方定义。

> The [[device]] for a GPUDevice is the device that the GPUDevice refers to.    
> GPUDevice has the methods listed in its WebIDL definition above. Those not defined here are defined elsewhere in this document.

**destroy()**
销毁 device，阻止对它的进一步操作。未完成的异步操作将失败。       
**注意：** 多次销毁一个设备是有效的。    
调用于: GPUDevice 实例上。    

> **destroy()**    
> Destroys the device, preventing further operations on it. Outstanding asynchronous operations will fail.    
> **NOTE:** It is valid to destroy a device multiple times.    
> Called on: GPUDevice this.    

以下是destory 操作步骤： 
在上下时间线步骤：
1. unmap() 这个设备的所有 GPUBuffers。
2. 在这个设备的 设备时间线上 发起后续步骤。

> Content timeline steps:    
> 1. unmap() all GPUBuffers from this device.   
> 2. Issue the subsequent steps on the Device timeline of this.

设备时间线的步骤：
1. 一旦在这个设备上任何队列的 所有当前排队的操作都完成了，就在 当前时间线上 发起后续步骤：

当前时间线上： 
1. 设置失去设备（this.[[device]], "destroyed"）。

> Device timeline steps:
> 1. Once all currently-enqueued operations on any queue on this device are completed, issue the subsequent steps on the current timeline.

> 1. Lose the device(this.[[device]], "destroyed").

**注意：** 由于在这个设备上不能再排队操作，实现可以立即中止未完成的异步操作并释放资源分配，包括刚刚解除映射的映射内存。

> **NOTE:** Since no further operations can be enqueued on this device, implementations can abort outstanding asynchronous operations immediately and free resource allocations, including mapped memory that was just unmapped.

一个 GPUDevice 允许的缓冲区用法如下：
* 总是允许的包括：MAP_READ, MAP_WRITE, COPY_SRC, COPY_DST, INDEX, VERTEX, UNIFORM, STORAGE, INDIRECT, QUERY_RESOLVE

> A GPUDevice's allowed buffer usages are:
> * Always allowed: MAP_READ, MAP_WRITE, COPY_SRC, COPY_DST, INDEX, VERTEX, UNIFORM, STORAGE, INDIRECT, QUERY_RESOLVE

一个 GPUDevice 允许的纹理用法如下：
* 总是允许的包括：COPY_SRC, COPY_DST, TEXTURE_BINDING, STORAGE_BINDING, RENDER_ATTACHMENT

> A GPUDevice's allowed texture usages are:
> * Always allowed: COPY_SRC, COPY_DST, TEXTURE_BINDING, STORAGE_BINDING, RENDER_ATTACHMENT

## 4.5. Example

> 4.5. Example

```
// 例子 11
EXAMPLE 11

// 一个更具鲁棒性的请求 GPUAdapter 和 GPUDevice 的例子，此例带有错误处理。
// A more robust example of requesting a GPUAdapter and GPUDevice with error handling:

let gpuDevice = null;

async function initializeWebGPU() {
    // 检查以便确保用户代理是否支持 WebGPU
    // Check to ensure the user agent supports WebGPU.
    if (!('gpu' in navigator)) {
        console.error("User agent doesn’t support WebGPU.");
        return false;
    }

    // 请求获取 adapter.
    // Request an adapter.
    const gpuAdapter = await navigator.gpu.requestAdapter();

    // 如果合适的 adapter 没有找到的话，requestAdapter 方法可能返回null。
    // requestAdapter may resolve with null if no suitable adapters are found.
    if (!gpuAdapter) {
        console.error('No WebGPU adapters found.');
        return false;
    }

    // 请求一个设备。
    // 注意： 如果无效的选项传递给 可选的参数字典，可能会导致 promise 状态设置为rejected.
    // 为了避免 promise 返回rejected，在调用 requestDevice() 之前，
    // 一定要根据适配器的特性和限制 来检查 任何所需特性 和 限制。

    // Request a device.
    // Note that the promise will reject if invalid options are passed to the optional
    // dictionary. To avoid the promise rejecting always check any features and limits
    // against the adapters features and limits prior to calling requestDevice().
    gpuDevice = await gpuAdapter.requestDevice();

    // requestDevice 永远也不会返回 null，但是如果一个有效设备请求由于某种原因不能得到满足，
    // promise 虽然是 resolved，但其值却是一个已经丢失的设备。
    // 此外，设备在创建后的任何时候都 有可能因为各种原因而丢失。
    // (例如： 浏览器的资源管理，驱动程序的更新等)
    // 因此，最好是能够优雅地处理丢失的设备。

    // requestDevice will never return null, but if a valid device request can’t be
    // fulfilled for some reason it may resolve to a device which has already been lost.
    // Additionally, devices can be lost at any time after creation for a variety of reasons
    // (ie: browser resource management, driver updates), so it’s a good idea to always
    // handle lost devices gracefully.
    gpuDevice.lost.then((info) => {
        console.error(`WebGPU device was lost: ${info.message}`);

        gpuDevice = null;

        // 导致设备丢失的许多原因都是短暂的，所以应用程序应该在以前的设备丢失后尝试获取一个新的设备。
        // 除非以前的设备的丢失原因是因为应用程序故意销毁设备引起的。
        // 注意：任何以前旧设备创建的 WebGPU 资源(缓冲区，纹理等)都需要在新的设备上重新创建。
  
        // Many causes for lost devices are transient, so applications should try getting a
        // new device once a previous one has been lost unless the loss was caused by the
        // application intentionally destroying the device. Note that any WebGPU resources
        // created with the previous device (buffers, textures, etc) will need to be
        // re-created with the new one.
        if (info.reason != 'destroyed') {
            initializeWebGPU();
        }
    });

    onWebGPUInitialized();

    return true;
}

function onWebGPUInitialized() {
    // 开始在这里创建 WebGPU 资源。
    // Begin creating WebGPU resources here...
}

initializeWebGPU();
```
