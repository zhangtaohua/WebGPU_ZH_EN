WebGPU

W3C Working Draft, 6 April 2023 

More details about this document

This version:

https://www.w3.org/TR/2023/WD-webgpu-20230406/

---
---
# 20\. 查询

## 20.1. GPU查询集

```
[Exposed=(Window, DedicatedWorker), SecureContext]
interface GPUQuerySet {
    undefined destroy();

    readonly attribute GPUQueryType type;
    readonly attribute GPUSize32 count;
};
GPUQuerySet includes GPUObjectBase;
```

---

20\. Queries

20.1. GPUQuerySet

```
[Exposed=(Window, DedicatedWorker), SecureContext]
interface GPUQuerySet {
    undefined destroy();

    readonly attribute GPUQueryType type;
    readonly attribute GPUSize32 count;
};
GPUQuerySet includes GPUObjectBase;
```

---
---

GPUQuerySet 具有以下属性：

**type, 类型为： GPUQueryType, 只读**       
由这个 GPUQuerySet 所管理的查询类型。

**count，类型为： GPUSize32，只读**
由这个 GPUQuerySet 所管理的查询 数量。

GPUQuerySet有以下内部插槽：

[[state]], 类型为： query set state
GPUQuerySet 的当前状态。

每个 GPUQuerySet 在设备时间线上都有一个当前的查询集状态，它是以下之一：

"available"
在这个上下文中， GPUQuerySet 对 GPU的操作是可用的。

"destroyed"
GPUQuerySet 不再可用于任何操作，除了销毁操作。

---

GPUQuerySet has the following attributes:

**type, of type GPUQueryType, readonly**        
The type of the queries managed by this GPUQuerySet.

**count, of type GPUSize32, readonly**          
The number of queries managed by this GPUQuerySet.

GPUQuerySet has the following internal slots:

**[[state]], of type query set state**      
The current state of the GPUQuerySet.

Each GPUQuerySet has a current query set state on the Device timeline which is one of the following:

**"available"**     
The GPUQuerySet is available for GPU operations on its content.

**"destroyed"**         
The GPUQuerySet is no longer available for any operations except destroy.

---
---

### 20.1.1. 查询集的创建

GPUQuerySetDescriptor 指定了创建 GPUQuerySet 时使用的选项描述符。

```
dictionary GPUQuerySetDescriptor : GPUObjectDescriptorBase {
    required GPUQueryType type;
    required GPUSize32 count;
};
```

**type，类型为： GPUQueryType**     
由 GPUQuerySet 所管理的查询类型。

**count，类型为： GPUSize32**       
由 GPUQuerySet 所管理的查询数量。

---

20.1.1. QuerySet Creation

A GPUQuerySetDescriptor specifies the options to use in creating a GPUQuerySet.

```
dictionary GPUQuerySetDescriptor : GPUObjectDescriptorBase {
    required GPUQueryType type;
    required GPUSize32 count;
};
```

**type, of type GPUQueryType**      
The type of queries managed by GPUQuerySet.

**count, of type GPUSize32**            
The number of queries managed by GPUQuerySet.

---
---

**createQuerySet(descriptor)**

创建一个 GPUQuerySet。

**调用于：** GPUDevice 实例自身。

**参数：**

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| descriptor | GPUQuerySetDescriptor | ✘ | ✘ | 要创建的 GPUQuerySet 的描述。 | 

**返回值：** GPUQuerySet

---

**createQuerySet(descriptor)**  

Creates a GPUQuerySet.

**Called on:** GPUDevice this.

**Arguments:**

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| descriptor | GPUQuerySetDescriptor | ✘ | ✘ | Description of the GPUQuerySet to create. | 

Returns: GPUQuerySet

---
---

**上下文时间轴步骤：**
1. 如果 descriptor.type 是 "timestamp"，但 "timestamp-query " 并没有为此启用：
    1. 抛出一个 TypeError。
2. 让 q 成为一个新的 GPUQuerySet 对象。
3. 将 q.type 设置为 descriptor.type。
4. 将 q.count 设置为 descriptor.count。
5. 在实例 this 的设备时间轴上发起初始化步骤。
6. 返回 q。

---

**Content timeline steps:**
1. If descriptor.type is "timestamp", but "timestamp-query" is not enabled for this:
    1. Throw a TypeError.
2. Let q be a new GPUQuerySet object.
3. Set q.type to descriptor.type.
4. Set q.count to descriptor.count.
5. Issue the initialization steps on the Device timeline of this.
6. Return q.

---
---

**设备时间轴的初始化步骤：**
1. 如果以下任何要求没有得到满足，产生一个验证错误，使 q 无效，并停止。
    * this 为有效。
    * descriptor.count 必须小于等于 4096。
2、 设置q.[[stage]] 为可用的。

---

**Device timeline initialization steps:**

1. If any of the following requirements are unmet, generate a validation error, make q invalid, and stop.
    * this is valid.
    * descriptor.count must be ≤ 4096.
2. Set q.[[state]] to available.

---
---

```
// 例子 26
// 创建一个GPUQuerySet，其中包含 32 个闭塞查询结果。

const querySet = gpuDevice.createQuerySet({
    type: 'occlusion',
    count: 32
});

```

---

```
// EXAMPLE 26
// Creating a GPUQuerySet which holds 32 occlusion query results.

const querySet = gpuDevice.createQuerySet({
    type: 'occlusion',
    count: 32
});
```

