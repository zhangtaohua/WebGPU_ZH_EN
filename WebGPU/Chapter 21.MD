WebGPU

W3C Working Draft, 6 April 2023

More details about this document

This version:

https://www.w3.org/TR/2023/WD-webgpu-20230406/

---
---
21.画布渲染

21.1.HTMLCanvasElement.getContext()

一个 GPUCanvasContext 对象是通过 HTMLCanvasElement 实例上的 getContext() 方法创建的，该方法将字符串 "webgpu "作为它的 contextType 参数。

要在画布（HTMLCanvasElement 或 OffscreenCanvas）上创建一个 'webgpu' 上下文：
1. 让 context 成为一个新的 GPUCanvasContext。
2. 将 context.canvas 设置为 canvas。
3. 替换上下文的绘图缓冲区。
4. 返回上下文(context)。

```
// 例子 27
// 从屏幕外的 HTMLCanvasElement 中获取一个 GPUCanvasContext：

const canvas = document.createElement('canvas');
const context = canvas.getContext('webgpu');
```

---

21\. Canvas Rendering

21.1. HTMLCanvasElement.getContext()

A GPUCanvasContext object is created via the getContext() method of an HTMLCanvasElement instance by passing the string literal 'webgpu' as its contextType argument.

To create a 'webgpu' context on a canvas (HTMLCanvasElement or OffscreenCanvas) canvas:
1. Let context be a new GPUCanvasContext.
2. Set context.canvas to canvas.
3. Replace the drawing buffer of context.
4. Return context.

```
// EXAMPLE 27
// Get a GPUCanvasContext from an offscreen HTMLCanvasElement:

const canvas = document.createElement('canvas');
const context = canvas.getContext('webgpu');
```

---
---

21.2. GPUCanvasContext
```
[Exposed=(Window, DedicatedWorker), SecureContext]
interface GPUCanvasContext {
    readonly attribute (HTMLCanvasElement or OffscreenCanvas) canvas;

    undefined configure(GPUCanvasConfiguration configuration);
    undefined unconfigure();

    GPUTexture getCurrentTexture();
};
```

---

21.2. GPUCanvasContext
```
[Exposed=(Window, DedicatedWorker), SecureContext]
interface GPUCanvasContext {
    readonly attribute (HTMLCanvasElement or OffscreenCanvas) canvas;

    undefined configure(GPUCanvasConfiguration configuration);
    undefined unconfigure();

    GPUTexture getCurrentTexture();
};
```

---
---

GPUCanvasContext具有以下原生属性(attribute)：

**canvas，类型为： （HTMLCanvasElement或OffscreenCanvas），只读**       
表示这个上下文是由哪个画布创建而来的。

GPUCanvasContext有以下的内部插槽：

**[[configuration]], 类型为: GPUCanvasConfiguration?, 最初为 null**         
此上下文当前配置的选项。

如果该上下文没有被配置或已经是未配置的，则为 null。

**[[textureDescriptor]]，类型为：GPUTextureDescriptor，最初为 null。**
表示当前配置的纹理描述符，从 [[configuration]] 和 canvas 中导出。

如果该上下文没有被配置或已经是未配置的，则为 null。

**[[drawingBuffer]]，一个图像，最初是一个透明的黑色图像，大小与画布相同。**
绘图缓冲区是画布的工作拷贝图像数据。它是由 [[currentTexture]]（由getCurrentTexture()返回）暴露出来的，并且是可以写入的。

绘图缓冲区被用来获取一个上下文的图像内容的副本，这在画布被显示或以其他方式被读取时发生。它可能是透明的，即使[[configuration]].alphaMode 是 "不透明的"(opaque)。alphaMode 配置项只影响 "获取一个上下文的图像内容的副本 "的算法的结果。

绘图缓冲区比 [[currentTexture]] 的寿命长，即使在画布被呈现后，它也包含先前渲染的内容。它只有在替换绘图缓冲区时才会被清除。

在任何时候读取绘图缓冲区，具体的实现必须确保所有先前提交的工作（例如队列提交）已经通过 [[currentTexture]] 完成对它的写入操作。

---

GPUCanvasContext has the following attributes:

**canvas, of type (HTMLCanvasElement or OffscreenCanvas), readonly**      
The canvas this context was created from.

GPUCanvasContext has the following internal slots:

**[[configuration]], of type GPUCanvasConfiguration?, initially null**      
The options this context is currently configured with.

null if the context has not been configured or has been unconfigured.

