# Task 01 · 口试评分标准

仅考官使用。先遵守 [口试执行规则](README.md)。共 5 题，5/5 通过；最多一次不带提示的澄清追问。

## 1. 呈现等待

题目：没有其他等价同步时，去掉 presentation 对 render-finished semaphore 的等待会破坏什么保证？为什么不能只凭画面或 validation 判断正确性？

通过标准：
- 区分渲染提交与呈现所需的完成关系，说明缺少依赖可能让呈现与渲染发生冲突。
- 说明未必稳定出现某一种画面故障；工具版本、启用的验证范围和执行条件影响观察结果，未报错不构成正确性证明。

未通过示例：只说“一定黑屏”；以同一 queue 或没有报错作为无需同步的全部理由。

待补阅读：[Vulkan Spec：WSI / Queue Presentation](https://docs.vulkan.org/spec/latest/chapters/VK_KHR_surface/wsi.html)。

## 2. Frames-in-flight

题目：frames-in-flight 设为 1、2、3 分别对延迟和吞吐有什么影响？

通过标准：
- 说明 CPU / GPU 可重叠工作的数量、资源占用与排队延迟之间的关系。
- 区分允许的并行程度与实际性能，结合瓶颈及呈现设置解释为什么不能保证 3 比 2 更快。

未通过示例：把帧数与 FPS 等同；把 frames-in-flight 和 swapchain image 数量认作同一个配置。

待补阅读：[vkguide：Rendering Loop](https://vkguide.dev/docs/new_chapter_1/vulkan_mainloop/)；[Vulkan Spec：Synchronization](https://docs.vulkan.org/spec/latest/chapters/synchronization.html)。

## 3. 两种 index

题目：`vkAcquireNextImageKHR` 返回的 index 和当前 frame index 为什么不是同一个东西？

通过标准：
- 分别解释获取的 swapchain image 与应用帧资源槽，说明数量及轮转不必相同。
- 结合自己的代码指出对象使用哪个 index，以及复用的完成依据；不能用渲染提交 fence 的完成证明呈现已结束。

未通过示例：只是背诵“两种 index 不一样”，无法解释自己选取对象的依据。

待补阅读：[Vulkan Guide：Swapchain Semaphore Reuse](https://docs.vulkan.org/guide/latest/swapchain_semaphore_reuse.html)。

## 4. Resize

题目：窗口 resize 时哪些对象要重建、哪些不用？为什么？

通过标准：
- 根据自己实现中的 extent、format 和引用关系判断 swapchain、views、相关附件等对象；不背固定清单。
- 区分动态和静态管线状态，说明哪些配置改变才影响 pipeline；能解释旧资源使用完成与零尺寸窗口的处理约束。

未通过示例：无条件销毁整个 device；认为 resize 只影响窗口而与附件无关。

待补阅读：[Vulkan Spec：WSI / Swapchain](https://docs.vulkan.org/spec/latest/chapters/VK_KHR_surface/wsi.html)；[Vulkan Guide：Pipeline Dynamic State](https://docs.vulkan.org/guide/latest/dynamic_state.html)。

## 5. Wait 与 acquire 的顺序

题目：把 fence wait 放到 acquire 之后时，需要检查哪些对象的复用条件？为什么不能只靠调用顺序判断安全性？

通过标准：
- 结合实际实现检查 acquire 所用同步对象和帧资源是否仍在使用；能说明后面的等待不能倒过来保证已经发起的操作合法。
- 不断言这种顺序在所有设计中都错误；判断取决于是否另有完成证明，并能检查提前返回路径中的 fence 状态。

未通过示例：只说“教程先 wait 所以必须先 wait”，或“acquire 返回就代表全部 GPU 工作完成”。

待补阅读：[Vulkan Spec：vkAcquireNextImageKHR](https://docs.vulkan.org/refpages/latest/refpages/source/vkAcquireNextImageKHR.html)；[Vulkan Spec：Synchronization](https://docs.vulkan.org/spec/latest/chapters/synchronization.html)。
