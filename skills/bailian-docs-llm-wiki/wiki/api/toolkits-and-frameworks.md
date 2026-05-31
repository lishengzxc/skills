# toolkits and [frameworks](frameworks.md)

阿里云百炼平台提供与 OpenAI 兼容的多种 API 接口，开发者只需调整 API Key、BASE_URL 和模型名称，即可将现有 OpenAI 应用迁移至百炼服务。同时，百炼也支持与 LangChain 等主流框架集成，满足不同开发场景需求。

## 兼容接口总览

百炼平台提供以下 [OpenAI 兼容接口](../concepts/openai-compatible-api.md)：

| 接口类型 | 用途 | 端点路径 |
|---------|------|---------|
| Chat Completions | 对话生成 | `/compatible-mode/v1/chat/completions` |
| Responses | 智能体原生功能（Chat API 演进版） | `/compatible-mode/v1/responses` |
| Completions | 文本/代码补全 | `/compatible-mode/v1/completions` |
| Embeddings | 文本向量化 | `/compatible-mode/v1/embeddings` |
| Files | 文件上传/管理 | `/compatible-mode/v1/files` |
| Batch | 批量推理（文件输入） | `/compatible-mode/v1/batches` |
| Batch Chat | 批量对话（同步等待） | `batch.dashscope.aliyuncs.com/compatible-mode/v1/chat/completions` |
| Conversations | 会话管理 | `/compatible-mode/v1/conversations` |

## 关键参数配置

### BASE_URL

根据部署地域选择对应的 BASE_URL：

| 地域 | BASE_URL |
|------|----------|
| 北京 | `https://dashscope.aliyuncs.com/compatible-mode/v1` |
| 弗吉尼亚 | `https://dashscope-us.aliyuncs.com/compatible-mode/v1` |
| 新加坡 | `https://dashscope-intl.aliyuncs.com/compatible-mode/v1` |
| 法兰克福 | `https://{WorkspaceId}.eu-central-1.maas.aliyuncs.com/compatible-mode/v1` |

> **注意**：中国香港地域的旧版 URL `https://cn-hongkong.dashscope.aliyuncs.com/compatible-mode/v1` 即将下线，请迁移至新版路径。不同地域的 API Key 不同，切换地域时需同步更换。

### API Key

通过[阿里云百炼控制台](https://help.aliyun.com/zh/model-studio/get-api-key)获取 API Key，建议配置到环境变量 `DASHSCOPE_API_KEY` 中以降低泄露风险。

## 支持的模型

### 文本生成（Chat Completions）

根据 [OpenAI Chat接口兼容](../../raw/model-api-reference/toolkits-and-frameworks/compatibility-of-openai-with-dashscope.md) 文档，中国内地支持的商业版模型包括：

- **千问 Max 系列**：qwen3.7-max、qwen3-max 等
- **千问 Plus 系列**：qwen3.6-plus、qwen3.5-plus、qwen-plus 等
- **千问 Flash 系列**：qwen3.6-flash、qwen3.5-flash、qwen-flash 等
- **千问 Coder 系列**：qwen3-coder-plus、qwen3-coder-flash 等
- **QwQ 系列**：qwq-plus
- **开源版**：qwen3-235b-a22b、qwen3-32b、qwq-32b 等

### Responses API

根据 [OpenAI Responses接口兼容](../../raw/model-api-reference/toolkits-and-frameworks/compatibility-with-openai-responses-api.md) 文档，支持的模型包括：`qwen3.7-max`、`qwen3.6-plus`、`qwen3.6-flash`、`qwen3.5-plus`、`qwen3.5-flash`、`qwen-plus`、`qwen-flash`、`qwen3-coder-plus`、`qwen3-coder-flash` 等。

### Completions（代码补全）

支持 Qwen Coder 部分模型：qwen2.5-coder-7b-instruct、qwen2.5-coder-14b-instruct、qwen2.5-coder-32b-instruct、qwen-coder-turbo 等。仅适用于北京地域。

### 视觉模型

支持 qwen3-vl-plus、qwen3-vl-flash、qwen-vl-max、qwen-vl-plus、QVQ 系列及 OCR 系列模型。

### Embedding 模型

支持 text-embedding-v1 至 v4，其中 v4 属于 Qwen3-Embedding 系列，支持 100+ 语种，向量维度可选 64~2048。

## 使用方式

### 基础调用（OpenAI SDK）

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
        {'role': 'system', 'content': 'You are a helpful assistant.'},
        {'role': 'user', 'content': '你是谁？'}
    ]
)
print(completion.model_dump_json())
```

### LangChain 集成

根据 [在LangChain中使用阿里云百炼](../../raw/model-api-reference/toolkits-and-frameworks/use-bailian-in-langchain.md) 文档，支持 Python、JavaScript、Java 三种语言，且每种语言均有 OpenAI 兼容模式和 DashScope 原生模式两种接入方式：

**Python（OpenAI 兼容模式）：**

```python
from langchain_openai import ChatOpenAI

