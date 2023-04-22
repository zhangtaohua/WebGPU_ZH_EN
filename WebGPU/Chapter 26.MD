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
# 26\. 附录

## 26.1. 纹理格式功能

### 26.1.1. 纯色格式

所有纯色格式都支持 COPY_SRC、COPY_DST 和 TEXTURE_BINDING 用法。

RENDER_ATTACHMENT 和 STORAGE_BINDING 列分别指定对 GPUTextureUsage.RENDER_ATTACHMENT 和 GPUTextureUsage.STORAGE_BINDING 用法的支持。

渲染目标像素字节成本和渲染目标分量对齐方式用于验证maxColorAttachmentBytesPerSample 限制。

注意：这些格式中的每一种的 texel 块内存成本与它的 texel块 复制足迹相同。

---

26\. Appendices

26.1. Texture Format Capabilities

26.1.1. Plain color formats

All plain color formats support COPY_SRC, COPY_DST, and TEXTURE_BINDING usage.

The RENDER_ATTACHMENT and STORAGE_BINDING columns specify support for GPUTextureUsage.RENDER_ATTACHMENT and GPUTextureUsage.STORAGE_BINDING usage respectively.

The render target pixel byte cost and render target component alignment are used to validate the maxColorAttachmentBytesPerSample limit.

**NOTE:** The texel block memory cost of each of these formats is the same as its texel block copy footprint.

---
---

***表格暂时不做翻译***

---