**[[textureDescriptor]], of type GPUTextureDescriptor?, initially null**    
The currently configured texture descriptor, derived from the [[configuration]] and canvas.

null if the context has not been configured or has been unconfigured.

**[[drawingBuffer]], an image, initially a transparent black image with the same size as the canvas**     
The drawing buffer is the working-copy image data of the canvas. It is exposed as writable by [[currentTexture]] (returned by getCurrentTexture()).

The drawing buffer is used to get a copy of the image contents of a context, which occurs when the canvas is displayed or otherwise read. It may be transparent, even if [[configuration]].alphaMode is "opaque". The alphaMode only affects the result of the "get a copy of the image contents of a context" algorithm.

The drawing buffer outlives the [[currentTexture]] and contains the previously-rendered contents even after the canvas has been presented. It is only cleared in Replace the drawing buffer.

Any time the drawing buffer is read, implementations must ensure that all previously submitted work (e.g. queue submissions) have completed writing to it via [[currentTexture]].

---
---

**[[currentTexture]], 类型为： GPUTexture?, 最初为 null**

在当前帧中绘制的 GPUTexture。如果为空，在其底层填充到这个插槽上的 [[drawingBuffer]]. getCurrentTexture() 会暴露一个可写的视图，然后将其返回。

在一个可见画布的稳定状态下，在更新 WebGPU 画布的渲染时，绘图缓冲区的任何更改都会通过 currentTexture 来呈现出来。在这一时刻或之前，纹理也会被销毁，并且 [[currentTexture]] 被设置为 null，表示将通过下次调用 getCurrentTexture() 来创建一个新的纹理。

销毁 currentTexture 对绘图缓冲区的内容没有影响；它只是提前终止了对绘图缓冲区的写入访问。在同一帧中，getCurrentTexture() 会继续返回同一个被销毁的纹理。

为了使当前 texture 失效，将 currentTexture 设置为 null。 currentTexture 也被 configure() 函数调用、用于调整画布大小、演示、transferToImageBitmap()等调用。

---

**[[currentTexture]], of type GPUTexture?, initially null**   
The GPUTexture to draw into for the current frame. It exposes a writable view onto the underlying [[drawingBuffer]]. getCurrentTexture() populates this slot if null, then returns it.

In the steady-state of a visible canvas, any changes to the drawing buffer made through the currentTexture get presented when updating the rendering of a WebGPU canvas. At or before that point, the texture is also destroyed and [[currentTexture]] is set to to null, signalling that a new one is to be created by the next call to getCurrentTexture().

Destroying the currentTexture has no effect on the drawing buffer contents; it only terminates write-access to the drawing buffer early. During the same frame, getCurrentTexture() continues returning the same destroyed texture.

Expire the current texture sets the currentTexture to null. It is called by configure(), resizing the canvas, presentation, transferToImageBitmap(), and others.

---
---

GPUCanvasContext 有以下方法：

**configure(configuration)**     

为这个画布配置其上下文。这将把绘图缓冲区清除为透明的黑色（通过替换绘图缓冲区数据的方式）。

**调用于：** GPUCanvasContext 实例自身。

**参数：**

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| configuration | GPUCanvasConfiguration | ✘ | ✘ | 上下文的预期配置项。 |

**返回值：** undefined

---

GPUCanvasContext has the following methods:

**configure(configuration)**    

Configures the context for this canvas. This clears the drawing buffer to transparent black (in Replace the drawing buffer).

**Called on:** GPUCanvasContext this.
**Arguments:**

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| configuration | GPUCanvasConfiguration | ✘ | ✘ | Desired configuration for the context. |

**Returns:** undefined

---
---

**上下文时间轴步骤：**
1. 让 device 是 configuration.device 。
2. ?用 device.[[device]] 验证 `configuration.format 所要求的特征的纹理格式`。
3. ?用 device.[[device]] 验证 `configuration.viewFormats 中每个元素所要求的特征的纹理格`。
4. 让 descriptor 成为 canvas 和 `configuration(this.canvas, configuration) 函数的` GPUTextureDescriptor
5. 将 this.[[configuration]] 设置为 configuration。
6. 将 this.[[textureDescriptor]] 设置为 descriptor。
7. 用具有新格式和标签的位图 替换 this.[[drawingBuffer]] 的绘图缓冲区中数据，这将重新设置 this.[[drawingBuffer]]。
8. 在设备的设备时间轴上发起后续步骤。

---

