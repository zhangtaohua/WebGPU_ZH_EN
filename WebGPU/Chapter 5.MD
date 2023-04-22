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

# 5\. Buffers
## 5.1. GPUBuffer

> 5\. Buffers    

> 5.1. GPUBuffer   


一个 GPUBuffer 代表一个可用于GPU操作的内存块。数据是以线性布局方式存储的，这意味着分配的每个字节都可以通过基于GPUBuffer 开始地址的偏移量来寻址，但根据不同的操作，会存在相应对齐限制。一些 GPUBuffer 可以被映射，这使得该内存块可以通过一个称为其映射的 ArrayBuffer 来访问。

GPUBuffers 是通过 createBuffer() 创建的，缓冲区可以在创建时被映射。

> A GPUBuffer represents a block of memory that can be used in GPU operations. Data is stored in linear layout, meaning that each byte of the allocation can be addressed by its offset from the start of the GPUBuffer, subject to alignment restrictions depending on the operation. Some GPUBuffers can be mapped which makes the block of memory accessible via an ArrayBuffer called its mapping.

> GPUBuffers are created via createBuffer(). Buffers may be mappedAtCreation.
```
[Exposed=(Window, DedicatedWorker), SecureContext]
interface GPUBuffer {
    readonly attribute GPUSize64 size;
    readonly attribute GPUBufferUsageFlags usage;

    readonly attribute GPUBufferMapState mapState;

    Promise<undefined> mapAsync(GPUMapModeFlags mode, optional GPUSize64 offset = 0, optional GPUSize64 size);
    ArrayBuffer getMappedRange(optional GPUSize64 offset = 0, optional GPUSize64 size);
    undefined unmap();

    undefined destroy();
};
GPUBuffer includes GPUObjectBase;

enum GPUBufferMapState {
    "unmapped",
    "pending",
    "mapped"
};
```

GPUBuffer 有以下 不可变的属性。    
**size，类型为：GPUSize64，只读**    
GPUBuffer 分配的字节数长度。    
**usage，类型为：GPUBufferUsageFlags，只读**    
GPUBuffer 允许使用的方法。    
**[[internals]], 类型是：buffer internals, 只读, 可覆盖**    

> GPUBuffer has the following immutable properties:    
> **size, of type GPUSize64, readonly**    
> The length of the GPUBuffer allocation in bytes.    
> **usage, of type GPUBufferUsageFlags, readonly**   
> The allowed usages for this GPUBuffer.    
> **[[internals]], of type buffer internals, readonly, override**    

GPUBuffer有以下上下文时间线属性：    
**mapState，类型为：GPUBufferMapState，只读**
用于描述缓冲区的当前GPU缓冲区映射状态（GPUBufferMapState），状态值有以下几种：    
* "unmapped"：    
  缓冲区没有被映射，无法被this.getMappedRange()使用。
* "pending"    
  已经发出对缓冲区进行映射的请求，但正在等待。它可能成功，也可能在mapAsync()中验证失败。
* "mapped"    
  缓冲区已经被映射，this.getMappedRange()函数已经使用了。

> GPUBuffer has the following content timeline properties:    
> **mapState, of type GPUBufferMapState, readonly**    
> The current GPUBufferMapState of the buffer:    
>>    "unmapped"    
>>        The buffer is not mapped for use by this.getMappedRange().    
>>    "pending"     
>>        A mapping of the buffer has been requested, but is pending. It may succeed, or fail validation in mapAsync().    
>>    "mapped"    
>>        The buffer is mapped and this.getMappedRange() may be used.

getter 的步骤是：
**上下文时间线的步骤：**
1. 如果this.[[mapping]]不是空的，返回 "mapped"。
2. 如果this.[[pending_map]]不是空的，返回 "pending"。
3. 返回 "unmapped"。

> The getter steps are:    
> **Content timeline steps:**    
> 1. If this.[[mapping]] is not null, return "mapped".    
> 2. If this.[[pending_map]] is not null, return "pending".    
> 3. Return "unmapped".    

[[pending_map]]，类型为 Promise\<void\> 或 null ，初始值为：null    
> * Promise 的结果是由当前正在进行的 mapAsync()调用返回的。
> * 绝对不能有一个以上的pending map，因为如果一个请求正在进行，那么 mapAsync() 会立即拒绝。

> **[[pending_map]], of type Promise<void> or null, initially null**    
> The Promise returned by the currently-pending mapAsync() call.    
> There is never more than one pending map, because mapAsync() will refuse immediately if a request is already in flight.    

[[mapping]]，类型为：活动的缓冲区映射 或 null，初始值为：null    
* 当且仅当缓冲区当前被映射为 可通过getMappedRange()使用时 才可设置mapping的值，其值结构见下文描述。否则为空（即使有一个[[pending_map]]）。

> **[[mapping]], of type active buffer mapping or null, initially null**    
> Set if and only if the buffer is currently mapped for use by getMappedRange(). Null otherwise (even if there is a [[pending_map]]).    

一个活动的缓冲区的mapping 数据 是一个具有以下字段的结构体：
> **data, 类型为： Data Block**    
> 此字段描述 GPUBuffer的映射。该数据通过ArrayBuffers访问，ArrayBuffers也是该数据的视图，由getMappedRange()返回并存储在视图中。    
> **mode, 类型为： GPUMapModeFlags**    
映射的GPUMapModeFlags，是在 mapAsync() 或 createBuffer() 相应的调用中指定的。    
> **range, 类型为：tuple [unsigned long long, unsigned long long].**
被映射的这个 GPUBuffer 的范围。    
> **views，类型为list<ArrayBuffer>**
通过getMappedRange()返回给应用程序的 ArrayBuffers。它们是被跟踪的，所以当 unmap() 被调用时它们可以被分离。

> An active buffer mapping is a structure with the following fields:    
>> **data, of type Data Block**    
>>  The mapping for this GPUBuffer. This data is accessed through ArrayBuffers which are views onto this data, returned by getMappedRange() and stored in views.    
>> **mode, of type GPUMapModeFlags**    
>> The GPUMapModeFlags of the map, as specified in the corresponding call to mapAsync() or createBuffer().    
>> **range, of type tuple [unsigned long long, unsigned long long]**    
>> The range of this GPUBuffer that is mapped.
>> **views, of type list<ArrayBuffer>**    
>> The ArrayBuffers returned via getMappedRange() to the application. They are tracked so they can be detached when unmap() is called.

要初始化一个具有模式为 mode 和范围为range的活动缓冲区映射，参考以下步骤：
1. 让 size 为range[1] - range[0]。
2. 让数据为 ?CreateByteDataBlock(size)调用的返回结果。
> **注意：** 这可能导致抛出一个 RangeError。为了一致性和可预测性:
>    * 对于任何大小的 new ArrayBuffer()操作，如果在一个给定的时候会成功，下次这个大小的分配任务应该在那个时刻就成功。
>    * 对于任何大小的 new ArrayBuffer()操作，如果会确定地抛出 RangeError，下次这个大小的分配任务也应该如此。
3. 返回一个带有如下状态的活动缓冲区映射:
  * 数据 设置为 data。
  * 模式 设置为 mode。
  * 范围 设置为 range。
  * 视图 设置为 []。

>  To initialize an active buffer mapping with mode mode and range range:    
> 1. Let size be range[1] - range[0].    
> 2. Let data be ?CreateByteDataBlock(size).    
>   **Note:** This may result in a RangeError being thrown. For consistency and predictability:
>    * For any size at which new ArrayBuffer() would succeed at a given moment, this allocation should succeed at that moment.
>    * For any size at which new ArrayBuffer() deterministically throws a RangeError, this allocation should as well.
> 3. Return an active buffer mapping with:
>    * data set to data.
>    * mode set to mode.
>    * range set to range.
>    * views set to [].

GPUBuffer 的内部对象是 buffer internals，它是通过以下 设备时间线插槽 扩展了内部对象：    
**state**    
该缓冲区的当前内部状态。    
> **"available"**      
> 缓冲区可以在队列操作中使用（除非它是无效的）。    
> **"unavailable"**    
> 缓冲区由于被映射而不能在队列操作中使用。    
> **"destroyed"**   
> 缓冲区由于被 destroy() 了，所以不能用于任何操作。   

> GPUBuffer's internal object is buffer internals, which extends internal object with the following device timeline slots:    
> **state**    
>    The current internal state of the buffer:    
>>   **"available"**    
>>   The buffer may be used in queue operations (unless it is invalid).    
>>   **"unavailable"**    
>>   The buffer may not be used in queue operations due to being mapped.    
>>   **"destroyed"**    
>>   The buffer may not be used in any operations due to being destroy()ed.    


> EXAMPLE 12    

