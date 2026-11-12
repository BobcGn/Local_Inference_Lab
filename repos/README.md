# 第三方源码

本目录用于独立检出外部运行时，供源码阅读、构建和研究验证，不是已集成的推理产品。源码身份与修改溯源契约见 [owning SPEC](SPEC.md)，平台及研究边界见 [目标与范围](../goal-and-requirements.md)。

输入是上游 URL、明确版本和本地修改；输出是独立源码工作区，以及写入 [运行记录](../benchmarks/run-template.md) 的源码身份与构建信息。源码及构建产物不提交总仓库，不擅自改成 submodule 或 vendor。本次文档工作未执行下载、构建或推理。

在实验需要时，将源码检出到这里，每个源码目录保留自己的 Git 历史。先检查现有目录，避免覆盖本地修改。准备命令需要 Git、网络与足够磁盘空间；工具链和构建选项按具体实验说明准备。源码可阅读不等于目标平台可运行，上游 Android 展示或发布意向不作为 macOS／Windows 的验证结果。

| 本地目录 | 准备时机和用途 | 上游 |
| --- | --- | --- |
| `llama.cpp/` | 实验 02 从上游 clone，建立 GGUF 的 CPU／Metal 基线 | [llama.cpp](https://github.com/ggml-org/llama.cpp) |
| `diskinfer/` | 实验 03 查阅上游，在独立检出中验证常规加载与流式加载 | [DiskInfer Android](https://github.com/Horn-Studio/diskinfer-android) |
| `vllm-metal/` | 实验 04 检查专家卸载实现和权重布局兼容性时按需检出 | [vLLM Metal](https://github.com/vllm-project/vllm-metal) |

## 实验 02 · 固定 llama.cpp 版本

从总仓库根目录执行。已有 `repos/llama.cpp/` 时可直接核对它的当前提交。

~~~sh
git clone https://github.com/ggml-org/llama.cpp.git repos/llama.cpp
git -C repos/llama.cpp checkout --detach 956591f4570aa6636895d90439892fef7c075b17
git -C repos/llama.cpp rev-parse HEAD
~~~

macOS 与 Windows 使用同一源码版本。构建命令见 [macOS 基线](../experiments/02-llama-metal/README.md) 和 [Windows CPU 验证](../docs/windows-validation.md)。现有目录若有修改，先按下节记录，勿直接切换或清理。

## 只读核验与记录

以下命令从总仓库根目录执行，适用于 macOS shell 和 Windows PowerShell（均需 Git）。以 llama.cpp 为例；其他运行时替换目录，并记录自己实际选择的完整提交，不能套用本节的 llama.cpp SHA。

~~~sh
git -C repos/llama.cpp remote -v
git -C repos/llama.cpp rev-parse HEAD
git -C repos/llama.cpp status --short --untracked-files=all
git -C repos/llama.cpp diff HEAD --stat
git -C repos/llama.cpp diff HEAD --binary
git -C repos/llama.cpp submodule status --recursive
~~~

- `remote -v` 应对应表中上游（镜像或 fork 需注明）；分享日志前移除远端 URL 内可能携带的凭据。
- `rev-parse HEAD` 应输出上方固定 SHA。`status` 无输出只说明没有列出的修改或未跟踪文件，不证明构建产物对应此提交；忽略文件也不在其检查范围。
- `diff HEAD --binary` 展示相对当前提交的已跟踪修改（包含暂存与未暂存的最终差异），不包含未跟踪文件。若存在子模块，记录实际子模块提交和各自修改，不能只凭主仓库 SHA。
- 将这些输出和实际工具链版本、构建命令／选项、执行的二进制路径与构建日志关联到同一运行记录；版本相符不证明后端启用或运行正确，仍需实际运行证据。

## 修改、贡献与许可

修改上游时，运行记录提供基准 SHA 和可访问的修改提交，或可复现补丁（包括必要的未跟踪源码及应用方法）；仅写本地分支名不足以复现。不要分享带凭据的补丁，也不要把整个检出或构建目录放进总仓库。总仓库可保存经审查的版本说明和补丁。

新增运行时先说明对应研究问题、上游来源、固定版本、平台与验证状态，再补所需实验步骤；尚未选择的 DiskInfer／vLLM Metal 提交不在这里虚构固定值。第三方代码修改、补丁及二进制分发分别遵守该版本的 LICENSE、NOTICE 等要求，记录上游地址不等于获得再分发授权。
