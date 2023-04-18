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


# 摘要
> Abstract

WebGPU 提供一系列用于操作GPU的API接口，比如渲染和计算。

> WebGPU exposes an API for performing operations, such as rendering and computation, on a Graphics Processing Unit.

# 文档说明
> Status of this document

这一节说明此时此刻文档的发布状态，一系列W3C所发布的以及最新版本的技术报告可以在W3C技术报告目录中查看，网址如下：https://www.w3.org/TR/

> This section describes the status of this document at the time of its publication. A list of current W3C publications and the latest revision of this technical report can be found in the W3C technical reports index at https://www.w3.org/TR/.


欢迎针对此规范提供您保贵的反馈和建议。[GitHub讨论版](https://github.com/gpuweb/gpuweb/issues)块提供了非常好的讨论场所，另外，你也可以发送邮件到网页工作组织(The Web Working Group)的GPU小组邮件部门，邮件地址是：public-gpu@w3.org ([邮件文档库](https://lists.w3.org/Archives/Public/public-gpu/))。这个草稿版本高亮一些组织内部仍在讨论且悬而未决的问题。这些问题是否需要解决，以及这些问题的最终解决方案都还没有决定，以便引入正式版的规范文档中。

> Feedback and comments on this specification are welcome. GitHub Issues are preferred for discussion on this specification. Alternatively, you can send comments to the GPU for the Web Working Group’s mailing-list, public-gpu@w3.org (archives). This draft highlights some of the pending issues that are still to be discussed in the working group. No decision has been taken on the outcome of these issues including whether they are valid.

这份文档由[网页工作组织的GPU小组](https://www.w3.org/groups/wg/gpu)采用最先进[文档发布追踪技术](https://www.w3.org/2021/Process-20211102/#recs-and-notes)发布的，这份文档最终将成为W3C的标准规范。

> This document was published by the GPU for the Web Working Group as a Working Draft using the Recommendation track. This document is intended to become a W3C Recommendation.

需要提醒的是：作为草案发布的规范并不意味着此文档就得到了W3C及其成员的认可。

> Publication as a Working Draft does not imply endorsement by W3C and its Members.

这分草案随时可能更新，替代，甚至淘汰了。在正在进行中的工作中引用这份文档是不太合适的。

> This is a draft document and may be updated, replaced or obsoleted by other documents at any time. It is inappropriate to cite this document as other than work in progress.

这份文档由一个组织在[W3C专利政策](https://www.w3.org/Consortium/Patent-Policy/)下进行组织编辑的。W3C保留一份与此组织的提供可交付成果有关的[专利披露公开清单](https://www.w3.org/groups/wg/gpu/ipr); 这一页也包括披露专利的说明，任何个人如果真正了解相关专利知识，并认为包含基本权利要求的专利必须要披露相关信息时，必须按照[W3C专利政策的第6条](https://www.w3.org/Consortium/Patent-Policy/#sec-Disclosure)披露该信息。

> This document was produced by a group operating under the W3C Patent Policy. W3C maintains a public list of any patent disclosures made in connection with the deliverables of the group; that page also includes instructions for disclosing a patent. An individual who has actual knowledge of a patent which the individual believes contains Essential Claim(s) must disclose the information in accordance with section 6 of the W3C Patent Policy.

这份文档由[2021年11月2日W3C 开发协议](https://www.w3.org/2021/Process-20211102/)进行监督管理

> This document is governed by the 2 November 2021 W3C Process Document.