<div class="overlarge"><table class="data">
    <thead class="stickyheader">
     <tr>
      <th>Format 
      </th><th><code class="idl"><a data-link-type="idl" href="#enumdef-gputexturesampletype" id="ref-for-enumdef-gputexturesampletype③">GPUTextureSampleType</a></code> 
      </th><th><span class="vertical"><code class="idl"><a data-link-type="idl" href="#dom-gputextureusage-render_attachment" id="ref-for-dom-gputextureusage-render_attachment①⑦">RENDER_ATTACHMENT</a></code></span> 
      </th><th><span class="vertical"><a data-link-type="dfn" href="#blendable" id="ref-for-blendable①">blendable</a></span> 
      </th><th><span class="vertical">multisampling</span> 
      </th><th><span class="vertical">resolve</span> 
      </th><th><span class="vertical"><code class="idl"><a data-link-type="idl" href="#dom-gputextureusage-storage_binding" id="ref-for-dom-gputextureusage-storage_binding①⓪">STORAGE_BINDING</a></code></span> 
      </th><th><a data-link-type="dfn" href="#texel-block-copy-footprint" id="ref-for-texel-block-copy-footprint④">Texel block copy footprint</a> (Bytes) 
      </th><th><a data-link-type="dfn" href="#render-target-pixel-byte-cost" id="ref-for-render-target-pixel-byte-cost①">Render target pixel byte cost</a> (Bytes) 
    </th></tr></thead><tbody>
     <tr>
      <th colspan="9">8 bits per component (1-byte <a data-link-type="dfn" href="#render-target-component-alignment" id="ref-for-render-target-component-alignment①">render target component alignment</a>) 
     </th></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-r8unorm" id="ref-for-dom-gputextureformat-r8unorm①">r8unorm</a></code> 
      </td><td><code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-float" id="ref-for-dom-gputexturesampletype-float⑨">"float"</a></code>,<br><code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-unfilterable-float" id="ref-for-dom-gputexturesampletype-unfilterable-float⑥">"unfilterable-float"</a></code> 
      </td><td>✓ 
      </td><td>✓ 
      </td><td>✓ 
      </td><td>✓ 
      </td><td>
      </td><td colspan="2">1 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-r8snorm" id="ref-for-dom-gputextureformat-r8snorm">r8snorm</a></code> 
      </td><td><code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-float" id="ref-for-dom-gputexturesampletype-float①⓪">"float"</a></code>,<br><code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-unfilterable-float" id="ref-for-dom-gputexturesampletype-unfilterable-float⑦">"unfilterable-float"</a></code> 
      </td><td>
      </td><td>
      </td><td>
      </td><td>
      </td><td>
      </td><td>1 
      </td><td>– 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-r8uint" id="ref-for-dom-gputextureformat-r8uint">r8uint</a></code> 
      </td><td><code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-uint" id="ref-for-dom-gputexturesampletype-uint④">"uint"</a></code> 
      </td><td>✓ 
      </td><td>
      </td><td>✓ 
      </td><td>
      </td><td>
      </td><td colspan="2">1 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-r8sint" id="ref-for-dom-gputextureformat-r8sint">r8sint</a></code> 
      </td><td><code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-sint" id="ref-for-dom-gputexturesampletype-sint④">"sint"</a></code> 
      </td><td>✓ 
      </td><td>
      </td><td>✓ 
      </td><td>
      </td><td>
      </td><td colspan="2">1 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-rg8unorm" id="ref-for-dom-gputextureformat-rg8unorm①">rg8unorm</a></code> 
      </td><td><code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-float" id="ref-for-dom-gputexturesampletype-float①①">"float"</a></code>,<br><code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-unfilterable-float" id="ref-for-dom-gputexturesampletype-unfilterable-float⑧">"unfilterable-float"</a></code> 
      </td><td>✓ 
      </td><td>✓ 
      </td><td>✓ 
      </td><td>✓ 
      </td><td>
      </td><td colspan="2">2 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-rg8snorm" id="ref-for-dom-gputextureformat-rg8snorm">rg8snorm</a></code> 
      </td><td><code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-float" id="ref-for-dom-gputexturesampletype-float①②">"float"</a></code>,<br><code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-unfilterable-float" id="ref-for-dom-gputexturesampletype-unfilterable-float⑨">"unfilterable-float"</a></code> 
      </td><td>
      </td><td>
      </td><td>
      </td><td>
      </td><td>
      </td><td>2 
      </td><td>– 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-rg8uint" id="ref-for-dom-gputextureformat-rg8uint">rg8uint</a></code> 
      </td><td><code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-uint" id="ref-for-dom-gputexturesampletype-uint⑤">"uint"</a></code> 
      </td><td>✓ 
      </td><td>
      </td><td>✓ 
      </td><td>
      </td><td>
      </td><td colspan="2">2 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-rg8sint" id="ref-for-dom-gputextureformat-rg8sint">rg8sint</a></code> 
      </td><td><code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-sint" id="ref-for-dom-gputexturesampletype-sint⑤">"sint"</a></code> 
      </td><td>✓ 
      </td><td>
      </td><td>✓ 
      </td><td>
      </td><td>
      </td><td colspan="2">2 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-rgba8unorm" id="ref-for-dom-gputextureformat-rgba8unorm⑤">rgba8unorm</a></code> 
      </td><td><code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-float" id="ref-for-dom-gputexturesampletype-float①③">"float"</a></code>,<br><code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-unfilterable-float" id="ref-for-dom-gputexturesampletype-unfilterable-float①⓪">"unfilterable-float"</a></code> 
      </td><td>✓ 
      </td><td>✓ 
      </td><td>✓ 
      </td><td>✓ 
      </td><td>✓ 
      </td><td>4 
      </td><td>8 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-rgba8unorm-srgb" id="ref-for-dom-gputextureformat-rgba8unorm-srgb②">rgba8unorm-srgb</a></code> 
      </td><td><code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-float" id="ref-for-dom-gputexturesampletype-float①④">"float"</a></code>,<br><code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-unfilterable-float" id="ref-for-dom-gputexturesampletype-unfilterable-float①①">"unfilterable-float"</a></code> 
      </td><td>✓ 
      </td><td>✓ 
      </td><td>✓ 
      </td><td>✓ 
      </td><td>
      </td><td>4 
      </td><td>8 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-rgba8snorm" id="ref-for-dom-gputextureformat-rgba8snorm">rgba8snorm</a></code> 
      </td><td><code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-float" id="ref-for-dom-gputexturesampletype-float①⑤">"float"</a></code>,<br><code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-unfilterable-float" id="ref-for-dom-gputexturesampletype-unfilterable-float①②">"unfilterable-float"</a></code> 
      </td><td>
      </td><td>
      </td><td>
      </td><td>
      </td><td>✓ 
      </td><td>4 
      </td><td>– 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-rgba8uint" id="ref-for-dom-gputextureformat-rgba8uint">rgba8uint</a></code> 
      </td><td><code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-uint" id="ref-for-dom-gputexturesampletype-uint⑥">"uint"</a></code> 
      </td><td>✓ 
      </td><td>
      </td><td>✓ 
      </td><td>
      </td><td>✓ 
      </td><td colspan="2">4 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-rgba8sint" id="ref-for-dom-gputextureformat-rgba8sint">rgba8sint</a></code> 
      </td><td><code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-sint" id="ref-for-dom-gputexturesampletype-sint⑥">"sint"</a></code> 
      </td><td>✓ 
      </td><td>
      </td><td>✓ 
      </td><td>
      </td><td>✓ 
      </td><td colspan="2">4 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-bgra8unorm" id="ref-for-dom-gputextureformat-bgra8unorm⑥">bgra8unorm</a></code> 
      </td><td><code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-float" id="ref-for-dom-gputexturesampletype-float①⑥">"float"</a></code>,<br><code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-unfilterable-float" id="ref-for-dom-gputexturesampletype-unfilterable-float①③">"unfilterable-float"</a></code> 
      </td><td>✓ 
      </td><td>✓ 
      </td><td>✓ 
      </td><td>✓ 
      </td><td>If <code class="idl"><a data-link-type="idl" href="#bgra8unorm-storage" id="ref-for-bgra8unorm-storage①">"bgra8unorm-storage"</a></code> is enabled 
      </td><td>4 
      </td><td>8 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-bgra8unorm-srgb" id="ref-for-dom-gputextureformat-bgra8unorm-srgb②">bgra8unorm-srgb</a></code> 
      </td><td><code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-float" id="ref-for-dom-gputexturesampletype-float①⑦">"float"</a></code>,<br><code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-unfilterable-float" id="ref-for-dom-gputexturesampletype-unfilterable-float①④">"unfilterable-float"</a></code> 
      </td><td>✓ 
      </td><td>✓ 
      </td><td>✓ 
      </td><td>✓ 
      </td><td>
      </td><td>4 
      </td><td>8 
     </td></tr><tr>
      <th colspan="9">16 bits per component (2-byte <a data-link-type="dfn" href="#render-target-component-alignment" id="ref-for-render-target-component-alignment②">render target component alignment</a>) 
     </th></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-r16uint" id="ref-for-dom-gputextureformat-r16uint">r16uint</a></code> 
      </td><td><code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-uint" id="ref-for-dom-gputexturesampletype-uint⑦">"uint"</a></code> 
      </td><td>✓ 
      </td><td>
      </td><td>✓ 
      </td><td>
      </td><td>
      </td><td colspan="2">2 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-r16sint" id="ref-for-dom-gputextureformat-r16sint">r16sint</a></code> 
      </td><td><code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-sint" id="ref-for-dom-gputexturesampletype-sint⑦">"sint"</a></code> 
      </td><td>✓ 
      </td><td>
      </td><td>✓ 
      </td><td>
      </td><td>
      </td><td colspan="2">2 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-r16float" id="ref-for-dom-gputextureformat-r16float①">r16float</a></code> 
      </td><td><code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-float" id="ref-for-dom-gputexturesampletype-float①⑧">"float"</a></code>,<br><code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-unfilterable-float" id="ref-for-dom-gputexturesampletype-unfilterable-float①⑤">"unfilterable-float"</a></code> 
      </td><td>✓ 
      </td><td>✓ 
      </td><td>✓ 
      </td><td>✓ 
      </td><td>
      </td><td colspan="2">2 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-rg16uint" id="ref-for-dom-gputextureformat-rg16uint">rg16uint</a></code> 
      </td><td><code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-uint" id="ref-for-dom-gputexturesampletype-uint⑧">"uint"</a></code> 
      </td><td>✓ 
      </td><td>
      </td><td>✓ 
      </td><td>
      </td><td>
      </td><td colspan="2">4 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-rg16sint" id="ref-for-dom-gputextureformat-rg16sint">rg16sint</a></code> 
      </td><td><code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-sint" id="ref-for-dom-gputexturesampletype-sint⑧">"sint"</a></code> 
      </td><td>✓ 
      </td><td>
      </td><td>✓ 
      </td><td>
      </td><td>
      </td><td colspan="2">4 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-rg16float" id="ref-for-dom-gputextureformat-rg16float①">rg16float</a></code> 
      </td><td><code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-float" id="ref-for-dom-gputexturesampletype-float①⑨">"float"</a></code>,<br><code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-unfilterable-float" id="ref-for-dom-gputexturesampletype-unfilterable-float①⑥">"unfilterable-float"</a></code> 
      </td><td>✓ 
      </td><td>✓ 
      </td><td>✓ 
      </td><td>✓ 
      </td><td>
      </td><td colspan="2">4 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-rgba16uint" id="ref-for-dom-gputextureformat-rgba16uint">rgba16uint</a></code> 
      </td><td><code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-uint" id="ref-for-dom-gputexturesampletype-uint⑨">"uint"</a></code> 
      </td><td>✓ 
      </td><td>
      </td><td>✓ 
      </td><td>
      </td><td>✓ 
      </td><td colspan="2">8 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-rgba16sint" id="ref-for-dom-gputextureformat-rgba16sint">rgba16sint</a></code> 
      </td><td><code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-sint" id="ref-for-dom-gputexturesampletype-sint⑨">"sint"</a></code> 
      </td><td>✓ 
      </td><td>
      </td><td>✓ 
      </td><td>
      </td><td>✓ 
      </td><td colspan="2">8 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-rgba16float" id="ref-for-dom-gputextureformat-rgba16float②">rgba16float</a></code> 
      </td><td><code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-float" id="ref-for-dom-gputexturesampletype-float②⓪">"float"</a></code>,<br><code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-unfilterable-float" id="ref-for-dom-gputexturesampletype-unfilterable-float①⑦">"unfilterable-float"</a></code> 
      </td><td>✓ 
      </td><td>✓ 
      </td><td>✓ 
      </td><td>✓ 
      </td><td>✓ 
      </td><td colspan="2">8 
     </td></tr><tr>
      <th colspan="9">32 bits per component (4-byte <a data-link-type="dfn" href="#render-target-component-alignment" id="ref-for-render-target-component-alignment③">render target component alignment</a>) 
     </th></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-r32uint" id="ref-for-dom-gputextureformat-r32uint">r32uint</a></code> 
      </td><td><code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-uint" id="ref-for-dom-gputexturesampletype-uint①⓪">"uint"</a></code> 
      </td><td>✓ 
      </td><td>
      </td><td>
      </td><td>
      </td><td>✓ 
      </td><td colspan="2">4 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-r32sint" id="ref-for-dom-gputextureformat-r32sint">r32sint</a></code> 
      </td><td><code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-sint" id="ref-for-dom-gputexturesampletype-sint①⓪">"sint"</a></code> 
      </td><td>✓ 
      </td><td>
      </td><td>
      </td><td>
      </td><td>✓ 
      </td><td colspan="2">4 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-r32float" id="ref-for-dom-gputextureformat-r32float②">r32float</a></code> 
      </td><td>
       <code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-unfilterable-float" id="ref-for-dom-gputexturesampletype-unfilterable-float①⑧">"unfilterable-float"</a></code> 
       <ul>
        <li data-md="">
         <p><code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-float" id="ref-for-dom-gputexturesampletype-float②①">"float"</a></code> if <code class="idl"><a data-link-type="idl" href="#float32-filterable" id="ref-for-float32-filterable①">"float32-filterable"</a></code> is enabled</p>
       </li></ul>
      </td><td>✓ 
      </td><td>
      </td><td>✓ 
      </td><td>
      </td><td>✓ 
      </td><td colspan="2">4 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-rg32uint" id="ref-for-dom-gputextureformat-rg32uint">rg32uint</a></code> 
      </td><td><code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-uint" id="ref-for-dom-gputexturesampletype-uint①①">"uint"</a></code> 
      </td><td>✓ 
      </td><td>
      </td><td>
      </td><td>
      </td><td>✓ 
      </td><td colspan="2">8 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-rg32sint" id="ref-for-dom-gputextureformat-rg32sint">rg32sint</a></code> 
      </td><td><code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-sint" id="ref-for-dom-gputexturesampletype-sint①①">"sint"</a></code> 
      </td><td>✓ 
      </td><td>
      </td><td>
      </td><td>
      </td><td>✓ 
      </td><td colspan="2">8 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-rg32float" id="ref-for-dom-gputextureformat-rg32float②">rg32float</a></code> 
      </td><td>
       <code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-unfilterable-float" id="ref-for-dom-gputexturesampletype-unfilterable-float①⑨">"unfilterable-float"</a></code> 
       <ul>
        <li data-md="">
         <p><code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-float" id="ref-for-dom-gputexturesampletype-float②②">"float"</a></code> if <code class="idl"><a data-link-type="idl" href="#float32-filterable" id="ref-for-float32-filterable②">"float32-filterable"</a></code> is enabled</p>
       </li></ul>
      </td><td>✓ 
      </td><td>
      </td><td>
      </td><td>
      </td><td>✓ 
      </td><td colspan="2">8 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-rgba32uint" id="ref-for-dom-gputextureformat-rgba32uint">rgba32uint</a></code> 
      </td><td><code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-uint" id="ref-for-dom-gputexturesampletype-uint①②">"uint"</a></code> 
      </td><td>✓ 
      </td><td>
      </td><td>
      </td><td>
      </td><td>✓ 
      </td><td colspan="2">16 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-rgba32sint" id="ref-for-dom-gputextureformat-rgba32sint">rgba32sint</a></code> 
      </td><td><code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-sint" id="ref-for-dom-gputexturesampletype-sint①②">"sint"</a></code> 
      </td><td>✓ 
      </td><td>
      </td><td>
      </td><td>
      </td><td>✓ 
      </td><td colspan="2">16 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-rgba32float" id="ref-for-dom-gputextureformat-rgba32float②">rgba32float</a></code> 
      </td><td>
       <code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-unfilterable-float" id="ref-for-dom-gputexturesampletype-unfilterable-float②⓪">"unfilterable-float"</a></code> 
       <ul>
        <li data-md="">
         <p><code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-float" id="ref-for-dom-gputexturesampletype-float②③">"float"</a></code> if <code class="idl"><a data-link-type="idl" href="#float32-filterable" id="ref-for-float32-filterable③">"float32-filterable"</a></code> is enabled</p>
       </li></ul>
      </td><td>✓ 
      </td><td>
      </td><td>
      </td><td>
      </td><td>✓ 
      </td><td colspan="2">16 
     </td></tr><tr>
      <th colspan="9">mixed component width, 32 bits per texel (4-byte <a data-link-type="dfn" href="#render-target-component-alignment" id="ref-for-render-target-component-alignment④">render target component alignment</a>) 
     </th></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-rgb10a2unorm" id="ref-for-dom-gputextureformat-rgb10a2unorm①">rgb10a2unorm</a></code> 
      </td><td><code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-float" id="ref-for-dom-gputexturesampletype-float②④">"float"</a></code>,<br><code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-unfilterable-float" id="ref-for-dom-gputexturesampletype-unfilterable-float②①">"unfilterable-float"</a></code> 
      </td><td>✓ 
      </td><td>✓ 
      </td><td>✓ 
      </td><td>✓ 
      </td><td>
      </td><td>4 
      </td><td>8 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-rg11b10ufloat" id="ref-for-dom-gputextureformat-rg11b10ufloat①">rg11b10ufloat</a></code> 
      </td><td><code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-float" id="ref-for-dom-gputexturesampletype-float②⑤">"float"</a></code>,<br><code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-unfilterable-float" id="ref-for-dom-gputexturesampletype-unfilterable-float②②">"unfilterable-float"</a></code> 
      </td><td colspan="4">If <code class="idl"><a data-link-type="idl" href="#rg11b10ufloat-renderable" id="ref-for-rg11b10ufloat-renderable①">"rg11b10ufloat-renderable"</a></code> is enabled 
      </td><td>
      </td><td>4 
      </td><td>8 
   </td></tr></tbody></table></div>

