# Task 06 · 口试评分标准

仅考官使用。先遵守 [口试执行规则](README.md)。共 4 题，4/4 通过。源码问题按用户记录的 NVRHI 版本核对。

## 1. NVRHI 的状态管理边界

题目：NVRHI 的自动状态跟踪和 barrier 处理覆盖哪些范围？用户还需要提供哪些边界信息，什么时候会选择手动管理？

通过标准：
- 知道 NVRHI 支持自动状态跟踪与 barrier，而不是只能原样暴露 Vulkan barrier。
- 结合所读版本解释 command list 边界所需信息，以及选择自动 / 手动控制的原因，能指出相关接口或源码。

未通过示例：沿用“NVRHI 完全不自动处理”的前提；认为自动跟踪消除了所有上层资源与提交约束。

待补阅读：[NVRHI Programming Guide：State Tracking and Barriers](https://github.com/NVIDIA-RTX/NVRHI/blob/main/doc/ProgrammingGuide.md#state-tracking-and-barriers)。

## 2. 多余 barrier

题目：你找到的多余 barrier 为什么 validation 不报？它的实际代价是什么？

通过标准：
- 用自己的捕获说明正确性约束与性能优化的区别，指出过度同步可能限制重叠或带来不必要的处理。
- 性能代价有同条件对照；若实际没有发现可删除 barrier，可以如实说明检查范围和判断依据，再讨论一个明确标注为假设的例子，不要求编造发现。

未通过示例：认为 validation 会指出所有性能浪费；只用“一定慢很多”代替测量。

待补阅读：[Vulkan Samples：Using Pipeline Barriers Efficiently](https://docs.vulkan.org/samples/latest/samples/performance/pipeline_barriers/README.html)。

## 3. Binding 抽象

题目：NVRHI 的 `BindingLayout` / `BindingSet` 和你的 descriptor 抽象相比，多解决了什么问题？

通过标准：
- 以实际接口为依据比较布局描述与具体资源绑定，指出两者在自身实现与 NVRHI 中的对应关系。
- 至少具体说明一个跨 API 表达、资源管理或验证方面的差异及其代价；不能凭名称推断保证，也允许说明某项能力两者相同。

未通过示例：只回答“封装得更好”；无法指出自己抽象的边界。

待补阅读：[NVRHI interface](https://github.com/NVIDIA-RTX/NVRHI/blob/main/include/nvrhi/nvrhi.h)；[Programming Guide](https://github.com/NVIDIA-RTX/NVRHI/blob/main/doc/ProgrammingGuide.md)。

## 4. 映射到 D3D12

题目：如果要支持 D3D12 后端，你现有代码里哪一处最难映射？

通过标准：
- 选择一个真实接口或行为，指出依赖的 Vulkan 语义以及跨 API 时需要重新表达的约束。
- 说明当前掌握的证据和未知部分，给出可验证的调查方向或边界调整；不要求编写 D3D12 实现。

未通过示例：只说函数名不同；猜测目标 API 行为并把猜测当作事实。

待补阅读：[NVRHI interface](https://github.com/NVIDIA-RTX/NVRHI/blob/main/include/nvrhi/nvrhi.h)；[Microsoft：Direct3D 12 Programming Guide](https://learn.microsoft.com/en-us/windows/win32/direct3d12/directx-12-programming-guide)。
