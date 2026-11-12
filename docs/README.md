# 技术文档

本目录把外部资料、源码阅读和实验记录整理为可追溯的研究知识，不是产品使用手册。文档分工与发布边界归 [owning SPEC](SPEC.md)；这里只提供入口和记录方法。

## 从哪里开始

- [根 README](../README.md)：实验路线与目录；[目标与范围](../goal-and-requirements.md)：研究定位、平台分工与当前事实；[根架构](../architecture.md)：系统边界与证据流。
- [实验契约](../experiments/SPEC.md) 与 [实验路线](../README.md#实验路线)：选择待验证的问题和手动步骤。
- [模型](../models/README.md)（[SPEC](../models/SPEC.md)）、[源码](../repos/README.md)（[SPEC](../repos/SPEC.md)）：准备固定输入并核验身份。
- [测量入口](../benchmarks/README.md)、[测量 SPEC](../benchmarks/SPEC.md)、[运行记录模板](../benchmarks/run-template.md)：测量定义与原始证据；此处不另设测量口径。
- [脚本现状](../scripts/README.md)（[SPEC](../scripts/SPEC.md)）：目前无自动化工具。
- Windows 由朋友独立验证，从 [Mellum2.1 CPU 验证步骤](windows-validation.md) 开始；macOS 主线是 M5、32 GB 统一内存，两平台结果分别记录。

## 知识分类与贡献

| 分类 | 输入 | 输出与入口 |
| --- | --- | --- |
| 运行时实现说明 | 固定提交的源码、相关路径及资料 | [architecture](architecture/README.md)：数据流、资源生命周期与待验证点 |
| 研究笔记 | 来源摘记、假设、源码观察或运行证据 | [research-notes](research-notes/README.md)：分开标注事实与推断，结论附适用条件 |
| 决策检索／历史 | owning spec 的位置与历史修订 | [decisions](decisions/README.md)：只做索引与历史线索，不复制决策理由 |

新增记录先选分类，按对应 README 的精简结构填写来源版本、验证状态和证据链接，再检查相对链接。长期边界、契约或决策的变更应修改 owning spec；这里链接到它，不另写一份规范。分享前去除密钥、个人路径和无权公开的输入，模型及构建产物不作为文档附件；仓库没有顶层 LICENSE，不代表资料与代码可自由再分发。

当前没有已提交实测证据或自动化实现；维护者报告已下载模型，不代表本次检查验证了文件完整性。外部文档、源码阅读与实际运行必须区分，不能由计划推出已实现能力。

## 原理阅读入口

- 用 Mellum2.1 区分模型总参数规模与每个 token 的激活规模，随后再追踪运行时的专家权重布局。 [JetBrains Mellum2.1 模型卡](https://huggingface.co/JetBrains/Mellum2.1-12B-A2.5B-Thinking)
- 沿 DiskInfer 的模型加载、张量索引、按需读取、arena 和 CPU 节点回调梳理一次权重生命周期，并在实验 03 核对对应的版本。 [DiskInfer 历史设计说明](https://github.com/Horn-Studio/diskinfer-android/blob/main/readme_old.md)
- 在实验 04 对照 MLX safetensors 专家缓存与 GGUF 分页的实现边界。 [vLLM Metal 文档](https://docs.vllm.ai/projects/vllm-metal/en/latest/moe_offload/) · [llama.cpp 分页提案](https://github.com/ggml-org/llama.cpp/discussions/23324)
