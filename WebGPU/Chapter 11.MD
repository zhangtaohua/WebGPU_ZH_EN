# WebGPU

W3C Working Draft, 28 March 2023 

More details about this document

This version:

https://www.w3.org/TR/2023/WD-webgpu-20230328/

---
---

# 11. 复制

## 11.1. 缓冲区复制

缓冲区复制操作是对原始字节的操作。

WebGPU 提供 "具有缓冲特性的" GPUCommandEncoder 命令：
* copyBufferToBuffer()
* clearBuffer()

以及 "即时" GPUQueue 操作：
* writeBuffer()，用于 ArrayBuffer 到 GPUBuffer 的写入操作。

---
11\. Copies

11.1. Buffer Copies

Buffer copy operations operate on raw bytes.

WebGPU provides "buffered" GPUCommandEncoder commands:
* copyBufferToBuffer()
* clearBuffer()

and "immediate" GPUQueue operations:
* writeBuffer(), for ArrayBuffer-to-GPUBuffer writes

---
---

## 11.2. 图像复制

图像复制操作是针对对 纹理/"图像"数据，而不是以字节为单位进行操作。

WebGPU 提供了 "具有缓冲特性的" GPUCommandEncoder 命令：
* copyTextureToTexture()
* copyBufferToTexture()
* copyTextureToBuffer()

和 "即时" GPUQueue 操作：
* writeTexture()，用于从 ArrayBuffer 到 GPUTexture 的写入数据。
* copyExternalImageToTexture()，用于从网络平台图像源复制数据到纹理。

一些 texel 值有多种可能的表示方法，例如，作为 r8snorm 格式，-1.0 可以表示为 -127 或 -128 。复制命令不能保证能够保留源数据的位表示格式。

以下章节中的定义使用了这些方法。

---

11.2. Image Copies

Image copy operations operate on texture/"image" data, rather than bytes.

WebGPU provides "buffered" GPUCommandEncoder commands:
* copyTextureToTexture()
* copyBufferToTexture()
* copyTextureToBuffer()

and "immediate" GPUQueue operations:
* writeTexture(), for ArrayBuffer-to-GPUTexture writes
* copyExternalImageToTexture(), for copies from Web Platform image sources to textures

Some texel values have multiple possible representations of some values, e.g. as r8snorm, -1.0 can be represented as either -127 or -128. Copy commands are not guaranteed to preserve the source’s bit-representation.

The following definitions are used by these methods.

---
---

11.2.1. GPU 图片数据布局

```
dictionary GPUImageDataLayout {
    GPUSize64 offset = 0;
    GPUSize32 bytesPerRow;
    GPUSize32 rowsPerImage;
};
```

GPUImageDataLayout 是一种图像布局，其内存地址是线性连续的。当在纹理和 GPUBuffer 之间复制数据时，或者当从 GPUQueue 写入数据到纹理时，就会用到它。
* 对于2D纹理，数据是在一个或多个连续的图像和阵列层之间复制的。
* 对于3D纹理，数据是在一个或多个连续的图像和深度片层之间复制的。

>> ISSUE 17 定义更精确地图像，特别是，将它们定义为由 texel 块组成。

在字节数组和纹理之间的复制操作总是与 texel 块的行数据一起工作，我们称之为块行(block rows)。只更新一个 texel 块的一部分是不可能的。

在图像拷贝的线性内存布局中，texel 块被紧密地包装在每个块行中，即每个后续的 texel 块紧跟在前一个 texel 块之后，没有任何填充数据。这包括对深度或模板格式纹理的特定方面的拷贝：模板值被紧密地包装在一个字节数组中；深度值被紧密地包装在一个适当类型的数组中（类型可能为 "depth16unorm" 或 "depth32float"）。

---

11.2.1. GPUImageDataLayout

```
dictionary GPUImageDataLayout {
    GPUSize64 offset = 0;
    GPUSize32 bytesPerRow;
    GPUSize32 rowsPerImage;
};
```

