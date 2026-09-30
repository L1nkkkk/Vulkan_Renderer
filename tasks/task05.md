# Task 05 · 多 pass 与 GPU 计时

状态：等待 Task 04。建议分支：`task/05-passes`。

目标：增加 shadow pass，在主 pass 采样阴影，并能用工具解释各 pass 的 GPU 时间。

## 分工

| agent 可直接生成 | agent 只可提问 / review | 必须独立完成 |
|---|---|---|
| shadow shader、按用户头文件实现 `GpuTimer`、ImGui 曲线绘制 | pass 依赖图与 `GpuTimer.h` | pass 依赖图、shadow map barrier、`GpuTimer.h`、`docs/task05-passes.md`；未列为可生成的 pass 连接逻辑 |

先有用户的依赖图和计时接口，再生成实现。图和接口只允许 review，不由 agent 补齐设计。

## 执行检查表

- [ ] 05.1 确认 04 全部验收通过，选择继续进阶路线。
- [ ] 05.2 用户画出 pass 与资源读写关系，标明跨 pass 的依赖及自己判断的 barrier 位置。
- [ ] 05.3 用户写 `GpuTimer.h`，明确测量边界、容量、结果所属帧、未就绪和回收行为；请求 review 并自行修改。
- [ ] 05.4 agent 实现允许的 shader、计时器和曲线；用户审查与逐行注释。
- [ ] 05.5 用户实现 shadow pass 与主 pass 的连接，手写 shadow map 状态转换和同步依据。
- [ ] 05.6 验证阴影、光源变化与 bias，检查 shadow map 实际内容和主 pass 采样结果。
- [ ] 05.7 在不同负载与帧数设置下观察计时结果，验证结果未就绪时的行为；不以固定经过一帧证明 GPU 已完成。
- [ ] 05.8 用 Nsight 对照相同场景下的 GPU 时间，记录测量范围、工具开销与量级差异。
- [ ] 05.9 用户完成 `docs/task05-passes.md`，讨论依赖图与 render graph 的关系，区分 IMR 实测和 TBR 推测。
- [ ] 05.10 完成 `/exam 05`。

## 验收证据

| 场景 | 通过条件 |
|---|---|
| shadow / main pass | 阴影正确，读写资源与用户依赖图一致 |
| 同步 | barrier 理由由用户说明，Validation 无错误 |
| query 生命周期 | 重用、未就绪与结果归属符合用户接口约定 |
| GPU 时间 | 数值单位正确，与 Nsight 同范围测量的量级一致；异常有解释 |

## 结束条件

- [ ] 阴影、曲线和工具对照完成。
- [ ] 用户文档及 `docs/oral/task05.md` 齐备，口试 5/5 通过。
- [ ] 回顾 04～05 的 debug-log，更新总进度；按用户指令标记 `task-05`。

阅读：[Vulkan Spec：Queries](https://docs.vulkan.org/spec/latest/chapters/queries.html)；[Sascha Willems examples](https://github.com/SaschaWillems/Vulkan) 的 shadowmapping / timestampqueries；[任务入口](README.md)。