**Content timeline steps:**
1. Let device be configuration.device.
2. ? Validate texture format required features of configuration.format with device.[[device]].
3. ? Validate texture format required features of each element of configuration.viewFormats with device.[[device]].
4. Let descriptor be the GPUTextureDescriptor for the canvas and configuration(this.canvas, configuration).
5. Set this.[[configuration]] to configuration.
6. Set this.[[textureDescriptor]] to descriptor.
7. Replace the drawing buffer of this, which resets this.[[drawingBuffer]] with a bitmap with the new format and tags.
8. Issue the subsequent steps on the Device timeline of device.

---
---

**设备时间轴步骤：**
1. 如果以下任何要求未能得到满足，则产生一个验证错误并停止。
    * 验证 `GPUTextureDescriptor(device, descriptor)`必须返回 true。
    * 支持的上下文格式中 必须包含配置项所配置的格式。           
   **注意：** 这种早期验证在下一次 configure() 调用之前一直有效，但对尺寸的验证除外，这是因为尺寸会在画布调整大小时发生变化。

---

**Device timeline steps:**
1. If any of the following requirements are unmet, generate a validation error and stop.
    * validating GPUTextureDescriptor(device, descriptor) must return true.
    * Supported context formats must contain configuration.format.          
    **NOTE:** This early validation remains valid until the next configure() call, except for validation of the size, which changes when the canvas is resized.

---
---

**unconfigure()**

移除上下文配置。销毁配置时所产生的任何纹理。

**调用于：** GPUCanvasContext 实例自身。

**返回值：** undefined

**上下文时间线步骤：**
1. 将 this.[[configuration]] 设置为 null。
2. 将 this.[[textureDescriptor]] 设置为 null。
3. 替换这个的绘图缓冲区的数据。

---

**unconfigure()**

Removes the context configuration. Destroys any textures produced while configured.

**Called on:** GPUCanvasContext this.

**Returns:** undefined

**Content timeline steps:**
1. Set this.[[configuration]] to null.
2. Set this.[[textureDescriptor]] to null.
3. Replace the drawing buffer of this.

---
---

**getCurrentTexture()**

获取将由 GPUCanvasContext 接下来合成到文档中的 GPUTexture。

**注意：** 应用程序应该在渲染到画布纹理的同一任务中调用 getCurrentTexture()。否则，纹理可能会在应用程序完成渲染之前就被这些步骤销毁了。         
过期任务（定义见下文）是可以选择实现的。即使实现了，任务源的优先级也没有规范性的定义，所以可能早到下一个任务，也可能晚到所有其他任务源都空了之后才发生（见自动到期任务源）。只有在可见画布显示时（即更新 WebGPU 画布的渲染）和替换绘图缓冲区数据的其他函数调用中，才保证任务会过期。

**调用于：**GPUCanvasContext 实例自身。

**返回值：** GPUTexture

---

**getCurrentTexture()**

Get the GPUTexture that will be composited to the document by the GPUCanvasContext next.

**Note:** An application should call getCurrentTexture() in the same task that renders to the canvas texture. Otherwise, the texture could get destroyed by these steps before the application is finished rendering to it.     

The expiry task (defined below) is optional to implement. Even if implemented, task source priority is not normatively defined, so may happen as early as the next task, or as late as after all other task sources are empty (see automatic expiry task source). Expiry is only guaranteed when a visible canvas is displayed (updating the rendering of a WebGPU canvas) and in other callers of Replace the drawing buffer.

**Called on:** GPUCanvasContext this.

**Returns:** GPUTexture

---
---

**上下文时间轴步骤：**  
1. 如果 this.[[configuration]]是 null：
    * 抛出一个 InvalidStateError 错误并停止。
2. 断言 this.[[textureDescriptor]] 不是 null。
3. 让设备(device)成为 this.[[configuration]].device。
4. 如果 this.[[currentTexture]] 为 null：
    1. 替换实例 this 的绘图缓冲区。
    2. 将 this.[[currentTexture]] 设置为用实例 this.[textureDescriptor]] 调用 device.createTexture()的结果，只是需要 GPUTexture 的底层存储指向 this.[[drawingBuffer]]。     
    **注意：** 如果纹理不能被创建（例如，由于验证失败或内存不足），这将产生一个错误并返回一个无效的 GPUTexture。这里的一些验证与 configure()中的验证是重复多余的。但实现不能跳过这些多余的验证。
