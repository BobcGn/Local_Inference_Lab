# 02 · llama.cpp CPU／Metal 基线

用同一份 Mellum2.1 Thinking MXFP4_MOE GGUF 和同一份上游构建，对照 M5 上的 CPU 与 Metal 推理。Windows 参与者可以用同一模型、提示词和源码版本完成独立的 [Windows CPU 验证](../../docs/windows-validation.md)。

## macOS 构建与首次生成

先按 [第三方源码说明](../../repos/README.md) 准备 llama.cpp，再按 [模型目录说明](../../models/README.md) 准备 GGUF 并核对 SHA-256。在项目根目录执行。

~~~sh
cmake -S repos/llama.cpp -B repos/llama.cpp/build-mac-metal -DCMAKE_BUILD_TYPE=Release -DGGML_METAL=ON -DLLAMA_OPENSSL=OFF
cmake --build repos/llama.cpp/build-mac-metal --config Release --target llama-cli llama-bench --parallel

GGUF_PATH="${GGUF_PATH:-models/Mellum2.1/gguf-mxfp4/Mellum2.1-12B-A2.5B-Thinking-MXFP4_MOE.gguf}"
LLAMA_BIN=repos/llama.cpp/build-mac-metal/bin
PROMPT_FILE=experiments/01-mlx-baseline/prompts/page-cache.txt

"$LLAMA_BIN/llama-cli" -m "$GGUF_PATH" --device none -ngl 0 --no-kv-offload --no-op-offload --fit off --spec-type none -c 512 -n 32 --temp 0 --single-turn -f "$PROMPT_FILE"
"$LLAMA_BIN/llama-cli" -m "$GGUF_PATH" -ngl all --fit off --spec-type none -c 512 -n 32 --temp 0 --single-turn -f "$PROMPT_FILE"
~~~

首轮生成验证模型能加载并开始输出。用日志确认 CPU 组的实际后端及 Metal 组卸载的层数。

## 正式对照

1. 记录 llama.cpp commit SHA、编译器、构建选项、GGUF 校验值和实际后端。
2. 固定模板、上下文、批大小、线程、输入和采样，两组关闭 MTP。用 `llama-bench` 做固定长度测试。预设输入 128 和 1024 token、生成 128 token，每组重复三次并保留 JSON。
3. 保存自然提示词的加载时间、TTFT、内存和输出。记录 CPU／Metal 的首个输出差异（若有）。吞吐、时间和内存的记录方式见 [基准测量口径](../../benchmarks/README.md)。

验收以两个后端都能完成同一组输入、日志可确认实际后端、运行可重现为准。实验 01 的 MLX 6-bit 和本实验的 GGUF MXFP4_MOE 代表两套不同的运行配置。同一台 M5 上的同权重 CPU／Metal 实验用于衡量后端差异。
