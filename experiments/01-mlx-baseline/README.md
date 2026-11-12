# 01 · Mellum2.1 MLX 基线

在 M5 MacBook Pro（32 GB 统一内存）上用 MLX-LM 运行 Mellum2.1 Thinking 6-bit，测量常规 MoE 推理的首 token 延迟、prefill、decode 与内存表现。当前状态为待运行。

## 环境和模型

从总仓库根目录执行。使用 Miniforge 创建独立环境，确认 Python 架构为 `arm64`。

~~~sh
mamba create -n local-inference-lab python=3.11
conda activate local-inference-lab
python -c 'import platform; print(platform.machine())'
python -m pip install mlx==0.32.3 mlx-lm==0.32.0
python -m pip show mlx mlx-lm
~~~

这组 MLX／MLX-LM 版本取自转换快照的记录。使用 [模型目录说明](../../models/README.md) 中固定版本的完整快照，记录实际模型路径、快照 SHA 和 macOS 版本。

## 第一次生成

~~~sh
MLX_MODEL_DIR="${MLX_MODEL_DIR:-models/Mellum2.1/mlx-6bit}"
mlx_lm.generate --model "$MLX_MODEL_DIR" --prompt "$(cat experiments/01-mlx-baseline/prompts/page-cache.txt)" --max-tokens 128 --temp 0
~~~

128 token 用于确认加载、聊天模板和生成路径。正式运行复用 [页缓存](prompts/page-cache.txt)、[C++ 引用](prompts/cpp-reference.txt) 和 [Issue 排查](prompts/issue-triage.txt) 三个输入。先试跑确定统一生成预算，初始值为 1024 token。固定模板、贪心采样及生成上限，记录实际输入和输出 token 数、停止原因和是否截断。

## 记录与验收

- 首次 CLI 运行保存 `Prompt`、`Generation` 和 `Peak memory`。为每项记录 MLX-LM 的报告口径。
- 正式测量时保持模型已加载，每个提示词预热一次并计时三次，每次使用独立请求和新的 KV 缓存。记录逐次值和中位数。
- 用 `stream_generate` 从已加载模型开始处理请求计时，到首个 token 可用为 TTFT。Thinking token 计入生成长度和 decode 吞吐。若能识别最终回答的起点，可单独记录到达该起点的时间。CLI 首次验证的 TTFT 记为待测。
- 验收时三题均有完整可检查的最终回答，原始输出和运行条件可查，指标可复跑。用 [基准测量口径](../../benchmarks/README.md) 和 [运行记录模板](../../benchmarks/run-template.md) 留证。

[MLX-LM 使用说明](https://github.com/ml-explore/mlx-lm) · [Mellum2.1 MLX 转换记录](https://huggingface.co/mlx-community/Mellum2.1-12B-A2.5B-Thinking-6bit/blob/main/README.md)
