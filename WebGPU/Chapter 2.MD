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

# 2\. 恶意使用的考虑
> 2\. Malicious use considerations

这一节是非规范性的。这一节描述了在网络上(web)公开发布此API的相关风险。

> This section is non-normative. It describes the risks associated with exposing this API on the Web.

## 2.1. 安全考虑
> 2.1. Security Considerations

WebGPU的安全要求和以网的web安全要求一样，同样是不容商量的。通常的做法是在命令到达GPU之前严格的验证所有命令，以确保一个页面只能使用自己的数据。

> The security requirements for WebGPU are the same as ever for the web, and are likewise non-negotiable. The general approach is strictly validating all the commands before they reach GPU, ensuring that a page can only work with its own data.

### 2.1.1. 基于CPU的未定义行为

> 2.1.1. CPU-based undefined behavior

WebGPU实现了将用户提交的[作负载任务 转换成 相对应的目标平台特有的API指令集。特定平台原生的API则明确规定了指令的有效用法(例如，[vkCreateDescriptorSetLayout](https://www.khronos.org/registry/vulkan/specs/1.2-extensions/man/html/vkCreateDescriptorSetLayout.html)指令)，但是如果在使用过程中，不能遵循有效的使用规则，一般原生的API是不保证会得任何有效结果的。这一行为被称为“未定义的行为”，而这一不安全的行为会被攻击者(黑客)利用来访问他们本没有权限访问的内存，或者从而强制驱动程序执行任意恶意代码。

> A WebGPU implementation translates the workloads issued by the user into API commands specific to the target platform. Native APIs specify the valid usage for the commands (for example, see vkCreateDescriptorSetLayout) and generally don’t guarantee any outcome if the valid usage rules are not followed. This is called "undefined behavior", and it can be exploited by an attacker to access memory they don’t own, or force the driver to execute arbitrary code.

为了禁止不安全的使用，用户提交的任何工作任务输入，都被WebGPU限制在一个允许的范围内。一个WebGPU的实现需要验证用户提交的所有输入，只有合法的工作任务才能提交到驱动程序去执行。本文档说明了所有的错误产生条件和异常处理方法。例如，在调用[copyBufferToBuffer()](https://www.w3.org/TR/webgpu/#dom-gpucommandencoder-copybuffertobuffer)函数时，如果“源参数”和“目标参数”中指定的内存缓冲区，如果有相同的交叉部分就会导致 [GPUCommandEncoder](https://www.w3.org/TR/webgpu/#gpucommandencoder)产生错误，并且不再执行后续的其他操作了。

> In order to disallow insecure usage, the range of allowed WebGPU behaviors is defined for any input. An implementation has to validate all the input from the user and only reach the driver with the valid workloads. This document specifies all the error conditions and handling semantics. For example, specifying the same buffer with intersecting ranges in both "source" and "destination" of copyBufferToBuffer() results in GPUCommandEncoder generating an error, and no other operation occurring.

请查看[第22章 错误和调度](https://www.w3.org/TR/webgpu/#errors-and-debugging)，可以获取更多关于错误处理的信息。
> See § 22 Errors & Debugging for more information about error handling.

### 2.1.2. 基于GPU的未定义行为
> 2.1.2. GPU-based undefined behavior

WebGPU的着色器程序是由GPU硬件的内部计算单元执行的。在原生的API接口中，一些着色器指令可能会导致在GPU上的未定义行为。为了解决这个问题，WebGPU严格定义了着色器的指令集和其定义好的行为。当一个着色器程序被提交到[createShaderModule()](https://www.w3.org/TR/webgpu/#dom-gpudevice-createshadermodule)执行时，WebGPU的实现必须首先进行严格验证，然后再将其编译转换(即编译成特定平台的着色器程序)和传递。

> WebGPU shaders are executed by the compute units inside GPU hardware. In native APIs, some of the shader instructions may result in undefined behavior on the GPU. In order to address that, the shader instruction set and its defined behaviors are strictly defined by WebGPU. When a shader is provided to createShaderModule(), the WebGPU implementation has to validate it before doing any translation (to platform-specific shaders) or transformation passes.

### 2.1.3. 未初始化数据
> 2.1.3. Uninitialized data

通常，分配新内存可能会暴露出系统中运行的其他应用程序所遗留下来的的数据值，为了解决这个问题，WebGPU的实现需要默认初始化所有资源为零值。尽管如此，但在一些实践中，如果程序开发者已经对内容进行了手动初始化，WebGPU的实现可能(可以)跳过这一步骤，这些需要被初始化的资源包括变量和着色器内部的共享工作组内存。

> Generally, allocating new memory may expose the leftover data of other applications running on the system. In order to address that, WebGPU conceptually initializes all the resources to zero, although in practice an implementation may skip this step if it sees the developer initializing the contents manually. This includes variables and shared workgroup memory inside shaders.

由于不同平台的清除工作组内存的内部精确机制可能有所不同。如果原生API没有提供相应清除机制，那么，WebGPU的实现，在此平台进行编译转换计算着色器程序前，首先要在所有调用中进行清除操作，然后同步确认所有清楚工作全部完成，最后再继续执行开发者的代码。
> The precise mechanism of clearing the workgroup memory can differ between platforms. If the native API does not provide facilities to clear it, the WebGPU implementation transforms the compute shader to first do a clear across all invocations, synchronize them, and continue executing developer’s code.

> **注意**： 如果一些内存资源是被用于队列操作，那么他的初始化状态只有在入队操作时才能被确定(而不是在其被编码成指令缓冲区时确定)，因此，一些在入队时未优化的延迟清除机制是必要的（例如，清除一个纹理操作，不是将[GPULoadOP](https://www.w3.org/TR/webgpu/#enumdef-gpuloadop)函数的[“load”](https://www.w3.org/TR/webgpu/#dom-gpuloadop-load)命名为[“clear”](https://www.w3.org/TR/webgpu/#dom-gpuloadop-clear)，而是此函数会默认运行时进行清除操作)   
因此，所有的实现中都应该对这种潜在的性能缺陷在开发者控制台发出警告，即使在该实现中不存在缺陷。
>> Note: The initialization status of a resource used in a queue operation can only be known when the operation is enqueued (not when it is encoded into a command buffer, for example). Therefore, some implementations will require an unoptimized late-clear at enqueue time (e.g. clearing a texture, rather than changing GPULoadOp "load" to "clear").
As a result, all implementations should issue a developer console warning about this potential performance penalty, even if there is no penalty in that implementation.

### 2.1.4. 首色器的越界访问
> 2.1.4. Out-of-bounds access in shaders

[着色器](https://www.w3.org/TR/webgpu/#shaders)程序可以直接访问相应的[物理资源](https://www.w3.org/TR/webgpu/#physical-resources)(例如，通过一个由[GPUBufferBinding](https://www.w3.org/TR/webgpu/#dictdef-gpubufferbinding)函数绑定的[“uniform" 变量](https://www.w3.org/TR/webgpu/#dom-gpubufferbindingtype-uniform))，或者通过一个纹理单元间接访问物理资源，这里的纹理单元是指一个处理纹理坐标转换的固定功能硬件模块。WebGPU的API的实现中验证机制，只能保证提供了所有必要的输入给着色器程序，并且保证这些输入具有正确的用法和输入类型，但如果没有调用包括[纹理单元](https://www.w3.org/TR/webgpu/#texture-unit)时，则不能保证所有数据在其边界内进行访问。

> Shaders can access physical resources either directly (for example, as a "uniform" GPUBufferBinding), or via texture units, which are fixed-function hardware blocks that handle texture coordinate conversions. Validation in the WebGPU API can only guarantee that all the inputs to the shader are provided and they have the correct usage and types. The WebGPU API can not guarantee that the data is accessed within bounds if the texture units are not involved.

为了防止着色器程序访问了应用程序没有权限的GPU内存，WebGPU的实现可能会在驱动程序中开启一个特殊的模式(称为”鲁棒性缓冲区访问“)，以便这种特殊的访问模式可以保证着色器访问内存时在缓冲区的边界内。

> In order to prevent the shaders from accessing GPU memory an application doesn’t own, the WebGPU implementation may enable a special mode (called "robust buffer access") in the driver that guarantees that the access is limited to buffer bounds.

另外，WebGPU在实现时，在编译转换用户代码时，可能通过插入一段手动边界检测代码到首色器程序中。如果采用这种方法实现时，那处越界访问检查机制仅仅适用于数组索引访问的类型。着色器程序中的结构体的普通字段访问则不需检查，因为在主机端存在[minBindingSize](https://www.w3.org/TR/webgpu/#dom-gpubufferbindinglayout-minbindingsize)验证机制，以确保其安全访问。
> Alternatively, an implementation may transform the shader code by inserting manual bounds checks. When this path is taken, the out-of-bound checks only apply to array indexing. They aren’t needed for plain field access of shader structures due to the minBindingSize validation on the host side.

如果着色器程序试图在[物理资源](https://www.w3.org/TR/webgpu/#physical-resources)边界之外加载数据，WebGPU的实现可以按以下方式处理  
  1. 返回一个值，此值在物理资源范围内，但在不同的位置。  
  1. 返回一个"(0, 0, 0, X)"的特殊向量值，其中的X为任意值。  
  1. 部分的丢弃绘制或者任务调度。

> If the shader attempts to load data outside of physical resource bounds, the implementation is allowed to:

> 1. return a value at a different location within the resource bounds
> 1. return a value vector of "(0, 0, 0, X)" with any "X"
> 1. partially discard the draw or dispatch call

如果着色器程序试图在[物理资源](https://www.w3.org/TR/webgpu/#physical-resources)边界之外写入数据，WebGPU的实现可以按以下方式处理  
  1. 将此值写入到物理资源范围内，但在不同的位置。  
  1. 放弃写入操作。
  1. 部分的丢弃绘制或者任务调度。

> If the shader attempts to write data outside of physical resource bounds, the implementation is allowed to:

> 1. write the value to a different location within the resource bounds
> 2. discard the write operation
> 3. partially discard the draw or dispatch call

### 2.1.5. 无效数据
> 2.1.5. Invalid data

当从CPU上传[浮点数](https://en.wikipedia.org/wiki/IEEE_754)数据到GPU时，或者在GPU上生成浮点数时，我们最终可能得到一个二进制数据系统，但且不能转换成一个有效的数值，而只能转换成infinity或者NaN(不是一个数)，这种情况是由于GPU的行为受限于GPU硬件单元的精度，而GPU的硬件单元精度也是基于IEEE-754标准实现的。WebGPU保证由于缺陷引入的无效浮点数据只会影响到算术计算的结果，而不会再有其他的副作用。

> When uploading floating-point data from CPU to GPU, or generating it on the GPU, we may end up with a binary representation that doesn’t correspond to a valid number, such as infinity or NaN (not-a-number). The GPU behavior in this case is subject to the accuracy of the GPU hardware implementation of the IEEE-754 standard. WebGPU guarantees that introducing invalid floating-point numbers would only affect the results of arithmetic computations and will not have other side effects.

### 2.1.6 驱动程序bugs
> 2.1.6. Driver bugs

GPU驱动程序像其他软件一样容易产生bugs.如果一个bug产生，黑客可能会利用驱动程序的错误行为来访问没有权限的数据。为了降低风险，WebGPU工作组会和GPU供应商协作，共同将WebGPU一致性测试组件(CTS)作为GPU供应商的驱动程序的测试流程的一部分而集成在一起，就像以前针对WebGL的工作一样。WebGPU的实现希望可以为一些已发现的bugs提供规避的方法，同时针对一些无法规避的已知
bugs则在驱动程序上禁用WebGPU.

> GPU drivers are subject to bugs like any other software. If a bug occurs, an attacker could possibly exploit the incorrect behavior of the driver to get access to unprivileged data. In order to reduce the risk, the WebGPU working group will coordinate with GPU vendors to integrate the WebGPU Conformance Test Suite (CTS) as part of their driver testing process, like it was done for WebGL. WebGPU implementations are expected to have workarounds for some of the discovered bugs, and disable WebGPU on drivers with known bugs that can’t be worked around.

### 2.1.7 时序攻击
> 2.1.7. Timing attacks

WebGPU通过Web Workers来实现多线程。因此，它的设计避免了用户遭遇现代高精度的时序攻击。一些对象，例如[GPUBuffer](https://www.w3.org/TR/webgpu/#gpubuffer)
和 [GPUQueue](https://www.w3.org/TR/webgpu/#gpuqueue), 具有一些可同时被访问的共享状态，他们可以产生竞态条件，类似于从多个Web Workers中访问同一个类型化共享数组缓存(SharedArrayBuffer)，此时，会导致线程的调度是可见的。

> WebGPU is designed to later support multi-threaded use via Web Workers. As such, it is designed not to open the users to modern high-precision timing attacks. Some of the objects, like GPUBuffer or GPUQueue, have shared state which can be simultaneously accessed. This allows race conditions to occur, similar to those of accessing a SharedArrayBuffer from multiple Web Workers, which makes the thread scheduling observable.

WebGPU为了解决这个问题，只有当[跨域隔离规则](https://web.dev/coop-coep/)启用后，同时此[工作代理](https://tc39.es/ecma262/#agent)需在[代理集群](https://tc39.es/ecma262/#sec-agent-clusters)中，对象才具备实现反系列化(或者共享)的能力。这项限制也符合针对恶意使用ShareArrayBuffer而采取的[缓解措施](https://hacks.mozilla.org/2020/07/safely-reviving-shared-memory/)。同样，用户代理也可能系列化代理共享的任何句柄以完全防止任何并发性的产生。

> WebGPU addresses this by limiting the ability to deserialize (or share) objects only to the agents inside the agent cluster, and only if the cross-origin isolated policies are in place. This restriction matches the mitigations against the malicious SharedArrayBuffer use. Similarly, the user agent may also serialize the agents sharing any handles to prevent any concurrency entirely.

最后说明一点，针对WebGPU中共享状态的竞态攻击将是SharedArrayBuffer攻击的一个小子集。

> In the end, the attack surface for races on shared state in WebGPU will be a small subset of the SharedArrayBuffer attacks.

WebGPU同样还规定说明了[“时间戳查询”](https://www.w3.org/TR/webgpu/#dom-gpufeaturename-timestamp-query)功能，它为GPU操作提供了高精度的时钟计时功能。这项功能是可选的，WebGPU的实现可将此功能限制在那些值得信任的场景中。另外，时间戳查询结果可以由计算着色器处理，并且对齐到较低的精度要求。

> WebGPU also specifies the "timestamp-query" feature, which provides high precision timing of GPU operations. The feature is optional, and a WebGPU implementation may limit its exposure only to those scenarios that are trusted. Alternatively, the timing query results could be processed by a compute shader and aligned to a lower precision.


### 2.1.8 行锤攻击
> 2.1.8. Row hammer attacks

[行锤](https://en.wikipedia.org/wiki/Row_hammer)是一类利用DRAM单元中的状态泄漏的攻击方式。它可以用在GPU攻击上，WebGPU没有任何具体的缓解措施，而是依赖于平台级的解决方案，例如减少内存的刷新间隔。

> Row hammer is a class of attacks that exploit the leaking of states in DRAM cells. It could be used on GPU. WebGPU does not have any specific mitigations in place, and relies on platform-level solutions, such as reduced memory refresh intervals.

### 2.1.9 拒绝服务 

> 2.1.9. Denial of service

WebGPU的应用程序可以访问GPU的内存和计算单元。WebGPU的实现可以限制一个应用程序可分配的GPU内存大小，以保证其他应用程序得到GPU的响应，针对GPU的处理时间间隔，WebGPU的实现可能设置“看门狗”时钟，用于确保一个应用程序不会超过几秒钟了仍无法得到GPU的响应，这些措施类似于WebGL中实现。

> WebGPU applications have access to GPU memory and compute units. A WebGPU implementation may limit the available GPU memory to an application, in order to keep other applications responsive. For GPU processing time, a WebGPU implementation may set up "watchdog" timer that makes sure an application doesn’t cause GPU unresponsiveness for more than a few seconds. These measures are similar to those used in WebGL.

### 2.1.10 工作负载识别

> 2.1.10. Workload identification

WebGPU提供了对在同一台机器上运行的不同程序（或网页）之间可以访问受限的全局共享资源机制。一个应用程序可以尝试间接探测这些全局资源的约束限制范围，以便根据这些共享资源的使用模式及使用大小来推断出其他已打开的网页所执行的工作负载量，这些问题通常类似于Javascript的问题一样，例如，系统内存和CPU的执行吞吐量。针对此类问题，WebGPU没有提供任何额外有效的缓解措施。

> WebGPU provides access to constrained global resources shared between different programs (and web pages) running on the same machine. An application can try to indirectly probe how constrained these global resources are, in order to reason about workloads performed by other open web pages, based on the patterns of usage of these shared resources. These issues are generally analogous to issues with Javascript, such as system memory and CPU execution throughput. WebGPU does not provide any additional mitigations for this.

# 2.1.11. 内存资源

> 2.1.11. Memory resources、

WebGPU暴露了来自于机器全局内存堆，例如VRAM，而直接操作内存堆也极易产生错误分配内存的问题。同时，通过尝试分配内存资源和检测内存分配是否失败等操作，可以推断出系统剩余可用内存的大小（对于给定的堆类型）。

> WebGPU exposes fallible allocations from machine-global memory heaps, such as VRAM. This allows for probing the size of the system’s remaining available memory (for a given heap type) by attempting to allocate and watching for allocation failures.

GPUs内部通常会有一个或者多个（一般只有两个）内存堆，用于所有正在运行程序的共享使用。当堆内存耗尽时，WebGPU 装饰无法再创建资源，很显然，这些内存堆的分配情况的可见性，可能使得恶意应用程序猜测出那些堆被其他应用程序使用了，以及这些应用程序从中分配了多少内存。

> GPUs internally have one or more (typically only two) heaps of memory shared by all running applications. When a heap is depleted, WebGPU would fail to create a resource. This is observable, which may allow a malicious application to guess what heaps are used by other applications, and how much they allocate from them.

### 2.1.12 计算资源

> 2.1.12. Computation resources

如果一个站点和另一个站点同时使用WebGPU，那个么个站点会发现，因为要处理一些额外的工作，导致响应时间增加了。例如，如果一个站点不断提交计算工作任务，并且跟综队列上的工作完成情况，可能明显的发现其他的一些工作任务也开始使用GPU。

> If one site uses WebGPU at the same time as another, it may observe the increase in time it takes to process some work. For example, if a site constantly submits compute workloads and tracks completion of work on the queue, it may observe that something else also started using the GPU.

GPU有许多可以独立测试单元，例如算术单元、文理采样单元、原子操作单元等。恶意的应用程序可能会感知到其中一些单元的使用压力情况，从而通过分析GPU相关单元的应力模式(压力模式)来推测别的应用程序的工作负载量，这点和JavaScript的CPU的实际执行方式非常相似。

> A GPU has many parts that can be tested independently, such as the arithmetic units, texture sampling units, atomic units, etc. A malicious application may sense when some of these units are stressed, and attempt to guess the workload of another application by analyzing the stress patterns. This is analogous to the realities of CPU execution of Javascript.

# 2.1.13. 滥用权限
> 2.1.13. Abuse of capabilities

恶意网站可能会滥用WebGPU的功能来运行对用户或者用户体验毫无意义的计算程序，而只是计算对网站有用的程序。例如，隐藏的密码挖掘、密码破解或者是彩虹表的计算。

> Malicious sites could abuse the capabilities exposed by WebGPU to run computations that don’t benefit the user or their experience and instead only benefit the site. Examples would be hidden crypto-mining, password cracking or rainbow tables computations.

针对这种情况，是不太可能防范这类型的应用对API的调用，因为浏览器是无法区分有效的工作负载和滥用资源的工作负载。这也是网络上所有通用计算能力所普遍面临的一个问题。JavaScript, WebAssembly 或者 WebGL等都是如此。WebGPU只是让一些工作负载程序变的更加容易开发，容易实现，或者是提供比使用WebGL更加高效运行效率。

> It is not possible to guard against these types of uses of the API because the browser is not able to distinguish between valid workloads and abusive workloads. This is a general problem with all general-purpose computation capabilities on the Web: JavaScript, WebAssembly or WebGL. WebGPU only makes some workloads easier to implement, or slightly more efficient to run than using WebGL.

为了缓解这些滥用行为，浏览器可以针对标签页中具有后台运行程序要求资源进行节制，可以警告用户某个标签页正在使用大量的资源，并且限制哪些上下文允许使用WebGPU.

> To mitigate this form of abuse, browsers can throttle operations on background tabs, could warn that a tab is using a lot of resource, and restrict which contexts are allowed to use WebGPU.

用户代理可以启发式的向用户发出一些高能耗、高资源利用率的警告，特别是潜在的恶意使用。如果一个用户代理实现了这样的警告，那么除了包括JavaScript, WebAssembly, WebGL等之外，还应该将WebGPU的使用也纳入到启发式警告机制中。

> User agents can heuristically issue warnings to users about high power use, especially due to potentially malicious usage. If a user agent implements such a warning, it should include WebGPU usage in its heuristics, in addition to JavaScript, WebAssembly, WebGL, and so on.

*  **问题 1** WebGPU是否应该具有权限策略/功能策略 [Issue #gpuweb/gpuweb#3483](https://github.com/gpuweb/gpuweb/issues/3483)

> * **ISSUE 1** Should WebGPU have Permissions Policy / Feature Policy? [Issue #gpuweb/gpuweb#3483]


## 2.2 隐私
2.2. Privacy Considerations

WebGPU的隐私策略与WebGL的类似。GPU API是非常复杂的，并且必须公开GPU设备各个方面的能力，以便开发者能够有效的利用这些能力。一般关于隐私方面的缓解措施包括对潜在的识别信息进行标准化或者分档，并在可能的情况下执行统一的行为。

> The privacy considerations for WebGPU are similar to those of WebGL. GPU APIs are complex and must expose various aspects of a device’s capabilities out of necessity in order to enable developers to take advantage of those capabilities effectively. The general mitigation approach involves normalizing or binning potentially identifying information and enforcing uniform behavior where possible.

一个用户代理不得显示超过 32 个可区分的配置或桶。

> A user agent must not reveal more than 32 distinguishable configurations or buckets.

### 2.2.1 特定机器的特征和限制

> 2.2.1. Machine-specific features and limits

WebGPU 可以暴躁出很多关于GPU底层架构和设备几何的细节。包括可用的物理适配器在内，以及其他一些可选用的特定硬件功能，都可对其要使用的GPU和CPU资源进行限制(例如，限制最大的纹理尺寸)。

> WebGPU can expose a lot of detail on the underlying GPU architecture and the device geometry. This includes available physical adapters, many limits on the GPU and CPU resources that could be used (such as the maximum texture size), and any optional hardware-specific capabilities that are available.

用户代理没有义务暴露真正的硬件限制，但他们完全可以控制机器细节的暴露程度。其中一个策略就是，不要将指纹识别（身份信息）信息绑定到所有的目标平台信息中，而是分成少数几个组中。一般来说，暴露硬件对隐私的影响限制与WebGL是一致的。

> User agents are not obligated to expose the real hardware limits, they are in full control of how much the machine specifics are exposed. One strategy to reduce fingerprinting is binning all the target platforms into a few number of bins. In general, the privacy impact of exposing the hardware limits matches the one of WebGL.


[默认限制策略](https://www.w3.org/TR/webgpu/#limit-default)会被用于满足绝大多数应用程序在不要求更高限制的情况下正常工作。所有API的使用都要根据限制要求来进行验证，所以实际的硬件能力不会意外的就暴露给了用户。

> The default limits are also deliberately high enough to allow most applications to work without requesting higher limits. All the usage of the API is validated according to the requested limits, so the actual hardware capabilities are not exposed to the users by accident.

### 2.2.3 特定机器的性能
> 2.2.3. Machine-specific performance

区分用户的另一个因素是测量GPU上运行特定操作的性能。即使是低精度的计时器，重复的执行一个操作也能显示出用户的机器是否可以快速的响应特定的工作负载。这是一个相当常的因素（在WebGL和JavaScript中一样存在），但它也是一个低信号量，而且相对的难以真正的规范化分析(量化分析)。

> Another factor for differentiating users is measuring the performance of specific operations on the GPU. Even with low precision timing, repeated execution of an operation can show if the user’s machine is fast at specific workloads. This is a fairly common vector (present in both WebGL and Javascript), but it’s also low-signal and relatively intractable to truly normalize.

WebGPU的计算管道通过固定功能的硬件单元直接暴露了GPU的访问。这为特定的独特的设备身份识别带来了额外的风险。用户代理可以采取必要措施，通过将逻辑上的GPU调用与实际的计算单元分开的策略，以便减少这样的风险。

> WebGPU compute pipelines expose access to GPU unobstructed by the fixed-function hardware. This poses an additional risk for unique device fingerprinting. User agents can take steps to dissociate logical GPU invocations with actual compute units to reduce this risk.

2.2.4 用户代理状态(模式)

> 2.2.4. User Agent State

本规范文档最开始并没有定义任何额外的用户代理状态。然而，很明显，用户代理将会需要一定的编译缓存，用于处理和存储珍贵的编译结果，例如，这些编译结果可由 [GPUShaderModule](https://www.w3.org/TR/webgpu/#gpushadermodule), [GPURenderPipeline](https://www.w3.org/TR/webgpu/#gpurenderpipeline) 和 [GPUComputePipeLine](https://www.w3.org/TR/webgpu/#gpucomputepipeline) 得到。这些缓存对于改善WebGPU的应用程序在第一次访问时的加载时间方面非常重要。

> This specification doesn’t define any additional user-agent state for an origin. However it is expected that user agents will have compilation caches for the result of expensive compilation like GPUShaderModule, GPURenderPipeline and GPUComputePipeline. These caches are important to improve the loading time of WebGPU applications after the first visit.

从本规范层面来说，这些缓存与难以置信的忆速编译过程是无关的，但是从应用程序层来来说，就很容易测量出[createComputePipelineAsync()](https://www.w3.org/TR/webgpu/#dom-gpudevice-createcomputepipelineasync)函数的调用花费了多少时间来解决问题。这也会导致跨域的信息泄露(例如“用户是否采用一个特定的着色器程序访问了别的网站”)，所以用户代理应该遵循[存储分区](https://github.com/privacycg/storage-partitioning)的最佳实践方案。

> For the specification, these caches are indifferentiable from incredibly fast compilation, but for applications it would be easy to measure how long createComputePipelineAsync() takes to resolve. This can leak information across origins (like "did the user access a site with this specific shader") so user agents should follow the best practices in storage partitioning.

系统的GPU驱动也可能有它自己的缓存来编译着色器程序和管道。用户代理可能需要在所有可能的情况下禁用这项功能，或者为每个着色器程序添加额外的分区数据，以便让GPU驱动程序认为他们是不一样的。

> The system’s GPU driver may also have its own cache of compiled shaders and pipelines. User agents may want to disable these when at all possible, or add per-partition data to shaders in ways that will make the GPU driver consider them different.

### 2.2.5. 驱动bugs

> 2.2.5. Driver bugs

除了[2.1节安全策略](https://www.w3.org/TR/webgpu/#security-driver-bugs)中所考虑的问题外，驱动程序bugs可能会带来行为上的差异，这点可以用来作为观察区分用户的一种方法。在安全策略的中提到的缓解措施同样适用于此，其中包括和GPU供应商协作，并在用户代理中针对已知的问题采取必要变通措施。

> In addition to the concerns outlined in Security Considerations, driver bugs may introduce differences in behavior that can be observed as a method of differentiating users. The mitigations mentioned in Security Considerations apply here as well, including coordinating with GPU vendors and implementing workarounds for known issues in the user agent.

### 2.2.6. 适配器标识符

> 2.2.6. Adapter Identifiers

过去在WebGL上的开发经验表明，开发人员有非常合理的需求，需要能够识别他们代码是运行在那个GPU上的，以便更好的创建和维护基于GPU的更具鲁棒性的代码。例如，识别具有已知驱动bugs的适配器，可以采用一定方式回避这些bug，或者避免在特定硬件上表现出比预期更差的结果。

> Past experience with WebGL has demonstrated that developers have a legitimate need to be able to identify the GPU their code is running on in order to create and maintain robust GPU-based content. For example, to identify adapters with known driver bugs in order to work around them or to avoid features that perform more poorly than expected on a given class of hardware.

但是暴露适配器标识符也自然而然的扩大了可用身份信息量，所以人们也希望能够限制我们识别适配器的精确度。

> But exposing adapter identifiers also naturally expands the amount of fingerprinting information available, so there’s a desire to limit the precision with which we identify the adapter.

这里，有几项缓解措施可以应用，以便在实现更具鲁棒性的程序和保护隐私之间取得一定的平衡。首先是当用户开始通过浏览器使用GPU时，用户代理可以通过识别和解决已知的驱动程序问题来减少开发人员的心智负担。

> There are several mitigations that can be applied to strike a balance between enabling robust content and preserving privacy. First is that user agents can reduce the burden on developers by identifying and working around known driver issues, as they have since browsers began making use of GPUs.

当适配器标识符以默认的方式暴露时，它们应该在保持可用性的前提下，尽可能的放松限制条件。例如，尽可能的去识别适配器的供应商和通用架构，而不是去识别正在作用的特定适配器。同样的，在一些情况下，适配器标识符是真实适配器的非常合理的代理，这种代理是可以报告的。
> When adapter identifiers are exposed by default they should be as broad as possible while still being useful. Possibly identifying, for example, the adapter’s vendor and general architecture without identifying the specific adapter in use. Similarly, in some cases identifiers for an adapter that is considered a reasonable proxy for the actual adapter may be reported.

在一些情况下，适配器完整且详细的信息是非常有用的(例如，在填写bug缺陷报告时)， 此时可以征求用户的许可，将他们硬件的额外信息透露给该页面。

> In cases where full and detailed information about the adapter is useful (for example: when filing bug reports) the user can be asked for consent to reveal additional information about their hardware to the page.

最后，如果用户代理在认为合理的，必要的情况，例如在增强的隐私模式下，它将始终拥有不报告适配器标识符的决定权。
> Finally, the user agent will always have the discretion to not report adapter identifiers at all if it considers it appropriate, such as in enhanced privacy modes.