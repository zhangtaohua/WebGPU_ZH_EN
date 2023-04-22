WebGPU

W3C Working Draft, 6 April 2023 

More details about this document

This version:

https://www.w3.org/TR/2023/WD-webgpu-20230406/

---
---

# 18\. 绑定

## 18.1. GPURenderBundle

```
[Exposed=(Window, DedicatedWorker), SecureContext]
interface GPURenderBundle {
};
GPURenderBundle includes GPUObjectBase;
```

**[[command_list]], 类型为： list\<GPU command\>**      
当 GPURenderBundle 被执行时，提交给 GPURenderPassEncoder 的 GPU命令的列表。

**[[layout]], 类型为： GPURenderPassLayout**    
渲染绑定包的布局。

**[[depthReadOnly]], 类型为： boolean**     
如果为true，表示执行此渲染绑定包时不会修改深度组件。

[[stencilReadOnly]]，类型为布尔值
如果为true，表示执行此渲染绑定包时不会修改模板组件。

[[drawCount]], of type GPUSize64
此 GPURenderBundle 中绘制命令的数量。

---

18\. Bundles

18.1. GPURenderBundle

```
[Exposed=(Window, DedicatedWorker), SecureContext]
interface GPURenderBundle {
};
GPURenderBundle includes GPUObjectBase;
```

**[[command_list]], of type list\<GPU command\>**       
A list of GPU commands to be submitted to the GPURenderPassEncoder when the GPURenderBundle is executed.

**[[layout]], of type GPURenderPassLayout**     
The layout of the render bundle.

**[[depthReadOnly]], of type boolean**      
If true, indicates that the depth component is not modified by executing this render bundle.

**[[stencilReadOnly]], of type boolean**        
If true, indicates that the stencil component is not modified by executing this render bundle.

**[[drawCount]], of type GPUSize64**        
The number of draw commands in this GPURenderBundle.

---
---

### 18.1.1. 渲染包的创建

```
dictionary GPURenderBundleDescriptor : GPUObjectDescriptorBase {
};
```
```
[Exposed=(Window, DedicatedWorker), SecureContext]
interface GPURenderBundleEncoder {
    GPURenderBundle finish(optional GPURenderBundleDescriptor descriptor = {});
};
GPURenderBundleEncoder includes GPUObjectBase;
GPURenderBundleEncoder includes GPUCommandsMixin;
GPURenderBundleEncoder includes GPUDebugCommandsMixin;
GPURenderBundleEncoder includes GPUBindingCommandsMixin;
GPURenderBundleEncoder includes GPURenderCommandsMixin;
```

---

18.1.1. Render Bundle Creation

```
dictionary GPURenderBundleDescriptor : GPUObjectDescriptorBase {
};
```
```
[Exposed=(Window, DedicatedWorker), SecureContext]
interface GPURenderBundleEncoder {
    GPURenderBundle finish(optional GPURenderBundleDescriptor descriptor = {});
};
GPURenderBundleEncoder includes GPUObjectBase;
GPURenderBundleEncoder includes GPUCommandsMixin;
GPURenderBundleEncoder includes GPUDebugCommandsMixin;
GPURenderBundleEncoder includes GPUBindingCommandsMixin;
GPURenderBundleEncoder includes GPURenderCommandsMixin;
```

---
---

**createRenderBundleEncoder(descriptor)**

创建一个 GPURenderBundleEncoder。

**调用于：** GPUDevice 实例自身。

**参数：**

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| descriptor | GPURenderBundleEncoderDescriptor | ✘ | ✘ | 要创建的GPURenderBundleEncoder的描述符。 |

**返回值：** GPURenderBundleEncoder

---

createRenderBundleEncoder(descriptor)

Creates a GPURenderBundleEncoder.

**Called on:** GPUDevice this.

**Arguments:**

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| descriptor | GPURenderBundleEncoderDescriptor | ✘ | ✘ | Description of the GPURenderBundleEncoder to create. |

Returns: GPURenderBundleEncoder

---
---

**上下文时间轴步骤：**
1. ?用 this.[[device]] 验证 descriptor.colorFormats 中的每个非空元素的纹理格式要求特征。
2. ?用 this.[[device]] 验证 descriptor.depthStencilFormat 的纹理格式要求特征。
3. 让 e 成为一个新的 GPURenderBundleEncoder 对象。
4. 在实例 this 的设备时间轴上发起初始化步骤。
5. 返回 e。

---

**Content timeline steps:**
1. ? Validate texture format required features of each non-null element of descriptor.colorFormats with this.[[device]].
2. ? Validate texture format required features of descriptor.depthStencilFormat with this.[[device]].
3. Let e be a new GPURenderBundleEncoder object.
4. Issue the initialization steps on the Device timeline of this.
5. Return e.

---
---

**设备时间轴初始化步骤：** 
1. 如果以下任何条件不满足，产生一个验证错误，使 e 无效，并停止。
    * 实例 this 是有效的。
    * descriptor.colorFormats.length 必须小于等于 this.[[limit]].maxColorAttachments。
    * 对于 descriptor.colorFormats.length 中的每个非空的 colorFormat：
        * colorFormat必须是一个可渲染的颜色格式。
    * `计算每个样本的颜色附件字节数（descriptor.colorFormats`）必须 ≤ this.[[limit]].maxColorAttachmentBytesPerSample。
    * 如果提供了 descriptor.depthStencilFormat：
        * descriptor.depthStencilFormat 必须是深度或模板格式。
        * 如果 descriptor.depthStencilFormat 是一种深度/钢网的组合格式：
            * descriptor.depthReadOnly 必须等于 descriptor.stencilReadOnly。
    * 必须至少存在一个附件，要么：
        * 是 descriptor.colorFormats 中的一个非空值，或
        * 是一个 descriptor.depthStencilFormat。
