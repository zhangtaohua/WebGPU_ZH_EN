WebGPU

W3C Working Draft, 13 April 2023

More details about this document

This version:

https://www.w3.org/TR/2023/WD-webgpu-20230413/

---
---

# 24\. 类型定义

```
typedef [EnforceRange] unsigned long GPUBufferDynamicOffset;
typedef [EnforceRange] unsigned long GPUStencilValue;
typedef [EnforceRange] unsigned long GPUSampleMask;
typedef [EnforceRange] long GPUDepthBias;

typedef [EnforceRange] unsigned long long GPUSize64;
typedef [EnforceRange] unsigned long GPUIntegerCoordinate;
typedef [EnforceRange] unsigned long GPUIndex32;
typedef [EnforceRange] unsigned long GPUSize32;
typedef [EnforceRange] long GPUSignedOffset32;

typedef unsigned long GPUFlagsConstant;
```

---

24\. Type Definitions
```
typedef [EnforceRange] unsigned long GPUBufferDynamicOffset;
typedef [EnforceRange] unsigned long GPUStencilValue;
typedef [EnforceRange] unsigned long GPUSampleMask;
typedef [EnforceRange] long GPUDepthBias;

typedef [EnforceRange] unsigned long long GPUSize64;
typedef [EnforceRange] unsigned long GPUIntegerCoordinate;
typedef [EnforceRange] unsigned long GPUIndex32;
typedef [EnforceRange] unsigned long GPUSize32;
typedef [EnforceRange] long GPUSignedOffset32;

typedef unsigned long GPUFlagsConstant;
```

---
---

24.1. 颜色和向量

```
dictionary GPUColorDict {
    required double r;
    required double g;
    required double b;
    required double a;
};

typedef (sequence<double> or GPUColorDict) GPUColor;
```

**注意：** double足够大，可以精确容纳 32 位有符号/无符号整数 和单精度浮点数。

---

24.1. Colors & Vectors

```
dictionary GPUColorDict {
    required double r;
    required double g;
    required double b;
    required double a;
};

typedef (sequence<double> or GPUColorDict) GPUColor;
```

**NOTE:** double is large enough to precisely hold 32-bit signed/unsigned integers and single-precision floats.

---
---
对于一个给定的 GPUColor 类型的 color 值，根据其类型，其语法为：
* color.r 指的是 GPUColorDict.r 或 序列的第一项（断言有这样一个项目）。
* color.g 指的是 GPUColorDict.g 或 序列的第二项（断言有这样一个项目）。
* color.b 指的是 GPUColorDict.b 或 序列的第三项（断言有这样一个项目）。
* color.a 指的是 GPUColorDict.a 或 序列的第四项（断言有这样一个项目）。

---

For a given GPUColor value color, depending on its type, the syntax:
* color.r refers to either GPUColorDict.r or the first item of the sequence (asserting there is such an item).
* color.g refers to either GPUColorDict.g or the second item of the sequence (asserting there is such an item).
* color.b refers to either GPUColorDict.b or the third item of the sequence (asserting there is such an item).
* color.a refers to either GPUColorDict.a or the fourth item of the sequence (asserting there is such an item).

---
---

**验证 GPUColor 形状(color)**

**参数：**
* color：要验证的 GPUColor。

**返回值：** undefined

1. 如果 color 是一个序列 且 color.length ≠ 4，则抛出一个 TypeError 的类型错误。

```
dictionary GPUOrigin2DDict {
    GPUIntegerCoordinate x = 0;
    GPUIntegerCoordinate y = 0;
};
typedef (sequence<GPUIntegerCoordinate> or GPUOrigin2DDict) GPUOrigin2D;
```

对于一个给定的 GPUOrigin2D 类型的 origin 值，根据其类型，其语法为：
* origin.x 指的是 GPUOrigin2DDict.x 或 序列的第一项（如果不存在则为 0）。
* origin.y 指的是 GPUOrigin2DDict.y 或 序列的第二项（如果不存在则为 0）。

---

**validate GPUColor shape(color)**

**Arguments:**
* color: The GPUColor to validate.

**Returns:** undefined

1. Throw a TypeError if color is a sequence and color.length ≠ 4.

```
dictionary GPUOrigin2DDict {
    GPUIntegerCoordinate x = 0;
    GPUIntegerCoordinate y = 0;
};
typedef (sequence<GPUIntegerCoordinate> or GPUOrigin2DDict) GPUOrigin2D;
```

For a given GPUOrigin2D value origin, depending on its type, the syntax:
* origin.x refers to either GPUOrigin2DDict.x or the first item of the sequence (0 if not present).
* origin.y refers to either GPUOrigin2DDict.y or the second item of the sequence (0 if not present).


