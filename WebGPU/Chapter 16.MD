# WebGPU

W3C Working Draft, 28 March 2023

More details about this document

This version:

https://www.w3.org/TR/2023/WD-webgpu-20230328/

---
---

# 16\. 计算管道
## 16.1. GPU计算管道编码器

```
[Exposed=(Window, DedicatedWorker), SecureContext]
interface GPUComputePassEncoder {
    undefined setPipeline(GPUComputePipeline pipeline);
    undefined dispatchWorkgroups(GPUSize32 workgroupCountX, optional GPUSize32 workgroupCountY = 1, optional GPUSize32 workgroupCountZ = 1);
    undefined dispatchWorkgroupsIndirect(GPUBuffer indirectBuffer, GPUSize64 indirectOffset);

    undefined end();
};
GPUComputePassEncoder includes GPUObjectBase;
GPUComputePassEncoder includes GPUCommandsMixin;
GPUComputePassEncoder includes GPUDebugCommandsMixin;
GPUComputePassEncoder includes GPUBindingCommandsMixin;
```

GPUComputePassEncoder 有以下内部插槽：

**[[command_encoder]]，类型为： GPUCommandEncoder，只读**       
创建这个计算通道编码器的 GPUCommandEncoder。

**[[pipeline]], 类型为： GPUComputePipeline, 只读**
当前的 GPUComputePipeline，初始化设置为空。

**[[endTimestampWrites]], 类型为： GPUComputePassTimestampWrites, 只读**        
当管道结束时需要执行的时间戳附件。

---
16\. Compute Passes

16.1. GPUComputePassEncoder

```
[Exposed=(Window, DedicatedWorker), SecureContext]
interface GPUComputePassEncoder {
    undefined setPipeline(GPUComputePipeline pipeline);
    undefined dispatchWorkgroups(GPUSize32 workgroupCountX, optional GPUSize32 workgroupCountY = 1, optional GPUSize32 workgroupCountZ = 1);
    undefined dispatchWorkgroupsIndirect(GPUBuffer indirectBuffer, GPUSize64 indirectOffset);

    undefined end();
};
GPUComputePassEncoder includes GPUObjectBase;
GPUComputePassEncoder includes GPUCommandsMixin;
GPUComputePassEncoder includes GPUDebugCommandsMixin;
GPUComputePassEncoder includes GPUBindingCommandsMixin;
```

GPUComputePassEncoder has the following internal slots:
**[[command_encoder]], of type GPUCommandEncoder, readonly**     
The GPUCommandEncoder that created this compute pass encoder.

**[[pipeline]], of type GPUComputePipeline, readonly**        
The current GPUComputePipeline, initially null.

**[[endTimestampWrites]], of type GPUComputePassTimestampWrites, readonly**       
The timestamp attachments which need to be executed when the pass ends.

---
---

### 16.1.1. 计算管道编码器的创建

```
enum GPUComputePassTimestampLocation {
    "beginning",
    "end"
};

dictionary GPUComputePassTimestampWrite {
    required GPUQuerySet querySet;
    required GPUSize32 queryIndex;
    required GPUComputePassTimestampLocation location;
};

typedef sequence<GPUComputePassTimestampWrite> GPUComputePassTimestampWrites;

dictionary GPUComputePassDescriptor : GPUObjectDescriptorBase {
    GPUComputePassTimestampWrites timestampWrites = [];
};
```

**timestampWrites，类型为： GPUComputePassTimestampWrites，默认为[] 。**
一系列的 GPUComputePassTimestampWrite 值定义了这个通道将在何处和何时将时间戳的值写入。

---

16.1.1. Compute Pass Encoder Creation

```
enum GPUComputePassTimestampLocation {
    "beginning",
    "end"
};

dictionary GPUComputePassTimestampWrite {
    required GPUQuerySet querySet;
    required GPUSize32 queryIndex;
    required GPUComputePassTimestampLocation location;
};

typedef sequence<GPUComputePassTimestampWrite> GPUComputePassTimestampWrites;

dictionary GPUComputePassDescriptor : GPUObjectDescriptorBase {
    GPUComputePassTimestampWrites timestampWrites = [];
};
```

**timestampWrites, of type GPUComputePassTimestampWrites, defaulting to []**        
A sequence of GPUComputePassTimestampWrite values define where and when timestamp values will be written for this pass.

---
---

### 16.1.2. 调度

**setPipeline(pipeline)**       

设置当前的 GPUComputePipeline。

