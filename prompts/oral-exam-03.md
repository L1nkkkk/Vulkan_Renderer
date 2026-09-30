# Task 03 · 口试评分标准

仅考官使用。先遵守 [口试执行规则](README.md)。共 5 题，5/5 通过。

## 1. Stage 与 access

题目：`srcStageMask` 和 `srcAccessMask` 分别约束什么？只写 stage 不写 access 会怎样？

通过标准：
- 区分执行作用域与内存访问作用域，能针对自己的生产者 / 消费者说明依赖。
- 说明只有执行依赖不等于完成写入可见性；同时知道某些依赖只需执行顺序，不能把 access 为零一概判错。

未通过示例：把 stage 当作资源类型；认为 stage 足够大就自动解决所有内存可见性。

待补阅读：[Vulkan Spec：Synchronization and Cache Control](https://docs.vulkan.org/spec/latest/chapters/synchronization.html)；[Synchronization Examples](https://docs.vulkan.org/guide/latest/synchronization_examples.html)。

## 2. 上传到采样

题目：一张纹理从 `UNDEFINED` 到 `TRANSFER_DST` 再到 `SHADER_READ_ONLY`，两次转换分别在什么时机、由谁发起？

通过标准：
- 结合自己的上传路径指出旧内容是否需要保留、哪个操作首次写入、哪个操作随后读取，以及谁负责录制相应命令。
- 区分 CPU 录制、GPU 执行和跨队列情形；说明提交关系、layout 与内存依赖需要一起成立。

未通过示例：认为 descriptor 更新自动完成 image layout 转换；把 CPU 调用结束当作上传执行结束。

待补阅读：[Synchronization Examples：Transfer Dependencies](https://docs.vulkan.org/guide/latest/synchronization_examples.html)；用户的实际上传路径。

## 3. Descriptor pool

题目：descriptor pool 耗尽时你的 allocator 怎么处理？单个大 pool 与多个可增长 pool 有什么取舍？

通过标准：
- 说明自己的容量统计、失败处理及回收契约，能区分 set 数量、descriptor 类型配额等限制。
- 比较容量估计、内存、生命周期与增长管理成本，知道 reset / 回收必须满足使用完成约束。

未通过示例：认为只增大 `maxSets` 就解决所有耗尽；认为“大 pool 永远错”或可无视 GPU 使用直接 reset。

待补阅读：[Vulkan Spec：Descriptor Sets / Descriptor Pools](https://docs.vulkan.org/spec/latest/chapters/descriptorsets.html)。

## 4. 两个读取 pass

题目：如果一个 image 被两个 pass 读取但没有写入，中间需要 barrier 吗？

通过标准：
- 在 layout、queue family ownership 等条件兼容时，能够解释只读之间不存在读写冲突。
- 主动检查此前写入的可见性和其他状态要求；不能把“没有新写入”扩展成永远不需要同步或状态转换。

未通过示例：无条件回答“需要”或“不需要”，无法陈述前提。

待补阅读：[Vulkan Spec：Synchronization and Cache Control](https://docs.vulkan.org/spec/latest/chapters/synchronization.html)。

## 5. TBR 与 IMR

题目：在 TBR GPU 上，一次 layout 转换的代价和 IMR 上有什么不同？

通过标准：
- 说明 tile memory、附件保存 / 加载、访问范围及 render pass 边界可能影响带宽和执行重叠。
- 明确一次 layout 转换不等于必然发生一次 tile flush；结论受具体硬件、转换和使用方式影响，能提出验证依据。

未通过示例：断言所有转换都必须写回显存，或认为 IMR 的 barrier 没有代价。

待补阅读：[Vulkan Guide：TBR Best Practices](https://docs.vulkan.org/guide/latest/tile_based_rendering_best_practices.html)；[Vulkan Samples：Pipeline Barriers](https://docs.vulkan.org/samples/latest/samples/performance/pipeline_barriers/README.html)。
