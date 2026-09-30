# Task 06 · 帧分析与 RHI 对照

状态：等待 Task 05。建议分支：`task/06-analysis`。

学习入口：[Task 06 学习资料与阅读顺序](../README.md#task06-learning)。先分析自己的帧，再带着五个对照维度读 NVRHI 文档和接口。

目标：用真实捕获和成熟 RHI 源码审视已有实现。本 Task 不增加渲染功能。

## 分工

| agent 可直接生成 | agent 只可提问 / review | 必须独立完成 |
|---|---|---|
| 无 | 用户的分析与对照表的事实校对；解释用户读不懂的 NVRHI 设计 | RenderDoc 分析、Nsight 对照、问题修复、两份分析文档 |

agent 不替用户抓结论、填对照表或起草分析。讨论源码时核对实际版本，不把 RHI 分类标签当作功能事实。

## 执行检查表

- [ ] 06.1 确认 05 全部验收通过，选定固定场景、分辨率和相机条件。
- [ ] 06.2 用户捕获一帧，按 draw / pass 检查资源绑定、状态与 barrier，记录具体事件或代码位置。
- [ ] 06.3 用户提出多余或缺失 barrier 的判断，基于证据自行修复并重新捕获对照；未发现问题时如实记录，不制造结论。
- [ ] 06.4 在相同条件下检查 Nsight GPU timeline，对照自己的 timestamp，说明范围差异和测量限制。
- [ ] 06.5 用户通读所选 NVRHI 版本的 `include/nvrhi/nvrhi.h` 与相关 Programming Guide，记录 commit / tag 和疑问。
- [ ] 06.6 围绕 barrier、descriptor、PSO、command list、资源生命周期五项查证接口及实现，用户自行填对照表。
- [ ] 06.7 用户阅读《Writing an efficient Vulkan renderer》，记录与自己实现相关的观察和未验证假设。
- [ ] 06.8 用户完成 `docs/task06-frame-analysis.md` 与 `docs/task06-rhi-compare.md`，需要时请求 agent 校对事实。
- [ ] 06.9 完成 `/exam 06`。

## 验收证据

| 内容 | 通过条件 |
|---|---|
| 帧分析 | 判断有对应事件、资源或源码位置；已确认的正确性问题修复并复验 |
| 性能观察 | 保持可比的运行条件，不用一次测量或 Validation 沉默证明性能结论 |
| RHI 对照 | 五个维度完整，引用可定位的版本和接口，区分自动处理与用户责任 |
| 文档作者 | 分析、结论与取舍由用户写出；agent 仅校对事实并追加记录 |

## 结束条件

- [ ] 两份文档完成，已发现问题妥善处理；“未发现”也须有检查范围与证据。
- [ ] `docs/oral/task06.md` 记录口试 4/4 通过。
- [ ] 更新总进度；按用户指令标记 `task-06`。

[返回任务入口](README.md)。