调用于：GPUComputePassEncoder 实例自身。

参数：

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| `pipeline` | `GPUComputePipeline` | ✘ | ✘ | 后续调度命令要使用的计算管道。 |

返回： undefined

**内容时间线步骤：**
1. 在 this.[[device]] 的设备时间线上发起后续步骤。

**设备时间线步骤：**    
1. 验证实例的编码器状态，如果它返回值为false，则停止。
2. 如果以下任何条件不满足，使之无效并停止。
    * 实例 this 所指向的管道是有效的。
3. 设置 this.[[pipeline]] 为 pipeline。

---

16.1.2. Dispatch

**setPipeline(pipeline)**       
Sets the current GPUComputePipeline.

**Called on:** GPUComputePassEncoder this.  

**Arguments:**

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| `pipeline` | `GPUComputePipeline` | ✘ | ✘ | The compute pipeline to use for subsequent dispatch commands. |

Returns: undefined

**Content timeline steps:**
1. Issue the subsequent steps on the Device timeline of this.[[device]].

**Device timeline steps:**
1. Validate the encoder state of this. If it returns false, stop.
2. If any of the following conditions are unsatisfied, make this invalid and stop.
    * pipeline is valid to use with this.
3. Set this.[[pipeline]] to be pipeline.

---
---

**dispatchWorkgroups(workgroupCountX, workgroupCountY, workgroupCountZ)**   

用当前 GPUComputePipeline 调度要执行的工作。详细规范参见 § 23.2 计算。

调用于：GPUComputePassEncoder 实例自身。

参数：

| Parameter| Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| workgroupCountX | GPUSize32 | ✘ | ✘ | 要分配的工作组网格的 X 尺寸。 |
| workgroupCountY | GPUSize32 | ✘ | ✔ | 要分配的工作组网格的 Y 尺寸。 |
| workgroupCountZ | GPUSize32 | ✘ | ✔ | 要分配的工作组网格的 Z 尺寸。 |

**注意：** 传递给 dispatchWorkgroups() 和 dispatchWorkgroupsIndirect() 的x、y和z值是每个维度要调度的工作组的数量，而不是每个维度要执行的着色器调用的数量。这与现代原生GPU APIs的行为相匹配，但与 OpenCL 的行为不同。

这意味着，如果一个 GPUShaderModule 定义了一个 @workgroup_size(4, 4) 的入口点，并通过调用computePass.dispatchWorkgroups(8, 8) 将工作分派给了它；该入口点函数总共将被调用1024次：沿着X轴和Y轴，将一个 4x4 的工作组调度8次。(4*4*8*8=1024)

返回： undefined 。

---

**dispatchWorkgroups(workgroupCountX, workgroupCountY, workgroupCountZ)**

Dispatch work to be performed with the current GPUComputePipeline. See § 23.2 Computing for the detailed specification.

**Called on:** GPUComputePassEncoder this.

**Arguments:**

| Parameter| Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| workgroupCountX | GPUSize32 | ✘ | ✘ | X dimension of the grid of workgroups to dispatch |
| workgroupCountY | GPUSize32 | ✘ | ✔ | Y dimension of the grid of workgroups to dispatch. |
| workgroupCountZ | GPUSize32 | ✘ | ✔ | Z dimension of the grid of workgroups to dispatch. |

**Note:** The x, y, and z values passed to dispatchWorkgroups() and dispatchWorkgroupsIndirect() are the number of workgroups to dispatch for each dimension, not the number of shader invocations to perform across each dimension. This matches the behavior of modern native GPU APIs, but differs from the behavior of OpenCL.

This means that if a GPUShaderModule defines an entry point with @workgroup_size(4, 4), and work is dispatched to it with the call computePass.dispatchWorkgroups(8, 8); the entry point will be invoked 1024 times total: Dispatching a 4x4 workgroup 8 times along both the X and Y axes. (4*4*8*8=1024)

Returns: undefined

---
---

**内容时间线步骤：**
1. 在 this.[[device]] 的设备时间线上发起后续步骤。

**设备时间线步骤：**    
1. 验证实例的编码器状态，如果它返回值为false，则停止。
2. 如果以下任何条件不满足，使之无效并停止。
    * 验证`编码器绑定groups(this, this.[[pipeline]])`为真。
    * 所有的 workgroupCountX、workgroupCountY 和 workgroupCountZ 都小于等于 this.device.limit.maxComputeWorkgroupsPerDimension。