A GPUImageDataLayout is a layout of images within some linear memory. It’s used when copying data between a texture and a GPUBuffer, or when scheduling a write into a texture from the GPUQueue.
* For 2d textures, data is copied between one or multiple contiguous images and array layers.
* For 3d textures, data is copied between one or multiple contiguous images and depth slices.

>> ISSUE 17 Define images more precisely. In particular, define them as being comprised of texel blocks.

Operations that copy between byte arrays and textures always work with rows of texel blocks, which we’ll call block rows. It’s not possible to update only a part of a texel block.

Texel blocks are tightly packed within each block row in the linear memory layout of an image copy, with each subsequent texel block immediately following the previous texel block, with no padding. This includes copies to/from specific aspects of depth-or-stencil format textures: stencil values are tightly packed in an array of bytes; depth values are tightly packed in an array of the appropriate type ("depth16unorm" or "depth32float").

---
---

>> ISSUE 18 通过参考复制方法所共享的共同算法，定义更加准确的复制语义。

**offset，类型为： GPUSize64，默认为 0**        
偏移量，以字节计数，表示从图像数据源（例如 GPUImageCopyBuffer.buffer）开始的地址 到该数据源中真正的图像数据的开始地址的偏移量。

**bytesPerRow, 类型为： GPUSize32**      
表示在每个区块行的开始地址与随后的区块行之间地址的跨度，单位是字节。

如果有多个块行则需要（即复制高度或深度超过一个块）。

**rowsPerImage，类型为GPUSize32**       
纹理的每一个图像的块行数量。 rowsPerImage × bytesPerRow 是指每个数据图像的开始地址与随后的图像地址之间的跨度，单位是字节。

如果有多个图像则需要（即复制的深度超过一个）。

---

>> ISSUE 18 Define the exact copy semantics, by reference to common algorithms shared by the copy methods.

**offset, of type GPUSize64, defaulting to 0**      
The offset, in bytes, from the beginning of the image data source (such as a GPUImageCopyBuffer.buffer) to the start of the image data within that source.

**bytesPerRow, of type GPUSize32**      
The stride, in bytes, between the beginning of each block row and the subsequent block row.

Required if there are multiple block rows (i.e. the copy height or depth is more than one block).

**rowsPerImage, of type GPUSize32**     
Number of block rows per single image of the texture. rowsPerImage × bytesPerRow is the stride, in bytes, between the beginning of each image of data and the subsequent image.

Required if there are multiple images (i.e. the copy depth is more than one).

---
---

### 11.2.2. GPU 图像复制缓冲区

在图像复制操作中，GPUImageCopyBuffer 定义了一个 GPUBuffer，并与 copySize 一起定义了图像数据在缓冲区内存中的布局方式（参见 GPUImageDataLayout）。

```
dictionary GPUImageCopyBuffer : GPUImageDataLayout {
    required GPUBuffer buffer;
};
```

**buffer，类型为： GPUBuffer**       
一个缓冲区，它要么包含要复制的图像数据，要么将存储正在复制的图像数据，这取决于传递给它的方法。

---

11.2.2. GPUImageCopyBuffer

In an image copy operation, GPUImageCopyBuffer defines a GPUBuffer and, together with the copySize, how image data is laid out in the buffer’s memory (see GPUImageDataLayout).

```
dictionary GPUImageCopyBuffer : GPUImageDataLayout {
    required GPUBuffer buffer;
};
```

**buffer, of type GPUBuffer**     
A buffer which either contains image data to be copied or will store the image data being copied, depending on the method it is being passed to.

---
---

**验证 GPUImageCopyBuffer**     

**参数：**      
* GPUImageCopyBuffer 图像复制缓冲区

**返回：** 布尔值

1. 当且仅当以下所有条件都得到满足时，返回 true：
* imageCopyBuffer.buffer 必须是一个有效的 GPUBuffer。
* imageCopyBuffer.bytesPerRow 必须是 256 的倍数。