5. 可选地进行，用实例设备的 device 和以下步骤将一个自动过期任务排入队列：
    1. 将实例 this 的当前纹理 过期处理。        
    **注意：** 如果在更新 WebGPU 画布的渲染时已经发生了这种情况，那么它就没有效果。
6. 返回 this.[[currentTexture]]。

**注意：** 每次调用 getCurrentTexture() 都会返回同一个 GPUTexture 对象，直到过期当前纹理流程（"Expire the current texture"）运行为止，即使该 GPUTexture 被销毁、验证失败或分配内存失败。

---

**Content timeline steps:**
1. If this.[[configuration]] is null:
    1. Throw an InvalidStateError and stop.
2. Assert this.[[textureDescriptor]] is not null.
3. Let device be this.[[configuration]].device.
4. If this.[[currentTexture]] is null:
    1. Replace the drawing buffer of this.
    2. Set this.[[currentTexture]] to the result of calling device.createTexture() with this.[[textureDescriptor]], except with the GPUTexture's underlying storage pointing to this.[[drawingBuffer]].         
    **NOTE:** If the texture can’t be created (e.g. due to validation failure or out-of-memory), this generates and error and returns an invalid GPUTexture. Some validation here is redundant with that done in configure(). Implementations must not skip this redundant validation.
5. Optionally, queue an automatic expiry task with device device and the following steps:
    1. Expire the current texture of this.          
    NOTE: If this already happened when updating the rendering of a WebGPU canvas, it has no effect.
6. Return this.[[currentTexture]].

**NOTE:** The same GPUTexture object will be returned by every call to getCurrentTexture() until "Expire the current texture" runs, even if that GPUTexture is destroyed, failed validation, or failed to allocate.

---
---

`获取一个上下文的图像内容的副本：`

**参数：**
* context: 即 GPUCanvasContext

**返回值：** 图像内容。

1. 确保所有提交的工作项目（如队列提交）通过 context.[[currentTexture]] 都已完成对图像的写入。
2. 让 context.[[drawingBuffer]] 的一个副本 成为一个快照。
3. 让 alphaMode 成为 context.[[configuration]].alphaMode。
4. 如果 alphaMode 是 "不透明的"：
    1. 将快照的 alpha 通道清除为 1.0。
    2. 标记第二步的快照为不透明。           
    **注意：** 如果 [[currentTexture]]，如果有的话，已经被销毁了（例如在替换绘图缓冲区时），则 alpha 通道是不可观察的，并且具体的实现可以就地清除 alpha 通道。
    **否则：**
    用 alphaMode 标记快照。
5. 返回快照。

---

To get a copy of the image contents of a context:

**Arguments:**
* context: the GPUCanvasContext

**Returns:** image contents

1. Ensure that all submitted work items (e.g. queue submissions) have completed writing to the image (via context.[[currentTexture]]).
2. Let snapshot be a copy of context.[[drawingBuffer]].
3. Let alphaMode be context.[[configuration]].alphaMode.
4. If alphaMode is "opaque":
    1. Clear the alpha channel of snapshot to 1.0.
    2. Tag snapshot as being opaque.            
    **NOTE:** If the [[currentTexture]], if any, has been destroyed (for example in Replace the drawing buffer), the alpha channel is unobservable, and implementations may clear the alpha channel in-place.           
    **Otherwise:**          
    Tag snapshot with alphaMode.        
5. Return snapshot.

---
---

`替换 GPUCanvasContext 上下文的绘图缓冲区：`

1. 终止上下文的当前纹理。
2. 让 configuration 成为context.[[configuration]]。
3. 将 context.[[drawingBuffer]] 设置为与 context.canvas 相同大小的透明黑色图像。
    * 如果配置为空，则绘图缓冲区被标记为颜色空间是 "srgb"。在这种情况下，绘图缓冲区将保持空白，直到上下文被配置好。
    * 如果配置不是空的，绘图缓冲区就会被指定为 configuration.format，并被标记为指定的configuration.colorSpace。     
    **注意**：configuration.alphaMode 在 `"获取一个上下文的图像内容的副本"` 之前被忽略。            
    **注意：** 如果绘图缓冲区已经被清空并且拥有正确的配置，这通常是一个无用的操作。

---

To Replace the drawing buffer of a GPUCanvasContext context:

