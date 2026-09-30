# Task 03 · 描述符、纹理与 barrier

状态：等待 Task 02。建议分支：`task/03-resources`。

目标：加载带贴图、多网格、多材质的 glTF 场景，并逐条解释资源状态转换。

## 分工

| agent 可直接生成 | agent 只可提问 / review | 必须独立完成 |
|---|---|---|
| fastgltf 加载与解包、staging 拷贝流程、按用户头文件实现 `DescriptorAllocator` / `DescriptorWriter` | 每条 barrier 的 stage / access 选择 | 全部 `vkCmdPipelineBarrier2` 调用及理由、descriptor set layout、两个辅助类接口、`docs/task03-barrier.md` |

上传辅助函数不包含 agent 代写的 barrier；遇到边界先让用户提供接口或自行补齐。不能用隐藏在工具函数中的状态转换绕过分工。

## 执行检查表

- [ ] 03.1 确认 02 全部验收通过，网格与深度功能可复现。
- [ ] 03.2 用户设计相机 UBO 与 descriptor set layout，写出 allocator / writer 接口及容量和生命周期约束。
- [ ] 03.3 agent 只实现已给出的接口与允许的加载辅助部分；用户 review 并逐行注释。
- [ ] 03.4 用户将相机矩阵改由 UBO 提供，核对布局与多帧数据复用。
- [ ] 03.5 先加载并采样单张纹理；用户完成每个 barrier 及其 src / dst stage、access、layout、范围选择的注释。
- [ ] 03.6 扩展到多网格多材质 glTF，记录样例资源来源；用 RenderDoc 对照 mesh、descriptor 和 image 状态。
- [ ] 03.7 验证 descriptor 分配增长、场景重复使用与退出，检查生命周期和 Validation 输出。
- [ ] 03.8 用户完成 `docs/task03-barrier.md`，逐条列出真实 barrier，讨论必要性、可合并性和 TBR 上需要验证的判断。
- [ ] 03.9 主要功能稳定后执行 `/sabotage 03`；用户独立定位并记录根因。
- [ ] 03.10 完成 `/exam 03`。

## 验收证据

| 场景 | 通过条件 |
|---|---|
| 单纹理与完整 glTF | 贴图、网格与材质对应正确 |
| 资源状态 | 文档、代码与捕获中观察到的状态一致 |
| pool 容量变化 | 预设容量不足时按用户设计处理，无无效 descriptor 使用 |
| 同步理解 | 每条 barrier 有用户写的理由；注入问题由用户独立找到 |

## 结束条件

- [ ] 场景、描述符与同步验收通过，Validation 无错误。
- [ ] 用户文档及 `docs/oral/task03.md` 齐备，口试 5/5 通过。
- [ ] 注入练习已正确结束，正常路径已再次验证。
- [ ] 回顾 02～03 的 debug-log，更新总进度；按用户指令标记 `task-03`。

阅读：[Vulkan Guide：Synchronization Examples](https://docs.vulkan.org/guide/latest/synchronization_examples.html)；[Vulkan Spec：Descriptor Sets](https://docs.vulkan.org/spec/latest/chapters/descriptorsets.html)；[任务入口](README.md)。
