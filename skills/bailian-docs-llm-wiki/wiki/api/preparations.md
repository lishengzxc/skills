# preparations

在使用阿里云百炼平台的模型 API 之前，需要完成几项准备工作：获取 API Key、将其安全地配置到环境变量、以及安装相应的 SDK。本文汇总了这些前置步骤的核心信息，并涵盖了常见错误码的排查指引，帮助开发者快速完成接入准备。

## 获取 API Key

API Key 是调用百炼大模型或应用的鉴权凭证。详细的创建流程请参阅 [获取API Key](../../raw/model-api-reference/preparations/get-api-key.md)。

### 支持的地域与 Base URL

| 地域 | Base URL |
|------|----------|
| 华北2（北京） | `https://dashscope.aliyuncs.com/compatible-mode/v1` |
| 新加坡 | `https://dashscope-intl.aliyuncs.com/compatible-mode/v1` |
| 美国（弗吉尼亚） | `https://dashscope-us.aliyuncs.com/compatible-mode/v1` |
| 德国（法兰克福） | `https://{WorkspaceId}.eu-central-1.maas.aliyuncs.com/compatible-mode/v1` |

### 权限与业务空间

- **默认业务空间的 API Key**：可调用所有标准模型及该空间内的 [[application-introduction]]。
- **子业务空间的 API Key**：仅可调用已授权的标准模型及该空间内的应用。
- 同一空间内的 API Key 权限相同，无需为不同模型类型（文生文、文生图、语音合成等）分别创建。

### 关键限制

- 每个主账号在华北2（北京）、新加坡、德国（法兰克福）地域各最多创建 **50** 个 API Key；美国（弗吉尼亚）地域每个归属账号最多 **20** 个。
- API Key 无失效日期，手动删除后即失效。如需临时授权，可生成有效期 60 秒的临时 API Key。
- 目前仅华北2（北京）地域支持 IP 访问白名单等精细权限控制。
- RAM 用户被禁用或删除后，其创建的所有 API Key 均失效。

> **注意**：Coding Plan 使用专属 API Key（格式 `sk-sp-xxxxx`），与本文所述的百炼通用 API Key（格式 `sk-xxxxx`）不同，请勿混用。

## 配置 API Key 到环境变量

为避免在代码中硬编码 API Key 导致泄露风险，建议将其配置为环境变量 `DASHSCOPE_API_KEY`。[将API Key配置到环境变量](../../raw/model-api-reference/preparations/configure-api-key-through-environment-variables.md) 中提供了 Linux、macOS、Windows 三种系统的详细操作步骤。

### 快速参考

| 系统 | 永久性配置 | 临时性配置 |
|------|-----------|-----------|
| Linux (Bash) | `echo "export DASHSCOPE_API_KEY='sk-xxx'" >> ~/.bashrc && source ~/.bashrc` | `export DASHSCOPE_API_KEY="sk-xxx"` |
| macOS (Zsh) | `echo "export DASHSCOPE_API_KEY='sk-xxx'" >> ~/.zshrc && source ~/.zshrc` | `export DASHSCOPE_API_KEY="sk-xxx"` |
| Windows (CMD) | `setx DASHSCOPE_API_KEY "sk-xxx"` | `set DASHSCOPE_API_KEY=sk-xxx` |
| Windows (PowerShell) | `[Environment]::SetEnvironmentVariable("DASHSCOPE_API_KEY", "sk-xxx", [EnvironmentVariableTarget]::User)` | `$env:DASHSCOPE_API_KEY = "sk-xxx"` |

### 常见排查

如果 `echo` 命令确认变量已设置但代码仍提示找不到 API Key，通常原因是：
- 仅设置了临时环境变量，对已启动的 IDE 不生效
- 设置永久变量后未重启 IDE / 应用
- 使用 `sudo` 运行脚本时未传递环境变量（可改用 `sudo -E`）

## 安装 SDK

百炼支持通过 **DashScope SDK**（Python、Java）或 **OpenAI 兼容 SDK**（Python、Node.js、Java、Go）调用模型。完整的安装说明参见 [安装SDK](../../raw/model-api-reference/preparations/install-sdk.md)。

### Python（需 >= 3.8）

```bash
# OpenAI SDK
pip install -U openai

# DashScope SDK
pip install -U dashscope
```

### Java

- **DashScope SDK**：Maven artifact `com.alibaba:dashscope-sdk-java`
- **OpenAI Java SDK**：Maven artifact `com.openai:openai-java`（推荐 >= 3.5.0，需 Java 8+）

### Node.js

```bash
npm install --save openai
```

### Go（需 >= 1.22）

```bash
go get 'github.com/openai/openai-go/v3'
```

安装完成后，可参考 [[qwen-api-reference]]、[[text-to-image-v2-api-reference]]、[[cosyvoice-python-sdk]] 等文档开始调用各类模型，也可查阅 [[compatibility-of-openai-with-dashscope]] 了解 OpenAI 兼容性详情。

## 错误码与故障排查

调用模型失败时返回的错误码及解决方案，可参阅 [错误码](../../raw/model-api-reference/preparations/error-code.md)。以下列出几类高频错误：

### 参数类错误（400-InvalidParameter）

| 错误关键词 | 常见原因 | 解决方向 |
|-----------|---------|---------|
| `enable_thinking must be set to false for non-streaming calls` | 非流式调用了思考模式模型 | 设置 `enable_thinking=false` 或改用 [[stream]] 模式 |
| `Range of input length should be [1, xxx]` | 输入 Token 超限 | 控制 messages 长度或开启新对话 |
| `Range of max_tokens should be [1, xxx]` | `max_tokens` 超出模型上限 | 参照模型列表调整 |
| `Model not exist.` | 模型名称拼写/大小写错误 | 使用百炼模型 ID，勿混用开源社区名称 |
| `This model only [[support|support]] stream mode` | 模型仅支持[[streaming|流式输出]] | 启用 `stream=true` |

### 文件与多模态错误

| 错误关键词 | 常见原因 |
|-----------|---------|
| `Exceeded limit on max bytes per data-uri item` | 本地文件 Base64 编码后超 10 MB |
| `File format is not [[support|support]]ed` | Qwen-Long 仅支持 TXT/DOCX/PDF/EPUB/MOBI/MD |
| `Too many files provided` | file-id 数量超过 100 |

### 工具调用错误

- 发起 [[function-calling]] 时，需先将模型返回的 Assistant Message（含 `tool_calls`）添加到 messages 数组，再追加 Tool Message。
- `tool_choice` 仅支持 `"auto"` 或 `"none"`。

> **注意**：推荐使用[阿里云 AI 助理](https://www.aliyun.com/ai-assistant/)，输入完整报错信息即可获取针对性解决方案。

## 来源文档

- [获取API Key](../../raw/model-api-reference/preparations/get-api-key.md)
- [将API Key配置到环境变量](../../raw/model-api-reference/preparations/configure-api-key-through-environment-variables.md)
- [安装SDK](../../raw/model-api-reference/preparations/install-sdk.md)
- [错误码](../../raw/model-api-reference/preparations/error-code.md)