1. Expire the current texture of context.
2. Let configuration be context.[[configuration]].
3. Set context.[[drawingBuffer]] to a transparent black image of the same size as context.canvas.
    * If configuration is null, the drawing buffer is tagged with the color space "srgb". In this case, the drawing buffer will remain blank until the context is configured.
    * If not, the drawing buffer has the specified configuration.format and is tagged with the specified configuration.colorSpace.

    **NOTE:** configuration.alphaMode is ignored until "get a copy of the image contents of a context".

    **NOTE:** This will often be a no-op, if the drawing buffer is already cleared and has the correct configuration.

---
---

`使 GPUCanvasContext 上下文的当前纹理失效：`
1. 如果 context.[[currentTexture]] 不是空的：
    1. 调用 context.[[currentTexture]].destroy() 以终止对图像的写入访问。（这里不会销毁 context.[[drawingBuffer]] ）。
    2. 将 context.[[currentTexture]] 设置为 null。

---

To Expire the current texture of a GPUCanvasContext context:
1. If context.[[currentTexture]] is not null:
    1. Call context.[[currentTexture]].destroy() (without destroying context.[[drawingBuffer]]) to terminate write access to the image.
    2. Set context.[[currentTexture]] to null.

---
---

## 21.3. HTML 钩子函数

以下算法与HTML规范中的算法 "挂钩"，并且必须在指定的时间点上运行。

当从具有 GPUCanvasContext 上下文的 HTMLCanvasElement 或 OffscreenCanvas 中读取 "位图"时：
1. 返回上下文的图像内容的一个副本。

**注意：** 这会发生在很多地方，包括：
* 当一个 HTMLCanvasElement 的渲染被更新时。
* 当一个有占位符画布元素的 OffscreenCanvas 的渲染被更新时。
* 当 transferToImageBitmap() 从位图创建一个 ImageBitmap 时。        
* 当 WebGPU 画布内容被其他网络API读取时，如drawImage(), texImage2D(), texSubImage2D(), toDataURL(), toBlob()等等。      
如果 alphaMode 是 "不透明"，这就将会对 alpha 通道的进行清空。当具体的实现能够以忽略 alpha 通道的方式读取或显示图像时，可以跳过这个步骤。

如果一个应用程序只需要一个画布来进行内部交互操作（而不是用于显示），如果不需要的话，就避免使用 "不透明"。

---

21.3. HTML Specification Hooks

The following algorithms "hook" into algorithms in the HTML specification, and must run at the specified points.

When the "bitmap" is read from an HTMLCanvasElement or OffscreenCanvas with a GPUCanvasContext context:
1. Return a copy of the image contents of context.

**Note:** This occurs in many places, including:
* When an HTMLCanvasElement has its rendering updated.
* When an OffscreenCanvas with a placeholder canvas element has its rendering updated.
* When transferToImageBitmap() creates an ImageBitmap from the bitmap.
* When WebGPU canvas contents are read using other Web APIs, like drawImage(), texImage2D(), texSubImage2D(), toDataURL(), toBlob(), and so on.

If alphaMode is "opaque", this incurs a clear of the alpha channel. Implementations may skip this step when they are able to read or display images in a way that ignores the alpha channel.

If an application needs a canvas only for interop (not presentation), avoid "opaque" if it is not needed.

---
---

当用 GPUCanvasContext 上下文 `更新 WebGPU画布（一个HTMLCanvasElement，或者带有占位符画布元素的OffscreenCanvas）的渲染时`，这将会在事件循环处理模型中触发以下子步骤中：
* "更新该 Document 的渲染或用户界面"
* "更新该专门工作者(worker)的渲染"

运行以下步骤：
1. 使当前上下文的当前纹理失效。   
    **注意：** 如果这已经在 getCurrentTexture() 排队任务中发生了，那么它就没有效果。

**注意：** 这不会发生在独立的 OffscreenCanvas 上（即由 new OffscreenCanvas() 创建的）。

---

When updating the rendering of a WebGPU canvas (an HTMLCanvasElement or an OffscreenCanvas with a placeholder canvas element) with a GPUCanvasContext context, which occurs in the following sub-steps of the event loop processing model:
* "update the rendering or user interface of that Document"
* "update the rendering of that dedicated worker"

**Run the following steps:**        
1. Expire the current texture of context.
    **NOTE:** If this already happened in the task queued by getCurrentTexture(), it has no effect.

**NOTE:** This does not happen for standalone OffscreenCanvases (created by new OffscreenCanvas()).

---
---

当在具有 GPUCanvasContext 上下文的画布上调用transferToImageBitmap()时，在从画布的位图创建 ImageBitmap 后：
1. 替换上下文的绘图缓冲区。

