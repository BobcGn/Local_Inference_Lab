# 02 · llama.cpp CPU／Metal 基线

用同一份 Mellum2.1 Thinking MXFP4_MOE GGUF 和同一份上游构建，对照 M5 上的 CPU 与 Metal 常规推理。**状态：待验证，尚无实测结果；以下是手动入口，不是已验证的平台可用性承诺。**

[实验索引](../README.md) · [实验契约](../SPEC.md)

## 前提与输入

维护者在 macOS M5、32 GB 统一内存上执行，需准备 CMake、Apple 编译工具链与 Metal 构建条件。按 [第三方源码说明](../../repos/README.md) 固定 llama.cpp 为 `956591f4570aa6636895d90439892fef7c075b17`；按 [模型目录说明](../../models/README.md) 准备 GGUF 并核对 SHA-256。自然语言输入使用 [三个固定提示词](../01-mlx-baseline/prompts/README.md)。

Windows 研究由朋友独立负责，从 [Windows CPU 验证](../../docs/windows-validation.md) 开始；不是本实验的 Metal 对照，也不代表流式读取已可用。跨平台比较边界见 [架构](../../architecture.md)。

## macOS 构建与首次生成

在项目根目录执行，保留固定版本，不因参数报错就无记录地换成最新源码。

~~~sh
cmake -S repos/llama.cpp -B repos/llama.cpp/build-mac-metal -DCMAKE_BUILD_TYPE=Release -DGGML_METAL=ON -DLLAMA_OPENSSL=OFF
cmake --build repos/llama.cpp/build-mac-metal --config Release --target llama-cli llama-bench --parallel

GGUF_PATH="${GGUF_PATH:-models/Mellum2.1/gguf-mxfp4/Mellum2.1-12B-A2.5B-Thinking-MXFP4_MOE.gguf}"
LLAMA_BIN=repos/llama.cpp/build-mac-metal/bin
PROMPT_FILE=experiments/01-mlx-baseline/prompts/page-cache.txt

"$LLAMA_BIN/llama-cli" -m "$GGUF_PATH" --device none -ngl 0 --no-kv-offload --no-op-offload --fit off --spec-type none -c 512 -n 32 --temp 0 --single-turn -f "$PROMPT_FILE"
"$LLAMA_BIN/llama-cli" -m "$GGUF_PATH" -ngl all --fit off --spec-type none -c 512 -n 32 --temp 0 --single-turn -f "$PROMPT_FILE"
~~~

首轮的 `-c 512 -n 32` 仅验证模型能加载并开始输出，不是完整回答或正式长度测试配置。用日志确认 CPU 组的实际后端及 Metal 组卸载的层数；Metal 层卸载不等于磁盘专家卸载。

## 正式对照与输出

1. 记录 llama.cpp commit SHA、编译器、构建选项、GGUF 校验值和实际后端。
2. 两组固定模板、上下文、批大小、线程、输入和采样，关闭 MTP。先试跑确定可容纳输入、Thinking 与最终回答的上下文和生成预算，再固定为正式条件；不要沿用上述短输出上限验收三题。
3. 用 `llama-bench` 做独立的固定长度测试，预设输入 128 和 1024 token、生成 128 token。执行前核对固定版本的帮助及参数，保存实际命令与 JSON；此处不提供尚未验证的 macOS benchmark 命令。合成长度测试不能替代自然提示词检查。
4. 三个自然提示词分别保存原始输出、停止原因与截断状态，按 [人工检查说明](../01-mlx-baseline/prompts/README.md) 检查。保存 CPU／Metal 输出对照及首个分歧；无法取得 token ID 时说明只比较文本。
5. 测量次数与统计、加载／请求计时、内存等统一遵循 [基准测量口径](../../benchmarks/README.md)。当前没有自动化计时／采样实现，无法取得的 TTFT 等项目写“未测”。输出为 [运行记录](../../benchmarks/run-template.md)、两组日志、回答检查与指标汇总，按 [实验索引](../README.md) 归档。

## 验收与失败记录

- 两个后端都能完成同一组输入，日志确认实际后端，完整条件可复现；三题最终回答的人工检查与输出对照单独列出。
- 性能结论仅覆盖已按口径完成的测量；是否更快、是否更省内存由结果决定。本实验不验收磁盘流式读取或专家卸载。
- 构建失败、参数不被固定版本支持、模型不兼容、后端与预期不符、OOM、截断或输出分歧均保留原始记录，见 [失败记录指引](../README.md#输入产物与失败记录)。未跑通的一组不从报告中移除。

实验 01 的 MLX 6-bit 和本实验的 GGUF MXFP4_MOE 是不同配置，不以两者差异单独归因后端；对照边界见 [架构](../../architecture.md)。
