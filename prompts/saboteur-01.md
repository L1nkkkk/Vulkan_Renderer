# Task 01 · 故障注入清单

仅出题者读取，不向学习者展示。先遵守 [注入执行规则](README.md) 与 AGENTS.md。每次只选一项，只改一个代码位置；要求应用仍可构建，故障来自真实的同步约束。

## S01-01 · 丢失呈现等待

- 适用条件：呈现依赖渲染提交发出的 binary semaphore；没有其他等价的完成保证。
- 单处注入：在最终呈现参数构造处取消对该 semaphore 的等待，保持渲染提交及其 signal 不变。
- 出题者核对：只有这个等待发生变化；没有顺手调整 stage、对象数量、图像索引或其他提交。
- 内部预期：呈现缺少渲染完成依赖，可能有画面异常、同步诊断或后续 semaphore 复用错误；不保证每次都出现相同症状。
- 正确根因：用户指出具体呈现等待被取消，说明生产者、消费者与被破坏的顺序；只说“semaphore 有问题”不足以通过。
- 内部依据：[Vulkan Spec：WSI](https://docs.vulkan.org/spec/latest/chapters/VK_KHR_surface/wsi.html)。

## S01-02 · 丢失帧资源完成等待

- 适用条件：复用帧槽前依靠该 fence wait 证明上一轮提交完成；没有其他已经成立的完成证明。
- 单处注入：取消这一处帧资源复用前的 fence wait，保留其余操作。
- 出题者核对：该等待确实是对应复用路径的必要条件，避免在另有有效等待的实现中注入无效题目。
- 内部预期：帧资源可能在仍被使用时重录、reset 或修改；可能被 Validation 指出，也可能依赖负载才暴露。
- 正确根因：用户定位被取消的等待，指出至少一个仍可能在使用的具体对象及复用为什么不安全。
- 内部依据：[Vulkan Spec：Synchronization](https://docs.vulkan.org/spec/latest/chapters/synchronization.html)。

## S01-03 · 丢失提交 fence 的 reset

- 适用条件：该路径使用已 signaled 的 fence，并依靠唯一的 reset 使其可用于下一次提交；取消一个调用不会导致语法错误。
- 单处注入：取消这一次 reset，保留等待与后续提交。
- 出题者核对：不存在其他 reset 或替换对象路径，确保一次修改就能触发明确的状态错误。
- 内部预期：再次提交时 fence 状态不满足要求，通常产生 Validation 诊断；后续症状以实际实现为准。
- 正确根因：用户说明 wait 与 reset 的职责不同，并定位 signaled fence 被重新用于提交的路径。
- 内部依据：[Vulkan Spec：Synchronization / Fences](https://docs.vulkan.org/spec/latest/chapters/synchronization.html)。

## 判定与收尾

以本次 `.sabotage-active` 记录为准，只判断被注入的根因。注入后只回复规定短句；排查中不给方向。用户修复后需要重新验证正常运行，不能把偶然没有报错当成练习结束。