---
**validating GPUImageCopyBuffer**         

**Arguments:**    
* GPUImageCopyBuffer imageCopyBuffer

**Returns:** boolean

1. Return true if and only if all of the following conditions are satisfied:
    * imageCopyBuffer.buffer must be a valid GPUBuffer.
    * imageCopyBuffer.bytesPerRow must be a multiple of 256.

---
---

### 11.2.3. GPUImageCopyTexture

在图像复制操作中，GPUImageCopyTexture 定义了一个 GPUTexture，并与 copySize 一起定义了纹理的子区域（横跨同一 mip-map 级别的一个或多个连续的纹理子资源）。

```
dictionary GPUImageCopyTexture {
    required GPUTexture texture;
    GPUIntegerCoordinate mipLevel = 0;
    GPUOrigin3D origin = {};
    GPUTextureAspect aspect = "all";
};
```

**texture，类型为： GPUTexture**        
需要复制 到/从 的纹理

**mipLevel，类型为： GPUIntegerCoordinate，默认为0。**     
需要复制 到/从 的纹理的 Mip-map 级别。

**origin，类型为： GPUOrigin3D，默认为{}。**     
定义进行复制操作数据的原点位置，即要复制 到/从 的纹理子区域的最小角。与 copySize 一起，定义了完整的复制子区域。

**aspect，类型为： GPUTextureAspect，默认为 "all"。**     
定义纹理的哪些方面需要复制 到/从。

---

11.2.3. GPUImageCopyTexture

In an image copy operation, a GPUImageCopyTexture defines a GPUTexture and, together with the copySize, the sub-region of the texture (spanning one or more contiguous texture subresources at the same mip-map level).

```
dictionary GPUImageCopyTexture {
    required GPUTexture texture;
    GPUIntegerCoordinate mipLevel = 0;
    GPUOrigin3D origin = {};
    GPUTextureAspect aspect = "all";
};
```

**texture, of type GPUTexture**         
Texture to copy to/from.

**mipLevel, of type GPUIntegerCoordinate, defaulting to 0**         
Mip-map level of the texture to copy to/from.

**origin, of type GPUOrigin3D, defaulting to {}**           
Defines the origin of the copy - the minimum corner of the texture sub-region to copy to/from. Together with copySize, defines the full copy sub-region.

**aspect, of type GPUTextureAspect, defaulting to "all"**       
Defines which aspects of the texture to copy to/from.

---
---

**验证 GPUImageCopyTexture**     

**参数：**      
* GPUImageCopyTexture 图像复制纹理
* GPUExtent3D 复制大小 

**返回：** 布尔值

1. 让 blockWidth 成为 imageCopyTexture.texture.format 的 texel块 的宽度。
2. 让 blockHeight 成为 imageCopyTexture.texture.format 的 texel块 的高度。
3. 当且仅当以下所有条件都适用时，返回true：
    * imageCopyTexture.texture 必须是一个有效的 GPUTexture。
    * imageCopyTexture.mipLevel 必须小于 imageCopyTexture.texture.mipLevelCount。
    * imageCopyTexture.origin.x 必须是 blockWidth 的倍数。
    * imageCopyTexture.origin.y 必须是 blockHeight 的倍数。
    * 如果以下任一条件为真，imageCopyTexture 的 子资源大小 等于 copySize：
        * imageCopyTexture.texture.format 是一种深度/模板格式。
        * imageCopyTexture.texture.sampleCount > 1。