<figure>
     <figcaption>对一个缓冲区进行映射和解除映射 详情图</figcaption>
     <figcaption>Mapping and unmapping a buffer.</figcaption>
     <object data="img/buffer-map-unmap.mmd.svg" type="image/svg+xml">
     <svg xmlns="http://www.w3.org/2000/svg" id="mermaid-0" width="100%" height="747" style="max-width: 544px; background-color: black;" viewBox="-72 -10 544 747" role="img" aria-labelledby="chart-title-mermaid-0 chart-desc-mermaid-0"><title id="chart-title-mermaid-0"/><desc id="chart-desc-mermaid-0"/><style>#mermaid-0 {font-family:"trebuchet ms",verdana,arial,sans-serif;font-size:16px;fill:#ccc;}#mermaid-0 .error-icon{fill:#a44141;}#mermaid-0 .error-text{fill:#ddd;stroke:#ddd;}#mermaid-0 .edge-thickness-normal{stroke-width:2px;}#mermaid-0 .edge-thickness-thick{stroke-width:3.5px;}#mermaid-0 .edge-pattern-solid{stroke-dasharray:0;}#mermaid-0 .edge-pattern-dashed{stroke-dasharray:3;}#mermaid-0 .edge-pattern-dotted{stroke-dasharray:2;}#mermaid-0 .marker{fill:lightgrey;stroke:lightgrey;}#mermaid-0 .marker.cross{stroke:lightgrey;}#mermaid-0 svg{font-family:"trebuchet ms",verdana,arial,sans-serif;font-size:16px;}#mermaid-0 .actor{stroke:#81B1DB;fill:#1f2020;}#mermaid-0 text.actor&gt;tspan{fill:lightgrey;stroke:none;}#mermaid-0 .actor-line{stroke:lightgrey;}#mermaid-0 .messageLine0{stroke-width:1.5;stroke-dasharray:none;stroke:lightgrey;}#mermaid-0 .messageLine1{stroke-width:1.5;stroke-dasharray:2,2;stroke:lightgrey;}#mermaid-0 #arrowhead path{fill:lightgrey;stroke:lightgrey;}#mermaid-0 .sequenceNumber{fill:black;}#mermaid-0 #sequencenumber{fill:lightgrey;}#mermaid-0 #crosshead path{fill:lightgrey;stroke:lightgrey;}#mermaid-0 .messageText{fill:lightgrey;stroke:lightgrey;}#mermaid-0 .labelBox{stroke:#81B1DB;fill:#1f2020;}#mermaid-0 .labelText,#mermaid-0 .labelText&gt;tspan{fill:lightgrey;stroke:none;}#mermaid-0 .loopText,#mermaid-0 .loopText&gt;tspan{fill:lightgrey;stroke:none;}#mermaid-0 .loopLine{stroke-width:2px;stroke-dasharray:2,2;stroke:#81B1DB;fill:#81B1DB;}#mermaid-0 .note{stroke:hsl(180, 0%, 18.3529411765%);fill:hsl(180, 1.5873015873%, 28.3529411765%);}#mermaid-0 .noteText,#mermaid-0 .noteText&gt;tspan{fill:rgb(183.8476190475, 181.5523809523, 181.5523809523);stroke:none;}#mermaid-0 .activation0{fill:hsl(180, 1.5873015873%, 28.3529411765%);stroke:#81B1DB;}#mermaid-0 .activation1{fill:hsl(180, 1.5873015873%, 28.3529411765%);stroke:#81B1DB;}#mermaid-0 .activation2{fill:hsl(180, 1.5873015873%, 28.3529411765%);stroke:#81B1DB;}#mermaid-0 .actorPopupMenu{position:absolute;}#mermaid-0 .actorPopupMenuPanel{position:absolute;fill:#1f2020;box-shadow:0px 8px 16px 0px rgba(0,0,0,0.2);filter:drop-shadow(3px 5px 2px rgb(0 0 0 / 0.4));}#mermaid-0 .actor-man line{stroke:#81B1DB;fill:#1f2020;}#mermaid-0 .actor-man circle,#mermaid-0 line{stroke:#81B1DB;fill:#1f2020;stroke-width:2px;}#mermaid-0 .messageText{stroke:none;}#mermaid-0 :root{--mermaid-font-family:"trebuchet ms",verdana,arial,sans-serif;}</style><g/><defs><symbol id="computer" width="24" height="24"><path transform="scale(.5)" d="M2 2v13h20v-13h-20zm18 11h-16v-9h16v9zm-10.228 6l.466-1h3.524l.467 1h-4.457zm14.228 3h-24l2-6h2.104l-1.33 4h18.45l-1.297-4h2.073l2 6zm-5-10h-14v-7h14v7z"/></symbol></defs><defs><symbol id="database" fill-rule="evenodd" clip-rule="evenodd"><path transform="scale(.5)" d="M12.258.001l.256.004.255.005.253.008.251.01.249.012.247.015.246.016.242.019.241.02.239.023.236.024.233.027.231.028.229.031.225.032.223.034.22.036.217.038.214.04.211.041.208.043.205.045.201.046.198.048.194.05.191.051.187.053.183.054.18.056.175.057.172.059.168.06.163.061.16.063.155.064.15.066.074.033.073.033.071.034.07.034.069.035.068.035.067.035.066.035.064.036.064.036.062.036.06.036.06.037.058.037.058.037.055.038.055.038.053.038.052.038.051.039.05.039.048.039.047.039.045.04.044.04.043.04.041.04.04.041.039.041.037.041.036.041.034.041.033.042.032.042.03.042.029.042.027.042.026.043.024.043.023.043.021.043.02.043.018.044.017.043.015.044.013.044.012.044.011.045.009.044.007.045.006.045.004.045.002.045.001.045v17l-.001.045-.002.045-.004.045-.006.045-.007.045-.009.044-.011.045-.012.044-.013.044-.015.044-.017.043-.018.044-.02.043-.021.043-.023.043-.024.043-.026.043-.027.042-.029.042-.03.042-.032.042-.033.042-.034.041-.036.041-.037.041-.039.041-.04.041-.041.04-.043.04-.044.04-.045.04-.047.039-.048.039-.05.039-.051.039-.052.038-.053.038-.055.038-.055.038-.058.037-.058.037-.06.037-.06.036-.062.036-.064.036-.064.036-.066.035-.067.035-.068.035-.069.035-.07.034-.071.034-.073.033-.074.033-.15.066-.155.064-.16.063-.163.061-.168.06-.172.059-.175.057-.18.056-.183.054-.187.053-.191.051-.194.05-.198.048-.201.046-.205.045-.208.043-.211.041-.214.04-.217.038-.22.036-.223.034-.225.032-.229.031-.231.028-.233.027-.236.024-.239.023-.241.02-.242.019-.246.016-.247.015-.249.012-.251.01-.253.008-.255.005-.256.004-.258.001-.258-.001-.256-.004-.255-.005-.253-.008-.251-.01-.249-.012-.247-.015-.245-.016-.243-.019-.241-.02-.238-.023-.236-.024-.234-.027-.231-.028-.228-.031-.226-.032-.223-.034-.22-.036-.217-.038-.214-.04-.211-.041-.208-.043-.204-.045-.201-.046-.198-.048-.195-.05-.19-.051-.187-.053-.184-.054-.179-.056-.176-.057-.172-.059-.167-.06-.164-.061-.159-.063-.155-.064-.151-.066-.074-.033-.072-.033-.072-.034-.07-.034-.069-.035-.068-.035-.067-.035-.066-.035-.064-.036-.063-.036-.062-.036-.061-.036-.06-.037-.058-.037-.057-.037-.056-.038-.055-.038-.053-.038-.052-.038-.051-.039-.049-.039-.049-.039-.046-.039-.046-.04-.044-.04-.043-.04-.041-.04-.04-.041-.039-.041-.037-.041-.036-.041-.034-.041-.033-.042-.032-.042-.03-.042-.029-.042-.027-.042-.026-.043-.024-.043-.023-.043-.021-.043-.02-.043-.018-.044-.017-.043-.015-.044-.013-.044-.012-.044-.011-.045-.009-.044-.007-.045-.006-.045-.004-.045-.002-.045-.001-.045v-17l.001-.045.002-.045.004-.045.006-.045.007-.045.009-.044.011-.045.012-.044.013-.044.015-.044.017-.043.018-.044.02-.043.021-.043.023-.043.024-.043.026-.043.027-.042.029-.042.03-.042.032-.042.033-.042.034-.041.036-.041.037-.041.039-.041.04-.041.041-.04.043-.04.044-.04.046-.04.046-.039.049-.039.049-.039.051-.039.052-.038.053-.038.055-.038.056-.038.057-.037.058-.037.06-.037.061-.036.062-.036.063-.036.064-.036.066-.035.067-.035.068-.035.069-.035.07-.034.072-.034.072-.033.074-.033.151-.066.155-.064.159-.063.164-.061.167-.06.172-.059.176-.057.179-.056.184-.054.187-.053.19-.051.195-.05.198-.048.201-.046.204-.045.208-.043.211-.041.214-.04.217-.038.22-.036.223-.034.226-.032.228-.031.231-.028.234-.027.236-.024.238-.023.241-.02.243-.019.245-.016.247-.015.249-.012.251-.01.253-.008.255-.005.256-.004.258-.001.258.001zm-9.258 20.499v.01l.001.021.003.021.004.022.005.021.006.022.007.022.009.023.01.022.011.023.012.023.013.023.015.023.016.024.017.023.018.024.019.024.021.024.022.025.023.024.024.025.052.049.056.05.061.051.066.051.07.051.075.051.079.052.084.052.088.052.092.052.097.052.102.051.105.052.11.052.114.051.119.051.123.051.127.05.131.05.135.05.139.048.144.049.147.047.152.047.155.047.16.045.163.045.167.043.171.043.176.041.178.041.183.039.187.039.19.037.194.035.197.035.202.033.204.031.209.03.212.029.216.027.219.025.222.024.226.021.23.02.233.018.236.016.24.015.243.012.246.01.249.008.253.005.256.004.259.001.26-.001.257-.004.254-.005.25-.008.247-.011.244-.012.241-.014.237-.016.233-.018.231-.021.226-.021.224-.024.22-.026.216-.027.212-.028.21-.031.205-.031.202-.034.198-.034.194-.036.191-.037.187-.039.183-.04.179-.04.175-.042.172-.043.168-.044.163-.045.16-.046.155-.046.152-.047.148-.048.143-.049.139-.049.136-.05.131-.05.126-.05.123-.051.118-.052.114-.051.11-.052.106-.052.101-.052.096-.052.092-.052.088-.053.083-.051.079-.052.074-.052.07-.051.065-.051.06-.051.056-.05.051-.05.023-.024.023-.025.021-.024.02-.024.019-.024.018-.024.017-.024.015-.023.014-.024.013-.023.012-.023.01-.023.01-.022.008-.022.006-.022.006-.022.004-.022.004-.021.001-.021.001-.021v-4.127l-.077.055-.08.053-.083.054-.085.053-.087.052-.09.052-.093.051-.095.05-.097.05-.1.049-.102.049-.105.048-.106.047-.109.047-.111.046-.114.045-.115.045-.118.044-.12.043-.122.042-.124.042-.126.041-.128.04-.13.04-.132.038-.134.038-.135.037-.138.037-.139.035-.142.035-.143.034-.144.033-.147.032-.148.031-.15.03-.151.03-.153.029-.154.027-.156.027-.158.026-.159.025-.161.024-.162.023-.163.022-.165.021-.166.02-.167.019-.169.018-.169.017-.171.016-.173.015-.173.014-.175.013-.175.012-.177.011-.178.01-.179.008-.179.008-.181.006-.182.005-.182.004-.184.003-.184.002h-.37l-.184-.002-.184-.003-.182-.004-.182-.005-.181-.006-.179-.008-.179-.008-.178-.01-.176-.011-.176-.012-.175-.013-.173-.014-.172-.015-.171-.016-.17-.017-.169-.018-.167-.019-.166-.02-.165-.021-.163-.022-.162-.023-.161-.024-.159-.025-.157-.026-.156-.027-.155-.027-.153-.029-.151-.03-.15-.03-.148-.031-.146-.032-.145-.033-.143-.034-.141-.035-.14-.035-.137-.037-.136-.037-.134-.038-.132-.038-.13-.04-.128-.04-.126-.041-.124-.042-.122-.042-.12-.044-.117-.043-.116-.045-.113-.045-.112-.046-.109-.047-.106-.047-.105-.048-.102-.049-.1-.049-.097-.05-.095-.05-.093-.052-.09-.051-.087-.052-.085-.053-.083-.054-.08-.054-.077-.054v4.127zm0-5.654v.011l.001.021.003.021.004.021.005.022.006.022.007.022.009.022.01.022.011.023.012.023.013.023.015.024.016.023.017.024.018.024.019.024.021.024.022.024.023.025.024.024.052.05.056.05.061.05.066.051.07.051.075.052.079.051.084.052.088.052.092.052.097.052.102.052.105.052.11.051.114.051.119.052.123.05.127.051.131.05.135.049.139.049.144.048.147.048.152.047.155.046.16.045.163.045.167.044.171.042.176.042.178.04.183.04.187.038.19.037.194.036.197.034.202.033.204.032.209.03.212.028.216.027.219.025.222.024.226.022.23.02.233.018.236.016.24.014.243.012.246.01.249.008.253.006.256.003.259.001.26-.001.257-.003.254-.006.25-.008.247-.01.244-.012.241-.015.237-.016.233-.018.231-.02.226-.022.224-.024.22-.025.216-.027.212-.029.21-.03.205-.032.202-.033.198-.035.194-.036.191-.037.187-.039.183-.039.179-.041.175-.042.172-.043.168-.044.163-.045.16-.045.155-.047.152-.047.148-.048.143-.048.139-.05.136-.049.131-.05.126-.051.123-.051.118-.051.114-.052.11-.052.106-.052.101-.052.096-.052.092-.052.088-.052.083-.052.079-.052.074-.051.07-.052.065-.051.06-.05.056-.051.051-.049.023-.025.023-.024.021-.025.02-.024.019-.024.018-.024.017-.024.015-.023.014-.023.013-.024.012-.022.01-.023.01-.023.008-.022.006-.022.006-.022.004-.021.004-.022.001-.021.001-.021v-4.139l-.077.054-.08.054-.083.054-.085.052-.087.053-.09.051-.093.051-.095.051-.097.05-.1.049-.102.049-.105.048-.106.047-.109.047-.111.046-.114.045-.115.044-.118.044-.12.044-.122.042-.124.042-.126.041-.128.04-.13.039-.132.039-.134.038-.135.037-.138.036-.139.036-.142.035-.143.033-.144.033-.147.033-.148.031-.15.03-.151.03-.153.028-.154.028-.156.027-.158.026-.159.025-.161.024-.162.023-.163.022-.165.021-.166.02-.167.019-.169.018-.169.017-.171.016-.173.015-.173.014-.175.013-.175.012-.177.011-.178.009-.179.009-.179.007-.181.007-.182.005-.182.004-.184.003-.184.002h-.37l-.184-.002-.184-.003-.182-.004-.182-.005-.181-.007-.179-.007-.179-.009-.178-.009-.176-.011-.176-.012-.175-.013-.173-.014-.172-.015-.171-.016-.17-.017-.169-.018-.167-.019-.166-.02-.165-.021-.163-.022-.162-.023-.161-.024-.159-.025-.157-.026-.156-.027-.155-.028-.153-.028-.151-.03-.15-.03-.148-.031-.146-.033-.145-.033-.143-.033-.141-.035-.14-.036-.137-.036-.136-.037-.134-.038-.132-.039-.13-.039-.128-.04-.126-.041-.124-.042-.122-.043-.12-.043-.117-.044-.116-.044-.113-.046-.112-.046-.109-.046-.106-.047-.105-.048-.102-.049-.1-.049-.097-.05-.095-.051-.093-.051-.09-.051-.087-.053-.085-.052-.083-.054-.08-.054-.077-.054v4.139zm0-5.666v.011l.001.02.003.022.004.021.005.022.006.021.007.022.009.023.01.022.011.023.012.023.013.023.015.023.016.024.017.024.018.023.019.024.021.025.022.024.023.024.024.025.052.05.056.05.061.05.066.051.07.051.075.052.079.051.084.052.088.052.092.052.097.052.102.052.105.051.11.052.114.051.119.051.123.051.127.05.131.05.135.05.139.049.144.048.147.048.152.047.155.046.16.045.163.045.167.043.171.043.176.042.178.04.183.04.187.038.19.037.194.036.197.034.202.033.204.032.209.03.212.028.216.027.219.025.222.024.226.021.23.02.233.018.236.017.24.014.243.012.246.01.249.008.253.006.256.003.259.001.26-.001.257-.003.254-.006.25-.008.247-.01.244-.013.241-.014.237-.016.233-.018.231-.02.226-.022.224-.024.22-.025.216-.027.212-.029.21-.03.205-.032.202-.033.198-.035.194-.036.191-.037.187-.039.183-.039.179-.041.175-.042.172-.043.168-.044.163-.045.16-.045.155-.047.152-.047.148-.048.143-.049.139-.049.136-.049.131-.051.126-.05.123-.051.118-.052.114-.051.11-.052.106-.052.101-.052.096-.052.092-.052.088-.052.083-.052.079-.052.074-.052.07-.051.065-.051.06-.051.056-.05.051-.049.023-.025.023-.025.021-.024.02-.024.019-.024.018-.024.017-.024.015-.023.014-.024.013-.023.012-.023.01-.022.01-.023.008-.022.006-.022.006-.022.004-.022.004-.021.001-.021.001-.021v-4.153l-.077.054-.08.054-.083.053-.085.053-.087.053-.09.051-.093.051-.095.051-.097.05-.1.049-.102.048-.105.048-.106.048-.109.046-.111.046-.114.046-.115.044-.118.044-.12.043-.122.043-.124.042-.126.041-.128.04-.13.039-.132.039-.134.038-.135.037-.138.036-.139.036-.142.034-.143.034-.144.033-.147.032-.148.032-.15.03-.151.03-.153.028-.154.028-.156.027-.158.026-.159.024-.161.024-.162.023-.163.023-.165.021-.166.02-.167.019-.169.018-.169.017-.171.016-.173.015-.173.014-.175.013-.175.012-.177.01-.178.01-.179.009-.179.007-.181.006-.182.006-.182.004-.184.003-.184.001-.185.001-.185-.001-.184-.001-.184-.003-.182-.004-.182-.006-.181-.006-.179-.007-.179-.009-.178-.01-.176-.01-.176-.012-.175-.013-.173-.014-.172-.015-.171-.016-.17-.017-.169-.018-.167-.019-.166-.02-.165-.021-.163-.023-.162-.023-.161-.024-.159-.024-.157-.026-.156-.027-.155-.028-.153-.028-.151-.03-.15-.03-.148-.032-.146-.032-.145-.033-.143-.034-.141-.034-.14-.036-.137-.036-.136-.037-.134-.038-.132-.039-.13-.039-.128-.041-.126-.041-.124-.041-.122-.043-.12-.043-.117-.044-.116-.044-.113-.046-.112-.046-.109-.046-.106-.048-.105-.048-.102-.048-.1-.05-.097-.049-.095-.051-.093-.051-.09-.052-.087-.052-.085-.053-.083-.053-.08-.054-.077-.054v4.153zm8.74-8.179l-.257.004-.254.005-.25.008-.247.011-.244.012-.241.014-.237.016-.233.018-.231.021-.226.022-.224.023-.22.026-.216.027-.212.028-.21.031-.205.032-.202.033-.198.034-.194.036-.191.038-.187.038-.183.04-.179.041-.175.042-.172.043-.168.043-.163.045-.16.046-.155.046-.152.048-.148.048-.143.048-.139.049-.136.05-.131.05-.126.051-.123.051-.118.051-.114.052-.11.052-.106.052-.101.052-.096.052-.092.052-.088.052-.083.052-.079.052-.074.051-.07.052-.065.051-.06.05-.056.05-.051.05-.023.025-.023.024-.021.024-.02.025-.019.024-.018.024-.017.023-.015.024-.014.023-.013.023-.012.023-.01.023-.01.022-.008.022-.006.023-.006.021-.004.022-.004.021-.001.021-.001.021.001.021.001.021.004.021.004.022.006.021.006.023.008.022.01.022.01.023.012.023.013.023.014.023.015.024.017.023.018.024.019.024.02.025.021.024.023.024.023.025.051.05.056.05.06.05.065.051.07.052.074.051.079.052.083.052.088.052.092.052.096.052.101.052.106.052.11.052.114.052.118.051.123.051.126.051.131.05.136.05.139.049.143.048.148.048.152.048.155.046.16.046.163.045.168.043.172.043.175.042.179.041.183.04.187.038.191.038.194.036.198.034.202.033.205.032.21.031.212.028.216.027.22.026.224.023.226.022.231.021.233.018.237.016.241.014.244.012.247.011.25.008.254.005.257.004.26.001.26-.001.257-.004.254-.005.25-.008.247-.011.244-.012.241-.014.237-.016.233-.018.231-.021.226-.022.224-.023.22-.026.216-.027.212-.028.21-.031.205-.032.202-.033.198-.034.194-.036.191-.038.187-.038.183-.04.179-.041.175-.042.172-.043.168-.043.163-.045.16-.046.155-.046.152-.048.148-.048.143-.048.139-.049.136-.05.131-.05.126-.051.123-.051.118-.051.114-.052.11-.052.106-.052.101-.052.096-.052.092-.052.088-.052.083-.052.079-.052.074-.051.07-.052.065-.051.06-.05.056-.05.051-.05.023-.025.023-.024.021-.024.02-.025.019-.024.018-.024.017-.023.015-.024.014-.023.013-.023.012-.023.01-.023.01-.022.008-.022.006-.023.006-.021.004-.022.004-.021.001-.021.001-.021-.001-.021-.001-.021-.004-.021-.004-.022-.006-.021-.006-.023-.008-.022-.01-.022-.01-.023-.012-.023-.013-.023-.014-.023-.015-.024-.017-.023-.018-.024-.019-.024-.02-.025-.021-.024-.023-.024-.023-.025-.051-.05-.056-.05-.06-.05-.065-.051-.07-.052-.074-.051-.079-.052-.083-.052-.088-.052-.092-.052-.096-.052-.101-.052-.106-.052-.11-.052-.114-.052-.118-.051-.123-.051-.126-.051-.131-.05-.136-.05-.139-.049-.143-.048-.148-.048-.152-.048-.155-.046-.16-.046-.163-.045-.168-.043-.172-.043-.175-.042-.179-.041-.183-.04-.187-.038-.191-.038-.194-.036-.198-.034-.202-.033-.205-.032-.21-.031-.212-.028-.216-.027-.22-.026-.224-.023-.226-.022-.231-.021-.233-.018-.237-.016-.241-.014-.244-.012-.247-.011-.25-.008-.254-.005-.257-.004-.26-.001-.26.001z"/></symbol></defs><defs><symbol id="clock" width="24" height="24"><path transform="scale(.5)" d="M12 2c5.514 0 10 4.486 10 10s-4.486 10-10 10-10-4.486-10-10 4.486-10 10-10zm0-2c-6.627 0-12 5.373-12 12s5.373 12 12 12 12-5.373 12-12-5.373-12-12-12zm5.848 12.459c.202.038.202.333.001.372-1.907.361-6.045 1.111-6.547 1.111-.719 0-1.301-.582-1.301-1.301 0-.512.77-5.447 1.125-7.445.034-.192.312-.181.343.014l.985 6.238 5.394 1.011z"/></symbol></defs><g><line id="actor0" x1="75" y1="5" x2="75" y2="681" class="200" stroke-width="0.5px" stroke="#999"/><g id="root-0"><rect x="0" y="0" fill="#eaeaea" stroke="#666" width="150" height="65" rx="3" ry="3" class="actor"/><text x="75" y="32.5" dominant-baseline="central" alignment-baseline="central" class="actor" style="text-anchor: middle; font-size: 14px; font-weight: 400; font-family: &quot;Open Sans&quot;, sans-serif;"><tspan x="75" dy="0">Content timeline</tspan></text></g></g><g><line id="actor1" x1="296" y1="5" x2="296" y2="681" class="200" stroke-width="0.5px" stroke="#999"/><g id="root-1"><rect x="221" y="0" fill="#eaeaea" stroke="#666" width="150" height="65" rx="3" ry="3" class="actor"/><text x="296" y="32.5" dominant-baseline="central" alignment-baseline="central" class="actor" style="text-anchor: middle; font-size: 14px; font-weight: 400; font-family: &quot;Open Sans&quot;, sans-serif;"><tspan x="296" dy="0">Device timeline</tspan></text></g></g><defs><marker id="arrowhead" refX="9" refY="5" markerUnits="userSpaceOnUse" markerWidth="12" markerHeight="12" orient="auto"><path d="M 0 0 L 10 5 L 0 10 z"/></marker></defs><defs><marker id="crosshead" markerWidth="15" markerHeight="8" orient="auto" refX="16" refY="4"><path fill="black" stroke="#000000" stroke-width="1px" d="M 9,2 V 6 L16,4 Z" style="stroke-dasharray: 0, 0;"/><path fill="none" stroke="#000000" stroke-width="1px" d="M 0,1 L 6,7 M 6,1 L 0,7" style="stroke-dasharray: 0, 0;"/></marker></defs><defs><marker id="filled-head" refX="18" refY="7" markerWidth="20" markerHeight="28" orient="auto"><path d="M 18,7 L9,13 L14,7 L9,1 Z"/></marker></defs><defs><marker id="sequencenumber" refX="15" refY="15" markerWidth="60" markerHeight="40" orient="auto"><circle cx="15" cy="15" r="6"/></marker></defs><g><rect x="-8" y="75" fill="#EDF2AE" stroke="#666" width="166" height="52" rx="0" ry="0" class="note"/><text x="75" y="80" text-anchor="middle" dominant-baseline="middle" alignment-baseline="middle" class="noteText" dy="1em" style="font-family: &quot;trebuchet ms&quot;, verdana, arial, sans-serif; font-size: 14px; font-weight: 400;"><tspan x="75">[[mapping]] is null</tspan></text><text x="75" y="96" text-anchor="middle" dominant-baseline="middle" alignment-baseline="middle" class="noteText" dy="1em" style="font-family: &quot;trebuchet ms&quot;, verdana, arial, sans-serif; font-size: 14px; font-weight: 400;"><tspan x="75">[[pending_map]] is null</tspan></text></g><g><rect x="177.5" y="137" fill="#EDF2AE" stroke="#666" width="237" height="36" rx="0" ry="0" class="note"/><text x="296" y="142" text-anchor="middle" dominant-baseline="middle" alignment-baseline="middle" class="noteText" dy="1em" style="font-family: &quot;trebuchet ms&quot;, verdana, arial, sans-serif; font-size: 14px; font-weight: 400;"><tspan x="296">[[internals]].[[state]] is "available"</tspan></text></g><g><rect x="-22" y="231" fill="#EDF2AE" stroke="#666" width="194" height="52" rx="0" ry="0" class="note"/><text x="75" y="236" text-anchor="middle" dominant-baseline="middle" alignment-baseline="middle" class="noteText" dy="1em" style="font-family: &quot;trebuchet ms&quot;, verdana, arial, sans-serif; font-size: 14px; font-weight: 400;"><tspan x="75">[[mapping]] is null</tspan></text><text x="75" y="252" text-anchor="middle" dominant-baseline="middle" alignment-baseline="middle" class="noteText" dy="1em" style="font-family: &quot;trebuchet ms&quot;, verdana, arial, sans-serif; font-size: 14px; font-weight: 400;"><tspan x="75">[[pending_map]] is non-null</tspan></text></g><g><rect x="170" y="293" fill="#EDF2AE" stroke="#666" width="252" height="36" rx="0" ry="0" class="note"/><text x="296" y="298" text-anchor="middle" dominant-baseline="middle" alignment-baseline="middle" class="noteText" dy="1em" style="font-family: &quot;trebuchet ms&quot;, verdana, arial, sans-serif; font-size: 14px; font-weight: 400;"><tspan x="296">[[internals]].[[state]] is "unavailable"</tspan></text></g><g><rect x="170" y="387" fill="#EDF2AE" stroke="#666" width="252" height="36" rx="0" ry="0" class="note"/><text x="296" y="392" text-anchor="middle" dominant-baseline="middle" alignment-baseline="middle" class="noteText" dy="1em" style="font-family: &quot;trebuchet ms&quot;, verdana, arial, sans-serif; font-size: 14px; font-weight: 400;"><tspan x="296">[[internals]].[[state]] is "unavailable"</tspan></text></g><g><rect x="-8" y="433" fill="#EDF2AE" stroke="#666" width="166" height="52" rx="0" ry="0" class="note"/><text x="75" y="438" text-anchor="middle" dominant-baseline="middle" alignment-baseline="middle" class="noteText" dy="1em" style="font-family: &quot;trebuchet ms&quot;, verdana, arial, sans-serif; font-size: 14px; font-weight: 400;"><tspan x="75">[[mapping]] is non-null</tspan></text><text x="75" y="454" text-anchor="middle" dominant-baseline="middle" alignment-baseline="middle" class="noteText" dy="1em" style="font-family: &quot;trebuchet ms&quot;, verdana, arial, sans-serif; font-size: 14px; font-weight: 400;"><tspan x="75">[[pending_map]] is null</tspan></text></g><g><rect x="-8" y="543" fill="#EDF2AE" stroke="#666" width="166" height="52" rx="0" ry="0" class="note"/><text x="75" y="548" text-anchor="middle" dominant-baseline="middle" alignment-baseline="middle" class="noteText" dy="1em" style="font-family: &quot;trebuchet ms&quot;, verdana, arial, sans-serif; font-size: 14px; font-weight: 400;"><tspan x="75">[[mapping]] is null</tspan></text><text x="75" y="564" text-anchor="middle" dominant-baseline="middle" alignment-baseline="middle" class="noteText" dy="1em" style="font-family: &quot;trebuchet ms&quot;, verdana, arial, sans-serif; font-size: 14px; font-weight: 400;"><tspan x="75">[[pending_map]] is null</tspan></text></g><g><rect x="177.5" y="605" fill="#EDF2AE" stroke="#666" width="237" height="36" rx="0" ry="0" class="note"/><text x="296" y="610" text-anchor="middle" dominant-baseline="middle" alignment-baseline="middle" class="noteText" dy="1em" style="font-family: &quot;trebuchet ms&quot;, verdana, arial, sans-serif; font-size: 14px; font-weight: 400;"><tspan x="296">[[internals]].[[state]] is "available"</tspan></text></g><text x="186" y="188" text-anchor="middle" dominant-baseline="middle" alignment-baseline="middle" class="messageText" dy="1em" style="font-family: &quot;trebuchet ms&quot;, verdana, arial, sans-serif; font-size: 16px; font-weight: 400;">mapAsync()</text><line x1="75" y1="221" x2="296" y2="221" class="messageLine0" stroke-width="2" stroke="none" marker-end="url(#arrowhead)" style="fill: none;"/><text x="186" y="344" text-anchor="middle" dominant-baseline="middle" alignment-baseline="middle" class="messageText" dy="1em" style="font-family: &quot;trebuchet ms&quot;, verdana, arial, sans-serif; font-size: 16px; font-weight: 400;">mapAsync() response</text><line x1="296" y1="377" x2="75" y2="377" class="messageLine0" stroke-width="2" stroke="none" marker-end="url(#arrowhead)" style="fill: none;"/><text x="186" y="500" text-anchor="middle" dominant-baseline="middle" alignment-baseline="middle" class="messageText" dy="1em" style="font-family: &quot;trebuchet ms&quot;, verdana, arial, sans-serif; font-size: 16px; font-weight: 400;">unmap()</text><line x1="75" y1="533" x2="296" y2="533" class="messageLine0" stroke-width="2" stroke="none" marker-end="url(#arrowhead)" style="fill: none;"/><g><rect x="0" y="661" fill="#eaeaea" stroke="#666" width="150" height="65" rx="3" ry="3" class="actor"/><text x="75" y="693.5" dominant-baseline="central" alignment-baseline="central" class="actor" style="text-anchor: middle; font-size: 14px; font-weight: 400; font-family: &quot;Open Sans&quot;, sans-serif;"><tspan x="75" dy="0">Content timeline</tspan></text></g><g><rect x="221" y="661" fill="#eaeaea" stroke="#666" width="150" height="65" rx="3" ry="3" class="actor"/><text x="296" y="693.5" dominant-baseline="central" alignment-baseline="central" class="actor" style="text-anchor: middle; font-size: 14px; font-weight: 400; font-family: &quot;Open Sans&quot;, sans-serif;"><tspan x="296" dy="0">Device timeline</tspan></text></g></svg>
     </object>
