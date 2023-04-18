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

# 6\. 纹理和纹理视图

## 6.1. 纹理(GPUTexture)
 ***ISSUE 5 Remove this definition: texture***

> 6\. Textures and Texture Views    
> 6.1. GPUTexture    
> ***ISSUE 5 移除这部分定义： texture***    

一个纹理是由一个或多个纹理子资源组成，仅针对于2D纹理，包括 阵列层 和 长宽比，每个子资源都有由一个mipmap级别唯一标识。

一个纹理子资源就是一个这样的子资源：每个子资源可以在一个使用范围内 用于 不同的内部用法使用。

> One texture consists of one or more texture subresources, each uniquely identified by a mipmap level and, for 2d textures only, array layer and aspect.    
> A texture subresource is a subresource: each can be used in different internal usages within a single usage scope.    

一个 mipmap 级别中的每个子资源在每个空间维度上大约是较低级别中相应资源的一半大小（见逻辑 miplevel特定纹理范围）。0级的子资源有纹理本身的尺寸。这些通常是用来表示纹理的细节层次的。GPUSampler 和 WGSL 提供了 明确的或自动的方法来，选择和插值不同层级的细节。

> Each subresource in a mipmap level is approximately half the size, in each spatial dimension, of the corresponding resource in the lesser level (see logical miplevel-specific texture extent). The subresource in level 0 has the dimensions of the texture itself. These are typically used to represent levels of detail of a texture. GPUSampler and WGSL provide facilities for selecting and interpolating between levels of detail, explicitly or automatically.    

一个 "2d "纹理可以是一个阵列图层的阵列。一个层中的每个子资源与其他层中的相应资源大小相同。对于非2D纹理，所有子资源的阵列层索引都是0。

> A "2d" texture may be an array of array layers. Each subresource in a layer is the same size as the corresponding resources in other layers. For non-2d textures, all subresources have an array layer index of 0. 

每个子资源都有一个特殊的属性。颜色纹理只有一个特殊属性：颜色。深度及模式格式的纹理可能有多个特殊属性：深度方面、模式方面或两者都有，并可能以特殊方式使用，例如在 depthStencilAttachment 和 "depth"绑定 中使用。

一个 "3D" 纹理可以有多个切片，每个切片 是纹理中 某个特定Z轴 的二维图像。切片不是独立的子资源。

> Each subresource has an aspect. Color textures have just one aspect: color. Depth-or-stencil format textures may have multiple aspects: a depth aspect, a stencil aspect, or both, and may be used in special ways, such as in depthStencilAttachment and in "depth" bindings.    
> A "3d" texture may have multiple slices, each being the two-dimensional image at a particular z value in the texture. Slices are not separate subresources.    

```
[Exposed=(Window, DedicatedWorker), SecureContext]
interface GPUTexture {
    GPUTextureView createView(optional GPUTextureViewDescriptor descriptor = {});

    undefined destroy();

    readonly attribute GPUIntegerCoordinate width;
    readonly attribute GPUIntegerCoordinate height;
    readonly attribute GPUIntegerCoordinate depthOrArrayLayers;
    readonly attribute GPUIntegerCoordinate mipLevelCount;
    readonly attribute GPUSize32 sampleCount;
    readonly attribute GPUTextureDimension dimension;
    readonly attribute GPUTextureFormat format;
    readonly attribute GPUTextureUsageFlags usage;
};
GPUTexture includes GPUObjectBase;
```

GPUTexture 有以下属性：    
**width, 类型为： GPUIntegerCoordinate, 只读**    
GPUTexture的宽度。    
**height, 类型为： GPUIntegerCoordinate, 只读**    
GPUTexture的高度。    
**depthOrArrayLayers，类型为： GPUIntegerCoordinate，只读。**    
GPUTexture的深度或层数。    
**mipLevelCount，类型为： GPUIntegerCoordinate，只读。**    
此PUTexture的mip层的数量。    
    
> GPUTexture has the following attributes:
>> **width, of type GPUIntegerCoordinate, readonly**    
>> The width of this GPUTexture.      
>> **height, of type GPUIntegerCoordinate, readonly**    
>> The height of this GPUTexture.    
>> **depthOrArrayLayers, of type GPUIntegerCoordinate, readonly**    
>> The depth or layer count of this GPUTexture.    
>> **mipLevelCount, of type GPUIntegerCoordinate, readonly**    
>> The number of mip levels of this GPUTexture.    

**sampleCount, 类型为： GPUSize32, 只读**    
GPUTexture的样本数。    
**dimension, 类型为： GPUTextureDimension, 只读**    
GPUTexture的每个子资源的 texel 集合的尺寸。    
**format, 类型为： GPUTextureFormat, 只读**    
GPUTexture的格式。    
**usage，类型为： GPUTextureUsageFlags，只读**    
GPUTexture允许的使用方式。

>> **sampleCount, of type GPUSize32, readonly**    
>> The number of sample count of this GPUTexture.    
>> **dimension, of type GPUTextureDimension, readonly**    
>> The dimension of the set of texel for each of this GPUTexture's subresources.    
>> **format, of type GPUTextureFormat, readonly**    
>> The format of this GPUTexture.    
>> **usage, of type GPUTextureUsageFlags, readonly**    
>> The allowed usages for this GPUTexture.     

GPUTexture有以下内部插槽：
**[[size]]，类型为： GPUExtent3D**    
纹理的大小（与宽度、高度和depthOrArrayLayers属性相同）。    
**[[viewFormats]]，类型为： sequence\<GPUTextureFormat\>。**    
在此GPUTexture上创建视图时，可以使用 GPUTextureViewDescriptor.format 的GPUTextureFormats集合。    
**[[destroyed]]，类型为： boolean，最初为false。**    
如果纹理被销毁，那么它就不能再被用于任何操作，其对应的底层内存可以被释放。    

> GPUTexture has the following internal slots:    
>> **[[size]], of type GPUExtent3D**    
>> The size of the texture (same as the width, height, and depthOrArrayLayers attributes).    
>> **[[viewFormats]], of type sequence\<GPUTextureFormat\>**    
>> The set of GPUTextureFormats that can be used GPUTextureViewDescriptor.format when creating views on this GPUTexture.    
>> **[[destroyed]], of type boolean, initially false**    
>> If the texture is destroyed, it can no longer be used in any operation, and its underlying memory can be freed.

**计算渲染范围（baseSize, mipLevel）。**    
**参数：**    
  * GPUExtent3D baseSize    
  * GPUSize32 mipLevel    
返回值： GPUExtent3DDict    
1. 让 extent 成为一个新的 GPUExtent3DDict 对象。
2. 设置 extent.width 设置为 max(1, baseSize.width ≫ mipLevel)。
3. 将 extent.height 设置为 max(1, baseSize.height ≫ mipLevel)。
4. 设置 extent.depthOrArrayLayers 为1。
5. 返回 extent。

>> **compute render extent**(baseSize, mipLevel)    
>> **Arguments:**    
>> * GPUExtent3D baseSize
>> * GPUSize32 mipLevel
>> **Returns:** GPUExtent3DDict    
>> 1. Let extent be a new GPUExtent3DDict object.
>> 2. Set extent.width to max(1, baseSize.width ≫ mipLevel).
>> 3. Set extent.height to max(1, baseSize.height ≫ mipLevel).
>> 4. Set extent.depthOrArrayLayers to 1.
>> 5. Return extent.
 
纹理的逻辑特定的 miplevel 纹理范围是指 纹理在特定 miplevel 级别上的 以texels为单位的纹理大小。它是由下面的程序过程计算出来的:

> The logical miplevel-specific texture extent of a texture is the size of the texture in texels at a specific miplevel. It is calculated by this procedure:

**逻辑上特定的 miplevel 纹理范围(descriptor, mipLevel)函数**     
**参数是以下两个：**    
* GPUTextureDescriptor descriptor
* GPUSize32 mipLevel

> **Logical miplevel-specific texture extent**(descriptor, mipLevel)
> **Arguments:**
> * GPUTextureDescriptor descriptor
> * GPUSize32 mipLevel    


**返回值：** GPUExtent3DDict    
1. 让 extent 成为一个新的 GPUExtent3DDict 对象。
2. 如果 descriptor.dimension 是：
    * **"1d"**    
        * 将 extent.width 设置为 max(1, descriptor.size.width ≫ mipLevel)。
        * 将 extent.height 设置为 1。
        * 将 extent.depthOrArrayLayers 设置为 1。

    * **"2d"**    
        * 将 extent.width 设置为 max(1, descriptor.size.width ≫ mipLevel)。
        * 将 extent.height 设置为 max(1, descriptor.size.height ≫ mipLevel)。
        * 将 extent.depthOrArrayLayers 设置为 descriptor.size.depthOrArrayLayers。

    * **"3d"**    
        * 将 extent.width 设置为 max(1, descriptor.size.width ≫ mipLevel)。
        * 将 extent.height 设置为 max(1, descriptor.size.height ≫ mipLevel)。
        * 将 extent.depthOrArrayLayers 设置为： max(1, descriptor.size.depthOrArrayLayers ≫ mipLevel)。