---
---

# 26.1.2.深度/模板格式

深度或模板格式是具有深度 和/或 模板方面的任何格式。一个深度/模板组合格式是一种既有深度又有模板的深度/模板格式。

所有的深度/模板格式都支持 COPY_SRC、COPY_DST、TEXTURE_BINDING 和 RENDER_ATTACHMENT 的用法。所有这些格式都支持多采样。然而，某些复制操作也限制了源和目标格式。

深度纹理不能与 "过滤" 采样器一起使用，但总是可以与 "比较"采样器一起使用（它可能使用过滤）。

---

26.1.2. Depth-stencil formats

A depth-or-stencil format is any format with depth and/or stencil aspects. A combined depth-stencil format is a depth-or-stencil format that has both depth and stencil aspects.

All depth-or-stencil formats support the COPY_SRC, COPY_DST, TEXTURE_BINDING, and RENDER_ATTACHMENT usages. All of these formats support multisampling. However, certain copy operations also restrict the source and destination formats.

Depth textures cannot be used with "filtering" samplers, but can always be used with "comparison" samplers (which may use filtering).

---
---

***表格暂时不做翻译***

---

<div class="overlarge"><table class="data">
    <thead>
     <tr>
      <th>Format 
      </th><th class="note"><a data-link-type="dfn" href="#texel-block-memory-cost" id="ref-for-texel-block-memory-cost①">Texel block memory cost</a> (Bytes) 
      </th><th>Aspect 
      </th><th><code class="idl"><a data-link-type="idl" href="#enumdef-gputexturesampletype" id="ref-for-enumdef-gputexturesampletype④">GPUTextureSampleType</a></code> 
      </th><th>Valid <a data-link-type="dfn" href="#image-copy" id="ref-for-image-copy④">image copy</a> source 
      </th><th>Valid <a data-link-type="dfn" href="#image-copy" id="ref-for-image-copy⑤">image copy</a> destination 
      </th><th><a data-link-type="dfn" href="#texel-block-copy-footprint" id="ref-for-texel-block-copy-footprint⑤">Texel block copy footprint</a> (Bytes) 
      </th><th><dfn class="dfn-paneled has-dfn-panel" data-dfn-type="dfn" data-noexport="" id="aspect-specific-format" role="button" aria-expanded="false" tabindex="0">Aspect-specific format</dfn><span style="position: relative; height: 0px;"><aside class="dfn-panel" id="infopanel-for-aspect-specific-format" data-for="aspect-specific-format" aria-labelled-by="infopaneltitle-for-aspect-specific-format"><span id="infopaneltitle-for-aspect-specific-format" style="display:none">Info about the 'Aspect-specific format'  reference.</span><a href="#aspect-specific-format">#aspect-specific-format</a><b>Referenced in:</b><ul><li><a href="#ref-for-aspect-specific-format">6.3. Texture Formats</a> </li><li><a href="#ref-for-aspect-specific-format①">13.5. Image Copy Commands</a> <a href="#ref-for-aspect-specific-format②">(2)</a> </li><li><a href="#ref-for-aspect-specific-format③">19.2. GPUQueue