3. 让 passState 成为实例当前状态的一个快照。
4. 在此实例的队列中入队一个命令，当此命令在队列时间线上执行时，将发起后续步骤。

**队列时间线的步骤：**
1. 使用 passState.[[bind_groups]]，结合 passState.[[pipeline]]，执行一个 `维度为 [workgroupCountX, workgroupCountY, workgroupCountZ]`  的工作组的一个网络。

---

**Content timeline steps:** 
1. Issue the subsequent steps on the Device timeline of this.[[device]].

**Device timeline steps:**
1. Validate the encoder state of this. If it returns false, stop.
2. If any of the following conditions are unsatisfied, make this invalid and stop.
    * Validate encoder bind groups(this, this.[[pipeline]]) is true.
    * all of workgroupCountX, workgroupCountY and workgroupCountZ are ≤ this.device.limits.maxComputeWorkgroupsPerDimension.
3. Let passState be a snapshot of this’s current state.
4. Enqueue a command on this which issues the subsequent steps on the Queue timeline.

**Queue timeline steps:**       
1. Execute a grid of workgroups with dimensions [workgroupCountX, workgroupCountY, workgroupCountZ] with passState.[[pipeline]] using passState.[[bind_groups]].

---
---

**dispatchWorkgroupsIndirect(indirectBuffer, indirectOffset)**          
使用从 GPUBuffer 中读取的参数对当前 GPUComputePipeline 进行调度工作。详细规范请参见 § 23.2 计算。

缓冲区中编码的间接调度参数必须是由三个32位无符号整数值组成的紧密包块（共12个字节），其顺序与 dispatchWorkgroups() 的参数相同。比如说：

```
let dispatchIndirectParameters = new Uint32Array(3);
dispatchIndirectParameters[0] = workgroupCountX;
dispatchIndirectParameters[1] = workgroupCountY;
dispatchIndirectParameters[2] = workgroupCountZ;
```

---

**dispatchWorkgroupsIndirect(indirectBuffer, indirectOffset)**      

Dispatch work to be performed with the current GPUComputePipeline using parameters read from a GPUBuffer. See § 23.2 Computing for the detailed specification.

The indirect dispatch parameters encoded in the buffer must be a tightly packed block of three 32-bit unsigned integer values (12 bytes total), given in the same order as the arguments for dispatchWorkgroups(). For example:

```
let dispatchIndirectParameters = new Uint32Array(3);
dispatchIndirectParameters[0] = workgroupCountX;
dispatchIndirectParameters[1] = workgroupCountY;
dispatchIndirectParameters[2] = workgroupCountZ;
```

---
---

调用于： GPUComputePassEncoder 实例自身.

参数：

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| indirectBuffer | GPUBuffer | ✘ | ✘ | 包含间接调度参数的缓冲区。 | 
| indirectOffset | GPUSize64 | ✘ | ✘ | 以字节为单位，间接调度参数缓冲区中调度数据开始的  数据的偏移量。 | 

返回： undefined

**上下文时间轴步骤：**      
1. 在 this.[[device]] 的设备时间线上发起后续步骤。

---

**Called on:** GPUComputePassEncoder this.

**Arguments:**

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| indirectBuffer | GPUBuffer | ✘ | ✘ | Buffer containing the indirect dispatch parameters. | 
| indirectOffset | GPUSize64 | ✘ | ✘ | Offset in bytes into indirectBuffer where the dispatch data begins. | 

**Returns:** undefined

**Content timeline steps:**
1. Issue the subsequent steps on the Device timeline of this.[[device]].

---
---

**设备时间轴步骤：**      
1. 验证实例的编码器状态，如果它返回值为false，则停止。
2. 如果以下任何条件不满足，使之无效并停止。
    * 验证`编码器绑定groups(this, this.[[pipeline]])`为真。
    * 实例 this 所指向的 indirectBuffer是有效的。
    * indirectBuffer.usage 包含 INDIRECT。
    * indirectOffset + sizeof(indirect dispatch parameters) ≤ indirectBuffer.size。
    * indirectOffset 是 4 的倍数。
3. 将 indirectBuffer 作为 INDIRECT 添加到使用范围中。
4. 让 passState 成为这个实例当前状态的一个快照。
5. 在此实例的队列中入队一个命令，当此命令在队列时间线上执行时，将发起后续步骤。

---

