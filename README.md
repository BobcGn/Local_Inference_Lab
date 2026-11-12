# Local_Inference_Lab

在 MacBook Pro M5（32 GB 统一内存）上研究本地大模型推理。先用 Mellum2.1 Thinking 建立可复现的基线，再测量权重驻留、磁盘流式读取和 MoE 专家按需加载对内存、延迟、吞吐量及输出一致性的影响。

研究动机来自 [DiskInfer 在 Android 设备运行大 GGUF 的实践](https://github.com/Horn-Studio/diskinfer-android)：类似的低内存权重管理方式，能否用于 Mac 和朋友的 Windows 电脑？本仓库以可行性验证、可复现实验和证据为交付，不做产品。

MoE 的路由决定每个 token 要计算哪些专家，运行时的权重管理决定其余专家是否驻留内存。Mellum2.1 因其 MoE 结构被选为初始研究样本，但通用权重流式读取与按路由加载专家并非同一种机制；两者都需要分别验证。

> **当前状态：实验尚未开始，没有已提交的实测结果或自动化脚本。** 维护者已下载模型，但本仓库不包含权重，本次文档整理也未校验本地模型或执行推理。下列命令与路线是待验证的操作方案，不是成功运行或性能收益的承诺。

## 实验路线

| 实验 | 要回答的问题 | 状态 |
| --- | --- | --- |
| [01 · Mellum2.1 MLX 基线](experiments/01-mlx-baseline/README.md) | MLX-LM 运行 Mellum2.1 Thinking 6-bit 时，模型能否稳定生成，首 token 延迟、吞吐量和内存占用是多少？ | 待运行 |
| [02 · llama.cpp CPU／Metal 基线](experiments/02-llama-metal/README.md) | 在同一份 Mellum2.1 GGUF 和相同推理参数下，CPU 与 Metal 表现有何差异？ | 待验证 |
| [03 · DiskInfer CPU 磁盘流式推理](experiments/03-disk-streaming/README.md) | 在 macOS 上能否用有限的权重缓冲区正确执行同一个小型 GGUF？ | 待验证 |
| [04 · MoE 专家卸载](experiments/04-moe-expert-offloading/README.md) | 在兼容的运行时中，专家缓存预算如何影响内存、I/O、吞吐量和输出？ | 待验证 |

macOS 从实验 01 的 MLX 基线开始，再做实验 02 的 CPU／Metal 对照。Windows 参与者从 [Windows CPU 验证](docs/windows-validation.md) 开始，用同一份 GGUF 记录本机结果。两条路线所需权重和固定版本见 [模型目录](models/README.md)。维护者负责的 macOS 主线与协作朋友负责的 Windows 验证分别保留本机证据；Windows 常规 CPU 生成并不证明磁盘流式读取可用。项目范围以 [目标与需求](goal-and-requirements.md) 为准，API／AgentHub 集成不属于当前交付。

## 目录

~~~text
Local_Inference_Lab/
├── repos/          # llama.cpp、DiskInfer、vLLM Metal 等独立源码检出
├── models/         # Mellum2.1/{mlx-6bit,gguf-mxfp4} 等本地模型文件
├── experiments/    # 实验方案与固定输入（目前无实验实现代码）
├── benchmarks/     # memory、throughput、correctness 与运行记录
├── docs/           # architecture、research-notes、decisions
└── scripts/        # download、build、benchmark
~~~

实验方案、固定输入和可分享的结果归总仓库管理。第三方源码保留各自的 Git 历史，模型文件与本地构建产物由 `.gitignore` 排除。

- 性能指标、运行条件与报告格式见 [基准测量口径](benchmarks/README.md) 和 [运行记录模板](benchmarks/run-template.md)。
- 源码版本与模型版本的记录方式见 [第三方源码](repos/README.md) 和 [模型目录](models/README.md)。
- 原理阅读入口与待验证项见 [技术文档索引](docs/README.md)。

## 从哪里开始

1. 阅读 [实验索引](experiments/README.md)，选择 macOS 基线或 Windows CPU 验证入口；不要为文档贡献下载权重或编译上游。
2. 按模型与源码目录说明准备固定版本，在本机核验，再执行所选实验的加载与生成检查。
3. 用运行记录模板保存条件、输出、失败与未测项。看到常规生成成功后，再单独验证流式或卸载机制。
4. 分享可复现的结果或修正文档前，阅读 [贡献指南](CONTRIBUTING.md)。

## 规格与协作入口

- [目标与需求](goal-and-requirements.md)：为什么研究、平台分工与非目标。
- [系统架构](architecture.md)：工作区边界、证据流和跨模块依赖。
- [规格与文档索引](docs/README.md)：每个真实职责边界的契约，当前均为待审阅的 `draft`。
- [贡献指南](CONTRIBUTING.md)：新增实验、Windows 协作、提交结果与文档检查。

README 提供操作入口，SPEC 记录意图和契约，运行记录提供证据。没有结果的路线保持待验证，不以文档齐全替代实验完成。
