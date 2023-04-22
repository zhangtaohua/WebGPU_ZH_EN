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

# 3\. 基本概念
> 3\. Fundamentals

## 3.1. 公约

> 3.1. Conventions

### 3.1.1. 点符号语法（缩写语法）

> 3.1.1. Syntactic Shorthands

在本规范中，使用以下点符号语法：

> In this specification, the following syntactic shorthands are used:

**.("点")符号语法，在编程语言中非常常见**     
短语“Foo.bar”表示：“Foo对象中一个名为Bar成员变量的值”。   
短语“Foo.Bar” 如果提供了，那么这句话意味着 " Bar 成员存在于map对象 Foo 中"

> **The . ("dot") syntax, common in programming languages.**    
> The phrasing "Foo.Bar" means "the Bar member of the value (or interface) Foo."    
> The phrasing "Foo.Bar is provided" means "the Bar member exists in the map value Foo"

**?.("可选链式")语法，是为了适配JavaScript而定义的。**  
短语“Foo?.bar”表示：“如果Foo对象是null或者undefined, 那么则得到一个undefined值，否则返回Foo中Bar成员变量的值”。  
例如： 当一个缓冲区是GPUBuffer时，buffer?.[[device]].[[adapter]]表示：如果buffer是null或者undefined， 那么将得到一个undefined值，否则，将获得buffer的内部插槽[[device]]的内部插槽[[adapter]]。

> **The ?. ("optional chaining") syntax, adopted from JavaScript.**   
> The phrasing "Foo?.Bar" means "if Foo is null or undefined, undefined; otherwise, Foo.Bar".
> For example, where buffer is a GPUBuffer, buffer?.[[device]].[[adapter]] means "if buffer is null or undefined, then undefined; otherwise, the [[adapter]] internal slot of the [[device]] internal slot of buffer.

**?？("判空联合")语法，是为了适配JavaScript而定义的。**     
短语“x ?? y”表示：“如果x不是null或者不是undefined, 那么则得到x的值，否则返回y的值”。  

> **The ?? ("nullish coalescing") syntax, adopted from JavaScript.**    
> The phrasing "x ?? y" means "x, if x is not null/undefined, and y otherwise".

**基于插槽的属性**  
WebIDL属性是由一个同名的内部插槽所支持的，它可能是易变的，也可能不是易变的。

> **slot-backed attribute**
> A WebIDL attribute which is backed by an internal slot of the same name. It may or may not be mutable.


### 3.1.2. WebGPU 接口

> 3.1.2. WebGPU Interfaces

