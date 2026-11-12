# 本地模型

本目录登记实验所用模型的来源与身份，本地存放权重，不提供推理服务。资产契约见 [owning SPEC](SPEC.md)，研究范围见 [目标与范围](../goal-and-requirements.md)。

在项目根目录执行下载命令，需网络、足够磁盘空间及可用的 `uv`（提供 `uvx`）；访问限制按来源要求处理。默认路径相对于项目根目录，模型快照与下载缓存由 `.gitignore` 排除。已有文件时先核对版本和路径，不要盲目覆盖。维护者报告已下载模型，但本次文档工作未校验本地文件或加载模型。

输入是下列仓库、固定 revision 与文件选择；输出是本地快照及核验记录，供 [实验路线](../README.md#实验路线) 使用。下载、完整性核对、运行时可加载和输出正确是不同验证步骤；格式或 MoE 标签不保证兼容。

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

## 下载后核验（项目根目录）

以下命令读取文件，不下载或加载模型；散列计算会完整读取权重，可能耗时。不在性能测量期间执行。记录实际输出与日期到 [运行记录](../benchmarks/run-template.md)，不要把这里的命令当作已通过的结果。

### GGUF：大小与已知 SHA-256

macOS 自带 `stat` / `shasum`（此处的 `stat -f` 不是 Linux 语法）：

~~~sh
model=models/Mellum2.1/gguf-mxfp4/Mellum2.1-12B-A2.5B-Thinking-MXFP4_MOE.gguf
test "$(stat -f %z "$model")" = 7029537056 &&
  printf '%s  %s\n' cf09584a30423f6f80b0c9fe2f42e152520dabf08fecd1cad641ce21a7588166 "$model" | shasum -a 256 -c -
~~~

只有退出码为 0 且输出 `OK` 才表示两项匹配；失败时先确认路径和文件大小。

Windows PowerShell（不使用上面的 shell 语法）：

~~~powershell
$model = 'models/Mellum2.1/gguf-mxfp4/Mellum2.1-12B-A2.5B-Thinking-MXFP4_MOE.gguf'
if ((Get-Item -LiteralPath $model -ErrorAction Stop).Length -ne 7029537056) { throw 'GGUF 大小不匹配' }
$hash = (Get-FileHash -LiteralPath $model -Algorithm SHA256 -ErrorAction Stop).Hash
if ($hash -ne 'cf09584a30423f6f80b0c9fe2f42e152520dabf08fecd1cad641ce21a7588166') { throw 'GGUF SHA-256 不匹配' }
'GGUF 大小与 SHA-256 匹配'
~~~

### MLX：文件清单与本地指纹

macOS：

~~~sh
find models/Mellum2.1/mlx-6bit -type f ! -path '*/.cache/*' -exec shasum -a 256 {} \;
~~~

Windows PowerShell：

~~~powershell
Get-ChildItem -LiteralPath 'models/Mellum2.1/mlx-6bit' -Recurse -File -ErrorAction Stop |
  Where-Object { $_.FullName -notmatch '[\\/]\.cache[\\/]' } |
  Get-FileHash -Algorithm SHA256 -ErrorAction Stop |
  Format-List Path, Hash
~~~

将文件清单与上述固定 revision 的源仓库文件列表逐项核对；打开权重索引，确认其 `weight_map` 引用的分片均存在，并保留配置、分词器与聊天模板。以上命令只计算现有文件的本地指纹：没有可信的逐文件校验清单时，不能单凭有输出就断言快照完整、来自该 revision 或能够运行。不要用 GGUF 的单文件哈希代替 MLX 快照核验。

## 新增／修改模型记录

先说明研究用途、来源与完整 revision、文件集合、格式／量化及适用实验，再补下载和核验方法；未验证的架构支持明确标注。转换或修改权重时保留原始身份、转换工具版本与参数，使用独立目录，并记录新文件大小和哈希，不能沿用原文件身份。共享核验元数据即可，不提交权重、缓存或访问令牌；下载和衍生模型的使用、修改、再分发须分别核对模型卡及许可证，本仓库不代为授权。
