# WebGPU

W3C Working Draft, 22 March 2023 

More details about this document

This version:

https://www.w3.org/TR/2023/WD-webgpu-20230322/

https://www.w3.org/TR/2023/WD-webgpu-20230322/#bind-group-creation

---
---

# 7\. 采样器

## 7.1.GPUSampler

GPUSampler 可以对 变换 和 过滤信息 进行编码，在着色器中使用时以便解释纹理资源数据。

GPUSamplers 是通过 createSampler() 函数创建的。
```
[Exposed=(Window, DedicatedWorker), SecureContext]
interface GPUSampler {
};
GPUSampler includes GPUObjectBase;
```

---

7\. Samplers

7.1. GPUSampler

A GPUSampler encodes transformations and filtering information that can be used in a shader to interpret texture resource data.

GPUSamplers are created via createSampler().
```
[Exposed=(Window, DedicatedWorker), SecureContext]
interface GPUSampler {
};
GPUSampler includes GPUObjectBase;
```
---
---

GPUSampler 有以下内部插槽：

**[[descriptor]], 类型为：GPUSamplerDescriptor, 只读**    
创建 GPUSampler 时使用的 GPUSamplerDescriptor。

**[[isComparison]], 类型为：boolean**    
创建的 GPUSampler 是否被用作 比较采样器。

**[[isFiltering]], 类型为：boolean**    
GPUSampler 是否对一个纹理的多个样本进行加权采样。

---

GPUSampler has the following internal slots:

**[[descriptor]], of type GPUSamplerDescriptor, readonly**      
The GPUSamplerDescriptor with which the GPUSampler was created.

**[[isComparison]], of type boolean**   
Whether the GPUSampler is used as a comparison sampler.

**[[isFiltering]], of type boolean**    
Whether the GPUSampler weights multiple samples of a texture.

---
---

## 7.2 创建采样器

### 7.2.1.GPUSamplerDescriptor

GPUSamplerDescriptor 用于指定创建一个 GPUSampler 的选项。

```
dictionary GPUSamplerDescriptor : GPUObjectDescriptorBase {
    GPUAddressMode addressModeU = "clamp-to-edge";
    GPUAddressMode addressModeV = "clamp-to-edge";
    GPUAddressMode addressModeW = "clamp-to-edge";
    GPUFilterMode magFilter = "nearest";
    GPUFilterMode minFilter = "nearest";
    GPUMipmapFilterMode mipmapFilter = "nearest";
    float lodMinClamp = 0;
    float lodMaxClamp = 32;
    GPUCompareFunction compare;
    [Clamp] unsigned short maxAnisotropy = 1;
};
```

---

7.2. Sampler Creation

7.2.1. GPUSamplerDescriptor

A GPUSamplerDescriptor specifies the options to use to create a GPUSampler.

```
dictionary GPUSamplerDescriptor : GPUObjectDescriptorBase {
    GPUAddressMode addressModeU = "clamp-to-edge";
    GPUAddressMode addressModeV = "clamp-to-edge";
    GPUAddressMode addressModeW = "clamp-to-edge";
    GPUFilterMode magFilter = "nearest";
    GPUFilterMode minFilter = "nearest";
    GPUMipmapFilterMode mipmapFilter = "nearest";
    float lodMinClamp = 0;
    float lodMaxClamp = 32;
    GPUCompareFunction compare;
    [Clamp] unsigned short maxAnisotropy = 1;
};
```

---
---

addressModeU，类型为： GPUAddressMode，默认为 "clamp-to-edge"。     
addressModeV，类型为： GPUAddressMode，默认为 "clamp-to-edge"。     
addressModeW，类型为： GPUAddressMode，默认为 "clamp-to-edge"。     
分别指定纹理宽度、高度和深度坐标的 地址模式。

magFilter，类型为： GPUFilterMode，默认为 "nearest"。
指定 当采样足迹小于或等于一个texel时的 采样行为。

minFilter，类型为： GPUFilterMode，默认为 "nearest"。
指定 当采样足迹大于一个texel时的 采样行为。

mipmapFilter，类型为： GPUMipmapFilterMode，默认为 "nearest"。
指定 mipmap级别之间的 采样行为。

---

> **addressModeU, of type GPUAddressMode, defaulting to "clamp-to-edge"**   
> **addressModeV, of type GPUAddressMode, defaulting to "clamp-to-edge"**   
> **addressModeW, of type GPUAddressMode, defaulting to "clamp-to-edge"**   
>     Specifies the address modes for the texture width, height, and depth coordinates, respectively.

> **magFilter, of type GPUFilterMode, defaulting to "nearest"**    
Specifies the sampling behavior when the sample footprint is smaller than or equal to one texel.

> **minFilter, of type GPUFilterMode, defaulting to "nearest"**    
Specifies the sampling behavior when the sample footprint is larger than one texel.

> **mipmapFilter, of type GPUMipmapFilterMode, defaulting to "nearest"**    
Specifies behavior for sampling between mipmap levels.

---
---

