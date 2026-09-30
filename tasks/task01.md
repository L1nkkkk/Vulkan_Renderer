# Task 01 · 帧循环与同步

状态：等待 Task 00。建议分支：`task/01-sync`。

目标：用自己的帧循环画出随时间变化的清屏颜色，并能解释每项同步的必要性。

## 分工

| agent 可直接生成 | agent 只可提问 / review | 必须独立完成 |
|---|---|---|
| vk-bootstrap 初始化、窗口事件循环、swapchain 重建骨架 | 同步对象编排、layout barrier 的选择 | `FrameData`、每帧 acquire → record → submit → present 逻辑、`docs/task01-sync.md` |

生成初始化或重建骨架时，不代填用户的同步与资源状态逻辑。文件混合了两类职责时，只改已明确授权的区域。

## 执行检查表

- [ ] 01.1 确认 00 验收通过，并能重新启动 starter。
- [ ] 01.2 用户说明初始化所需行为；agent 仅完成允许的初始化和窗口部分。
- [ ] 01.3 用户设计帧资源及其归属，说明 frame index 和 swapchain image index 各自表示什么。
- [ ] 01.4 用户实现帧循环和清屏，标出资源复用、录制、提交与呈现各自的约束。
- [ ] 01.5 用户补齐 resize、最小化、恢复和 acquire / present 返回异常状态的处理，再验证资源重建路径。
- [ ] 01.6 开启 Validation 和同步验证；遇到报错先用一句话复述含义，再讨论排查。
- [ ] 01.7 用 RenderDoc 捕获一帧，逐一检查 README 附录 B 的面板，指出与自己录制命令对应的事件。
- [ ] 01.8 用户自行完成同步流程图和 `docs/task01-sync.md`，结合实际代码解释复用依据。
- [ ] 01.9 主要功能稳定后执行 `/sabotage 01`，独立排查并说明根因；将实际过程记入 debug-log。
- [ ] 01.10 完成 `/exam 01`，未通过题目补阅读后重考。

## 验收证据

| 场景 | 通过条件 |
|---|---|
| 正常运行 | 颜色连续变化，Validation 无错误 |
| 反复 resize、最小化和恢复 | 无崩溃或无法继续绘制，能解释重建对象范围 |
| 1 / 2 / 3 帧 in-flight | 都能正确运行，并能区分正确性与延迟 / 吞吐观察 |
| 资源复用 | 分别说明帧资源与呈现相关资源的完成依据 |
| 工具与排错 | 有捕获观察记录，注入根因由用户独立定位 |

## 结束条件

- [ ] 清屏、重建与同步验收通过。
- [ ] 用户文档及 `docs/oral/task01.md` 齐备，口试 5/5 通过。
- [ ] 注入问题由用户找到，练习状态已结束；修复后重新验证正常运行。
- [ ] 回顾 00～01 的 debug-log；更新总进度，按用户指令结束分支与标记 `task-01`。

阅读：[vkguide：Rendering Loop](https://vkguide.dev/docs/new_chapter_1/vulkan_mainloop/)；[Vulkan Guide：Swapchain Semaphore Reuse](https://docs.vulkan.org/guide/latest/swapchain_semaphore_reuse.html)；[任务入口](README.md)。