**注意：** 这相当于将（可能已清除 alpha 的）图像内容 "移动"到 ImageBitmap 中，而不需要复制。

---

When transferToImageBitmap() is called on a canvas with GPUCanvasContext context, after creating an ImageBitmap from the canvas’s bitmap:
1. Replace the drawing buffer of context.

**NOTE:** This is equivalent to "moving" the (possibly alpha-cleared) image contents into the ImageBitmap, without a copy.

---
---

## 21.4. GPUCanvasConfiguration

所支持的上下文格式是一组 GPUTextureFormats，当被指定为 GPUCanvasConfiguration.format时，无论给定的 GPUCanvasConfiguration.device 如何，都必须支持这些格式，最初设置为：`""bgra8unorm", "rgba8unorm", "rgba16float""` 。

**注意：** 画布配置不能使用像 "bgra8unorm-srgb" 等 srgb格式。相反，使用 非srgb格式相等操作（"bgra8unorm"），在 viewFormats 中指定 srgb格式，并使用 createView() 来创建一个具有 srgb 格式的视图。

```
enum GPUCanvasAlphaMode {
    "opaque",
    "premultiplied"
};

dictionary GPUCanvasConfiguration {
    required GPUDevice device;
    required GPUTextureFormat format;
    GPUTextureUsageFlags usage = 0x10;  // GPUTextureUsage.RENDER_ATTACHMENT
    sequence<GPUTextureFormat> viewFormats = [];
    PredefinedColorSpace colorSpace = "srgb";
    GPUCanvasAlphaMode alphaMode = "opaque";
};
```

---

21.4. GPUCanvasConfiguration

The supported context formats are a set of GPUTextureFormats that must be supported when specified as a GPUCanvasConfiguration.format regardless of the given GPUCanvasConfiguration.device, initially set to: «"bgra8unorm", "rgba8unorm", "rgba16float"».

**NOTE:** Canvas configuration cannot use srgb formats like "bgra8unorm-srgb". Instead, use the non-srgb equivalent ("bgra8unorm"), specify the srgb format in the viewFormats, and use createView() to create a view with an srgb format.

```
enum GPUCanvasAlphaMode {
    "opaque",
    "premultiplied"
};

dictionary GPUCanvasConfiguration {
    required GPUDevice device;
    required GPUTextureFormat format;
    GPUTextureUsageFlags usage = 0x10;  // GPUTextureUsage.RENDER_ATTACHMENT
    sequence<GPUTextureFormat> viewFormats = [];
    PredefinedColorSpace colorSpace = "srgb";
    GPUCanvasAlphaMode alphaMode = "opaque";
};
```

---
---

GPUCanvasConfiguration有 以下成员：

**device，类型为： GPUDevice**        
由 getCurrentTexture() 返回与其纹理相兼容的 GPUDevice。

**format，类型为：GPUTextureFormat**        
由 getCurrentTexture() 返回的纹理的格式。必须是所支持的上下文格式之一。

**usage，类型为： GPUTextureUsageFlags，默认为 0x10**           
由 getCurrentTexture() 返回的纹理所具有的用途，用途 RENDER_ATTACHMENT 是默认的具备的，但如果明确设置了相应的用法，则不会自动包含此默认用法。如果你希望使用 getCurrentTexture() 返回的纹理作为渲染通道的颜色目标，请确保在设置自定义用途时包含 RENDER_ATTACHMENT 用法。

**viewFormats，类型为： sequence<GPUTextureFormat>，默认为 []**         
从 getCurrentTexture() 返回的纹理而创建的 视图 可以使用的格式集。

**colorSpace，类型为： PredefinedColorSpace，默认为 "srgb"**
写入 getCurrentTexture() 返回的纹理中的颜色空间，它们将被用来显示。

**alphaMode，类型为： GPUCanvasAlphaMode，默认为 "opaque" 即不透明**
决定 alpha 值在读取、显示或作为图像源使用时， 对 getCurrentTexture() 所返回的纹理内容的影响。

---

GPUCanvasConfiguration has the following members:

**device, of type GPUDevice**           
The GPUDevice that textures returned by getCurrentTexture() will be compatible with.

**format, of type GPUTextureFormat**            
The format that textures returned by getCurrentTexture() will have. Must be one of the Supported context formats.