WebGPU的接口定义了一个WebGPU对象，它可用作以下用途：  
* 在创建的[内容时间线](https://www.w3.org/TR/webgpu/#content-timeline)上，WebGPU同时被创建了，它是一个JavaScript暴露的WebIDL接口。  
* 在所有其它的时间线上，只有[不可变的属性](https://www.w3.org/TR/webgpu/#immutable-property)是可以被访问的。  

> A WebGPU interface defines a WebGPU object. It can be used:
> * On the content timeline where it was created, where it is a JavaScript-exposed WebIDL interface.
> * On all other timelines, where only immutable properties can be accessed.

在[WebGPU 接口](https://www.w3.org/TR/webgpu/#webgpu-interface)上可以定义以下面特殊的属性类型。
> The following special property types can be defined on WebGPU interfaces:

***不可变属性***
在对象的初始化过程中设置只读的插槽，他可以在任何时间线上进行访问。 
> **注意：**由于插槽是不可变的，WebGPU的实现可能根据一定的需要在多个时间线上生成一些属性的副本，但在本规范中，以这种方式定义的[不可变属性](https://www.w3.org/TR/webgpu/#immutable-property)即避免了多个副本的描述。  
如果命名是[[带有双中括号的格式]]变量，表示它是一个内部插槽，如果命名方式为不带括号的，表示它是一个[基于插槽的](https://www.w3.org/TR/webgpu/#slot-backed-attribute)只读属性。

> ***immutable property***  
> A read-only slot set during initialization of the object. It can be accessed from any timeline.
>> NOTE: Since the slot is immutable, implementations may have a copy on multiple timelines, as needed. Immutable properties are defined in this way to avoid describing multiple copies in this spec.  
> If named [[with brackets]], it is an internal slot. If named withoutBrackets, it is a readonly slot-backed attribute.

***内容时间线属性***  
内容时间线属性是指，一个只能 从创建该对象的 [内容时间线](https://www.w3.org/TR/webgpu/#content-timeline)上访问的属性.   
如果命名是[[带有双中括号的格式]]变量，表示它是一个内部插槽，如果命名方式是不带括号的，表示它是一个[基于插槽的属性](https://www.w3.org/TR/webgpu/#slot-backed-attribute)。  
任何包含 [GPUObjectBase](https://www.w3.org/TR/webgpu/#gpuobjectbase) 的接口都是[WebGPU接口](https://www.w3.org/TR/webgpu/#webgpu-interface)。

> ***content timeline property***  
> A property which is only accessible from the content timeline where the object was created.  
> If named [[with brackets]], it is an internal slot. If named withoutBrackets, it is a slot-backed attribute.

> Any interface which includes GPUObjectBase is a WebGPU interface.

举例代码如下：

```
interface mixin GPUObjectBase {
    attribute USVString label;
};
```

为了***创建一个新的WebGPU对象***   
****注意：*** 由于下面语句类似于代码，这里不做翻译。后面的创建过程翻译也不是很清楚，建议直接查看英语原文*  
（[GPUObjectBase parent](https://www.w3.org/TR/webgpu/#gpuobjectbase), interface T, [GPUObjectDescriptorBase](https://www.w3.org/TR/webgpu/#dictdef-gpuobjectdescriptorbase) descriptor) (where T extends [GPUObjectBase](https://www.w3.org/TR/webgpu/#gpuobjectbase)):   
1. 让新建对象的device为父对象的[[device]]。
1. 让新建对象是T的一新的实例。  
1. 让内部变量是一个新的(未初始化的)类型为 T.[[internals]] 的实例，（可能会覆盖 [GPUObjectBase](https://www.w3.org/TR/webgpu/#gpuobjectbase).[[internals]]）, 而这些变量只可以从 设备的device时间线上进行访问。  
1. 设置 object.[[device]] 为 device。    
1. 设置 object.[[internals]] 为 ingernals。   
1. 设置 object.label 为 描述符的.label。  
1. 返回 [object, internals]。  

> To ***create a new WebGPU object*** (GPUObjectBase parent, interface T, GPUObjectDescriptorBase descriptor) (where T extends GPUObjectBase):
> 1. Let device be parent.[[device]].   
> 1. Let object be a new instance of T.   
> 1. Let internals be a new (uninitialized) instance of the type of T.[[internals]] (which may override GPUObjectBase.[[internals]]) that is accessible only from the device timeline of device.   
> 1. Set object.[[device]] to device.    
> 1. Set object.[[internals]] to internals.   
> 1. Set object.label to descriptor.label.    
> 1. Return [object, internals].   

[GPUObjectBase](https://www.w3.org/TR/webgpu/#gpuobjectbase) 对象具有下面 [不可变属性](https://www.w3.org/TR/webgpu/#immutable-property)。
> GPUObjectBase has the following immutable properties:

***[[internals]]，类型为[内部对象](https://www.w3.org/TR/webgpu/#internal-object)，只读，可重写***  
> [内部对象](https://www.w3.org/TR/webgpu/#internal-object)  
>> 对这个对象的内容进行操作时，则[断言](https://infra.spec.whatwg.org/#assert)它们是在[设备时间线上](https://www.w3.org/TR/webgpu/#device-timeline)运行的，并且其设备是[有效的](https://www.w3.org/TR/webgpu/#valid)。  
>> 对于每个子类型为GPUObjectBase的接口，可以用一个内部对象的子类型来重写。 这个插槽最初被设置为该类型的一个未初始化的对象。  

> ***[[internals]], of type internal object, readonly, overridable***   
>> The internal object.
>> Operations on the contents of this object assert they are running on the device timeline, and that the device is valid.  
>> For each interface that subtypes GPUObjectBase, this may be overridden with a subtype of internal object. This slot is initially set to an uninitialized object of that type.

***[[device]]，类型为[device](https://www.w3.org/TR/webgpu/#device)，只读，***  
>> 设备拥有[内部对象](https://www.w3.org/TR/webgpu/#internal-object)。  
>> 对这个对象的内容进行操作时，则断言它们是在设备时间线上运行的，并且其设备是有效的。   
> ***[[device]], of type device, readonly***  
>> The device that owns the internal object.  
>> Operations on the contents of this object assert they are running on the device timeline, and that the device is valid.  

GPUObjectBase拥有以下的内容时间线上的属性。

> GPUObjectBase has the following content timeline properties:

> *** label, 类型为[USVString](https://webidl.spec.whatwg.org/#idl-USVString) ***  
>> 一个开发者提供的标签，是在一种实现定义的方式中使用。他可以被浏览器，操作系统，或者其他工具所使用，以便帮助开发者识别底层的内部对象。例如，可在控制台警告信息中，浏览器开发工具中，或者其他平台相关的调试工具中，显示包含有用户设置的标签的[GPUError](https://www.w3.org/TR/webgpu/#gpuerror)错误信息。  
>>> **注意：**标签信息是GPUObjectBase的一个属性。两个GPUObjectBase“包装”对象拥有完全独立的标签状态，即使它们引用的同一个底层对象( 例如由[getBindGroupLayout()](https://www.w3.org/TR/webgpu/#dom-gpupipelinebase-getbindgrouplayout)函数调用返回的对象)。标签属性是不会改变的，除非是由JavaScript代码设置的。  
>>> 这意味着一个底层对象可以和多个标签相关联。本规范没有定义标签是如何传递到设备时间线上的。如何使用标签完全由WebGPU的实现者决定：错误信息可以显示最新设置的标签，或者显示所有已知的标签，或者根本没有标签可显示。
>>> 此属性被定义为USVString类型，是因为一些用户代理可能会将其提供给底层原生的API调度机制使用。

> ***label, of type USVString***  
> A developer-provided label which is used in an implementation-defined way. It can be used by the browser, OS, or other tools to help identify the underlying internal object to the developer. Examples include displaying the label in GPUError messages, console warnings, browser developer tools, and platform debugging utilities.

>> **Note**: The label is a property of the GPUObjectBase. Two GPUObjectBase "wrapper" objects have completely separate label states, even if they refer to the same underlying object (for example returned by getBindGroupLayout()). The label property will not change except by being set from JavaScript.  

>> This means one underlying object could be associated with multiple labels. This specification does not define how the label is propagated to the device timeline. How labels are used is completely implementation-defined: error messages could show the most recently set label, all known labels, or no labels at all.  

>> It is defined as a USVString because some user agents may supply it to the debug facilities of the underlying native APIs.

> **注意：**在理想情况下，[WebGPU接口](https://www.w3.org/TR/webgpu/#webgpu-interface)不应该阻止它们的父对象被垃圾回收机制回收，例如那些拥有他们的[[device]]。但这并不是被保证的，因为在某些具体的WebGPU实现中，可能需要持有对父对象的强引用。  
> 因此，开发人员应该假设WebGPU接口可能不会被垃圾回收，直到该接口的所有子对象都被垃圾回收了。这可能会导致一些资源的最终占用保持时间会超过预期。  
> 如果有必要的话，开发者在可以准确的预测已分配资源的释放时机，应该优先调用WebGPU接口上的destory方法（例如： [GPUDevice.destroy()](https://www.w3.org/TR/webgpu/#dom-gpudevice-destroy) 或者 [GPUBuffer.destroy()](https://www.w3.org/TR/webgpu/#dom-gpubuffer-destroy) ），而不是依赖于浏览器的垃圾回收机制。  

> Note: Ideally WebGPU interfaces should not prevent their parent objects, such as the [[device]] that owns them, from being garbage collected. This cannot be guaranteed, however, as holding a strong reference to a parent object may be required in some implementations.

> As a result, developers should assume that a WebGPU interface may not be garbage collected until all child objects of that interface have also been garbage collected. This may cause some resources to remain allocated longer than anticipated.

> Calling the destroy method on a WebGPU interface (such as GPUDevice.destroy() or GPUBuffer.destroy()) should be favored over relying on garbage collection if predictable release of allocated resources is needed.

### 3.1.3. 内部对象

> 3.1.3. Internal Objects

一个内部对象用来跟踪WebGPU对象的状态，这些对象只能在设备时间线上使用，或者在设备时间线插槽中，他们可能是可变的。

> An internal object tracks state of WebGPU objects that may only be used on the device timeline, in device timeline slots, which may be mutable.

> 设备时间线插槽   
>> 一个内部的插槽，只能从设备时间线上进行访问。

> device timeline slot
>> An internal slot which is only accessible from the device timeline.

对一个内部对象的易变状态的 所有的读/写操作 都来自于在一个单一有序的设置时间线上执行的步骤。 这些步骤可能由任何多线程代理的一个内容时间线算法提交的。

> All reads/writes to the mutable state of an internal object occur from steps executing on a single well-ordered device timeline. These steps may have been issued from a content timeline algorithm on any of multiple agents.

> **注意：** 一个“代理”是指一个JavaScript"线程"（例如主线程，或者Web Worker）

>> NOTE: An "agent" refers to a JavaScript "thread" (i.e. main thread, or Web Worker).


### 3.1.4. 对象描述符
3.1.4. Object Descriptors

对象描述符持有创建一个对象所需的所有信息，通常是通过GPUDevice的一个create*方法来完成。
```
dictionary GPUObjectDescriptorBase {
    USVString label;
};
```
`GPUObjectDescriptorBase`有以下成员:

```
label, 类型为USVString
  表示 GPUObjectBase.label 的初始值。
```

> An object descriptor holds the information needed to create an object, which is typically done via one of the create* methods of GPUDevice.

```
dictionary GPUObjectDescriptorBase {
    USVString label;
};
```
> GPUObjectDescriptorBase has the following members:
```
label, of type USVString
  The initial value of GPUObjectBase.label.
```

3.2.异步性
> 3.2. Asynchrony

3.2.1.无效的内部对象和传染性的无效性
> 3.2.1. Invalid Internal Objects & Contagious Invalidity

WebGPU中的对象创建操作并不返回promises，但在内部是异步的。返回的对象引用了在设备时间线上被操作的内部对象。在设备时间线上发生的大多数错误都会通过 相关设备上生成的 GPUErrors 来传达的，而不是抛出异常或拒绝来表示创建失败。

内部对象要么有效，要么无效。一个无效的对象在以后的时间里永远不会再变得有效，但一些有效的对象可能会变得无效。

> Object creation operations in WebGPU don’t return promises, but nonetheless are internally asynchronous. Returned objects refer to internal objects which are manipulated on a device timeline. Rather than fail with exceptions or rejections, most errors that occur on a device timeline are communicated through GPUErrors generated on the associated device.

> Internal objects are either valid or invalid. An invalid object will never become valid at a later time, but some valid objects may become invalid.

如果不可能创建对象，那么对象从创建开始就是无效的。这是可能发生的，例如，如果对象描述符 没有 描述一个有效的对象，或者没有足够的内存来为对象分配相应的资源。

大多数类型的内部对象在创建后是不会变得无效的，但仍然可能变得无法使用，例如，如果拥有此对象的设备丢失或被销毁了，或者对象有一个特殊的内部状态，如缓冲区有一个"被销毁"的状态。

少娄类型的内部对象在创建后也可能变得无效；特别是设备、适配器、GPUCommandBuffers和命令/传递/捆绑编码器。

> Objects are invalid from creation if it wasn’t possible to create them. This can happen, for example, if the object descriptor doesn’t describe a valid object, or if there is not enough memory to allocate a resource.

> Internal objects of most types cannot become invalid after they are created, but still may become unusable, e.g. if the owning device is lost or destroyed, or the object has a special internal state, like buffer state "destroyed".

> Internal objects of some types can become invalid after they are created; specifically, devices, adapters, GPUCommandBuffers, and command/pass/bundle encoders.

当且仅当满足以下要求时，一个给定的GPUObjectBase对象可以有效地与一个targetObject一起使用：
* 对象必须是有效的。
* object.[[device]]必须有效。
* object.[[设备]]必须等于targetObject.[[设备]]。

> A given GPUObjectBase object is valid to use with a targetObject if and only if the following requirements are met:
> * object must be valid.
> * object.[[device]] must be valid.
> * object.[[device]] must equal targetObject.[[device]].

### 3.2.2 Promise 顺序

> 3.2.2. Promise Ordering

WebGPU 中的一些操作会返回 promise.

> Several operations in WebGPU return promises.

***下面一段代码不译***， 例如： 

> * [GPU.requestAdapter()](https://www.w3.org/TR/2023/WD-webgpu-20230202/#dom-gpu-requestadapter)    
> * [GPUAdapter.requestDevice()](https://www.w3.org/TR/2023/WD-webgpu-20230202/#dom-gpuadapter-requestdevice)   
> * [GPUAdapter.requestAdapterInfo()](https://www.w3.org/TR/2023/WD-webgpu-20230202/#dom-gpuadapter-requestadapterinfo)   
> * [GPUDevice.createComputePipelineAsync()](https://www.w3.org/TR/2023/WD-webgpu-20230202/#dom-gpudevice-createcomputepipelineasync)  
> * [GPUDevice.createRenderPipelineAsync()](https://www.w3.org/TR/2023/WD-webgpu-20230202/#dom-gpudevice-createrenderpipelineasync)   
> * [GPUBuffer.mapAsync()](https://www.w3.org/TR/2023/WD-webgpu-20230202/#dom-gpubuffer-mapasync)   
> * [GPUShaderModule.getCompilationInfo()](https://www.w3.org/TR/webgpu/#dom-gpushadermodule-getcompilationinfo)   
> * [GPUQueue.onSubmittedWorkDone()](https://www.w3.org/TR/2023/WD-webgpu-20230202/#dom-gpuqueue-onsubmittedworkdone)   
> * [GPUDevice.lost](https://www.w3.org/TR/2023/WD-webgpu-20230202/#dom-gpudevice-lost)   
> * [GPUDevice.popErrorScope()](https://www.w3.org/TR/2023/WD-webgpu-20230202/#dom-gpudevice-poperrorscope)   


WebGPU对这些Promise的设定（解决或拒绝）顺序不做任何保证，但以下情况除外：
* 如果 p1 = b.mapAsync() 在 p2 = q.onSubmittedWorkDone() 之前被调用，并且 b 最后被专门用于 q，那么 p2 一定不会在 p1 解决之前设定为解决状态。

应用程序不得依赖任何其他promise的设定排序。

> WebGPU does not make any guarantees about the order in which these promises settle (resolve or reject), except for the following:

> * If p1 = b.mapAsync() is called before p2 = q.onSubmittedWorkDone(), and b was last used exclusively on q, then p2 must not resolve before p1 resolves.

> Applications must not rely on any other promise settlement ordering.

3.3.坐标系统

> 3.3. Coordinate Systems

Y轴向上是归一化设备坐标系（NDC）：NDC中的点（-1.0，-1.0）位于NDC的左下角。此外，NDC中的x和y应该在-1.0和1.0之间，而NDC中的z应该在0.0和1.0之间。在NDC中，超出这个范围的顶点不会引发任何错误，但它们会被剪掉，而不会被渲染显示和计算得到期望结果。   
Y轴在帧缓冲区坐标、视口坐标和片段/像素坐标中是向下的：原点（0，0）位于这些坐标系的左上角。    
窗口/显示坐标系与帧缓冲区坐标系相匹配。  
纹理坐标系中原点(0, 0)表示 UV纹理内存中的第一个texel（最低字节）。

**注意：** WebGPU的坐标系统与DirectX在图形管道中的坐标系统一致。

> * Y-axis is up in normalized device coordinate (NDC): point(-1.0, -1.0) in NDC is located at the bottom-left corner of NDC. In addition, x and y in NDC should be between -1.0 and 1.0 inclusive, while z in NDC should be between 0.0 and 1.0 inclusive. Vertices out of this range in NDC will not introduce any errors, but they will be clipped.   
> * Y-axis is down in framebuffer coordinate, viewport coordinate and fragment/pixel coordinate: origin(0, 0) is located at the top-left corner in these coordinate systems.   
> * Window/present coordinate matches framebuffer coordinate.   
> * UV of origin(0, 0) in texture coordinate represents the first texel (the lowest byte) in texture memory.

> NOTE: WebGPU’s coordinate systems match DirectX’s coordinate systems in a graphics pipeline.

## 3.4. 编程模型

3.4. Programming Model

### 3.4.1.时间线
3.4.1. Timelines

本节为非规范性内容。  
一个在前端有用户代理、在后端有GPU的 计算机系统中，有一些组件在不同的时间线上并行工作：   
> * **内容时间轴**   
> 与Web脚本的执行有关。它包括调用本规范所描述的所有方法。    
> 要想从 GPUDevice 设备上的操作 向 内容时间线 发送操作步骤，请在一个全局任务队列中 为 GPUDevice设备排队执行这些步骤。   
> * **设备时间线**    
与用户代理发送的 GPU设备操作 相关。它包括创建适配器、设备以及GPU资源和相关状态对象，从控制GPU的用户代理的角度来看，这些操作通常是同步的，但可以在一个单独的操作系统进程中生存、执行。   
>*  **队列时间线**    
与GPU的计算单元上的操作执行有关。它包括在GPU上运行的实际的绘制、复制和计算工作。


> This section is non-normative.   
> A computer system with a user agent at the front-end and GPU at the back-end has components working on different timelines in parallel:
> * **Content timeline**  
>    Associated with the execution of the Web script. It includes calling all methods described by this specification.  
>    To issue steps to the content timeline from an operation on GPUDevice device, queue a global task for GPUDevice device with those steps.   
> * **Device timeline**   
> Associated with the GPU device operations that are issued by the user agent. It includes creation of adapters, devices, and GPU resources and state objects, which are typically synchronous operations from the point of view of the user agent part that controls the GPU, but can live in a separate OS process.   
> * **Queue timeline**  
> Associated with the execution of operations on the compute units of the GPU. It includes actual draw, copy, and compute jobs that run on the GPU.

```
举例 1：
下面显示了与每个时间线相关的步骤和值的样式。这种设计风格是非规范性的；规范文本总是描述相关联的。

不可变值的示例定义
可以在任何时间线上使用。

内容-时间线示例定义
只能在内容时间线上使用。

设备-时间线示例定义
只能在设备时间线上使用。

队列-时间线示例定义
只能在队列时间线上使用。

在 内容时间线上 执行的步骤看起来像这样：
不可变的值示例定义。内容-时间线示例定义。

在 设备时间线上执行的步骤 看起来有。
不可变的值的定义示例。设备时间线的定义示例。

在 队列时间线上执行的步骤 有。
不可变的值的定义示例。队列时间线的定义示例。
```

```
EXAMPLE 1

The following show the styling of steps and values associated with each timeline. This styling is non-normative; the specification text always describes the association.  

Immutable value example definition
  Can be used on any timeline.

Content-timeline example definition
  Can only be used on the content timeline.

Device-timeline example definition
  Can only be used on the device timeline.

Queue-timeline example definition
  Can only be used on the queue timeline.

Steps executed on the content timeline look like this.
  Immutable value example definition. Content-timeline example definition.

Steps executed on the device timeline look like this.
  Immutable value example definition. Device-timeline example definition.

Steps executed on the queue timeline look like this.
  Immutable value example definition. Queue-timeline example definition.
```

在本规范中，在内容时间线以外的 任何时间线上的工作执行所产生的结果，就会发生异步操作。它们由JavaScript中的回调(callbacks)和promises表示。

In this specification, asynchronous operations are used when the result value depends on work that happens on any timeline other than the Content timeline. They are represented by callbacks and promises in JavaScript.


```
示例二：

GPUComputePassEncoder.dispatchWorkgroups()。
用户通过调用 GPUComputePassEncoder 的方法派发一个dispatchWorkgroups命令进行编码，该方法发生在内容时间线上。

用户调用GPUQueue.submit()函数，将GPUCommandBuffer提交给用户代理，用户代理通过调用操作系统驱动程序进行更低级别任务处理，这发生在设备时间线上。

提交到GPU的任务，被GPU调用调度器分派到实际的计算单元上执行，这发生在队列时间线上。

```

```
EXAMPLE 2  

GPUComputePassEncoder.dispatchWorkgroups():  

User encodes a dispatchWorkgroups command by calling a method of the GPUComputePassEncoder which happens on the Content timeline.

User issues GPUQueue.submit() that hands over the GPUCommandBuffer to the user agent, which processes it on the Device timeline by calling the OS driver to do a low-level submission.

The submit gets dispatched by the GPU invocation scheduler onto the actual compute units for execution, which happens on the Queue timeline.
```

```
示例3
GPUDevice.createBuffer()。
用户填写一个GPUBufferDescriptor的描述符，并用它创建一个GPUBuffer，这发生在内容时间线上。

用户代理在设备时间线上创建一个低级别的缓冲区。
```

```
EXAMPLE 3
GPUDevice.createBuffer():
User fills out a GPUBufferDescriptor and creates a GPUBuffer with it, which happens on the Content timeline.

User agent creates a low-level buffer on the Device timeline.
```

```
示例4
GPUBuffer.mapAsync()。
用户请求在内容时间线上映射一个GPUBuffer，并得到一个promise返回值。

用户代理检查该缓冲区是否正在被GPU使用，如果正在使用，则等待并提醒自己在其被使用结束后，再次进行检查。

当在队列时间线上操作的GPU使用完该缓冲区后，用户代理将其映射到内存后并返回resolves的 promise。
```

```
EXAMPLE 4
GPUBuffer.mapAsync():
User requests to map a GPUBuffer on the Content timeline and gets a promise in return.

User agent checks if the buffer is currently used by the GPU and makes a reminder to itself to check back when this usage is over.

After the GPU operating on Queue timeline is done using the buffer, the user agent maps it to memory and resolves the promise.
```

#3.4.2. 内存模型
3.4.2. Memory Model

本节为非规范性内容。    
一旦应用程序在初始化过程中获得了GPUDevice，我们就可以将WebGPU平台描述成由以下几层组成：
* 实施该规范的用户代理。
* 具有该设备低级别原生API驱动的操作系统。
* 实际的CPU和GPU硬件。

> This section is non-normative.

> Once a GPUDevice has been obtained during an application initialization routine, we can describe the WebGPU platform as consisting of the following layers:
> 1. User agent implementing the specification.
> 1. Operating system with low-level native API drivers for this device.
> 1. Actual CPU and GPU hardware.

WebGPU平台的每一层级都可能对应着不同的内存类型，用户代理在实现该规范时需要考虑这个问题。  
* 脚本拥有的内存，例如由脚本创建的ArrayBuffer，一般不能被GPU驱动访问。
* 一个用户代理可能由不同的进程负责运行内容或者和GPU驱动进行通信。在这种情况下，它们将使用进程间共享内存来传输数据。
* 专用独立的GPU有着自己的高带宽的内存，而集成的GPU通常与系统共享内存。

Each layer of the WebGPU platform may have different memory types that the user agent needs to consider when implementing the specification:
* The script-owned memory, such as an ArrayBuffer created by the script, is generally not accessible by a GPU driver.
* A user agent may have different processes responsible for running the content and communication to the GPU driver. In this case, it uses inter-process shared memory to transfer data.
* Dedicated GPUs have their own memory with high bandwidth, while integrated GPUs typically share memory with the system.

大多数物理资源被分配在内存中不同的类型，主要是为了让GPU计算或渲染更加有效率。当用户需要向GPU提供新的数据时，这些数据首先可能需要跨越进程边界，以到达与GPU驱动程序进行通信的用户代理部分。然后，它可能需要对驱动程序可见，有时，这需要复制这些数据到驱动程序分配的暂存内存中。最后，它可能需要被转移到专用GPU的内存中，这可能会改变内部内存布局，使其能被GPU高效的操作。

> Most physical resources are allocated in the memory of type that is efficient for computation or rendering by the GPU. When the user needs to provide new data to the GPU, the data may first need to cross the process boundary in order to reach the user agent part that communicates with the GPU driver. Then it may need to be made visible to the driver, which sometimes requires a copy into driver-allocated staging memory. Finally, it may need to be transferred to the dedicated GPU memory, potentially changing the internal layout into one that is most efficient for GPUs to operate on.

所有这些数据复制、传输都是由用户代理的WebGPU具体实现完成的。

**注意：** 以下的例子是最坏的实现情况，在实践中，实现可能不需要跨越进程边界，而可能是将驱动管理的内存直接暴露给用户代理的ArrayBuffer，从而避免了任何数据拷贝。

> All of these transitions are done by the WebGPU implementation of the user agent.      
> NOTE: This example describes the worst case, while in practice the implementation may not need to cross the process boundary, or may be able to expose the driver-managed memory directly to the user behind an ArrayBuffer, thus avoiding any data copies.

### 3.4.3.资源使用

3.4.3. Resource Usages

一个物理资源可以在GPU上作为一个内部使用量被使用。
* **输入**    
是一个带有输入数据的缓冲区，用于绘制或调度调用。主要用于保留内容，允许使用于索引缓存区、顶点缓存区或直接访问缓存区。
* **常量**  
从着色器的角度来看，常量是恒定的资源绑定。主要用于保留内容，允许使用于 UNIFORM缓冲区 或TEXTURE_BINDING的纹理。
* **存储**   
可写的存储资源绑定。允许使用于STORAGE缓冲区或STORAGE_BINDING的纹理。
* **存储-读**   
只读的存储资源绑定。用于保留内容，允许使用于STORAGE缓冲区。
* **附件**   
在渲染过程中，作为输出附件而使用的纹理。允许使用于RENDER_ATTACHMENT的纹理。
* **附件-读**    
在渲染过程中，作为只读附件而使用的纹理。用于保留内容，允许使用于 RENDER_ATTACHMENT的纹理。

> A physical resource can be used on GPU with an internal usage:      
> * **input**  
>   Buffer with input data for draw or dispatch calls. Preserves the contents. Allowed by buffer INDEX, buffer VERTEX, or buffer INDIRECT.   
> * **constant**   
>   Resource bindings that are constant from the shader point of view. Preserves the contents. Allowed by buffer UNIFORM or texture TEXTURE_BINDING.    
> * **storage**   
>   Writable storage resource binding. Allowed by buffer STORAGE or texture STORAGE_BINDING.     
> * **storage-read**    
>   Read-only storage resource bindings. Preserves the contents. Allowed by buffer STORAGE.    
> * **attachment**   
>   Texture used as an output attachment in a render pass. Allowed by texture RENDER_ATTACHMENT.    
> * **attachment-read**    
>   Texture used as a read-only attachment in a render pass. Preserves the contents. Allowed by texture RENDER_ATTACHMENT.    

我们将子资源 定义其要么是整个缓冲区，或者是一个纹理的子资源。

一些资源内部用法 与资源的其他用法是兼容的。一个子资源可以处于一个将多种使用方式结合在一起的状态。假设，一个列表 U 是一个兼容的使用列表，如果它们满足以下任何规则(也仅满足)：
* 在列表 U 中的每个资源用法都是输入、常量、存储-读取或附件-读取。
* 在列表 U 中的每个用法都是存储。
* 在列表 U 正好包含一个用法：附件。

> We define subresource to be either a whole buffer, or a texture subresource.
> * Some internal usages are compatible with others. A subresource can be in a state that combines multiple usages together. We consider a list U to be a compatible usage list if (and only if) it satisfies any of the following rules:
> * Each usage in U is input, constant, storage-read, or attachment-read.
> * Each usage in U is storage.
> * U contains exactly one element: attachment.

强制规定这些资源使用方法 合并到一个可兼容的资源使用方法列表中，使得API在处理内存时，可以限制数据竞赛的发生。这一特性使得针对WebGPU编写的应用程序，更有可能在不同的平台上不加修改就可以运行。

> Enforcing that the usages are only combined into a compatible usage list allows the API to limit when data races can occur in working with memory. That property makes applications written against WebGPU more likely to run without modification on different platforms.

一般来说，当一个WebGPU的具体实现在处理一个 以不同于其当前正在使用的资源使用方式来使用子资源的操作时，将会导致资源进行转换到新的状态。在某些情况下，比如，在一个开放的GPURenderPassEncoder中，由于硬件的限制，这种资源使用方式转换是不可能的。我们将这些地方定义为资源使用范围。

> Generally, when an implementation processes an operation that uses a subresource in a different way than its current usage allows, it schedules a transition of the resource into the new state. In some cases, like within an open GPURenderPassEncoder, such a transition is impossible due to the hardware limitations. We define these places as usage scopes.

主要的使用规则是，对于任何一个子资源，在其内部使用方式的列表中，必须是在一个使用范围内，且是一系列可兼容的使用方式列表。

例如，一个绑定了相同的缓冲区用于存储和输入时，如果GPURenderPassEncoder中编码器的使用方式 和 其所属的GPUCommandEncoder中编码器的使用方式一样，会进入错误状态。这种使用方式的组合，并不构成一个可兼容的使用列表。

> The main usage rule is, for any one subresource, its list of internal usages within one usage scope must be a compatible usage list.

> For example, binding the same buffer for storage as well as for input within the same GPURenderPassEncoder would put the encoder as well as the owning GPUCommandEncoder into the error state. This combination of usages does not make a compatible usage list.

**注意：** 允许在同一个使用范围内，在可写存储缓冲区/纹理的 多线程竞态条件是被允许的。

提供给 GPURenderPassColorAttachment.view和GPURenderPassColorAttachment.resolveTarget的视图中，他们所包含的纹理子资源，被当作 是此渲染通道中使用范围的附件而使用。

> NOTE: race condition of multiple writable storage buffer/texture usages in a single usage scope is allowed.

> The subresources of textures included in the views provided to GPURenderPassColorAttachment.view and GPURenderPassColorAttachment.resolveTarget are considered to be used as attachment for the usage scope of this render pass.

### 3.4.4 同步化
> 3.4.4. Synchronization

对于物理资源的每个子资源，其内部使用标志集在队列时间线上被跟踪。

在队列时间线上，有一个使用范围的有序序列。在每个范围的持续时间内，任何给定的子资源的内部使用标志集是不变的。一个子资源可以在两个使用范围的边界处进行转换为新的使用方式。

> For each subresource of a physical resource, its set of internal usage flags is tracked on the Queue timeline.

> On the Queue timeline, there is an ordered sequence of usage scopes. For the duration of each scope, the set of internal usage flags of any given subresource is constant. A subresource may transition to new usages at the boundaries between usage scopes.

本规范定义了以下几种使用范围：
* 在一个通道之外（在GPUCommandEncoder中），每个（非状态设置）命令是一个使用范围（例如copyBufferToTexture()）。
* 在一个计算通道中，每个调度命令（dispatchWorkgroups()或dispatchWorkgroupsIndirect()）就是一个使用范围。如果一个子资源有可能被命令访问到，那么就认为它在使用范围内被"使用"了。在一个调度中，对于当前GPUComputePipeline.[[layout]]所使用的每一个绑定组插槽，在该绑定组中所引用的每个子资源，都在使用范围中被 "使用"。状态设置的计算通道命令，如setBindGroup()，并不直接决定着使用范围；相反，它通过改变调度命令中被检查的状态来间接影响使用范围。
* 一个渲染通道也是一个使用范围。如果一个子资源被任何（状态设置或非状态设置的）命令所引用，它就在此使用范围中被 "使用"了。例如，在setBindGroup()中，bindGroup中的每个子资源都在渲染通道的使用范围内被 "使用"。

> This specification defines the following usage scopes:     
> * Outside of a pass (in GPUCommandEncoder), each (non-state-setting) command is one usage scope (e.g. copyBufferToTexture()).    
> * In a compute pass, each dispatch command (dispatchWorkgroups() or dispatchWorkgroupsIndirect()) is one usage scope. A subresource is "used" in the usage scope if it is potentially accessible by the command. Within a dispatch, for each bind group slot that is used by the current GPUComputePipeline's [[layout]], every subresource referenced by that bind group is "used" in the usage scope. State-setting compute pass commands, like setBindGroup(), do not contribute directly to a usage scope; they instead change the state that is checked in dispatch commands.      
> * One render pass is one usage scope. A subresource is "used" in the usage scope if it’s referenced by any (state-setting or non-state-setting) command. For example, in setBindGroup(), every subresource in bindGroup is "used" in the render pass’s usage scope.    

**待解问题 2** 上面使用范围描述也许应该和GPU一起讲一下，但是，我们还没有办法引用特定的GPU命令(例如调度)

 > **ISSUE 2** The above should probably talk about GPU commands. But we don’t have a way to reference specific GPU commands (like dispatch) yet.

**注意：** 上述规则意味着下面的例子中，资源使用被包含在使用范围验证中。
* 在渲染管道中，在任何setBindGroup()调用中使用的子资源，无论是当前绑定的管道的着色器或布局，实际依赖于这些绑定，或者这些绑定组是否被另一个 "set "调用所覆盖(影射)。
* 在任何setVertexBuffer()调用中所使用的缓冲区，无论任何绘图调用是否依赖于该缓冲区，或者该缓冲区是否被另一个'set'调用所覆盖(影射)。
* 在任何setIndexBuffer()调用中所使用的缓冲区，无论任何绘图调用是否依赖于该缓冲区，或者该缓冲区是否被另一个'set'调用所覆盖(影射)。
* 在beginRenderPass()调用中，用作GPURenderPassDescriptor中的颜色附件、解析附件、或深度/模式附件的纹理子资源，不管着色器是否实际依赖于这些附件。
* 在绑定组中使用的资源，其可见度为0，或者在渲染管道中使用时只能在计算阶段可见（反之亦然，即在计算管着中使用时只能在渲染阶段可见）。

> **Note:** The above rules mean the following example resource usages are included in usage scope validation:    
> * In a render pass, subresources used in any setBindGroup() call, regardless of whether the currently bound pipeline’s shader or layout actually depends on these bindings, or the bind group is shadowed by another 'set' call.
> * A buffer used in any setVertexBuffer() call, regardless of whether any draw call depends on this buffer, or this buffer is shadowed by another 'set' call.
> * A buffer used in any setIndexBuffer() call, regardless of whether any draw call depends on this buffer, or this buffer is shadowed by another 'set' call.
> * A texture subresource used as a color attachment, resolve attachment, or depth/stencil attachment in GPURenderPassDescriptor by beginRenderPass(), regardless of whether the shader actually depends on these attachments.
> * Resources used in bind group entries with visibility 0, or visible only to the compute stage but used in a render pass (or vice versa).

在命令编码期间，子资源的每一次使用都被记录在命令缓冲区的一个使用范围内。对于每个使用范围，WebGPU的具体实现要进行使用范围验证，通过合成列表里，在使用范围内的每个子资源的所有内部使用标志而进行验证。如果这些列表中的使用范围有任何一个是不兼容的，那么GPUCommandEncoder.finish()将产生一个验证错误。

> During command encoding, every usage of a subresource is recorded in one of the usage scopes in the command buffer. For each usage scope, the implementation performs usage scope validation by composing the list of all internal usage flags of each subresource used in the usage scope. If any of those lists is not a compatible usage list, GPUCommandEncoder.finish() will generate a validation error.

## 3.5 核心内部对象

> 3.5. Core Internal Objects

### 3.5.1.适配器
> 3.5.1. Adapters

适配器标识了系统中一个WebGPU的具体实现：既是浏览器平台上底层 计算/渲染 功能的实例，也是浏览器WebGPU在该功能顶层具体实现的实例。

适配器并不唯一地代表底层实现：多次调用requestAdapter()，每次都会返回一个不同的适配器对象。

> An adapter identifies an implementation of WebGPU on the system: both an instance of compute/rendering functionality on the platform underlying a browser, and an instance of a browser’s implementation of WebGPU on top of that functionality.

> Adapters do not uniquely represent underlying implementations: calling requestAdapter() multiple times returns a different adapter object each time.

一个适配器对象可能在任何时候变得无效。这发生在"丢失设备"和"标记适配器过期"里面。一个无效的适配器是无法创建新设备对象。

**注意：** 这种机制确保各种适配器的创建场景看起来与应用程序相似，因此它们可以很容易地以较少的测试来获得更多的更具鲁棒性的场景：首次初始化、由于未接入的适配器而导致的重新初始化、由于测试GPUDevice.destroy()调用而导致重新初始化等等。它还确保应用程序使用最新的系统状态来决定使用哪个适配器。

> An adapter object may become invalid at any time. This happens inside "lose the device" and "mark adapters stale". An invalid adapter is unable to vend new devices.

> **NOTE:** This mechanism ensures that various adapter-creation scenarios look similar to applications, so they can easily be robust to more scenarios with less testing: first initialization, reinitialization due to an unplugged adapter, reinitialization due to a test GPUDevice.destroy() call, etc. It also ensures applications use the latest system state to make decisions about which adapter to use.

如果一个适配器有明显的性能缺陷，以换取更广泛的兼容性、更可预测的行为或改进的隐私等，那么它可以被认为是一个备用适配器。并不要求每个系统上都有备用适配器。

> An adapter may be considered a fallback adapter if it has significant performance caveats in exchange for some combination of wider compatibility, more predictable behavior, or improved privacy. It is not required that a fallback adapter is available on every system.

>一个适配器有以下内部插槽。    
> * [[features]], 类型是有序集合<GPUFeatureName>, 只读    
>    这些features可用于在该适配器上创建设备。
> * [[limit]], 类型为所支持的限制值, 只读    
>    它的最佳限制可用于在此适配器上创建设备的。    
>    每个适配器的限制必须与所支持的限制中的默认值相同，甚至要更好。    
> * [[fallback]], 类型是布尔值
>    如果设置为"true"，表示该适配器是一个后备适配器。
> * [[unmaskedIdentifiers]], 类型为有序集合<DOMString>。
>    用户代理选择报告此适配器的GPUAdapterInfo字段中的名称列表。最初填入GPUAdapterInfo任意字段的名称，是用户代理选择报告的，且无需用户同意的名称集。

> An adapter has the following internal slots:
> * [[features]], of type ordered set<GPUFeatureName>, readonly    
>    The features which can be used to create devices on this adapter.
> * [[limits]], of type supported limits, readonly    
>    The best limits which can be used to create devices on this adapter.    
>    Each adapter limit must be the same or better than its default value in supported limits.    
> * [[fallback]], of type boolean    
>    If set to true indicates that the adapter is a fallback adapter.
> * [[unmaskedIdentifiers]], of type ordered set<DOMString>    
>    A list of names of GPUAdapterInfo fields the user agent has chosen to report for this adapter. Initially populated with the names of any GPUAdapterInfo fields the user agent has chosen to report without user consent.

适配器通过GPUAdapter暴露出来。
>  Adapters are exposed via GPUAdapter.

### 3.5.2.设备
> 3.5.2. Devices

一个设备是一个适配器的逻辑实例，通过它创建内部对象。它可以在多个代理之间共享（例如，专用工作线程）。

一个设备是由它所创建的所有内部对象的唯一所有者：当设备变得无效时（丢失或被销毁），它和它创建的所有对象（直接，如createTexture()，或间接，如createView()）都变成隐式地无法使用。

> A device is the logical instantiation of an adapter, through which internal objects are created. It can be shared across multiple agents (e.g. dedicated workers).

> A device is the exclusive owner of all internal objects created from it: when the device becomes invalid (is lost or destroyed), it and all objects created on it (directly, e.g. createTexture(), or indirectly, e.g. createView()) become implicitly unusable.

> 一个设备有以下内部插槽:    
> * [[adapter]]，类型为adapter，只读    
> 此适配器就是创建该设备的适配器。
> * [[features]], 类型为有序集合<GPUFeatureName>, 只读    
> 可以在这个设备上使用的功能。即使底层的适配器可以支持它们，没有声明在此的额外的功能也不能使用，
> * [[limit]], 类型为所支持的限制值 只读     
> 可以在这个设备上使用的限制。但没有声明在此的限制，即使底层适配器可以支持它们，也不能使用更好的限制，

> A device has the following internal slots:    
> * [[adapter]], of type adapter, readonly    
>   The adapter from which this device was created.    
> * [[features]], of type ordered set<GPUFeatureName>, readonly    
>    The features which can be used on this device. No additional features can be used, even if the underlying adapter can support them.
> * [[limits]], of type supported limits, readonly    
>    The limits which can be used on this device. No better limits can be used, even if the underlying adapter can support them.

当一个新的设备装置从具有GPUDeviceDescriptor描述符的adapter创建时:
* 将device.[[adapter]]设置为adapter。
* 将device.[[features]]设置为descriptor.requiredFeatures中的集合值。
* 让device.[[limit]]是一个支持的limit对象，具有默认值。对于descriptor.requiredLimits中的每对(key, value)值，将device.[[limits]]中与key对应的成员值设置为 value和supported limits中的默认值两者之中的较佳值。

> When a new device device is created from adapter adapter with GPUDeviceDescriptor descriptor:
> Set device.[[adapter]] to adapter.
> Set device.[[features]] to the set of values in descriptor.requiredFeatures.
> Let device.[[limits]] be a supported limits object with the default values. For each (key, value) pair in descriptor.requiredLimits, set the member corresponding to key in device.[[limits]] to the better value of value or the default value in supported limits.

任何时候，当用户代理需要撤销对一个设备的访问时，它就会调用设备时间轴上的丢失设备指令(device, undefined)，此操作可能会发生在该时间轴当前排队里其他操作之前。

如果一个操作失败，其副作用会明显地改变设备上的对象的状态，或者可能会破坏内部实现/驱动程序的状态，那么设备应该被丢失，以防止这些变化被观察到。

> Any time the user agent needs to revoke access to a device, it calls lose the device(device, undefined) on the device’s device timeline, potentially ahead of other operations currently queued on that timeline.

> If an operation fails with side effects that would observably change the state of objects on the device or potentially corrupt internal implementation/driver state, the device should be lost to prevent these changes from being observable.

> 丢失设备指令(device, reason)。   
> 1. 使device.[[adapter]] 变得无效。
> 2. 使device无效。
> 3. 让gpuDevice成为与device相对应的内容时间轴上GPUDevice。   
> **待决问题 3**  更严格地定义这个。   
> 4. 在gpuDevice的内容时间线上派发以下步骤：   
>     1. 用一个新的原因设置为形参中reason的GPUDeviceLostInfo来解决device.lost，消息设置为一个WebGPU具体实现所定义的值。   
>     **注意：** 消息不应该透露不必要的用户/系统信息，并且永远不要被应用程序解析。

> To lose the device(device, reason):
> 1. Make device.[[adapter]] invalid.    
> 2. Make device invalid.    
> 3. Let gpuDevice be the content timeline GPUDevice corresponding to device.     
> **ISSUE 3** Define this more rigorously.    
> 4. Issue the following steps on the content timeline of gpuDevice:
>    1. Resolve device.lost with a new GPUDeviceLostInfo with reason set to reason and message set to an implementation-defined value.    
>    **NOTE:**  message should not disclose unnecessary user/system information and should never be parsed by applications.

设备是通过GPUDevice暴露的。
> Devices are exposed via GPUDevice.

## 3.6.可选的能力
> 3.6. Optional Capabilities

WebGPU的适配器和设备都有能力(Capabilities)，这些能力描述了WebGPU不同具体实现方式之间功能差异，典型的，由于硬件或系统软件的限制。一个能力要么是一个特征，要么是一个限制。

关于隐私方面的考虑，请参见第2.2.1节 机器特定的特征和限制。

> WebGPU adapters and devices have capabilities, which describe WebGPU functionality that differs between different implementations, typically due to hardware or system software constraints. A capability is either a feature or a limit.

> For privacy considerations, see § 2.2.1 Machine-specific features and limits.

### 3.6.1.特征

3.6.1. Features

特性是一组可选的WebGPU功能，通常由于硬件或系统软件的限制，并不是在所有WebGPU具体实现中都要支持的。

每个GPUAdapter都会公开一组可用的特性。只有这些功能可以在requestDevice()中被请求。

> A feature is a set of optional WebGPU functionality that is not supported on all implementations, typically due to hardware or system software constraints.

> Each GPUAdapter exposes a set of available features. Only those features may be requested in requestDevice().

实用性是指，只有在设备创建时明确要求的功能（在requiredFeatures中），才可以被使用，这些功能通常只是所有支持的特征功能的一部分。另外，以别的方式使用现已存在的API的功能，通常会导致验证错误，而使用可选的API则会导致以下结果。
* 使用一个新的方法或枚举值总是抛出一个TypeError。
* 使用一个新的具有（正确类型的）非默认值的字典成员变量通常会导致验证错误。
* 使用一个新的WGSL使能指令总是导致createShaderModule()验证错误。

> Functionality that is part of a feature may only be used if the feature was requested at device creation (in requiredFeatures). Otherwise, using existing API surfaces in a new way typically results in a validation error, and using optional API surfaces results in the following:
> * Using a new method or enum value always throws a TypeError.
> * Using a new dictionary member with a (correctly-typed) non-default value typically results in a validation error.
> * Using a new WGSL enable directive always results in a createShaderModule() validation error.

当且仅当object.[[device]].[[features]]包含此feature时，一个GPUFeatureName的特征才会被GPUObjectBase对象启用。  
关于每个特性所启用的功能描述，请参见特性索引。

> A GPUFeatureName feature is enabled for a GPUObjectBase object if and only if object.[[device]].[[features]] contains feature.

> See the Feature Index for a description of the functionality each feature enables.



### 3.6.2. 限制

> 3.6.2. Limits

每个限制都是WebGPU对设备的使用上，一个数字限制。

一个支持的限制的对象对每个定义的限制都有一个数值。每个适配器(adapter)一组支持的限制，设备在创建时也有特定的支持限制。无论适配器的限制是什么，设备的限制都会被强制执行。

> Each limit is a numeric limit on the usage of WebGPU on a device.

> A supported limits object has a value for every defined limit. Each adapter has a set of supported limits, and devices are created with specific supported limits in place. The device limits are enforced regardless of the adapter’s limits.


每个限制都有一个默认值。每个适配器(adapter)都保证支持默认值的限制值或甚至更好的值。如果在要求限制(requiredLimits)中没有明确指定一个值，则使用默认值。

一个限制值可能比另一个更好。一个更好的限制值总是能放松验证，严格的使能，使得更多程序参数被验证。对于每个限制类，"更好的限制值"是有定义的。

> Each limit has a default value. Every adapter is guaranteed to support the default value or better. The default is used if a value is not explicitly specified in requiredLimits.

> One limit value may be better than another. A better limit value always relaxes validation, enabling strictly more programs to be valid. For each limit class, "better" is defined.

不同的限制值有不同的限制类。
> * 最大    
> 该限制要求传入API的是一类限制值的最大值。    
> 更大的限制值则更好。    
> 只能设置的限制值至少要大于等于(≥)默认值，较小的限制值会被钳位到默认值(就是低于默认值的限制值会被自动设置为默认值)。    
> * 对齐   
> 訪限制要求传入API的限制值必须强制和一些值的最小值对齐；也就是说，该值必须是可选限制值的倍数。    
> 较小的限制则更好。    
> 只能设置为2的幂，且要小于等于(≤)默认值，不是2的幂的值是无效的。较高的2次方的幂值会被钳制在默认值上。    


> Different limits have different limit classes:
> * maximum    
>    The limit enforces a maximum on some value passed into the API.    
>    Higher values are better.    
>    May only be set to values ≥ the default. Lower values are clamped to the default.    
> * alignment    
>    The limit enforces a minimum alignment on some value passed into the API; that is, the value must be a multiple of the limit.     
>    Lower values are better.    
>    May only be set to powers of 2 which are ≤ the default. Values which are not powers of 2 are invalid. Higher powers of 2 are clamped to the default.  

**注意：** 设置 "更好的 "限制值不一定是可取的，因为它们可能会影响性能。正因为如此，为了提高跨设备和具体实现的可移植性，应用程序通常应该请求对其工作有效的 "最差 "限制值（最好就是默认值）。

> **NOTE:** Setting "better" limits may not necessarily be desirable, as they may have a performance impact. Because of this, and to improve portability across devices and implementations, applications should generally request the "worst" limits that work for their content (ideally, the default values).

***下面又是一个麻烦的表格了，这个表格麻烦了，直接复制了html文本，然后在这个里面直接翻译了，表格中的双划线表示原文本一段描述的分隔，有些内容太影像原文意思的则不进行翻译处理了。***

<table class="data no-colspan-center">
    <thead>
     <tr>
      <th>Limit name 
      </th><th>Type 
      </th><th><a data-link-type="dfn" href="#limit-class" id="ref-for-limit-class①">Limit class</a> 
      </th><th><a data-link-type="dfn" href="#limit-default" id="ref-for-limit-default⑤">Default</a> 
    </th></tr></thead><tbody>
     <tr>
      <td><dfn class="idl-code" data-dfn-for="supported limits" data-dfn-type="attribute" data-export="" id="dom-supported-limits-maxtexturedimension1d"><code>maxTextureDimension1D</code><a class="self-link" href="#dom-supported-limits-maxtexturedimension1d"></a></dfn> 
      </td><td><code class="idl"><a data-link-type="idl" href="#typedefdef-gpusize32" id="ref-for-typedefdef-gpusize32">GPUSize32</a></code> 
      </td><td><a data-link-type="dfn" href="#limit-class-maximum" id="ref-for-limit-class-maximum">maximum</a> 
      </td><td>8192 
     </td></tr>
<tr>
  <td colspan="4">用"1d"维度来创建的纹理的size.width所允许的最大值。</td>
</tr>
     <tr class="row-continuation">
      <td colspan="4"> The maximum allowed value for the <code class="idl"><a data-link-type="idl" href="#dom-gputexturedescriptor-size" id="ref-for-dom-gputexturedescriptor-size">size</a></code>.<a data-link-type="dfn" href="#extent3d-width" id="ref-for-extent3d-width">width</a> of a <a data-link-type="dfn" href="#texture" id="ref-for-texture">texture</a> created with <code class="idl"><a data-link-type="idl" href="#dom-gputexturedescriptor-dimension" id="ref-for-dom-gputexturedescriptor-dimension">dimension</a></code> <code class="idl"><a data-link-type="idl" href="#dom-gputexturedimension-1d" id="ref-for-dom-gputexturedimension-1d">"1d"</a></code>. 
     </td></tr>
<tr>
  <td colspan="4"></td>
</tr>
     <tr>
      <td><dfn class="dfn-paneled idl-code" data-dfn-for="supported limits" data-dfn-type="attribute" data-export="" id="dom-supported-limits-maxtexturedimension2d"><code>maxTextureDimension2D</code></dfn> 
      </td><td><code class="idl"><a data-link-type="idl" href="#typedefdef-gpusize32" id="ref-for-typedefdef-gpusize32①">GPUSize32</a></code> 
      </td><td><a data-link-type="dfn" href="#limit-class-maximum" id="ref-for-limit-class-maximum①">maximum</a> 
      </td><td>8192 
     </td></tr>
<tr>
  <td colspan="4">用"2d"维度来创建的纹理的size.width和size.height所允许的最大值。</td>
</tr>
     <tr class="row-continuation">
      <td colspan="4"> The maximum allowed value for the <code class="idl"><a data-link-type="idl" href="#dom-gputexturedescriptor-size" id="ref-for-dom-gputexturedescriptor-size①">size</a></code>.<a data-link-type="dfn" href="#extent3d-width" id="ref-for-extent3d-width①">width</a> and <code class="idl"><a data-link-type="idl" href="#dom-gputexturedescriptor-size" id="ref-for-dom-gputexturedescriptor-size②">size</a></code>.<a data-link-type="dfn" href="#extent3d-height" id="ref-for-extent3d-height">height</a> of a <a data-link-type="dfn" href="#texture" id="ref-for-texture①">texture</a> created with <code class="idl"><a data-link-type="idl" href="#dom-gputexturedescriptor-dimension" id="ref-for-dom-gputexturedescriptor-dimension①">dimension</a></code> <code class="idl"><a data-link-type="idl" href="#dom-gputexturedimension-2d" id="ref-for-dom-gputexturedimension-2d">"2d"</a></code>. 
     </td></tr>
<tr>
  <td colspan="4"></td>
</tr>
<tr>
      <td><dfn class="idl-code" data-dfn-for="supported limits" data-dfn-type="attribute" data-export="" id="dom-supported-limits-maxtexturedimension3d"><code>maxTextureDimension3D</code><a class="self-link" href="#dom-supported-limits-maxtexturedimension3d"></a></dfn> 
      </td><td><code class="idl"><a data-link-type="idl" href="#typedefdef-gpusize32" id="ref-for-typedefdef-gpusize32②">GPUSize32</a></code> 
      </td><td><a data-link-type="dfn" href="#limit-class-maximum" id="ref-for-limit-class-maximum②">maximum</a> 
      </td><td>2048 
     </td></tr>
<tr>
  <td colspan="4">用"3d"维度来创建的纹理的size.width,size.height和size.depthOrArrayLayers所允许的最大值。</td>
</tr>
<tr class="row-continuation">
      <td colspan="4"> The maximum allowed value for the <code class="idl"><a data-link-type="idl" href="#dom-gputexturedescriptor-size" id="ref-for-dom-gputexturedescriptor-size③">size</a></code>.<a data-link-type="dfn" href="#extent3d-width" id="ref-for-extent3d-width②">width</a>, <code class="idl"><a data-link-type="idl" href="#dom-gputexturedescriptor-size" id="ref-for-dom-gputexturedescriptor-size④">size</a></code>.<a data-link-type="dfn" href="#extent3d-height" id="ref-for-extent3d-height①">height</a> and <code class="idl"><a data-link-type="idl" href="#dom-gputexturedescriptor-size" id="ref-for-dom-gputexturedescriptor-size⑤">size</a></code>.<a data-link-type="dfn" href="#extent3d-depthorarraylayers" id="ref-for-extent3d-depthorarraylayers">depthOrArrayLayers</a> of a <a data-link-type="dfn" href="#texture" id="ref-for-texture②">texture</a> created with <code class="idl"><a data-link-type="idl" href="#dom-gputexturedescriptor-dimension" id="ref-for-dom-gputexturedescriptor-dimension②">dimension</a></code> <code class="idl"><a data-link-type="idl" href="#dom-gputexturedimension-3d" id="ref-for-dom-gputexturedimension-3d">"3d"</a></code>. 
     </td></tr>
<tr>
  <td colspan="4"> </td>
</tr>
<tr>
      <td><dfn class="idl-code" data-dfn-for="supported limits" data-dfn-type="attribute" data-export="" id="dom-supported-limits-maxtexturearraylayers"><code>maxTextureArrayLayers</code><a class="self-link" href="#dom-supported-limits-maxtexturearraylayers"></a></dfn> 
      </td><td><code class="idl"><a data-link-type="idl" href="#typedefdef-gpusize32" id="ref-for-typedefdef-gpusize32③">GPUSize32</a></code> 
      </td><td><a data-link-type="dfn" href="#limit-class-maximum" id="ref-for-limit-class-maximum③">maximum</a> 
      </td><td>256 
     </td></tr>
<tr>
  <td colspan="4">用"2d"维度来创建的纹理的size.depthOrArrayLayers所允许的最大值。</td>
</tr>     
<tr class="row-continuation">
      <td colspan="4"> The maximum allowed value for the <code class="idl"><a data-link-type="idl" href="#dom-gputexturedescriptor-size" id="ref-for-dom-gputexturedescriptor-size⑥">size</a></code>.<a data-link-type="dfn" href="#extent3d-depthorarraylayers" id="ref-for-extent3d-depthorarraylayers①">depthOrArrayLayers</a> of a <a data-link-type="dfn" href="#texture" id="ref-for-texture③">texture</a> created with <code class="idl"><a data-link-type="idl" href="#dom-gputexturedescriptor-dimension" id="ref-for-dom-gputexturedescriptor-dimension③">dimension</a></code> <code class="idl"><a data-link-type="idl" href="#dom-gputexturedimension-2d" id="ref-for-dom-gputexturedimension-2d①">"2d"</a></code>. 
     </td></tr>
<tr>
  <td colspan="4"> </td>
</tr>
<tr>
      <td><dfn class="dfn-paneled idl-code" data-dfn-for="supported limits" data-dfn-type="attribute" data-export="" id="dom-supported-limits-maxbindgroups"><code>maxBindGroups</code></dfn> 
      </td><td><code class="idl"><a data-link-type="idl" href="#typedefdef-gpusize32" id="ref-for-typedefdef-gpusize32④">GPUSize32</a></code> 
      </td><td><a data-link-type="dfn" href="#limit-class-maximum" id="ref-for-limit-class-maximum④">maximum</a> 
      </td><td>4 
     </td></tr>
<tr>
  <td colspan="4">当创建GPUPipelineLayout时，bindGroupLayouts中所允许的GPUBindGroupLayouts的最大值。</td>
</tr>     
<tr class="row-continuation">
      <td colspan="4"> The maximum number of <code class="idl"><a data-link-type="idl" href="#gpubindgrouplayout" id="ref-for-gpubindgrouplayout">GPUBindGroupLayouts</a></code> allowed in <code class="idl"><a data-link-type="idl" href="#dom-gpupipelinelayoutdescriptor-bindgrouplayouts" id="ref-for-dom-gpupipelinelayoutdescriptor-bindgrouplayouts">bindGroupLayouts</a></code> when creating a <code class="idl"><a data-link-type="idl" href="#gpupipelinelayout" id="ref-for-gpupipelinelayout">GPUPipelineLayout</a></code>. 
     </td></tr>
<tr>
  <td colspan="4"> </td>
</tr>     
<tr>
      <td><dfn class="dfn-paneled idl-code" data-dfn-for="supported limits" data-dfn-type="attribute" data-export="" id="dom-supported-limits-maxbindingsperbindgroup"><code>maxBindingsPerBindGroup</code></dfn> 
      </td><td><code class="idl"><a data-link-type="idl" href="#typedefdef-gpusize32" id="ref-for-typedefdef-gpusize32⑤">GPUSize32</a></code> 
      </td><td><a data-link-type="dfn" href="#limit-class-maximum" id="ref-for-limit-class-maximum⑤">maximum</a> 
      </td><td>640 
     </td></tr>
<tr>
  <td colspan="4">
   <p>当创建GPUBindGroupLayout时， GPUBindGroupLayoutEntry.binding所允许的最大值。</p>
   <p>对于一个适配器，支持的限制值必须 ≥（ 每个着色器阶段的最大绑定值 × 每个管道的最大着色器阶段数量 ）。</p>
<p>每个着色器阶段的最大绑定值是（maxSampledTexturesPerShaderStage + maxSamplersPerShaderStage + maxStorageBuffersPerShaderStage + maxStorageTexturesPerShaderStage + maxUniformBuffersPerShaderStage）。</p>

<p>每个管道的最大着色器阶段数量是2，因为一个GPURenderPipeline同时支持顶点和片段着色器。</p>

<p>**注意：** 对于使用默认绑定插槽限制的人来说，640"应该是足够的"。</p>
</td>
</tr>     
<tr class="row-continuation">
      <td colspan="4">
        The maximum allowed number of <code class="idl"><a data-link-type="idl" href="#dictdef-gpubindgrouplayoutentry" id="ref-for-dictdef-gpubindgrouplayoutentry">GPUBindGroupLayoutEntry</a></code>.<code class="idl"><a data-link-type="idl" href="#dom-gpubindgrouplayoutentry-binding" id="ref-for-dom-gpubindgrouplayoutentry-binding">binding</a></code> values when creating a <code class="idl"><a data-link-type="idl" href="#gpubindgrouplayout" id="ref-for-gpubindgrouplayout①">GPUBindGroupLayout</a></code>. 
       <p>The <a data-link-type="dfn" href="#supported-limits" id="ref-for-supported-limits⑦">supported limit</a> for an <a data-link-type="dfn" href="#adapter" id="ref-for-adapter①⑥">adapter</a> must be ≥
        (<a data-link-type="dfn" href="#max-bindings-per-shader-stage" id="ref-for-max-bindings-per-shader-stage">max bindings per shader stage</a> × <a data-link-type="dfn" href="#max-shader-stages-per-pipeline" id="ref-for-max-shader-stages-per-pipeline">max shader stages per pipeline</a>) for the same <a data-link-type="dfn" href="#adapter" id="ref-for-adapter①⑦">adapter</a>.</p>
       <p><dfn class="dfn-paneled" data-dfn-for="" data-dfn-type="dfn" data-noexport="" id="max-bindings-per-shader-stage">max bindings per shader stage</dfn> is
        (<code class="idl"><a data-link-type="idl" href="#dom-supported-limits-maxsampledtexturespershaderstage" id="ref-for-dom-supported-limits-maxsampledtexturespershaderstage">maxSampledTexturesPerShaderStage</a></code> + <code class="idl"><a data-link-type="idl" href="#dom-supported-limits-maxsamplerspershaderstage" id="ref-for-dom-supported-limits-maxsamplerspershaderstage">maxSamplersPerShaderStage</a></code> + <code class="idl"><a data-link-type="idl" href="#dom-supported-limits-maxstoragebufferspershaderstage" id="ref-for-dom-supported-limits-maxstoragebufferspershaderstage">maxStorageBuffersPerShaderStage</a></code> + <code class="idl"><a data-link-type="idl" href="#dom-supported-limits-maxstoragetexturespershaderstage" id="ref-for-dom-supported-limits-maxstoragetexturespershaderstage">maxStorageTexturesPerShaderStage</a></code> + <code class="idl"><a data-link-type="idl" href="#dom-supported-limits-maxuniformbufferspershaderstage" id="ref-for-dom-supported-limits-maxuniformbufferspershaderstage">maxUniformBuffersPerShaderStage</a></code>).</p>
       <p><dfn class="dfn-paneled" data-dfn-for="" data-dfn-type="dfn" data-noexport="" id="max-shader-stages-per-pipeline">max shader stages per pipeline</dfn> is <code>2</code>, because a <code class="idl"><a data-link-type="idl" href="#gpurenderpipeline" id="ref-for-gpurenderpipeline②">GPURenderPipeline</a></code> supports both a vertex and fragment shader.</p>
       <p class="note" role="note"><span class="marker">Note:</span> 640 "ought to be enough for anybody" who is using the default <a data-link-type="dfn" href="#exceeds-the-binding-slot-limits" id="ref-for-exceeds-the-binding-slot-limits">binding slot limits</a>.</p>
     </td></tr>
<tr>
  <td colspan="4"> </td>
</tr>
<tr>
      <td><dfn class="dfn-paneled idl-code" data-dfn-for="supported limits" data-dfn-type="attribute" data-export="" id="dom-supported-limits-maxdynamicuniformbuffersperpipelinelayout"><code>maxDynamicUniformBuffersPerPipelineLayout</code></dfn> 
      </td><td><code class="idl"><a data-link-type="idl" href="#typedefdef-gpusize32" id="ref-for-typedefdef-gpusize32⑥">GPUSize32</a></code> 
      </td><td><a data-link-type="dfn" href="#limit-class-maximum" id="ref-for-limit-class-maximum⑥">maximum</a> 
      </td><td>8 
     </td></tr>
<tr>
  <td colspan="4">针对具有动态偏移的uniform缓冲区的GPUPipelineLayout，此限制条目说明跨越所有GPUPipelineLayout的GPUBindGroupLayoutEntry所允许的最大值。参见[超出绑定插槽的限制]。</td>
</tr>
<tr class="row-continuation">
      <td colspan="4"> The maximum number of <code class="idl"><a data-link-type="idl" href="#dictdef-gpubindgrouplayoutentry" id="ref-for-dictdef-gpubindgrouplayoutentry①">GPUBindGroupLayoutEntry</a></code> entries across a <code class="idl"><a data-link-type="idl" href="#gpupipelinelayout" id="ref-for-gpupipelinelayout①">GPUPipelineLayout</a></code> which are uniform buffers with dynamic offsets.
        See <a data-link-type="dfn" href="#exceeds-the-binding-slot-limits" id="ref-for-exceeds-the-binding-slot-limits①">Exceeds the binding slot limits</a>. 
     </td></tr>
<tr>
  <td colspan="4"> </td>
</tr>
<tr>
      <td><dfn class="dfn-paneled idl-code" data-dfn-for="supported limits" data-dfn-type="attribute" data-export="" id="dom-supported-limits-maxdynamicstoragebuffersperpipelinelayout"><code>maxDynamicStorageBuffersPerPipelineLayout</code></dfn> 
      </td><td><code class="idl"><a data-link-type="idl" href="#typedefdef-gpusize32" id="ref-for-typedefdef-gpusize32⑦">GPUSize32</a></code> 
      </td><td><a data-link-type="dfn" href="#limit-class-maximum" id="ref-for-limit-class-maximum⑦">maximum</a> 
      </td><td>4 
     </td></tr>
<tr>
  <td colspan="4">针对具有动态偏移的storage缓冲区的GPUPipelineLayout，此限制条目说明跨越所有GPUPipelineLayout的GPUBindGroupLayoutEntry所允许的最大值。参见[超出绑定插槽的限制]。</td>
</tr>
<tr class="row-continuation">
      <td colspan="4"> The maximum number of <code class="idl"><a data-link-type="idl" href="#dictdef-gpubindgrouplayoutentry" id="ref-for-dictdef-gpubindgrouplayoutentry②">GPUBindGroupLayoutEntry</a></code> entries across a <code class="idl"><a data-link-type="idl" href="#gpupipelinelayout" id="ref-for-gpupipelinelayout②">GPUPipelineLayout</a></code> which are storage buffers with dynamic offsets.
        See <a data-link-type="dfn" href="#exceeds-the-binding-slot-limits" id="ref-for-exceeds-the-binding-slot-limits②">Exceeds the binding slot limits</a>. 
     </td></tr>
<tr>
  <td colspan="4"> </td>
</tr>
     <tr>
      <td><dfn class="dfn-paneled idl-code" data-dfn-for="supported limits" data-dfn-type="attribute" data-export="" id="dom-supported-limits-maxsampledtexturespershaderstage"><code>maxSampledTexturesPerShaderStage</code></dfn> 
      </td><td><code class="idl"><a data-link-type="idl" href="#typedefdef-gpusize32" id="ref-for-typedefdef-gpusize32⑧">GPUSize32</a></code> 
      </td><td><a data-link-type="dfn" href="#limit-class-maximum" id="ref-for-limit-class-maximum⑧">maximum</a> 
      </td><td>16 
     </td></tr>
<tr>
  <td colspan="4">对于每个可能的GPUShaderStage阶段，针对采样纹理的 GPUPipelineLayout，跨越所有此类 GPUPipelineLayout 的GPUBindGroupLayoutEntry所充许的最大值。参见[超出绑定插槽的限制]。</td>
</tr>
<tr class="row-continuation">
      <td colspan="4"> For each possible <code class="idl"><a data-link-type="idl" href="#namespacedef-gpushaderstage" id="ref-for-namespacedef-gpushaderstage">GPUShaderStage</a></code> <code>stage</code>,
        the maximum number of <code class="idl"><a data-link-type="idl" href="#dictdef-gpubindgrouplayoutentry" id="ref-for-dictdef-gpubindgrouplayoutentry③">GPUBindGroupLayoutEntry</a></code> entries across a <code class="idl"><a data-link-type="idl" href="#gpupipelinelayout" id="ref-for-gpupipelinelayout③">GPUPipelineLayout</a></code> which are sampled textures.
        See <a data-link-type="dfn" href="#exceeds-the-binding-slot-limits" id="ref-for-exceeds-the-binding-slot-limits③">Exceeds the binding slot limits</a>. 
     </td></tr>
<tr>
  <td colspan="4"> </td>
</tr>
<tr>
      <td><dfn class="dfn-paneled idl-code" data-dfn-for="supported limits" data-dfn-type="attribute" data-export="" id="dom-supported-limits-maxsamplerspershaderstage"><code>maxSamplersPerShaderStage</code></dfn> 
      </td><td><code class="idl"><a data-link-type="idl" href="#typedefdef-gpusize32" id="ref-for-typedefdef-gpusize32⑨">GPUSize32</a></code> 
      </td><td><a data-link-type="dfn" href="#limit-class-maximum" id="ref-for-limit-class-maximum⑨">maximum</a> 
      </td><td>16 
     </td></tr>
<tr>
  <td colspan="4">对于每个可能的GPUShaderStage阶段，针对采样器(samplers)的 GPUPipelineLayout，跨越所有此类 GPUPipelineLayout 的GPUBindGroupLayoutEntry所充许的最大值。参见[超出绑定插槽的限制]。</td>
</tr>
<tr class="row-continuation">
      <td colspan="4"> For each possible <code class="idl"><a data-link-type="idl" href="#namespacedef-gpushaderstage" id="ref-for-namespacedef-gpushaderstage①">GPUShaderStage</a></code> <code>stage</code>,
        the maximum number of <code class="idl"><a data-link-type="idl" href="#dictdef-gpubindgrouplayoutentry" id="ref-for-dictdef-gpubindgrouplayoutentry④">GPUBindGroupLayoutEntry</a></code> entries across a <code class="idl"><a data-link-type="idl" href="#gpupipelinelayout" id="ref-for-gpupipelinelayout④">GPUPipelineLayout</a></code> which are samplers.
        See <a data-link-type="dfn" href="#exceeds-the-binding-slot-limits" id="ref-for-exceeds-the-binding-slot-limits④">Exceeds the binding slot limits</a>. 
     </td></tr>
<tr>
  <td colspan="4"> </td>
</tr>
<tr>
      <td><dfn class="dfn-paneled idl-code" data-dfn-for="supported limits" data-dfn-type="attribute" data-export="" id="dom-supported-limits-maxstoragebufferspershaderstage"><code>maxStorageBuffersPerShaderStage</code></dfn> 
      </td><td><code class="idl"><a data-link-type="idl" href="#typedefdef-gpusize32" id="ref-for-typedefdef-gpusize32①⓪">GPUSize32</a></code> 
      </td><td><a data-link-type="dfn" href="#limit-class-maximum" id="ref-for-limit-class-maximum①⓪">maximum</a> 
      </td><td>8 
     </td></tr>
<tr>
  <td colspan="4">对于每个可能的GPUShaderStage阶段，针对 存储缓冲区(storage buffers)的 GPUPipelineLayout，跨越所有此类 GPUPipelineLayout 的GPUBindGroupLayoutEntry所充许的最大值。参见[超出绑定插槽的限制]。</td>
</tr>
<tr class="row-continuation">
      <td colspan="4"> For each possible <code class="idl"><a data-link-type="idl" href="#namespacedef-gpushaderstage" id="ref-for-namespacedef-gpushaderstage②">GPUShaderStage</a></code> <code>stage</code>,
        the maximum number of <code class="idl"><a data-link-type="idl" href="#dictdef-gpubindgrouplayoutentry" id="ref-for-dictdef-gpubindgrouplayoutentry⑤">GPUBindGroupLayoutEntry</a></code> entries across a <code class="idl"><a data-link-type="idl" href="#gpupipelinelayout" id="ref-for-gpupipelinelayout⑤">GPUPipelineLayout</a></code> which are storage buffers.
        See <a data-link-type="dfn" href="#exceeds-the-binding-slot-limits" id="ref-for-exceeds-the-binding-slot-limits⑤">Exceeds the binding slot limits</a>. 
     </td></tr>
<tr>
  <td colspan="4"> </td>
</tr>
<tr>
      <td><dfn class="dfn-paneled idl-code" data-dfn-for="supported limits" data-dfn-type="attribute" data-export="" id="dom-supported-limits-maxstoragetexturespershaderstage"><code>maxStorageTexturesPerShaderStage</code></dfn> 
      </td><td><code class="idl"><a data-link-type="idl" href="#typedefdef-gpusize32" id="ref-for-typedefdef-gpusize32①①">GPUSize32</a></code> 
      </td><td><a data-link-type="dfn" href="#limit-class-maximum" id="ref-for-limit-class-maximum①①">maximum</a> 
      </td><td>4 
     </td></tr>
<tr>
  <td colspan="4">对于每个可能的GPUShaderStage阶段，针对存储纹理(storage textures)的 GPUPipelineLayout，跨越所有此类 GPUPipelineLayout 的GPUBindGroupLayoutEntry所充许的最大值。参见[超出绑定插槽的限制]。</td>
</tr>
<tr class="row-continuation">
      <td colspan="4"> For each possible <code class="idl"><a data-link-type="idl" href="#namespacedef-gpushaderstage" id="ref-for-namespacedef-gpushaderstage③">GPUShaderStage</a></code> <code>stage</code>,
        the maximum number of <code class="idl"><a data-link-type="idl" href="#dictdef-gpubindgrouplayoutentry" id="ref-for-dictdef-gpubindgrouplayoutentry⑥">GPUBindGroupLayoutEntry</a></code> entries across a <code class="idl"><a data-link-type="idl" href="#gpupipelinelayout" id="ref-for-gpupipelinelayout⑥">GPUPipelineLayout</a></code> which are storage textures.
        See <a data-link-type="dfn" href="#exceeds-the-binding-slot-limits" id="ref-for-exceeds-the-binding-slot-limits⑥">Exceeds the binding slot limits</a>. 
     </td></tr>
<tr>
  <td colspan="4"> </td>
</tr>
<tr>
      <td><dfn class="dfn-paneled idl-code" data-dfn-for="supported limits" data-dfn-type="attribute" data-export="" id="dom-supported-limits-maxuniformbufferspershaderstage"><code>maxUniformBuffersPerShaderStage</code></dfn> 
      </td><td><code class="idl"><a data-link-type="idl" href="#typedefdef-gpusize32" id="ref-for-typedefdef-gpusize32①②">GPUSize32</a></code> 
      </td><td><a data-link-type="dfn" href="#limit-class-maximum" id="ref-for-limit-class-maximum①②">maximum</a> 
      </td><td>12 
     </td></tr>
<tr>
  <td colspan="4">对于每个可能的GPUShaderStage阶段，针对uniform缓冲区(uniform buffers)的 GPUPipelineLayout，跨越所有此类 GPUPipelineLayout 的GPUBindGroupLayoutEntry所充许的最大值。参见[超出绑定插槽的限制]。</td>
</tr>
<tr class="row-continuation">
      <td colspan="4"> For each possible <code class="idl"><a data-link-type="idl" href="#namespacedef-gpushaderstage" id="ref-for-namespacedef-gpushaderstage④">GPUShaderStage</a></code> <code>stage</code>,
        the maximum number of <code class="idl"><a data-link-type="idl" href="#dictdef-gpubindgrouplayoutentry" id="ref-for-dictdef-gpubindgrouplayoutentry⑦">GPUBindGroupLayoutEntry</a></code> entries across a <code class="idl"><a data-link-type="idl" href="#gpupipelinelayout" id="ref-for-gpupipelinelayout⑦">GPUPipelineLayout</a></code> which are uniform buffers.
        See <a data-link-type="dfn" href="#exceeds-the-binding-slot-limits" id="ref-for-exceeds-the-binding-slot-limits⑦">Exceeds the binding slot limits</a>. 
     </td></tr>
<tr>
  <td colspan="4"> </td>
</tr>
<tr>
      <td><dfn class="dfn-paneled idl-code" data-dfn-for="supported limits" data-dfn-type="attribute" data-export="" id="dom-supported-limits-maxuniformbufferbindingsize"><code>maxUniformBufferBindingSize</code></dfn> 
      </td><td><code class="idl"><a data-link-type="idl" href="#typedefdef-gpusize64" id="ref-for-typedefdef-gpusize64">GPUSize64</a></code> 
      </td><td><a data-link-type="dfn" href="#limit-class-maximum" id="ref-for-limit-class-maximum①③">maximum</a> 
      </td><td>65536 bytes 
     </td></tr>
<tr>
  <td colspan="4">此条目限制最大GPUBufferBinding.size，用于限制绑定的GPUBindGroupLayoutEntry的entry，其中entry.buffer?.type为"uniform"。</td>
</tr>
<tr class="row-continuation">
      <td colspan="4"> The maximum <code class="idl"><a data-link-type="idl" href="#dictdef-gpubufferbinding" id="ref-for-dictdef-gpubufferbinding①">GPUBufferBinding</a></code>.<code class="idl"><a data-link-type="idl" href="#dom-gpubufferbinding-size" id="ref-for-dom-gpubufferbinding-size">size</a></code> for bindings with a <code class="idl"><a data-link-type="idl" href="#dictdef-gpubindgrouplayoutentry" id="ref-for-dictdef-gpubindgrouplayoutentry⑧">GPUBindGroupLayoutEntry</a></code> <var>entry</var> for which <var>entry</var>.<code class="idl"><a data-link-type="idl" href="#dom-gpubindgrouplayoutentry-buffer" id="ref-for-dom-gpubindgrouplayoutentry-buffer">buffer</a></code>?.<code class="idl"><a data-link-type="idl" href="#dom-gpubufferbindinglayout-type" id="ref-for-dom-gpubufferbindinglayout-type">type</a></code> is <code class="idl"><a data-link-type="idl" href="#dom-gpubufferbindingtype-uniform" id="ref-for-dom-gpubufferbindingtype-uniform①">"uniform"</a></code>. 
     </td></tr>
<tr>
  <td colspan="4"> </td>
</tr>
<tr>
      <td><dfn class="dfn-paneled idl-code" data-dfn-for="supported limits" data-dfn-type="attribute" data-export="" id="dom-supported-limits-maxstoragebufferbindingsize"><code>maxStorageBufferBindingSize</code></dfn> 
      </td><td><code class="idl"><a data-link-type="idl" href="#typedefdef-gpusize64" id="ref-for-typedefdef-gpusize64①">GPUSize64</a></code> 
      </td><td><a data-link-type="dfn" href="#limit-class-maximum" id="ref-for-limit-class-maximum①④">maximum</a> 
      </td><td> 134217728 bytes (128 MiB) 
     </td></tr>
<tr>
  <td colspan="4">此条目限制最大GPUBufferBinding.size，用于限制绑定的GPUBindGroupLayoutEntry的entry，其中，关于entry的类型，要么是entry.buffer?.type为"storage"，要么为"read-only-storage"。</td>
</tr>
<tr class="row-continuation">
      <td colspan="4"> The maximum <code class="idl"><a data-link-type="idl" href="#dictdef-gpubufferbinding" id="ref-for-dictdef-gpubufferbinding②">GPUBufferBinding</a></code>.<code class="idl"><a data-link-type="idl" href="#dom-gpubufferbinding-size" id="ref-for-dom-gpubufferbinding-size①">size</a></code> for bindings with a <code class="idl"><a data-link-type="idl" href="#dictdef-gpubindgrouplayoutentry" id="ref-for-dictdef-gpubindgrouplayoutentry⑨">GPUBindGroupLayoutEntry</a></code> <var>entry</var> for which <var>entry</var>.<code class="idl"><a data-link-type="idl" href="#dom-gpubindgrouplayoutentry-buffer" id="ref-for-dom-gpubindgrouplayoutentry-buffer①">buffer</a></code>?.<code class="idl"><a data-link-type="idl" href="#dom-gpubufferbindinglayout-type" id="ref-for-dom-gpubufferbindinglayout-type①">type</a></code> is <code class="idl"><a data-link-type="idl" href="#dom-gpubufferbindingtype-storage" id="ref-for-dom-gpubufferbindingtype-storage">"storage"</a></code> or <code class="idl"><a data-link-type="idl" href="#dom-gpubufferbindingtype-read-only-storage" id="ref-for-dom-gpubufferbindingtype-read-only-storage">"read-only-storage"</a></code>. 
     </td></tr>
<tr>
  <td colspan="4"> </td>
</tr>
<tr>
      <td><dfn class="dfn-paneled idl-code" data-dfn-for="supported limits" data-dfn-type="attribute" data-export="" id="dom-supported-limits-minuniformbufferoffsetalignment"><code>minUniformBufferOffsetAlignment</code></dfn> 
      </td><td><code class="idl"><a data-link-type="idl" href="#typedefdef-gpusize32" id="ref-for-typedefdef-gpusize32①③">GPUSize32</a></code> 
      </td><td><a data-link-type="dfn" href="#limit-class-alignment" id="ref-for-limit-class-alignment">alignment</a> 
      </td><td>256 bytes 
     </td></tr>
<tr>
  <td colspan="4">
  <p>此条目限制GPUBufferBinding.offset以及setBindGroup()中提供的动态偏移设置所需的对齐方式，用于限制绑定的GPUBindGroupLayoutEntry的entry，其中，关于entry的类型，即entry.buffer?.type为"uniform"</p>

  <p>具体的WebGPU实现必须支持大于等于32字节的值。</p>

  <p>注意：32字节将是vec4<f64>的对齐方式。参见[WebGPU着色语言规范 §5.3.6.1 对齐和大小]。</p>
  </td>
</tr>
<tr class="row-continuation">
      <td colspan="4">
        The required alignment for <code class="idl"><a data-link-type="idl" href="#dictdef-gpubufferbinding" id="ref-for-dictdef-gpubufferbinding③">GPUBufferBinding</a></code>.<code class="idl"><a data-link-type="idl" href="#dom-gpubufferbinding-offset" id="ref-for-dom-gpubufferbinding-offset">offset</a></code> and
        the dynamic offsets provided in <a data-link-type="dfn" href="#gpubindingcommandsmixin-setbindgroup" id="ref-for-gpubindingcommandsmixin-setbindgroup③">setBindGroup()</a>,
        for bindings with a <code class="idl"><a data-link-type="idl" href="#dictdef-gpubindgrouplayoutentry" id="ref-for-dictdef-gpubindgrouplayoutentry①⓪">GPUBindGroupLayoutEntry</a></code> <var>entry</var> for which <var>entry</var>.<code class="idl"><a data-link-type="idl" href="#dom-gpubindgrouplayoutentry-buffer" id="ref-for-dom-gpubindgrouplayoutentry-buffer②">buffer</a></code>?.<code class="idl"><a data-link-type="idl" href="#dom-gpubufferbindinglayout-type" id="ref-for-dom-gpubufferbindinglayout-type②">type</a></code> is <code class="idl"><a data-link-type="idl" href="#dom-gpubufferbindingtype-uniform" id="ref-for-dom-gpubufferbindingtype-uniform②">"uniform"</a></code>. 
       <p>Implementations <strong>must not</strong> support values less than 32 bytes.</p>
       <p class="note" role="note"><span class="marker">Note:</span> 32 bytes would be the alignment of <code>vec4&lt;f64&gt;</code>. See <a href="https://www.w3.org/TR/WGSL/#alignment-and-size"><cite>WebGPU Shading Language</cite> § 5.3.6.1 Alignment and Size</a>.</p>
     </td></tr>
<tr>
  <td colspan="4"> </td>
</tr>
<tr>
      <td><dfn class="dfn-paneled idl-code" data-dfn-for="supported limits" data-dfn-type="attribute" data-export="" id="dom-supported-limits-minstoragebufferoffsetalignment"><code>minStorageBufferOffsetAlignment</code></dfn> 
      </td><td><code class="idl"><a data-link-type="idl" href="#typedefdef-gpusize32" id="ref-for-typedefdef-gpusize32①④">GPUSize32</a></code> 
      </td><td><a data-link-type="dfn" href="#limit-class-alignment" id="ref-for-limit-class-alignment①">alignment</a> 
      </td><td>256 bytes 
     </td></tr>
<tr>
  <td colspan="4">
  <p>此条目限制GPUBufferBinding.offset以及setBindGroup()中提供的动态偏移设置所需的对齐方式，用于限制绑定的GPUBindGroupLayoutEntry的entry，其中，关于entry的类型，要么是entry.buffer?.type为"storage"，要么为"read-only-storage"。</p>

  <p>具体的WebGPU实现必须支持大于等于32字节的值。</p>

  <p>注意：32字节将是vec4<f64>的对齐方式。参见[WebGPU着色语言规范 §5.3.6.1 对齐和大小]。</p>
  </td>
</tr>
<tr class="row-continuation">
      <td colspan="4">
        The required alignment for <code class="idl"><a data-link-type="idl" href="#dictdef-gpubufferbinding" id="ref-for-dictdef-gpubufferbinding④">GPUBufferBinding</a></code>.<code class="idl"><a data-link-type="idl" href="#dom-gpubufferbinding-offset" id="ref-for-dom-gpubufferbinding-offset①">offset</a></code> and
        the dynamic offsets provided in <a data-link-type="dfn" href="#gpubindingcommandsmixin-setbindgroup" id="ref-for-gpubindingcommandsmixin-setbindgroup④">setBindGroup()</a>,
        for bindings with a <code class="idl"><a data-link-type="idl" href="#dictdef-gpubindgrouplayoutentry" id="ref-for-dictdef-gpubindgrouplayoutentry①①">GPUBindGroupLayoutEntry</a></code> <var>entry</var> for which <var>entry</var>.<code class="idl"><a data-link-type="idl" href="#dom-gpubindgrouplayoutentry-buffer" id="ref-for-dom-gpubindgrouplayoutentry-buffer③">buffer</a></code>?.<code class="idl"><a data-link-type="idl" href="#dom-gpubufferbindinglayout-type" id="ref-for-dom-gpubufferbindinglayout-type③">type</a></code> is <code class="idl"><a data-link-type="idl" href="#dom-gpubufferbindingtype-storage" id="ref-for-dom-gpubufferbindingtype-storage①">"storage"</a></code> or <code class="idl"><a data-link-type="idl" href="#dom-gpubufferbindingtype-read-only-storage" id="ref-for-dom-gpubufferbindingtype-read-only-storage①">"read-only-storage"</a></code>. 
       <p>Implementations <strong>must not</strong> support values less than 32 bytes.</p>
       <p class="note" role="note"><span class="marker">Note:</span> 32 bytes would be the alignment of <code>vec4&lt;f64&gt;</code>. See <a href="https://www.w3.org/TR/WGSL/#alignment-and-size"><cite>WebGPU Shading Language</cite> § 5.3.6.1 Alignment and Size</a>.</p>
     </td></tr>
<tr>
  <td colspan="4"> </td>
</tr>
<tr>
      <td><dfn class="dfn-paneled idl-code" data-dfn-for="supported limits" data-dfn-type="attribute" data-export="" id="dom-supported-limits-maxvertexbuffers"><code>maxVertexBuffers</code></dfn> 
      </td><td><code class="idl"><a data-link-type="idl" href="#typedefdef-gpusize32" id="ref-for-typedefdef-gpusize32①⑤">GPUSize32</a></code> 
      </td><td><a data-link-type="dfn" href="#limit-class-maximum" id="ref-for-limit-class-maximum①⑤">maximum</a> 
      </td><td>8 
     </td></tr>
<tr>
  <td colspan="4"> </td>
</tr>
<tr class="row-continuation">
      <td colspan="4"> The maximum number of <code class="idl"><a data-link-type="idl" href="#dom-gpuvertexstate-buffers" id="ref-for-dom-gpuvertexstate-buffers">buffers</a></code> when creating a <code class="idl"><a data-link-type="idl" href="#gpurenderpipeline" id="ref-for-gpurenderpipeline③">GPURenderPipeline</a></code>. 
     </td></tr>
<tr>
  <td colspan="4">当创建GPURenderPipeline时缓冲区的最大限制值</td>
</tr>
<tr>
      <td><dfn class="dfn-paneled idl-code" data-dfn-for="supported limits" data-dfn-type="attribute" data-export="" id="dom-supported-limits-maxbuffersize"><code>maxBufferSize</code></dfn> 
      </td><td><code class="idl"><a data-link-type="idl" href="#typedefdef-gpusize64" id="ref-for-typedefdef-gpusize64②">GPUSize64</a></code> 
      </td><td><a data-link-type="dfn" href="#limit-class-maximum" id="ref-for-limit-class-maximum①⑥">maximum</a> 
      </td><td>268435456 bytes (256 MiB) 
     </td></tr>
<tr>
  <td colspan="4"> <p>当创建GPUBuffer时，其尺寸的最大限制值。</p>
  <p>一个适配器所支持的限制值必须是 ≥ maxUniformBufferBindingSize 且 ≥ maxStorageBufferBindingSize。</p>

  </td>
</tr>
<tr class="row-continuation">
      <td colspan="4">
        The maximum size of <code class="idl"><a data-link-type="idl" href="#dom-gpubufferdescriptor-size" id="ref-for-dom-gpubufferdescriptor-size">size</a></code> when creating a <code class="idl"><a data-link-type="idl" href="#gpubuffer" id="ref-for-gpubuffer⑦">GPUBuffer</a></code>. 
       <p>The <a data-link-type="dfn" href="#supported-limits" id="ref-for-supported-limits⑧">supported limit</a> for an <a data-link-type="dfn" href="#adapter" id="ref-for-adapter①⑧">adapter</a> must be
        ≥ <code class="idl"><a data-link-type="idl" href="#dom-supported-limits-maxuniformbufferbindingsize" id="ref-for-dom-supported-limits-maxuniformbufferbindingsize">maxUniformBufferBindingSize</a></code> and
        ≥ <code class="idl"><a data-link-type="idl" href="#dom-supported-limits-maxstoragebufferbindingsize" id="ref-for-dom-supported-limits-maxstoragebufferbindingsize">maxStorageBufferBindingSize</a></code>.</p>
     </td></tr>
<tr>
  <td colspan="4"> </td>
</tr>
<tr>
      <td><dfn class="dfn-paneled idl-code" data-dfn-for="supported limits" data-dfn-type="attribute" data-export="" id="dom-supported-limits-maxvertexattributes"><code>maxVertexAttributes</code></dfn> 
      </td><td><code class="idl"><a data-link-type="idl" href="#typedefdef-gpusize32" id="ref-for-typedefdef-gpusize32①⑥">GPUSize32</a></code> 
      </td><td><a data-link-type="dfn" href="#limit-class-maximum" id="ref-for-limit-class-maximum①⑦">maximum</a> 
      </td><td>16 
     </td></tr>
<tr>
  <td colspan="4">
 当创建GPURenderPipeline时，跨越所有缓冲区的属性数量的最大限制值。 
  </td>
</tr>
<tr class="row-continuation">
      <td colspan="4"> The maximum number of <code class="idl"><a data-link-type="idl" href="#dom-gpuvertexbufferlayout-attributes" id="ref-for-dom-gpuvertexbufferlayout-attributes">attributes</a></code> in total across <code class="idl"><a data-link-type="idl" href="#dom-gpuvertexstate-buffers" id="ref-for-dom-gpuvertexstate-buffers①">buffers</a></code> when creating a <code class="idl"><a data-link-type="idl" href="#gpurenderpipeline" id="ref-for-gpurenderpipeline④">GPURenderPipeline</a></code>. 
     </td></tr>
<tr>
  <td colspan="4"> </td>
</tr>
<tr>
      <td><dfn class="dfn-paneled idl-code" data-dfn-for="supported limits" data-dfn-type="attribute" data-export="" id="dom-supported-limits-maxvertexbufferarraystride"><code>maxVertexBufferArrayStride</code></dfn> 
      </td><td><code class="idl"><a data-link-type="idl" href="#typedefdef-gpusize32" id="ref-for-typedefdef-gpusize32①⑦">GPUSize32</a></code> 
      </td><td><a data-link-type="dfn" href="#limit-class-maximum" id="ref-for-limit-class-maximum①⑧">maximum</a> 
      </td><td>2048 bytes 
     </td></tr>
<tr>
  <td colspan="4">当创建GPURenderPipeline时，允许的最大数组跨度(数组宽度？ arrayStride)。</td>
</tr>
<tr class="row-continuation">
      <td colspan="4"> The maximum allowed <code class="idl"><a data-link-type="idl" href="#dom-gpuvertexbufferlayout-arraystride" id="ref-for-dom-gpuvertexbufferlayout-arraystride">arrayStride</a></code> when creating a <code class="idl"><a data-link-type="idl" href="#gpurenderpipeline" id="ref-for-gpurenderpipeline⑤">GPURenderPipeline</a></code>. 
     </td></tr>
<tr>
  <td colspan="4"> </td>
</tr>
<tr>
      <td><dfn class="dfn-paneled idl-code" data-dfn-for="supported limits" data-dfn-type="attribute" data-export="" id="dom-supported-limits-maxinterstageshadercomponents"><code>maxInterStageShaderComponents</code></dfn> 
      </td><td><code class="idl"><a data-link-type="idl" href="#typedefdef-gpusize32" id="ref-for-typedefdef-gpusize32①⑧">GPUSize32</a></code> 
      </td><td><a data-link-type="dfn" href="#limit-class-maximum" id="ref-for-limit-class-maximum①⑨">maximum</a> 
      </td><td>60 
     </td></tr>
<tr>
  <td colspan="4">用于限制组件里的，内部阶段间通信用的输入或输出变量的最大允许值（如顶点着色器的输出或片段着色器的输入）。</td>
</tr>
<tr class="row-continuation">
      <td colspan="4"> The maximum allowed number of components of input or output variables
        for inter-stage communication (like vertex outputs or fragment inputs). 
     </td></tr>
<tr>
  <td colspan="4"> </td>
</tr>
<tr>
      <td><dfn class="dfn-paneled idl-code" data-dfn-for="supported limits" data-dfn-type="attribute" data-export="" id="dom-supported-limits-maxinterstageshadervariables"><code>maxInterStageShaderVariables</code></dfn> 
      </td><td><code class="idl"><a data-link-type="idl" href="#typedefdef-gpusize32" id="ref-for-typedefdef-gpusize32①⑨">GPUSize32</a></code> 
      </td><td><a data-link-type="dfn" href="#limit-class-maximum" id="ref-for-limit-class-maximum②⓪">maximum</a> 
      </td><td>16 
     </td></tr>
<tr>
  <td colspan="4">用于限制 内部阶段间通信用的输入或输出变量的最大允许值（如顶点着色器的输出或片段着色器的输入）。 </td>
</tr>
<tr class="row-continuation">
      <td colspan="4"> The maximum allowed number of input or output variables for inter-stage
        communication (like vertex outputs or fragment inputs). 
     </td></tr>
<tr>
  <td colspan="4"> </td>
</tr>
<tr>
      <td><dfn class="dfn-paneled idl-code" data-dfn-for="supported limits" data-dfn-type="attribute" data-export="" id="dom-supported-limits-maxcolorattachments"><code>maxColorAttachments</code></dfn> 
      </td><td><code class="idl"><a data-link-type="idl" href="#typedefdef-gpusize32" id="ref-for-typedefdef-gpusize32②⓪">GPUSize32</a></code> 
      </td><td><a data-link-type="dfn" href="#limit-class-maximum" id="ref-for-limit-class-maximum②①">maximum</a> 
      </td><td>8 
     </td></tr>
<tr>
  <td colspan="4">在 GPURenderPipelineDescriptor.fragment.target， GPURenderPassDescriptor.colorAttachments 和 GPURenderPassLayout.colorFormats 中允许的最大颜色附件数量。</td>
</tr>
<tr class="row-continuation">
      <td colspan="4"> The maximum allowed number of color attachments in <code class="idl"><a data-link-type="idl" href="#dictdef-gpurenderpipelinedescriptor" id="ref-for-dictdef-gpurenderpipelinedescriptor">GPURenderPipelineDescriptor</a></code>.<code class="idl"><a data-link-type="idl" href="#dom-gpurenderpipelinedescriptor-fragment" id="ref-for-dom-gpurenderpipelinedescriptor-fragment">fragment</a></code>.<code class="idl"><a data-link-type="idl" href="#dom-gpufragmentstate-targets" id="ref-for-dom-gpufragmentstate-targets">targets</a></code>, <code class="idl"><a data-link-type="idl" href="#dictdef-gpurenderpassdescriptor" id="ref-for-dictdef-gpurenderpassdescriptor①">GPURenderPassDescriptor</a></code>.<code class="idl"><a data-link-type="idl" href="#dom-gpurenderpassdescriptor-colorattachments" id="ref-for-dom-gpurenderpassdescriptor-colorattachments">colorAttachments</a></code>,
        and <code class="idl"><a data-link-type="idl" href="#dictdef-gpurenderpasslayout" id="ref-for-dictdef-gpurenderpasslayout">GPURenderPassLayout</a></code>.<code class="idl"><a data-link-type="idl" href="#dom-gpurenderpasslayout-colorformats" id="ref-for-dom-gpurenderpasslayout-colorformats">colorFormats</a></code>. 
     </td></tr>
<tr>
  <td colspan="4"> </td>
</tr>
<tr>
      <td><dfn class="dfn-paneled idl-code" data-dfn-for="supported limits" data-dfn-type="attribute" data-export="" id="dom-supported-limits-maxcolorattachmentbytespersample"><code>maxColorAttachmentBytesPerSample</code></dfn> 
      </td><td><code class="idl"><a data-link-type="idl" href="#typedefdef-gpusize32" id="ref-for-typedefdef-gpusize32②①">GPUSize32</a></code> 
      </td><td><a data-link-type="dfn" href="#limit-class-maximum" id="ref-for-limit-class-maximum②②">maximum</a> 
      </td><td>32 
     </td></tr>
<tr>
  <td colspan="4">用于限制跨越所有颜色附件里，渲染管道输出数据的一个样本（像素或子像素）所需的最大字节数。</td>
</tr>
<tr class="row-continuation">
      <td colspan="4"> The maximum number of bytes necessary to hold one sample (pixel or subpixel)
        of render pipeline output data, across all color attachments. 
     </td></tr>
<tr>
  <td colspan="4"> </td>
</tr>
<tr>
      <td><dfn class="dfn-paneled idl-code" data-dfn-for="supported limits" data-dfn-type="attribute" data-export="" id="dom-supported-limits-maxcomputeworkgroupstoragesize"><code>maxComputeWorkgroupStorageSize</code></dfn> 
      </td><td><code class="idl"><a data-link-type="idl" href="#typedefdef-gpusize32" id="ref-for-typedefdef-gpusize32②②">GPUSize32</a></code> 
      </td><td><a data-link-type="dfn" href="#limit-class-maximum" id="ref-for-limit-class-maximum②③">maximum</a> 
      </td><td>16384 bytes 
     </td></tr>
<tr>
  <td colspan="4">用于限制计算阶段的 GPUShaderModule 入口点的最大字节数。</td>
</tr>
<tr class="row-continuation">
      <td colspan="4"> The maximum number of bytes used for a compute stage <code class="idl"><a data-link-type="idl" href="#gpushadermodule" id="ref-for-gpushadermodule③">GPUShaderModule</a></code> entry-point. 
     </td></tr>
<tr>
  <td colspan="4"> </td>
</tr>
<tr>
      <td><dfn class="dfn-paneled idl-code" data-dfn-for="supported limits" data-dfn-type="attribute" data-export="" id="dom-supported-limits-maxcomputeinvocationsperworkgroup"><code>maxComputeInvocationsPerWorkgroup</code></dfn> 
      </td><td><code class="idl"><a data-link-type="idl" href="#typedefdef-gpusize32" id="ref-for-typedefdef-gpusize32②③">GPUSize32</a></code> 
      </td><td><a data-link-type="dfn" href="#limit-class-maximum" id="ref-for-limit-class-maximum②④">maximum</a> 
      </td><td>256 
     </td></tr>
<tr>
  <td colspan="4">用于限制计算阶段 GPUShaderModule 入口点的 workgroup_size尺寸乘积的最大值。</td>
</tr>
<tr class="row-continuation">
      <td colspan="4"> The maximum value of the product of the <code>workgroup_size</code> dimensions for a
        compute stage <code class="idl"><a data-link-type="idl" href="#gpushadermodule" id="ref-for-gpushadermodule④">GPUShaderModule</a></code> entry-point. 
     </td></tr>
<tr>
  <td colspan="4"> </td>
</tr>
<tr>
      <td><dfn class="dfn-paneled idl-code" data-dfn-for="supported limits" data-dfn-type="attribute" data-export="" id="dom-supported-limits-maxcomputeworkgroupsizex"><code>maxComputeWorkgroupSizeX</code></dfn> 
      </td><td><code class="idl"><a data-link-type="idl" href="#typedefdef-gpusize32" id="ref-for-typedefdef-gpusize32②④">GPUSize32</a></code> 
      </td><td><a data-link-type="dfn" href="#limit-class-maximum" id="ref-for-limit-class-maximum②⑤">maximum</a> 
      </td><td>256 
     </td></tr>
<tr>
  <td colspan="4">用于限制计算阶段 GPUShaderModule 入口点的 workgroup_size 的 X 维度的最大值。</td>
</tr>
<tr class="row-continuation">
      <td colspan="4"> The maximum value of the <code>workgroup_size</code> X dimension for a
        compute stage <code class="idl"><a data-link-type="idl" href="#gpushadermodule" id="ref-for-gpushadermodule⑤">GPUShaderModule</a></code> entry-point. 
     </td></tr>
<tr>
  <td colspan="4"> </td>
</tr>
<tr>
      <td><dfn class="dfn-paneled idl-code" data-dfn-for="supported limits" data-dfn-type="attribute" data-export="" id="dom-supported-limits-maxcomputeworkgroupsizey"><code>maxComputeWorkgroupSizeY</code></dfn> 
      </td><td><code class="idl"><a data-link-type="idl" href="#typedefdef-gpusize32" id="ref-for-typedefdef-gpusize32②⑤">GPUSize32</a></code> 
      </td><td><a data-link-type="dfn" href="#limit-class-maximum" id="ref-for-limit-class-maximum②⑥">maximum</a> 
      </td><td>256 
     </td></tr>
<tr>
  <td colspan="4">用于限制计算阶段 GPUShaderModule 入口点的 workgroup_size 的 Y 维度的最大值。</td>
</tr>
<tr class="row-continuation">
      <td colspan="4"> The maximum value of the <code>workgroup_size</code> Y dimensions for a
        compute stage <code class="idl"><a data-link-type="idl" href="#gpushadermodule" id="ref-for-gpushadermodule⑥">GPUShaderModule</a></code> entry-point. 
     </td></tr>
<tr>
  <td colspan="4"> </td>
</tr>
<tr>
      <td><dfn class="dfn-paneled idl-code" data-dfn-for="supported limits" data-dfn-type="attribute" data-export="" id="dom-supported-limits-maxcomputeworkgroupsizez"><code>maxComputeWorkgroupSizeZ</code></dfn> 
      </td><td><code class="idl"><a data-link-type="idl" href="#typedefdef-gpusize32" id="ref-for-typedefdef-gpusize32②⑥">GPUSize32</a></code> 
      </td><td><a data-link-type="dfn" href="#limit-class-maximum" id="ref-for-limit-class-maximum②⑦">maximum</a> 
      </td><td>64 
     </td></tr>
<tr>
  <td colspan="4">用于限制计算阶段 GPUShaderModule 入口点的 workgroup_size 的 Z 维度的最大值。</td>
</tr>
<tr class="row-continuation">
      <td colspan="4"> The maximum value of the <code>workgroup_size</code> Z dimensions for a
        compute stage <code class="idl"><a data-link-type="idl" href="#gpushadermodule" id="ref-for-gpushadermodule⑦">GPUShaderModule</a></code> entry-point. 
     </td></tr>
<tr>
  <td colspan="4"> </td>
</tr>
<tr>
      <td><dfn class="dfn-paneled idl-code" data-dfn-for="supported limits" data-dfn-type="attribute" data-export="" id="dom-supported-limits-maxcomputeworkgroupsperdimension"><code>maxComputeWorkgroupsPerDimension</code></dfn> 
      </td><td><code class="idl"><a data-link-type="idl" href="#typedefdef-gpusize32" id="ref-for-typedefdef-gpusize32②⑦">GPUSize32</a></code> 
      </td><td><a data-link-type="dfn" href="#limit-class-maximum" id="ref-for-limit-class-maximum②⑧">maximum</a> 
      </td><td>65535 
     </td></tr>
<tr>
  <td colspan="4">用于限制 dispatchWorkgroups(workgroupCountX, workgroupCountY, workgroupCountZ) 参数的最大值。 </td>
</tr>
<tr class="row-continuation">
      <td colspan="4"> The maximum value for the arguments of <code class="idl"><a data-link-type="idl" href="#dom-gpucomputepassencoder-dispatchworkgroups" id="ref-for-dom-gpucomputepassencoder-dispatchworkgroups②">dispatchWorkgroups(workgroupCountX, workgroupCountY, workgroupCountZ)</a></code>. 
   </td></tr>
<tr>
  <td colspan="4"> </td>
</tr>
</tbody></table>

#### 3.6.2.1.GPUSupportedLimits
> 3.6.2.1. GPUSupportedLimits

GPUSupportedLimits 暴露了适配器或设备所支持的限制。参见 [GPUAdapter.limit](https://www.w3.org/TR/2023/WD-webgpu-20230202/#dom-gpuadapter-limits) 和 [GPUDevice.limit](https://www.w3.org/TR/2023/WD-webgpu-20230202/#dom-gpudevice-limits) 。

> GPUSupportedLimits exposes the limits supported by an adapter or device. See GPUAdapter.limits and GPUDevice.limits.

```
// 原文代码不翻译
[Exposed=(Window, DedicatedWorker), SecureContext]
interface GPUSupportedLimits {
    readonly attribute unsigned long maxTextureDimension1D;
    readonly attribute unsigned long maxTextureDimension2D;
    readonly attribute unsigned long maxTextureDimension3D;
    readonly attribute unsigned long maxTextureArrayLayers;
    readonly attribute unsigned long maxBindGroups;
    readonly attribute unsigned long maxBindingsPerBindGroup;
    readonly attribute unsigned long maxDynamicUniformBuffersPerPipelineLayout;
    readonly attribute unsigned long maxDynamicStorageBuffersPerPipelineLayout;
    readonly attribute unsigned long maxSampledTexturesPerShaderStage;
    readonly attribute unsigned long maxSamplersPerShaderStage;
    readonly attribute unsigned long maxStorageBuffersPerShaderStage;
    readonly attribute unsigned long maxStorageTexturesPerShaderStage;
    readonly attribute unsigned long maxUniformBuffersPerShaderStage;
    readonly attribute unsigned long long maxUniformBufferBindingSize;
    readonly attribute unsigned long long maxStorageBufferBindingSize;
    readonly attribute unsigned long minUniformBufferOffsetAlignment;
    readonly attribute unsigned long minStorageBufferOffsetAlignment;
    readonly attribute unsigned long maxVertexBuffers;
    readonly attribute unsigned long long maxBufferSize;
    readonly attribute unsigned long maxVertexAttributes;
    readonly attribute unsigned long maxVertexBufferArrayStride;
    readonly attribute unsigned long maxInterStageShaderComponents;
    readonly attribute unsigned long maxInterStageShaderVariables;
    readonly attribute unsigned long maxColorAttachments;
    readonly attribute unsigned long maxColorAttachmentBytesPerSample;
    readonly attribute unsigned long maxComputeWorkgroupStorageSize;
    readonly attribute unsigned long maxComputeInvocationsPerWorkgroup;
    readonly attribute unsigned long maxComputeWorkgroupSizeX;
    readonly attribute unsigned long maxComputeWorkgroupSizeY;
    readonly attribute unsigned long maxComputeWorkgroupSizeZ;
    readonly attribute unsigned long maxComputeWorkgroupsPerDimension;
};
```

#### 3.6.2.2. GPUSupportedFeatures
> 3.6.2.2. GPUSupportedFeatures

GPUSupportedFeatures 是一个类似集合的接口。它的集合条目 是适配器或设备所支持的功能特征的 GPUFeatureName 值。它必须只包含来自 GPUFeatureName 枚举 里的字符串。

> GPUSupportedFeatures is a setlike interface. Its set entries are the GPUFeatureName values of the features supported by an adapter or device. It must only contain strings from the GPUFeatureName enum.

```
// 原文代码不翻译
[Exposed=(Window, DedicatedWorker), SecureContext]
interface GPUSupportedFeatures {
    readonly setlike<DOMString>;
};
```

注意：GPUSupportedFeatures 的集合条目的类型是DOMString，以允许用户代理优雅地验证GPUFeatureNames的有效性，这些GPUFeatureNames名称在规范的后期修订版本中被添加，但用户代理有可能尚未更新以识别它们。如果 集合条目的类型是GPUFeatureName，下面的代码将抛出一个TypeError，而不是仅仅报告false。

>> Note: The type of the GPUSupportedFeatures set entries is DOMString to allow user agents to gracefully handle valid GPUFeatureNames which are added in later revisions of the spec but which the user agent has not been updated to recognize yet. If the set entries type was GPUFeatureName the following code would throw an TypeError rather than reporting false:

```
EXAMPLE 5
// 用于判断 是否支持一个未识别的功能。
Check for support of an unrecognized feature:
if (adapter.features.has('unknown-feature')) {
    // Use unknown-feature
} else {
    console.warn('unknown-feature is not supported by this adapter.');
}
```

#### 3.6.2.3.GPUAdapterInfo
3.6.2.3. GPUAdapterInfo

GPUAdapterInfo 暴露了关于适配器的各种识别信息。

GPUAdapterInfo 中的所有成员都不能保证都被披露了。用户代理可以自行决定披露哪些值，而且在某些设备上很可能没有一个值会被了披露。因此，应用程序必须能够处理任何可能的 GPUAdapterInfo 值，包括没有这些值的情况。

> GPUAdapterInfo exposes various identifying information about an adapter.

> None of the members in GPUAdapterInfo are guaranteed to be populated. It is at the user agent’s discretion which values to reveal, and it is likely that on some devices none of the values will be populated. As such, applications must be able to handle any possible GPUAdapterInfo values, including the absence of those values.

关于隐私方面的考虑，见[§2.2.6适配器标识符](https://www.w3.org/TR/2023/WD-webgpu-20230202/#privacy-adapter-identifiers)。

> For privacy considerations, see § 2.2.6 Adapter Identifiers.
```
[Exposed=(Window, DedicatedWorker), SecureContext]
interface GPUAdapterInfo {
    readonly attribute DOMString vendor;
    readonly attribute DOMString architecture;
    readonly attribute DOMString device;
    readonly attribute DOMString description;
};
```

GPUAdapterInfo有以下属性。
* vendor，类型为DOMString，只读   
  如果有的话，表示适配器的供应商名称，否则为空字符串。
* architecture，类型为DOMString，只读
  如果有的话，表示适配器的GPU所属的家族或类别的名称，否则为空字符串。
* device，类型为DOMString，只读
  如果有的话，适配器的特定供应商标识符，否则为空字符串。
  **注意：** 这是一个代表适配器类型的值。例如，它可能是一个PCI的设备ID。它不像序列号那样唯一地标识一个特定的硬件。
* description，类型为DOMString，只读
  如果有的话，是一个易读的字符串，用于描述由驱动程序报告的适配器，否则为空字符串。
  **注意：** 由于没有对描述进行格式化处理的要求，因此不建议尝试解析这个值以获得有用信息。应用程序基于 GPUAdapterInfo 来改变其行为，例如在采取变通办法来处理已知的驱动程序问题时，应尽可能地依赖适配器的其他字段。

> GPUAdapterInfo has the following attributes:    
> * **vendor, of type DOMString, readonly**    
>     The name of the vendor of the adapter, if available. Empty string otherwise.    
> * **architecture, of type DOMString, readonly**    
>    The name of the family or class of GPUs the adapter belongs to, if available. Empty string otherwise.    
> * **device, of type DOMString, readonly**    
>    A vendor-specific identifier for the adapter, if available. Empty string otherwise.        
>    > NOTE: This is a value that represents the type of adapter. For example, it may be a PCI device ID. It does not uniquely identify a given piece of hardware like a serial number.    
> * **description, of type DOMString, readonly**    
>    A human readable string describing the adapter as reported by the driver, if available. Empty string otherwise.    
>    > NOTE: Because no formatting is applied to description attempting to parse this value is not recommended. Applications which change their behavior based on the GPUAdapterInfo, such as applying workarounds for known driver issues, should rely on the other fields when possible.

```
要为一个给定的适配卡硬件的适配器创建一个新的适配器信息，请运行以下步骤。
1. 让 adapterInfo 成为一个新的 GPUAdapterInfo。
2. 让 unmaskedValues 成为adapter.[[unmaskedIdentifiers]]。

To create a new adapter info for a given adapter adapter, run the following steps:
1. Let adapterInfo be a new GPUAdapterInfo.
2. Let unmaskedValues be adapter.[[unmaskedIdentifiers]]

3. 如果 unmaskedValues 包含 "vendor"，并且该厂商是已知的:
  1. 将 adapterInfo.vendor 设置为适配器的供应商名称，作为一个规范化的标识符字符串。
  否则。
  将 adapterInfo.vendor 设置为空字符串或合理近似的供应商名称，作为一个规范化标识符字符串。  

3. If unmaskedValues contains "vendor" and the vendor is known:
    1. Set adapterInfo.vendor to the name of adapter’s vendor as a normalized identifier string.
    Otherwise:
    1. Set adapterInfo.vendor to the empty string or a reasonable approximation of the vendor as a normalized identifier string.

4. 如果 unmaskedValues 包含 "architecture" 并且架构的信息是已经的。
  1. 将 adapterInfo.architecture 设置为代表适配器所属的家族或类别，作为一个规范化标识符字符串。
  否则。
  1. 将 adapterInfo.architecture 设置为空字符串或者近似合理的架构名称，作为一个规范化标识符字符串。

4. If unmaskedValues contains "architecture" and the architecture is known:
    1. Set adapterInfo.architecture to a normalized identifier string representing the family or class of adapters to which adapter belongs.
    Otherwise:
    1. Set adapterInfo.architecture to the empty string or a reasonable approximation of the architecture as a normalized identifier string.

5. 如果 unmaskedValues 包含 "device" ，并且设备信息是已知的。
  1. 将 adapterInfo.device 设置为代表适配器的特定厂商的标识符，作为一个规范化标识符字符串。
  否则。
  1. 将 adapterInfo.device 设置为空字符串或者一个合理的近似于厂商特定的标识符，作为一个规范化标识符字符串。

5. If unmaskedValues contains "device" and the device is known:
    1. Set adapterInfo.device to a normalized identifier string representing a vendor-specific identifier for adapter.
    Otherwise:
    1.Set adapterInfo.device to to the empty string or a reasonable approximation of a vendor-specific identifier as a normalized identifier string.

6. 如果 unmaskedValues 包含 "description" 并且有一个描述信息是已知的。
  1. 将 adapterInfo.description 设置为适配器的描述，作为驱动程序报告给用户所用。
  否则。
  1. 将adapterInfo.description设置为空字符串或者一个合理的描述。

6. If unmaskedValues contains "description" and a description is known:
    1. Set adapterInfo.description to a description of the adapter as reported by the driver.
    Otherwise:
    1. Set adapterInfo.description to the empty string or a reasonable approximation of a description.

7. 返回 adapterInfo。

7 Return adapterInfo.
```

```
一个规范化的标识符字符串遵循以下正则模式。
[a-z0-9]+(-[a-z0-9]+)*

A normalized identifier string is one that follows the following pattern:
[a-z0-9]+(-[a-z0-9]+)*

EXAMPLE 6
// 有效的规范化标识符字符串的例子:
Examples of valid normalized identifier strings include:
gpu
3d
0x3b2f
next-gen
series-x20-ultra
```

## 3.7.扩展文件

> 3.7. Extension Documents

"扩展文档"是描述非规范性的新功能的额外文档，其不属于WebGPU/WGSL规范的一部分。它们描述了建立在这些规范之上的功能，通常包括一个或多个新的API特征标示符 和/或者 WGSL启用使能的指令，或者是与其他网络规范草案的交互。

> "Extension Documents" are additional documents which describe new functionality which is non-normative and not part of the WebGPU/WGSL specifications. They describe functionality that builds upon these specifications, often including one or more new API feature flags and/or WGSL enable directives, or interactions with other draft web specifications.

WebGPU具体的实现不得暴露扩展功能；这样做违反规范。新功能只有在集成到WebGPU规范（此文本）和/或者 WGSL规范中后才会成为WebGPU标准的一部分。

> WebGPU implementations must not expose extension functionality; doing so is a spec violation. New functionality does not become part of the WebGPU standard until it is integrated into the WebGPU specification (this document) and/or WGSL specification.

## 3.8.原始资源限制

> 3.8. Origin Restrictions

WebGPU允许访问存储在图像、视频和画布中的图像数据。对跨域媒体访问进行了限制，因为着色器可以间接访问纹理的内容，因为这些资源已经上传到GPU了。

如果资源的源头不是清洁的，WebGPU不允许上传图像资源。

这也意味着，WebGPU用于渲染的画布的origin-clean标示符，永远不会被设置为false。

> WebGPU allows accessing image data stored in images, videos, and canvases. Restrictions are imposed on the use of cross-domain media, because shaders can be used to indirectly deduce the contents of textures which have been uploaded to the GPU.

> WebGPU disallows uploading an image source if it is not origin-clean.

> This also implies that the origin-clean flag for a canvas rendered using WebGPU will never be set to false.

关于针对图像和视频等元素发出CORS请求的更多信息，请参考：
* [HTML §2.5.4 CORS设置属性](https://html.spec.whatwg.org/multipage/urls-and-fetching.html#cors-settings-attributes)
* [HTML §4.8.3 img元素 \<img\>](https://html.spec.whatwg.org/multipage/embedded-content.html#the-img-element)
* [HTML §4.8.11 媒体元素 HTMLMediaElement](https://html.spec.whatwg.org/multipage/media.html#media-elements)

> For more information on issuing CORS requests for image and video elements, consult:
> * [HTML § 2.5.4 CORS settings attributes]()
> * [HTML § 4.8.3 The img element img]()
> * [HTML § 4.8.11 Media elements HTMLMediaElement]()

## 3.9.任务来源

> 3.9. Task Sources

# 3.9.1.WebGPU任务源

> 3.9.1. WebGPU Task Source

WebGPU定义了一个新的任务源，称为WebGPU任务源。它被用于 uncapturederror 事件和 GPUDevice.lost。

> WebGPU defines a new task source called the WebGPU task source. It is used for the uncapturederror event and GPUDevice.lost.

要为 GPUDevice 设备的一个全局任务排队，有一系列的步骤：
  1. 将一个全局任务 在WebGPU任务源上进行排队，使用创建设备的全局对象的步骤，以及他的步骤的步骤。

> To queue a global task for GPUDevice device, with a series of steps steps:
>  1. Queue a global task on the WebGPU task source, with the global object that was used to create device, and the steps steps.

### 3.9.2.自动过期任务源

> 3.9.2. Automatic Expiry Task Source

WebGPU定义了一个新的任务源，叫做自动过期任务源。它用于某些对象的自动或定时的失效（销毁）：
* 由 getCurrentTexture() 返回的 GPUTexture。
* 从 HTMLVideoElements 创建的 GPUExternalTextures。

> WebGPU defines a new task source called the automatic expiry task source. It is used for the automatic, timed expiry (destruction) of certain objects:
> * GPUTextures returned by getCurrentTexture()
> * GPUExternalTextures created from HTMLVideoElements

要在GPUDevice设备上为自动过期任务排队，需要按一系列的步骤进行。
1. 将一个全局任务在自动过期任务源上排队，使用创建设备的全局对象的步骤，以及他的步骤的步骤。

> To queue an automatic expiry task with GPUDevice device and a series of steps steps:
> 1. Queue a global task on the automatic expiry task source, with the global object that was used to create device, and the steps steps.

来自自动过期任务源的任务应该被优先处理；特别是，一旦自动过期任务进入队列，它们应该在用户定义的（JavaScript）任务之前运行。这导致了更为严格的行为，可以帮助开发者编写更多的可移植应用程序，但仍然强烈鼓励开发者在多个具体的实现中进行测试。

> Tasks from the automatic expiry task source should be processed with high priority; in particular, once queued, they should run before user-defined (JavaScript) tasks. This results in stricter behavior that helps developers write more portable applications, but developers are still strongly encouraged to test in multiple implementations.

**注意：** 由于任务源的优先级没有规范性的定义，所以这个任务队列的任务可以在一定范围内执行。在实践中，这意味着这些任务可以在HTML处理模型里面用固定的步骤实现。以下所有选项都是有效的。
* 在任何运行中任务后之后立即执行。
* 在运行动画帧回调之后立即执行。
* 在渲染之前立即进行。注意这个选项会导致更难预测的过期任务，特别是对于不可见的画布（例如滚出屏幕或在后台标签中），有可能会允许无效的程序出现正确的，或者是改变（不正确地）依赖过期时间的网站的行为。    
在画布纹理过期的情况下，过期也会在更新WebGPU画布的渲染过程中完成，因此，对于一个实现来说，这相当于完全省略了这项任务。


> Note: Since task source priority is not normatively defined, there is a range of points at which tasks from this task queue can execute. In practice, this means these tasks can be implemented with fixed steps inside the HTML processing model. All of the following options are valid:
> * Immediately after running any task.
> * Immediately after running animation frame callbacks.
> * Immediately before rendering. Note this choice results in less predictable expiry, especially for non-visible canvases (e.g. scrolled off the screen or in a background tab), potentially allowing invalid programs to appear correct or changing the behavior of websites which (incorrectly) rely on expiry timing.    
> In the case of canvas texture expiry, expiry is also done in update the rendering of the WebGPU canvas, so it’s equivalent for an implementation to elide the task entirely.

## 3.10.色彩空间和编码

3.10. Color Spaces and Encoding

WebGPU不提供色彩管理。WebGPU中的所有值（如纹理元素）都是原始数值，而不是色彩管理中的颜色值。

> WebGPU does not provide color management. All values within WebGPU (such as texture elements) are raw numeric values, not color-managed color values.

WebGPU有相应的接口和 颜色管理的输出（通过GPUCanvasConfiguration）和输入（通过copyExternalImageToTexture()和importExternalTexture()）进行对接。因此，必须在WebGPU的数值和外部颜色值之间进行颜色转换。每个这样的接口都要在本地定义了一个编码（如色彩空间、传递函数和alpha预乘法），以便WebGPU的颜色数值将在其中被转换处理。

> WebGPU does interface with color-managed outputs (via GPUCanvasConfiguration) and inputs (via copyExternalImageToTexture() and importExternalTexture()). Thus, color conversion must be performed between the WebGPU numeric values and the external color values. Each such interface point locally defines an encoding (color space, transfer function, and alpha premultiplication) in which the WebGPU numeric values are to be interpreted.


 > WebGPU allows all of the color spaces in the PredefinedColorSpace enum. Note, each color space is defined over an extended range, as defined by the referenced CSS definitions, to represent color values outside of its space (in both chrominance and luminance).

WebGPU允许在 PredefinedColorSpace 枚举中定义的所有颜色空间。注意，每个颜色空间都是在一个扩展范围内定义的，正如参考CSS定义那样，来定义颜色空间，以表示其空间之外的颜色值（在色度和亮度方面）。

> 考虑将srgb编码的图像上传到线性编码的纹理的方法。[问题#gpuweb/gpuweb#1715](https://github.com/gpuweb/gpuweb/issues/1715) 。

 >> ISSUE 4: Consider a path for uploading srgb-encoded images into linearly-encoded textures. [Issue #gpuweb/gpuweb#1715]

一个超出色域预乘的RGBA值是指 有任何一个R/G/B通道的数值超过了alpha通道的数值。例如，预乘的sRGB颜色空间的RGBA值[1.0, 0, 0, 0.5]表示（未预乘的）颜色值为[2, 0, 0]，而α值为50%，在CSS中可以写成rgb(srgb 2 0 0 / 50%)。就像sRGB色域之外的任何颜色值一样，这是扩展颜色空间中一个定义明确的要点（除了当alpha为0时，在这种情况下是没有颜色显示的）。然而，当这样的值被输出到可见的画布上时，其结果是未定义的（参见 [GPUCanvasAlphaMode]() ["premultiplied"]()）。

 > An out-of-gamut premultiplied RGBA value is one where any of the R/G/B channel values exceeds the alpha channel value. For example, the premultiplied sRGB RGBA value [1.0, 0, 0, 0.5] represents the (unpremultiplied) color [2, 0, 0] with 50% alpha, written rgb(srgb 2 0 0 / 50%) in CSS. Just like any color value outside the sRGB color gamut, this is a well defined point in the extended color space (except when alpha is 0, in which case there is no color). However, when such values are output to a visible canvas, the result is undefined (see GPUCanvasAlphaMode "premultiplied").

### 3.10.1.颜色空间的转换

> 3.10.1. Color Space Conversions

一种颜色在不同空间之间的转换是指根据上述定义将其在一个空间的表现形式转换为另一个空间的表现形式。

> A color is converted between spaces by translating its representation in one space to a representation in another according to the definitions above.

如果源值的RGBA通道少于4个，那么在颜色转换之前，缺失的 绿色/蓝色/alpha通道 会被分别设置为0、0、1，以便进行色彩空间/编码和alpha预乘法。转换后，如果目标值需要的通道少于4个，额外的通道将被忽略。

> If the source value has fewer than 4 RGBA channels, the missing green/blue/alpha channels are set to 0, 0, 1, respectively, before converting for color space/encoding and alpha premultiplication. After conversion, if the destination needs fewer than 4 channels, the additional channels are ignored.

**注意：**灰度图像在其色彩空间中一般表示RGB值(V，V，V)，或RGBA值(V，V，V，A)。

> NOTE: Grayscale images generally represent RGB values (V, V, V), or RGBA values (V, V, V, A) in their color space.

在颜色转换过程中，颜色不会被无损钳制：如果源颜色值超出了目标颜色空间的色域范围，那么从一个颜色空间转换到另一个颜色空间就会导致数值超出[0, 1]的范围。例如，在sRGB颜色空间中，如果源颜色是rgba16float，在像Display-P3这样更宽的色彩空间中，或者是预乘的，并包含超出色域的数值，就会出现这种情况。

> Colors are not lossily clamped during conversion: converting from one color space to another will result in values outside the range [0, 1] if the source color values were outside the range of the destination color space’s gamut. For an sRGB destination, for example, this can occur if the source is rgba16float, in a wider color space like Display-P3, or is premultiplied and contains out-of-gamut values.

同样，如果源颜色值具有较高的比特深度（例如，PNG的每个分量为16比特）或扩展范围（例如，canvas是采用float16进行存储的），这些颜色通过色彩空间转换后也会得到精度上的保留，即中间的计算过程至少具有源颜色值的精度要求。

> Similarly, if the source value has a high bit depth (e.g. PNG with 16 bits per component) or extended range (e.g. canvas with float16 storage), these colors are preserved through color space conversion, with intermediate computations having at least the precision of the source.

### 3.10.2.色彩空间转换的消除

> 3.10.2. Color Space Conversion Elision

如果源值和目标值的色彩空间/编码转换是相同的，那么转换就没有必要。一般来说，如果转换的任何给定步骤都是一个身份标识函数（没有具体操作），为了性能，实现应该删除它。

> If the source and destination of a color space/encoding conversion are the same, then conversion is not necessary. In general, if any given step of the conversion is an identity function (no-op), implementations should elide it, for performance.

为了获得最佳性能，应用程序应该设置他们的色彩空间和编码选项，以便在整个过程中使必要的转换数量最小化。对于GPUImageCopyExternalImage 的各种图像源:
* ImageBitmap。
  * 预乘法(Premultiplication)是通过 premultiplyAlpha 来控制的。
  * 色彩空间通过 colorSpaceConversion 来控制。
* 2d画布。
  * 始终要经过预乘的。
  * 色彩空间通过 颜色空间上下文的创建属性 来控制。
* WebGL画布。
  * 预乘法(Premultiplication)是通过 WebGLContextAttributes 中的 premultipliedAlpha 选项控制的。
  * 色彩空间是通过 WebGLRenderingContext 的 drawingBufferColorSpace 状态来控制的。

> For optimal performance, applications should set their color space and encoding options so that the number of necessary conversions is minimized throughout the process. For various image sources of GPUImageCopyExternalImage:
> * ImageBitmap:
>   * Premultiplication is controlled via premultiplyAlpha.
>   * Color space is controlled via colorSpaceConversion.
> * 2d canvas:
>   * Always premultiplied.
>   * Color space is controlled via the colorSpace context creation attribute.
> * WebGL canvas:
>   * Premultiplication is controlled via the premultipliedAlpha option in WebGLContextAttributes.
>   * Color space is controlled via the WebGLRenderingContext's drawingBufferColorSpace state.

**注意：** 在使用这些功能之前，请检查浏览器的实现是否对这些功能进行了支持。

> **NOTE:** Check browser implementation support for these features before relying on them.

## 3.11. 从JavaScript到WGSL的数字转换

> 3.11. Numeric conversions from JavaScript to WGSL

WebGPU API的几个部分（如可重载管道的常量和渲染时传递的清除值）， 从 WebIDL 中获取数值（double或float）并将其转换为 WGSL 数值（bool, i32, u32, f32, f16）。

> Several parts of the WebGPU API (pipeline-overridable constants and render pass clear values) take numeric values from WebIDL (double or float) and convert them to WGSL values (bool, i32, u32, f32, f16).

> 将一个IDL值，如果idlValue的类型为 double 或 float 转换成 WGSL 类型 T，可能会抛出一个 TypeError。        
> **注意：** 这个 TypeError 是在设备的时间轴上产生的，绝对不会冒泡传递给 JavaScript。    
> 1. 断言 idlValue 是一个有限的值，因为它不是无限制的 double 或无限制的 float。    
> 2. 让 v 是一个 ECMAScript数， 此值是由将 idlValue 转换为 ECMAScript 值而产生的。    

>> To convert an IDL value idlValue of type double or float to WGSL type T, possibly throwing a TypeError:    
>> **NOTE:** This TypeError is generated in the device timeline and never surfaced to JavaScript.    
>> 1. Assert idlValue is a finite value, since it is not unrestricted double or unrestricted float.
>> 2. Let v be the ECMAScript Number resulting from ! converting idlValue to an ECMAScript value.

> 3. (***接上***)
> **如果 T 是 bool**    
> 返回一个WGSL bool值，此值对应于 将 v 转换为 布尔类型的 IDL值 的结果。   
> **注意：** 这个算法是在 ECMAScript 值转换为IDL double或float值后再调用的。如果原始的 ECMAScript值 是一个非数字、非布尔值，如[]或{}，那么 WGSL布尔值的结果可能会和 直接将 ECMAScript值 转换为IDL布尔值的情况不同。

>> 3.   **If T is bool**    
>>      Return the WGSL bool value corresponding to the result of ! converting v to an IDL value of type boolean.    
>>     **NOTE:**  This algorithm is called after the conversion from an ECMAScript value to an IDL double or float value. If the original ECMAScript value was a non-numeric, non-boolean value like [] or {}, then the WGSL bool result may be different than if the ECMAScript value had been converted to IDL boolean directly.

> **如果 T 是 i32**      
> 返回的WGSL i32值，此值对应于将 v 转换为 [EnforceRange]long 类型的 IDL值 的结果。

>>    **If T is i32**    
>>    Return the WGSL i32 value corresponding to the result of ? converting v to an IDL value of type [EnforceRange] long.


> **如果 T 是 u32**    
> 返回WGSL u32值，此值对应于将 v 转换为[EnforceRange]无符号整数 类型的 IDL值 的结果。

>>   **If T is u32**    
>>   Return the WGSL u32 value corresponding to the result of ? converting v to an IDL value of type [EnforceRange] unsigned long.

> **如果 T 是 f32 **    
> 返回 WGSL f32 值，该值对应于 将 v 转换为 float 类型的 IDL 值的结果相对应。

>>   **If T is f32**    
>>   Return the WGSL f32 value corresponding to the result of ? converting v to an IDL value of type float.

> **如果 T 是 f16**      
> 1. 让 wgslF32 成为WGSL f32值，此值对应于将 v 转换为 float类型的 IDL值 的结果。    
> 2. 返回 f16(wgslF32)，即把 WGSL f32 值转换为 WGSL 浮点转换中定义的 f16 的结果。    

>>   **If T is f16**
>>   1. Let wgslF32 be the WGSL f32 value corresponding to the result of ? converting v to an IDL value of type float.
>>   2. Return f16(wgslF32), the result of ! converting the WGSL f32 value to f16 as defined in WGSL floating point conversion.

> **注意：** 只要该值在f32的范围内，就不会产生错误，即使该值在超出了 f16 的范围。

>>   **NOTE:**  As long as the value is in-range of f32, no error is thrown, even if the value is out-of-range of f16.

> ***个人加上的，另外一点***，
> 将 GPUColor 颜色转换为 texture 格式的 texel值 的格式时，可能会抛出一个 TypeError 错误。    
> **注意：** 这个 TypeError 也是在设备的时间轴上产生的，从来没有传递冒泡给 JavaScript。    

>> To convert a GPUColor color to a texel value of texture format format, possibly throwing a TypeError:    
>> **NOTE:** This TypeError is generated in the device timeline and never surfaced to JavaScript.   

> 1. 如果组件的 format（断言它们都有相同的类型）是:       
> **浮点类型 或 规范化类型**    
> 让 T 是 f32。        
> 有符号整数类型    
> 让 T 是 i32。       
> 无符号整数类型    
> 让 T 为 u32。    

>> 1. If the components of format (assert they all have the same type) are:
>>    floating-point types or normalized types        
>>    Let T be f32.
>>    <br />
>>    signed integer types    
>>    Let T be i32.    
>>    <br />
>>    unsigned integer types     
>>    Let T be u32.    

> 2. 让 wgslColor 是一个 vec4<T> 类型的 WGSL值，其中4个分量是颜色的RGBA通道，每个分量都转换为 WGSL 类型 T。    
> 3. 将 wgslColor 转换为相应格式的结果，并返回，使用的是和本规范[§ 23.3.7 输出合并步骤]()相同的转换规则。     
>    **注意：** 对于非整数类型，数值的明确选择是由具体实现定义的。对于规范化的类型，数值会被钳制在类型的范围内。

>> 2.  Let wgslColor be a WGSL value of type vec4<T>, where the 4 components are the RGBA channels of color, each ? converted to WGSL type T.    
>> 3.  Convert wgslColor to format using the same conversion rules as the § 23.3.7 Output Merging step, and return the result.    
>>    **NOTE:** For non-integer types, the exact choice of value is implementation-defined. For normalized types, the value is clamped to the range of the type.

注意：换句话说，写入的值将如同由 WGSL着色器 写入的值一样，该着色器输出的值可表示为类型是 f32、i32 或 u32 的vec4。

>> **NOTE:** In other words, the value written will be as if it was written by a WGSL shader that outputs the value represented as a vec4 of f32, i32, or u32.