lodMinClamp，类型为： float，默认为0
lodMaxClamp，类型为： float，默认为32
指定对纹理进行采样时，内部使用的 最小和最大的细节级别。

compare，类型为： GPUCompareFunction。
当提供时，采样器将是一个具备指定的 GPUCompareFunction 函数的 比较采样器。

**注意：** 比较取样器可以用于过滤，但采样结果将取决于具体的实现，可能与正常的过滤规则不同。

maxAnisotropy，类型为：无符号短整形，默认为1
指定采样器使用的最大各向异性钳值。

**注意：** 大多数具体的实现支持的 maxAnisotropy值在1到16之间。maxAnisotropy的最终使用值将被钳制为平台所支持的最大值。

---

> **lodMinClamp, of type float, defaulting to 0**    
> **lodMaxClamp, of type float, defaulting to 32**    
> Specifies the minimum and maximum levels of detail, respectively, used internally when sampling a texture.

> **compare, of type GPUCompareFunction**    
> When provided the sampler will be a comparison sampler with the specified GPUCompareFunction.
>   
>   **NOTE:** Comparison samplers may use filtering, but the sampling results will be implementation-dependent and may differ from the normal filtering rules.

> **maxAnisotropy, of type unsigned short, defaulting to 1**    
> Specifies the maximum anisotropy value clamp used by the sampler.
> 
> **NOTE:** Most implementations support maxAnisotropy values in range between 1 and 16, inclusive. The used value of maxAnisotropy will be clamped to the maximum value that the platform supports.

---
---

>> ISSUE 7 解释 LOD 是如何计算的，以及不同平台之间是否存在差异。

>> ISSUE 8 解释什么是各向异性的采样。

GPUAddressMode 描述了 当采样足迹超出了被采样纹理的边界时，采样器的行为是什么样的。

>> ISSUE 8 更详细地描述一下"采样足迹"。

 ```
enum GPUAddressMode {
    "clamp-to-edge",
    "repeat",
    "mirror-repeat"
};
```

 ---

>> **ISSUE 7** explain how LOD is calculated and if there are differences here between platforms.

>> **ISSUE 8** explain what anisotropic sampling is

GPUAddressMode describes the behavior of the sampler if the sample footprint extends beyond the bounds of the sampled texture.

>> **ISSUE 9** Describe a "sample footprint" in greater detail.

```
enum GPUAddressMode {
    "clamp-to-edge",
    "repeat",
    "mirror-repeat"
};
```

---
---

"clamp-to-edge"
纹理坐标被钳制在 0.0和1.0 之间，包括边界值。

"repeat"
纹理坐标包络到纹理的另一边。

"mirror-repeat"
纹理坐标包络到纹理的另一边，但当坐标的整数部分为奇数时，纹理会被翻转。

GPUFilterMode 和 GPUMipmapFilterMode 描述了 当采样足迹不完全匹配一个texel时，采样器的行为。

---

**"clamp-to-edge"**    
Texture coordinates are clamped between 0.0 and 1.0, inclusive.

**"repeat"**    
Texture coordinates wrap to the other side of the texture.

**"mirror-repeat"**    
Texture coordinates wrap to the other side of the texture, but the texture is flipped when the integer part of the coordinate is odd.

GPUFilterMode and GPUMipmapFilterMode describe the behavior of the sampler if the sample footprint does not exactly match one texel.

---
---
```
enum GPUFilterMode {
    "nearest",
    "linear"
};

enum GPUMipmapFilterMode {
    "nearest",
    "linear"
};
```

"nearest"
返回与纹理坐标最接近的texel的值。

"linear"
在每个维度上选择两个texel值，并返回它们之间的线性插值。

GPUCompareFunction 函数指定了一个比较采样器的行为。如果在着色器中使用了比较采样器，输入值将与采样的纹理值进行比较，这个比较测试的结果将用于过滤操作（0.0f代表测试通过，1.0f代表测试失败）。

>> **ISSUE 10** 描述滤波与比较采样的交互方式。

---

```
enum GPUFilterMode {
    "nearest",
    "linear"
};

enum GPUMipmapFilterMode {
    "nearest",
    "linear"
};
```

**"nearest"**    
Return the value of the texel nearest to the texture coordinates.

**"linear"**    
Select two texels in each dimension and return a linear interpolation between their values.

GPUCompareFunction specifies the behavior of a comparison sampler. If a comparison sampler is used in a shader, an input value is compared to the sampled texture value, and the result of this comparison test (0.0f for pass, or 1.0f for fail) is used in the filtering operation.

>> ISSUE 10 describe how filtering interacts with comparison sampling.

---
---

```
enum GPUCompareFunction {
    "never",
    "less",
    "equal",
    "less-equal",
    "greater",
    "not-equal",
    "greater-equal",
    "always"
};
```

"never"     
比较试验从未通过。

"less"      
如果提供的值小于采样值，则通过比较测试。

"equal"     
如果提供的值与采样值相等，则通过比较测试。

"less-equal"        
如果提供的值小于或等于采样值，则通过比较测试。

