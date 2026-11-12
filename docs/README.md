# 技术文档

- `architecture/` 记录读到的运行时实现、数据流和资源生命周期，附源码版本和路径。
- `research-notes/` 保存资料摘记、研究假设和实验观察，标明资料来源和验证状态。
- `decisions/` 记录会影响实验复现的选择、取舍和依据。

Windows 参与者可从 [Mellum2.1 CPU 验证步骤](windows-validation.md) 开始。

## 原理阅读入口

- 用 Mellum2.1 区分模型总参数规模与每个 token 的激活规模，随后再追踪运行时的专家权重布局。 [JetBrains Mellum2.1 模型卡](https://huggingface.co/JetBrains/Mellum2.1-12B-A2.5B-Thinking)
- 沿 DiskInfer 的模型加载、张量索引、按需读取、arena 和 CPU 节点回调梳理一次权重生命周期，并在实验 03 核对对应的版本。 [DiskInfer 历史设计说明](https://github.com/Horn-Studio/diskinfer-android/blob/main/readme_old.md)
- 在实验 04 对照 MLX safetensors 专家缓存与 GGUF 分页的实现边界。 [vLLM Metal 文档](https://docs.vllm.ai/projects/vllm-metal/en/latest/moe_offload/) · [llama.cpp 分页提案](https://github.com/ggml-org/llama.cpp/discussions/23324)