**usage, of type GPUTextureUsageFlags, defaulting to 0x10**     
The usage that textures returned by getCurrentTexture() will have. RENDER_ATTACHMENT is the default, but is not automatically included if the usage is explicitly set. Be sure to include RENDER_ATTACHMENT when setting a custom usage if you wish to use textures returned by getCurrentTexture() as color targets for a render pass.

**viewFormats, of type sequence<GPUTextureFormat>, defaulting to []**           
The formats that views created from textures returned by getCurrentTexture() may use.

**colorSpace, of type PredefinedColorSpace, defaulting to "srgb"**          
The color space that values written into textures returned by getCurrentTexture() should be displayed with.

**alphaMode, of type GPUCanvasAlphaMode, defaulting to "opaque"**       
Determines the effect that alpha values will have on the content of textures returned by getCurrentTexture() when read, displayed, or used as an image source.

---
---

```
// 例子 28
// 配置一个 GPUCanvasContext, 使之与特定的 GPUDevice 一起使用，并使用喜欢的格式应用到该上下文上：
// EXAMPLE 28
// Configure a GPUCanvasContext to be used with a specific GPUDevice, using the preferred format for this context:

const canvas = document.createElement('canvas');
const context = canvas.getContext('webgpu');

context.configure({
    device: gpuDevice,
    format: navigator.gpu.getPreferredCanvasFormat(),
});
```

---

```
EXAMPLE 28
Configure a GPUCanvasContext to be used with a specific GPUDevice, using the preferred format for this context:

const canvas = document.createElement('canvas');
const context = canvas.getContext('webgpu');

context.configure({
    device: gpuDevice,
    format: navigator.gpu.getPreferredCanvasFormat(),
});
```

---
---

`画布和配置（（HTMLCanvasElement或OffscreenCanvas）的画布，GPUCanvasConfiguration 的配置）的GPUTextureDescriptor` 是一个 GPUTextureDescriptor，且有以下成员：
* size: [canvas.width, canvas.height, 1].
* format：配置.格式。
* usage: 配置.用法。
* viewFormats：配置.视图格式集。

以及其他设置为默认值的成员。

canvas.width 是指 HTMLCanvasElement.width 或 OffscreenCanvas.width。canvas.height 是指 HTMLCanvasElement.height 或 OffscreenCanvas.height。

---

The GPUTextureDescriptor for the canvas and configuration( (HTMLCanvasElement or OffscreenCanvas) canvas, GPUCanvasConfiguration configuration) is a GPUTextureDescriptor with the following members:
* size: [canvas.width, canvas.height, 1].
* format: configuration.format.
* usage: configuration.usage.
* viewFormats: configuration.viewFormats.

and other members set to their defaults.

canvas.width refers to HTMLCanvasElement.width or OffscreenCanvas.width. canvas.height refers to HTMLCanvasElement.height or OffscreenCanvas.height.

---
---

### 21.4.1. 画布色彩空间

在显示过程中，在 [0, 1] 范围以外的颜色值的色度不应钳制在该范围内；在配置的格式和用户的显示能力允许的情况下，可以使用扩展值来显示画布颜色空间的主色域以外的颜色。这与亮度相反，亮度应该被钳制在最大标准动态范围内的亮度。

---

21.4.1. Canvas Color Space

During presentation, the chrominance of color values outside of the [0, 1] range is not to be clamped to that range; extended values may be used to display colors outside of the gamut defined by the canvas' color space’s primaries, when permitted by the configured format and the user’s display capabilities. This is in contrast with luminance, which is to be clamped to the maximum standard dynamic range luminance.

---
---

21.4.2.画布上下文的尺寸
所有的画布配置都在 configure() 中进行设置，除了画布的分辨率，它是由画布的宽度和高度设置的。

**注意：** 与 WebGL 和 2D 画布一样，调整 WebGPU 画布的大小会丢失绘图缓冲区的当前内容。在 WebGPU 中，它是通过替换绘图缓冲区来实现的。

当具有 GPUCanvasContext 上下文的 HTMLCanvasElement 或 OffscreenCanvas 画布的宽度或高度属性被修改时，更新画布的大小：
1. 替换上下文的绘图缓冲区。
2. 让配置 configuration 成为  context.[[configuration]]
3. 如果配置不是空的：
    1. 将 context.[[textureDescriptor]] 设置为 `画布和配置的 GPUTextureDescriptor（canvas，configuration）`。

**注意：** 这可能导致 GPUTextureDescriptor 超过设备的 maxTextureDimension2D。在这种情况下，`getCurrentTexture()` 中的验证操作将会失败。

---

21.4.2. Canvas Context sizing

