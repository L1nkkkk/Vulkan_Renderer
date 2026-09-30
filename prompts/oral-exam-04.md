# Task 04 · 口试评分标准

仅考官使用。先遵守 [口试执行规则](README.md)。共 5 题，5/5 通过。不得替用户补写其 review 报告。

## 1. 延迟销毁

题目：deletion queue 为什么要按帧延迟销毁？延迟几帧才安全？

通过标准：
- 区分 CPU 不再持有与 GPU 不再使用，指出自己资源的最后一次使用及完成证明。
- 不把固定等 N 帧当作充分条件；说明帧槽回收、多个提交或队列、呈现相关对象如何影响判断。

未通过示例：只回答“双缓冲所以等两帧”；认为 C++ 析构触发就代表 GPU 已完成。

待补阅读：[Vulkan Spec：Fundamentals / Object Lifetime](https://docs.vulkan.org/spec/latest/chapters/fundamentals.html)；[Swapchain Semaphore Reuse](https://docs.vulkan.org/guide/latest/swapchain_semaphore_reuse.html)。

## 2. Descriptor 与 image 生命周期

题目：一个 descriptor set 引用的 image 被销毁了但 set 还在，会发生什么？

通过标准：
- 说明 descriptor set 不会自动延长底层 image / view 的生命周期；set 对象存在不代表它引用的资源仍有效。
- 区分仍有已提交访问时销毁、已无使用但 descriptor 留有旧引用、未来重新使用这几种情况，说明需要保证什么才可继续访问。

未通过示例：认为 descriptor 拥有 image 并自动引用计数；把不再使用的旧引用与正在访问的资源混为一谈。

待补阅读：[Vulkan Spec：Descriptor Sets](https://docs.vulkan.org/spec/latest/chapters/descriptorsets.html)；[Resource Creation](https://docs.vulkan.org/spec/latest/chapters/resources.html)。

## 3. 材质排序

题目：材质排序的收益来自哪里？在 Vulkan 下比在 OpenGL 下收益大还是小？

通过标准：
- 联系 pipeline / binding 切换、状态复用及可能的缓存行为说明收益，并指出透明排序等正确性约束。
- 不给跨 API 的固定大小结论，能区分 CPU 驱动开销、GPU 工作与场景特征，提出对比测量方法。

未通过示例：认为材质排序一定减少 draw 数量，或认为 Vulkan 没有状态切换成本。

待补阅读：[Vulkan Spec：Pipelines](https://docs.vulkan.org/spec/latest/chapters/pipelines.html)；[Writing an efficient Vulkan renderer](https://zeux.io/2020/02/27/writing-an-efficient-vulkan-renderer/)。

## 4. 对生成实现的 review

题目：agent 生成的代码里你最不满意的一处是什么？如果是你写会怎么做？

通过标准：
- 指向用户实际 review 过的一处代码，用约束、失败条件或维护成本说明问题。
- 说明自己作出的修改或有依据的保留决定，并给出验证结果，不能只复述 agent 的评论。

未通过示例：没有实际代码位置；只有喜好，没有行为影响或取舍。

待补阅读：用户自己的 `docs/task04-review.md`、对应生成实现与修改记录。

## 5. ImGui 后端

题目：ImGui 的 Vulkan 后端自己管理了哪些资源？它和你的 deletion queue 如何共处？

通过标准：
- 依据项目实际使用的 ImGui 版本与初始化配置，区分应用提供的对象和后端创建 / 持有的资源。
- 说明 shutdown、字体 / 纹理相关资源、descriptor pool 与重建路径的归属，避免重复销毁或过早释放；不背与版本无关的固定清单。

未通过示例：认为所有 Vulkan 对象都由 ImGui 释放；无法指出实际初始化与关闭路径。

待补阅读：[ImGui Vulkan backend source](https://github.com/ocornut/imgui/blob/master/backends/imgui_impl_vulkan.cpp)；项目锁定版本的后端头文件和示例。
