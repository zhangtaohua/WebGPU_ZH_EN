# WebGPU

W3C Working Draft, 28 March 2023

More details about this document

This version:

https://www.w3.org/TR/2023/WD-webgpu-20230328/

---
---

# 15\. 调试标签

GPUDebugCommandsMixin 提供了将调试标签 应用于命令组， 或在命令序列中插入单个标签的方法。

调试组可以嵌套的创建一个深度层次的标签命令组，并且必须是可平衡的。

就像`对象标签`一样，这些标签没有必要的行为，但可以在错误信息和浏览器开发者工具中显示，也可以传递给原生API的后端使用。

```
interface mixin GPUDebugCommandsMixin {
    undefined pushDebugGroup(USVString groupLabel);
    undefined popDebugGroup();
    undefined insertDebugMarker(USVString markerLabel);
};
```

---

15\. Debug Markers

GPUDebugCommandsMixin provides methods to apply debug labels to groups of commands or insert a single label into the command sequence.

Debug groups can be nested to create a hierarchy of labeled commands, and must be well-balanced.

Like object labels, these labels have no required behavior, but may be shown in error messages and browser developer tools, and may be passed to native API backends.

```
interface mixin GPUDebugCommandsMixin {
    undefined pushDebugGroup(USVString groupLabel);
    undefined popDebugGroup();
    undefined insertDebugMarker(USVString markerLabel);
};
```

---
---

GPUDebugCommandsMixin 假定 GPUObjectBase 的展示成员 和 GPUCommandsMixin 成员 在同一对象上存在。它必须也仅仅只被 包含这些混合器的接口 所包含。

GPUDebugCommandsMixin 为包含它的接口添加了以下内部插槽：

[[debug_group_stack]]，类型为 stack<USVString>。
活动调试组标签的堆栈。

GPUDebugCommandsMixin 为包含它的接口添加了以下方法：

---

GPUDebugCommandsMixin assumes the presence of GPUObjectBase and GPUCommandsMixin members on the same object. It must only be included by interfaces which also include those mixins.

GPUDebugCommandsMixin adds the following internal slots to interfaces which include it:

**[[debug_group_stack]], of type stack<USVString>**   
A stack of active debug group labels.

GPUDebugCommandsMixin adds the following methods to interfaces which include it:

---
---

**pushDebugGroup(groupLabel)**  

在调试栈中 压入一个有标签的调试组，且也包含后续命令。

调用于：GPUDebugCommandsMixin 实例自身

参数：

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| `groupLabel` | `USVString` | ✘ | ✘ | 命令组的标签。 |
 
返回：undefined

**上下文时间线步骤：**
1. 在this.[[device]]的设备时间线上发起后续步骤。

**设备时间线步骤：**    
1. 验证实例上编码器的状态，如果它返回 false，则停止。
2. 将 groupLabel 压入到 this.[[debug_group_stack]]。

---

**pushDebugGroup(groupLabel)**

Begins a labeled debug group containing subsequent commands.

**Called on:** GPUDebugCommandsMixin this.

**Arguments:**

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| `groupLabel` | `USVString` | ✘ | ✘ | The label for the command group. |

**Returns:** undefined

**Content timeline steps:**       
1. Issue the subsequent steps on the Device timeline of this.[[device]].

**Device timeline steps:**    
1. Validate the encoder state of this. If it returns false, stop.
2. Push groupLabel onto this.[[debug_group_stack]].

---
---
popDebugGroup()

在调试栈中 弹出最近由 pushDebugGroup()压入的 有标签的调试组。

调用于：GPUDebugCommandsMixin 实例自身。

返回：undefined

**上下文时间线步骤：**    
1. 在 this.device 的设备时间轴上的发起后续步骤。

**设备时间线步骤：**    
1. 验证实例的编码器的状态，如果它返回false，则停止。
2. 如果以下任何要求没有得到满足，使之无效，并停止。
    * this.[[debug_group_stack]] 必须不是空的。
3. 从 this.[[debug_group_stack]] 上弹出一个条目。

---

**popDebugGroup()**   

Ends the labeled debug group most recently started by pushDebugGroup().

**Called on:** GPUDebugCommandsMixin this.

**Returns:** undefined

**Content timeline steps:**
1. Issue the subsequent steps on the Device timeline of this.[[device]].

**Device timeline steps:**
1. Validate the encoder state of this. If it returns false, stop.
2. If any of the following requirements are unmet, make this invalid, and stop.
    * this.[[debug_group_stack]] must not be empty.
3. Pop an entry off of this.[[debug_group_stack]].

---
---

insertDebugMarker(markerLabel)

在命令流中用一个标签标记一个点。

调用于：GPUDebugCommandsMixin 实例自身。

参数：

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| `markerLabel` | `USVString` | ✘ | ✘ | 要插入的标签。 |

返回： undefined

**上下文时间线步骤：**    
1. 在 this.device 的设备时间轴上的发起后续步骤。

**设备时间线步骤：**    
1. 验证实例的编码器的状态，如果它返回false，则停止。

---

**insertDebugMarker(markerLabel)**

Marks a point in a stream of commands with a label.

**Called on:** GPUDebugCommandsMixin this.

**Arguments:**

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| `markerLabel` | `USVString` | ✘ | ✘ | The label to insert. |

Returns: undefined

**Content timeline steps:**     
1. Issue the subsequent steps on the Device timeline of this.[[device]].

**Device timeline steps:**
1. Validate the encoder state of this. If it returns false, stop.