3. 返回extent

> **Returns:** GPUExtent3DDict
> 1. Let extent be a new GPUExtent3DDict object.
> 2. If descriptor.dimension is:    
>    **"1d"**    
>    * Set extent.width to max(1, descriptor.size.width ≫ mipLevel).
>    * Set extent.height to 1.
>    * Set extent.depthOrArrayLayers to 1.

>    **"2d"**    
>    * Set extent.width to max(1, descriptor.size.width ≫ mipLevel).
>    * Set extent.height to max(1, descriptor.size.height ≫ mipLevel).
>    * Set extent.depthOrArrayLayers to descriptor.size.depthOrArrayLayers.

>    **"3d"**    
>    * Set extent.width to max(1, descriptor.size.width ≫ mipLevel).
>    * Set extent.height to max(1, descriptor.size.height ≫ mipLevel).
>    * Set extent.depthOrArrayLayers to max(1, descriptor.size.depthOrArrayLayers ≫ mipLevel).
> 3.  Return extent.

纹理的 物理上特定的 miplevel 纹理范围是 纹理在特定 miplevel 级别上的以 texels 为单位的纹理大小，包括可能存在的额外填充，以便形成纹理中的完整texel块。它是由以下这个程序过程计算出来的：


> The physical miplevel-specific texture extent of a texture is the size of the texture in texels at a specific miplevel that includes the possible extra padding to form complete texel blocks in the texture. It is calculated by this procedure:

**物理上 miplevel-specific texture extent(descriptor, mipLevel)。**    
**参数:**    
* GPUTextureDescriptor descriptor
* GPUSize32 mipLevel

> **Physical miplevel-specific texture extent**(descriptor, mipLevel)    
> **Arguments:**
> * GPUTextureDescriptor descriptor
> * GPUSize32 mipLevel

**返回值：**  GPUExtent3DDict
1. 让extent成为一个新的GPUExtent3DDict对象。
2. 让logicalExtent成为逻辑上特定于miplevel的纹理范围（descriptor, mipLevel）。
3. 如果descriptor.dimension是:     
    * **"1d"**    
        * 将 extent.width 设置为 logicalExtent.width， 并且四舍五入为描述符的texel宽度的最近倍数。
        * 将 extent.height 设置为 1。
        * 将 extent.depthOrArrayLayers 设置为 1。

    * **"2d"**    
        * 将 extent.width 设置为 logicalExtent.width， 并且四舍五入为描述符的texel宽度的最近倍数。
        * 将 extent.height 设置为 logicalExtent.height， 并且四舍五入为描述符的texel高度的最近倍数。
        * 将 extent.depthOrArrayLayers 设置为 logicalExtent.depthOrArrayLayers。

    * **"3d"**    
        * 将 extent.width 设置为 logicalExtent.width， 并且四舍五入为描述符的texel宽度的最近倍数。
        * 将 extent.height 设置为 logicalExtent.height， 并且四舍五入为描述符的texel高度的最近倍数。
        * 将 extent.depthOrArrayLayers 设置为 logicalExtent.depthOrArrayLayers。

4. 返回 extent。

> **Returns:** GPUExtent3DDict     
> 1. Let extent be a new GPUExtent3DDict object.
> 2. Let logicalExtent be logical miplevel-specific texture extent(descriptor, mipLevel).
> 3. If descriptor.dimension is:

>    **"1d"**    
>    * Set extent.width to logicalExtent.width rounded up to the nearest multiple of descriptor’s texel block width.
>    * Set extent.height to 1.
>    * Set extent.depthOrArrayLayers to 1.

>    **"2d"**    
>    * Set extent.width to logicalExtent.width rounded up to the nearest multiple of descriptor’s texel block width.
>    * Set extent.height to logicalExtent.height rounded up to the nearest multiple of descriptor’s texel block height.
>    * Set extent.depthOrArrayLayers to logicalExtent.depthOrArrayLayers.

>    **"3d"**    
>    * Set extent.width to logicalExtent.width rounded up to the nearest multiple of descriptor’s texel block width.
>    * Set extent.height to logicalExtent.height rounded up to the nearest multiple of descriptor’s texel block height.
>    * Set extent.depthOrArrayLayers to logicalExtent.depthOrArrayLayers.

> 4. Return extent.

### 6.1.1. 创建纹理(Texture Creation)

> 6.1.1. Texture Creation   

GPUTextures 是通过 createTexture() 创建的。

> GPUTextures are created via createTexture().

```
dictionary GPUTextureDescriptor : GPUObjectDescriptorBase {
    required GPUExtent3D size;
    GPUIntegerCoordinate mipLevelCount = 1;
    GPUSize32 sampleCount = 1;
    GPUTextureDimension dimension = "2d";
    required GPUTextureFormat format;
    required GPUTextureUsageFlags usage;
    sequence<GPUTextureFormat> viewFormats = [];
};
```

GPUTextureDescriptor 有以下成员变量：    
**size，类型为： GPUExtent3D**    
纹理的宽度、高度和深度或层数。

**mipLevelCount，类型为: GPUIntegerCoordinate，默认为1。**    
纹理所包含的 mip层的数量。

**sampleCount，类型为: GPUSize32，默认为1。**    
纹理的采样次数。sampleCount大于1表示是多采样纹理。

**dimension，类型为: GPUTextureDimension，默认为 "2d"。**    
无论纹理是一维的，还是二维的数组，还是三维的。

**format，类型为： GPUTextureFormat**    
纹理的格式。

**usage，类型为：GPUTextureUsageFlags**
纹理的允许使用方法的标记。

**viewFormats，类型为 sequence<GPUTextureFormat>，默认为[] 。**    
指定在此纹理上调用createView()时允许的视图格式值（除了纹理的实际格式外）。

> GPUTextureDescriptor has the following members:
>> **size, of type GPUExtent3D**    
The width, height, and depth or layer count of the texture.    
>>    
>> **mipLevelCount, of type GPUIntegerCoordinate, defaulting to 1**    
The number of mip levels the texture will contain.    
>>    
>> **sampleCount, of type GPUSize32, defaulting to 1**    
The sample count of the texture. A sampleCount > 1 indicates a multisampled texture.    
>>    
>> **dimension, of type GPUTextureDimension, defaulting to "2d"**    
Whether the texture is one-dimensional, an array of two-dimensional layers, or three-dimensional.    
>>    
>> **format, of type GPUTextureFormat**    
The format of the texture.    
>>    
>> **usage, of type GPUTextureUsageFlags**    
The allowed usages for the texture.    
>>    
>> **viewFormats, of type sequence\<GPUTextureFormat\>, defaulting to []**    
Specifies what view format values will be allowed when calling createView() on this texture (in addition to the texture’s actual format).    

**注意：**在这个列表中增加一个格式可能会对性能产生重大影响，所以最好避免增加不必要的格式。    

实际的性能影响在很大程度上取决于目标运行系统；开发人员必须测试各种系统，以找出对他们特定应用的影响。例如，在一些系统上，任何纹理的格式或视图条目入口包含 "rgba8unorm-srgb" 格式的性能 都会比只包有 "rgba8unorm "格式的纹理性能更差。在其他系统上，其他格式和其他格式组合，也应有类似的注意事项。

>> **Note:** Adding a format to this list may have a significant performance impact, so it is best to avoid adding formats unnecessarily.    
>>    
>> The actual performance impact is highly dependent on the target system; developers must test various systems to find out the impact on their particular application. For example, on some systems any texture with a format or viewFormats entry including "rgba8unorm-srgb" will perform less optimally than a "rgba8unorm" texture which does not. Similar caveats exist for other formats and pairs of formats on other systems.

在此列表中的格式必须是纹理的视图模式，而且要与纹理实际格式相兼容。

如果满足下面两个条件之一， 那么两个 GPUTextureFormats 的format 和 viewFormat 就是 纹理视图格式相兼容的。
* format 等同于 viewFormat，或
* format 和 viewFormat 是否仅在srgb格式（有-srgb后缀）上有区别。

```
ISSUE 6 定义更大的兼容类。[问题 #gpuweb/gpuweb#168] 。
```

>> Formats in this list must be texture view format compatible with the texture format.    
>> Two GPUTextureFormats format and viewFormat are texture view format compatible if:
>> * format equals viewFormat, or
>> * format and viewFormat differ only in whether they are srgb formats (have the -srgb suffix).

```
ISSUE 6 Define larger compatibility classes. [Issue #gpuweb/gpuweb#168]
```

```
enum GPUTextureDimension {
    "1d",
    "2d",
    "3d"
};
```

**"1d"**    
指定仅有一个维度的纹理，即宽度。

**"2d"**    
指定一个有宽度和高度的纹理，并且可能会有层。只有 "2d" 纹理可以有 mipmaps、多采样、使用 压缩 或 深度/模式格式，并作为渲染附件使用。

**"3d"**    
指定一个有宽度、高度和深度的纹理。

