# assistantapi

Assistant API 是百炼平台提供的一套用于构建大模型智能体应用的 API，采用 Assistant、Thread、Message、Run 等核心对象来组织对话与任务执行流程。**该 API 目前正在下线中**，建议开发者迁移至 [[responses-api]]（内置多种工具并支持多轮上下文管理）作为替代方案。

> **注意**：所有 7 篇原始文档均标注 Assistant API 处于"下线中"状态，请勿用于新项目开发，仅供存量业务维护参考。

## 核心概念与组件

Assistant API 围绕以下五个核心对象构建：

| 对象 | 说明 |
|------|------|
| **Assistant** | 智能体实例，绑定模型、指令（instructions）和工具（tools） |
| **Thread** | 对话线程，承载消息上下文 |
| **Message** | 线程中的单条消息，目前仅支持 `role="user"` 创建 |
| **Run** | 在指定线程上运行指定智能体的任务，支持流式与非流式 |
| **Run Step** | 运行过程中的单个步骤，类型为 `message_creation` 或 `tool_calls` |

典型调用流程为：创建 Assistant → 创建 Thread（附带初始消息）→ 创建 Run → 轮询或流式获取结果 → 通过 Messages.list 读取回复。完整代码示例见 [Assistant API 调用示例（下线中）](../../raw/application-api-reference/assistantapi/call-example.md)。

## 支持的模型与工具

### 模型

创建 Assistant 时通过 `model` 参数指定模型，示例中常用 `qwen-max`，也可按需替换为其他百炼支持的模型。详见 [Assistants（下线中）](../../raw/application-api-reference/assistantapi/assistant.md) 中创建智能体的参数说明。

### 内置工具

| 工具类型 | 说明 | 支持[[streaming|流式输出]] |
|----------|------|:---:|
| `code_interpreter` | 代码解释器 | ✅ |
| `search` / `quark_search` | 夸克搜索 | ✅ |
| `text_to_image` | 文生图 | ✅ |
| `calculator` | 计算器 | ✅ |
| `function` | 自定义函数调用 | ❌（需通过 `requires_action` 处理） |
| 自定义插件 | 通过插件 ID 指定，支持 `user_http` 鉴权 | ❌ |

## 关键参数

### Assistant 创建参数

| 参数 | 类型 | 必须 | 说明 |
|------|------|:----:|------|
| `model` | string | 是 | 模型名称 |
| `name` | string | 否 | 智能体名称 |
| `instructions` | string | 否 | System [[prompt|prompt]] |
| `tools` | array | 否 | 可调用的工具列表 |
| `temperature` | float | 否 | 控制随机性 |
| `top_p` | float | 否 | 核采样阈值 |
| `top_k` | integer | 否 | 采样候选集大小 |
| `metadata` | object | 否 | 自定义键值对（最多 16 个，键≤64 字符，值≤512 字符） |

### Run 创建参数

| 参数 | 类型 | 必须 | 说明 |
|------|------|:----:|------|
| `thread_id` | string | 是 | 目标线程 ID（URL 路径传入） |
| `assistant_id` | string | 是 | 目标智能体 ID |
| `stream` | boolean | 否 | 是否流式返回 |
| `model` / `instructions` / `tools` | - | 否 | 覆盖 Assistant 中定义的对应值 |

### 通用参数

所有接口均支持 `api_key` 和 `workspace`（子业务空间场景）参数。建议通过环境变量 `DASHSCOPE_API_KEY` 配置 [[api-key]]。

## 使用方式

### HTTP 调用

所有接口基于 `https://dashscope.aliyuncs.com/api/v1/` 前缀，认证方式为 `Authorization: Bearer $DASHSCOPE_API_KEY`。

### SDK 调用

- **Python SDK**：需 `dashscope >= 1.18.0`，通过 `pip install -U dashscope` 更新。核心模块包括 `Assistants`、`Threads`、`Messages`、`Runs`、`Steps`。
- **Java SDK**：需 `dashscope >= 2.14.2`。核心类位于 `com.alibaba.dashscope.assistants` 和 `com.alibaba.dashscope.threads` 包下。

### [[streaming|流式输出]]

通过在 Run 创建时设置 `stream=True`，可实时获取事件流。事件流由 `event`（事件名）和 `data`（事件数据）组成，涵盖线程创建、运行状态变更、消息增量等完整生命周期。流式相关的增量对象包括：

- **消息增量对象**（`thread.message.delta`）：大模型生成的文本片段
- **运行步骤增量对象**（`thread.run.step.delta`）：工具调用返回的结果片段

详细的事件列表和增量对象结构见 [Assistant API [[streaming|流式输出]]参数说明（下线中）](../../raw/application-api-reference/assistantapi/event-streaming.md)。

### 函数调用（Function Calling）

当 Run 状态变为 `requires_action` 时，表示模型请求调用自定义函数。开发者需要：

1. 从 Run 对象的 `required_action.submit_tool_outputs.tool_calls` 获取函数名和参数
2. 在本地执行函数
3. 通过 `Runs.submit_tool_outputs` 提交结果
4. 继续等待 Run 完成

## 数据持久化与生命周期

- 所有 Assistant、Thread 实例均保存在阿里云百炼服务器上，**目前没有失效日期**。
- 可通过 `assistant.id` 或 `thread.id` 随时检索。
- Run 状态包括：`queued` → `in_progress` → `completed` / `failed` / `cancelled` / `expired` / `requires_action`。

## 限制和注意事项

- **下线状态**：Assistant API 正在下线，建议尽快迁移至 [[responses-api]]。
- **Message 角色限制**：创建消息时目前仅支持 `role="user"`。
- **智能体应用与 Assistant 的区别**：百炼控制台中的"智能体应用"与 Assistant API 创建的 Assistant 功能相互独立，不可混用。智能体应用通过控制台管理并使用 [[agent-application-api]] 调用，Assistant 仅通过 Assistant API 管理和调用。
- **错误处理**：调用失败时请参考 [[error-code]] 进行排查。
- **Run Steps** 中的时间戳为 Unix 13 位毫秒级时间戳，详见 [Run Steps（下线中）](../../raw/application-api-reference/assistantapi/run-steps.md)。

## 来源文档

- [Threads（下线中）](../../raw/application-api-reference/assistantapi/thread.md)
- [Assistants（下线中）](../../raw/application-api-reference/assistantapi/assistant.md)
- [Run Steps（下线中）](../../raw/application-api-reference/assistantapi/run-steps.md)
- [Messages（下线中）](../../raw/application-api-reference/assistantapi/message.md)
- [Runs（下线中）](../../raw/application-api-reference/assistantapi/runs.md)
- [Assistant API 流式输出参数说明（下线中）](../../raw/application-api-reference/assistantapi/event-streaming.md)
- [Assistant API 调用示例（下线中）](../../raw/application-api-reference/assistantapi/call-example.md)

