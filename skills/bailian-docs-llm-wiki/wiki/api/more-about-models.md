# [[more|more]] about models

本页面汇总了阿里云百炼平台在模型调用过程中涉及的进阶功能与配置，涵盖 API Key 安全管理、异步任务处理、子业务空间调用、连接优化以及文件上传等方面。这些内容适用于已完成基础模型调用、需要在生产环境中进一步优化和管理模型服务的开发者。

---

## API Key 安全管理

### 临时 API Key

在浏览器、移动 App 等不可信环境中，直接使用永久 API Key 存在泄露风险。百炼支持通过后端服务生成临时 API Key，有效期可配置为 1~1800 秒（默认 60 秒）。详见 [生成临时API Key](../../raw/model-api-reference/[[more|more]]-about-models/generate-temporary-api-key.md)。

**关键要点：**

- 临时 API Key **继承**生成它的永久 API Key 的全部权限（包括模型和知识库的访问限制）。
- 通过 `expire_in_seconds` 参数控制有效期，到期自动失效，**无法手动删除**。
- 请求示例：

```bash
curl -X POST "https://dashscope.aliyuncs.com/api/v1/tokens?expire_in_seconds=1800" \
-H "Authorization: Bearer $DASHSCOPE_API_KEY"
```

- 响应中返回 `token`（临时 Key）和 `expires_at`（UNIX 时间戳）。

> **注意**：各地域（北京、新加坡、弗吉尼亚）的 API Key 和 Endpoint 不同，请根据实际地域替换 URL。

### 子业务空间的 API Key

默认业务空间的 API Key 拥有调用所有模型的权限。如需进行权限管控或费用分账，可创建子业务空间并使用该空间专属的 API Key 进行调用。详见 [子业务空间的模型调用](../../raw/model-api-reference/[[more|more]]-about-models/model-calling-in-sub-workspace.md)。

**适用场景：**

- **权限管控**：限制 [[ram-user]] 仅可调用特定模型。
- **费用分账**：不同业务或场景各自独立生成账单。

**使用限制：**

- 调用标准模型（如 `qwen-plus`）前，需为子业务空间设置模型调用权限。
- 调用经百炼 [[model-training]] 调优并部署的模型无需额外授权，但仅限所在空间的 API Key 调用。
- 调优后的模型**仅支持通过 DashScope 方式调用**，不支持 [[openai-compatible-api|OpenAI 兼容接口]]。

---

## 异步任务管理

百炼中部分处理时间较长的模型（如 [[text-to-image]]、[[text-to-video]]）采用异步调用机制：先创建任务获取 `task_id`，再查询结果。

### 任务管理 API

百炼提供三个通用异步任务接口，所有接口限流 **20 QPS**（按主账号维度）：

| 接口 | 方法 | 说明 |
|------|------|------|
| 查询任务结果 | `GET /api/v1/tasks/{task_id}` | 根据 task_id 查询状态和结果 |
| 批量查询任务状态 | `GET /api/v1/tasks/` | 支持按时间、模型、状态等条件筛选 |
| 取消任务 | `POST /api/v1/tasks/{task_id}/cancel` | 仅支持取消 `PENDING` 状态的任务 |

**任务状态值**：`PENDING`（排队中）、`RUNNING`（处理中）、`SUCCEEDED`（成功）、`FAILED`（失败）、`CANCELED`（已取消）、`UNKNOWN`（不存在或未知）。

> **注意**：异步任务完成后通常保留 **24 小时**（具体以各模型 API 文档为准），超时后系统自动清理。跨主账号的任务无法查询或取消。

### 异步任务完成通知

相比频繁轮询查询结果接口，通过 [[eventbridge]] 事件总线接收任务完成通知是更高效的方案。详见 [通过HTTP回调URL或MQ接收异步任务完成通知](../../raw/model-api-reference/more-about-models/async-task-api.md)。

百炼支持两种通知接收方式：

| 方式 | 特点 | 适用场景 |
|------|------|----------|
| HTTP 回调 URL | 需公网或 VPC 可访问的 POST 接口 | 通用场景 |
| RocketMQ | 消息无丢失，支持失败重试 | 对消息可靠性要求高的场景 |

**事件关键字段**：

- 事件源：`acs.dashscope`
- 事件类型：`dashscope:System:AsyncTaskFinish`
- `data.task_id`：已完成的任务 ID
- `data.task_status`：任务最终状态