>> **"1d"**    
>> Specifies a texture that has one dimension, width.    
>>    
>> **"2d"**    
Specifies a texture that has a width and height, and may have layers. Only "2d" textures may have mipmaps, be multisampled, use a compressed or depth/stencil format, and be used as a render attachment.    
>>    
>> **"3d"**    
>> Specifies a texture that has a width, height, and depth.   

```
typedef [EnforceRange] unsigned long GPUTextureUsageFlags;
[Exposed=(Window, DedicatedWorker)]
namespace GPUTextureUsage {
    const GPUFlagsConstant COPY_SRC          = 0x01;
    const GPUFlagsConstant COPY_DST          = 0x02;
    const GPUFlagsConstant TEXTURE_BINDING   = 0x04;
    const GPUFlagsConstant STORAGE_BINDING   = 0x08;
    const GPUFlagsConstant RENDER_ATTACHMENT = 0x10;
};
```

GPUTextureUsage 标志决定了一个 GPUTexture 在创建后可以被如何使用：

**COPY_SRC**    
该纹理可以被用作复制操作的源。(例如：作为 copyTextureToTexture() 或 copyTextureToBuffer() 调用的源参数）。

**COPY_DST**    
该纹理可以作为复制或写入操作的目标。(例如：作为 copyTextureToTexture() 或 copyBufferToTexture() 调用的目标参数，或者作为 writeTexture() 调用的写入目标）。

**texture_binding**    
该纹理可以被绑定，以便在着色器中作为采样纹理使用（例如：作为 GPUTextureBindingLayout 的绑定组条目。）

**storage_binding**     
纹理可以被绑定，以便在着色器中作为存储纹理使用（例如：作为 GPUStorageTextureBindingLayout 的绑定组条目。）

render_attachment（渲染附件
该纹理可以作为渲染通道中的 颜色 或 深度/模式 的附件使用。(例如：作为 GPURenderPassColorAttachment.view 或GPURenderPassDepthStencilAttachment.view。）

> The GPUTextureUsage flags determine how a GPUTexture may be used after its creation:    
>> **COPY_SRC**    
>> The texture can be used as the source of a copy operation. (Examples: as the source argument of a copyTextureToTexture() or copyTextureToBuffer() call.)    
>>    
>> **COPY_DST**    
>> The texture can be used as the destination of a copy or write operation. (Examples: as the destination argument of a copyTextureToTexture() or copyBufferToTexture() call, or as the target of a writeTexture() call.)    
>>    
>> **TEXTURE_BINDING**    
>> The texture can be bound for use as a sampled texture in a shader (Example: as a bind group entry for a GPUTextureBindingLayout.)    
>>    
>> **STORAGE_BINDING**    
>> The texture can be bound for use as a storage texture in a shader (Example: as a bind group entry for a GPUStorageTextureBindingLayout.)    
>>    
>> **RENDER_ATTACHMENT**    
>> The texture can be used as a color or depth/stencil attachment in a render pass. (Example: as a GPURenderPassColorAttachment.view or GPURenderPassDepthStencilAttachment.view.)    

 **maximum mipLevel count(dimension, size)**    
函数是 计算最大mipLevel数量，参数是维度和尺寸：      
* dimension 大小
* size 大小 
1. 假设要计算最大尺寸值为 m：
    * 如果尺寸是: 
        * **"1d"**    
          返回1。
        * **"2d"**    
          让 m = max(size.width, size.height)。
        * **"3d"**    
          让 m = max(max(size.width, size.height), size.depthOrArrayLayer)。    
        
返回 floor( log<SUB>2</SUB>(m)) + 1。

>> **maximum mipLevel count(dimension, size)**    
>> **Arguments:**    
>> * dimension dimension    
>> * size size    
>>    
>> 1. Calculate the max dimension value m:    
>>      * If dimension is:    
>>          * "1d"    
>>          Return 1.
>>          
>>          * "2d"    
>>          Let m = max(size.width, size.height).
>>          
>>          * "3d"    
>>          Let m = max(max(size.width, size.height), size.depthOrArrayLayer).
>> 2. Return floor(log2(m)) + 1.

**createTexture(descriptor)**    
用于创建一个 GPUTexture。    
**调用于：** GPUDevice 实例本身。    
**参数：**

| Parameter | Type | Nullable | Optional | Description |
| -------- | -------- | -------- | -------- | -------- |
| descriptor | GPUSize64 | ✘ | ✘ | 要创建的GPUTexture的描述符。  |


>> **createTexture(descriptor)**    
>> Creates a GPUTexture.    
>> **Called on:**  GPUDevice this.     
>> **Arguments:**    

| Parameter | Type | Nullable | Optional | Description |
| -------- | -------- | -------- | -------- | -------- |
| descriptor | GPUSize64 | ✘ | ✘ | Description of the GPUTexture to create. |

**返回值**：GPUTexture

在上下文时间线上的步骤：

1. 用 this.[[device]] 验证 descriptor.format 所要求特征的 的纹理格式。
2. 用 this.[[device]] 验证 descriptor.viewFormats 中每个元素所要求特征 的纹理格式。
3. 让 t 成为一个新的 GPUTexture 对象。
4. 将 t.width 设置为 descriptor.size.width。
5. 将 t.height 设置为 descriptor.size.height。
6. 将 t.depthOrArrayLayers 设置为 descriptor.size.depthOrArrayLayers。
7. 将 t.mipLevelCount 设置为 descriptor.mipLevelCount。
8. 将 t.sampleCount 设为 descriptor.sampleCount。
9. 将 t.dimension 设置为 descriptor.dimension。
10. 将 t.format 设为 descriptor.format。
11. 将 t.usage 设为 descriptor.usage。
12. 在这个实例的设备时间线上发起以下初始化步骤：
13. 返回t。

>> **Returns:** GPUTexture     
>> **Content timeline steps:**    
>> 1. Validate texture format required features of descriptor.format with this.[[device]].
>> 2. Validate texture format required features of each element of descriptor.viewFormats with this.[[device]].
>> 3. Let t be a new GPUTexture object.
>> 4. Set t.width to descriptor.size.width.
>> 5. Set t.height to descriptor.size.height.
>> 6. Set t.depthOrArrayLayers to descriptor.size.depthOrArrayLayers.
>> 7. Set t.mipLevelCount to descriptor.mipLevelCount.
>> 8. Set t.sampleCount to descriptor.sampleCount.
>> 9. Set t.dimension to descriptor.dimension.
>> 10. Set t.format to descriptor.format.
>> 11. Set t.usage to descriptor.usage.
>> 12. Issue the initialization steps on the Device timeline of this.
>> 13. Return t.    


**设备时间线上的初始化步骤：**    
1. 如果以下任何条件不满足，产生一个验证错误，使 t 无效，并停止：
    * 验证 GPUTextureDescriptor(this, descriptor) 返回值要为 true。
2. 将 t.[[size]] 设置为 descriptor.size。
3. 将 t.[[viewFormats]] 设置为 descriptor.viewFormats。

>> **Device timeline initialization steps:**    
>> 1. If any of the following conditions are unsatisfied generate a validation error, make t invalid, and stop.    
>>    
>>    * validating GPUTextureDescriptor(this, descriptor) returns true.
>> 2. Set t.[[size]] to descriptor.size.
>> 3. Set t.[[viewFormats]] to descriptor.viewFormats.

**validating GPUTextureDescriptor(GPUDevice this, GPUTextureDescriptor descriptor):**    
如果满足以下所有要求，则返回true，否则返回false：
  * this 对象必须是一个有效的 GPUDevice。
  * descriptor.usage 不能为 0。
  * descriptor.usage 的设置位，必须只包含存在 GPUDevice 实例自身所允许的 texture 的使用方法 中的位。
  * descriptor.size.width, descriptor.size.height, and descriptor.size.depthOrArrayLayers 必须大于 0。
  * descriptor.mipLevelCount 必须大于 0。
  * descriptor.sampleCount 必须是 1 或 4。

>> **validating GPUTextureDescriptor(GPUDevice this, GPUTextureDescriptor descriptor):**    
>> Return true if all of the following requirements are met, and false otherwise:    
>> * this must be a valid GPUDevice.
>> * descriptor.usage must not be 0.
>> * descriptor.usage must contain only bits present in this’s allowed texture usages.
>> * descriptor.size.width, descriptor.size.height, and descriptor.size.depthOrArrayLayers must be > zero.
>> * descriptor.mipLevelCount must be > zero.
>> * descriptor.sampleCount must be either 1 or 4.

如果descriptor.dimension：
> **"1d"**    
> * descriptor.size.width 必须是小于等于 this.limit.maxTextureDimension1D。
> * descriptor.size.height 必须是 1。
> * descriptor.size.depthOrArrayLayers 必须为 1。
> * descriptor.sampleCount 必须是 1。
> * descriptor.format 必须不是压缩格式 或不是 深度/模式 格式。

>> * If descriptor.dimension is:    
>>    * **"1d"**    
>>        * descriptor.size.width must be ≤ this.limits.maxTextureDimension1D.
>>        * descriptor.size.height must be 1.
>>        * descriptor.size.depthOrArrayLayers must be 1.
>>        * descriptor.sampleCount must be 1.
>>        * descriptor.format must not be a compressed format or depth-or-stencil format.

> **"2d"**    
> * descriptor.size.width 必须是小于等于 this.limit.maxTextureDimension2D。
> * descriptor.size.height 必须是小于等于 this.limits.maxTextureDimension2D。
> * descriptor.size.depthOrArrayLayers 必须是小于等于 this.limits.maxTextureArrayLayers。

>>    * **"2d"**    
>>        * descriptor.size.width must be ≤ this.limits.maxTextureDimension2D.
>>        * descriptor.size.height must be ≤ this.limits.maxTextureDimension2D.
>>        * descriptor.size.depthOrArrayLayers must be ≤ this.limits.maxTextureArrayLayers.

> **"3d"**    
> * descriptor.size.width 必须是小于等于 this.limit.maxTextureDimension3D。
> * descriptor.size.height 必须是小于等于 this.limits.maxTextureDimension3D。
> * descriptor.size.depthOrArrayLayers 必须是小于等于 this.limit.maxTextureDimension3D。
> * descriptor.sampleCount 必须是 1。
> * descriptor.format 必须不是压缩格式 或不是 深度/模式 格式。

>>    * **"3d"**    
>>        * descriptor.size.width must be ≤ this.limits.maxTextureDimension3D.
>>        * descriptor.size.height must be ≤ this.limits.maxTextureDimension3D.
>>        * descriptor.size.depthOrArrayLayers must be ≤ this.limits.maxTextureDimension3D.
>>        * descriptor.sampleCount must be 1.
>>        * descriptor.format must not be a compressed format or depth-or-stencil format.

> * descriptor.size.width 必须是 texel块宽度 的倍数。
> * escriptor.size.height 必须是 texel块高度 的倍数。
> * 如果 descriptor.sampleCount > 1:
>    * descriptor.mipLevelCount 必须是 1。
>    * descriptor.size.depthOrArrayLayers 必须是 1。
>    * descriptor.usage 必须不包括 STORAGE_BINDING 位。
>    * descriptor.usage 必须包括 RENDER_ATTACHMENT 位。
>    * descriptor.format 必须根据 §26.1 纹理格式功能 章节中所支持多采样。

>> * descriptor.size.width must be multiple of texel block width.
>> * descriptor.size.height must be multiple of texel block height.
>> * If descriptor.sampleCount > 1:    
>>    * descriptor.mipLevelCount must be 1.
>>    * descriptor.size.depthOrArrayLayers must be 1.
>>    * descriptor.usage must not include the STORAGE_BINDING bit.
>>    * descriptor.usage must include the RENDER_ATTACHMENT bit.
>>    * descriptor.format must support multisampling according to § 26.1 Texture Format Capabilities.

> * descriptor.mipLevelCount 必须小于等于 最大的 mipLevel 数量（descriptor.dimension，descriptor.size）。
> * 如果 descriptor.usage 包括 RENDER_ATTACHMENT 位。
>    * descriptor.format 必须是一种可渲染的格式。
>    * descriptor.dimensionn必须是 "2d"。
> * 如果 descriptor.usage 包括 STORAGE_BINDING 位。
>    * descriptor.format 必须列在 §26.1.1素色格式表中，并具有 STORAGE_BINDING 功能。
> * 对于 descriptor.viewFormats 中的每个 viewFormat，descriptor.format 和 viewFormat 必须是兼容的纹理视图格式。

> * descriptor.mipLevelCount must be ≤ maximum mipLevel count(descriptor.dimension, descriptor.size).
> * If descriptor.usage includes the RENDER_ATTACHMENT bit:
>    * descriptor.format must be a renderable format.
>    * descriptor.dimension must be "2d".

> * If descriptor.usage includes the STORAGE_BINDING bit:
>    * descriptor.format must be listed in § 26.1.1 Plain color formats table with STORAGE_BINDING capability.
> * For each viewFormat in descriptor.viewFormats, descriptor.format and viewFormat must be texture view format compatible.

```
EXAMPLE 14
// 创建一个 16x16，RGBA，具有一个阵列层和一个金子塔层的 2D 纹理。
Creating a 16x16, RGBA, 2D texture with one array layer and one mip level:

const texture = gpuDevice.createTexture({
    size: { width: 16, height: 16 },
    format: 'rgba8unorm',
    usage: GPUTextureUsage.TEXTURE_BINDING,
});
```

### 6.1.2. 纹理析构

一个不再需要 GPUTexture 的应用程序可以通过调用destroy()，以便在垃圾回收机制之前，失去对它的控制访问。

**注意：** 这允许用户代理 在所有先前提交的使用 GPUTexture 的操作完成后，回收与该 GPUTexture 相关的GPU内存。

> 6.1.2. Texture Destruction    
> An application that no longer requires a GPUTexture can choose to lose access to it before garbage collection by calling destroy().

> **NOTE:**  This allows the user agent to reclaim the GPU memory associated with the GPUTexture once all previously submitted operations using it are complete.

`destroy()`    
销毁 GPUTexture。    
调用于: GPUTexture 实例自身。    
返回值：undefined。    
上下文时间线步骤：    
1. 将 this.[[destroyed]] 设为true。

> **destroy()**    
> Destroys the GPUTexture.    
>    
> **Called on:** GPUTexture this.    
> **Returns:** undefined    
> Content timeline steps:
> 1. Set this.[[destroyed]] to true.

## 6.2. 纹理视图（GPUTextureView）    
GPUTextureView 是对特定 GPUTexture 所定义的纹理子资源的某个子集的视图。

> 6.2. GPUTextureView    

> A GPUTextureView is a view onto some subset of the texture subresources defined by a particular GPUTexture.    

```
[Exposed=(Window, DedicatedWorker), SecureContext]
interface GPUTextureView {
};
GPUTextureView includes GPUObjectBase;
````

GPUTextureView 有以下内部插槽:
> **[[texture]]**    
表明这是那个 GPUTexture 的视图。
> 
> **[[descriptor]]**   
描述这个纹理视图的 GPUTextureViewDescriptor。
> 
> GPUTextureViewDescriptor 的所有可选字段都已定义。
> 
> **[[renderExtent]]**    
对于可渲染的视图，这是用于渲染的有效 GPUExtent3DDict。
> 
> **注意：** 这个渲染范围取决于 baseMipLevel。

> GPUTextureView has the following internal slots:    
>> **[[texture]]**    
>> The GPUTexture into which this is a view.    
>>    
>> **[[descriptor]]**    
>> The GPUTextureViewDescriptor describing this texture view.    
>> All optional fields of GPUTextureViewDescriptor are defined.    
>>    
>> **[[renderExtent]]**    
>> For renderable views, this is the effective GPUExtent3DDict for rendering.    
>> **NOTE:** this extent depends on the baseMipLevel.    

具有 [[descriptor]] 描述信息的纹理视图的视图的子资源集，是 view.[[texture]] 的子资源的子集，其中每个子资源 s 来说，需要满足以下条件：
> * s 的 mipmap级别 ≥ desc.baseMipLevel 并且 < desc.baseMipLevel + desc.mipLevelCount。
> * s 的阵列层 ≥ desc.baseArrayLayer 并且 < desc.baseArrayLayer + desc.arrayLayerCount。
> * s 的 特征 是在 desc.aspect 所描述的特征集合中。

当且仅当两个 GPUTextureView 对象的子资源集相交时，那么，这两个纹理对象 是 纹理视图相等价的？？？。

>> The set of subresources of a texture view view, with [[descriptor]] desc, is the subset of the subresources of view.[[texture]] for which each subresource s satisfies the following:
>> * The mipmap level of s is ≥ desc.baseMipLevel and < desc.baseMipLevel + desc.mipLevelCount.
>> * The array layer of s is ≥ desc.baseArrayLayer and < desc.baseArrayLayer + desc.arrayLayerCount.
>> * The aspect of s is in the set of aspects of desc.aspect.
>>    
>> Two GPUTextureView objects are texture-view-aliasing if and only if their sets of subresources intersect.



### 6.2.1. 纹理视图创建

> 6.2.1. Texture View Creation

```
dictionary GPUTextureViewDescriptor : GPUObjectDescriptorBase {
    GPUTextureFormat format;
    GPUTextureViewDimension dimension;
    GPUTextureAspect aspect = "all";
    GPUIntegerCoordinate baseMipLevel = 0;
    GPUIntegerCoordinate mipLevelCount;
    GPUIntegerCoordinate baseArrayLayer = 0;
    GPUIntegerCoordinate arrayLayerCount;
};
```

GPUTextureDescriptor 具有以下成员：
> **format，类型为: GPUTextureFormat**    
纹理视图的格式。必须是纹理的格式或者是创建纹理时指定的 viewFormats 之一。
> 
> **dimension，类型为: GPUTextureViewDimension。**    
用于查看纹理的尺寸。
> 
> **aspect，类型为: GPUTextureAspect，默认为 "all"。**    
纹理视图中可以看到的纹理的哪个特征面？？。
> 
> **baseMipLevel，类型为: GPUIntegerCoordinate，默认为 0。**    
纹理视图可以访问的第一个（即最详细的）mipmap级别。
> 
> **mipLevelCount，类型为: GPUIntegerCoordinate。**    
从 baseMipLevel 开始，纹理视图可以访问多少个 mipmap 级别。
> 
> **baseArrayLayer，类型为: GPUIntegerCoordinate，默认为0。**    
纹理视图可以访问的第一个阵列层的索引。
>
> **arrayLayerCount，类型为: GPUIntegerCoordinate。**    
从 baseArrayLayer 开始，纹理视图可以访问多少个阵列层。

>> GPUTextureDescriptor has the following members:
>>    
>> **format, of type GPUTextureFormat**    
>> The format of the texture view. Must be either the format of the texture or one of the viewFormats specified during its creation.
>>    
>> **dimension, of type GPUTextureViewDimension**    
>> The dimension to view the texture as.
>>    
>> **aspect, of type GPUTextureAspect, defaulting to "all"**    
>> Which aspect(s) of the texture are accessible to the texture view.
>>    
>> **baseMipLevel, of type GPUIntegerCoordinate, defaulting to 0**    
>> The first (most detailed) mipmap level accessible to the texture view.
>>    
>> **mipLevelCount, of type GPUIntegerCoordinate**    
>> How many mipmap levels, starting with baseMipLevel, are accessible to the texture view.
>>    
>> **baseArrayLayer, of type GPUIntegerCoordinate, defaulting to 0**    
>> The index of the first array layer accessible to the texture view.
>>    
>> **arrayLayerCount, of type GPUIntegerCoordinate**    
>> How many array layers, starting with baseArrayLayer, are accessible to the texture view.

```
// 代码不译不复制

enum GPUTextureViewDimension {
    "1d",
    "2d",
    "2d-array",
    "cube",
    "cube-array",
    "3d"
};
```
接上面解释 GPUTextureViewDimension 中值的意义：    
**"1d"**    
表明纹理被看作是一个一维的图像。    
对应的 WGSL 类型为：    
* texture_1d
* texture_storage_1d

**"2d"**    
表明纹理被看成是一个单一的二维图像。    
对应的 WGSL 类型为：    
* texture_2d
* texture_storage_2d
* texture_multisampled_2d
* texture_depth_2d
* texture_depth_multisampled_2d

**"2d-array"**    
表明纹理视图被看作是一个二维图像的阵列。    
对应的 WGSL 类型。    
* texture_2d_array
* texture_storage_2d_array
* texture_depth_2d_array

**"cube"**    
表明纹理被看做是一个立方体地图。该视图有6个阵列层，分别对应立方体的[+X, -X, +Y, -Y, +Z, -Z]面。采样是在 cubemap 的各个面之间无缝进行的。    
对应的 WGSL 类型。    
* texture_cube
* texture_depth_cube

**"cube-array"**    
表明纹理被看作是一个由n个立方体图组成的包阵列，每个立方体有6个阵列层，分别对应于立方体的[+X, -X, +Y, -Y, +Z, -Z]面。采样是在立方体的各个面上无缝进行的。
对应的 WGSL 类型为：    
* texture_cube_array    
* texture_depth_cube_array

**"3d"**    
表明纹理被看作是一个三维的图像。
对应的 WGSL 类型为：    
* texture_3d
* texture_storage_3d

> **"1d"**    
> The texture is viewed as a 1-dimensional image.    
> Corresponding WGSL types:    
> * texture_1d
> * texture_storage_1d
>    
> **"2d"**    
> The texture is viewed as a single 2-dimensional image.    
> Corresponding WGSL types:    
> * texture_2d
> * texture_storage_2d
> * texture_multisampled_2d
> * texture_depth_2d
> * texture_depth_multisampled_2d
>    
> **"2d-array"**    
> The texture view is viewed as an array of 2-dimensional images.    
> Corresponding WGSL types:    
> * texture_2d_array
> * texture_storage_2d_array
> * texture_depth_2d_array
>    
> **"cube"**    
> The texture is viewed as a cubemap. The view has 6 array layers, corresponding to the [+X, -X, +Y, -Y, +Z, -Z] faces of the cube. Sampling is done seamlessly across the faces of the cubemap.    
> Corresponding WGSL types:    
> * texture_cube
> * texture_depth_cube
>    
> **"cube-array"**    
> The texture is viewed as a packed array of n cubemaps, each with 6 array layers corresponding to the [+X, -X, +Y, -Y, +Z, -Z] faces of the cube. Sampling is done seamlessly across the faces of the cubemaps.    
> Corresponding WGSL types:    
> * texture_cube_array
> * texture_depth_cube_array
>    
> **"3d"**    
> The texture is viewed as a 3-dimensional image.    
> Corresponding WGSL types:    
> * texture_3d
> * texture_storage_3d

每个 GPUTextureAspect 值都对应着一组 特征方面值。下面为每个值定义了一组 特征方面值。

> Each GPUTextureAspect value corresponds to a set of aspects. The set of aspects are defined for each value below.

```
// 代码不译不复制
enum GPUTextureAspect {
    "all",
    "stencil-only",
    "depth-only"
};
```

> **"all"**    
> 纹理格式的所有可用特征方面都将被纹理视图访问。对于颜色格式，颜色方面将被访问。对于深度/模板的组合格式，深度和模板方面都可以访问。具有单一方面的深度或模板格式将只使该方面的特征可以访问。
>    
> 特征方面的集合是[color, depth, stencil]。

>> **"all"**    
>> All available aspects of the texture format will be accessible to the texture view. For color formats the color aspect will be accessible. For combined depth-stencil formats both the depth and stencil aspects will be accessible. Depth-or-stencil formats with a single aspect will only make that aspect accessible.
>>    
>> The set of aspects is [color, depth, stencil].

> **"stencil-only"**    
> 只有深度或模板格式(depth-or-stencil format)的模板方面可以被纹理视图访问。
>    
> 这方面的集合是[stencil]。

> **"depth-only"**    
> 只有深度或模板格式(depth-or-stencil format)的深度方面可以被纹理视图访问。
>    
> 这方面的集合是[depth]。

>> **"stencil-only"**    
>> Only the stencil aspect of a depth-or-stencil format format will be accessible to the texture view.
>>    
>> The set of aspects is [stencil].
>>    
>> **"depth-only"**    
>> Only the depth aspect of a depth-or-stencil format format will be accessible to the texture view.
>>    
>> The set of aspects is [depth].

**createView(descriptor)**    
用于创建一个 GPUTextureView 的函数。

**注意：默认情况下，createView()将创建一个可以代表整个纹理的尺寸的视图。例如，针对一个具有多层的 "2D" 纹理，在调用createView()时，不特别的指定一个维度尺寸，将创建一个 "2D-array" 的 GPUTextureView，即使指定 arrayLayerCount 为 1 也将如此。

在开发时，对于从资源创建的纹理，如果不知道纹理的层数，建议在调用 createView() 时提供一个明确的尺寸，以确保着色器的兼容性。


> **createView(descriptor)**    
> Creates a GPUTextureView.
>    
>> **Note:** By default createView() will create a view with a dimension that can represent the entire texture. For example, calling createView() without specifying a dimension on a "2d" texture with more than one layer will create a "2d-array" GPUTextureView, even if an arrayLayerCount of 1 is specified.
>>    
>> For textures created from sources where the layer count is unknown at the time of development it is recommended that calls to createView() are provided with an explicit dimension to ensure shader compatibility.

**调用于：** GPUTexture 实例自身。

**参数：**

| Parameter | Type | Nullable | Optional | Description |
| -------- | -------- | -------- | -------- | -------- |
| descriptor | GPUTextureViewDescriptor | ✘ | ✔ | 要创建的GPUTextureView的描述符信息 |

**返回值：** view，即视图， 类型为：GPUTextureView

创建过程如下：    
**上下文时间轴步骤： **    
1. 用 this.[[device]] 验证纹理 是否满足 descriptor.format 要求的纹理格式特征。
2. 让 view 成为一个新的 GPUTextureView 对象。
3. 在 this.[[device]] 的设备时间线上发起后续初始化步骤。
4. 返回view。

> **Called on**: GPUTexture this.    
> **Arguments:**    

| Parameter | Type | Nullable | Optional | Description |
| -------- | -------- | -------- | -------- | -------- |
| descriptor | GPUTextureViewDescriptor | ✘ | ✔ | Description of the GPUTextureView to create。 |

> **Returns:** view, of type GPUTextureView.
>    
> **Content timeline steps:**
> 1. Validate texture format required features of descriptor.format with this.[[device]].
> 2. Let view be a new GPUTextureView object.
> 3. Issue the initialization steps on the Device timeline of this.
> 4. Return view.

**设备时间轴初始化步骤**   
1. 将描述符设置为 Promise成功返回的 GPUTextureViewDescriptor 的结果值，默认就是为此而设置的该描述符。
2. 如果以下任何条件不满足，产生一个验证错误，使视图无效，并停止。
    * 实例自身是有效的，即 this is valid。
    * descriptor.aspect 必须存在于 this.format 中。
    * 如果 descriptor.aspect 是 "all"。
        * descriptor.format 必须等于 this.format 或者 this.[[viewFormats]] 中的任何一个格式。    
        否则。
        * descriptor.format 必须等于 函数 GPUTextureAspect(this.format, descriptor.aspect)调用后，其Promise成功后返回的结果。
    * descriptor.mipLevelCount 必须大于0。
    * descriptor.baseMipLevel + descriptor.mipLevelCount 必须小于等于 this.mipLevelCount。
    * descriptor.arrayLayerCount 必须大于0。
    * descriptor.baseArrayLayer + descriptor.arrayLayerCount 必须小于等于 this.array层数。
    * 如果 this.sampleCount > 1，descriptor.dimension 必须是 "2d"。

> **Device timeline initialization steps:** 
> 1. Set descriptor to the result of resolving GPUTextureViewDescriptor defaults for this with descriptor.
> 2. If any of the following conditions are unsatisfied generate a validation error, make view invalid, and stop.    
>>    * this is valid.
>>    * descriptor.aspect must be present in this.format.
>>    * If the descriptor.aspect is "all":    
>>        * descriptor.format must equal either this.format or one of the formats in this.[[viewFormats]].    
>>        Otherwise:    
>>        * descriptor.format must equal the result of resolving GPUTextureAspect( this.format, descriptor.aspect).
>>    * descriptor.mipLevelCount must be > 0.
>>    * descriptor.baseMipLevel + descriptor.mipLevelCount must be ≤ this.mipLevelCount.
>>    * descriptor.arrayLayerCount must be > 0.
>>    * descriptor.baseArrayLayer + descriptor.arrayLayerCount must be ≤ the array layer count of this.
>>    * If this.sampleCount > 1, descriptor.dimension must be "2d".

  * 如果 descriptor.dimension 是以下各种情况的话，需单独判断：
      * **"1d"**    
          * this.dimension 必须是 "1d"。
          * descriptor.arrayLayerCount 必须是1。
      * **"2d"**
          * this.dimension 必须是 "2d"。
          * descriptor.arrayLayerCount 必须是1。
      * **"2d-array"**
          * this.dimension 必须是 "2d"。
      * **"cube"**
          * this.dimension 必须是 "2d"。
          * descriptor.arrayLayerCount 必须是6。
          * this.width 必须等于 this.height。
      * **"cube-array"**
          * this.dimension 必须是 "2d"。
          * descriptor.arrayLayerCount 必须是6的倍数。
          * this.width 必须等于 this.height。
      * **"3d"**
          * this.dimension 必须是 "3d"。
          * descriptor.arrayLayerCount 必须是1。

>>    * If descriptor.dimension is:
>>
>>       **"1d"**    
>>        * this.dimension must be "1d".
>>        * descriptor.arrayLayerCount must be 1.
>>        
>>       **"2d"**    
>>        * this.dimension must be "2d".
>>        * descriptor.arrayLayerCount must be 1.
>>        
>>       **"2d-array"**    
>>        * this.dimension must be "2d".
>>        
>>       **"cube"**    
>>        * this.dimension must be "2d".
>>        * descriptor.arrayLayerCount must be 6.
>>        * this.width must equal this.height.
>>        
>>       **"cube-array"**    
>>        * this.dimension must be "2d".
>>        * descriptor.arrayLayerCount must be a multiple of 6.
>>        * this.width must equal this.height.
>>        
>>       **"3d"**    
>>        * this.dimension must be "3d".
>>        * descriptor.arrayLayerCount must be 1.

3. 让 view 成为一个新的 GPUTextureView 对象。
4. 将 view.[[texture]] 设置为实例自身，因为此函数是在GPUTexture实例自身上调用的。
5. 将 view.[[descriptor]] 设置为descriptor。
6. 如果 this.usage 包含 RENDER_ATTACHMENT：
    1. 让 renderExtent 是计算渲染范围函数调用的结果，即computer_render_extent(this.[[size]], descriptor.baseMipLevel)调用的结果。
    2. 将 view.[[renderExtent]] 设置为 renderExtent。

> 3. Let view be a new GPUTextureView object.
> 4. Set view.[[texture]] to this.
> 5. Set view.[[descriptor]] to descriptor.
> 6. If this.usage contains RENDER_ATTACHMENT:
>    1. Let renderExtent be compute render extent(this.[[size]], descriptor.baseMipLevel).
>    2. Set view.[[renderExtent]] to renderExtent.

当用 GPUTextureViewDescriptor 描述符 来创建 GPUTextureView 纹理时， 当获得一个成功的 GPUTextureViewDescriptor 默认值时，请执行以下步骤：

1. 让 resolved 成为描述符的一个副本。
2. 如果 resolved.format 是未定义的：
    1. 让 format 是函数 GPUTextureAspect(format, descriptor.aspect)的Promise 成功的结果。    
    2. 如果 format 是空的：
        * 将 resolved.format 设置为 texture.format。    
        否则：
        * 将 resolved.format 设为 format。
3. 如果 resolved.mipLevelCount 是未定义的，将 resolved.mipLevelCount 设置为 texture.mipLevelCount - resolved.baseMipLevel。
4. 如果 resolved.dimension 是未定义的，并且 texture.dimension 是以下各种情况时：    
    **"1d"**     
    将 resolved.dimension 设置为 "1d"。    
    **"2d"**    
    如果纹理的阵列层数为1：
        * 将 resolved.dimension 设置为 "2d"。
        否则：
        * 将 resolved.dimension 设置为 "2d-array"。
    **"3d"**    
    将 resolved.dimension 设置为 "3d"。

> When resolving GPUTextureViewDescriptor defaults for GPUTextureView texture with a GPUTextureViewDescriptor descriptor run the following steps:
> 1. Let resolved be a copy of descriptor.
> 2. If resolved.format is undefined,
>    1. Let format be the result of resolving GPUTextureAspect( format, descriptor.aspect).
>    2. If format is null:
>       * Set resolved.format to texture.format.    
>       Otherwise:
>       * Set resolved.format to format.
>    
> 3. If resolved.mipLevelCount is undefined, set resolved.mipLevelCount to texture.mipLevelCount − resolved.baseMipLevel.
> 4. If resolved.dimension is undefined and texture.dimension is:
>> **"1d"**    
>> Set resolved.dimension to "1d".
>>    
>> **"2d"**    
>> If the array layer count of texture is 1:
>>    * Set resolved.dimension to "2d".    
>>        Otherwise:
>>    * Set resolved.dimension to "2d-array".
>>    
>> **"3d"**    
Set resolved.dimension to "3d".

5. 如果 resolved.arrayLayerCount 是未定义的，并且 resolved.dimension是以下各种情况时：    
    **"1d", "2d", 或 "3d"**   
    将 resolved.arrayLayerCount 设为1。
    **"cube"**     
    将 resolved.arrayLayerCount 设置为6。
    **"2D-array" 或 "cube-array"**    
    设置 resolved.arrayLayerCount 为纹理的阵列层数 - resolved.baseArrayLayer。
6. 返回 resolved

> 5.If resolved.arrayLayerCount is undefined and resolved.dimension is:
>> **"1d", "2d", or "3d"**    
>>    Set resolved.arrayLayerCount to 1.
>>    
>> **"cube"**    
>> Set resolved.arrayLayerCount to 6.    
>>    
>> **"2d-array" or "cube-array"**    
>> Set resolved.arrayLayerCount to the array layer count of texture − resolved.baseArrayLayer.
>>    
> 6. Return resolved.

要确定 GPUTexture 的 texture 的阵列层数，请执行以下步骤：
1. 如果 texture.dimension 是：     
    **"1d" 或 "3d"**    
    返回 1。    
    **"2d"**      
    返回 texture.depthOrArrayLayers。

> To determine the array layer count of GPUTexture texture, run the following steps:
> 1. If texture.dimension is:
>    
>> **"1d" or "3d"**    
>> Return 1.
>>    
>> **"2d"**    
>> Return texture.depthOrArrayLayers.


## 6.3. 纹理格式(Texture Formats)

格式的名称指定了格式组件顺序、纹理的每个组件的比特数 以及 组件的数据类型。    
r, g, b, a = 红、绿、蓝、α通道    
unorm = 无符号归一化数据    
snorm = 有符号归一化数据    
uint = 无符号整数    
sint = 有符号整数   
float = 浮点    

> 6.3. Texture Formats    
> The name of the format specifies the order of components, bits per component, and data type for the component.
> * r, g, b, a = red, green, blue, alpha
> * unorm = unsigned normalized
> * snorm = signed normalized
> * uint = unsigned int
> * sint = signed int
> * float = floating point

如果格式有 -srgb 的后缀，那么在着色器中读取和写入颜色值的过程中，就会进行从伽玛到线性的sRGB转换，反之亦然。压缩纹理格式是由特征提供的。它们的命名应该遵循这里的惯例，以纹理名称作为前缀。例如，etc2-rgba8unorm。

在基于像素的 GPUTextureFormats 格式中，texel块是单个可寻址的纹理元素，而在基于 块压缩 GPUTextureFormats 格式中，则是纹理的单个压缩块。

> If the format has the -srgb suffix, then sRGB conversions from gamma to linear and vice versa are applied during the reading and writing of color values in the shader. Compressed texture formats are provided by features. Their naming should follow the convention here, with the texture name as a prefix. e.g. etc2-rgba8unorm.
>    
> The texel block is a single addressable element of the textures in pixel-based GPUTextureFormats, and a single compressed block of the textures in block-based compressed GPUTextureFormats.
 
纹理块的宽度和高度指定了一个纹理块的尺寸。
    * 对于基于像素的 GPUTextureFormats，texel块宽度和高度总是 1。
    * 对于基于块压缩 GPUTextureFormats，texel块的宽度是 一个texel块中每一行的texel数量，而texel块的高度是一个texel块中的texel行数。请参阅第26.1节Texture Format Capabilities，以了解每种纹理格式的详尽数值列表。

> The texel block width and texel block height specifies the dimension of one texel block.
> * For pixel-based GPUTextureFormats, the texel block width and texel block height are always 1.
>    
> * For block-based compressed GPUTextureFormats, the texel block width is the number of texels in each row of one texel block, and the texel block height is the number of texel rows in one texel block. See § 26.1 Texture Format Capabilities for an exhaustive list of values for every texture format.


GPUTextureFormat 的 texel块 大小是存储一个texel块的字节数。除了 "stencil8"、"depth24plus "和 "depth24plus-stencil8"，每个 GPUTextureFormat 的 texel块 大小都是常数值。

> The texel block size of a GPUTextureFormat is the number of bytes to store one texel block. The texel block size of each GPUTextureFormat is constant except for "stencil8", "depth24plus", and "depth24plus-stencil8".

```
enum GPUTextureFormat {
    // 8 位格式
    // 8-bit formats
    "r8unorm",
    "r8snorm",
    "r8uint",
    "r8sint",

    // 16 位格式
    // 16-bit formats
    "r16uint",
    "r16sint",
    "r16float",
    "rg8unorm",
    "rg8snorm",
    "rg8uint",
    "rg8sint",

    // 32 位格式
    // 32-bit formats
    "r32uint",
    "r32sint",
    "r32float",
    "rg16uint",
    "rg16sint",
    "rg16float",
    "rgba8unorm",
    "rgba8unorm-srgb",
    "rgba8snorm",
    "rgba8uint",
    "rgba8sint",
    "bgra8unorm",
    "bgra8unorm-srgb",
    
    // 包 32 位格式
    // Packed 32-bit formats
    "rgb9e5ufloat",
    "rgb10a2unorm",
    "rg11b10ufloat",

    // 64 位格式
    // 64-bit formats
    "rg32uint",
    "rg32sint",
    "rg32float",
    "rgba16uint",
    "rgba16sint",
    "rgba16float",

    // 128 位格式
    // 128-bit formats
    "rgba32uint",
    "rgba32sint",
    "rgba32float",

    // 深度/模板格式 
    // Depth/stencil formats
    "stencil8",
    "depth16unorm",
    "depth24plus",
    "depth24plus-stencil8",
    "depth32float",

    // "depth32float-stencil8" feature
    "depth32float-stencil8",

    // 如果 "texture-compression-bc "同时被 设备/用户代理支持,
    // 并在 requestDevice 中启用，则 BC 压缩格式可用.
    // BC compressed formats usable if "texture-compression-bc" is both
    // supported by the device/user agent and enabled in requestDevice.
    "bc1-rgba-unorm",
    "bc1-rgba-unorm-srgb",
    "bc2-rgba-unorm",
    "bc2-rgba-unorm-srgb",
    "bc3-rgba-unorm",
    "bc3-rgba-unorm-srgb",
    "bc4-r-unorm",
    "bc4-r-snorm",
    "bc5-rg-unorm",
    "bc5-rg-snorm",
    "bc6h-rgb-ufloat",
    "bc6h-rgb-float",
    "bc7-rgba-unorm",
    "bc7-rgba-unorm-srgb",

    // 如果 "texture-compression-etc2 "同时被 设备/用户代理支持,
    // 并在 requestDevice 中启用，则 ETC2 压缩格式可用.
    // ETC2 compressed formats usable if "texture-compression-etc2" is both
    // supported by the device/user agent and enabled in requestDevice.
    "etc2-rgb8unorm",
    "etc2-rgb8unorm-srgb",
    "etc2-rgb8a1unorm",
    "etc2-rgb8a1unorm-srgb",
    "etc2-rgba8unorm",
    "etc2-rgba8unorm-srgb",
    "eac-r11unorm",
    "eac-r11snorm",
    "eac-rg11unorm",
    "eac-rg11snorm",

    // 如果 "texture-compression-astc "同时被 设备/用户代理支持,
    // 并在 requestDevice 中启用，则 ASTC 压缩格式可用.
    // ASTC compressed formats usable if "texture-compression-astc" is both
    // supported by the device/user agent and enabled in requestDevice.
    "astc-4x4-unorm",
    "astc-4x4-unorm-srgb",
    "astc-5x4-unorm",
    "astc-5x4-unorm-srgb",
    "astc-5x5-unorm",
    "astc-5x5-unorm-srgb",
    "astc-6x5-unorm",
    "astc-6x5-unorm-srgb",
    "astc-6x6-unorm",
    "astc-6x6-unorm-srgb",
    "astc-8x5-unorm",
    "astc-8x5-unorm-srgb",
    "astc-8x6-unorm",
    "astc-8x6-unorm-srgb",
    "astc-8x8-unorm",
    "astc-8x8-unorm-srgb",
    "astc-10x5-unorm",
    "astc-10x5-unorm-srgb",
    "astc-10x6-unorm",
    "astc-10x6-unorm-srgb",
    "astc-10x8-unorm",
    "astc-10x8-unorm-srgb",
    "astc-10x10-unorm",
    "astc-10x10-unorm-srgb",
    "astc-12x10-unorm",
    "astc-12x10-unorm-srgb",
    "astc-12x12-unorm",
    "astc-12x12-unorm-srgb"
};
```

"depth24plus" 和 "depth24plus-stencil8" 格式的深度部分 可以被实现为一个 24位的深度值 或 一个 "depth32float"值。

```
 ISSUE 7  在 GPUAdapter(?) 上添加一些内容，则会对 "stencil8"、"depth24plus-stencil8" 和 "depth32float-stencil8" 格式的每个 texel 的字节数进行估算。[问题 #gpuweb/gpuweb#1887](https://github.com/gpuweb/gpuweb/issues/1887) 。
 ```

> The depth component of the "depth24plus" and "depth24plus-stencil8" formats may be implemented as either a 24-bit depth value or a "depth32float" value.

```
 ISSUE 7  add something on GPUAdapter(?) that gives an estimate of the bytes per texel of "stencil8", "depth24plus-stencil8", and "depth32float-stencil8". [Issue #gpuweb/gpuweb#1887]
```

"stencil8"格式 可以被实现为 一个真正的 "stencil8"，或 "depth24stencil8" 格式，其中深度方面会被隐藏，且无法访问。

**注意：** 虽然对于可表示范围(0.0 到 1.0)内的所有值，depth32float 通道的精度会严格高于24位深度通道的精度，但请注意，可表示值的集合不是一个精确的超集。
    * 对于 24位深度，1个 ULP 具有一个 1/（224-1）的常量值。
    * 对于 depth32float，1个 ULP 的可变值不大于 1 / (224)。

> The stencil8 format may be implemented as either a real "stencil8", or "depth24stencil8", where the depth aspect is hidden and inaccessible.

>> **Note:** While the precision of depth32float channels is strictly higher than the precision of 24-bit depth channels for all values in the representable range (0.0 to 1.0), note that the set of representable values is not an exact superset.
>> * For 24-bit depth, 1 ULP has a constant value of 1 / (224 − 1).
>>    
>> * For depth32float, 1 ULP has a variable value no greater than 1 / (224).

一个可渲染格式，要么是彩色可渲染格式，要么是深度/模板格式。如果一个格式被列在第26.1.1节中，即在 具有RENDER_ATTACHMENT功能的 平面颜色格式列表中，它就是一个可渲染的颜色格式。任何其他格式都不是彩色可渲染格式。所有深度/模板格式都是可渲染的。

> A renderable format is either a color renderable format, or a depth-or-stencil format. If a format is listed in § 26.1.1 Plain color formats with RENDER_ATTACHMENT capability, it is a color renderable format. Any other format is not a color renderable format. All depth-or-stencil formats are renderable.

**GPUTextureAspect(format, aspect)**    
**参数：**    
    * GPUTextureFormat format   
    * GPUTextureAspect aspect   

**返回值：** GPUTextureFormat或null, 为Promise    

>> **resolving GPUTextureAspect(format, aspect)**    
>> **Arguments:**    
>> * GPUTextureFormat format
>> * GPUTextureAspect aspect
>> **Returns:** GPUTextureFormat or null

1. 如果 aspect 是：
    **"all"**   
    返回 format。
    **"depth-only"**    
    **"stencil-only"**    
    如果format是一个深度/模板格式：根据§26.1.2 深度/模板格式，返回格式的 特定方面格式，如果格式中不存在该方面，则返回null。

2. 返回null。

>> 1. If aspect is:
>>    
>> **"all"**    
>>        Return format.
>>    
>> **"depth-only"**    
>> **"stencil-only"**    
>>        If format is a depth-stencil-format: Return the aspect-specific format of format according to § 26.1.2 Depth-stencil formats or null if the aspect is not present in format.
>>    
>> 2. Return null.

使用某些纹理格式需要在GPUDevice上启用一个功能。因为新的格式可以被添加到规范中，这些枚举值可能不被具体的实现者所知。为了使不同实现的行为正常化，一致化，如果设备上没有启用相关的功能，试图使用某种功能的格式将抛出一个异常。这使得不同具体的实现的行为，在不知道该格式时表现相同。

关于哪些 GPUTextureFormats 需要特征的信息，请参见第26.1节Texture Format Capabilities。

> Use of some texture formats require a feature to be enabled on the GPUDevice. Because new formats can be added to the specification, those enum values may not be known by the implementation. In order to normalize behavior across implementations, attempting to use a format that requires a feature will throw an exception if the associated feature is not enabled on the device. This makes the behavior the same as when the format is unknown to the implementation.

> See § 26.1 Texture Format Capabilities for information about which GPUTextureFormats require features.

**验证纹理格式要求**    
通过逻辑设备的 device 行以下步骤 来验证一个 GPUTextureFormat 格式的纹理格式要求的特征：
1. 如果格式需要一个特征而 device.[[features]]不包含该特征：
    1. 抛出一个TypeError。

>> **Validate texture format required** features of a GPUTextureFormat format with logical device device by running the following steps:
>> 1. If format requires a feature and device.[[features]] does not contain the feature:
>>     1. Throw a TypeError.


## 6.4. GPUExternalTexture

> 6.4. GPUExternalTexture

> A GPUExternalTexture is a sampleable 2D texture wrapping an external video object. The contents of a GPUExternalTexture object are a snapshot and may not change, either from inside WebGPU (it is only sampleable) or from outside WebGPU (e.g. due to video frame advancement).

> They are bound into bind group layouts using the externalTexture bind group layout entry member. External textures use several binding slots: see Exceeds the binding slot limits.


>> **Note:** External textures can be implemented without creating a copy of the imported source, but this depends implementation-defined factors. Ownership of the underlying representation may either be exclusive or shared with other owners (such as a video decoder), but this is not visible to the application.


>> The underlying representation of an external texture is unobservable (except for sampling behavior) but typically may include
>> * Up to three 2D planes of data (e.g. RGBA, Y+UV, Y+U+V).
>> * Metadata for converting coordinates before reading from those planes (crop and rotation).
>> * Metadata for converting values into the specified output color space (matrices, gammas, 3D LUT).

>> The configuration used may not be stable across time, systems, user agents, media sources, or frames within a single video source. In order to account for many possible representations, the binding conservatively uses the following, for each external texture:
>> * three sampled texture bindings (for up to 3 planes),
>> * one sampled texture binding for a 3D LUT,
>> * one sampler binding to sample the 3D LUT, and
>> * one uniform buffer binding for metadata.

```
[Exposed=(Window, DedicatedWorker), SecureContext]
interface GPUExternalTexture {
};
GPUExternalTexture includes GPUObjectBase;
```

> GPUExternalTexture has the following internal slots:

> **[[expired]], of type boolean**    
> Indicates whether the object has expired (can no longer be used). Initially set to false.
> 
> **NOTE:** Unlike similar [[destroyed]] slots, this can change from true back to false.
>     
> **[[descriptor]], of type GPUExternalTextureDescriptor**    
> The descriptor with which the texture was created.


### 6.4.1. Importing External Textures

> 6.4.1. Importing External Textures

> An external texture is created from an external video object using importExternalTexture().
>    
> An external texture created from an HTMLVideoElement is destroyed automatically in a task after it is imported, instead of manually or upon garbage collection like other resources. When an external texture expires, its [[expired]] slot changes to true.

> Once the GPUExternalTexture expires, importExternalTexture() must be called again. However, the user agent may un-expire and return the same GPUExternalTexture again, instead of creating a new one. This will commonly happen unless the execution of the application is scheduled to match the video’s frame rate (e.g. using requestVideoFrameCallback()). If the same object is returned again, it will compare equal, and GPUBindGroups, GPURenderBundles, etc. referencing the previous object can still be used.

```
dictionary GPUExternalTextureDescriptor : GPUObjectDescriptorBase {
    required HTMLVideoElement source;
    PredefinedColorSpace colorSpace = "srgb";
};
```

> **importExternalTexture(descriptor)**   
> Creates a GPUExternalTexture wrapping the provided image source.
>   
> **Called on:** GPUDevice this.
> **Arguments:**    


| Parameter | Type | Nullable | Optional | Description |
| -------- | -------- | -------- | -------- | -------- |
| descriptor | GPUExternalTextureDescriptor | ✘ | ✘ | Provides the external image source object (and any creation options). |

> **Returns:** GPUExternalTexture   
>   
> **Content timeline** steps:
> 1. Let source be descriptor.source.
> 2. If the current image contents of source are the same as the most recent importExternalTexture() call with the same descriptor (ignoring label), and the user agent chooses to reuse it:
>     1. Let previousResult be the GPUExternalTexture returned previously.
>     2. Set previousResult.[[expired]] to false, renewing ownership of the underlying resource.
>     3. Let result be previousResult.
>      
>     **NOTE:** This allows the application to detect duplicate imports and avoid re-creating dependent objects (such as GPUBindGroups). Implementations still need to be able to handle a single frame being wrapped by multiple GPUExternalTexture, since import metadata like colorSpace can change even for the same frame.
>       
>     Otherwise:
>     1. If source is not origin-clean, throw a SecurityError and stop.
>     2. Let usability be ? check the usability of the image argument(source).
>     3. If usability is not good:
>         1. Generate a validation error.
>         2. Return an invalid GPUExternalTexture.
>     4. Let data be the result of converting the current image contents of source into the color space descriptor.colorSpace with unpremultiplied alpha.
>   
>           This may result in values outside of the range [0, 1]. If clamping is desired, it may be performed after sampling.
>       
>           **NOTE:** This is described like a copy, but may be implemented as a reference to read-only underlying data plus appropriate metadata to perform conversion later.
>     5. Let result be a new GPUExternalTexture object wrapping data.

> 3. queue an automatic expiry task with device this and the following steps:
>>    1. Set result.[[expired]] to true, releasing ownership of the underlying resource.
>>    
>>    **NOTE:** An external video texture should be imported in the same task that samples the texture (which should generally be scheduled using requestVideoFrameCallback or requestAnimationFrame() depending on the application). Otherwise, a texture could get destroyed by these steps before it is used.
> 4. Set result.label to descriptor.label.
> 5. Return result.

```
EXAMPLE 15
// Rendering using an video element external texture at the page animation frame rate:

const videoElement = document.createElement('video');

// ... set up videoElement, wait for it to be ready...

let externalTexture;

function frame() {
    requestAnimationFrame(frame);

    // Re-import only if necessary
    if (!externalTexture || externalTexture.expired) {
        externalTexture = gpuDevice.importExternalTexture({
            source: videoElement
        });
    }

    // ... render using externalTexture...
}
requestAnimationFrame(frame);
```

```
EXAMPLE 16

// Rendering using an video element external texture at the video’s frame rate, if requestVideoFrameCallback is available:

const videoElement = document.createElement('video');

// ... set up videoElement...

function frame() {
    videoElement.requestVideoFrameCallback(frame);

    // Always re-import, because we know the video frame has advanced
    const externalTexture = gpuDevice.importExternalTexture({
        source: videoElement
    });

    // ... render using externalTexture...
}
videoElement.requestVideoFrameCallback(frame);
```

### 6.4.2. Sampling External Textures

> 6.4.2. Sampling External Textures

> External textures are represented in WGSL with texture_external and may be read using textureLoad and textureSampleBaseClampToEdge.

> The sampler provided to textureSampleBaseClampToEdge is used to sample the underlying textures. The result is in the color space set by colorSpace. It is implementation-dependent whether, for any given external texture, the sampler (and filtering) is applied before or after conversion from underlying values into the specified color space.

> **NOTE:** If the internal representation is an RGBA plane, sampling behaves as on a regular 2D texture. If there are several underlying planes (e.g. Y+UV), the sampler is used to sample each underlying texture separately, prior to conversion from YUV to the specified color space.
