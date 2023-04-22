WebGPU

W3C Working Draft, 18 April 2023

More details about this document

This version:

https://www.w3.org/TR/2023/WD-webgpu-20230418/

Latest published version:

https://www.w3.org/TR/webgpu/

Editor's Draft:

https://gpuweb.github.io/gpuweb/

Previous Versions:

https://www.w3.org/TR/2023/WD-webgpu-20230413/

---
---

# 25\. 特征索引

## 25.1. "深度-剪接-控制"

允许禁用深度剪裁功能。

这个特性增加了以下可选的API 表现值：
* 新的 GPUPrimitiveState 字典成员：
    * unclippedDepth

---

25\. Feature Index

25.1. "depth-clip-control"

Allows depth clipping to be disabled.

This feature adds the following optional API surfaces:
* New GPUPrimitiveState dictionary members:
    * unclippedDepth

---
---

## 25.2. "depth32float-stencil8"

允许显式创建格式为 "depth32float-stencil8" 的纹理。

这个特性增加了以下可选的API 表现值：
* 新的 GPUTextureFormat 枚举值：
    * "depth32float-stencil8"

---

25.2. "depth32float-stencil8"

Allows for explicit creation of textures of format "depth32float-stencil8".

This feature adds the following optional API surfaces:
* New GPUTextureFormat enum values:
    * "depth32float-stencil8"

---
---

## 25.3. "texture-compression-bc"

允许明确的创建 BC 压缩格式的纹理。

这个特性增加了以下可选的API 表现值：
* 新的 GPUTextureFormat 枚举值：
    * "bc1-rgba-unorm"
    * "bc1-rgba-unorm-srgb"
    * "bc2-rgba-unorm"
    * "bc2-rgba-unorm-srgb"
    * "bc3-rgba-unorm"
    * "bc3-rgba-unorm-srgb"
    * "bc4-r-unorm"
    * "bc4-r-snorm"
    * "bc5-rg-unorm"
    * "bc5-rg-snorm"
    * "bc6h-rgb-ufloat"
    * "bc6h-rgb-float"
    * "bc7-rgba-unorm"
    * "bc7-rgba-unorm-srgb"

---

25.3. "texture-compression-bc"

Allows for explicit creation of textures of BC compressed formats.

This feature adds the following optional API surfaces:
* New GPUTextureFormat enum values:
    * "bc1-rgba-unorm"
    * "bc1-rgba-unorm-srgb"
    * "bc2-rgba-unorm"
    * "bc2-rgba-unorm-srgb"
    * "bc3-rgba-unorm"
    * "bc3-rgba-unorm-srgb"
    * "bc4-r-unorm"
    * "bc4-r-snorm"
    * "bc5-rg-unorm"
    * "bc5-rg-snorm"
    * "bc6h-rgb-ufloat"
    * "bc6h-rgb-float"
    * "bc7-rgba-unorm"
    * "bc7-rgba-unorm-srgb"

---
---

## 25.4. "texture-compression-etc2"

允许显式的创建 ETC2 压缩格式的纹理。

这个特性增加了以下可选的API 表现值：
* 新的 GPUTextureFormat 枚举值：
    * "etc2-rgb8unorm"
    * "etc2-rgb8unorm-srgb"
    * "etc2-rgb8a1unorm"
    * "etc2-rgb8a1unorm-srgb"
    * "etc2-rgba8unorm"
    * "etc2-rgba8unorm-srgb"
    * "eac-r11unorm"
    * "eac-r11snorm"
    * "eac-rg11unorm"
    * "eac-rg11snorm"

---

25.4. "texture-compression-etc2"

Allows for explicit creation of textures of ETC2 compressed formats.

This feature adds the following optional API surfaces:
* New GPUTextureFormat enum values:
    * "etc2-rgb8unorm"
    * "etc2-rgb8unorm-srgb"
    * "etc2-rgb8a1unorm"
    * "etc2-rgb8a1unorm-srgb"
    * "etc2-rgba8unorm"
    * "etc2-rgba8unorm-srgb"
    * "eac-r11unorm"
    * "eac-r11snorm"
    * "eac-rg11unorm"
    * "eac-rg11snorm"

---
---

## 25.5. "texture-compression-astc"

允许明确创建ASTC压缩格式的纹理。

