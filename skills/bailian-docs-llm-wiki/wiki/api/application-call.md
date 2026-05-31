# application call

阿里云百炼平台支持通过 API 调用已创建的**智能体**和**工作流**应用。平台提供两套 API 体系：DashScope API 和 OpenAI 兼容的 Responses API，均支持单轮/多轮对话、[[streaming|流式输出]]、参数传递等核心功能。调用前需获取 APP ID 和 [[api-key]]，并根据应用所在业务空间决定是否需要 Workspace ID。

## 支持的应用类型

- **智能体应用**（含新版 Agent 2.0）：支持文本对话、图像输入、文件输入、插件调用等
- **工作流应用**：支持文本对话、自定义参数传递、插件参数传递等

> **注意**：以上所有 API 文档目前**仅适用于中国大陆版（北京地域）**。德国（法兰克福）地域的调用需额外提供 Workspace ID。

## 前置准备

调用应用 API 前需完成以下步骤：

1. **创建应用**：在[应用管理](https://bailian.console.aliyun.com/?tab=app#/app-center)中创建智能体或工作流应用
2. **获取 APP ID**：从应用管理页面复制目标应用的 APP ID（详见 [获取APP ID和Workspace ID](../../raw/application-api-reference/application-call/obtain-the-app-id-and-workspace-id.md)）
3. **获取 API Key**：通过[密钥管理](https://bailian.console.aliyun.com/?tab=app#/api-key)获取，并配置到环境变量 `DASHSCOPE_API_KEY`
4. **安装 SDK（可选）**：根据所选 API 体系安装对应的 [[sdk]]

如果应用位于**子业务空间**，还需同时提供 Workspace ID。目前 APP ID 和 Workspace ID 只能通过控制台手动获取，不支持通过 API 或 CLI 查询。

## 两套 API 体系

### DashScope API

DashScope API 是百炼原生的应用调用接口，功能最全面。

- **Endpoint**：`POST https://dashscope.aliyuncs.com/api/v1/apps/{APP_ID}/completion`
- **SDK 支持**：Python（`dashscope`）、Java（`dashscope-sdk`）
- **HTTP 支持**：curl、PHP、Node.js、C#、Go
- **适用范围**：智能体（含新版 Agent 2.0）、工作流

详细参数和示例参见 [应用 API 参考](../../raw/application-api-reference/application-call/application-dashscope-api-reference/agent-and-workflow-application-api-reference.md)。新版智能体（Agent 2.0）的专用参数参见 [新版智能体应用 API 参考](../../raw/application-api-reference/application-call/application-dashscope-api-reference/new-agent-application-api-reference.md)。

基本调用示例（Python）：

```python
import os
from dashscope import Application

response = Application.call(
    api_key=os.getenv("DASHSCOPE_API_KEY"),
    app_id='APP_ID',
    [[prompt|prompt]]='你是谁？'
)
print(response.output.text)
```

### OpenAI Responses API

Responses API 兼容 OpenAI 协议，便于复用现有 OpenAI 生态代码和工具。

- **Endpoint**：`POST https://dashscope.aliyuncs.com/api/v2/apps/agent/{APP_ID}/compatible-mode/v1/responses`
- **SDK 支持**：OpenAI Python SDK、OpenAI Java SDK
- **调用模式**：同步调用、异步调用（`background=true`）

基本调用示例（Python）：

```python
from openai import OpenAI
import os

client = OpenAI(
    api_key=os.getenv("DASHSCOPE_API_KEY"),
    base_url=f'https://dashscope.aliyuncs.com/api/v2/apps/agent/{app_id}/compatible-mode/v1/'
)
response = client.responses.create(input="你是谁？")
```

## 关键功能

### 多轮对话

两套 API 的多轮对话实现方式不同：

| 特性 | DashScope API | Responses API |
|------|--------------|---------------|
| 上下文维护 | 通过 `session_id`（服务端管理） | 通过 `input` 消息数组（客户端管理） |
| 首次请求 | 不传 `session_id`，响应返回新 ID | 传入完整对话历史 |
| 后续请求 | 携带上次响应的 `session_id` | 追加新消息到数组 |
| 有效期 | `session_id` 最后请求后 1 小时有效 | 无限制（客户端自行维护） |

> **注意**：Responses API 中基于 `pre_response_id` 或 `conversation_id` 的上下文功能**尚未支持**，目前需在每次请求时传递完整对话历史。

### [[streaming|流式输出]]

- **DashScope API**：通过设置 `stream=True` 启用
- **Responses API**：通过设置 `stream=true` 启用
- **工作流应用**需在结束节点或流程输出节点中启用**[[streaming|流式输出]]**开关并重新发布

### 异步调用

Responses API 支持异步模式，适用于耗时较长的任务（如报告生成、多步骤工具调用），避免请求超时。核心流程：

1. 创建任务时设置 `background=true`，立即获取任务 ID
2. 通过 `retrieve` 方法轮询任务状态
3. 状态变为 `completed`/`failed`/`cancelled` 时获取结果

> **注意**：异步任务**不支持**流式输出（`stream=true`）。

### 多模态输入

Responses API 支持在 `content` 数组中传入多种类型：

- **图像输入**（`input_image`）：需选用通义千问 VL 系列模型
- **文件输入**（`input_file`）：仅智能体应用支持，需配置文件处理方式（全文引用或切片检索）

### 参数传递

- **自定义参数**：DashScope API 通过 `biz_params` 传递；Responses API 通过 `extra_body.biz_params` 传递
- **插件参数**：在应用中配置插件工具后，通过 API 传入对应参数

## 限制和注意事项

- APP ID 和 Workspace ID 目前仅支持通过控制台手动获取
- 默认业务空间下的应用只需 APP ID；子业务空间下的应用需同时提供 APP ID 和 Workspace ID
- 不建议在生产环境中将 API Key 硬编码到代码中，应通过环境变量配置
- RAM 子账号访问"业务空间管理"页面需要 [[permission-management]] 中的超级管理员权限
- DashScope Java SDK 建议版本 >= 2.12.0
- 在线调试可通过**应用卡片 → 发布 → API 调试**进入

## 来源文档

- [获取APP ID和Workspace ID](../../raw/application-api-reference/application-call/obtain-the-app-id-and-workspace-id.md)
- [新版智能体应用 API 参考](../../raw/application-api-reference/application-call/application-dashscope-api-reference/new-agent-application-api-reference.md)
- [异步调用API参考](../../raw/application-api-reference/application-call/openai-responses-api/asynchronous-call-api-reference.md)
- [同步调用 API 参考](../../raw/application-api-reference/application-call/openai-responses-api/synchronous-call-api-reference.md)
- [应用 API 参考](../../raw/application-api-reference/application-call/application-dashscope-api-reference/agent-and-workflow-application-api-reference.md)