2. 将 e.[[layout]] 设置为 descriptor 内含的 GPURenderPassLayout 接口的副本。
3. 将 e.[[depthReadOnly]] 设置为 descriptor.depthReadOnly。
4. 将 e.[[stencilReadOnly]] 设置为 descriptor.stencilReadOnly。
5. 将 e.[[state]] 设置为 "open"。
6. 将 e.[[drawCount]] 设置为 0。

>> **ISSUE 28** 描述 createRenderBundleEncoder() 的重置步骤。
 
---

**Device timeline initialization steps:**           
1. If any of the following conditions are unsatisfied generate a validation error, make e invalid, and stop.
    * this is valid.
    * descriptor.colorFormats.length must be ≤ this.[[limits]].maxColorAttachments.
    * For each non-null colorFormat in descriptor.colorFormats:
        * colorFormat must be a color renderable format.
    * Calculating color attachment bytes per sample(descriptor.colorFormats) must be ≤ this.[[limits]].maxColorAttachmentBytesPerSample.
    * If descriptor.depthStencilFormat is provided:
        * descriptor.depthStencilFormat must be a depth-or-stencil format.
        * If descriptor.depthStencilFormat is a combined depth-stencil format:
            * descriptor.depthReadOnly must be equal to descriptor.stencilReadOnly.
    * There must exist at least one attachment, either:
        * A non-null value in descriptor.colorFormats, or
        * A descriptor.depthStencilFormat.
2. Set e.[[layout]] to a copy of descriptor’s included GPURenderPassLayout interface.
3. Set e.[[depthReadOnly]] to descriptor.depthReadOnly.
4. Set e.[[stencilReadOnly]] to descriptor.stencilReadOnly.
5. Set e.[[state]] to "open".
6. Set e.[[drawCount]] to 0.

>> ISSUE 28 Describe the reset of the steps for createRenderBundleEncoder().

---
---

18.1.2. 编码

```
dictionary GPURenderBundleEncoderDescriptor : GPURenderPassLayout {
    boolean depthReadOnly = false;
    boolean stencilReadOnly = false;
};
```

---

18.1.2. Encoding

```
dictionary GPURenderBundleEncoderDescriptor : GPURenderPassLayout {
    boolean depthReadOnly = false;
    boolean stencilReadOnly = false;
};
```

---
---

18.1.3. 最终化

**finish(descriptor)**      

完成对渲染包命令序列的记录。

**调用于：** GPURenderBundleEncoder 实例自身。

**参数：**  

| Parameter | Type | Nullable | Optional | Description | 
| --- | --- | --- | --- | --- |
| descriptor | GPURenderBundleDescriptor | ✘ | ✔ |  | 

**返回值：** GPURenderBundle

---

18.1.3. Finalization

**finish(descriptor)**

Completes recording of the render bundle commands sequence.

**Called on:** GPURenderBundleEncoder this.

**Arguments:**

| Parameter | Type | Nullable | Optional | Description | 
| --- | --- | --- | --- | --- |
| descriptor | GPURenderBundleDescriptor | ✘ | ✔ |  | 

**Returns:** GPURenderBundle

---
---

**上下文时间线步骤：**      
1. 让 renderBundle 成为一个新的 GPURenderBundle。
2. 在 this.[[device]] 的设备时间线上发起完成步骤。
3. 返回 renderBundle。

---
**Content timeline steps:**
1. Let renderBundle be a new GPURenderBundle.
2. Issue the finish steps on the Device timeline of this.[[device]].
3. Return renderBundle.

---
---

**设备时间轴完成步骤：**
1. 如果满足以下所有要求，让 validationSucceeded 为 true，否则为 false。
    * 实例 this 必须是有效的。
    * this.[[state]] 必须是 "open"。
    * this.[[debug_group_stack]] 必须为空。
    * 实例 this 中包含的每个使用范围都必须满足使用范围验证的要求。
2. 将 this.[[state]] 设置为 "ended"。
3. 如果 validationSucceeded 为 false，那么：
    1. 产生一个验证错误。
    2. 返回一个新的无效的 GPURenderBundle。
4. 将 renderBundle.[[command_list]] 设置为 this.[[command]]。
5. 将 renderBundle.[[drawCount]] 设置为 this.[[drawCount]]。

---

**Device timeline finish steps:**
1. Let validationSucceeded be true if all of the following requirements are met, and false otherwise.
    * this must be valid.
    * this.[[state]] must be "open".
    * this.[[debug_group_stack]] must be empty.
    * Every usage scope contained in this must satisfy the usage scope validation.
2. Set this.[[state]] to "ended".
3. If validationSucceeded is false, then:
    1. Generate a validation error.
    2. Return a new invalid GPURenderBundle.
5. Set renderBundle.[[command_list]] to this.[[commands]].
5. Set renderBundle.[[drawCount]] to this.[[drawCount]].

