# Task 02 · 管线与网格

状态：等待 Task 01。建议分支：`task/02-mesh`。

目标：从三角形走到可旋转、遮挡正确的网格，第一次完成“用户先定接口、agent 再实现”的循环。

## 分工

| agent 可直接生成 | agent 只可提问 / review | 必须独立完成 |
|---|---|---|
| 按用户头文件实现 `PipelineBuilder`、VMA buffer 上传辅助函数、GLSL 编译集成 | `PipelineBuilder.h` 的接口 review | `PipelineBuilder.h`、depth attachment 配置、`docs/task02-pso.md` |

未列为可生成的 shader、绘制连接逻辑等，由用户完成；有需要时先明确归属，不因存在辅助类就扩大生成范围。

## 执行检查表

- [ ] 02.1 确认 01 的口试、排错和文档都通过，保留可复现的清屏基线。
- [ ] 02.2 用户先写 `PipelineBuilder.h`，为每个方法说明意图、输入约束和所有权。
- [ ] 02.3 执行 `/review <PipelineBuilder.h 的实际路径>`；用户记录问题与取舍，并自行修改接口。
- [ ] 02.4 agent 按确认后的接口实现允许的辅助部分；用户逐行注释、核对实现与接口意图。
- [ ] 02.5 用户完成三角形绘制，确认 GLSL 编译、SPIR-V 加载与图形管线能够串起来。
- [ ] 02.6 接入 VMA 网格上传和 buffer device address 读取顶点，检查 CPU 与 shader 对数据布局的理解。
- [ ] 02.7 用户自行配置 depth attachment、深度测试与 push constant MVP，观察旋转网格的遮挡。
- [ ] 02.8 切换 shader 或相关 PSO，检查无关资源是否保持稳定；用 RenderDoc 查看 pipeline 与 mesh。
- [ ] 02.9 用户完成 `docs/task02-pso.md`，包含状态清单、dynamic state 选择、PSO hash key 的判断和接口 review 取舍。
- [ ] 02.10 完成 `/exam 02`。

## 验收证据

| 场景 | 通过条件 |
|---|---|
| 三角形与网格 | 顶点数据、坐标变换和绘制结果一致 |
| 前后遮挡与窗口变化 | 深度结果正确，附件尺寸与渲染配置一致 |
| shader / PSO 切换 | 变化范围符合用户说明，无关对象不被重建 |
| 接口先行 | 能追溯到用户先写的头文件、review 取舍和生成实现 |

## 结束条件

- [ ] 网格、深度和资源重建范围通过验收，Validation 无错误。
- [ ] 用户文档及 `docs/oral/task02.md` 齐备，口试 5/5 通过。
- [ ] 按分工完成注释与提交，更新总进度；按用户指令标记 `task-02`。

阅读：[vkguide](https://vkguide.dev/) Chapter 2、3；[Vulkan Spec：Pipelines](https://docs.vulkan.org/spec/latest/chapters/pipelines.html)；[任务入口](README.md)。