</figure>

<figure>
     <figcaption>失败的映射一个缓冲区 详情图.</figcaption>
     <figcaption>Failing to map a buffer.</figcaption>
     <object data="img/buffer-map-failure.mmd.svg" type="image/svg+xml">
     <svg xmlns="http://www.w3.org/2000/svg" id="mermaid-0" width="100%" height="545" style="max-width: 536.5px; background-color: black;" viewBox="-72 -10 536.5 545" role="img" aria-labelledby="chart-title-mermaid-0 chart-desc-mermaid-0"><title id="chart-title-mermaid-0"/><desc id="chart-desc-mermaid-0"/><style>#mermaid-0 {font-family:"trebuchet ms",verdana,arial,sans-serif;font-size:16px;fill:#ccc;}#mermaid-0 .error-icon{fill:#a44141;}#mermaid-0 .error-text{fill:#ddd;stroke:#ddd;}#mermaid-0 .edge-thickness-normal{stroke-width:2px;}#mermaid-0 .edge-thickness-thick{stroke-width:3.5px;}#mermaid-0 .edge-pattern-solid{stroke-dasharray:0;}#mermaid-0 .edge-pattern-dashed{stroke-dasharray:3;}#mermaid-0 .edge-pattern-dotted{stroke-dasharray:2;}#mermaid-0 .marker{fill:lightgrey;stroke:lightgrey;}#mermaid-0 .marker.cross{stroke:lightgrey;}#mermaid-0 svg{font-family:"trebuchet ms",verdana,arial,sans-serif;font-size:16px;}#mermaid-0 .actor{stroke:#81B1DB;fill:#1f2020;}#mermaid-0 text.actor&gt;tspan{fill:lightgrey;stroke:none;}#mermaid-0 .actor-line{stroke:lightgrey;}#mermaid-0 .messageLine0{stroke-width:1.5;stroke-dasharray:none;stroke:lightgrey;}#mermaid-0 .messageLine1{stroke-width:1.5;stroke-dasharray:2,2;stroke:lightgrey;}#mermaid-0 #arrowhead path{fill:lightgrey;stroke:lightgrey;}#mermaid-0 .sequenceNumber{fill:black;}#mermaid-0 #sequencenumber{fill:lightgrey;}#mermaid-0 #crosshead path{fill:lightgrey;stroke:lightgrey;}#mermaid-0 .messageText{fill:lightgrey;stroke:lightgrey;}#mermaid-0 .labelBox{stroke:#81B1DB;fill:#1f2020;}#mermaid-0 .labelText,#mermaid-0 .labelText&gt;tspan{fill:lightgrey;stroke:none;}#mermaid-0 .loopText,#mermaid-0 .loopText&gt;tspan{fill:lightgrey;stroke:none;}#mermaid-0 .loopLine{stroke-width:2px;stroke-dasharray:2,2;stroke:#81B1DB;fill:#81B1DB;}#mermaid-0 .note{stroke:hsl(180, 0%, 18.3529411765%);fill:hsl(180, 1.5873015873%, 28.3529411765%);}#mermaid-0 .noteText,#mermaid-0 .noteText&gt;tspan{fill:rgb(183.8476190475, 181.5523809523, 181.5523809523);stroke:none;}#mermaid-0 .activation0{fill:hsl(180, 1.5873015873%, 28.3529411765%);stroke:#81B1DB;}#mermaid-0 .activation1{fill:hsl(180, 1.5873015873%, 28.3529411765%);stroke:#81B1DB;}#mermaid-0 .activation2{fill:hsl(180, 1.5873015873%, 28.3529411765%);stroke:#81B1DB;}#mermaid-0 .actorPopupMenu{position:absolute;}#mermaid-0 .actorPopupMenuPanel{position:absolute;fill:#1f2020;box-shadow:0px 8px 16px 0px rgba(0,0,0,0.2);filter:drop-shadow(3px 5px 2px rgb(0 0 0 / 0.4));}#mermaid-0 .actor-man line{stroke:#81B1DB;fill:#1f2020;}#mermaid-0 .actor-man circle,#mermaid-0 line{stroke:#81B1DB;fill:#1f2020;stroke-width:2px;}#mermaid-0 .messageText{stroke:none;}#mermaid-0 :root{--mermaid-font-family:"trebuchet ms",verdana,arial,sans-serif;}</style><g/><defs><symbol id="computer" width="24" height="24"><path transform="scale(.5)" d="M2 2v13h20v-13h-20zm18 11h-16v-9h16v9zm-10.228 6l.466-1h3.524l.467 1h-4.457zm14.228 3h-24l2-6h2.104l-1.33 4h18.45l-1.297-4h2.073l2 6zm-5-10h-14v-7h14v7z"/></symbol></defs><defs><symbol id="database" fill-rule="evenodd" clip-rule="evenodd"><path transform="scale(.5)" d="M12.258.001l.256.004.255.005.253.008.251.01.249.012.247.015.246.016.242.019.241.02.239.023.236.024.233.027.231.028.229.031.225.032.223.034.22.036.217.038.214.04.211.041.208.043.205.045.201.046.198.048.194.05.191.051.187.053.183.054.18.056.175.057.172.059.168.06.163.061.16.063.155.064.15.066.074.033.073.033.071.034.07.034.069.035.068.035.067.035.066.035.064.036.064.036.062.036.06.036.06.037.058.037.058.037.055.038.055.038.053.038.052.038.051.039.05.039.048.039.047.039.045.04.044.04.043.04.041.04.04.041.039.041.037.041.036.041.034.041.033.042.032.042.03.042.029.042.027.042.026.043.024.043.023.043.021.043.02.043.018.044.017.043.015.044.013.044.012.044.011.045.009.044.007.045.006.045.004.045.002.045.001.045v17l-.001.045-.002.045-.004.045-.006.045-.007.045-.009.044-.011.045-.012.044-.013.044-.015.044-.017.043-.018.044-.02.043-.021.043-.023.043-.024.043-.026.043-.027.042-.029.042-.03.042-.032.042-.033.042-.034.041-.036.041-.037.041-.039.041-.04.041-.041.04-.043.04-.044.04-.045.04-.047.039-.048.039-.05.039-.051.039-.052.038-.053.038-.055.038-.055.038-.058.037-.058.037-.06.037-.06.036-.062.036-.064.036-.064.036-.066.035-.067.035-.068.035-.069.035-.07.034-.071.034-.073.033-.074.033-.15.066-.155.064-.16.063-.163.061-.168.06-.172.059-.175.057-.18.056-.183.054-.187.053-.191.051-.194.05-.198.048-.201.046-.205.045-.208.043-.211.041-.214.04-.217.038-.22.036-.223.034-.225.032-.229.031-.231.028-.233.027-.236.024-.239.023-.241.02-.242.019-.246.016-.247.015-.249.012-.251.01-.253.008-.255.005-.256.004-.258.001-.258-.001-.256-.004-.255-.005-.253-.008-.251-.01-.249-.012-.247-.015-.245-.016-.243-.019-.241-.02-.238-.023-.236-.024-.234-.027-.231-.028-.228-.031-.226-.032-.223-.034-.22-.036-.217-.038-.214-.04-.211-.041-.208-.043-.204-.045-.201-.046-.198-.048-.195-.05-.19-.051-.187-.053-.184-.054-.179-.056-.176-.057-.172-.059-.167-.06-.164-.061-.159-.063-.155-.064-.151-.066-.074-.033-.072-.033-.072-.034-.07-.034-.069-.035-.068-.035-.067-.035-.066-.035-.064-.036-.063-.036-.062-.036-.061-.036-.06-.037-.058-.037-.057-.037-.056-.038-.055-.038-.053-.038-.052-.038-.051-.039-.049-.039-.049-.039-.046-.039-.046-.04-.044-.04-.043-.04-.041-.04-.04-.041-.039-.041-.037-.041-.036-.041-.034-.041-.033-.042-.032-.042-.03-.042-.029-.042-.027-.042-.026-.043-.024-.043-.023-.043-.021-.043-.02-.043-.018-.044-.017-.043-.015-.044-.013-.044-.012-.044-.011-.045-.009-.044-.007-.045-.006-.045-.004-.045-.002-.045-.001-.045v-17l.001-.045.002-.045.004-.045.006-.045.007-.045.009-.044.011-.045.012-.044.013-.044.015-.044.017-.043.018-.044.02-.043.021-.043.023-.043.024-.043.026-.043.027-.042.029-.042.03-.042.032-.042.033-.042.034-.041.036-.041.037-.041.039-.041.04-.041.041-.04.043-.04.044-.04.046-.04.046-.039.049-.039.049-.039.051-.039.052-.038.053-.038.055-.038.056-.038.057-.037.058-.037.06-.037.061-.036.062-.036.063-.036.064-.036.066-.035.067-.035.068-.035.069-.035.07-.034.072-.034.072-.033.074-.033.151-.066.155-.064.159-.063.164-.061.167-.06.172-.059.176-.057.179-.056.184-.054.187-.053.19-.051.195-.05.198-.048.201-.046.204-.045.208-.043.211-.041.214-.04.217-.038.22-.036.223-.034.226-.032.228-.031.231-.028.234-.027.236-.024.238-.023.241-.02.243-.019.245-.016.247-.015.249-.012.251-.01.253-.008.255-.005.256-.004.258-.001.258.001zm-9.258 20.499v.01l.001.021.003.021.004.022.005.021.006.022.007.022.009.023.01.022.011.023.012.023.013.023.015.023.016.024.017.023.018.024.019.024.021.024.022.025.023.024.024.025.052.049.056.05.061.051.066.051.07.051.075.051.079.052.084.052.088.052.092.052.097.052.102.051.105.052.11.052.114.051.119.051.123.051.127.05.131.05.135.05.139.048.144.049.147.047.152.047.155.047.16.045.163.045.167.043.171.043.176.041.178.041.183.039.187.039.19.037.194.035.197.035.202.033.204.031.209.03.212.029.216.027.219.025.222.024.226.021.23.02.233.018.236.016.24.015.243.012.246.01.249.008.253.005.256.004.259.001.26-.001.257-.004.254-.005.25-.008.247-.011.244-.012.241-.014.237-.016.233-.018.231-.021.226-.021.224-.024.22-.026.216-.027.212-.028.21-.031.205-.031.202-.034.198-.034.194-.036.191-.037.187-.039.183-.04.179-.04.175-.042.172-.043.168-.044.163-.045.16-.046.155-.046.152-.047.148-.048.143-.049.139-.049.136-.05.131-.05.126-.05.123-.051.118-.052.114-.051.11-.052.106-.052.101-.052.096-.052.092-.052.088-.053.083-.051.079-.052.074-.052.07-.051.065-.051.06-.051.056-.05.051-.05.023-.024.023-.025.021-.024.02-.024.019-.024.018-.024.017-.024.015-.023.014-.024.013-.023.012-.023.01-.023.01-.022.008-.022.006-.022.006-.022.004-.022.004-.021.001-.021.001-.021v-4.127l-.077.055-.08.053-.083.054-.085.053-.087.052-.09.052-.093.051-.095.05-.097.05-.1.049-.102.049-.105.048-.106.047-.109.047-.111.046-.114.045-.115.045-.118.044-.12.043-.122.042-.124.042-.126.041-.128.04-.13.04-.132.038-.134.038-.135.037-.138.037-.139.035-.142.035-.143.034-.144.033-.147.032-.148.031-.15.03-.151.03-.153.029-.154.027-.156.027-.158.026-.159.025-.161.024-.162.023-.163.022-.165.021-.166.02-.167.019-.169.018-.169.017-.171.016-.173.015-.173.014-.175.013-.175.012-.177.011-.178.01-.179.008-.179.008-.181.006-.182.005-.182.004-.184.003-.184.002h-.37l-.184-.002-.184-.003-.182-.004-.182-.005-.181-.006-.179-.008-.179-.008-.178-.01-.176-.011-.176-.012-.175-.013-.173-.014-.172-.015-.171-.016-.17-.017-.169-.018-.167-.019-.166-.02-.165-.021-.163-.022-.162-.023-.161-.024-.159-.025-.157-.026-.156-.027-.155-.027-.153-.029-.151-.03-.15-.03-.148-.031-.146-.032-.145-.033-.143-.034-.141-.035-.14-.035-.137-.037-.136-.037-.134-.038-.132-.038-.13-.04-.128-.04-.126-.041-.124-.042-.122-.042-.12-.044-.117-.043-.116-.045-.113-.045-.112-.046-.109-.047-.106-.047-.105-.048-.102-.049-.1-.049-.097-.05-.095-.05-.093-.052-.09-.051-.087-.052-.085-.053-.083-.054-.08-.054-.077-.054v4.127zm0-5.654v.011l.001.021.003.021.004.021.005.022.006.022.007.022.009.022.01.022.011.023.012.023.013.023.015.024.016.023.017.024.018.024.019.024.021.024.022.024.023.025.024.024.052.05.056.05.061.05.066.051.07.051.075.052.079.051.084.052.088.052.092.052.097.052.102.052.105.052.11.051.114.051.119.052.123.05.127.051.131.05.135.049.139.049.144.048.147.048.152.047.155.046.16.045.163.045.167.044.171.042.176.042.178.04.183.04.187.038.19.037.194.036.197.034.202.033.204.032.209.03.212.028.216.027.219.025.222.024.226.022.23.02.233.018.236.016.24.014.243.012.246.01.249.008.253.006.256.003.259.001.26-.001.257-.003.254-.006.25-.008.247-.01.244-.012.241-.015.237-.016.233-.018.231-.02.226-.022.224-.024.22-.025.216-.027.212-.029.21-.03.205-.032.202-.033.198-.035.194-.036.191-.037.187-.039.183-.039.179-.041.175-.042.172-.043.168-.044.163-.045.16-.045.155-.047.152-.047.148-.048.143-.048.139-.05.136-.049.131-.05.126-.051.123-.051.118-.051.114-.052.11-.052.106-.052.101-.052.096-.052.092-.052.088-.052.083-.052.079-.052.074-.051.07-.052.065-.051.06-.05.056-.051.051-.049.023-.025.023-.024.021-.025.02-.024.019-.024.018-.024.017-.024.015-.023.014-.023.013-.024.012-.022.01-.023.01-.023.008-.022.006-.022.006-.022.004-.021.004-.022.001-.021.001-.021v-4.139l-.077.054-.08.054-.083.054-.085.052-.087.053-.09.051-.093.051-.095.051-.097.05-.1.049-.102.049-.105.048-.106.047-.109.047-.111.046-.114.045-.115.044-.118.044-.12.044-.122.042-.124.042-.126.041-.128.04-.13.039-.132.039-.134.038-.135.037-.138.036-.139.036-.142.035-.143.033-.144.033-.147.033-.148.031-.15.03-.151.03-.153.028-.154.028-.156.027-.158.026-.159.025-.161.024-.162.023-.163.022-.165.021-.166.02-.167.019-.169.018-.169.017-.171.016-.173.015-.173.014-.175.013-.175.012-.177.011-.178.009-.179.009-.179.007-.181.007-.182.005-.182.004-.184.003-.184.002h-.37l-.184-.002-.184-.003-.182-.004-.182-.005-.181-.007-.179-.007-.179-.009-.178-.009-.176-.011-.176-.012-.175-.013-.173-.014-.172-.015-.171-.016-.17-.017-.169-.018-.167-.019-.166-.02-.165-.021-.163-.022-.162-.023-.161-.024-.159-.025-.157-.026-.156-.027-.155-.028-.153-.028-.151-.03-.15-.03-.148-.031-.146-.033-.145-.033-.143-.033-.141-.035-.14-.036-.137-.036-.136-.037-.134-.038-.132-.039-.13-.039-.128-.04-.126-.041-.124-.042-.122-.043-.12-.043-.117-.044-.116-.044-.113-.046-.112-.046-.109-.046-.106-.047-.105-.048-.102-.049-.1-.049-.097-.05-.095-.051-.093-.051-.09-.051-.087-.053-.085-.052-.083-.054-.08-.054-.077-.054v4.139zm0-5.666v.011l.001.02.003.022.004.021.005.022.006.021.007.022.009.023.01.022.011.023.012.023.013.023.015.023.016.024.017.024.018.023.019.024.021.025.022.024.023.024.024.025.052.05.056.05.061.05.066.051.07.051.075.052.079.051.084.052.088.052.092.052.097.052.102.052.105.051.11.052.114.051.119.051.123.051.127.05.131.05.135.05.139.049.144.048.147.048.152.047.155.046.16.045.163.045.167.043.171.043.176.042.178.04.183.04.187.038.19.037.194.036.197.034.202.033.204.032.209.03.212.028.216.027.219.025.222.024.226.021.23.02.233.018.236.017.24.014.243.012.246.01.249.008.253.006.256.003.259.001.26-.001.257-.003.254-.006.25-.008.247-.01.244-.013.241-.014.237-.016.233-.018.231-.02.226-.022.224-.024.22-.025.216-.027.212-.029.21-.03.205-.032.202-.033.198-.035.194-.036.191-.037.187-.039.183-.039.179-.041.175-.042.172-.043.168-.044.163-.045.16-.045.155-.047.152-.047.148-.048.143-.049.139-.049.136-.049.131-.051.126-.05.123-.051.118-.052.114-.051.11-.052.106-.052.101-.052.096-.052.092-.052.088-.052.083-.052.079-.052.074-.052.07-.051.065-.051.06-.051.056-.05.051-.049.023-.025.023-.025.021-.024.02-.024.019-.024.018-.024.017-.024.015-.023.014-.024.013-.023.012-.023.01-.022.01-.023.008-.022.006-.022.006-.022.004-.022.004-.021.001-.021.001-.021v-4.153l-.077.054-.08.054-.083.053-.085.053-.087.053-.09.051-.093.051-.095.051-.097.05-.1.049-.102.048-.105.048-.106.048-.109.046-.111.046-.114.046-.115.044-.118.044-.12.043-.122.043-.124.042-.126.041-.128.04-.13.039-.132.039-.134.038-.135.037-.138.036-.139.036-.142.034-.143.034-.144.033-.147.032-.148.032-.15.03-.151.03-.153.028-.154.028-.156.027-.158.026-.159.024-.161.024-.162.023-.163.023-.165.021-.166.02-.167.019-.169.018-.169.017-.171.016-.173.015-.173.014-.175.013-.175.012-.177.01-.178.01-.179.009-.179.007-.181.006-.182.006-.182.004-.184.003-.184.001-.185.001-.185-.001-.184-.001-.184-.003-.182-.004-.182-.006-.181-.006-.179-.007-.179-.009-.178-.01-.176-.01-.176-.012-.175-.013-.173-.014-.172-.015-.171-.016-.17-.017-.169-.018-.167-.019-.166-.02-.165-.021-.163-.023-.162-.023-.161-.024-.159-.024-.157-.026-.156-.027-.155-.028-.153-.028-.151-.03-.15-.03-.148-.032-.146-.032-.145-.033-.143-.034-.141-.034-.14-.036-.137-.036-.136-.037-.134-.038-.132-.039-.13-.039-.128-.041-.126-.041-.124-.041-.122-.043-.12-.043-.117-.044-.116-.044-.113-.046-.112-.046-.109-.046-.106-.048-.105-.048-.102-.048-.1-.05-.097-.049-.095-.051-.093-.051-.09-.052-.087-.052-.085-.053-.083-.053-.08-.054-.077-.054v4.153zm8.74-8.179l-.257.004-.254.005-.25.008-.247.011-.244.012-.241.014-.237.016-.233.018-.231.021-.226.022-.224.023-.22.026-.216.027-.212.028-.21.031-.205.032-.202.033-.198.034-.194.036-.191.038-.187.038-.183.04-.179.041-.175.042-.172.043-.168.043-.163.045-.16.046-.155.046-.152.048-.148.048-.143.048-.139.049-.136.05-.131.05-.126.051-.123.051-.118.051-.114.052-.11.052-.106.052-.101.052-.096.052-.092.052-.088.052-.083.052-.079.052-.074.051-.07.052-.065.051-.06.05-.056.05-.051.05-.023.025-.023.024-.021.024-.02.025-.019.024-.018.024-.017.023-.015.024-.014.023-.013.023-.012.023-.01.023-.01.022-.008.022-.006.023-.006.021-.004.022-.004.021-.001.021-.001.021.001.021.001.021.004.021.004.022.006.021.006.023.008.022.01.022.01.023.012.023.013.023.014.023.015.024.017.023.018.024.019.024.02.025.021.024.023.024.023.025.051.05.056.05.06.05.065.051.07.052.074.051.079.052.083.052.088.052.092.052.096.052.101.052.106.052.11.052.114.052.118.051.123.051.126.051.131.05.136.05.139.049.143.048.148.048.152.048.155.046.16.046.163.045.168.043.172.043.175.042.179.041.183.04.187.038.191.038.194.036.198.034.202.033.205.032.21.031.212.028.216.027.22.026.224.023.226.022.231.021.233.018.237.016.241.014.244.012.247.011.25.008.254.005.257.004.26.001.26-.001.257-.004.254-.005.25-.008.247-.011.244-.012.241-.014.237-.016.233-.018.231-.021.226-.022.224-.023.22-.026.216-.027.212-.028.21-.031.205-.032.202-.033.198-.034.194-.036.191-.038.187-.038.183-.04.179-.041.175-.042.172-.043.168-.043.163-.045.16-.046.155-.046.152-.048.148-.048.143-.048.139-.049.136-.05.131-.05.126-.051.123-.051.118-.051.114-.052.11-.052.106-.052.101-.052.096-.052.092-.052.088-.052.083-.052.079-.052.074-.051.07-.052.065-.051.06-.05.056-.05.051-.05.023-.025.023-.024.021-.024.02-.025.019-.024.018-.024.017-.023.015-.024.014-.023.013-.023.012-.023.01-.023.01-.022.008-.022.006-.023.006-.021.004-.022.004-.021.001-.021.001-.021-.001-.021-.001-.021-.004-.021-.004-.022-.006-.021-.006-.023-.008-.022-.01-.022-.01-.023-.012-.023-.013-.023-.014-.023-.015-.024-.017-.023-.018-.024-.019-.024-.02-.025-.021-.024-.023-.024-.023-.025-.051-.05-.056-.05-.06-.05-.065-.051-.07-.052-.074-.051-.079-.052-.083-.052-.088-.052-.092-.052-.096-.052-.101-.052-.106-.052-.11-.052-.114-.052-.118-.051-.123-.051-.126-.051-.131-.05-.136-.05-.139-.049-.143-.048-.148-.048-.152-.048-.155-.046-.16-.046-.163-.045-.168-.043-.172-.043-.175-.042-.179-.041-.183-.04-.187-.038-.191-.038-.194-.036-.198-.034-.202-.033-.205-.032-.21-.031-.212-.028-.216-.027-.22-.026-.224-.023-.226-.022-.231-.021-.233-.018-.237-.016-.241-.014-.244-.012-.247-.011-.25-.008-.254-.005-.257-.004-.26-.001-.26.001z"/></symbol></defs><defs><symbol id="clock" width="24" height="24"><path transform="scale(.5)" d="M12 2c5.514 0 10 4.486 10 10s-4.486 10-10 10-10-4.486-10-10 4.486-10 10-10zm0-2c-6.627 0-12 5.373-12 12s5.373 12 12 12 12-5.373 12-12-5.373-12-12-12zm5.848 12.459c.202.038.202.333.001.372-1.907.361-6.045 1.111-6.547 1.111-.719 0-1.301-.582-1.301-1.301 0-.512.77-5.447 1.125-7.445.034-.192.312-.181.343.014l.985 6.238 5.394 1.011z"/></symbol></defs><g><line id="actor0" x1="75" y1="5" x2="75" y2="479" class="200" stroke-width="0.5px" stroke="#999"/><g id="root-0"><rect x="0" y="0" fill="#eaeaea" stroke="#666" width="150" height="65" rx="3" ry="3" class="actor"/><text x="75" y="32.5" dominant-baseline="central" alignment-baseline="central" class="actor" style="text-anchor: middle; font-size: 14px; font-weight: 400; font-family: &quot;Open Sans&quot;, sans-serif;"><tspan x="75" dy="0">Content timeline</tspan></text></g></g><g><line id="actor1" x1="296" y1="5" x2="296" y2="479" class="200" stroke-width="0.5px" stroke="#999"/><g id="root-1"><rect x="221" y="0" fill="#eaeaea" stroke="#666" width="150" height="65" rx="3" ry="3" class="actor"/><text x="296" y="32.5" dominant-baseline="central" alignment-baseline="central" class="actor" style="text-anchor: middle; font-size: 14px; font-weight: 400; font-family: &quot;Open Sans&quot;, sans-serif;"><tspan x="296" dy="0">Device timeline</tspan></text></g></g><defs><marker id="arrowhead" refX="9" refY="5" markerUnits="userSpaceOnUse" markerWidth="12" markerHeight="12" orient="auto"><path d="M 0 0 L 10 5 L 0 10 z"/></marker></defs><defs><marker id="crosshead" markerWidth="15" markerHeight="8" orient="auto" refX="16" refY="4"><path fill="black" stroke="#000000" stroke-width="1px" d="M 9,2 V 6 L16,4 Z" style="stroke-dasharray: 0, 0;"/><path fill="none" stroke="#000000" stroke-width="1px" d="M 0,1 L 6,7 M 6,1 L 0,7" style="stroke-dasharray: 0, 0;"/></marker></defs><defs><marker id="filled-head" refX="18" refY="7" markerWidth="20" markerHeight="28" orient="auto"><path d="M 18,7 L9,13 L14,7 L9,1 Z"/></marker></defs><defs><marker id="sequencenumber" refX="15" refY="15" markerWidth="60" markerHeight="40" orient="auto"><circle cx="15" cy="15" r="6"/></marker></defs><g><rect x="-8" y="75" fill="#EDF2AE" stroke="#666" width="166" height="52" rx="0" ry="0" class="note"/><text x="75" y="80" text-anchor="middle" dominant-baseline="middle" alignment-baseline="middle" class="noteText" dy="1em" style="font-family: &quot;trebuchet ms&quot;, verdana, arial, sans-serif; font-size: 14px; font-weight: 400;"><tspan x="75">[[mapping]] is null</tspan></text><text x="75" y="96" text-anchor="middle" dominant-baseline="middle" alignment-baseline="middle" class="noteText" dy="1em" style="font-family: &quot;trebuchet ms&quot;, verdana, arial, sans-serif; font-size: 14px; font-weight: 400;"><tspan x="75">[[pending_map]] is null</tspan></text></g><g><rect x="177.5" y="137" fill="#EDF2AE" stroke="#666" width="237" height="36" rx="0" ry="0" class="note"/><text x="296" y="142" text-anchor="middle" dominant-baseline="middle" alignment-baseline="middle" class="noteText" dy="1em" style="font-family: &quot;trebuchet ms&quot;, verdana, arial, sans-serif; font-size: 14px; font-weight: 400;"><tspan x="296">[[internals]].[[state]] is "available"</tspan></text></g><g><rect x="-22" y="231" fill="#EDF2AE" stroke="#666" width="194" height="52" rx="0" ry="0" class="note"/><text x="75" y="236" text-anchor="middle" dominant-baseline="middle" alignment-baseline="middle" class="noteText" dy="1em" style="font-family: &quot;trebuchet ms&quot;, verdana, arial, sans-serif; font-size: 14px; font-weight: 400;"><tspan x="75">[[mapping]] is null</tspan></text><text x="75" y="252" text-anchor="middle" dominant-baseline="middle" alignment-baseline="middle" class="noteText" dy="1em" style="font-family: &quot;trebuchet ms&quot;, verdana, arial, sans-serif; font-size: 14px; font-weight: 400;"><tspan x="75">[[pending_map]] is non-null</tspan></text></g><g><rect x="204" y="293" fill="#EDF2AE" stroke="#666" width="184" height="36" rx="0" ry="0" class="note"/><text x="296" y="298" text-anchor="middle" dominant-baseline="middle" alignment-baseline="middle" class="noteText" dy="1em" style="font-family: &quot;trebuchet ms&quot;, verdana, arial, sans-serif; font-size: 14px; font-weight: 400;"><tspan x="296">(failure, state unchanged)</tspan></text></g><g><rect x="-8" y="387" fill="#EDF2AE" stroke="#666" width="166" height="52" rx="0" ry="0" class="note"/><text x="75" y="392" text-anchor="middle" dominant-baseline="middle" alignment-baseline="middle" class="noteText" dy="1em" style="font-family: &quot;trebuchet ms&quot;, verdana, arial, sans-serif; font-size: 14px; font-weight: 400;"><tspan x="75">[[mapping]] is null</tspan></text><text x="75" y="408" text-anchor="middle" dominant-baseline="middle" alignment-baseline="middle" class="noteText" dy="1em" style="font-family: &quot;trebuchet ms&quot;, verdana, arial, sans-serif; font-size: 14px; font-weight: 400;"><tspan x="75">[[pending_map]] is null</tspan></text></g><text x="186" y="188" text-anchor="middle" dominant-baseline="middle" alignment-baseline="middle" class="messageText" dy="1em" style="font-family: &quot;trebuchet ms&quot;, verdana, arial, sans-serif; font-size: 16px; font-weight: 400;">mapAsync()</text><line x1="75" y1="221" x2="296" y2="221" class="messageLine0" stroke-width="2" stroke="none" marker-end="url(#arrowhead)" style="fill: none;"/><text x="186" y="344" text-anchor="middle" dominant-baseline="middle" alignment-baseline="middle" class="messageText" dy="1em" style="font-family: &quot;trebuchet ms&quot;, verdana, arial, sans-serif; font-size: 16px; font-weight: 400;">mapAsync() response</text><line x1="296" y1="377" x2="75" y2="377" class="messageLine0" stroke-width="2" stroke="none" marker-end="url(#arrowhead)" style="fill: none;"/><g><rect x="0" y="459" fill="#eaeaea" stroke="#666" width="150" height="65" rx="3" ry="3" class="actor"/><text x="75" y="491.5" dominant-baseline="central" alignment-baseline="central" class="actor" style="text-anchor: middle; font-size: 14px; font-weight: 400; font-family: &quot;Open Sans&quot;, sans-serif;"><tspan x="75" dy="0">Content timeline</tspan></text></g><g><rect x="221" y="459" fill="#eaeaea" stroke="#666" width="150" height="65" rx="3" ry="3" class="actor"/><text x="296" y="491.5" dominant-baseline="central" alignment-baseline="central" class="actor" style="text-anchor: middle; font-size: 14px; font-weight: 400; font-family: &quot;Open Sans&quot;, sans-serif;"><tspan x="296" dy="0">Device timeline</tspan></text></g>
     </svg>
  </object>
