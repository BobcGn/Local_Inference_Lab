# 第三方源码

在实验需要时，将源码检出到这里。每个源码目录保留自己的 Git 历史。实验记录注明远端地址、完整 commit SHA、是否有工作区修改及构建参数。需要复现修改时，在对应第三方仓库记录分支或提交。

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

macOS 与 Windows 使用同一源码版本。构建命令见 [macOS 基线](../experiments/02-llama-metal/README.md) 和 [Windows CPU 验证](../docs/windows-validation.md)。
