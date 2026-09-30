# Task 05 · 口试评分标准

仅考官使用。先遵守 [口试执行规则](README.md)。共 5 题，5/5 通过。

## 1. Shadow map 依赖

题目：shadow pass 和主 pass 之间的 barrier，src 和 dst stage 各是什么？为什么？

通过标准：
- 结合实际绘制明确深度写入的生产阶段和真实采样的消费阶段，能够解释对应访问及 layout。
- 不把深度写入简单当作 fragment shader 写入；若实现采用不同消费阶段或队列，能依据真实操作解释差异。

未通过示例：只背一个 stage 名称，不知道它对应哪个操作；只转换 layout 就认为数据依赖一定成立。

待补阅读：[Synchronization Examples：Depth attachment write to sampling](https://docs.vulkan.org/guide/latest/synchronization_examples.html)。

## 2. 结果读回

题目：timestamp 查询结果为什么通常延迟读回？隔了一帧是否保证可用，同步等待读回会怎样？

通过标准：
- 区分 CPU 帧推进与 GPU 完成，说明如何依据结果可用性或对应完成条件读取；解释等待对 CPU / GPU 重叠的影响。
- 对照自己计时器的槽位复用、reset 和帧归属约定，正确说明 `timestampPeriod` 的单位与毫秒换算，知道需要检查队列支持与有效位。

未通过示例：认为下一帧必然完成；将尚未可用的结果直接记为零耗时，或只按 CPU 调用间隔计时。

待补阅读：[Vulkan Spec：Queries](https://docs.vulkan.org/spec/latest/chapters/queries.html)；[Vulkan Samples：Timestamp Queries](https://docs.vulkan.org/samples/latest/samples/api/timestamp_queries/README.html)。

## 3. GPU 计时的解释

题目：在 render pass 内部打 timestamp 在 IMR 和 TBR 上语义有何差别？

通过标准：
- 说明 timestamp 与所选 stage、GPU 流水线重叠及实现支持有关，两个时间点不必代表隔离的 shader 执行时间。
- 说明 tile 执行可能使 pass 内的差值难以代表全部 tile 的总成本，区分 API 保证与硬件观察，提出合理的 pass 边界或外部 profiler 对照。

未通过示例：认为任意两个 timestamp 的差就是这段代码的精确总 GPU 成本；把某厂商的行为说成所有 TBR 的固定规则。

待补阅读：[Vulkan Samples：Timestamp Queries](https://docs.vulkan.org/samples/latest/samples/api/timestamp_queries/README.html)；[Khronos：Elapsed Timer Query 的问题说明](https://docs.vulkan.org/features/latest/features/proposals/VK_QCOM_elapsed_timer_query.html)。后者用于理解 TBR 局限，不要求给 Vulkan 1.3 主线增加该扩展。

## 4. 可并行的 pass

题目：如果有三个 pass 且其中两个无依赖，你的依赖图能表达“可并行”吗？怎么表达？

通过标准：
- 使用自己的图解释有向依赖和允许的调度顺序，检查隐藏的资源冲突或别名关系。
- 区分逻辑上没有依赖与实际 GPU 并行执行；说明队列能力、调度和硬件资源仍会约束重叠。

未通过示例：认为图上画两条并列线就保证执行时间减半。

待补阅读：用户自己的 pass 依赖图；[Vulkan Spec：Synchronization](https://docs.vulkan.org/spec/latest/chapters/synchronization.html)。

## 5. 增加后处理

题目：加一个后处理 pass，依赖图和 barrier 会怎么变？

通过标准：
- 根据自己选定的后处理输入 / 输出重新识别生产者、消费者、访问方式和呈现路径，说明新增边与变化的资源使用。
- 能检查已有转换是否仍成立，解释图如何帮助发现遗漏或多余依赖；不要求唯一设计或现场实现。

未通过示例：只说“多加一个 barrier”，没有资源和依赖依据。

待补阅读：用户自己的 pass 图；[Synchronization Examples](https://docs.vulkan.org/guide/latest/synchronization_examples.html)。