**Device timeline steps:**
1. Validate the encoder state of this. If it returns false, stop.
2. If any of the following conditions are unsatisfied, make this invalid and stop.
    * Validate encoder bind groups(this, this.[[pipeline]]) is true.
    * indirectBuffer is valid to use with this.
    * indirectBuffer.usage contains INDIRECT.
    * indirectOffset + sizeof(indirect dispatch parameters) ≤ indirectBuffer.size.
    * indirectOffset is a multiple of 4.
    * Add indirectBuffer to the usage scope as INDIRECT.
3. Let passState be a snapshot of this’s current state.
4. Enqueue a command on this which issues the subsequent steps on the Queue timeline.

---
---

**队列时间轴步骤：**
1. 让 workgroupCountX 是一个无符号的 32 位整数，此值是从 indirectBuffer 中 indirectOffset 偏移字节处读取的。
2. 让 workgroupCountY 是一个无符号的 32 位整数，此值是从 indirectBuffer 中 (indirectOffset + 4) 偏移字节处读取的。
3. WorkgroupCountZ 是一个无符号的 32 位整数，此值是从 indirectBuffer 中 (indirectOffset + 8) 偏移字节处读取的。
4. 如果 workgroupCountX、workgroupCountY 或 workgroupCountZ 大于 this.device.limit.maxComputeWorkgroupsPerDimension，则停止。
5. 使用 passState.[[bind_groups]]，结合 passState.[[pipeline]]，执行一个 `维度为 [workgroupCountX, workgroupCountY, workgroupCountZ]`  的工作组的一个网络。

---

**Queue timeline steps:**       
1. Let workgroupCountX be an unsigned 32-bit integer read from indirectBuffer at indirectOffset bytes.
2. Let workgroupCountY be an unsigned 32-bit integer read from indirectBuffer at (indirectOffset + 4) bytes.
3. Let workgroupCountZ be an unsigned 32-bit integer read from indirectBuffer at (indirectOffset + 8) bytes.
4. If workgroupCountX, workgroupCountY, or workgroupCountZ is greater than this.device.limits.maxComputeWorkgroupsPerDimension, stop.
5. Execute a grid of workgroups with dimensions [workgroupCountX, workgroupCountY, workgroupCountZ] with passState.[[pipeline]] using passState.[[bind_groups]].


---
---

### 16.1.3. 最终化
一旦用户完成了通道的命令记录，计算通道编码器就可以通过调用end()来结束。一旦end()被调用，计算管道编码器就不能再被使用。

**end()**

完成计算管道命令序列的记录。

调用于：GPUComputePassEncoder 实例自身。
返回： undefined

**上下文时间轴步骤：**      
1. 在 this.[[device]] 的设备时间线上发起后续步骤。

---

16.1.3. Finalization

The compute pass encoder can be ended by calling end() once the user has finished recording commands for the pass. Once end() has been called the compute pass encoder can no longer be used.

**end()**       
Completes recording of the compute pass commands sequence.

**Called on:** GPUComputePassEncoder this.
**Returns:** undefined

**Content timeline steps:**
1. Issue the subsequent steps on the Device timeline of this.[[device]].

---
---

**设备的时间轴步骤：**
1. 让 parentEncoder 成为 this.[[command_encoder]]。
2. 如果以下任何要求未能得到满足，则产生一个验证错误并停止。
    * this.[[state]] 必须是 "open"。
    * parentEncoder.[[state]]必须是 "locked"。
3. 将 this.[[state]] 设置为 "end"。
4. 将 parentEncoder.[[state]] 设置为 "open"。
5. 如果以下任何要求没有得到满足，使 parentEncoder 无效并停止。
    * this 必须是有效的。
    * this.[[debug_group_stack]] 必须为空。
6. 用 this.[[command]] 扩展 parentEncoder.[[command]]。
7. 对于 this.[[endTimestampWrites]] 中的每个 timestampWrite：
    1. 断言：timestampWrite.location 是 "end"。
    2. 在 parentEncoder.[[commands]] 中添加一个 GPU命令，该命令将 GPU 的时间戳值写入 timestampWrite.querySet 中的 timestampWrite.queryIndexth 索引处。

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
6. Extend parentEncoder.[[commands]] with this.[[commands]].
7. For each timestampWrite in this.[[endTimestampWrites]]:
    1. Assert: timestampWrite.location is "end".
    2. Append a GPU command to parentEncoder.[[commands]] that writes the GPU’s timestamp value into the timestampWrite.queryIndexth index in timestampWrite.querySet.

