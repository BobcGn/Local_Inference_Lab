# Windows · 独立验证入口

Windows 研究由朋友在自己的机器上独立负责；维护者没有 Windows 环境，主线是 macOS M5、32 GB 统一内存。**当前尚无 Windows 实测证据，下述手动命令待本机验证，不承诺平台可用。** 仓库的研究目标是验证低内存运行大模型的思路，不是交付跨平台产品，见 [目标与范围](../goal-and-requirements.md)。

先用 llama.cpp CPU 加载 Mellum2.1 Thinking MXFP4_MOE，检查常规生成与固定长度吞吐。**这不是磁盘流式读取验证，也不是专家卸载验证。** Windows 后续流式可行性仍需朋友另选版本、模型与运行路径并提供证据，不能由 Android 展示或 Mac 结果替代。

[实验索引](../experiments/README.md) · [实验 02](../experiments/02-llama-metal/README.md)

## 前提与输入

以下命令面向 Windows x64。准备 Git、`uvx` 和 Visual Studio 2022 的 **使用 C++ 的桌面开发**工作负载（包含 CMake 工具），确认本机存储及内存条件。在 Developer PowerShell for VS 2022 中执行。Windows ARM64 需独立核查固定版本的构建文档，本文 x64 命令不构成 ARM64 验证。[llama.cpp Windows 构建说明](https://github.com/ggml-org/llama.cpp/blob/956591f4570aa6636895d90439892fef7c075b17/docs/build.md#build-llamacpp-locally)

输入为固定源码 `956591f4570aa6636895d90439892fef7c075b17`、下述 GGUF 及 [三个固定提示词](../experiments/01-mlx-baseline/prompts/README.md)。所有路径均相对于总仓库根目录。此处没有自动测量脚本。

## 获取仓库和模型

~~~powershell
git clone https://github.com/BobcGn/Local_Inference_Lab.git
Set-Location Local_Inference_Lab
~~~

按 [第三方源码说明](../repos/README.md) 下载并检出固定的 llama.cpp 版本。执行 [模型目录说明](../models/README.md) 中的 GGUF 下载命令。运行以下命令，将结果与模型目录中的 SHA-256 对照。

~~~powershell
$gguf = 'models/Mellum2.1/gguf-mxfp4/Mellum2.1-12B-A2.5B-Thinking-MXFP4_MOE.gguf'
Get-FileHash -Algorithm SHA256 $gguf
~~~

## 构建与运行

~~~powershell
cmake -S repos/llama.cpp -B repos/llama.cpp/build-win-cpu -G "Visual Studio 17 2022" -A x64 -DBUILD_SHARED_LIBS=OFF -DGGML_CUDA=OFF -DGGML_VULKAN=OFF -DLLAMA_OPENSSL=OFF
cmake --build repos/llama.cpp/build-win-cpu --config Release --target llama-cli llama-bench --parallel

$bin = 'repos/llama.cpp/build-win-cpu/bin/Release'
& "$bin/llama-cli.exe" -m $gguf --device none -ngl 0 --no-kv-offload --no-op-offload --fit off --spec-type none -c 512 -n 32 --temp 0 --single-turn -f experiments/01-mlx-baseline/prompts/page-cache.txt
& "$bin/llama-bench.exe" -m $gguf -dev none -ngl 0 -nkvo 1 -nopo 1 -p 128,1024 -n 128 -r 3 -o json
~~~

保存构建和运行日志，确认实际使用 CPU 并开始生成；参数期望不替代日志证据。`-c 512 -n 32` 仅用于加载／生成冒烟检查，32 token 可能全部用于 Thinking，不要求此时回答完整。`llama-bench` 是独立的固定长度吞吐测试，不能用其 JSON 证明三题回答正确或流式读取生效。

## 正式检查、输出与验收

1. 按 [实验 02](../experiments/02-llama-metal/README.md) 的正式对照步骤，先试跑并固定可容纳输入、Thinking 和最终回答的上下文与生成预算，再运行三个固定输入；Windows 本机只做 CPU 组，不照搬 Metal 配置。
2. 按 [提示词检查说明](../experiments/01-mlx-baseline/prompts/README.md) 保存并检查最终回答、停止原因与截断状态。将任务正确性与 token／文本一致性分开报告。
3. 把 Windows 版本、架构、CPU 型号、内存、编译器、线程数、源码 SHA、GGUF SHA-256、完整命令、日志、原始输出和吞吐 JSON 记入 [运行记录模板](../benchmarks/run-template.md)。正式测量与统计遵循 [测量口径](../benchmarks/README.md)；无法获得的 TTFT、内存或读量标“未测”及原因，不能从 CLI 总耗时反推。
4. 常规 CPU 入口通过需要本机日志确认加载与实际后端、三个输入有可检查的最终回答、运行条件可复现；性能仅报告已经测到的项目，不把一次生成当成正式基准完成。

输出为本机运行记录与可分享日志／JSON，按 [实验索引的归档与失败指引](../experiments/README.md#输入产物与失败记录) 整理。构建失败、参数不支持、GGUF 校验不符、OOM、只有 Thinking 或截断都保留错误、退出状态、完整命令和未完成项；不为跑通而悄悄更换固定版本。

Windows 与 macOS 数据分别对应不同机器、工具链与后端条件，不能用跨机差异单独归因操作系统或后端；比较边界见 [架构](../architecture.md)。

## 后续：Windows 流式／专家卸载研究（未开始）

由朋友独立确认候选源码与构建路径、兼容模型及可观测机制，再记录真实运行命令。当前 [03](../experiments/03-disk-streaming/README.md) 的 commit、小 GGUF、macOS 路径尚未选定，[04](../experiments/04-moe-expert-offloading/README.md) 的 runtime、checkpoint、layout 也未选定；这些方案不提供 Windows 可用性保证，本文也没有额外的 Windows 流式命令。

若推进流式读取，需本机常规／流式同权重对照、有限缓冲区约束和实际读取证据；若推进专家卸载，需独立确认布局、路由与缓存行为。两种机制分别验收，按 [实验契约](../experiments/SPEC.md) 和 [证据契约](../benchmarks/SPEC.md) 留证。兼容性不成立或无法观测时，负结果也是研究产物，不以常规 CPU 成功替代。
