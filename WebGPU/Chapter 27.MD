WebGPU

W3C Working Draft, 21 April 2023

More details about this document

This version:

https://www.w3.org/TR/2023/WD-webgpu-20230421/

Latest published version:

https://www.w3.org/TR/webgpu/

Editor's Draft:

https://gpuweb.github.io/gpuweb/

Previous Versions:

https://www.w3.org/TR/2023/WD-webgpu-20230419/

---
---

### 一致性

## 文件约定

一致性要求是通过描述性断言和 RFC 2119术语的组合来表达的。本文档中规范性部分的内容中所使用的关键词如 "MUST"、"MUST NOT"、"REQUIRED"、"SHALL"、"SHALL NOT"、"SHOULD NOT"、"RECOMMENDED"、"MAY "和 "OPTIONAL" 应按照 RFC 2119 中的描述来解释。然而，为了便于阅读，这些词在本规范中不以全部大写字母形式出现。

本规范的所有文本都是规范性的，除了明确标记为非规范性的部分、示例和注释。[RFC2119]

---

Conformance

Document conventions

Conformance requirements are expressed with a combination of descriptive assertions and RFC 2119 terminology. The key words “MUST”, “MUST NOT”, “REQUIRED”, “SHALL”, “SHALL NOT”, “SHOULD”, “SHOULD NOT”, “RECOMMENDED”, “MAY”, and “OPTIONAL” in the normative parts of this document are to be interpreted as described in RFC 2119. However, for readability, these words do not appear in all uppercase letters in this specification.

All of the text of this specification is normative except sections explicitly marked as non-normative, examples, and notes. [RFC2119]

---
---

本规范中的例子用 "for example" 来介绍，或者在HTML文档中，使用 class="example" 来与规范性文本相区分开来，像这样：

```
// 例子 36

这是一个表示相关信息的例子。
```

信息性的说明以 "注意" 开始，并且在 HTML 文档中 加入 class="note" 的类，以便和规范性文本区分开来，像这样：

> **注意**：这是一个信息性说明。

---

Examples in this specification are introduced with the words “for example” or are set apart from the normative text with class="example", like this:

```
EXAMPLE 36

This is an example of an informative example.
```

Informative notes begin with the word “Note” and are set apart from the normative text with class="note", like this:

> Note, this is an informative note.

---
---

## 符合要求的算法

在命令中，作为一部分算法描述所提出的要求（如 "剔除任何前置空字符" 或 "返回错误并中止这些步骤"），应根据介绍算法时所使用的关键词（"必须"、"应该"、"可能"等）的含义来进行解释。

以算法或具体步骤所表述的一致性要求可以用任何方式实现，只要最终结果是等价的。特别是，本规范中定义的算法旨在使其易于理解，而不是为了达到更高的性能要求。我们鼓励具体的实现进行相应的优化。

---

Conformant Algorithms

Requirements phrased in the imperative as part of algorithms (such as "strip any leading space characters" or "return false and abort these steps") are to be interpreted with the meaning of the key word ("must", "should", "may", etc) used in introducing the algorithm.

Conformance requirements phrased as algorithms or specific steps can be implemented in any manner, so long as the end result is equivalent. In particular, the algorithms defined in this specification are intended to be easy to understand and are not intended to be performant. Implementers are encouraged to optimize.