---
---

### 20.1.2. 查询集的销毁

不再需要 GPUQuerySet 的应用程序可以选择调用 destroy() 函数，这样在垃圾收集之前就可以失去对 GPUQuerySet 的访问。

**destroy()**       

销毁 GPUQuerySet。

**调用于：** GPUQuerySet 实例自身。

**返回值：** undefined

**上下文时间线步骤：**
1. 将this.[[state]] 设置为销毁的，即 destroyed。

---

20.1.2. QuerySet Destruction

An application that no longer requires a GPUQuerySet can choose to lose access to it before garbage collection by calling destroy().

**destroy()**       

Destroys the GPUQuerySet.

**Called on:** GPUQuerySet this.

**Returns:** undefined

**Content timeline steps:**
1. Set this.[[state]] to destroyed.

---
---

## 20.2. 查询类型

```
enum GPUQueryType {
    "occlusion",
    "timestamp"
};
```

---

20.2. QueryType

```
enum GPUQueryType {
    "occlusion",
    "timestamp"
};
```

---
---

## 20.3. 阻塞查询

阻塞查询仅适用于渲染通道，用于查询通过了 一组绘制命令的所有分片段测试的 片段样本的数量，包括裁剪、样本遮挡、alpha到覆盖、模板和深度测试。任何非零的查询结果值都表示至少有一个样本通过了测试，并且到达了渲染管道的输出混合阶段，0表示没有样本通过测试。

在开始渲染通道时，必须设置 GPURenderPassDescriptor.occlusionQuerySet ，以便在此通道中使用阻塞查询。一个阻塞查询是通过成对调用 beginOcclusionQuery() 和 endOcclusionQuery() 开始和结束的，且是不能嵌套。

---

20.3. Occlusion Query

Occlusion query is only available on render passes, to query the number of fragment samples that pass all the per-fragment tests for a set of drawing commands, including scissor, sample mask, alpha to coverage, stencil, and depth tests. Any non-zero result value for the query indicates that at least one sample passed the tests and reached the output merging stage of the render pipeline, 0 indicates that no samples passed the tests.

When beginning a render pass, GPURenderPassDescriptor.occlusionQuerySet must be set to be able to use occlusion queries during the pass. An occlusion query is begun and ended by calling beginOcclusionQuery() and endOcclusionQuery() in pairs that cannot be nested.

---
---

## 20.4. 时间戳查询

时间戳查询允许应用程序将时间戳写入 GPUQuerySet 中，使用：
* GPUCommandEncoder.writeTimestamp()
* GPUComputePassDescriptor.timestampWrites
* GPURenderPassDescriptor.timestampWrites

然后使用GPUCommandEncoder.resolveQuerySet()， 将时间戳值的解析值写入为 GPUBuffer（以纳秒为单位，采用64位无符号整数记录）。

时间戳查询需要设备启用 "timestamp-query" 功能。

---

20.4. Timestamp Query

Timestamp queries allow applications to write timestamps to a GPUQuerySet, using:
* GPUCommandEncoder.writeTimestamp()
* GPUComputePassDescriptor.timestampWrites
* GPURenderPassDescriptor.timestampWrites

and then resolve timestamp values (in nanoseconds as a 64-bit unsigned integer) into a GPUBuffer, using GPUCommandEncoder.resolveQuerySet().

Timestamp query requires "timestamp-query" to be enabled for the device.

---
---

**注意：** 如果物理设备重置时间戳计数器，时间戳值可能为零，请忽略它和它的后续数值。

>> ISSUE 32 编写关于时间戳值重置的规范性描述。

>> ISSUE 33 因为时间戳查询提供了高分辨率的GPU时间戳，如果有的话，我们需要决定对它的可用性有什么限制。

---

**NOTE:** The timestamp values may be zero if the physical device reset timestamp counter, please ignore it and the following values.

**ISSUE 32** Write normative text about timestamp value resets.

**ISSUE 33** Because timestamp query provides high-resolution GPU timestamp, we need to decide what constraints, if any, are on its availability.

---
---

**Validate timestampWrites(GPUDevice device, (GPUComputePassTimestampWrites or GPURenderPassTimestampWrites) timestampWrites)**     

如果满足以下要求则返回true，不满足则返回false。
* 如果 timestampWrites 不是空的，设备必须启用 "timestamp-query" 功能。
* timestampWrites 中没有两个条目可以有相同的 location (时区，还是存储位置呢？)。
* 对于 timestampWrites 中的每个 timestampWrite：
    * 在与设备实例一起使用时，timestampWrite.querySet 必须是有效的。
    * timestampWrite.querySet.type 是 "timestamp"。
    * timestampWrite.queryIndex < timestampWrite.querySet.count。

---

**Validate timestampWrites(GPUDevice device, (GPUComputePassTimestampWrites or GPURenderPassTimestampWrites) timestampWrites)**     

Return true if the following requirements are met, and false if not.
* If timestampWrites is not empty, "timestamp-query" must be enabled for device.
* No two entries in timestampWrites may have the same location.
* For each timestampWrite in timestampWrites:
    * timestampWrite.querySet must be valid to use with device.
    * timestampWrite.querySet.type is "timestamp".
    * timestampWrite.queryIndex < timestampWrite.querySet.count.

