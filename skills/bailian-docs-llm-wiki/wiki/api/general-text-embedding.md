# general text embedding

通用文本向量（General Text Embedding）是百炼平台提供的将文本转换为数值向量的能力，适用于语义搜索、推荐、聚类、分类等下游任务。平台提供同步调用和批处理两种接口模式，支持多个模型版本，覆盖 100+ 语种。

## 支持的模型

### 同步接口模型

根据 [同步接口API详情](../../raw/model-api-reference/general-text-embedding/text-embedding-synchronous-api.md) 提供的信息，同步模型如下：

| 模型名称 | 向量维度 | 最大行数 | 单行最大 Token | 单价（每千 Token） | 语种 |
|---|---|---|---|---|---|
| **text-embedding-v4**（Qwen3-Embedding 系列） | 2048/1536/1024（默认）/768/512/256/128/64 | 10 | 8,192 | 0.0005 元 | 100+ 语种及编程语言 |
| **text-embedding-v3** | 1024（默认）/768/512/256/128/64 | 10 | 8,192 | 0.0005 元 | 50+ 语种 |
| **text-embedding-v2** | 1,536 | 25 | 2,048 | 0.0007 元 | 10 种语言 |
| **text-embedding-v1** | 1,536 | 25 | 2,048 | 0.0007 元 | 6 种语言 |

### 批处理接口模型

根据 [批处理接口API详情](../../raw/model-api-reference/general-text-embedding/text-embedding-batch-api.md)，批处理模型如下：

| 模型名称 | 向量维度 | 最大行数 | 单行最大 Token | 语种 |
|---|---|---|---|---|
| **text-embedding-async-v2** | 1,536 | 100,000 | 2,048 | 10 种语言 |
| **text-embedding-async-v1** | 1,536 | 100,000 | 2,048 | 6 种语言 |

> **注意**：批处理模型（async 系列）与同步模型（v1–v4）是独立的模型名称，不可混用。批处理接口目前仅支持 v1/v2 对应的异步版本，尚无 v3/v4 的异步版本。

## 关键参数

### 同步接口参数

| 参数 | 类型 | 必选 | 说明 |
|---|---|---|---|
| `model` | string | 是 | 模型名称，如 `text-embedding-v4` |
| `input` | string / array\<string\> / file | 是 | 输入文本，支持单条字符串、字符串列表或文件 |
| `dimensions` | integer | 否 | 向量维度。仅 v3/v4 支持，默认 1024。v4 额外支持 2048 和 1536 |
| `encoding_format` | string | 否 | 返回格式，当前仅支持 `float` |

### 批处理接口参数

| 参数 | 类型 | 必选 | 说明 |
|---|---|---|---|
| `model` | string | 是 | 模型名称，如 `text-embedding-async-v2` |
| `input.url` | string | 是 | 待向量化文件的 HTTP URL（一行一条文本，文件≤200MB） |
| `parameters.text_type` | string | 否 | `query`（查询文本）或 `document`（底库文本，默认值）。用于检索等非对称任务时建议区分 |

> **注意**：`text_type` 参数仅在批处理接口（DashScope API）中明确提供。同步接口的 OpenAI 兼容模式下未暴露此参数。

## 使用方式

### 同步调用（OpenAI 兼容）

同步接口兼容 OpenAI SDK，base_url 为：

```
https://dashscope.aliyuncs.com/compatible-mode/v1
```

HTTP endpoint：`POST https://dashscope.aliyuncs.com/compatible-mode/v1/embeddings`

Python 示例：

```python
from openai import OpenAI
import os

client = OpenAI(
    api_key=os.getenv("DASHSCOPE_API_KEY"),
    base_url="https://dashscope.aliyuncs.com/compatible-mode/v1"
)

result = client.embeddings.create(
    model="text-embedding-v4",
    input="待向量化的文本",
    dimensions=1024,
    encoding_format="float"
)
print(result.model_dump_json())
```

`input` 支持三种形式：单条字符串、字符串列表（如 `['文本1', '文本2']`）、文件对象。详见 [同步接口API详情](../../raw/model-api-reference/general-text-embedding/text-embedding-synchronous-api.md)。

### 批处理调用（DashScope API / HTTP 异步）

批处理适用于大规模文本向量化场景，单次最多 10 万行。HTTP 调用需两步：

1. **创建任务**（需设置请求头 `X-DashScope-Async: enable`），获取 `task_id`
2. **轮询任务状态**：`GET https://dashscope.aliyuncs.com/api/v1/tasks/{task_id}`

DashScope SDK 封装了同步等待和异步回调两种模式，详见 [批处理接口API详情](../../raw/model-api-reference/general-text-embedding/text-embedding-batch-api.md)。

```python
from dashscope import BatchTextEmbedding

result = BatchTextEmbedding.call(
    BatchTextEmbedding.Models.text_embedding_async_v2,
    url="https://example.com/your-text-file.txt",
    text_type="document"
)
```

## 限制和注意事项

- **Token 限制**：v3/v4 单行最大 8,192 Token，v1/v2 单行最大 2,048 Token。
- **行数限制**：同步接口 v3/v4 最多 10 行，v1/v2 最多 25 行；批处理接口最多 100,000 行。
- **维度选择**：`dimensions` 参数仅 v3 和 v4 支持。v4 比 v3 多了 2048 和 1536 两个维度选项。v1/v2 固定为 1536。
- **批处理限流**：单用户最多 50 个排队+运行中任务，最多 3 个并发运行，任务下发 RPS 限制为 1。
- **任务数据保留**：批处理任务的结果（含 URL）仅保留 **24 小时**，需及时下载保存。
- **免费额度**：同步模型各 100 万 Token（v3/v4）或 50 万 Token（v1/v2）；批处理模型各 2,000 万 Token。均需在百炼开通后 90 天内使用。
- **前置条件**：调用前需获取 [[api-key]] 并配置到环境变量，SDK 调用还需 [[install-sdk]]。
- 同步接口也支持通过 [[batch-interfaces-compatible-with-openai]] 进行 Batch 调用，价格为同步调用的 50%。

## 来源文档

- [同步接口API详情](../../raw/model-api-reference/general-text-embedding/text-embedding-synchronous-api.md)
- [批处理接口API详情](../../raw/model-api-reference/general-text-embedding/text-embedding-batch-api.md)