</a> </li></ul></aside></span> 
    </th></tr></thead><tbody>
     <tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-stencil8" id="ref-for-dom-gputextureformat-stencil8②">stencil8</a></code> 
      </td><td>1 − 4 
      </td><td>stencil 
      </td><td><code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-uint" id="ref-for-dom-gputexturesampletype-uint①③">"uint"</a></code> 
      </td><td colspan="2">✓ 
      </td><td>1 
      </td><td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-stencil8" id="ref-for-dom-gputextureformat-stencil8③">stencil8</a></code> 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-depth16unorm" id="ref-for-dom-gputextureformat-depth16unorm">depth16unorm</a></code> 
      </td><td>2 
      </td><td>depth 
      </td><td><code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-depth" id="ref-for-dom-gputexturesampletype-depth④">"depth"</a></code>, <code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-unfilterable-float" id="ref-for-dom-gputexturesampletype-unfilterable-float②③">"unfilterable-float"</a></code> 
      </td><td colspan="2">✓ 
      </td><td>2 
      </td><td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-depth16unorm" id="ref-for-dom-gputextureformat-depth16unorm①">depth16unorm</a></code> 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-depth24plus" id="ref-for-dom-gputextureformat-depth24plus①">depth24plus</a></code> 
      </td><td>4 
      </td><td>depth 
      </td><td><code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-depth" id="ref-for-dom-gputexturesampletype-depth⑤">"depth"</a></code>, <code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-unfilterable-float" id="ref-for-dom-gputexturesampletype-unfilterable-float②④">"unfilterable-float"</a></code> 
      </td><td colspan="2">✗ 
      </td><td>– 
      </td><td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-depth24plus" id="ref-for-dom-gputextureformat-depth24plus②">depth24plus</a></code> 
     </td></tr><tr>
      <td rowspan="2" style="white-space: nowrap"><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-depth24plus-stencil8" id="ref-for-dom-gputextureformat-depth24plus-stencil8②">depth24plus-stencil8</a></code> 
      </td><td rowspan="2">4 − 8 
      </td><td>depth 
      </td><td><code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-depth" id="ref-for-dom-gputexturesampletype-depth⑥">"depth"</a></code>, <code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-unfilterable-float" id="ref-for-dom-gputexturesampletype-unfilterable-float②⑤">"unfilterable-float"</a></code> 
      </td><td colspan="2">✗ 
      </td><td>– 
      </td><td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-depth24plus" id="ref-for-dom-gputextureformat-depth24plus③">depth24plus</a></code> 
     </td></tr><tr>
      <td>stencil 
      </td><td><code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-uint" id="ref-for-dom-gputexturesampletype-uint①④">"uint"</a></code> 
      </td><td colspan="2">✓ 
      </td><td>1 
      </td><td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-stencil8" id="ref-for-dom-gputextureformat-stencil8④">stencil8</a></code> 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-depth32float" id="ref-for-dom-gputextureformat-depth32float①">depth32float</a></code> 
      </td><td>4 
      </td><td>depth 
      </td><td><code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-depth" id="ref-for-dom-gputexturesampletype-depth⑦">"depth"</a></code>, <code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-unfilterable-float" id="ref-for-dom-gputexturesampletype-unfilterable-float②⑥">"unfilterable-float"</a></code> 
      </td><td>✓ 
      </td><td>✗ 
      </td><td>4 
      </td><td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-depth32float" id="ref-for-dom-gputextureformat-depth32float②">depth32float</a></code> 
     </td></tr><tr>
      <td rowspan="2" style="white-space: nowrap"><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-depth32float-stencil8" id="ref-for-dom-gputextureformat-depth32float-stencil8③">depth32float-stencil8</a></code> 
      </td><td rowspan="2">5 − 8 
      </td><td>depth 
      </td><td><code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-depth" id="ref-for-dom-gputexturesampletype-depth⑧">"depth"</a></code>, <code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-unfilterable-float" id="ref-for-dom-gputexturesampletype-unfilterable-float②⑦">"unfilterable-float"</a></code> 
      </td><td>✓ 
      </td><td>✗ 
      </td><td>4 
      </td><td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-depth32float" id="ref-for-dom-gputextureformat-depth32float③">depth32float</a></code> 
     </td></tr><tr>
      <td>stencil 
      </td><td><code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-uint" id="ref-for-dom-gputexturesampletype-uint①⑤">"uint"</a></code> 
      </td><td colspan="2">✓ 
      </td><td>1 
      </td><td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-stencil8" id="ref-for-dom-gputextureformat-stencil8⑤">stencil8</a></code> 
   </td></tr></tbody></table></div>

