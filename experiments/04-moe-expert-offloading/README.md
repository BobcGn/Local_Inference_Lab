# 04 · MoE 专家卸载

验证按路由加载专家权重能否降低内存占用，并测量缓存预算对磁盘读取、TTFT、prefill、decode 和输出一致性的影响。

1. 固定待测运行时版本。先检查 Mellum2.1 的实际专家层实现、张量命名和权重布局。vLLM Metal 路线按该版本支持的 MLX-LM `SwitchGLU` 和 safetensors 布局核对。若布局不兼容，先用运行时已支持的 checkpoint 验证机制。llama.cpp 的 GGUF 磁盘分页提案作为独立的研究路径。
2. 在同一运行时与同一份权重上，对照常驻模式和专家卸载模式，从日志或仪表确认卸载生效。固定输入、采样、上下文与并发数。
3. 先测试一个适合机器内存的缓存预算，再调整预算。分别记录 prefill 和 decode 的缓存命中、读取量、延迟、吞吐、内存及输出 token ID。

验收以兼容性检查可复现、卸载实际生效、同权重对照可重复为准。对于输出差异，保留首个分歧及相关运行参数。

[vLLM Metal 专家卸载文档](https://docs.vllm.ai/projects/vllm-metal/en/latest/moe_offload/) · [llama.cpp 磁盘分页 RFC／PoC](https://github.com/ggml-org/llama.cpp/discussions/23324)
