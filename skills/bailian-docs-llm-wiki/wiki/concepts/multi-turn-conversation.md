# 多轮对话管理

多轮对话管理是指在用户与大模型交互过程中，维护和传递对话上下文（历史消息），使模型能够理解前序对话内容并生成连贯回复的机制。百炼平台在不同接口和应用类型中提供了多种实现方式，开发者可根据场景灵活选择。

## 在百炼平台中的使用场景

### 智能体与工作流应用调用

通过 `Application.call` 接口调用智能体或工作流应用时，支持两种多轮对话模式：

| 模式 | 实现方式 | 特点 |
|------|----------|------|
| `session_id` 模式 | 服务端自动存储和加载对话历史 | 实现简单，无需客户端维护上下文 |
| `messages` 模式 | 客户端自行维护对话历史数组 | 控制灵活，适合需要裁剪或编辑上下文的场景 |

> 若请求中同时包含 `session_id` 和 `messages`，系统将优先使用 `messages`。

### DashScope API 与 Responses API 的差异

- **DashScope API**：通过 `session_id` 实现多轮对话，首次请求无需传入，后续使用响应返回的 `session_id` 即可。服务端自动维护上下文。
- **OpenAI 兼容 Responses API**：当前需要在每次请求的 `input` 中传递完整的消息历史（`pre_response_id` 和 `conversation_id` 功能后续支持）。

### Assistant API（已下线）

Assistant API 通过 Thread 机制自动维护对话历史。开发者创建 Thread 后，所有 Message 自动关联到该 Thread，无需手动管理上下文。该 API 已下线，建议迁移至 Responses API。

### 智能体应用中的短期记忆配置

在百炼控制台创建智能体应用时，可配置**短期记忆**轮次（0-30 轮），控制模型能回溯的历史对话深度。轮次越多，模型可参考的上下文越丰富，但也会占用更多 Token。

### 结合长期记忆实现跨会话延续

多轮对话管理通常限于单次会话内。若需跨会话保留用户偏好和历史信息，可结合记忆库实现长期记忆：每轮对话结束后调用 `AddMemory` 写入关键信息，下次会话开始时通过 `SearchMemory` 检索相关记忆并注入 Prompt。

## 关键参数与配置

### session_id 模式

| 参数 | 说明 |
|------|------|
| `session_id` | 会话 ID，首次请求不传，后续使用响应返回的值 |
| 有效期 | 最后一次请求后 **1 小时**自动过期 |
| 最大轮次 | 最多 **50 轮**对话 |

```python
from dashscope import Application

# 首次请求，不传 session_id
response = Application.call(app_id='YOUR_APP_ID', prompt='你好')
session_id = response.output.session_id

# 后续请求，携带 session_id
response = Application.call(app_id='YOUR_APP_ID', prompt='继续上个话题', session_id=session_id)
```

### messages 模式（推荐）

| 参数 | 说明 |
|------|------|
| `messages` | 对话历史数组，每条消息包含 `role`（user/assistant）和 `content` |

```python
from dashscope import Application

messages = [
    {"role": "user", "content": "我想去杭州旅游"},
    {"role": "assistant", "content": "杭州有很多景点，您计划几天的行程？"},
    {"role": "user", "content": "三天"}
]

response = Application.call(app_id='YOUR_APP_ID', messages=messages)
```

> **工作流应用注意事项**：使用 `messages` 模式时，需在大模型节点中配置提示词变量 `historyList` 并重新发布应用。

## 最佳实践

- **优先使用 `messages` 模式**：对上下文有完整控制权，便于裁剪无关内容、降低 Token 消耗。
- **合理控制上下文长度**：历史消息过多会占用模型上下文窗口，影响响应质量和成本。可根据业务需要只保留最近 N 轮或摘要。
- **[流式输出](streaming.md)与多轮对话结合**：设置 `stream=True` 可在多轮对话中实现逐字输出，提升用户体验。
- **长期信息用记忆库补充**：将需要跨会话保留的信息写入记忆库，避免在每次请求中携带过长的历史消息。

## 关联主题页

- [assistant api](../guides/assistant-api.md)
- [bailian application calling](../guides/bailian-application-calling.md)
- [memory library overview](../guides/memory-library-overview.md)
- [long term memory new](../api/long-term-memory-new.md)
- [llm application](../guides/llm-application.md)
- [application call](../api/application-call.md)