chatLLM = ChatOpenAI(
    api_key=os.getenv("DASHSCOPE_API_KEY"),
    base_url="https://dashscope.aliyuncs.com/compatible-mode/v1",
    model="qwen-plus",
)
```

**Python（DashScope 原生模式）：**

```python
from langchain_community.chat_models.tongyi import ChatTongyi

chatLLM = ChatTongyi(
    model="qwen-plus",
    dashscope_api_key=os.getenv("DASHSCOPE_API_KEY"),
)
```

**Java（LangChain4j）：**

```java
ChatLanguageModel model = OpenAiChatModel.builder()
    .apiKey(System.getenv("DASHSCOPE_API_KEY"))
    .baseUrl("https://dashscope.aliyuncs.com/compatible-mode/v1")
    .modelName("qwen-plus")
    .build();
```

> **注意**：LangChain4j 1.0.0-beta3 需要 Java 17 及以上版本。

### Responses API（智能体增强）

Responses API 相比 Chat Completions 提供了内置工具、更灵活的输入格式以及通过 `previous_response_id` 简化多轮对话上下文管理的能力（响应 ID 有效期 7 天）。

### Batch 推理

批量推理有两种模式：
- **Batch Chat**：同步等待，base_url 为 `https://batch.dashscope.aliyuncs.com/compatible-mode/v1`，默认超时 3600 秒
- **Batch File**：通过 JSONL 文件异步提交，费用为实时调用的 50%

## 限制和注意事项

- **Completions 接口**仅适用于北京地域，仅支持 Qwen Coder 部分模型
- **Batch 场景**下，`qwen3.7-max`、`qwen3.6` 和 `qwen3.5` 系列单次输入最大支持 256K Token
- `qwen3.7-max`、`qwen3.6` 和 `qwen3.5` 系列模型**默认开启思考模式**，会产生额外 token 成本。建议显式设置 `enable_thinking` 参数
- 文件上传：百炼存储空间最大 10000 个文件、总大小不超过 100 GB；`file-extract` 用途单文件最大 150 MB，`batch` 用途单文件最大 500 MB
- **多模态 Embedding 模型**（如 qwen3-vl-embedding）不支持 [OpenAI 兼容接口](../concepts/openai-compatible-api.md)
- QVQ 模型仅支持[流式输出](../concepts/streaming.md)
- Conversations API 中删除会话时，会话中的消息项不会被删除

> **注意**：多篇文档中提到旧版 URL 路径即将停止维护（如 `/api/v2/apps/protocols/compatible-mode/v1/responses`），请尽快迁移至新版路径。

## 来源文档

- [OpenAI Chat接口兼容](../../raw/model-api-reference/toolkits-and-frameworks/compatibility-of-openai-with-dashscope.md)
- [OpenAI Responses接口兼容](../../raw/model-api-reference/toolkits-and-frameworks/compatibility-with-openai-responses-api.md)
- [completions 接口](../../raw/model-api-reference/toolkits-and-frameworks/completions.md)
- [OpenAI Vision接口兼容](../../raw/model-api-reference/toolkits-and-frameworks/qwen-vl-compatible-with-openai.md)
- [OpenAI文件接口兼容](../../raw/model-api-reference/toolkits-and-frameworks/openai-file-interface.md)
- [OpenAI兼容-Batch Chat](../../raw/model-api-reference/toolkits-and-frameworks/openai-compatible-batch-chat.md)
- [OpenAI Embedding接口兼容](../../raw/model-api-reference/toolkits-and-frameworks/embedding-interfaces-compatible-with-openai.md)
- [OpenAI兼容-Batch（文件输入）](../../raw/model-api-reference/toolkits-and-frameworks/batch-interfaces-compatible-with-openai.md)
- [OpenAI Conversations接口兼容](../../raw/model-api-reference/toolkits-and-frameworks/openai-compatible-conversations.md)
- [在LangChain中使用阿里云百炼](../../raw/model-api-reference/toolkits-and-frameworks/use-bailian-in-langchain.md)