这个特性增加了以下可选的API 表现值：
* 新的 GPUTextureFormat 枚举值：
    * "astc-4x4-unorm"
    * "astc-4x4-unorm-srgb"
    * "astc-5x4-unorm"
    * "astc-5x4-unorm-srgb"
    * "astc-5x5-unorm"
    * "astc-5x5-unorm-srgb"
    * "astc-6x5-unorm"
    * "astc-6x5-unorm-srgb"
    * "astc-6x6-unorm"
    * "astc-6x6-unorm-srgb"
    * "astc-8x5-unorm"
    * "astc-8x5-unorm-srgb"
    * "astc-8x6-unorm"
    * "astc-8x6-unorm-srgb"
    * "astc-8x8-unorm"
    * "astc-8x8-unorm-srgb"
    * "astc-10x5-unorm"
    * "astc-10x5-unorm-srgb"
    * "astc-10x6-unorm"
    * "astc-10x6-unorm-srgb"
    * "astc-10x8-unorm"
    * "astc-10x8-unorm-srgb"
    * "astc-10x10-unorm"
    * "astc-10x10-unorm-srgb"
    * "astc-12x10-unorm"
    * "astc-12x10-unorm-srgb"
    * "astc-12x12-unorm"
    * "astc-12x12-unorm-srgb"

---

25.5. "texture-compression-astc"

Allows for explicit creation of textures of ASTC compressed formats.

This feature adds the following optional API surfaces:
* New GPUTextureFormat enum values:
    * "astc-4x4-unorm"
    * "astc-4x4-unorm-srgb"
    * "astc-5x4-unorm"
    * "astc-5x4-unorm-srgb"
    * "astc-5x5-unorm"
    * "astc-5x5-unorm-srgb"
    * "astc-6x5-unorm"
    * "astc-6x5-unorm-srgb"
    * "astc-6x6-unorm"
    * "astc-6x6-unorm-srgb"
    * "astc-8x5-unorm"
    * "astc-8x5-unorm-srgb"
    * "astc-8x6-unorm"
    * "astc-8x6-unorm-srgb"
    * "astc-8x8-unorm"
    * "astc-8x8-unorm-srgb"
    * "astc-10x5-unorm"
    * "astc-10x5-unorm-srgb"
    * "astc-10x6-unorm"
    * "astc-10x6-unorm-srgb"
    * "astc-10x8-unorm"
    * "astc-10x8-unorm-srgb"
    * "astc-10x10-unorm"
    * "astc-10x10-unorm-srgb"
    * "astc-12x10-unorm"
    * "astc-12x10-unorm-srgb"
    * "astc-12x12-unorm"
    * "astc-12x12-unorm-srgb"

---
---

## 25.6. "timestamp-query"

增加了从 GPU 命令缓冲区查询时间戳的能力。参见 § 20.4 时间戳查询。

这个特性增加了以下可选的API 表现值：
* 新的 GPUQueryType 值：
    * "timestamp"

* 新的 GPUCommandEncoder 方法：
    * writeTimestamp()

* 新的 GPUComputePassDescriptor 成员：
    * TimestampWrites

* 新的 GPURenderPassDescriptor 成员：
    * timestampWrites

---

25.6. "timestamp-query"

Adds the ability to query timestamps from GPU command buffers. See § 20.4 Timestamp Query.

This feature adds the following optional API surfaces:
* New GPUQueryType values:
    * "timestamp"

* New GPUCommandEncoder methods:
    * writeTimestamp()

* New GPUComputePassDescriptor members:
    * timestampWrites

* New GPURenderPassDescriptor members:
    * timestampWrites

---
---

## 25.7. "indirect-first-instance"

允许在间接绘制参数 和 间接绘制索引参数中使用非零 firstInstance 值。

这个特性没有增加可选的 API 表现值。

---

25.7. "indirect-first-instance"

Allows the use of non-zero firstInstance values in indirect draw parameters and indirect drawIndexed parameters.

This feature adds no optional API surfaces.


---
---

## 25.8."shader-f16"

允许在 WGSL 中使用半精度浮点类型 f16。

这个特性增加了以下可选的API 表现值：

* 新的 WGSL 扩展：
    * f16

---

25.8. "shader-f16"

Allows the use of the half-precision floating-point type f16 in WGSL.

This feature adds the following optional API surfaces:
* New WGSL extensions:
    * f16

---
---

## 25.9. "rg11b10ufloat-renderable"

允许在格式为 "rg11b10ufloat" 的纹理上使用 RENDER_ATTACHMENT， 也允许该格式的纹理被混合和多采样。

这个特性没有增加任何可选的 API 表面值。

---

25.9. "rg11b10ufloat-renderable"

Allows the RENDER_ATTACHMENT usage on textures with format "rg11b10ufloat", and also allows textures of that format to be blended and multisampled.

This feature adds no optional API surfaces.

---
---

## 25.10. "bgra8unorm-storage"

允许在格式为 "bgra8unorm" 的纹理上使用 STORAGE_BINDING。

这个特性没有增加任何可选的 API 表面值。

---

25.10. "bgra8unorm-storage"

Allows the STORAGE_BINDING usage on textures with format "bgra8unorm".

This feature adds no optional API surfaces.

---
---

## 25.11. "float32-filterable"

使格式为 "r32float"、"rg32float" 和 "rgba32float" 的纹理可以被过滤。

---

25.11. "float32-filterable"

Makes textures with formats "r32float", "rg32float", and "rgba32float" filterable.