</figure>


## 5.2. 创建Buffer (Buffer Creation)    
### 5.2.1. GPUBufferDescriptor    

> 5.2. Buffer Creation    
> 5.2.1. GPUBufferDescriptor    

```
dictionary GPUBufferDescriptor : GPUObjectDescriptorBase {
    required GPUSize64 size;
    required GPUBufferUsageFlags usage;
    boolean mappedAtCreation = false;
};
```

GPUBufferDescriptor 有以下成员：
* size，类型为： GPUSize64    
缓冲区的大小，单位是字节。
* usage，类型为：GPUBufferUsageFlags    
缓冲区允许的使用的用法。    
* mappedAtCreation，类型为：boolean，默认值为：false    
如果为true，在已经映射的状态下创建缓冲区，允许立即调用getMappedRange()。即使用法不包含 MAP_READ 或 MAP_WRITE，将mappedAtCreation 设置为 true 也是有效的。这可以用来设置缓冲区的初始化数据。     
这保证了即使缓冲区的创建最终失败了，但它仍然会显得在映射范围是可 写/读 的，直到它被取消映射。

> GPUBufferDescriptor has the following members:    
> **size, of type GPUSize64**    
> The size of the buffer in bytes.    
> **usage, of type GPUBufferUsageFlags**    
> The allowed usages for the buffer.    
> **mappedAtCreation, of type boolean, defaulting to false**    
> If true creates the buffer in an already mapped state, allowing getMappedRange() to be called immediately. It is valid to set mappedAtCreation to true even if usage does not contain MAP_READ or MAP_WRITE. This can be used to set the buffer’s initial data.    
>    
> Guarantees that even if the buffer creation eventually fails, it will still appear as if the mapped range can be written/read to until it is unmapped.    

