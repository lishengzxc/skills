# toolkits and [[frameworks|frameworks]]

阿里云百炼平台提供了一系列兼容 OpenAI 规范的 API 接口，覆盖文本对话、视觉理解、文本向量化、文件管理、批量推理等场景。开发者可通过 OpenAI SDK 或 LangChain 等主流框架，仅需修改 `base_url`、`api_key` 和 `model` 三个参数，即可将现有应用快速迁移至百炼平台。

## 兼容接口总览

百炼平台目前提供以下 [[openai-compatible-api|OpenAI 兼容接口]]：

| 接口类型 | 适用场景 | 说明 |
|---------|---------|------|
| **Chat Completions** | 文本对话、多轮会话 | 最常用的对话接口，支持流式/非[[streaming|流式输出]] |
| **Responses** | 智能体原生功能、内置工具调用 | Chat Completions 的演进版本，支持联网搜索等内置工具 |
| **Completions** | 代码补全、内容续写 | 基于前缀/后缀的文本补全，仅支持 Qwen Coder 模型 |
| **Vision** | 图像/视频理解 | 支持通义千问 VL 系列和 QVQ 系列模型 |
| **Embedding** | 文本向量化 | 支持 text-embedding-v1 至 v4 系列 |
| **Files** | 文件上传与管理 | 用于 Qwen-Long 文档问答和 Batch 任务输入 |
| **Batch Chat** | 低成本批量推理（单请求） | 同步方式，费用为实时调用的 50% |
| **Batch (File)** | 低成本批量推理（文件输入） | 异步方式，通过 JSONL 文件批量提交 |
| **Conversations** | 跨设备/跨场景对话延续 | 配合 Responses API 自动管理上下文 |

## 服务地址（BASE_URL）

根据调用方式和部署地域选择对应的 `base_url`，详见 [OpenAI Chat接口兼容](../../raw/model-api-reference/toolkits-and-[[frameworks|frameworks]]/compatibility-of-openai-with-dashscope.md)：

| 地域 | SDK base_url | HTTP endpoint（Chat） |
|------|-------------|----------------------|
| 北京 | `https://dashscope.aliyuncs.com/compatible-mode/v1` | `POST .../v1/chat/completions` |
| 弗吉尼亚 | `https://dashscope-us.aliyuncs.com/compatible-mode/v1` | `POST .../v1/chat/completions` |
| 新加坡 | `https://dashscope-intl.aliyuncs.com/compatible-mode/v1` | `POST .../v1/chat/completions` |

Batch Chat 接口使用独立端点：`https://batch.dashscope.aliyuncs.com/compatible-mode/v1`。

> **注意**：中国香港地域的旧版 URL `https://cn-hongkong.dashscope.aliyuncs.com/compatible-mode/v1` 即将下线，请迁移到新版路径 `https://{WorkspaceId}.cn-hongkong.maas.aliyuncs.com/compatible-mode/v1`。

## 支持的模型

### 文本生成（Chat / Responses）

- **商业版**：千问 Max（qwen3.7-max 等）、Plus（qwen3.6-plus 等）、Flash（qwen3.6-flash 等）、Turbo、Coder、Long 系列，以及 QwQ 推理系列
- **开源版**：qwen3-235b-a22b、qwen3-32b、qwen3-14b、qwen3-8b 等多种规格

### 代码补全（Completions）

仅支持 Qwen Coder 部分模型：qwen2.5-coder-7b/14b/32b-instruct、qwen-coder-turbo 系列。详见 [completions 接口](../../raw/model-api-reference/toolkits-and-[[frameworks|frameworks]]/completions.md)。

### 视觉理解（Vision）

支持 qwen3-vl-plus/flash、qwen-vl-max/plus 以及 QVQ 推理模型和 OCR 模型。

### 文本向量（Embedding）

| 模型 | 维度 | 单行最大 Token | 语种支持 |
|------|------|--------------|---------|
| text-embedding-v4 | 64~2048（默认 1024） | 8,192 | 100+ 语种 |
| text-embedding-v3 | 64~1024（默认 1024） | 8,192 | 50+ 语种 |
| text-embedding-v2/v1 | 1,536 | 2,048 | 较少语种 |

> **注意**：多模态 Embedding 模型（如 qwen3-vl-embedding）不支持 [[openai-compatible-api|OpenAI 兼容接口]]，需使用 [[multimodal-embedding]] 专用接口。

## 使用方式

### 前提条件

1. 获取 [[api-key]] 并配置到环境变量
2. 安装 OpenAI SDK：`pip install -U openai`

### 基础调用（Chat Completions）

```python
from openai import OpenAI
import os

client = OpenAI(
    api_key=os.getenv("DASHSCOPE_API_KEY"),
    base_url="https://dashscope.aliyuncs.com/compatible-mode/v1",
)

completion = client.chat.completions.create(
    model="qwen-plus",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "你是谁？"}
    ]
)
print(completion.choices[0].message.content)
```

