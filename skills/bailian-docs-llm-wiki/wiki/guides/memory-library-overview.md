# memory library overview

记忆库是百炼平台提供的长期记忆解决方案，用于解决大模型因上下文窗口限制而无法跨会话保留信息的问题。它通过自动从对话中提取关键信息并持久化存储，在后续对话中基于语义检索相关记忆并注入上下文，使智能体能够持续理解用户偏好和历史信息。记忆库提供开放的 API 接口，可接入任意应用，也支持多应用共享同一记忆库。

## 核心功能

记忆库支持两种记忆内容类型，详见 [记忆库](../../raw/application-user-guide/memory-library-overview/memory-library.md)：

- **记忆片段**：从对话中自动提取关键事件和信息（如"用户每天上午9点需要喝水提醒"），支持语义检索和动态更新，也可直接指定要存入的记忆内容。适用于大多数长期记忆场景。
- **用户画像**：基于自定义画像模板，从对话中提取结构化用户属性（如年龄、职业、兴趣等）。适用于需要持久化存储固定属性的场景。

> **注意**：生成的记忆片段与用户画像暂无失效日期（通过 API 创建时），但通过控制台创建的默认记忆片段规则有效期为 180 天，可配置为 7 天、30 天、180 天或永不过期。

## 使用方式

### 通过 API 使用

根据 [长期记忆 API](../../raw/application-user-guide/memory-library-overview/long-term-memory-2-0.md)，核心 API 操作包括：

| 操作 | API | 说明 |
|------|-----|------|
| 写入记忆 | `AddMemory` | 保存对话内容，自动提取记忆片段并构建语义索引 |
| 检索记忆 | `SearchMemory` | 基于语义检索相关历史记忆 |
| 列出记忆 | `ListMemory` | 分页列出指定用户的所有记忆 |
| 更新记忆 | `UpdateMemory` | 更新指定记忆片段内容 |
| 删除记忆 | `DeleteMemory` | 删除指定记忆片段 |
| 创建画像模板 | `CreateProfileSchema` | 定义需要提取的用户属性字段 |
| 获取用户画像 | `GetUserProfile` | 获取完整的用户画像信息 |

**使用流程：**

1. 配置环境变量 `DASHSCOPE_API_KEY`，获取方式参见 [[get-api-key]]。
2. 创建或使用默认记忆库。
3. 每轮对话结束后调用 `AddMemory` 写入记忆。
4. 调用 `SearchMemory` 检索相关记忆，将结果注入 Prompt 实现个性化回答。

**关键参数：**

- `user_id`（必填）：用户标识符，用于隔离不同用户的记忆空间。
- `memory_library_id`（非必填）：记忆库 ID，不填则使用默认记忆库。
- `project_id`（非必填）：记忆片段规则 ID。
- `profile_schema`（非必填）：用户画像规则 ID，传入后 `AddMemory` 会同时提取用户画像。
- `top_k`：检索时返回的记忆条数，建议设置在 3~10 之间。
- `meta_data`（非必填）：自定义元数据，用于对记忆进行分类管理。

### 通过控制台使用

在百炼控制台的记忆库页面，可以：

- 创建和管理记忆库，配置记忆片段规则和用户画像规则。
- 查看记忆详情，按 `user_id` 筛选记忆实体。
- 调试记忆检索效果，配置意图判别召回、查询改写、排序等高级检索参数。

每个账号自带一个默认记忆库，无需额外创建即可使用。每个记忆库最多可配置 50 条记忆片段规则和 50 条用户画像规则。

### 通过 OpenClaw 插件使用

根据 [为 OpenClaw 配置长期记忆插件](../../raw/application-user-guide/memory-library-overview/modelstudio-memory-for-openclaw.md)，可通过安装 `@modelstudio/modelstudio-memory-for-openclaw` 插件，为 OpenClaw Agent 赋能跨会话记忆能力。插件通过生命周期钩子自动完成记忆的捕获与召回：

- **自动捕获**（`autoCapture`）：对话结束后自动提取关键信息存储。
- **自动召回**（`autoRecall`）：对话开始前自动检索相关记忆注入上下文。

插件还向 Agent 注册了 `memory_search`、`memory_store`、`memory_list`、`memory_forget` 四个工具，Agent 可在对话中主动调用。

> **注意**：记忆插件为统一配置，所有 Agent 共享同一记忆，暂不支持按 Agent 独立配置。且不支持配置百炼 Coding Plan 的 API Key。

## 最佳实践

- **及时写入**：在每轮对话结束后及时调用 `AddMemory` 保存记忆。
- **合理设置 top_k**：检索时建议将 `top_k` 设置在 3~10 之间，平衡性能和效果。
- **善用元数据**：使用 `meta_data` 对记忆进行分类管理（如按类别、优先级等），便于精确检索。
- **画像字段设计**：属性名称应语义唯一，避免出现"姓名/名称/名字"、"年龄/年纪/岁数"等重复语义字段。不应期望一次对话提取所有信息，应通过多轮对话逐步收集。
- **检索优化**：开启意图判别召回可避免无关检索；对口语化提问开启查询改写可提升准确率；排序相似度阈值建议设在 0.5~0.7 之间。

## 配额与限制

| API 接口 | 速率上限（阿里云账号级别） |
|----------|--------------------------|
| 所有接口合计 | 3000 QPM |
| AddMemory（写入） | 120 QPM |
| SearchMemory（查询） | 300 QPM |

**性能指标**（来自 OpenClaw 插件文档）：

- SearchMemory 端到端延迟：200–500ms
- AddMemory 延迟：500–1000ms
- 自动捕获异步执行，不影响响应速度

**Python SDK**：使用 `agentscope-runtime` 包（安装命令：`pip install agentscope-runtime`）。

## 相关概念

- [[long-term-memory-2-0]] — 长期记忆 API 完整参考
- [[memory-library]] — 控制台记忆库管理
- [[modelstudio-memory-for-openclaw]] — OpenClaw 长期记忆插件配置
- [[get-api-key]] — API Key 获取与配置

## 来源文档

- [长期记忆 API](../../raw/application-user-guide/memory-library-overview/long-term-memory-2-0.md)
- [为 OpenClaw 配置长期记忆插件](../../raw/application-user-guide/memory-library-overview/modelstudio-memory-for-openclaw.md)
- [记忆库](../../raw/application-user-guide/memory-library-overview/memory-library.md)

