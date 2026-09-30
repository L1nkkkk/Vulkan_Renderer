# Task 02 · 口试评分标准

仅考官使用。先遵守 [口试执行规则](README.md)。共 5 题，5/5 通过。

## 1. PSO 与材质

题目：为什么 Vulkan 把 PSO 设计成不可变对象？这对上层材质系统意味着什么？

通过标准：
- 解释把 shader 与固定状态组合的验证 / 编译成本放在创建阶段的意义，同时区分 pipeline 中声明为 dynamic 的状态。
- 说明材质变体、pipeline 复用及缓存身份的关系，不把每次材质参数变化都等同于重建 PSO。

未通过示例：认为所有绘制状态永远不能改变；只回答“性能更好”而没有因果。

待补阅读：[Vulkan Spec：Pipelines](https://docs.vulkan.org/spec/latest/chapters/pipelines.html)。

## 2. Dynamic state

题目：哪些状态适合 dynamic state？把所有能 dynamic 的都 dynamic 有什么代价？

通过标准：
- 从自己的绘制需求举例，区分运行时变化频率、pipeline 变体数量与支持的 feature / extension。
- 解释命令设置、状态追踪和可能的驱动优化取舍；不声称 dynamic 一定更快或一定更慢。

未通过示例：忽略设备能力；声明 dynamic 后仍认为创建时的值总能自动补上。

待补阅读：[Vulkan Guide：Pipeline Dynamic State](https://docs.vulkan.org/guide/latest/dynamic_state.html)。

## 3. Buffer device address

题目：buffer device address 相比 vertex input binding 的优劣是什么？

通过标准：
- 比较 shader 访问灵活性与固定顶点输入描述方式，说明自己的数据布局和索引访问责任。
- 解释能力要求、地址有效期和调试 / 移植约束，不把 GPU 地址当作可由 CPU 随意解引用的指针，也不预设性能收益。

未通过示例：认为拿到地址后 buffer 可以销毁；认为 BDA 自动解决内存布局和同步。

待补阅读：[Vulkan Guide：Buffer Device Address](https://docs.vulkan.org/guide/latest/buffer_device_address.html)。

## 4. 深度格式

题目：深度 attachment 的 format 选择依据是什么？D24 和 D32 在什么情况下有差别？

通过标准：
- 说明先核对所需用途的格式支持，并考虑 stencil、存储和精度需求；准确说出自己使用的具体格式。
- 区分常见 D24 UNORM 与 D32 SFLOAT 的数值表示，结合投影和深度范围讨论精度；不只用“位数更大所以一切更好”回答。

未通过示例：凭名称假定所有设备、所有用途均支持；把深度格式与 clear / compare 的选择完全割裂。

待补阅读：[Vulkan Guide：Depth](https://docs.vulkan.org/guide/latest/depth.html)。

## 5. 自己的接口

题目：你的 `PipelineBuilder` 接口里，哪个方法是你最不确定的？为什么？

通过标准：
- 指出实际头文件中的方法，说明意图、尚不确定的契约和可能造成的调用问题。
- 比较至少一种替代取舍，说明怎样用使用场景或验证来作决定；允许有理由地保留现状。

未通过示例：只谈命名喜好，无法解释方法对管线状态或生命周期的影响。

待补阅读：用户的 `PipelineBuilder.h` 与既有 review 记录；[Vulkan Spec：Pipelines](https://docs.vulkan.org/spec/latest/chapters/pipelines.html)。
