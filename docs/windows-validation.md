# Windows · Mellum2.1 CPU 验证

在 Windows 上用 llama.cpp CPU 加载 Mellum2.1 Thinking MXFP4_MOE，验证模型可以生成，并记录固定长度的吞吐。以下命令面向 Windows x64。准备 Git、`uvx` 和 Visual Studio 2022 的 **使用 C++ 的桌面开发**工作负载（包含 CMake 工具）。在 Developer PowerShell for VS 2022 中执行。Windows ARM64 可按固定源码版本的构建文档选择对应预设。[llama.cpp Windows 构建说明](https://github.com/ggml-org/llama.cpp/blob/956591f4570aa6636895d90439892fef7c075b17/docs/build.md#build-llamacpp-locally)

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

生成日志应显示模型成功加载、使用 CPU 并生成 token。32 token 上限用于验证加载与生成。正式验证回答时，再按 [实验 02](../experiments/02-llama-metal/README.md) 的参数补测。

把 Windows 版本、架构、CPU 型号、内存、编译器、线程数、GGUF SHA-256、完整运行命令、生成日志和吞吐 JSON 填入 [运行记录模板](../benchmarks/run-template.md)。Windows CPU 和 macOS CPU 的数据各自对应一台机器与一套工具链。同一台 M5 的 CPU／Metal 结果用于分析后端差异。
