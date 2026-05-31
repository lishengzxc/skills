# [[more|more]] models

百炼平台除了通用大语言模型外，还提供多种专用模型，涵盖文本排序（Rerank）、意图理解、法律行业等场景。这些模型通过 DashScope API 或兼容 OpenAI 的接口调用，适用于 RAG 检索增强、工具选择、行业垂直应用等开发需求。

## 支持的模型

### 文本排序模型

文本排序模型用于对检索召回阶段的文档做二次精排，提升结果相关性。详见 [文本排序](../../raw/model-api-reference/[[more|more]]-models/text-rerank-api.md)。

| 模型名称 | 最大文档数 | 单条最大Token | 请求最大Token | 语种 | 典型场景 |
|---------|-----------|-------------|-------------|------|---------|
| qwen3-rerank | 500 | 4,000 | — | 100+语种 | 文本语义检索、RAG |
| qwen3-vl-rerank | 文本100/图片40/视频4 | 8,000 | 120,000 | 33种语言 | 跨模态搜索、图片检索 |
| gte-rerank-v2 | — | — | 30,000 | 50+语种 | 文本排序 |

> **注意**：gte-rerank 模型将于 2026-05-30 下线，推荐迁移至 qwen3-rerank。

### 意图理解模型

| 模型名称 | 上下文长度 | 最大输出 | 输入成本 | 输出成本 |
|---------|-----------|---------|---------|---------|
| tongyi-intent-detect-v3 | 8,192 | 1,024 | 0.4元/百万Token | 1元/百万Token |

该模型可在百毫秒级完成意图识别和工具选择，详见 [意图理解能力](../../raw/model-api-reference/[[more|more]]-models/intent-detect-capability.md)。

### 法律行业模型

| 模型名称 | 上下文长度 | 最大输出 | 输入成本 |
|---------|-----------|---------|---------|
| farui-plus | 12k | 2k | 20元/百万Token |

通义法睿基于千问微调，支持法律咨询、文书生成、案情分析等任务，详见 [通义法睿大语言模型](../../raw/model-api-reference/more-models/tongyi-farui-api.md)。

## 调用方式

### 文本排序模型

不同模型使用不同 API 端点：

- **qwen3-rerank**：`POST https://dashscope.aliyuncs.com/compatible-api/v1/reranks`
- **qwen3-vl-rerank / gte-rerank-v2**：`POST https://dashscope.aliyuncs.com/api/v1/services/rerank/text-rerank/text-rerank`

两种接口的请求体结构不同。`qwen3-rerank` 的 `query`、`documents` 等参数位于顶层；其他模型需嵌套在 `input` 和 `parameters` 对象中。

SDK 调用示例（Python）：

```python
import dashscope

resp = dashscope.TextReRank.call(
    model="qwen3-rerank",
    query="什么是文本排序模型",
    documents=["文档1", "文档2", "文档3"],
    top_n=2,
    return_documents=True,
    instruct="Given a web search query, retrieve relevant passages that answer the query."
)
```

### 意图理解模型

通过 [[openai-compatible-api|OpenAI 兼容接口]]或 DashScope SDK 调用，使用 [[chat-completions]] 式的 messages 结构。关键在于 System Message 的设置：

- **意图 + 函数调用**：在 system [[prompt|prompt]] 中加入工具定义并声明 `Response in INTENT_MODE.`
- **仅意图分类**：在 system [[prompt|prompt]] 中提供意图标签列表并声明 `Just reply with the chosen tag.`
- **仅函数调用**：按标准 [[function-calling]] 方式配置

> **注意**：意图理解模型的响应使用 `<tags>`、`<tool_call>`、`<content>` 标签格式，需自行解析，不同于标准 OpenAI function_call 响应结构。

**性能优化技巧**：将意图类别映射为单个大写字母（A/B/C...），可使响应固定为 1 个 Token，显著降低延迟。

### 法律行业模型

farui-plus 使用标准的 `Generation.call` 接口，支持单轮、多轮对话及[[streaming|流式输出]]，与通用千问模型调用方式一致：

```python
import dashscope

response = dashscope.Generation.call(
    model="farui-plus",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "我哥欠我10000块钱，给我生成起诉书。"}
    ],
    result_format="message"
)
```

## 关键参数说明

### 排序模型特有参数

| 参数 | 说明 |
|------|------|
| `top_n` | 返回排序后前 N 个文档，默认返回全部 |
| `return_documents` | 是否返回文档原文（仅 gte-rerank-v2、qwen3-vl-rerank） |
| `instruct` | 排序任务类型说明（仅 qwen3-rerank、qwen3-vl-rerank），建议英文撰写 |
| `fps` | 视频帧率控制，范围 [0,1]（仅 qwen3-vl-rerank） |

`relevance_score` 为 0.0–1.0 的相对分数，仅用于当前请求内排序，不可跨请求比较。

### 请求 Token 计算（排序模型）

```
总Token = Query Tokens × Document数量 + Document Tokens总和
```

该值不得超过模型的请求最大输入 Token 限制。超长单条输入会被截断，可能导致排序结果不准确。

## 前提条件

所有模型调用前需要：

1. 获取 [[api-key]] 并配置到环境变量 `DASHSCOPE_API_KEY`
2. 如使用 SDK，需安装 [[dashscope-sdk]]

## 限制和注意事项

- 排序模型的 `qwen3-vl-rerank` 对不同模态文档数量有独立限制（文本 100、图片 40、视频 4）
- `qwen3-vl-rerank` 视频输入仅支持 URL，不支持 Base64；图片支持 URL 和 Base64
- 意图理解模型不支持标准 `tools` 参数传入，需通过 system [[prompt|prompt]] 注入工具定义
- farui-plus 仅支持输入计费（20元/百万Token），文档中未列出输出成本
- 模型限流条件请参考 [[rate-limit]]
- 错误码详情请参考 [[error-code]]

## 来源文档

- [文本排序](../../raw/model-api-reference/more-models/text-rerank-api.md)
- [意图理解能力](../../raw/model-api-reference/more-models/intent-detect-capability.md)
- [通义法睿大语言模型](../../raw/model-api-reference/more-models/tongyi-farui-api.md)