---
---

**验证GPUOrigin2D 形状(origin)**

**参数：**
* origin：要验证的 GPUOrigin2D。

**返回值：** undefined

1. 如果 origin 是一个序列且 origin.length > 2， 则抛出一个 TypeError 的类型错误。

```
dictionary GPUOrigin3DDict {
    GPUIntegerCoordinate x = 0;
    GPUIntegerCoordinate y = 0;
    GPUIntegerCoordinate z = 0;
};
typedef (sequence<GPUIntegerCoordinate> or GPUOrigin3DDict) GPUOrigin3D;
```

对于一个给定的 GPUOrigin3D 类型的 origin 值，根据其类型，其语法为：
* origin.x 指的是 GPUOrigin3DDict.x 或序列的第一项（如果不存在则为 0）。
* origin.y 指的是 GPUOrigin3DDict.y 或序列的第二项（如果不存在则为 0）。
* origin.z 指的是 GPUOrigin3DDict.z 或序列的第三项（如果不存在则为 0）。

---

**validate GPUOrigin2D shape(origin)**

**Arguments:**
* origin: The GPUOrigin2D to validate.

**Returns:** undefined

1. Throw a TypeError if origin is a sequence and origin.length > 2.

```
dictionary GPUOrigin3DDict {
    GPUIntegerCoordinate x = 0;
    GPUIntegerCoordinate y = 0;
    GPUIntegerCoordinate z = 0;
};
typedef (sequence<GPUIntegerCoordinate> or GPUOrigin3DDict) GPUOrigin3D;
```

For a given GPUOrigin3D value origin, depending on its type, the syntax:
* origin.x refers to either GPUOrigin3DDict.x or the first item of the sequence (0 if not present).
* origin.y refers to either GPUOrigin3DDict.y or the second item of the sequence (0 if not present).
* origin.z refers to either GPUOrigin3DDict.z or the third item of the sequence (0 if not present).

---
---

**验证GPUOrigin3D 形状(origin)**

**参数：**  
* origin：要验证的 GPUOrigin3D。

**返回值：** undefined

1 如果 origin 是一个序列且 origin.length > 3，则抛出一个 TypeError 的类型错误。

```
dictionary GPUExtent3DDict {
    required GPUIntegerCoordinate width;
    GPUIntegerCoordinate height = 1;
    GPUIntegerCoordinate depthOrArrayLayers = 1;
};
typedef (sequence<GPUIntegerCoordinate> or GPUExtent3DDict) GPUExtent3D;
```

对于一个给定的 GPUExtent3D 类型的 extent 值，根据其类型，其语法为：
* extent.width 指的是 GPUExtent3DDict.width 或序列的第一项（断言有这样一个项目）。
* extent.height 指的是 GPUExtent3DDict.height 或序列的第二项（如果不存在则为 1）。
* extent.depthOrArrayLayers 指的是 GPUExtent3DDict.depthOrArrayLayers 或序列的第三项（如果不存在则为 1）。

---

**validate GPUOrigin3D shape(origin)**

**Arguments:**
* origin: The GPUOrigin3D to validate.

**Returns:** undefined

1. Throw a TypeError if origin is a sequence and origin.length > 3.

```
dictionary GPUExtent3DDict {
    required GPUIntegerCoordinate width;
    GPUIntegerCoordinate height = 1;
    GPUIntegerCoordinate depthOrArrayLayers = 1;
};
typedef (sequence<GPUIntegerCoordinate> or GPUExtent3DDict) GPUExtent3D;
```

For a given GPUExtent3D value extent, depending on its type, the syntax:
* extent.width refers to either GPUExtent3DDict.width or the first item of the sequence (asserting there is such an item).
* extent.height refers to either GPUExtent3DDict.height or the second item of the sequence (1 if not present).
* extent.depthOrArrayLayers refers to either GPUExtent3DDict.depthOrArrayLayers or the third item of the sequence (1 if not present).

---
---

**验证 GPUExtent3D 形状(extent)**

**参数：**
* extent：要验证的 GPUExtent3D。

**返回值：** undefined

1. 如果以下条件满足，则抛出一个 TypeError 的类型错误：
*  extent 是一个序列，并且
* extent.length < 1 或者 extent.length > > 3。

---

**validate GPUExtent3D shape(extent)**

**Arguments:**
* extent: The GPUExtent3D to validate.

**Returns:** undefined

1. Throw a TypeError if:
* extent is a sequence, and
*extent.length < 1 or extent.length > > 3.