All canvas configuration is set in configure() except for the resolution of the canvas, which is set by the canvas’s width and height.

**NOTE:** Like WebGL and 2d canvas, resizing a WebGPU canvas loses the current contents of the drawing buffer. In WebGPU, it does so by replacing the drawing buffer.

When an HTMLCanvasElement or OffscreenCanvas canvas with a GPUCanvasContext context has its width or height properties modified, update the canvas size:
1. Replace the drawing buffer of context.
2. Let configuration be context.[[configuration]]
3. If configuration is not null:
    1. Set context.[[textureDescriptor]] to the GPUTextureDescriptor for the canvas and configuration(canvas, configuration).

**NOTE:** This may result in a GPUTextureDescriptor which exceeds the maxTextureDimension2D of the device. In this case, validation will fail inside getCurrentTexture().

---
---

```
// 例子 29
// 重新配置 GPUCanvasContext 以便响应画布的调整，使用 ResizeObserver 进行监测，以便获得画布的确切像素尺寸：
// EXAMPLE 29
// Reconfigure a GPUCanvasContext in response to canvas resize, monitored using ResizeObserver to get the exact pixel dimensions of the canvas:

const canvas = document.createElement('canvas');
const context = canvas.getContext('webgpu');

const resizeObserver = new ResizeObserver(entries => {
    for (const entry of entries) {
        if (entry.target != canvas) { continue; }
        canvas.width = entry.devicePixelContentBoxSize[0].inlineSize;
        canvas.height = entry.devicePixelContentBoxSize[0].blockSize;
    }
});
resizeObserver.observe(canvas);
```

---

```
// EXAMPLE 29
// Reconfigure a GPUCanvasContext in response to canvas resize, monitored using ResizeObserver to get the exact pixel dimensions of the canvas:

const canvas = document.createElement('canvas');
const context = canvas.getContext('webgpu');

const resizeObserver = new ResizeObserver(entries => {
    for (const entry of entries) {
        if (entry.target != canvas) { continue; }
        canvas.width = entry.devicePixelContentBoxSize[0].inlineSize;
        canvas.height = entry.devicePixelContentBoxSize[0].blockSize;
    }
});
resizeObserver.observe(canvas);
```

---
---

## 21.5. GPUCanvasAlphaMode
 
这个枚举选项选择了，当画布的内容在读取时，或者显示在屏幕上时，或者作为图像源（输入参数为drawImage、输出参数为 toDataURL，等等）时，画布内容的解释方式。

下面，src 是画布纹理中的一个值，dst 是画布被合成的图像（例如，HTML 页面渲染，或 2D 画布）。

**"opaque"**        
* 读取 RGB 为不透明，并且忽略 alpha 值。如果内容还不是不透明的，在 `"获取一个上下文的图像内容的副本"`中，alpha 通道被清除为 1.0。

**"premultiplied"**     
* 读取 RGBA 进行预乘算法：颜色值被预乘以其 alpha 值。100% 的红色在 50% 的 alpha 值下的结果是 [0.5, 0, 0, 0.5]。
* 如果超出色域(out-of-gamut)预乘的 RGBA 值被输出到画布上，那么，画布是：
    * 作为一个图像源使用        
    值会被保留，如色彩空间转换中所述的那样。
    * 显示在屏幕上      
    合成的结果是未定义的。即使色彩空间转换算法会在合成前产生在色域范围内的值(in-gamut)，这也是真的，因为本规范没有指定进行合成的中间格式。

---

21.5. GPUCanvasAlphaMode

This enum selects how the contents of the canvas will be interpreted when read, when displayed to the screen or used as an image source (in drawImage, toDataURL, etc.)

Below, src is a value in the canvas texture, and dst is an image that the canvas is being composited into (e.g. an HTML page rendering, or a 2D canvas).

**"opaque"**        
Read RGB as opaque and ignore alpha values. If the content is not already opaque, the alpha channel is cleared to 1.0 in "get a copy of the image contents of a context".

**"premultiplied"**         
Read RGBA as premultiplied: color values are premultiplied by their alpha value. 100% red at 50% alpha is [0.5, 0, 0, 0.5].

If out-of-gamut premultiplied RGBA values are output to the canvas, and the canvas is:

**used as an image source**     
Values are preserved, as described in color space conversion.

**displayed to the screen**     
Compositing results are undefined. This is true even if color space conversion would produce in-gamut values before compositing, because the intermediate format for compositing is not specified.