---
---
24位深度指的是 24位无符号归一化深度格式，范围从 0.0 到 1.0，如果暴露出来，会被拼写成 "depth24unorm"。

---

24-bit depth refers to a 24-bit unsigned normalized depth format with a range from 0.0 to 1.0, which would be spelled "depth24unorm" if exposed.

---
---

#### 26.1.2.1. 读取和采样 深度/模板纹理

可以将具有深度视角的 GPUTextureView 与 texture_depth_* 绑定 或与 其他非深度的2D/立方体纹理类型的绑定 进行必要的绑定。

具有模板方面的 GPUTextureView 必须被绑定到一个正常的纹理绑定类型。 GPUBindGroupLayout 中的 sampleType 必须是 "uint"。

读取或采样纹理的深度或模板方面的行为就像纹理包含的值（V, X, X, X）一样，其中 V 是实际的深度或模板值，每个 X 是一个与具体实现所定义的未指定的值。

对于深度方面的绑定，未指定的值通过 texture_depth_*类型的绑定 是不可见的。

---
26.1.2.1. Reading and Sampling Depth/Stencil Textures

It is possible to bind a depth-aspect GPUTextureView to either a texture_depth_* binding or a binding with other non-depth 2d/cube texture types.

A stencil-aspect GPUTextureView must be bound to a normal texture binding type. The sampleType in the GPUBindGroupLayout must be "uint".

