# Local_Inference_Lab

在 MacBook Pro M5（32 GB 统一内存）上研究本地大模型推理。先用 Mellum2.1 Thinking 建立可复现的基线，再测量权重驻留、磁盘流式读取和 MoE 专家按需加载对内存、延迟、吞吐量及输出一致性的影响。

MoE 的路由决定每个 token 要计算哪些专家。运行时的权重管理则决定其余专家是否需要驻留内存。实验会从常规推理入手，再逐步验证按需加载能节省多少内存，以及磁盘 I/O 会带来多少开销。

## 实验路线

| 实验 | 要回答的问题 | 状态 |
| --- | --- | --- |
| [01 · Mellum2.1 MLX 基线](experiments/01-mlx-baseline/README.md) | MLX-LM 运行 Mellum2.1 Thinking 6-bit 时，模型能否稳定生成，首 token 延迟、吞吐量和内存占用是多少？ | 待运行 |
| [02 · llama.cpp CPU／Metal 基线](experiments/02-llama-metal/README.md) | 在同一份 Mellum2.1 GGUF 和相同推理参数下，CPU 与 Metal 表现有何差异？ | 待验证 |
| [03 · DiskInfer CPU 磁盘流式推理](experiments/03-disk-streaming/README.md) | 在 macOS 上能否用有限的权重缓冲区正确执行同一个小型 GGUF？ | 待验证 |
| [04 · MoE 专家卸载](experiments/04-moe-expert-offloading/README.md) | 在兼容的运行时中，专家缓存预算如何影响内存、I/O、吞吐量和输出？ | 待验证 |

macOS 从实验 01 的 MLX 基线开始，再做实验 02 的 CPU／Metal 对照。Windows 参与者从 [Windows CPU 验证](docs/windows-validation.md) 开始，用同一份 GGUF 记录本机结果。两条路线所需权重和固定版本见 [模型目录](models/README.md)。更大的模型、Metal 流式权重调度与通过 OpenAI 兼容 API 接入 AgentHub 是后续研究方向。

## 目录

~~~text
Local_Inference_Lab/
├── repos/          # llama.cpp、DiskInfer、vLLM Metal 等独立源码检出
├── models/         # Mellum2.1/{mlx-6bit,gguf-mxfp4} 等本地模型文件
├── experiments/    # 分阶段的方案、输入与实验代码
├── benchmarks/     # memory、throughput、correctness 与运行记录
├── docs/           # architecture、research-notes、decisions
└── scripts/        # download、build、benchmark
~~~

实验方案、固定输入和可分享的结果归总仓库管理。第三方源码保留各自的 Git 历史，模型文件与本地构建产物由 `.gitignore` 排除。

- 性能指标、运行条件与报告格式见 [基准测量口径](benchmarks/README.md) 和 [运行记录模板](benchmarks/run-template.md)。
- 源码版本与模型版本的记录方式见 [第三方源码](repos/README.md) 和 [模型目录](models/README.md)。
- 原理阅读入口与待验证项见 [技术文档索引](docs/README.md)。
