# 本地模型

在项目根目录执行下载命令。默认路径相对于项目根目录，模型快照与下载缓存由 `.gitignore` 排除。已有文件时先核对版本和路径。

| 本地路径 | 用途 | 来源 |
| --- | --- | --- |
| `Mellum2.1/mlx-6bit/` | 实验 01，MLX-LM 常规 MoE 基线 | [MLX Community · Mellum2.1 Thinking 6-bit](https://huggingface.co/mlx-community/Mellum2.1-12B-A2.5B-Thinking-6bit) |
| `Mellum2.1/gguf-mxfp4/` | 实验 02，macOS 和 Windows 共用的 llama.cpp 模型 | [JetBrains · Mellum2.1 Thinking GGUF](https://huggingface.co/JetBrains/Mellum2.1-12B-A2.5B-Thinking-GGUF) |

## 实验 01 · MLX 6-bit

下载完整快照。两个权重分片与索引、配置、分词器及聊天模板一起保存在目标目录中。

~~~sh
uvx hf download mlx-community/Mellum2.1-12B-A2.5B-Thinking-6bit --revision 0c4927ce10ab23ff85db09c5bbede49dbebe1cb1 --local-dir models/Mellum2.1/mlx-6bit
~~~

## 实验 02 · GGUF MXFP4_MOE

在 macOS 或 Windows 只下载指定的 GGUF 文件。

~~~sh
uvx hf download JetBrains/Mellum2.1-12B-A2.5B-Thinking-GGUF Mellum2.1-12B-A2.5B-Thinking-MXFP4_MOE.gguf --revision 4159d684ffdb09ba89dc2a58feb3e75c54eef036 --local-dir models/Mellum2.1/gguf-mxfp4
~~~

文件大小为 7,029,537,056 字节，SHA-256 为 `cf09584a30423f6f80b0c9fe2f42e152520dabf08fecd1cad641ce21a7588166`。实验 03 的小型 GGUF 在确认 DiskInfer 所用版本支持模型架构和量化后再选。
