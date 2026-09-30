# Vulkan Renderer Tasks · Agent 协作版

一套以任务驱动、与 AI agent 结对完成的 Vulkan 1.3 学习仓库。全部完成后会得到一个能加载 glTF、带 shadow map、带 per-pass GPU 计时 overlay 的小渲染器，以及一份由你本人写出、能证明你理解了渲染架构的设计文档。

面向对象：有图形学或引擎使用经验（如 Unity / UE），未接触过 Vulkan，目标是引擎渲染方向，并且日常有 agent（Claude Code、Cursor、Codex 等）可用。

当前阶段：**Task 00 待开始**。`main` 已建立 Task 00～07 的规划基线；规划齐备不代表任务完成，工具链与运行效果仍需实际验收。

从 [Task 00 执行检查表](tasks/task00.md) 开始。完整执行约定见 [任务入口](tasks/README.md)，协作边界见 [AGENTS.md](AGENTS.md)。本文件由原 `README-agent.md` 整理而来，作为唯一的课程总览。

---

## 目录

- [设计理念](#设计理念)
- [Agent 的五种角色](#agent-的五种角色)
- [协作规则](#协作规则)
- [技术栈](#技术栈)
- [任务总览](#任务总览)
- [规划资料与执行方式](#规划资料与执行方式)
- [Task 00 环境与协作约定](#task-00-环境与协作约定)
- [Task 01 帧循环与同步](#task-01-帧循环与同步)
- [Task 02 管线与网格](#task-02-管线与网格)
- [Task 03 描述符、纹理与 barrier](#task-03-描述符纹理与-barrier)
- [Task 04 材质系统与资源生命周期](#task-04-材质系统与资源生命周期)
- [Task 05 多 pass 与 GPU 计时](#task-05-多-pass-与-gpu-计时)
- [Task 06 帧分析与 RHI 对照](#task-06-帧分析与-rhi-对照)
- [Task 07 接口先行：设计你的 RHI](#task-07-接口先行设计你的-rhi)
- [附录 A 仓库结构](#附录-a-仓库结构)
- [附录 B RenderDoc 速查](#附录-b-renderdoc-速查)
- [附录 C 参考资料](#附录-c-参考资料)
- [后续方向](#后续方向)

---

## 设计理念

有 agent 之后，写出能跑的 Vulkan 代码几乎没有门槛。如果课程仍以"代码能跑"为交付物，学习者很容易变成只会按回车的人。所以本仓库把学习单元从"实现"换成"决策、验证、解释"，agent 的定位是结对伙伴和考官，不是代码生成器。

三条原则：

1. 样板交给 agent，概念自己写。Vulkan 的知识密度集中在同步模型、资源状态转换、描述符体系、PSO 边界、资源生命周期。其余是样板。每个 Task 明确划分哪些代码允许 agent 生成、哪些必须独立完成。
2. 验收用 agent 无法代替的方式。代码能跑不是证据。每个 Task 的验收是三件事：口试通过、能独立找出注入的 bug、能写出一份 agent 没参与起草的设计说明。
3. 接口先行。从 Task 02 开始，先由你写头文件定接口，再让 agent 填实现，最后你 review 实现是否符合意图。课程重心自然从 API 用法滑向抽象设计，这正是引擎架构岗的工作方式。

### 关于 RHI

RHI（Render Hardware Interface）是引擎内部定义的图形 API 抽象层：上层渲染代码只调这层接口，下面由 Vulkan / D3D12 / Metal / GLES 各自实现。UE 里直接叫 RHI，Unity 对应 GfxDevice。

| 风格 | 特点 | 代表 |
|---|---|---|
| 薄封装 | 接口贴近底层图形 API，较多控制权交给调用者；也可能提供可选的自动管理 | NVRHI、UE5 RHI |
| 中等封装 | 隐藏 barrier 与资源状态转换，由 RHI 或 render graph 自动推导 | Godot 4 RenderingDevice、WebGPU |
| 厚封装 | 贴近 OpenGL 心智模型，内部攒 PSO 与 barrier | bgfx、Filament backend、Unity GfxDevice |

这里的分类仅用于讨论抽象高度，不能代替逐项核对。特别是 NVRHI 支持可选的资源状态跟踪与自动 barrier；Task 06 应以实际阅读的版本为准。[NVRHI Programming Guide](https://github.com/NVIDIA-RTX/NVRHI/blob/main/doc/ProgrammingGuide.md#state-tracking-and-barriers)

Task 07 会让你设计自己的一版，并对照上面三种风格评审。

---

## Agent 的五种角色

每个 Task 会指定本任务启用哪几种角色。

| 角色 | 做什么 | 不做什么 |
|---|---|---|
| 生成者 Generator | 写样板、辅助类、CMake、shader 编译脚本 | 不碰"必须独立完成"栏里的代码 |
| 导师 Mentor | 卡住时提问引导；解释 validation 报错对应的规范条款 | 不直接给答案、不给代码 |
| 审查者 Reviewer | 对你写的核心代码做 code review，指出问题与风险 | 不改代码，由你自己修 |
| 出题者 Saboteur | 在你的代码里注入一个 bug（漏 barrier、错 layout、少 wait），你用 RenderDoc 和 validation 找出来 | 注入后不给提示，直到你放弃或找到 |
| 考官 Examiner | 按 `prompts/oral-exam-XX.md` 题库口试，追问 why，给出通过或不通过 | 不放水，答不出就标记为未通过 |

---

## 协作规则

以下规则写入 `AGENTS.md`，任何 agent 打开仓库都会自动遵守。

- 每个 Task 的"必须独立完成"栏内的文件，agent 禁止直接生成代码，只能提问或 review。
- 你要求修 validation 报错时，agent 先要求你用一句话复述该报错的含义，再继续。
- agent 生成的任何代码，commit 前你必须逐行加自己的注释。注释不了的就是没懂，不许提交。
- 禁止对 agent 说"帮我让它能跑"这类目标模糊的指令。指令必须包含你期望的行为与你的当前判断。
- `docs/` 下所有设计文档由你起草，agent 只允许事后校对事实性错误，并在文档末尾追加一行校对记录。
- 每次卡住超过 30 分钟必须写入 `docs/debug-log.md`：现象、排查路径、根因、是谁解决的。每完成两个 Task 回顾一次，agent 解决的比例过高说明分工设定需要调整。
- 每个 Task 结束时由 agent 执行口试，口试记录存入 `docs/oral/taskXX.md`。

---

## 技术栈

| 类别 | 选择 | 说明 |
|---|---|---|
| 语言 / 构建 | C++20、CMake、vcpkg | |
| 教程主线 | vkguide.dev 2.0 | Task 01～05 对应其 Chapter 0～5 |
| 基建库 | volk、vk-bootstrap、VMA、SDL3 或 GLFW、glm、fastgltf、Dear ImGui | |
| Shader | GLSL + glslc，编译加 `-g` | |
| 调试 | Validation Layer 全程开启、RenderDoc、Nsight Graphics | |
| 参考硬件 | Windows + NVIDIA RTX | 工具链最完整 |
| Agent | 任意支持读取 `AGENTS.md` 的编码 agent | |

---

## 任务总览

| Task | 主题 | Agent 参与度 | 核心交付物 | 状态 |
|---|---|---|---|---|
| [00](tasks/task00.md) | 环境与协作约定 | 高 | 可运行 starter + `AGENTS.md` | [ ] 待开始 |
| [01](tasks/task01.md) | 帧循环与同步 | 低 | 清屏 + 同步流程图 + 口试 | [ ] 等待 00 |
| [02](tasks/task02.md) | 管线与网格 | 中（接口先行） | 带深度网格 + PSO 状态清单 | [ ] 等待 01 |
| [03](tasks/task03.md) | 描述符、纹理与 barrier | 低 | glTF 场景 + barrier 说明 + 找 bug | [ ] 等待 02 |
| [04](tasks/task04.md) | 材质系统与资源生命周期 | 高（练 review） | 多材质场景 + 生命周期图 | [ ] 等待 03 |
| [05](tasks/task05.md) | 多 pass 与 GPU 计时 | 中 | 阴影 + 耗时 overlay + pass 依赖图 | [ ] 等待 04 |
| [06](tasks/task06.md) | 帧分析与 RHI 对照 | 中 | 帧分析报告 + NVRHI 对照表 | [ ] 等待 05 |
| [07](tasks/task07.md) | 接口先行：设计你的 RHI | 中（你设计、agent 实现） | RHI 接口草案 + 设计文档 | [ ] 等待 06 |

Task 00～04 为必修主线，Task 05～07 为进阶。前三个概念 Task 的 agent 参与度刻意压低，Task 04 反过来让 agent 大量生成以练 review，Task 05～07 转为"你出设计、agent 出实现"。

每个 Task 的段落结构固定为：目标、知识点、Agent 分工、要求、交付物、口试题、验收、参考。

## 规划资料与执行方式

| 位置 | 用途 | 维护边界 |
|---|---|---|
| `README.md` | 课程目标、分工表、交付物与总进度 | 修改范围需与当前任务一致 |
| [tasks/](tasks/README.md) | Task 00～07 的执行顺序、检查表与验收证据 | 本次授权 agent 准备的规划资料，不包含作业答案 |
| [prompts/](prompts/README.md) | review 检查表、口试评分标准、故障注入清单 | agent 在对应角色下读取；考核前学习者避免阅读评分与注入细节 |
| `docs/` | 用户的环境记录、设计说明、review 报告、排错记录 | 用户起草；口试记录由考官按规则写入 |

执行时先确认 Task 编号，再按该任务的三栏分工表协作。每次只推进一个 Task；先完成前置任务的验收，再开始后续实现。Task 00 无口试，其余 Task 口试全部通过才允许结束。Task 01、03 还需完成独立故障排查。

`main` 保存共同规划和验收后的进度。开始某个 Task 时再创建对应工作分支，建议使用 `task/00-env`、`task/01-sync` 等名称，具体见检查表；不提前创建八个实现分支。是否合并或打 tag 以实际验收及用户指令为准。

代码分工不因规划完成而扩大。agent 生成代码前仍需遵守角色与接口先行要求，提交前由用户逐行注释；`[self]` 提交始终由用户执行。规划阶段只建立 `docs/debug-log.md` 空文件和目录占位，不预写任何学习文档或口试结果。

---

## Task 00 环境与协作约定

### 目标

搭好工具链，建立与 agent 的协作规则。本任务不涉及 Vulkan 概念。

### Agent 分工

| agent 可直接生成 | agent 只可提问 / review | 必须独立完成 |
|---|---|---|
| CMake、vcpkg 清单、shader 编译脚本 | | `AGENTS.md`、`docs/task00-env.md` |

角色：生成者。

### 要求

- [ ] 更新显卡驱动，安装 Vulkan SDK，运行 `vkcube` 确认出画面
- [ ] 安装 CMake、vcpkg、RenderDoc、Nsight Graphics
- [ ] 拉取 vkguide starting point 并编译通过
- [ ] 让 agent 解释 CMakeLists 每一段的作用，你复述一遍并写进文档
- [ ] 按上文协作规则写出 `AGENTS.md`
- [ ] 建立 `prompts/`、`docs/`、`docs/oral/` 目录与 `docs/debug-log.md` 空文件

### 交付物

- 可运行 starter 工程
- `AGENTS.md`
- `docs/task00-env.md`：SDK 与驱动版本、构建问题及解法

### 口试题

本任务无口试。

### 验收

`vkcube` 与 starter 均能启动；agent 读取 `AGENTS.md` 后能正确复述协作规则。

---

## Task 01 帧循环与同步

### 目标

搭出完整帧循环骨架，画出纯色清屏。理解 Vulkan 初始化链与同步模型。这是概念密度最高的 Task 之一，agent 参与度最低。

### 知识点

- instance → physical device → device → queue → swapchain → command pool / buffer → fence / semaphore
- frames-in-flight 双缓冲
- `vkCmdBeginRendering` / `vkCmdEndRendering`
- swapchain image layout 转换
- Validation 输出的阅读

### Agent 分工

| agent 可直接生成 | agent 只可提问 / review | 必须独立完成 |
|---|---|---|
| vk-bootstrap 初始化、窗口事件循环、swapchain 重建骨架 | 同步对象的编排、layout 转换的 barrier | `FrameData` 结构、每帧 acquire → record → submit → present 的同步逻辑、`docs/task01-sync.md` |

角色：生成者（仅初始化）、导师、出题者、考官。

### 要求

- [ ] 实现 N 帧 in-flight 的 command buffer 与同步对象管理
- [ ] 每帧清屏为随时间变化的颜色
- [ ] Validation 零报错；每条报错先自己复述含义再求助
- [ ] 让 agent 注入一个同步 bug（例如去掉某个 semaphore wait），你用 validation 与 RenderDoc 找出并说明现象
- [ ] RenderDoc 抓一帧，把附录 B 中的面板各点一遍

### 交付物

- 清屏程序
- `docs/task01-sync.md`：同步流程图，说明 fence、acquire semaphore、present wait semaphore 各自保护什么以及安全复用的依据；回答 swapchain image 的 layout 为什么要转换、录制与提交为什么分离
- `docs/oral/task01.md`：口试记录

### 口试题

1. 没有其他等价同步时，去掉 presentation 对 render-finished semaphore 的等待会破坏什么保证？为什么不能只凭画面或 validation 判断正确性？
2. frames-in-flight 设为 1、2、3 分别对延迟和吞吐有什么影响？
3. `vkAcquireNextImageKHR` 返回的 index 和当前 frame index 为什么不是同一个东西？
4. 窗口 resize 时哪些对象要重建、哪些不用？为什么？
5. 把 fence wait 放到 acquire 之后时，需要检查哪些对象的复用条件？为什么不能只靠调用顺序判断安全性？

### 验收

窗口稳定运行、resize 不崩溃、Validation 零报错、注入的 bug 被独立找出、口试通过。

### 参考

- vkguide Chapter 0～1
- Vulkan Tutorial：Swap chain、Rendering and presentation 两节
- TU Wien Vulkan Lecture Series：Synchronization
- [Vulkan Guide：Swapchain Semaphore Reuse](https://docs.vulkan.org/guide/latest/swapchain_semaphore_reuse.html)

---

## Task 02 管线与网格

### 目标

创建图形管线，从硬编码三角形到带深度测试的网格。首次练习"接口先行"。

### 知识点

- shader module、`VkGraphicsPipelineCreateInfo`
- dynamic rendering 的 attachment 格式声明
- push constant、buffer device address
- VMA 创建 buffer
- depth attachment
- dynamic state 与不可变 PSO 的边界

### Agent 分工

| agent 可直接生成 | agent 只可提问 / review | 必须独立完成 |
|---|---|---|
| `PipelineBuilder` 的实现（依据你写的头文件）、VMA buffer 上传辅助函数、GLSL 编译集成 | 你写的 `PipelineBuilder.h` 接口 | `PipelineBuilder.h` 接口设计、depth attachment 配置、`docs/task02-pso.md` |

角色：生成者、审查者、考官。

流程：你先写 `PipelineBuilder.h`，写明每个方法的意图；agent 按头文件实现；你 review 实现是否符合意图并逐行注释。

### 要求

- [ ] 编译 GLSL 到 SPIR-V 并加载
- [ ] 画出硬编码三角形
- [ ] 用 VMA 上传网格，通过 buffer device address 读取顶点
- [ ] 添加 depth attachment，开启深度测试
- [ ] 用 push constant 传 MVP
- [ ] 让 agent review 你的 `PipelineBuilder.h`，记录它指出的问题和你的取舍

### 交付物

- 可旋转的带深度网格
- `docs/task02-pso.md`：列出 PSO 包含的所有状态，标注哪些可设为 dynamic state；回答：如果做 RHI，PSO 的 hash key 应包含哪些字段

### 口试题

1. 为什么 Vulkan 把 PSO 设计成不可变对象？这对上层材质系统意味着什么？
2. 哪些状态适合 dynamic state？把所有能 dynamic 的都 dynamic 有什么代价？
3. buffer device address 相比 vertex input binding 的优劣是什么？
4. 深度 attachment 的 format 选择依据是什么？D24 和 D32 在什么情况下有差别？
5. 你的 `PipelineBuilder` 接口里，哪个方法是你最不确定的？为什么？

### 验收

网格正确、深度遮挡正确、切换 shader 无需重建其他对象、口试通过。

### 参考

- vkguide Chapter 2、Chapter 3 前半
- Vulkan Guide：Pipeline 相关章节

---

## Task 03 描述符、纹理与 barrier

### 目标

理解描述符体系与资源状态转换，加载并绘制带贴图的 glTF 场景。barrier 是本仓库最核心的手写内容。

### 知识点

- descriptor set layout、pool、set 更新
- UBO、image / image view / sampler
- `vkCmdPipelineBarrier2` 与 image layout 转换
- staging buffer 上传
- fastgltf

### Agent 分工

| agent 可直接生成 | agent 只可提问 / review | 必须独立完成 |
|---|---|---|
| fastgltf 加载与数据解包、staging buffer 拷贝流程、`DescriptorAllocator` / `DescriptorWriter` 实现（依据你的头文件） | 每一条 barrier 的 stage / access 选择 | 所有 `vkCmdPipelineBarrier2` 调用及其注释、descriptor set layout 设计、`docs/task03-barrier.md` |

角色：生成者、导师、出题者、考官。

### 要求

- [ ] 用 UBO 替代 push constant 传相机矩阵
- [ ] 写出 `DescriptorAllocator` / `DescriptorWriter` 接口，agent 实现，你 review
- [ ] 加载一张纹理并采样
- [ ] 用 fastgltf 加载含多网格多材质的 glTF 场景
- [ ] 每一处 barrier 写注释说明 src / dst stage 与 access 的理由
- [ ] 让 agent 注入一个 barrier bug（错误 layout 或缺失 access mask），你用 RenderDoc Resource Inspector 找出

### 交付物

- 带贴图的 glTF 场景
- `docs/task03-barrier.md`：列出本任务所有 barrier 及其必要性；思考题：这些 barrier 在 tile-based GPU 上是否会导致 tile flush，哪些可以合并

### 口试题

1. `srcStageMask` 和 `srcAccessMask` 分别约束什么？只写 stage 不写 access 会怎样？
2. 一张纹理从 `UNDEFINED` 到 `TRANSFER_DST` 再到 `SHADER_READ_ONLY`，两次转换分别在什么时机、由谁发起？
3. descriptor pool 耗尽时你的 allocator 怎么处理？单个大 pool 与多个可增长 pool 有什么取舍？
4. 如果一个 image 被两个 pass 读取但没有写入，中间需要 barrier 吗？
5. 在 TBR GPU 上，一次 layout 转换的代价和 IMR 上有什么不同？

### 验收

纹理正确、Validation 零报错、RenderDoc 中每个 image layout 与预期一致、注入 bug 被独立找出、口试通过。

### 参考

- vkguide Chapter 3 后半、Chapter 4
- Vulkan Guide：Synchronization Examples 页面
- Khronos Vulkan-Samples：`samples/performance`

---

## Task 04 材质系统与资源生命周期

### 目标

整理前三个 Task 的代码，形成材质系统与绘制流程，接入 ImGui。本任务反过来让 agent 大量生成代码，你专门练 review 和生命周期分析。

### 知识点

- 材质 = PSO + descriptor set 的组合
- draw context、按材质排序
- deletion queue 与销毁时机
- ImGui Vulkan 后端

### Agent 分工

| agent 可直接生成 | agent 只可提问 / review | 必须独立完成 |
|---|---|---|
| `Material` / `MaterialInstance` / `DrawContext` 实现、ImGui 接入、deletion queue 实现 | | 所有 agent 生成代码的逐行注释、对 agent 代码的 review 报告、`docs/task04-lifetime.md` |

角色：生成者、考官。你担任审查者。

流程：给 agent 明确的接口与行为约束，让它一次性生成材质系统；你逐行注释并写 review 报告，找出至少三个可改进点并自己修。

### 要求

- [ ] 实现 Material / MaterialInstance，DrawContext 按材质排序提交
- [ ] 接入 ImGui，显示帧率与统计
- [ ] 所有资源销毁走 deletion queue
- [ ] 写 review 报告，列出 agent 代码中的问题（命名、生命周期、错误处理、性能）并修复

### 交付物

- 多物体多材质场景 + ImGui overlay
- `docs/task04-review.md`：对 agent 生成代码的 review 报告
- `docs/task04-lifetime.md`：为 buffer、image、descriptor set、pipeline、command buffer、同步对象各画一张生命周期图（谁创建、谁持有、何时销毁、销毁为什么要等 fence）。这份图是 Task 07 的设计草稿

### 口试题

1. deletion queue 为什么要按帧延迟销毁？延迟几帧才安全？
2. 一个 descriptor set 引用的 image 被销毁了但 set 还在，会发生什么？
3. 材质排序的收益来自哪里？在 Vulkan 下比在 OpenGL 下收益大还是小？
4. agent 生成的代码里你最不满意的一处是什么？如果是你写会怎么做？
5. ImGui 的 Vulkan 后端自己管理了哪些资源？它和你的 deletion queue 如何共处？

### 验收

切换材质不重建资源、退出时无泄漏报告、review 报告至少三处修复、口试通过。

### 参考

- vkguide Chapter 5
- Dear ImGui：`examples/example_sdl3_vulkan`

---

## Task 05 多 pass 与 GPU 计时

### 目标

引入第二个 render pass 与跨 pass 资源依赖，实现 per-pass GPU 时间戳。开始"你出设计、agent 出实现"的模式。

### 知识点

- offscreen depth-only pass
- depth attachment 从写到读的 barrier
- shadow map 采样与 bias
- `VkQueryPool`、`vkCmdWriteTimestamp2`、`timestampPeriod`
- 查询结果延迟读回

### Agent 分工

| agent 可直接生成 | agent 只可提问 / review | 必须独立完成 |
|---|---|---|
| shadow shader、`GpuTimer` 实现（依据你的头文件）、ImGui 曲线绘制 | 你的 pass 依赖图与 `GpuTimer.h` 接口 | pass 依赖图、shadow map 的 barrier、`GpuTimer.h` 接口、`docs/task05-passes.md` |

角色：生成者、审查者、考官。

流程：先画 pass 依赖图（资源、读写、barrier 位置），再写 `GpuTimer.h`，agent 实现，你 review。

### 要求

- [ ] 实现 shadow pass，主 pass 采样产生阴影
- [ ] 手写 shadow map 从 depth write 到 shader read 的 barrier
- [ ] 每个 pass 前后写 timestamp；在后续帧确认结果可用后读回换算毫秒，不把“隔了一帧”视为完成证明
- [ ] ImGui 显示每个 pass 的耗时曲线
- [ ] 用 Nsight Graphics 校验 timestamp 量级

### 交付物

- 带阴影与 GPU 耗时 overlay 的场景
- `docs/task05-passes.md`：pass 依赖图；说明为什么这张图是 render graph 要自动化的对象；记录 timestamp 在 IMR GPU 上的表现，并思考在 TBR 上会有什么不同

### 口试题

1. shadow pass 和主 pass 之间的 barrier，src 和 dst stage 各是什么？为什么？
2. timestamp 查询结果为什么通常延迟读回？隔了一帧是否保证可用，同步等待读回会怎样？
3. 在 render pass 内部打 timestamp 在 IMR 和 TBR 上语义有何差别？
4. 如果有三个 pass 且其中两个无依赖，你的依赖图能表达"可并行"吗？怎么表达？
5. 加一个后处理 pass，依赖图和 barrier 会怎么变？

### 验收

阴影正确、timestamp 与 Nsight 量级一致、口试通过。

### 参考

- Sascha Willems：`shadowmapping`、`timestampquery`
- Vulkan Spec：Queries
- [Vulkan Spec：查询结果可用性与时间戳](https://docs.vulkan.org/spec/latest/chapters/queries.html)

---

## Task 06 帧分析与 RHI 对照

### 目标

用工具审视自己的帧，对照成熟 RHI 理解抽象高度的取舍。不写新功能。

### Agent 分工

| agent 可直接生成 | agent 只可提问 / review | 必须独立完成 |
|---|---|---|
| | 你的分析报告与对照表（事实校对） | RenderDoc 分析、Nsight 对照、`docs/task06-frame-analysis.md`、`docs/task06-rhi-compare.md` |

角色：导师（解释 NVRHI 中你不理解的设计）、考官。

### 要求

- [ ] RenderDoc 抓帧，逐 draw 检查资源状态，找出多余或缺失的 barrier 并修复
- [ ] Nsight Graphics 查看 GPU timeline，与自己的 timestamp 对照
- [ ] 通读 NVRHI 的 `include/nvrhi/nvrhi.h`，不理解的地方向 agent 提问，记录问答
- [ ] 阅读《Writing an efficient Vulkan renderer》

### 交付物

- `docs/task06-frame-analysis.md`：发现的问题与修复
- `docs/task06-rhi-compare.md`：对照表，至少覆盖 barrier、descriptor、PSO、command list、资源生命周期五项，说明 NVRHI 分别抽象到什么高度、你代码中哪些东西在它那里被隐藏了

### 口试题

1. NVRHI 的自动状态跟踪和 barrier 处理覆盖哪些范围？用户还需要提供哪些边界信息，什么时候会选择手动管理？
2. 你找到的多余 barrier 为什么 validation 不报？它的实际代价是什么？
3. NVRHI 的 `BindingLayout` / `BindingSet` 和你的 descriptor 抽象相比，多解决了什么问题？
4. 如果要支持 D3D12 后端，你现有代码里哪一处最难映射？

### 验收

两份文档完成、barrier 问题已修复、口试通过。

### 参考

- NVRHI、Godot `rendering_device.h`
- [NVRHI Programming Guide：State Tracking and Barriers](https://github.com/NVIDIA-RTX/NVRHI/blob/main/doc/ProgrammingGuide.md#state-tracking-and-barriers)
- Writing an efficient Vulkan renderer

---

## Task 07 接口先行：设计你的 RHI

### 目标

课程终点。你设计一版 RHI 接口草案，agent 用 Vulkan 实现其中一个子集，你评审实现并对照三种封装风格修订自己的设计。

### Agent 分工

| agent 可直接生成 | agent 只可提问 / review | 必须独立完成 |
|---|---|---|
| RHI 接口子集的 Vulkan 后端实现 | 你的接口草案、设计文档（事实校对） | `rhi/*.h` 接口草案、`docs/task07-design.md`、对 agent 实现的评审 |

角色：生成者、审查者（review 你的接口）、考官。

### 要求

- [ ] 写出 `rhi/` 下的接口头文件，至少覆盖 device、command list、buffer、texture、pipeline、binding、同步
- [ ] 选择一种封装高度并说明理由
- [ ] 让 agent 用 Vulkan 实现其中"清屏 + 一个 draw"所需的最小子集
- [ ] 评审实现，记录接口设计中暴露出的问题并修订
- [ ] 写设计文档

### 交付物

- `rhi/*.h` 接口草案与最小 Vulkan 实现
- `docs/task07-design.md`，3000 字以上，至少回答：
  - 选择哪种封装高度，为什么
  - 若在其上做 render graph，哪些决策应自动化，barrier 在编译期还是运行期决定
  - 资源别名如何做
  - TBR 与 IMR 下同一套 pass 编排有何差别
  - 与 NVRHI、Godot RenderingDevice、bgfx 各一处关键差异及理由

### 口试题

1. 你的 RHI 里 barrier 是谁的责任？如果换成另一种选择，上层代码会怎么变？
2. bindless 下你的 binding 抽象还成立吗？
3. 你的接口如何表达"这个资源这一帧结束后可以被别名复用"？
4. agent 实现你接口时，哪一处它理解错了？是接口表达不清还是实现问题？
5. 如果给你三个月把这个 RHI 做完整，你会先砍掉哪些接口？

### 验收

接口草案 + 最小实现可运行、设计文档完成、口试通过。

### 参考

- NVRHI、Godot RenderingDevice、bgfx
- Writing an efficient Vulkan renderer

---

## 附录 A 仓库结构

以下是目标结构。规划阶段已存在 `README.md`、`AGENTS.md`、`tasks/`、`prompts/` 与空的记录目录；构建文件、源码、资源和学习文档在对应 Task 中产生。

```
.
├── README.md
├── AGENTS.md                    # 写给 agent 的协作规则
├── tasks/
│   ├── README.md                 # 执行入口与统一完成条件
│   └── task00.md ... task07.md   # 逐任务检查表，不包含作业答案
├── CMakeLists.txt
├── vcpkg.json
├── prompts/
│   ├── README.md                 # 角色资料索引与使用规则
│   ├── oral-exam-01.md ... 07.md   # 口试题库与评分标准
│   ├── saboteur-01.md / 03.md      # bug 注入清单（agent 可读，你别看）
│   └── review-checklist.md         # code review 检查表
├── src/
│   ├── core/                    # 设备、swapchain、同步、deletion queue
│   ├── render/                  # pipeline builder、descriptor、material、draw context、gpu timer
│   ├── scene/                   # glTF 加载、相机
│   └── main.cpp
├── rhi/                         # Task 07 接口草案与最小实现
├── shaders/
├── assets/
└── docs/
    ├── debug-log.md             # 卡住 30 分钟以上的问题记录
    ├── oral/task01.md ... 07.md # 口试记录
    ├── task00-env.md
    ├── task01-sync.md
    ├── task02-pso.md
    ├── task03-barrier.md
    ├── task04-review.md
    ├── task04-lifetime.md
    ├── task05-passes.md
    ├── task06-frame-analysis.md
    ├── task06-rhi-compare.md
    └── task07-design.md
```

提交约定：每个 Task 完成后打 tag（`task-01` ...）。agent 生成的 commit 与你手写的 commit 分开提交，commit message 前缀分别用 `[agent]` 与 `[self]`，便于回看分工比例。

---

## 附录 B RenderDoc 速查

抓帧：Launch Application 填 exe 与工作目录 → Launch → 程序内按 F12 → 双击捕获进入分析。

| 面板 | 用途 |
|---|---|
| Event Browser | 按时间顺序列出这一帧所有命令，点任一条切到该时刻状态 |
| Texture Viewer | 查看 color / depth 附件；右键像素 → Pixel History 看哪些 draw 写过它 |
| Pipeline State | 当前 draw 绑定的 buffer、descriptor、shader、viewport、depth 设置 |
| Mesh Viewer | 顶点经 vertex shader 前后位置，判断矩阵是否传错 |
| Resource Inspector | 每个 image 在当前时刻的 layout，检查 barrier |

Shader 调试需 glslc 加 `-g`。排查顺序：Validation 输出 → RenderDoc → 最后才打断点。

---

## 附录 C 参考资料

### 主线教程

- [vkguide.dev](https://vkguide.dev) —— Task 01～05 对应 Chapter 0～5
- [Vulkan Tutorial](https://vulkan-tutorial.com) —— 概念补充

### 视频

- [TU Wien Vulkan Lecture Series](https://www.youtube.com/playlist?list=PLmIqTlJ6KsE1Jx5HV4sd2jOe3V1KMHHgn)

### 样例仓库

- [Sascha Willems Vulkan examples](https://github.com/SaschaWillems/Vulkan)
- [Khronos Vulkan-Samples](https://github.com/KhronosGroup/Vulkan-Samples) —— `samples/performance` 专讲 TBR

### 规范与指南

- [Vulkan Specification](https://docs.vulkan.org/spec/latest/index.html)
- [Vulkan Guide (Khronos)](https://docs.vulkan.org/guide/latest/index.html) —— Synchronization Examples 必读

### 架构文章

- [Writing an efficient Vulkan renderer](https://zeux.io/2020/02/27/writing-an-efficient-vulkan-renderer/)

### RHI 参考实现

- [NVRHI](https://github.com/NVIDIA-RTX/NVRHI) —— 薄封装
- [Godot RenderingDevice](https://github.com/godotengine/godot/blob/master/servers/rendering/rendering_device.h) —— 中等封装
- [bgfx](https://github.com/bkaradzic/bgfx) —— 厚封装

### 基建库

- [volk](https://github.com/zeux/volk) ｜ [vk-bootstrap](https://github.com/charles-lunarg/vk-bootstrap) ｜ [VMA](https://github.com/GPUOpen-LibrariesAndSDKs/VulkanMemoryAllocator) ｜ [fastgltf](https://github.com/spnda/fastgltf) ｜ [Dear ImGui](https://github.com/ocornut/imgui) ｜ [glm](https://github.com/g-truc/glm)

### 工具

- [Vulkan SDK](https://vulkan.lunarg.com/) ｜ [RenderDoc](https://renderdoc.org)（[Quick Start](https://renderdoc.org/docs/getting_started/quick_start.html)）｜ [Nsight Graphics](https://developer.nvidia.com/nsight-graphics)

---

## 后续方向

完成全部 Task 后，Task 07 的接口草案就是下一步小引擎的起点。建议目标定为"带 RHI 抽象层和 render graph 的小渲染器"，而非"Vulkan 引擎"：

- 以 Task 07 的 RHI 为底，Vulkan 作第一个后端，思考 D3D12 / Metal 如何映射
- 上层做简化版 render graph：声明 pass、声明读写资源、自动推导 barrier 与 renderpass 合并
- 功能范围克制：shadow map、一条 forward 或 deferred 管线、PBR、后处理链、简单 ECS 或 scene graph
- 把 per-pass GPU timing 与 profiler overlay 做成内置能力

继续沿用本仓库的协作模式：你写接口和设计文档，agent 填实现，你 review，每个里程碑口试一次。

替代路线：深读一个中等规模开源引擎（Godot RenderingDevice 层、bgfx、Filament）并贡献一个有分量的渲染特性。