Reading or sampling the depth or stencil aspect of a texture behaves as if the texture contains the values (V, X, X, X), where V is the actual depth or stencil value, and each X is an implementation-defined unspecified value.

For depth-aspect bindings, the unspecified values are not visible through bindings with texture_depth_* types.

---
---

```
例子 35
如果一个深度纹理被绑定到 tex 上，且类型为 texture_2d<f32>， 那么

textureSample(tex, ...) 将返回 vec4<f32>(D, X, X, X)。

textureGather(0, tex, ...) 将返回 vec4<f32>(D1, D2, D3, D4)。

textureGather(2, tex, ...) 将返回 vec4<f32>(X1, X2, X3, X4) (一个完全没有指定的值)。

```

如果不增加一个新的更有约束性的模板采样器类型（比如深度），那么对于具体的实现来说，在驱动差异性存在时，要有效地解决深度/模板的读取是不可行的。由于这不是 WebGL 的可移植性痛点，预计在 WebGPU 中也不会有问题。在实践中，根据硬件的不同，可以期待（V，V，V，V）或者（V，0，0，1）（其中V是深度或模板值）。

---

```
EXAMPLE 35
If a depth texture is bound to tex with type texture_2d<f32>:
textureSample(tex, ...) will return vec4<f32>(D, X, X, X).

textureGather(0, tex, ...) will return vec4<f32>(D1, D2, D3, D4).

textureGather(2, tex, ...) will return vec4<f32>(X1, X2, X3, X4) (a completely unspecified value).
```

**NOTE:**  Short of adding a new more constrained stencil sampler type (like depth), it’s infeasible for implementations to efficiently paper over the driver differences for depth/stencil reads. As this was not a portability pain point for WebGL, it’s not expected to be problematic in WebGPU. In practice, expect either (V, V, V, V) or (V, 0, 0, 1) (where V is the depth or stencil value), depending on hardware.

---
---

#### 6.1.2.2. 复制深度/模板纹理

深度方面 具有 depth32float格式（"depth32float" 和  "depth32float-stencil8" ）的纹理有一个有限的范围。因此，复制到这种纹理时，只能从相同格式的其他纹理进行才有效。

深度方法 具有 depth24plus格式（"depth24plus" 和 "depth24plus-stencil8"）的纹理有不透明的表示（具体的实现要么是  24位深度，要么是 "depth32float"）。因此，这些格式不允许应用于深度方面的图像复制。

---

6.1.2.2. Copying Depth/Stencil Textures

