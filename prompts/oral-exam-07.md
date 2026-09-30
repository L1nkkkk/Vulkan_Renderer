# Task 07 · 口试评分标准

仅考官使用。先遵守 [口试执行规则](README.md)。共 5 题，5/5 通过。设计题没有指定的封装高度；按契约是否完整、自洽、可验证判断，不按是否接近考官偏好评分。

## 1. Barrier 责任

题目：你的 RHI 里 barrier 是谁的责任？如果换成另一种选择，上层代码会怎么变？

通过标准：
- 依据自己的头文件与最小实现说明状态信息由谁提供、依赖由谁决定、错误由谁负责发现。
- 比较另一种分工如何改变上层契约与后端所需信息，能解释成本与适用场景。

未通过示例：只给“薄 / 厚封装”标签；接口没有表达所声称的责任边界。

待补阅读：用户自己的 `rhi/` 接口与设计文档；[NVRHI State Tracking and Barriers](https://github.com/NVIDIA-RTX/NVRHI/blob/main/doc/ProgrammingGuide.md#state-tracking-and-barriers)。

## 2. Bindless

题目：bindless 下你的 binding 抽象还成立吗？

通过标准：
- 从自己的抽象说明资源寻址、索引 / 句柄、更新和生命周期在 bindless 下是否可表达。
- 指出至少一个能力限制或需要改变的契约，说明旧路径与新路径的关系；允许有依据地选择当前不支持。

未通过示例：认为只扩大 descriptor 数量就自动完成全部设计；忽略正在使用的资源与索引生命周期。

待补阅读：[Vulkan Guide：Descriptor Indexing](https://docs.vulkan.org/guide/latest/extensions/VK_EXT_descriptor_indexing.html)；用户自己的 binding 接口。

## 3. 资源别名

题目：你的接口如何表达“这个资源这一帧结束后可以被别名复用”？

通过标准：
- 区分逻辑资源、内存和 GPU 最后一次使用，说明接口能表达什么完成证据与生命周期信息。
- 认识到 CPU 帧结束不是 GPU 完成，解释内存兼容和跨队列使用等需要满足的约束；可明确承认当前接口缺口并说明判断依据。

未通过示例：认为 CPU 离开作用域或帧数自增就足以证明内存可复用。

待补阅读：[Vulkan Spec：Resource Creation / Memory Aliasing](https://docs.vulkan.org/spec/latest/chapters/resources.html)；用户自己的生命周期图。

## 4. 接口表达与实现偏差

题目：agent 实现你接口时，哪一处它理解错了？是接口表达不清还是实现问题？

通过标准：
- 基于真实 review 记录指出契约与实现的差异，能区分规格缺口和未遵守已有约定。
- 说明用户如何修订或验证；若实际没有误解，允许用真实争议或最需要澄清的一处契约作分析，不能制造历史。

未通过示例：只归咎于 agent，无法找到接口约定或行为证据。

待补阅读：用户自己的接口迭代、实现 diff 和评审记录。

## 5. 范围取舍

题目：如果给你三个月把这个 RHI 做完整，你会先砍掉哪些接口？

通过标准：
- 根据明确的目标场景和资源约束选择保留 / 延后的功能，并说明相互依赖和最小可验证闭环。
- 区分暂不实现、保留扩展点与永久不支持，能解释对上层的影响，而非堆砌功能名。

未通过示例：没有优先级理由；删去已经依赖的能力却无法说明如何维持目标行为。

待补阅读：用户自己的设计文档；[NVRHI](https://github.com/NVIDIA-RTX/NVRHI)、[Godot RenderingDevice](https://github.com/godotengine/godot/blob/master/servers/rendering/rendering_device.h)、[bgfx](https://github.com/bkaradzic/bgfx) 中与所选范围对应的接口。
