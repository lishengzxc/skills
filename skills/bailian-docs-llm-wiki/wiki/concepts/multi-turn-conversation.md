# 多轮对话与上下文管理

多轮对话与上下文管理是指在用户与大模型的连续交互中，维护和传递对话历史信息的机制，使模型能够理解前文语境并给出连贯回复。在百炼平台中，上下文管理贯穿应用配置、API 调用和长期记忆三个层面，不同场景提供了不同的实现方式。

## 会话内上下文：API 层面的实现

百炼平台的两套应用调用 API 采用不同的上下文管理策略：

| 维度 | DashScope API | OpenAI Responses API |
|------|--------------|----------------------|
| 管理方式 | **服务端管理**：通过 `session_id` | **客户端管理**：通过 `input` 消息数组 |
| 首次请求 | 不传 `session_id`，响应返回新 ID | 传入初始消息 |
| 后续请求 | 携带上次响应返回的 `session_id` | 将完整对话历史追加到消息数组中 |
| 有效期 | 最后一次请求后 **1 小时**自动失效 | 无限制，由客户端自行维护 |

**DashScope API 示例（Python）：**

```python
from dashscope import Application
import os

# 首次请求，不传 session_id
response = Application.call(
    api_key=os.getenv("DASHSCOPE_API_KEY"),
    app_id='APP_ID',
    [[prompt|prompt]]='你好，我叫小明'
)
session_id = response.output.session_id  # 获取 session_id

# 后续请求，携带 session_id 维持上下文
response = Application.call(
    api_key=os.getenv("DASHSCOPE_API_KEY"),
    app_id='APP_ID',
    [[prompt|prompt]]='我叫什么名字？',
    session_id=session_id
)
```

> **注意**：Responses API 中基于 `pre_response_id` 或 `conversation_id` 的上下文功能**尚未支持**，目前需在每次请求时传递完整对话历史。

详见 [[application-call]]。

## Assistant API 中的 Thread 模型

[[assistant-api]] 通过 **Thread** 对象自动维护对话上下文，开发者无需手动拼接历史消息：

- **Thread**：对话线程，承载用户与 Assistant 之间的所有消息记录。
- **Message**：线程中的单条消息，包含角色（user/assistant）和内容。
- **Run**：在指定 Thread 上执行一次完整的模型响应，自动读取 Thread 中的历史消息作为上下文。

Thread 实例保存在百炼服务器上，目前没有失效日期。这种设计将上下文管理完全封装在服务端，开发者只需创建 Thread 并不断追加 Message 即可。

> **注意**：Assistant API 当前处于**下线中**状态，建议迁移至 [[responses-api]]。

## 智能体应用中的短期记忆配置

在百炼控制台创建 [[llm-application]] 时，可通过**记忆**配置控制上下文窗口：

- **短期记忆**：支持设置 **0–30 轮**上下文传递。设为 0 表示不传递历史记录，每次请求视为独立对话。
- **工作流应用**中可通过**会话变量**在工作流全生命周期内持久化参数，并通过预置变量 `historyList` 访问对话历史。

## 跨会话上下文：长期记忆

大模型的上下文窗口有限，无法跨会话保留信息。百炼平台通过 [[memory-library-overview]] 提供长期记忆解决方案：

- **记忆片段**：从对话中自动提取关键事件和信息，支持语义检索，适用于大多数跨会话场景。
- **用户画像**：基于自定义模板提取结构化用户属性（如年龄、职业、兴趣）。

**典型使用流程：**

1. 每轮对话结束后调用 `AddMemory` 写入记忆。
2. 新对话开始时调用 `SearchMemory` 检索相关记忆。
3. 将检索结果注入 Prompt，实现个性化回答。

详见 [[long-term-memory-new]]。

## 关键参数汇总

| 参数 | 所属 API / 配置 | 说明 |
|------|----------------|------|
| `session_id` | DashScope 应用调用 API | 服务端会话标识，1 小时无活动后失效 |
| `input`（消息数组） | Responses API | 客户端维护的完整对话历史 |
| 短期记忆轮次（0–30） | 控制台智能体配置 | 控制传递给模型的历史对话轮数 |
| `user_id` | 长期记忆 API | 用户标识符，隔离不同用户的记忆空间 |
| `top_k` | SearchMemory API | 检索返回的记忆条数，建议 3–10 |
| `memory_library_id` | 长期记忆 API | 记忆库 ID，不传则使用默认记忆库 |

## 场景选型建议

| 场景 | 推荐方案 |
|------|---------|
| 简单多轮对话（应用调用） | DashScope API 的 `session_id`（零管理成本） |
| 需要精细控制对话历史 | Responses API 的消息数组（客户端完全控制） |
| 需要跨会话记住用户偏好 | 长期记忆 API（记忆片段 + 用户画像） |
| 单轮独立问答 |

## 关联主题页

- [[application-call|application call]] — `../api/application-call.md`
- [[assistant-api|assistant api]] — `../guides/assistant-api.md`
- [[assistantapi|assistantapi]] — `../api/[[assistantapi|assistantapi]].md`
- [[memory-library-overview|memory library overview]] — `../guides/memory-library-overview.md`
- [[long-term-memory-new|long term memory new]] — `../api/long-term-memory-new.md`
- [[llm-application|llm application]] — `../guides/llm-application.md`