### Responses API 调用

Responses API 支持更简洁的输入和通过 `previous_response_id` 自动关联多轮上下文（有效期 7 天），详见 [OpenAI Responses接口兼容](../../raw/model-api-reference/toolkits-and-frameworks/compatibility-with-openai-responses-api.md)：

```python
response = client.responses.create(
    model="qwen3.6-plus",
    input="你能做些什么？"
)
print(response.output_text)
```

### Conversations API

配合 Responses API 使用，可创建、查询、更新、删除会话，并向会话追加消息。适用于需要跨设备或长时间中断后继续对话的场景。

### LangChain 集成

百炼同时支持通过 LangChain 的 OpenAI 兼容方式和原生 DashScope 方式集成，详见 [在LangChain中使用阿里云百炼](../../raw/model-api-reference/toolkits-and-frameworks/use-bailian-in-langchain.md)：

| 方式 | Python 包 | 模型覆盖 |
|------|----------|---------|
| `langchain_openai` (ChatOpenAI) | `pip install langchain_openai` | 仅 OpenAI 兼容模型 |
| `langchain_community` (ChatTongyi) | `pip install langchain-community dashscope` | 百炼全部文本生成模型 |
| LangChain4j (Java) | Maven: `langchain4j-open-ai` | OpenAI 兼容模型 |

## 批量推理

对于数据标注、内容生成等无需实时响应的场景，可使用 Batch 接口以 **50% 的费用** 完成推理：

- **Batch Chat**：同步方式提交单条请求，使用 `batch.dashscope.aliyuncs.com` 端点，默认超时 3600 秒
- **Batch (File)**：通过 JSONL 文件异步提交大批量请求，完成后下载结果文件

> **注意**：在 Batch 场景下，`qwen3.7-max`、`qwen3.6` 和 `qwen3.5` 系列模型默认开启思考模式，会产生额外的思考 tokens。建议显式设置 `enable_thinking` 参数。

## 关键参数

各接口共有的核心参数：

| 参数 | 说明 |
|------|------|
| `model` | 模型名称（必选） |
| `temperature` | 控制生成多样性，取值 [0, 2.0)，与 `top_p` 建议只设一个 |
| `top_p` | 核采样概率阈值，取值 (0, 1.0] |
| `max_tokens` | 返回的最大 Token 数（截断而非限制生成） |
| `stream` | 是否[[streaming|流式输出]]，`true` 边生成边输出 |
| `stop` | 停止生成的关键词或 token_id |
| `seed` | 用于结果可复现 |

## 文件管理

通过 [[openai-file-interface]] 上传文件用于 Qwen-Long 文档问答或 Batch 任务：

- `purpose="file-extract"`：用于文档分析，支持 TXT/DOCX/PDF 等格式，单文件最大 150 MB
- `purpose="batch"`：用于 Batch 任务输入，JSONL 格式，单文件最大 500 MB
- 存储空间上限：10000 个文件，总大小不超过 100 GB

## 限制和注意事项

- 不同地域的 API Key 不通用，切换地域时需同步更换
- QVQ 模型仅支持[[streaming|流式输出]]
- Completions 接口当前仅适用于中国内地（北京地域），暂不支持通过给定后缀生成前缀内容
- Conversations API 的旧版 URL 路径 `/api/v2/apps/protocols/...` 即将停止维护，请迁移至新版路径
- LangChain4j 1.0.0-beta3 需要 Java 17 及以上版本

## 来源文档

- [OpenAI Chat接口兼容](../../raw/model-api-reference/toolkits-and-frameworks/compatibility-of-openai-with-dashscope.md)
- [OpenAI Responses接口兼容](../../raw/model-api-reference/toolkits-and-frameworks/compatibility-with-openai-responses-api.md)
- [completions 接口](../../raw/model-api-reference/toolkits-and-frameworks/completions.md)
- [OpenAI Vision接口兼容](../../raw/model-api-reference/toolkits-and-frameworks/qwen-vl-compatible-with-openai.md)
- [OpenAI文件接口兼容](../../raw/model-api-reference/toolkits-and-frameworks/openai-file-interface.md)
- [OpenAI Embedding接口兼容](../../raw/model-api-reference/toolkits-and-frameworks/embedding-interfaces-compatible-with-openai.md)
- [OpenAI兼容-Batch Chat](../../raw/model-api-reference/toolkits-and-frameworks/openai-compatible-batch-chat.md)
- [OpenAI兼容-Batch（文件输入）](../../raw/model-api-reference/toolkits-and-frameworks/batch-interfaces-compatible-with-openai.md)
- [OpenAI Conversations接口兼容](../../raw/model-api-reference/toolkits-and-frameworks/openai-compatible-conversations.md)
- [在LangChain中使用阿里云百炼](../../raw/model-api-reference/toolkits-and-frameworks/use-bailian-in-langchain.md)