配置流程概要：在事件总线控制台（北京地域 `default` 总线）中创建事件规则，指定事件源和类型，配置事件目标（HTTP 或 RocketMQ），即可在任务完成后收到推送通知。

---

## 文件上传与临时 URL

调用多模态模型（如 `qwen-vl-plus`）时需要传入文件 URL。百炼提供免费临时存储空间，支持上传本地文件并获取 `oss://` 前缀的临时 URL（**有效期 48 小时**）。详见 [上传本地文件获取临时URL](../../raw/model-api-reference/more-about-models/get-temporary-file-url.md)。

**重要限制：**

| 限制项 | 说明 |
|--------|------|
| 文件与模型绑定 | 上传时须指定模型名称，且后续调用的模型必须一致 |
| 文件与主账号绑定 | 上传和调用的 API Key 需属于同一主账号 |
| 有效期 | 48 小时，过期自动清理 |
| 上传限流 | 100 QPS（按主账号+模型维度），不支持扩容 |
| 文件不可管理 | 上传后不可查询、修改或下载 |

> **注意**：临时 URL **不适用于生产环境**。生产环境请使用 [[aliyun-oss]] 等稳定存储。使用 `oss://` 形式的临时 URL 调用模型时，**必须**在 HTTP 请求头中添加 `X-DashScope-OssResourceResolve: enable`。

**上传方式：**

- **代码上传**（Python / Java）：调用 `/api/v1/uploads` 获取上传凭证，再上传文件至 OSS。
- **命令行工具**：`dashscope oss.upload --model <model_name> --file <file_path>`（需 DashScope Python SDK ≥ 1.24.0）。

---

## SDK 连接复用配置

在高并发场景下，通过连接复用可有效减少资源消耗、降低请求超时风险。

### Java SDK

内置连接池，默认启用。核心配置参数：

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `connectTimeout` | 120s | 建立连接超时 |
| `readTimeout` | 300s | 读取数据超时 |
| `connectionPoolSize` | 32 | 最大连接数 |
| `maximumAsyncRequests` | 32 | 最大并发请求数（需 ≤ 连接数） |
| `connectionIdleTimeout` | 300s | 空闲连接超时 |

通过 `Constants.connectionConfigurations` 进行全局配置。

### Python SDK

支持通过自定义 Session 实现连接复用：

- **异步调用**：使用 `aiohttp.ClientSession` + `aiohttp.TCPConnector`，可配置 `limit`（总连接数，默认 100）和 `limit_per_host`。
- **同步调用**：使用 `requests.Session`，推荐用 `with` 语句管理生命周期。

**最佳实践：**

- 异步架构（asyncio / FastAPI）使用异步调用方式；传统同步架构使用同步方式。
- Java 侧根据业务并发量合理调整 `connectionPoolSize` 和 `maximumAsyncRequests`。

---

## 调用方式与地域说明

百炼支持两种 API 调用协议：

| 协议 | 北京地域 base_url | 新加坡地域 base_url |
|------|-------------------|---------------------|
| OpenAI 兼容 | `https://dashscope.aliyuncs.com/compatible-mode/v1` | `https://dashscope-intl.aliyuncs.com/compatible-mode/v1` |
| DashScope | `https://dashscope.aliyuncs.com/api/v1` | `https://dashscope-intl.aliyuncs.com/api/v1` |

支持的 SDK 语言包括 Python、Java、Node.js、Go、C#、PHP 以及 curl 直接调用。

> **注意**：不同地域的 API Key 不通用，请确保 Endpoint 和 API Key 的地域一致。

## 来源文档

- [生成临时API Key](../../raw/model-api-reference/more-about-models/generate-temporary-api-key.md)
- [通过HTTP回调URL或MQ接收异步任务完成通知](../../raw/model-api-reference/more-about-models/async-task-api.md)
- [子业务空间的模型调用](../../raw/model-api-reference/more-about-models/model-calling-in-sub-workspace.md)
- [异步任务管理 API](../../raw/model-api-reference/more-about-models/manage-asynchronous-tasks.md)
- [DashScope SDK连接复用配置](../../raw/model-api-reference/more-about-models/connection-multiplexing-configuration.md)
- [上传本地文件获取临时URL](../../raw/model-api-reference/more-about-models/get-temporary-file-url.md)