The depth aspects of depth32float formats ("depth32float" and "depth32float-stencil8" have a limited range. As a result, copies into such textures are only valid from other textures of the same format.

The depth aspects of depth24plus formats ("depth24plus" and "depth24plus-stencil8") have opaque representations (implemented as either 24-bit depth or "depth32float"). As a result, depth-aspect image copies are not allowed with these formats.


---
---

**注意：** 模仿这些不允许的拷贝操作是可能的：
* 所有这些格式都可以写在渲染通道中，使用一个片段着色器 通过 frag_depth 输出来输出相应的深度值。

具有 "depth24plus"格式的纹理可以作为着色器纹理被读取，并写入纹理（作为渲染通道附件）或缓冲区（通过计算着色器中的存储缓冲区绑定）。

---

**Note:** It is possible to imitate these disallowed copies:
* All of these formats can be written in a render pass using a fragment shader that outputs depth values via the frag_depth output.

* Textures with "depth24plus" formats can be read as shader textures, and written to a texture (as a render pass attachment) or buffer (via a storage buffer binding in a compute shader).

---
---

### 26.1.3. 打包的格式

所有打包的纹理格式都支持 COPY_SRC、COPY_DST 和 TEXTURE_BINDING 的用法。所有这些格式都是可过滤的。这些格式都不是可渲染的，也不支持多采样。

压缩格式是指任何块大小大于 1×1 的格式。

**注意：** 这些格式中的每一种的 texel块 内存成本都与它的 texel块 复制足迹相同。

---

26.1.3. Packed formats

All packed texture formats support COPY_SRC, COPY_DST, and TEXTURE_BINDING usages. All of these formats are filterable. None of these formats are renderable or support multisampling.

A compressed format is any format with a block size greater than 1×1.

NOTE: The texel block memory cost of each of these formats is the same as its texel block copy footprint.

---
---

***表格暂时不做翻译***

---

<div class="overlarge"><table class="data">
    <thead class="stickyheader">
     <tr>
      <th>Format 
      </th><th><a data-link-type="dfn" href="#texel-block-copy-footprint" id="ref-for-texel-block-copy-footprint⑦">Texel block copy footprint</a> (Bytes) 
      </th><th><code class="idl"><a data-link-type="idl" href="#enumdef-gputexturesampletype" id="ref-for-enumdef-gputexturesampletype⑤">GPUTextureSampleType</a></code> 
      </th><th>Texel block <a data-link-type="dfn" href="#texel-block-width" id="ref-for-texel-block-width⑨">width</a>/<a data-link-type="dfn" href="#texel-block-height" id="ref-for-texel-block-height⑧">height</a> 
      </th><th><a data-link-type="dfn" href="#feature" id="ref-for-feature①②">Feature</a> 
    </th></tr></thead><tbody>
     <tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-rgb9e5ufloat" id="ref-for-dom-gputextureformat-rgb9e5ufloat">rgb9e5ufloat</a></code> 
      </td><td>4 
      </td><td><code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-float" id="ref-for-dom-gputexturesampletype-float②⑥">"float"</a></code>,<br><code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-unfilterable-float" id="ref-for-dom-gputexturesampletype-unfilterable-float②⑧">"unfilterable-float"</a></code> 
      </td><td>1 × 1 
      </td><td>
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-bc1-rgba-unorm" id="ref-for-dom-gputextureformat-bc1-rgba-unorm①">bc1-rgba-unorm</a></code> 
      </td><td rowspan="2">8 
      </td><td rowspan="14"><code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-float" id="ref-for-dom-gputexturesampletype-float②⑦">"float"</a></code>,<br><code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-unfilterable-float" id="ref-for-dom-gputexturesampletype-unfilterable-float②⑨">"unfilterable-float"</a></code> 
      </td><td rowspan="14">4 × 4 
      </td><td rowspan="14"><code class="idl"><a data-link-type="idl" href="#texture-compression-bc" id="ref-for-texture-compression-bc②">texture-compression-bc</a></code> 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-bc1-rgba-unorm-srgb" id="ref-for-dom-gputextureformat-bc1-rgba-unorm-srgb①">bc1-rgba-unorm-srgb</a></code> 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-bc2-rgba-unorm" id="ref-for-dom-gputextureformat-bc2-rgba-unorm①">bc2-rgba-unorm</a></code> 
      </td><td rowspan="2">16 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-bc2-rgba-unorm-srgb" id="ref-for-dom-gputextureformat-bc2-rgba-unorm-srgb①">bc2-rgba-unorm-srgb</a></code> 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-bc3-rgba-unorm" id="ref-for-dom-gputextureformat-bc3-rgba-unorm①">bc3-rgba-unorm</a></code> 
      </td><td rowspan="2">16 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-bc3-rgba-unorm-srgb" id="ref-for-dom-gputextureformat-bc3-rgba-unorm-srgb①">bc3-rgba-unorm-srgb</a></code> 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-bc4-r-unorm" id="ref-for-dom-gputextureformat-bc4-r-unorm①">bc4-r-unorm</a></code> 
      </td><td rowspan="2">8 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-bc4-r-snorm" id="ref-for-dom-gputextureformat-bc4-r-snorm①">bc4-r-snorm</a></code> 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-bc5-rg-unorm" id="ref-for-dom-gputextureformat-bc5-rg-unorm①">bc5-rg-unorm</a></code> 
      </td><td rowspan="2">16 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-bc5-rg-snorm" id="ref-for-dom-gputextureformat-bc5-rg-snorm①">bc5-rg-snorm</a></code> 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-bc6h-rgb-ufloat" id="ref-for-dom-gputextureformat-bc6h-rgb-ufloat①">bc6h-rgb-ufloat</a></code> 
      </td><td rowspan="2">16 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-bc6h-rgb-float" id="ref-for-dom-gputextureformat-bc6h-rgb-float①">bc6h-rgb-float</a></code> 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-bc7-rgba-unorm" id="ref-for-dom-gputextureformat-bc7-rgba-unorm①">bc7-rgba-unorm</a></code> 
      </td><td rowspan="2">16 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-bc7-rgba-unorm-srgb" id="ref-for-dom-gputextureformat-bc7-rgba-unorm-srgb①">bc7-rgba-unorm-srgb</a></code> 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-etc2-rgb8unorm" id="ref-for-dom-gputextureformat-etc2-rgb8unorm①">etc2-rgb8unorm</a></code> 
      </td><td rowspan="2">8 
      </td><td rowspan="10"><code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-float" id="ref-for-dom-gputexturesampletype-float②⑧">"float"</a></code>,<br><code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-unfilterable-float" id="ref-for-dom-gputexturesampletype-unfilterable-float③⓪">"unfilterable-float"</a></code> 
      </td><td rowspan="10">4 × 4 
      </td><td rowspan="10"><code class="idl"><a data-link-type="idl" href="#texture-compression-etc2" id="ref-for-texture-compression-etc2②">texture-compression-etc2</a></code> 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-etc2-rgb8unorm-srgb" id="ref-for-dom-gputextureformat-etc2-rgb8unorm-srgb①">etc2-rgb8unorm-srgb</a></code> 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-etc2-rgb8a1unorm" id="ref-for-dom-gputextureformat-etc2-rgb8a1unorm①">etc2-rgb8a1unorm</a></code> 
      </td><td rowspan="2">8 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-etc2-rgb8a1unorm-srgb" id="ref-for-dom-gputextureformat-etc2-rgb8a1unorm-srgb①">etc2-rgb8a1unorm-srgb</a></code> 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-etc2-rgba8unorm" id="ref-for-dom-gputextureformat-etc2-rgba8unorm①">etc2-rgba8unorm</a></code> 
      </td><td rowspan="2">16 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-etc2-rgba8unorm-srgb" id="ref-for-dom-gputextureformat-etc2-rgba8unorm-srgb①">etc2-rgba8unorm-srgb</a></code> 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-eac-r11unorm" id="ref-for-dom-gputextureformat-eac-r11unorm①">eac-r11unorm</a></code> 
      </td><td rowspan="2">8 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-eac-r11snorm" id="ref-for-dom-gputextureformat-eac-r11snorm①">eac-r11snorm</a></code> 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-eac-rg11unorm" id="ref-for-dom-gputextureformat-eac-rg11unorm①">eac-rg11unorm</a></code> 
      </td><td rowspan="2">16 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-eac-rg11snorm" id="ref-for-dom-gputextureformat-eac-rg11snorm①">eac-rg11snorm</a></code> 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-astc-4x4-unorm" id="ref-for-dom-gputextureformat-astc-4x4-unorm①">astc-4x4-unorm</a></code> 
      </td><td rowspan="2">16 
      </td><td rowspan="28"><code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-float" id="ref-for-dom-gputexturesampletype-float②⑨">"float"</a></code>,<br><code class="idl"><a data-link-type="idl" href="#dom-gputexturesampletype-unfilterable-float" id="ref-for-dom-gputexturesampletype-unfilterable-float③①">"unfilterable-float"</a></code> 
      </td><td rowspan="2">4 × 4 
      </td><td rowspan="28"><code class="idl"><a data-link-type="idl" href="#texture-compression-astc" id="ref-for-texture-compression-astc③">texture-compression-astc</a></code> 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-astc-4x4-unorm-srgb" id="ref-for-dom-gputextureformat-astc-4x4-unorm-srgb①">astc-4x4-unorm-srgb</a></code> 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-astc-5x4-unorm" id="ref-for-dom-gputextureformat-astc-5x4-unorm①">astc-5x4-unorm</a></code> 
      </td><td rowspan="2">16 
      </td><td rowspan="2">5 × 4 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-astc-5x4-unorm-srgb" id="ref-for-dom-gputextureformat-astc-5x4-unorm-srgb①">astc-5x4-unorm-srgb</a></code> 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-astc-5x5-unorm" id="ref-for-dom-gputextureformat-astc-5x5-unorm①">astc-5x5-unorm</a></code> 
      </td><td rowspan="2">16 
      </td><td rowspan="2">5 × 5 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-astc-5x5-unorm-srgb" id="ref-for-dom-gputextureformat-astc-5x5-unorm-srgb①">astc-5x5-unorm-srgb</a></code> 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-astc-6x5-unorm" id="ref-for-dom-gputextureformat-astc-6x5-unorm①">astc-6x5-unorm</a></code> 
      </td><td rowspan="2">16 
      </td><td rowspan="2">6 × 5 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-astc-6x5-unorm-srgb" id="ref-for-dom-gputextureformat-astc-6x5-unorm-srgb①">astc-6x5-unorm-srgb</a></code> 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-astc-6x6-unorm" id="ref-for-dom-gputextureformat-astc-6x6-unorm①">astc-6x6-unorm</a></code> 
      </td><td rowspan="2">16 
      </td><td rowspan="2">6 × 6 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-astc-6x6-unorm-srgb" id="ref-for-dom-gputextureformat-astc-6x6-unorm-srgb①">astc-6x6-unorm-srgb</a></code> 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-astc-8x5-unorm" id="ref-for-dom-gputextureformat-astc-8x5-unorm①">astc-8x5-unorm</a></code> 
      </td><td rowspan="2">16 
      </td><td rowspan="2">8 × 5 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-astc-8x5-unorm-srgb" id="ref-for-dom-gputextureformat-astc-8x5-unorm-srgb①">astc-8x5-unorm-srgb</a></code> 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-astc-8x6-unorm" id="ref-for-dom-gputextureformat-astc-8x6-unorm①">astc-8x6-unorm</a></code> 
      </td><td rowspan="2">16 
      </td><td rowspan="2">8 × 6 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-astc-8x6-unorm-srgb" id="ref-for-dom-gputextureformat-astc-8x6-unorm-srgb①">astc-8x6-unorm-srgb</a></code> 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-astc-8x8-unorm" id="ref-for-dom-gputextureformat-astc-8x8-unorm①">astc-8x8-unorm</a></code> 
      </td><td rowspan="2">16 
      </td><td rowspan="2">8 × 8 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-astc-8x8-unorm-srgb" id="ref-for-dom-gputextureformat-astc-8x8-unorm-srgb①">astc-8x8-unorm-srgb</a></code> 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-astc-10x5-unorm" id="ref-for-dom-gputextureformat-astc-10x5-unorm①">astc-10x5-unorm</a></code> 
      </td><td rowspan="2">16 
      </td><td rowspan="2">10 × 5 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-astc-10x5-unorm-srgb" id="ref-for-dom-gputextureformat-astc-10x5-unorm-srgb①">astc-10x5-unorm-srgb</a></code> 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-astc-10x6-unorm" id="ref-for-dom-gputextureformat-astc-10x6-unorm①">astc-10x6-unorm</a></code> 
      </td><td rowspan="2">16 
      </td><td rowspan="2">10 × 6 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-astc-10x6-unorm-srgb" id="ref-for-dom-gputextureformat-astc-10x6-unorm-srgb①">astc-10x6-unorm-srgb</a></code> 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-astc-10x8-unorm" id="ref-for-dom-gputextureformat-astc-10x8-unorm①">astc-10x8-unorm</a></code> 
      </td><td rowspan="2">16 
      </td><td rowspan="2">10 × 8 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-astc-10x8-unorm-srgb" id="ref-for-dom-gputextureformat-astc-10x8-unorm-srgb①">astc-10x8-unorm-srgb</a></code> 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-astc-10x10-unorm" id="ref-for-dom-gputextureformat-astc-10x10-unorm①">astc-10x10-unorm</a></code> 
      </td><td rowspan="2">16 
      </td><td rowspan="2">10 × 10 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-astc-10x10-unorm-srgb" id="ref-for-dom-gputextureformat-astc-10x10-unorm-srgb①">astc-10x10-unorm-srgb</a></code> 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-astc-12x10-unorm" id="ref-for-dom-gputextureformat-astc-12x10-unorm①">astc-12x10-unorm</a></code> 
      </td><td rowspan="2">16 
      </td><td rowspan="2">12 × 10 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-astc-12x10-unorm-srgb" id="ref-for-dom-gputextureformat-astc-12x10-unorm-srgb①">astc-12x10-unorm-srgb</a></code> 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-astc-12x12-unorm" id="ref-for-dom-gputextureformat-astc-12x12-unorm①">astc-12x12-unorm</a></code> 
      </td><td rowspan="2">16 
      </td><td rowspan="2">12 × 12 
     </td></tr><tr>
      <td><code class="idl"><a data-link-type="idl" href="#dom-gputextureformat-astc-12x12-unorm-srgb" id="ref-for-dom-gputextureformat-astc-12x12-unorm-srgb①">astc-12x12-unorm-srgb</a></code> 
   </td></tr></tbody></table></div>

