# Task 03 · 故障注入清单

仅出题者读取，不向学习者展示。先遵守 [注入执行规则](README.md) 与 AGENTS.md。每次只选一项，只改一个代码位置，不顺带修改 descriptor、shader 或图像资源。

## S03-01 · 上传后缺少正确的最终 layout

- 适用条件：纹理拷贝后依靠一条转换进入采样所需的 layout，descriptor 与后续访问都明确依赖该结果，后面没有补做转换。
- 单处注入：只把该转换的目标 layout 改为继续保持上传目的 layout；其他字段保持原样。
- 出题者核对：目标为实际被采样的纹理，不能选择不参与绘制的资源。
- 内部预期：图像实际状态与后续使用要求不一致；可能出现 layout 诊断或异常采样。
- 正确根因：用户指出哪张图像在哪个转换后保留了错误状态，说明后续访问为何不满足条件。
- 内部依据：[Vulkan Spec：Resource Creation / Image Layouts](https://docs.vulkan.org/spec/latest/chapters/resources.html)。

## S03-02 · 上传写入未进入内存依赖

- 适用条件：同一提交路径中，纹理拷贝写入到首次采样的可见性仅由这条 barrier 保证；不存在另一个覆盖该写读关系的 barrier 或 semaphore 依赖。
- 单处注入：只清空该 barrier 的源访问范围，保留 stage、layout、目标访问和图像范围。
- 出题者核对：原源访问确实覆盖上传写入；如果已有别的有效可见性保证，本项不适用。
- 内部预期：执行顺序与 layout 看似合理，但所需写入可见性没有被这条依赖建立；可能被同步验证发现，画面也可能暂时正常。
- 正确根因：用户定位缺失的源访问范围，结合真实生产者 / 消费者说明它为何必要；泛称“需要 barrier”不算通过。
- 内部依据：[Synchronization Examples：Transfer Dependencies](https://docs.vulkan.org/guide/latest/synchronization_examples.html)。

## S03-03 · 转换遗漏被采样的 mip

- 适用条件：已有至少两个 mip，原转换范围覆盖它们，且 sampler 的有效采样会访问更高 mip；图像此前在这些 mip 上有真实上传使用。
- 单处注入：只缩小这一条转换的 mip 范围，使一个确实会被采样的已上传 mip 被遗漏。
- 出题者核对：保持 image、aspect、array layer 与 descriptor 不变；只有单 mip 或根本不使用高 mip 时不可选本项。
- 内部预期：遗漏 mip 的状态与 descriptor / 采样要求不一致，症状可能随观察距离变化。
- 正确根因：用户指出遗漏的具体子资源和转换，说明为何某些观察距离或采样会触发问题。
- 内部依据：[Vulkan Spec：Resource Creation / Image Subresources](https://docs.vulkan.org/spec/latest/chapters/resources.html)。

## 判定与收尾

以本次 `.sabotage-active` 记录为准。不要求用户猜中固定症状；要求其说明具体根因和证据。没有适用项时不注入，不扩大题库后偷偷替换。用户修复后重新验证纹理与同步状态。
