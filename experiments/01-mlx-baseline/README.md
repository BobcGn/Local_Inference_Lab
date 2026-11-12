# 01 · Mellum2.1 MLX 基线

在 M5 MacBook Pro（32 GB 统一内存）上用 MLX-LM 运行 Mellum2.1 Thinking 6-bit，建立常规推理基线。**状态：待运行，无已提交实测证据。** 本实验不验证磁盘流式读取或专家卸载。

[实验索引](../README.md) · [实验契约](../SPEC.md)

## 前提与输入

维护者在 macOS 主线上执行。准备 Miniforge、完整的固定模型快照及足够本地存储；模型已下载的报告不能代替运行前的文件核对。输入为下述 MLX 6-bit 快照与 [三个固定提示词](prompts/README.md)，运行时版本、模板和生成条件一起进入记录。

## 环境和模型

以下手动命令从总仓库根目录执行，尚未在本仓库完成运行验证。使用 Miniforge 创建独立环境，确认 Python 架构为 `arm64`。

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

128 token 只用于确认加载、聊天模板和开始生成，不能作为完整回答验收。正式运行复用 [页缓存](prompts/page-cache.txt)、[C++ 引用](prompts/cpp-reference.txt) 和 [Issue 排查](prompts/issue-triage.txt) 三个输入。先试跑确定统一生成预算，初始值为 1024 token；若 Thinking 消耗预算而没有最终回答，保留截断记录后另设预算重跑，不改输入。

## 正式运行与输出

1. 固定模板、贪心采样及生成上限，保存实际输入／输出 token 数、停止原因和原始输出。按 [提示词人工检查说明](prompts/README.md) 逐题检查。
2. 首次 CLI 运行保存 `Prompt`、`Generation` 和 `Peak memory` 的原始报告及其来源，加载与已加载请求分开记录。
3. 已加载模型的正式测量使用独立请求和新的 KV 缓存，遵循 [基准测量口径](../../benchmarks/README.md)。`stream_generate` 可作为后续计时实现的入口，但本仓库尚无对应计时程序；需验证固定版本的接口与实际求值行为后再测。当前 CLI 首次验证的 TTFT 记为“未测”，不能由总耗时反推。
4. 输出为基于 [运行记录模板](../../benchmarks/run-template.md) 的记录、原始日志、三题回答及指标汇总，归档位置见 [实验索引](../README.md)。未测项注明原因。

## 验收与失败记录

- 加载／生成：固定快照在记录的环境中可加载并输出，版本、完整命令及日志可追溯。
- 回答：三题都有完整可检查的最终回答，逐题保留正确性与格式判断；仅生成若干 token 不算通过。
- 基线：正式指标按统一口径可复跑；缺少计时实现或采样时只报告已观察到的项目，不宣称基线已完成。
- 失败：环境安装、架构不符、模型缺文件、OOM、生成异常或截断均保存错误日志与复现条件，按 [失败记录指引](../README.md#输入产物与失败记录) 留档。即使三题生成成功，也不据此声称流式或卸载机制生效。

[MLX-LM 使用说明](https://github.com/ml-explore/mlx-lm) · [Mellum2.1 MLX 转换记录](https://huggingface.co/mlx-community/Mellum2.1-12B-A2.5B-Thinking-6bit/blob/main/README.md)