"greater"       
如果提供的值大于采样值，则通过比较测试。

"not-equal"     
如果提供的值不等于采样值，则通过比较测试。

"greater-equal"     
如果一个提供的值大于或等于采样值，则通过比较测试。

"always"        
比较测试总是通过。

---

```
enum GPUCompareFunction {
    "never",
    "less",
    "equal",
    "less-equal",
    "greater",
    "not-equal",
    "greater-equal",
    "always"
};
```

**"never"**    
Comparison tests never pass.

**"less"**    
A provided value passes the comparison test if it is less than the sampled value.

**"equal"**    
A provided value passes the comparison test if it is equal to the sampled value.

**"less-equal"**    
A provided value passes the comparison test if it is less than or equal to the sampled value.

**"greater"**    
A provided value passes the comparison test if it is greater than the sampled value.

**"not-equal"**    
A provided value passes the comparison test if it is not equal to the sampled value.

**"greater-equal"**    
A provided value passes the comparison test if it is greater than or equal to the sampled value.

**"always"**    
Comparison tests always pass.

---
---
**createSampler(descriptor) 函数描述**      

用于创建一个 GPUSampler.    
调用于： GPUDevice 实例自身。   
参数：  

| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| `descriptor` | `GPUSamplerDescriptor` | ✘ | ✔ |  创建 `GPUSampler` 的描述符 |

返回值： GPUSampler

---

**createSampler(descriptor)**    
Creates a GPUSampler.

Called on: GPUDevice this.

**Arguments:**


| Parameter | Type | Nullable | Optional | Description |
| --- | --- | --- | --- | --- |
| `descriptor` | `GPUSamplerDescriptor` | ✘ | ✔ | Description of the `GPUSampler` to create. |

**Returns:** `GPUSampler`

---
---

上下文时间轴步骤：
1. 让 s 表示一个新的 GPUSampler 对象。
2. 在GPUDevice 这个实例的 设备时间线上发起初始化步骤。
3. 返回s。

---

**Content timeline steps:**
1. Let s be a new GPUSampler object.
2. Issue the initialization steps on the Device timeline of this.
3. Return s.

---
---

设备时间轴初始化步骤：
1. 如果以下任何条件不满足，则产生一个验证错误，使 s 无效，并停止。
    * GPUDevice 实例的this 为有效值。
    * descriptor.lodMinClamp ≥ 0。
    * descriptor.lodMaxClamp ≥ descriptor.lodMinClamp。
    * descriptor.maxAnisotropy ≥ 1。        
    **注意：** 大多数具体实现都支持最大各向异性值在1到16之间，包括其边界值。提供的 maxAnisotropy 值将被钳制在平台所支持的最大值。       
    * 如果 descriptor.maxAnisotropy > 1：
        * descriptor.magFilter、descriptor.minFilter 和 descriptor.mipmapFilter 必须是 "linear"。
2. 将 s.[[descriptor]] 设为 descriptor。
3. 如果 s.[[descriptor]] 的比较属性为空或未定义，则将 s.[isComparison] 设置为 false ，否则，将其设置为 true。
4. 如果 minFilter、magFilter或mipmapFilter的值都不是"liner"，将 s.[[isFiltering]] 设置为 false，否则，将其设置为 true。

---

**Device timeline initialization steps:**
1. If any of the following conditions are unsatisfied generate a validation error, make s invalid, and stop.
    * this is valid.
    * descriptor.lodMinClamp ≥ 0.
    * descriptor.lodMaxClamp ≥ descriptor.lodMinClamp.
    * descriptor.maxAnisotropy ≥ 1.    
    **NOTE:** Most implementations support maxAnisotropy values in range between 1 and 16, inclusive. The provided maxAnisotropy value will be clamped to the maximum value that the platform supports.
    * If descriptor.maxAnisotropy > 1:
        * descriptor.magFilter, descriptor.minFilter, and descriptor.mipmapFilter must be "linear".
2. Set s.[[descriptor]] to descriptor.
3. Set s.[[isComparison]] to false if the compare attribute of s.[[descriptor]] is null or undefined. Otherwise, set it to true.
4. Set s.[[isFiltering]] to false if none of minFilter, magFilter, or mipmapFilter has the value of "linear". Otherwise, set it to true.
   
---
---

```
// EXAMPLE 17
// 创建一个GPUSampler，其具备三重线性过滤 和 重复纹理坐标 等特征。
// Creating a GPUSampler that does trilinear filtering and repeats texture coordinates:

const sampler = gpuDevice.createSampler({
    addressModeU: 'repeat',
    addressModeV: 'repeat',
    magFilter: 'linear',
    minFilter: 'linear',
    mipmapFilter: 'linear',
});
```

---

```
// EXAMPLE 17
// Creating a GPUSampler that does trilinear filtering and repeats texture coordinates:

const sampler = gpuDevice.createSampler({
    addressModeU: 'repeat',
    addressModeV: 'repeat',
    magFilter: 'linear',
    minFilter: 'linear',
    mipmapFilter: 'linear',
});
```