**验证 GPUBufferDescriptor(device, descriptor)**
1. 如果设备丢失，则返回false。
2. 如果描述符的任何使用方式的位设置了，但且不在设备可允许的缓冲区使用用法中，返回false。(译注：就是不同设备的缓冲区的用法是有固定几种的，而描述符是通过一个变量的某一位设置为1 或 0 来 标记要用那些方法，如果标记要用的方法多于可允许的方法就返回 false, 具体有那些用法请参见下文。）
3. 如果描述符的使用属性的 MAP_READ 和 MAP_WRITE 都被设置了，返回false。
4. 如果 descriptor.size 大于 device.[[device]].[[limit]].maxBufferSize ，返回false。
5. 返回true。

> **validating GPUBufferDescriptor(device, descriptor)**
> 1. If device is lost return false.
> 2. If any of the bits of descriptor’s usage aren’t present in device’s allowed buffer usages, return false.
> 3. If both the MAP_READ and MAP_WRITE bits of descriptor’s usage attribute are set, return false.
> 4. If descriptor.size is greater than device.[[device]].[[limits]].maxBufferSize, return false.
> 5. Return true.

### 5.2.2. 缓冲区用法(Buffer Usage)

> 5.2.2. Buffer Usage

```
typedef [EnforceRange] unsigned long GPUBufferUsageFlags;
[Exposed=(Window, DedicatedWorker)]
namespace GPUBufferUsage {
    const GPUFlagsConstant MAP_READ      = 0x0001;
    const GPUFlagsConstant MAP_WRITE     = 0x0002;
    const GPUFlagsConstant COPY_SRC      = 0x0004;
    const GPUFlagsConstant COPY_DST      = 0x0008;
    const GPUFlagsConstant INDEX         = 0x0010;
    const GPUFlagsConstant VERTEX        = 0x0020;
    const GPUFlagsConstant UNIFORM       = 0x0040;
    const GPUFlagsConstant STORAGE       = 0x0080;
    const GPUFlagsConstant INDIRECT      = 0x0100;
    const GPUFlagsConstant QUERY_RESOLVE = 0x0200;
};
```

GPUBufferUsage 标志决定了一个 GPUBuffer 在创建后如何使用：    
**MAP_READ**     
缓冲区可以被映射用于读取。(例如：用 GPUMapMode.READ 调用 mapAsync()）。    
只能与 COPY_DST 结合使用。
**MAP_WRITE**
缓冲区可以被映射用于写入。(例如：用 GPUMapMode.WRITE 调用mapAsync()）。    
只能与 COPY_SRC 结合使用。

> The GPUBufferUsage flags determine how a GPUBuffer may be used after its creation:    
> **MAP_READ**    
> The buffer can be mapped for reading. (Example: calling mapAsync() with GPUMapMode.READ)    
> May only be combined with COPY_DST.    
> **MAP_WRITE**    
> The buffer can be mapped for writing. (Example: calling mapAsync() with GPUMapMode.WRITE)    
> May only be combined with COPY_SRC.    

**COPY_SRC**    
缓冲区可以作为一个复制操作的源。(例如：作为 copyBufferToBuffer() 或 copyBufferToTexture() 调用的源参数）。    
**COPY_DST**    
缓冲区可以作为复制或写操作的目标。(例如：作为 copyBufferToBuffer() 或 copyTextureToBuffer() 调用的目标参数，或者作为 writeBuffer() 调用的目标）。

> **COPY_SRC**    
> The buffer can be used as the source of a copy operation. (Examples: as the source argument of a copyBufferToBuffer() or copyBufferToTexture() call.)    
> **COPY_DST**    
> The buffer can be used as the destination of a copy or write operation. (Examples: as the destination argument of a copyBufferToBuffer() or copyTextureToBuffer() call, or as the target of a writeBuffer() call.)    

**INDEX**    
缓冲区可以作为索引缓冲区使用。(例子：传递给 setIndexBuffer()。)    
**VERTEX**    
该缓冲区可以用作顶点缓冲区。(例如：传递给 setVertexBuffer()。）   
**UNIFORM**    
该缓冲区可以作为一个uniform缓冲区使用。(例如：将一个类型为"uniform"的缓冲区，作为 GPUBufferBindingLayout 的绑定组的入口点）。    
**STORAGE**    
缓冲区可以作为一个存储缓冲区使用。(例如：将一个类型为"storage"或 "read-only-storage"的缓冲区， 作为 GPUBufferBindingLayout 的绑定组的入口点）。    

> **INDEX**    
> The buffer can be used as an index buffer. (Example: passed to setIndexBuffer().)    
> **VERTEX**    
> The buffer can be used as a vertex buffer. (Example: passed to setVertexBuffer().)    
> **UNIFORM**    
> The buffer can be used as a uniform buffer. (Example: as a bind group entry for a GPUBufferBindingLayout with a buffer.type of "uniform".)    
> **STORAGE**    
> The buffer can be used as a storage buffer. (Example: as a bind group entry for a GPUBufferBindingLayout with a buffer.type of "storage" or "read-only-storage".)    

**INDIRECT**    
缓冲区可用于存储 间接命令参数。(例如：作为drawIndirect()或dispatchWorkgroupsIndirect()调用时的  indirectBuffer 参数）。    
**QUERY_RESOLVE**    
该缓冲区可用于捕获查询结果。(例如：作为 resolveQuerySet() 调用的目标参数）。    

> **INDIRECT**    
> The buffer can be used as to store indirect command arguments. (Examples: as the indirectBuffer argument of a drawIndirect() or dispatchWorkgroupsIndirect() call.)    
> **QUERY_RESOLVE**    
> The buffer can be used to capture query results. (Example: as the destination argument of a resolveQuerySet() call.)    

**createBuffer(descriptor)**  
创建一个GPUBuffer。    
调用于： GPUDevice 的实例上。   
参数，如下：    
| Parameter | Type | Nullable | Optional | Description |
| -------- | -------- | -------- | -------- | -------- |
| descriptor | GPUBufferDescriptor | ✘ | ✘ | 	创建 GPUBuffer 的 Description |

> **createBuffer(descriptor)**    
> Creates a GPUBuffer.    
> Called on: GPUDevice this.    
> Arguments:

| Parameter | Type | Nullable | Optional | Description |
| -------- | -------- | -------- | -------- | -------- |
| descriptor | GPUBufferDescriptor | ✘ | ✘ | Description of the GPUBuffer to create. |

返回值为：GPUBuffer
**上下文时间线步骤：**
1. 让 [b, bi] 为!创建的一个新的WebGPU对象(this, GPUBuffer, descriptor)。
2. 将 b.size 设置为 descriptor.size。
3. 将 b.usage 设置为 descriptor.usage。
4. 如果 descriptor.mappedAtCreation 为真：
    1. 设置 b.[[mapping]] 为一个活动的缓冲区，其初始化映射模式为 WRITE，范围为[0, descriptor.size]。    
5. 在它的设备时间线上发起初始化步骤。
6. 返回b.


> Returns: GPUBuffer
> **Content timeline steps:**
> 1. Let [b, bi] be !create a new WebGPU object(this, GPUBuffer, descriptor).
> 2. Set b.size to descriptor.size.
> 3. Set b.usage to descriptor.usage.
> 4. If descriptor.mappedAtCreation is true:
>     1. Set b.[[mapping]] to ? initialize an active buffer mapping with mode WRITE and range [0, descriptor.size].
> 5. Issue the initialization steps on the Device timeline of this.
> 6. Return b.

**设备时间线上初始化步骤。**    
1. 如果以下任何条件不满足，就产生一个验证错误，使bi无效，并停止后续步骤：
    * this是有效的，即GPUBuffer已经存在？？
    * 验证 GPUBufferDescriptor(this, descriptor) 返回值为 true。
    * descriptor.usage 不得为 0。
    * descriptor.usage 是 this 允许的缓冲区使用方法的一个子集。
    * 如果 descriptor.usage 包含 MAP_READ：
        * descriptor.usage 除了COPY_DST之外，不能包括其他任何标志。
    * 如果 descriptor.usage 包含 MAP_WRITE。
        * descriptor.usage 除了COPY_SRC之外，不能包含任何其他标志。
    * 如果 descriptor.mappedAtCreation 为真。
        * descriptor.size 要是4的倍数。

> **Device timeline initialization steps:**    
> 1. If any of the following conditions are unsatisfied generate a validation error, make bi invalid, and stop.
>    * this is valid.
>    * validating GPUBufferDescriptor(this, descriptor) returns true.
>    * descriptor.usage must not be 0.
>    * descriptor.usage is a subset of this’s allowed buffer usages.
>    * If descriptor.usage contains MAP_READ:
>        * descriptor.usage contains no other flags except COPY_DST.
>    * If descriptor.usage contains MAP_WRITE:
>        * descriptor.usage contains no other flags except COPY_SRC.
>    * If descriptor.mappedAtCreation is true:
>        * descriptor.size is a multiple of 4.

**注意：** 如果缓冲区创建失败，并且 descriptor.mappedAtCreation 为 false，任何对 mapAsync() 的调用promise都会返回 rejected 状态，所以任何为实现映射而分配的资源都可以，也都可能被丢弃或被回收。

1. 如果 descriptor.mappedAtCreation 为 true。
    1. 将 bi.state 设置为 "unavailable"。
    否则。  
    1. 将 bi.state 设为 "available"。
2. 为 bi 创建一个设备内存分配，其中每个字节都初始化为 0。
    1. 如果分配失败，没有任何副作用，产生一个内存不足的错误，使 bi 无效，并返回。

> **NOTE:** If buffer creation fails, and descriptor.mappedAtCreation is false, any calls to mapAsync() will reject, so any resources allocated to enable mapping can and may be discarded or recycled.
> 1. If descriptor.mappedAtCreation is true:
>    1. Set bi.state to "unavailable".       
>     Else:    
>    1. Set bi.state to "available".    
> 2. Create a device allocation for bi where each byte is zero.    
>    If the allocation fails without side-effects, generate an out-of-memory error, make bi invalid, and return.

```
// 创建一个 128 字节长的，且用于写入功能 的 uniform 缓冲器。
Creating a 128 byte uniform buffer that can be written into:

const buffer = gpuDevice.createBuffer({
    size: 128,
    usage: GPUBufferUsage.UNIFORM | GPUBufferUsage.COPY_DST
});
```

## 5.3. Buffer 析构(Buffer Destruction)

> 5.3. Buffer Destruction

一个不再需要 GPUBuffer 的应用程序可以选择通过调用 destroy()函数，这样在垃圾回收前，就会失去对它的访问。销毁一个缓冲区的同时也会解除它的映射，释放为映射分配的所有内存。

**注意：** 这允许用户代理在所有先前提交使用GPUBuffer操作完成之后，可以回收与 GPUBuffer 相关的GPU内存。

> An application that no longer requires a GPUBuffer can choose to lose access to it before garbage collection by calling destroy(). Destroying a buffer also unmaps it, freeing any memory allocated for the mapping.

> **NOTE:** This allows the user agent to reclaim the GPU memory associated with the GPUBuffer once all previously submitted operations using it are complete.

**destroy() 函数**    
销毁GPUBuffer。    
**注意：** 多次销毁一个缓冲区是有效的，不会产生错误。    
被调用的对象。GPUBuffer 实例对象。        
返回值： undefined    

**上下文时间线上步骤:**   
1. 调用this.unmap()。    
2. 在 this.[[device]] 的设备时间线上发布后续步骤：

**设备时间线上步骤**    
    将this.[[internals]].state设置为 "destroyed"。

**注意：** 由于不能再使用这个缓冲区进行进一步的入队操作，实现可以释放所有分配资源，包括刚刚解除映射的映射内存。

> **destroy()**    
> * Destroys the GPUBuffer.    
> **NOTE:** It is valid to destroy a buffer multiple times.    
>     
> Called on: GPUBuffer this.    
> Returns: undefined    
>> Content timeline steps:    
>> 1. Call this.unmap().    
>> 2. Issue the subsequent steps on the Device timeline of this.[[device]].    

>> **Device timeline steps:**    
>> Set this.[[internals]].state to "destroyed".    
>>     
>> **NOTE:** Since no further operations can be enqueued using this buffer, implementations can free resource allocations, including mapped memory that was just unmapped.    

## 5.4 内存映射(Buffer Mapping)
> 5.4. Buffer Mapping

应用程序可以请求映射 GPUBuffer，以便他们可以通过ArrayBuffers访问其内容，ArrayBuffers 代表着 GPUBuffer 在GPU端分配的部分内存。映射GPUBuffer是通过 mapAsync() 异步请求的，这样用户代理可以确保GPU在应用程序访问其内容之前完成对GPUBuffer的使用。被映射的 GPUBuffer 不能被GPU自身使用，要想使用它，必须在将工作提交到队列时间线之前，调用 unmap()解除映射。

> An application can request to map a GPUBuffer so that they can access its content via ArrayBuffers that represent part of the GPUBuffer's allocations. Mapping a GPUBuffer is requested asynchronously with mapAsync() so that the user agent can ensure the GPU finished using the GPUBuffer before the application can access its content. A mapped GPUBuffer cannot be used by the GPU and must be unmapped using unmap() before work using it can be submitted to the Queue timeline.

一旦 GPUBuffer 被映射，应用程序可以通过getMappedRange()同步请求访问其内容的范围。返回的ArrayBuffer只能通过unmap()分离（直接，或通过GPUBuffer.destroy()或GPUDevice.destroy()），而不能被转换(transferred)。任何试图这样做的其他操作都会抛出一个TypeError。

> Once the GPUBuffer is mapped, the application can synchronously ask for access to ranges of its content with getMappedRange(). The returned ArrayBuffer can only be detached by unmap() (directly, or via GPUBuffer.destroy() or GPUDevice.destroy()), and cannot be transferred. A TypeError is thrown by any other operation that attempts to do so.

```
typedef [EnforceRange] unsigned long GPUMapModeFlags;
[Exposed=(Window, DedicatedWorker)]
namespace GPUMapMode {
    const GPUFlagsConstant READ  = 0x0001;
    const GPUFlagsConstant WRITE = 0x0002;
};
```

GPUMapMode 标志决定了在调用 mapAsync() 时如何映射GPUBuffer。    
> READ 
> 只对以 MAP_READ 方式创建的缓冲区有效。
> 
> 一旦缓冲区被映射，调用getMappedRange()将返回一个包含缓冲区当前值的ArrayBuffer。在调用 unmap() 后，对返回的ArrayBuffer的更改将被丢弃。

> The GPUMapMode flags determine how a GPUBuffer is mapped when calling mapAsync():
>> **READ**    
>> Only valid with buffers created with the MAP_READ usage.    
>>    
>> Once the buffer is mapped, calls to getMappedRange() will return an ArrayBuffer containing the buffer’s current values. Changes to the returned ArrayBuffer will be discarded after unmap() is called.    

> WRITE    
> 只对以 MAP_WRITE 方式创建的缓冲区有效。    
> 一旦缓冲区被映射，调用getMappedRange()将返回一个包含缓冲区当前值的ArrayBuffer。在调用 unmap()后，对返回的ArrayBuffer的更改将被保存。    
> **注意：** 由于 MAP_WRITE 缓冲区的使用 只能与 COPY_SRC缓冲区的使用相结合，用于写入的映射永远不能返回由 GPU 自身产生的值，返回的 ArrayBuffer 将只包含默认的初始化数据（0）或由网页在之前的映射中写入的数据。

>> **WRITE**    
>> Only valid with buffers created with the MAP_WRITE usage.    
>>    
>> Once the buffer is mapped, calls to getMappedRange() will return an ArrayBuffer containing the buffer’s current values. Changes to the returned ArrayBuffer will be stored in the GPUBuffer after unmap() is called.    
>> **NOTE:** Since the MAP_WRITE buffer usage may only be combined with the COPY_SRC buffer usage, mapping for writing can never return values produced by the GPU, and the returned ArrayBuffer will only ever contain the default initialized data (zeros) or data written by the webpage during a previous mapping.

**mapAsync(mode, offset, size)函数**        
用于利用给定范围来映射 GPUBuffer，当GPUBuffer的内容可以用getMappedRange()访问时，返回状态resolved的Promise。

返回的Promise 成功状态只表明缓冲区已经被映射了。它不保证上下文时间线可见的任何其他操作也完成了，特别的，更不意味着从其他 GPUBuffer 上的 onSubmittedWorkDone() 或 mapAsync() 返回的任何其他Promise已经 resolved。

从 onSubmittedWorkDone() 返回成功的 Promise，确实意味着在该调用之前的 mapAsync() 调用是已经完成，即在相应队列上 GPUBuffers 最后的使用过程是完成的

>> **mapAsync(mode, offset, size)**    
>> Maps the given range of the GPUBuffer and resolves the returned Promise when the GPUBuffer's content is ready to be accessed with getMappedRange().    
>>    
>> The resolution of the returned Promise only indicates that the buffer has been mapped. It does not guarantee the completion of any other operations visible to the content timeline, and in particular does not imply that any other Promise returned from onSubmittedWorkDone() or mapAsync() on other GPUBuffers have resolved.    
>>    
>> The resolution of the Promise returned from onSubmittedWorkDone() does imply the completion of mapAsync() calls made prior to that call, on GPUBuffers last used exclusively on that queue.    


| Parameter | Type | Nullable | Optional | Description |
| -------- | -------- | -------- | -------- | -------- |
| mode | GPUMapModeFlags | ✘ | ✘ | 用于决定缓冲区是被映射用于读，还是写 |
| offset | GPUSize64 | ✘ | ✔ | 在缓冲区映射范围内，基于开始位置的偏移量，单位是字节。 |
| size | GPUSize64 | ✘ | ✔ | 冲区映射的范围，单位是字节 |

>> Called on: GPUBuffer this.    
>> Arguments:    

| Parameter | Type | Nullable | Optional | Description |
| -------- | -------- | -------- | -------- | -------- |
| mode | GPUMapModeFlags | ✘ | ✘ | Whether the buffer should be mapped for reading or writing. |
| offset | GPUSize64 | ✘ | ✔ | Offset in bytes into the buffer to the start of the range to map. |
| size | GPUSize64 | ✘ | ✔ | Size in bytes of the range to map. |

返回值：Promise<undefined>。
**上下文时间线上步骤：**    
1. 让 contentTimeline 成为当前的上下文时间线。
2. 如果this.[[pending_map]]不是null的：
    1. 返回一个以 OperationError为值，状态为rejected 的 Promise。
3. 让 p 是一个新的 Promise。
4. 将 this.[[pending_map]] 值设为 p。
5. 在 this.[[device]] 的设备时间线上发布后续验证步骤。
6. 返回 p。

>> Returns: Promise<undefined>    
>> Content timeline steps:    
>> 1. Let contentTimeline be the current Content timeline.
>> 2. If this.[[pending_map]] is not null:
>>     1. Return a promise rejected with OperationError.
>> 3. Let p be a new Promise.
>> 4. Set this.[[pending_map]] to p.
>> 5. Issue the validation steps on the Device timeline of this.[[device]].
>> 6. Return p.

**设备时间线上的 验证步骤。**    
1. 如果size大小未定：    
    .1. 让 rangeSize 为 max(0, this.size - offset)。    
    否则。    
    .1. 让 rangeSize 值为 size。

>> Device timeline validation steps:
>> 1. If size is undefined:    
>>    1. Let rangeSize be max(0, this.size - offset). 
>>          
>>    Otherwise:   
>>     
>>    1. Let rangeSize be size.

2. 如果以下任何一个条件未能得到满足：
    * 这就是一个有效的 GPUBuffer。
    * this.[[internals]].state 是 "available"。
    * offset是8的倍数。
    * rangeSize是4的倍数。
    * offset + rangeSize ≤ this.size
    * mode只包含 GPUMapMode 中定义的位。
    * mode正好只包含READ或WRITE中的一个。
    * 如果模式包含READ，那么this.usage必须包含MAP_READ。
    * 如果模式包含WRITE，那么this.usage必须包含MAP_WRITE。

>> 2. If any of the following conditions are unsatisfied:     
>> ***Valid Usage 这是备注***    
>>     * this is a valid GPUBuffer.
>>     * this.[[internals]].state is "available".
>>     * offset is a multiple of 8.
>>     * rangeSize is a multiple of 4.
>>     * offset + rangeSize ≤ this.size
>>     * mode contains only bits defined in GPUMapMode.
>>     * mode contains exactly one of READ or WRITE.
>>     * If mode contains READ then this.usage must contain MAP_READ.
>>     * If mode contains WRITE then this.usage must contain MAP_WRITE.

如果上面任何一个条件都不满足就进行以下步骤：    
1. 在 contentTimeline 上发起以下步骤：
**上下文时间线的的步骤**    
    1. 如果 this.[[pending_map]] != p:    
      **注意：** 该映射已经被unmap()取消了。
        1. 断言p为 rejected。
        2. 返回。
    2. 断言P是 pending。
    3. 将 this.[[pending_map]] 设为null，并以 OperationError为值 将 p 设置为 rejected。

2. 生成一个验证错误。
3. 返回。
>> Then:    
>>    1. Issue the following steps on contentTimeline:    
>>    **Content timeline steps:**    
>>        1. If this.[[pending_map]] != p:    
>>        **NOTE:** The map has been cancelled by unmap().
>>            1. Assert p is rejected.
>>            1. Return.
>>        2. Assert p is pending.
>>        3. Set this.[[pending_map]] to null, and reject p with an OperationError.
>>    2. Generate a validation error.
>>    3. Return.

>> ***这是最外层的第三步开如***   
3. 将 this.[[internals]].state 设为 "unavailable"。
4. 在一个未指定的点:    
    * 在此实例上，当前排队的操作完成后时。
    * 并且不迟于当前设备时间线获悉所有当前排队的操作的完成状态后的下一个设备时间线操作（无论它们是否使用这个实例）。    
    在 当前时间线上发起后续步骤：    
    **注意：**由于缓冲区被映射了，它的内容在此实例创建完成和unmap()之间不能改变 ？？。

>> 3. Set this.[[internals]].state to "unavailable".   
>> 4. At an unspecified point:
>>    * after the completion of currently-enqueued operations that use this,
>>    * and no later than the next device timeline operation after the device timeline becomes informed of the completion of all currently-enqueued operations (regardless of whether they use this),    
>>    issue the subsequent steps on the current timeline.
>>    **NOTE:** Since the buffer is mapped, its contents cannot change between this completion and unmap().

**设备时间线上的步骤：**  
1. 让 internalStateAtCompletion 成为 this.[[internals]].state。    
  **注意：** 如果，也只有当此时缓冲区由于unmap()调用而变得再次 "可用"，那么 [[pending_map]] != p ，所以映射在下面的步骤中不会成功。
2. 让 dataForMappedRegion 指向 从开始位置，偏移相应offset量位置处的内容，是 rangeSize字节 大小。
3. 在 contentTimeline 上发起后续步骤：

>> **Device timeline steps:**
>> 1. Let internalStateAtCompletion be this.[[internals]].state.    
>> **NOTE:** If, and only if, at this point the buffer has become "available" again due to an unmap() call, then [[pending_map]] != p below, so mapping will not succeed in the steps below.
>> 2. Let dataForMappedRegion be the contents of this starting at offset offset, for rangeSize bytes.
>> 3. Issue the subsequent steps on contentTimeline.

上下文时间线上步骤：    
1. 如果 this.[[pending_map]] != p:    
    **注意：** 该映射已经被unmap()取消了。
      1. 断言 p 为 rejected。
      2. 返回。
2. 断言 P 是 pending。
3. 断言 internalStateAtCompletion为 "unavailable"。
4. 让映射过程初始化一个活动的缓冲区映射，模式为mode，范围为 [offset, offset + rangeSize]。    
    如果这次分配失败：    
    1. 将 this.[[pending_map]] 设置为 null，并以RangeError为值, 将 p 设置为 rejected。
    2. 返回。
5. 将 mapping.data 的内容设置为 dataForMappedRegion。
6. 将 this.[[mapping]] 设置为当前的 mapping。
7. 将 this.[[pending_map]] 设置为null，并 p 的状态设置为 resolved。

>> **Content timeline steps:**    
>> 1. If this.[[pending_map]] != p:    
>>     **NOTE:** The map has been cancelled by unmap().
>>      1. Assert p is rejected.
>>      2. Return.
>> 2. Assert p is pending.
>> 3. Assert internalStateAtCompletion is "unavailable".
>> 4. Let mapping be initialize an active buffer mapping with mode mode and range [offset, offset + rangeSize].    
>>  If this allocation fails:    
>>      1. Set this.[[pending_map]] to null, and reject p with a RangeError.
>>      2. Return.
>> 5. Set the content of mapping.data to dataForMappedRegion.
>> 6. Set this.[[mapping]] to mapping.
>> 7. Set this.[[pending_map]] to null, and resolve p.


***这是下一个函数了***    

getMappedRange(offset, size)   
返回一个 ArrayBuffer，其中包含给定映射范围内的 GPUBuffer 的内容。    
调用于：GPUBuffer 实例自身。    
参数：    

| Parameter | Type | Nullable | Optional | Description |
| -------- | -------- | -------- | -------- | -------- |
| offset | GPUSize64 | ✘ | ✔ | 单位是字节，用于指定从缓冲区的多少偏移量开始返回内容。 |
| size | GPUSize64 | ✘ | ✔ | 单位是字节，用于指定返回多少大小的内容。 |

返回值： ArrayBuffer

> **getMappedRange(offset, size)**    
> Returns an ArrayBuffer with the contents of the GPUBuffer in the given mapped range.

>> Called on: GPUBuffer this.
>> Arguments:

| Parameter | Type | Nullable | Optional | Description |
| -------- | -------- | -------- | -------- | -------- |
| offset | GPUSize64 | ✘ | ✔ | Offset in bytes into the buffer to return buffer contents from. |
| size | GPUSize64 | ✘ | ✔ | Size in bytes of the ArrayBuffer to return. |

>> Returns: ArrayBuffer

**上下文时间线上步骤：**    
1. 如果缺少尺寸(size):    
    1. 让 rangeSize 为 max(0, this.size - offset)。    
    否则，让 rangeSize 为 size。    
2. 如果以下任何条件不满足，则抛出一个OperationError并停止。    
    * this.[[mapping]] 不是空的。
    * offset是8的倍数。
    * rangeSize是4的倍数。
    * offset ≥ this.[[mapping]].range[0]。
    * offset + rangeSize ≤ this.[[mapping]].range[1].
    * [offset, offset + rangeSize)不与 this.[[mapping]].views 中的其他范围相重叠。

    **注意：** 在mappedAtCreation时 获取一个 GPUBuffer 的映射范围(mapped ranges)总是有效的，即使他是无效的，这是因为上下文时间线可能不知道它是无效的。

>> **Content timeline steps:**    
>> 1. If size is missing:    
>>     1. Let rangeSize be max(0, this.size - offset).    
>> Otherwise, let rangeSize be size.
>> 2. If any of the following conditions are unsatisfied, throw an OperationError and stop.
>>     * this.[[mapping]] is not null.
>>     * offset is a multiple of 8.
>>     * rangeSize is a multiple of 4.
>>     * offset ≥ this.[[mapping]].range[0].
>>     * offset + rangeSize ≤ this.[[mapping]].range[1].
>>     * [offset, offset + rangeSize) does not overlap another range in this.[[mapping]].views.    
>>     **NOTE:** It is always valid to get mapped ranges of a GPUBuffer that is mappedAtCreation, even if it is invalid, because the Content timeline might not know it is invalid.

3. 让 data 成为 this.[[mapping]].data。
4. 让视图(view)成为创建一个大小为rangeSize的ArrayBuffer，但它的指针是可变地，引用偏移量为（offset - [[mapping]].range[0]）处的数据内容。
    **注意：** 这里可能不会抛出 RangeError，因为数据已经在 mapAsync() 或 createBuffer() 中被分配。
5. 设置 view.[[ArrayBufferDetachKey]] 为 "WebGPUBufferMapping"。
    **注意：** 如果不是通过 unmap() 方法，尝试采用别的方式解构ArrayBuffer(DetachArrayBuffer)，这将导致抛出一个TypeError。
6. 将view追加到this.[[mapping]].views。 
7. 返回view。    
**注意：** 如果 getMappedRange() 在没有检查映射状态的情况下成功的返回 ArrayBuffer，用户代理应该考虑向开发者发出一个清晰可见的警告，通过等待 mapAsync() 成功，查询 mapState 的状态是否为 "mapped"，或者通过等待后面调用  onSubmittedWorkDone() 是否成功。

>> 3. Let data be this.[[mapping]].data.
>> 4. Let view be ! create an ArrayBuffer of size rangeSize, but with its pointer mutably referencing the content of data at offset (offset - [[mapping]].range[0]).    
>> **NOTE:** A RangeError may not be thrown here, because the data has already been allocated during mapAsync() or createBuffer().
>> 5. Set view.[[ArrayBufferDetachKey]] to "WebGPUBufferMapping".    
>> **NOTE:** This causes a TypeError to be thrown if an attempt is made to DetachArrayBuffer, except by unmap().
>> 6. Append view to this.[[mapping]].views.
>> 7. Return view.    
>> **NOTE:** User agents should consider issuing a developer-visible warning if getMappedRange() succeeds without having checked the status of the map, by waiting for mapAsync() to succeed, querying a mapState of "mapped", or waiting for a later onSubmittedWorkDone() call to succeed.

**unmap()**     
解除 已映射成功的 GPUBuffer 的映射范围，使其所映射的内存范围可以再次供 GPU 使用。

> **unmap()**
> Unmaps the mapped range of the GPUBuffer and makes it’s contents available for use by the GPU again.

调用于： GPUBuffer 实例本身。    
返回值： undefined    
**上下文时间线上步骤:**
1. 如果 this.[[pending_map]] 不是 null 。
    1. 将相应于 this.[[pending_map]] 的 Promise 设置为 rejected，其错误值为： AbortError。
    2. 将 this.[[pending_map]] 设置为 null。
2. 如果 this.[[mapping]] 是 null。
    1. 返回。
3. 对于在 this.[[mapping]].views 中的每个ArrayBuffer，假如其名称为 ab，执行以下操作：
    1. 执行 DetachArrayBuffer(ab, "WebGPUBufferMapping")。

>> **Called on:** GPUBuffer this.    
>> **Returns:** undefined
>> **Content timeline steps:**    
>> 1. If this.[[pending_map]] is not null:    
>>    1. Reject this.[[pending_map]] with an AbortError.
>>    2. Set this.[[pending_map]] to null.
>> 2. If this.[[mapping]] is null:
>>    1. Return.
>> 3. For each ArrayBuffer ab in this.[[mapping]].views:
>>    1. Perform DetachArrayBuffer(ab, "WebGPUBufferMapping").

4. 让 bufferUpdate 为 null。
5. 如果 this.[[mapping]].mode 包含 WRITE 标记：
    1. 设置 bufferUpdate 为 { data: this.[[mapping]].data, offset: this.[[mapping]].range[0] }。
    **注意：** 当缓冲区是在没有 WRITE 模式标记的情况下而映射的，那么，在解除映射后，应用程序对此 ArrayBuffer 对应的映射范围内内容，所做的任何局部修改都会被丢弃，并且在以后的映射中，也不会影响到其内容。
6. 将 this.[[mapping]] 设为 null。
7. 在 this.[[device]] 的设备时间轴上发起后续步骤：

>> 4. Let bufferUpdate be null.
>> 5. If this.[[mapping]].mode contains WRITE:
>>    1. Set bufferUpdate to { data: this.[[mapping]].data, offset: this.[[mapping]].range[0] }.    
>>    **NOTE:** When a buffer is mapped without the WRITE mode, then unmapped, any local modifications done by the application to the mapped ranges ArrayBuffer are discarded and will not affect the content of later mappings.
>> 6. Set this.[[mapping]] to null.
>> 7. Issue the subsequent steps on the Device timeline of this.[[device]].

**设备的时间轴步骤。**     
1. 如果this.[[device]] 是无效的，返回。
2. 如果 bufferUpdate 不是 null:     
    1. 在 this.[[device]].queue 的 队列时间线上发起以下步骤：
        **队列时间线上步骤**     
        用bufferUpdate.data 中的数据 更新(还原) 偏移量为 bufferUpdate.offset 处用数据内容。
3. 设置 this.[[internals]].state 为 "available"。

>> **Device timeline steps:**    
>> 1. If this.[[device]] is invalid, return.
>> 2. If bufferUpdate is not null:
>>    1. Issue the following steps on the Queue timeline of this.[[device]].queue:    
>>    **Queue timeline steps:**    
>>        1. Update the contents of this at offset bufferUpdate.offset with the data bufferUpdate.data.
>> 3. Set this.[[internals]].state to "available".
