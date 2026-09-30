# Task 00 · 环境与协作约定

状态：待开始。前置：无。建议分支：`task/00-env`。

学习入口：[Task 00 学习资料与阅读顺序](../README.md#task00-learning)。先从 Building Project 开始，边读边完成下面的检查项。

目标：能够复现 starter 的构建与启动，并说清构建配置的用途。本 Task 无口试。

## 分工

| agent 可直接生成 | agent 只可提问 / review | 必须独立完成 |
|---|---|---|
| CMake、vcpkg 清单、shader 编译脚本 | 用户对构建配置的复述可讨论核对 | `AGENTS.md`、`docs/task00-env.md`；工具运行与环境验收 |

现有 AGENTS.md 已提供。工具是否安装、版本是否满足需求、starter 是否能启动，均尚未验收。规划资料已在 main 建立，不计作 starter 完成。

## 执行检查表

- [ ] 00.1 用户确认 GPU、驱动、Vulkan SDK 和 C++ 编译器的实际版本，运行 `vkcube` 观察结果。
- [ ] 00.2 核对 CMake、vcpkg、RenderDoc、Nsight Graphics；记录可用版本与暂不可用的工具。
- [ ] 00.3 选定 vkguide 2.0 starting point 的具体来源与提交；核对教程使用的窗口库和依赖，不混用 SDL 版本的 API。记录继续使用教程依赖还是适配 SDL3 / GLFW 的决定。
- [ ] 00.4 用户发出具体 `/gen` 请求，agent 只准备构建清单与编译脚本；教程源码作为带来源的外部基线保留。
- [ ] 00.5 从干净的构建目录配置并编译，启动 starter；由实际构建和运行结果判断成功。
- [ ] 00.6 agent 逐段解释构建配置，用户复述并逐行注释 agent 生成的代码，再自行记录环境和问题。
- [ ] 00.7 核对 `prompts/`、`docs/`、`docs/oral/` 和 `docs/debug-log.md`；确认协作规则与角色入口能够使用。

## 验收证据

| 检查 | 通过条件 | 证据位置 |
|---|---|---|
| 基础 Vulkan 环境 | `vkcube` 实际出画面 | 用户的 `docs/task00-env.md` |
| starter | 清晰的配置、构建和启动步骤能复现 | 同上，包含来源提交和构建参数 |
| 构建理解 | 用户能解释每段配置以及 shader 编译产物的用途 | 用户复述与自行记录 |
| 协作规则 | agent 正确复述分工、禁止事项和当前角色 | 当前会话 |

## 结束条件

- [ ] `vkcube` 与 starter 均已实际启动。
- [ ] `docs/task00-env.md` 由用户完成，说明 SDK / 驱动版本及构建问题。
- [ ] 用户确认生成代码已理解并完成注释，按约定分别提交。
- [ ] 更新 README 为 00 完成、01 待开始；不执行 `/exam 00`，不预写考试记录。

[返回任务入口](README.md)。
