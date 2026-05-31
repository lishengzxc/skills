# OpenAI 兼容接口

OpenAI 兼容接口是百炼平台提供的一组遵循 OpenAI API 规范的服务端点，开发者只需将 `base_url`、`api_key` 和 `model` 三个参数指向百炼，即可使用 OpenAI SDK 及生态工具调用千问（Qwen）全系列及 DeepSeek、Kimi、GLM 等第三方模型，无需修改业务代码逻辑。

## 核心配置

接入百炼 OpenAI 兼容接口只需配置三个参数：

| 参数 | 说明 | 示例 |
|------|------|------|
| `api_key` | 百炼 API Key，建议通过环境变量 `DASHSCOPE_API_KEY` 传入 | 参见 [[api-key]] |
| `base_url` | 服务端点，因地域和计费方案不同而异（见下表） | `https://dashscope.aliyuncs.com/compatible-mode/v1` |
| `model` | 模型名称 | `qwen-plus`、`qwen3.6-plus` 等，详见 [[models]] |

### 各地域 Base URL（按量计费）

| 地域 | Base URL |
|------|----------|
| 北京（默认） | `https://dashscope.aliyuncs.com/compatible-mode/v1` |
| 新加坡 | `https://dashscope-intl.aliyuncs.com/compatible-mode/v1` |
| 弗吉尼亚 | `https://dashscope-us.aliyuncs.com/compatible-mode/v1` |

其他计费方案使用独立端点：

| 方案 | Base URL |
|------|----------|
| Token Plan 团队版 | `https://token-plan.cn-beijing.maas.aliyuncs.com/compatible-mode/v1` |
| Coding Plan | `https://coding.dashscope.aliyuncs.com/v1` |
| Batch（批量推理） | `https://batch.dashscope.aliyuncs.com/compatible-mode/v1` |

## 支持的接口类型

| 接口 | 端点路径 | 适用场景 |
|------|----------|----------|
| **Chat Completions** | `/chat/completions` | 文本对话、多轮会话、视觉理解，最常用 |
| **Responses** | `/responses` | Chat Completions 的演进版，支持联网搜索等内置工具 |
| **Completions** | `/completions` | 代码补全（仅 Qwen Coder 模型） |
| **Embedding** | `/embeddings` | 文本向量化（text-embedding-v1~v4） |
| **Files** | `/files` | 文件上传与管理 |
| **Batch** | `/batches` | 异步批量推理，成本为实时调用的 50% |

## 基础用法

**Python（OpenAI SDK）**

```python
import os
from openai import OpenAI

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

**curl**

```bash
curl -X POST https://dashscope.aliyuncs.com/compatible-mode/v1/chat/completions \
  -H "Authorization: Bearer $DASHSCOPE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "qwen-plus",
    "messages": [{"role": "user", "content": "你是谁？"}]
  }'
```

安装 SDK：`pip install -U openai`

## 在百炼平台各场景中的使用

### 文本生成与视觉理解

文本对话、多轮会话、图片/视频分析等场景均通过 Chat Completions 接口调用。支持思考模式（`enable_thinking`）、Function Calling、结构化输出等高级能力。详见 [[deep-thinking]]、[[tool-calls]]、[[structured-output]]。

### 专用模型

机器翻译（`qwen-mt-plus`）、OCR（`qwen-vl-ocr-latest`）、GUI 自动化（`gui-plus`）等专用模型支持 OpenAI 兼容接口。专用参数通过 `extra_body` 传入，例如翻译模型的 `translation_options`。详见 [[specialized-model]]。

### 语音识别

仅 `qwen3-asr-flash` 模型支持通过 OpenAI 兼容接口进行录音文件识别；实时语音识别需使用 WebSocket 协议。

### 批量推理

通过 Batch 接口提交 JSONL 文件进行异步批量推理，成本约为实时调用的 50%。详见 [[batch-inference]]。

### 第三方工具与框架集成

Cursor、Claude Code、Codex、Cherry Studio、Dify 等工具均可通过配置 Base URL 和 API Key 接入百炼。LlamaIndex 和 Spring AI Alibaba 等开发框架同样基于该兼容接口集成。详见 [[

## 关联主题页

- [[get-started-with-models|get started with models]] — `../guides/get-started-with-models.md`
- [[toolkits-and-[[frameworks|frameworks]]|toolkits and frameworks]] — `../api/toolkits-and-[[frameworks|frameworks]].md`
- [[use-chat-client-or-development-tool|use chat client or development tool]] — `../guides/use-chat-client-or-development-tool.md`
- [[model-inference|model inference]] — `../guides/model-inference.md`
- [[specialized-model|specialized model]] — `../api/specialized-model.md`
- [[speech-recognition-api-reference|speech recognition api reference]] — `../api/speech-recognition-api-reference.md`
- [[frameworks|frameworks]] — `../api/[[frameworks|frameworks]].md`

