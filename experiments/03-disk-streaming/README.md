# 03 · DiskInfer CPU 磁盘流式推理

验证 DiskInfer 的 CPU 按需读取机制能否在 macOS arm64 上运行，以及 arena 预算如何影响内存、读量和输出一致性。先选择版本明确、两条对照路径均支持的 1B–3B GGUF，使用短上下文和固定生成长度。

1. 固定 DiskInfer 源码版本，确认模型元数据加载、张量偏移、`pread`、arena、淘汰策略和计算图回调的实际执行路径。若研究绕过文件缓存，单独验证 macOS 文件读行为。
2. 优先用相同版本和相同 GGUF 对比常规 CPU 加载和流式 CPU 加载。若只能使用不同构建，记录版本及差异来源。
3. 按模型单次所需权重空间选择几个 arena 预算。记录实际 arena 峰值、进程峰值 RSS、系统内存压力、按需读取字节数和耗时，以及 prefill／decode 的输出和耗时。
4. 使用相同模板和贪心采样比较生成 token ID。保存完整结果或首个分歧。应用层读取量和 SSD 设备读取量各自记录。

验收需要 macOS 实际运行日志、arena 预算约束与流式读的观测证据，以及小模型的正确性对照。完成这些测量后再尝试更大的 GGUF。

[DiskInfer 上游](https://github.com/Horn-Studio/diskinfer-android) · [基准测量口径](../../benchmarks/README.md)