>> ISSUE 19 用 1d 和 3d 纹理 来定义纹理。[问题#gpuweb/gpuweb#69] 。

---

**validating GPUImageCopyTexture**          

**Arguments:**      
* GPUImageCopyTexture imageCopyTexture
* GPUExtent3D copySize

**Returns:** boolean
1. Let blockWidth be the texel block width of imageCopyTexture.texture.format.
2. Let blockHeight be the texel block height of imageCopyTexture.texture.format.
3. Return true if and only if all of the following conditions apply:
    * imageCopyTexture.texture must be a valid GPUTexture.
    * imageCopyTexture.mipLevel must be < imageCopyTexture.texture.mipLevelCount.
    * imageCopyTexture.origin.x must be a multiple of blockWidth.
    * imageCopyTexture.origin.y must be a multiple of blockHeight.
    * The imageCopyTexture subresource size of imageCopyTexture is equal to copySize if either of the following conditions is true:
        * imageCopyTexture.texture.format is a depth-stencil format.
        * imageCopyTexture.texture.sampleCount > 1.

>> ISSUE 19 Define the copies with 1d and 3d textures. [Issue #gpuweb/gpuweb#69]

---
---

#### 11.2.4. GPUImageCopyTextureTagged

WebGPU 纹理持有原始数字数据，且并没有描述颜色的语义元数据的标记。然而，copyExternalImageToTexture() 可以从描述颜色的源数据中执行复制操作。

GPUImageCopyTextureTagged 是一个 GPUImageCopyTexture，且它被额外标记了色彩空间/编码 以及 alpha-remultiplication 元数据，因此在复制过程中可以保留语义色彩数据。这个元数据只影响 copyExternalImageToTexture() 操作的语义，而不影响目标纹理的语义。

---

11.2.4. GPUImageCopyTextureTagged

WebGPU textures hold raw numeric data, and are not tagged with semantic metadata describing colors. However, copyExternalImageToTexture() copies from sources that describe colors.

A GPUImageCopyTextureTagged is a GPUImageCopyTexture which is additionally tagged with color space/encoding and alpha-premultiplication metadata, so that semantic color data may be preserved during copies. This metadata affects only the semantics of the copyExternalImageToTexture() operation, not the semantics of the destination texture.

---
---

```
dictionary GPUImageCopyTextureTagged : GPUImageCopyTexture {
    PredefinedColorSpace colorSpace = "srgb";
    boolean premultipliedAlpha = false;
};
```

**colorSpace，类型为： PredefinedColorSpace，默认为 "srgb"。**         
描述了用于将原始数据编码到目标纹理的颜色空间和编码方式。

这可能会导致超出 [0, 1] 范围的值被写入目标纹理，如果它的格式可以表示它们的话。否则，结果数值会被钳制到目标纹理格式的范围内。

**注意：** 如果 colorSpace 与 源图像格式相匹配，可能不需要转换。参见 § 3.10.2 颜色空间转换消除。

---

```
dictionary GPUImageCopyTextureTagged : GPUImageCopyTexture {
    PredefinedColorSpace colorSpace = "srgb";
    boolean premultipliedAlpha = false;
};
```

**colorSpace, of type PredefinedColorSpace, defaulting to "srgb"**          
Describes the color space and encoding used to encode data into the destination texture.

This may result in values outside of the range [0, 1] being written to the target texture, if its format can represent them. Otherwise, the results are clamped to the target texture format’s range.

**NOTE:** If colorSpace matches the source image, conversion may not be necessary. See § 3.10.2 Color Space Conversion Elision.

---
---

**premultipliedAlpha，类型为： 布尔值，默认为 false。**
描述了写入纹理中的数据是否要将RGB通道数据分别预乘以 alpha 通道的值。

如果这个选项设置为 "true"，并且源数据也被预乘了，那么即使源的 RGB 值超过了相应的alpha值，也必须被保留下来。

**注意：** 如果 premultipliedAlpha 与源图像相匹配，可能不需要转换。参见 § 3.10.2 颜色空间转换消除。

---

**premultipliedAlpha, of type boolean, defaulting to false**        
Describes whether the data written into the texture should have its RGB channels premultiplied by the alpha channel, or not.

If this option is set to true and the source is also premultiplied, the source RGB values must be preserved even if they exceed their corresponding alpha values.

**NOTE:** If premultipliedAlpha matches the source image, conversion may not be necessary. See § 3.10.2 Color Space Conversion Elision.

---
---

### 11.2.5. GPUImageCopyExternalImage

```
dictionary GPUImageCopyExternalImage {
    required (ImageBitmap or HTMLVideoElement or HTMLCanvasElement or OffscreenCanvas) source;
    GPUOrigin2D origin = {};
    boolean flipY = false;
};
```

GPUImageCopyExternalImage 有以下成员：

**source，类型为： （ImageBitmap 或 HTMLVideoElement 或 HTMLCanvasElement 或 OffscreenCanvas）。**       
图像拷贝的来源。在 copyExternalImageToTexture() 执行的那一刻，要复制源数据被捕获。源数据的大小是由源数据的类型定义的，由下面这个表格给出：

| 源类型 | 宽度 | 高度 |
| --- | --- | --- |
| ImageBitmap | ImageBitmap.width | ImageBitmap.height |
| HTMLVideoElement | intrinsic width of the frame | intrinsic height of the frame |
| HTMLCanvasElement | HTMLCanvasElement.width | HTMLCanvasElement.height |
| OffscreenCanvas | OffscreenCanvas.width | OffscreenCanvas.height |

---

11.2.5. GPUImageCopyExternalImage

```
dictionary GPUImageCopyExternalImage {
    required (ImageBitmap or HTMLVideoElement or HTMLCanvasElement or OffscreenCanvas) source;
    GPUOrigin2D origin = {};
    boolean flipY = false;
};
```

GPUImageCopyExternalImage has the following members:

**source, of type (ImageBitmap or HTMLVideoElement or HTMLCanvasElement or OffscreenCanvas)**           
The source of the image copy. The copy source data is captured at the moment that copyExternalImageToTexture() is issued. Source size is defined by source type, given by this table:

| Source type | Width | Height |
| --- | --- | --- |
| ImageBitmap | ImageBitmap.width | ImageBitmap.height |
| HTMLVideoElement | intrinsic width of the frame | intrinsic height of the frame |
| HTMLCanvasElement | HTMLCanvasElement.width | HTMLCanvasElement.height |
| OffscreenCanvas | OffscreenCanvas.width | OffscreenCanvas.height |

---
---

**origin，类型为： GPUOrigin2D，默认为{}。**       
定义要复制的数据的原点，即源数据子区域的最小左上角，以便从中进行复制。与 copySize 一起，定义了完整的要复制的子区域。

**flipY，类型为： 布尔值，默认为false**        
描述源图像是否被垂直翻转。

如果该选项设置为 "true"，则复制是垂直翻转的：即源区域的最下面一行被复制到目标区域的第一行，以此类推。 原点选项仍然是相对于源图像的左上角，且向下递增。

---

**origin, of type GPUOrigin2D, defaulting to {}**       
Defines the origin of the copy - the minimum (top-left) corner of the source sub-region to copy from. Together with copySize, defines the full copy sub-region.

**flipY, of type boolean, defaulting to false**         
Describes whether the source image is vertically flipped, or not.

If this option is set to true, the copy is flipped vertically: the bottom row of the source region is copied into the first row of the destination region, and so on. The origin option is still relative to the top-left corner of the source image, increasing downward.

---
---

### 11.2.6.子程序

**计算 imageCopyTexture 子资源大小 的函数**    

**参数：**
* GPUImageCopyTexture imageCopyTexture

**返回：** GPUExtent3D

imageCopyTexture 的 imageCopyTexture子资源的大小计算方法如下：
* 它的宽度、高度和 depthOrArrayLayers 分别是 imageCopyTexture.texture 子资源在 mipmap 级别中的 imageCopyTexture.mipLevel 的 物理层面上特定的miplevle级别纹理范围所对应的 宽度、高度和深度。

---
11.2.6. Subroutines

**imageCopyTexture subresource size**

**Arguments:**
* GPUImageCopyTexture imageCopyTexture

**Returns:** GPUExtent3D

The imageCopyTexture subresource size of imageCopyTexture is calculated as follows:

Its width, height and depthOrArrayLayers are the width, height, and depth, respectively, of the physical miplevel-specific texture extent of imageCopyTexture.texture subresource at mipmap level imageCopyTexture.mipLevel.

---
---

**验证 线性纹理数据(layout, byteSize, format, copyExtent)。**       
**参数：**      
* GPUImageDataLayout 布局             
表示线性纹理数据的布局。

* GPUSize64 字节大小        
表示线性数据的总大小，单位是字节。

* GPUTextureFormat 格式     
表示纹理的格式。

* GPUExtent3D 复制范围      
表示要复制的纹理的范围。

---

**validating linear texture data(layout, byteSize, format, copyExtent)**                

**Arguments:**          
**GPUImageDataLayout layout**       
Layout of the linear texture data.      

**GPUSize64 byteSize**          
Total size of the linear data, in bytes.            

**GPUTextureFormat format**         
Format of the texture.              

**GPUExtent3D copyExtent**      
Extent of the texture to copy.         

---
---

1. 让：
    * widthInBlocks 是 copyExtent.width 除于 格式的 texel块的宽度。 并断言其最终值为整数。
    * heightInBlocks 是 copyExtent.height 除于 格式的 texel块的高度。 并断言其最终值为整数。
    * bytesInLastRow 是 widthInBlocks 乘以 格式的 texel块复制范围。
2. 如果不符合以下输入验证要求，则失败：
    * 如果 heightInBlocks > 1，必须指定 layout.bytesPerRow。
    * 如果 copyExtent.depthOrArrayLayers > 1，必须指定 layout.bytesPerRow 和 layout.rowsPerImage。
    * 如果指定了，那么 layout.bytesPerRow 必须大于等于 bytesInLastRow。
    * 如果指定了，那么 layout.rowsPerImage 必须大于等于 heightInBlocks。

3. 让：
    * bytesPerRow 是 layout.bytesPerRow ?？ 0.
    * rowsPerImage 是 layout.rowsPerImage ?? 0.
    **注意：** 这些默认值没有影响，因为它们总是乘以 0。
4. 让 requiredBytesInCopy 为 0。
5. 如果 copyExtent.depthOrArrayLayers > 0：
    1. 将 requiredBytesInCopy 递增为 bytesPerRow × rowsPerImage × (copyExtent.depthOrArrayLayers - 1) 。
    2. 如果 heightInBlocks > 0：
        1. 将 requiredBytesInCopy 递增为 bytesPerRow × (heightInBlocks - 1) + bytesInLastRow。 
6. 如果不满足以下条件则失败：
    * 布局要适合内部的线性数据： layout.offset + requiredBytesInCopy ≤ byteSize。

---

1. Let:
    * widthInBlocks be copyExtent.width ÷ the texel block width of format. Assert this is an integer.
    * heightInBlocks be copyExtent.height ÷ the texel block height of format. Assert this is an integer.
    * bytesInLastRow be widthInBlocks × the texel block copy footprint of format.

2. Fail if the following input validation requirements are not met:
    * If heightInBlocks > 1, layout.bytesPerRow must be specified.
    * If copyExtent.depthOrArrayLayers > 1, layout.bytesPerRow and layout.rowsPerImage must be specified.
    * If specified, layout.bytesPerRow must be ≥ bytesInLastRow.
    * If specified, layout.rowsPerImage must be ≥ heightInBlocks.

3. Let:
    * bytesPerRow be layout.bytesPerRow ?? 0.
    * rowsPerImage be layout.rowsPerImage ?? 0.     
    **NOTE:** These default values have no effect, as they’re always multiplied by 0.
4. Let requiredBytesInCopy be 0.
5. If copyExtent.depthOrArrayLayers > 0:
    1. Increment requiredBytesInCopy by bytesPerRow × rowsPerImage × (copyExtent.depthOrArrayLayers − 1).
    2. If heightInBlocks > 0:
        1. Increment requiredBytesInCopy by bytesPerRow × (heightInBlocks − 1) + bytesInLastRow.
6. Fail if the following condition is not satisfied:
    * The layout fits inside the linear data: layout.offset + requiredBytesInCopy ≤ byteSize.

---
---

**validating texture copy range**       

**验证纹理复制范围**

**参数：**      
* GPUImageCopyTexture imageCopyTexture      
被复制到的纹理子资源和复制原点。

* GPUExtent3D 复制大小      
表示纹理的大小。

---

**validating texture copy range**       

**Arguments:**          

**GPUImageCopyTexture imageCopyTexture**        
    The texture subresource being copied into and copy origin.

**GPUExtent3D copySize**        
The size of the texture.  

---
---

1. 让 blockWidth 成为 imageCopyTexture.texture.format 的 texel块 宽度。
2. 让 blockHeight 成为 imageCopyTexture.texture.format 的 texel块 高度。
3. 让 subresourceSize 是 图像复制纹理 的 imageCopyTexture 子资源的大小。
4. 返回，是否满足以下所有的条件：
    * (imageCopyTexture.origin.x + copySize.width) ≤ subresourceSize.width
    * (imageCopyTexture.origin.y + copySize.height) ≤ subresourceSize.height
    * (imageCopyTexture.origin.z + copySize.depthOrArrayLayers) ≤ subresourceSize.depthOrArrayLayers
    * copySize.width 必须是 blockWidth 的倍数。
    * copySize.height 必须是 blockHeight 的倍数。

---
1. Let blockWidth be the texel block width of imageCopyTexture.texture.format.
2. Let blockHeight be the texel block height of imageCopyTexture.texture.format.
3. Let subresourceSize be the imageCopyTexture subresource size of imageCopyTexture.
4. Return whether all the conditions below are satisfied:
   * (imageCopyTexture.origin.x + copySize.width) ≤ subresourceSize.width
   * (imageCopyTexture.origin.y + copySize.height) ≤ subresourceSize.height
   * (imageCopyTexture.origin.z + copySize.depthOrArrayLayers) ≤ subresourceSize.depthOrArrayLayers
   * copySize.width must be a multiple of blockWidth.
   * copySize.height must be a multiple of blockHeight.

---
---

两个 GPUTextureFormats 格式 format1 和 format2 在以下情况下是复制可兼容的：
* format1 等于 format2，或者
* format1 和 format2 只在是否为 srgb 格式上有区别 (即有 -srgb 后缀的格式)。

纹理复制的子资源集(imageCopyTexture, copySize) 函数的结果是 纹理的子资源的子集 ，其等于 每个满足以下条件的子资源 s 的 imageCopyTexture.texture ：
* s 的 mipmap 级别等于 imageCopyTexture.mipLevel。
* s 的 方面是在 imageCopyTexture.aspect 的方面集合中。
* 如果 texture.dimension 是 "2d"：
    * s 的阵列层 ≥ imageCopyTexture.origin.z 并且 < imageCopyTexture.origin.z + copySize.depthOrArrayLayers。

---

Two GPUTextureFormats format1 and format2 are copy-compatible if:
* format1 equals format2, or
* format1 and format2 differ only in whether they are srgb formats (have the -srgb suffix).

The set of subresources for texture copy(imageCopyTexture, copySize) is the subset of subresources of texture = imageCopyTexture.texture for which each subresource s satisfies the following:
* The mipmap level of s equals imageCopyTexture.mipLevel.
* The aspect of s is in the set of aspects of imageCopyTexture.aspect.
* If texture.dimension is "2d":
    * The array layer of s is ≥ imageCopyTexture.origin.z and < imageCopyTexture.origin.z + copySize.depthOrArrayLayers.
