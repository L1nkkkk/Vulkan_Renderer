# Task 00 · 环境与协作约定

状态：待开始。前置：无。建议分支：`task/00-env`。

学习入口：[Task 00 学习资料与阅读顺序](../README.md#task00-learning)。先读 SDL3 的 CMake 接入与程序入口，再用 vkguide 对照项目结构。

目标：能够复现 SDL3 最小窗口 starter 的构建与启动，并说清构建配置的用途。本 Task 无口试；Vulkan 初始化与帧循环从 Task 01 开始。

## 分工

| agent 可直接生成 | agent 只可提问 / review | 必须独立完成 |
|---|---|---|
| CMake、vcpkg 清单、shader 编译脚本、SDL3 最小窗口启动样板（仅创建 / 关闭与退出） | 用户对构建配置的复述可讨论核对 | `AGENTS.md`、`docs/task00-env.md`；工具运行与环境验收 |

现有 AGENTS.md 已提供。工具是否安装、版本是否满足需求、starter 是否能启动，均尚未验收。规划资料已在 main 建立，不计作 starter 完成。

## 执行检查表

- [ ] 00.1 用户确认 GPU、驱动、Vulkan SDK 和 C++ 编译器的实际版本，运行 `vkcube` 观察结果。
- [ ] 00.2 核对 CMake、vcpkg、RenderDoc、Nsight Graphics；记录可用版本与暂不可用的工具。
- [ ] 00.3 固定 vcpkg baseline，记录 SDL3 稳定版本及其余依赖实际版本；使用 `sdl3` 与 `SDL3::SDL3`，检查没有混入 SDL2 或重复接入教程第三方依赖。
- [ ] 00.4 用户发出具体 `/gen` 请求，agent 准备构建清单、编译脚本与最小 SDL3 窗口样板，附用途说明；样板仅含 SDL 初始化、窗口创建、关闭事件与退出。参考外部示例时记录来源和版本。
- [ ] 00.5 从干净的构建目录配置并编译，验证 SDL3 窗口能打开、关闭并正常退出；本项不要求 Vulkan surface、swapchain 或清屏。vkguide 的 SDL2 starter 不作为验收基线。
- [ ] 00.6 agent 逐段解释构建配置，用户复述并逐行注释 agent 生成的代码，再自行记录环境和问题。
- [ ] 00.7 核对 `prompts/`、`docs/`、`docs/oral/` 和 `docs/debug-log.md`；确认协作规则与角色入口能够使用。

## 验收证据

| 检查 | 通过条件 | 证据位置 |
|---|---|---|
| 基础 Vulkan 环境 | `vkcube` 实际出画面 | 用户的 `docs/task00-env.md` |
| SDL3 starter | 窗口能打开与关闭，配置、构建和启动步骤能复现 | 同上，包含依赖版本、vcpkg baseline、构建参数及外部示例来源（如有） |
| 构建理解 | 用户能解释每段配置以及 shader 编译产物的用途 | 用户复述与自行记录 |
| 协作规则 | agent 正确复述分工、禁止事项和当前角色 | 当前会话 |

## 结束条件

- [ ] `vkcube` 已实际出画面，SDL3 starter 已验证窗口打开、关闭与正常退出。
- [ ] `docs/task00-env.md` 由用户完成，说明 SDK / 驱动 / SDL3 版本、vcpkg baseline 及构建问题。
- [ ] 用户确认生成代码已理解并完成注释，按约定分别提交。
- [ ] 更新 README 为 00 完成、01 待开始；不执行 `/exam 00`，不预写考试记录。

[返回任务入口](README.md)。
