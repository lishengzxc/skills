# OpenAI 兼容接口

OpenAI 兼容接口是阿里云百炼平台提供的一套遵循 OpenAI API 协议规范的服务端点，开发者只需将 `base_url` 和 `api_key` 替换为百炼平台的对应值，即可使用 OpenAI SDK 及其生态工具直接调用百炼上的模型和应用，实现低成本迁移。

## 核心价值

- **零改造迁移**：已有 OpenAI 代码仅需修改 `base_url`、`api_key` 和 `model` 三个参数即可接入百炼。
- **生态兼容**：支持 LangChain、Cursor、Claude Code、Cherry Studio、Dify 等主流框架和工具通过该接口对接。
- **多接口覆盖**：涵盖对话生成、文本补全、向量化、文件管理、批量推理等常见场景。

## 支持的接口端点

| 接口类型 | 用途 | 端点路径 |
|---------|------|---------|
| Chat Completions | 对话生成（最常用） | `/compatible-mode/v1/chat/completions` |
| Responses | 内置工具的智能体调用 | `/compatible-mode/v1/responses` |
| Completions | 文本/代码补全 | `/compatible-mode/v1/completions` |
| Embeddings | 文本向量化 | `/compatible-mode/v1/embeddings` |
| Files | 文件上传与管理 | `/compatible-mode/v1/files` |
| Batch | 批量推理 | `/compatible-mode/v1/batches` |

## 关键参数配置

### BASE_URL

根据部署地域选择：

| 地域 | BASE_URL |
|------|----------|
| 华北2（北京） | `https://dashscope.aliyuncs.com/compatible-mode/v1` |
| 新加坡 | `https://dashscope-intl.aliyuncs.com/compatible-mode/v1` |
| 美国（弗吉尼亚） | `https://dashscope-us.aliyuncs.com/compatible-mode/v1` |
| 德国（法兰克福） | `https://{WorkspaceId}.eu-central-1.maas.aliyuncs.com/compatible-mode/v1` |

> **重要**：不同地域的 API Key 相互独立，切换地域时必须同步更换 API Key。模型列表也因地域而异。

### API Key

在百炼控制台的 API Key 管理页面创建，建议配置为环境变量：

```bash
export DASHSCOPE_API_KEY="sk-xxx"
```

### 模型名称

直接使用百炼平台的模型标识符，如 `qwen3.7-max`、`qwen3.6-plus`、`qwen3.6-flash`、`text-embedding-v4` 等。

## 使用场景

### 1. 模型直接调用

最基础的用法，通过 OpenAI SDK 调用百炼上的千问及第三方模型：

```python
from openai import OpenAI
import os

client = OpenAI(
    api_key=os.getenv("DASHSCOPE_API_KEY"),
    base_url="https://dashscope.aliyuncs.com/compatible-mode/v1",
)

completion = client.chat.completions.create(
    model="qwen3.6-plus",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "你是谁？"}
    ]
)
print(completion.choices[0].message.content)
```

### 2. 应用调用（智能体/工作流）

百炼应用也提供 OpenAI 兼容的 Responses API 调用方式，`base_url` 中需包含应用 ID：

```python
client = OpenAI(
    api_key=os.getenv("DASHSCOPE_API_KEY"),
    base_url=f"https://dashscope.aliyuncs.com/api/v2/apps/agent/{app_id}/compatible-mode/v1/"
)
response = client.responses.create(input="你是谁？")
```

### 3. 第三方工具接入

Cursor、Claude Code、Cherry Studio、Dify 等工具均支持配置自定义 OpenAI 兼容端点。只需在工具的 API 设置中填入百炼的 BASE_URL 和 API Key 即可完成对接。

### 4. 专用模型调用

机器翻译（Qwen-MT）、OCR（Qwen-OCR）、语音翻译等专用模型也通过 OpenAI 兼容接口调用，部分模型需通过 `extra_body` 传入非标准参数：

```python
completion = client.chat.completions.create(
    model="qwen-mt-plus",
    messages=[{"role": "user", "content": "需要翻译的文本"}],
    extra_body={"translation_options": {"source_lang": "zh", "target_lang": "en"}}
)
```

## 与其他接口的关系

百炼平台同时提供 DashScope 原生接口和 Anthropic 兼容接口。三者对比：

| 维度 | OpenAI 兼容接口 | DashScope 接口 | Anthropic 兼容接口 |
|------|----------------|---------------|-------------------|
| 迁移成本 | 最低（OpenAI 生态） | 需适配专有 SDK | 低（Anthropic 生态） |
| 功能覆盖 | 覆盖主流场景 | 最完整 | 支持思考和工具调用 |
| 适用场景 | 已有 OpenAI 代码或工具链 | 需要平台全部能力 | 使用

## 关联主题页

- [toolkits and frameworks](../api/toolkits-and-frameworks.md)
- [get started with models](../guides/get-started-with-models.md)
- [use chat client or development tool](../guides/use-chat-client-or-development-tool.md)
- [qwen api reference](../api/qwen-api-reference.md)
- [application call](../api/application-call.md)
- [specialized model](../api/specialized-model.md)
- [speech translation api reference](../api/speech-translation-api-reference.md)

