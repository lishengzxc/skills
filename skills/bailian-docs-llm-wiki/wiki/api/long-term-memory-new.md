# long term memory new

长期记忆（新）是百炼平台提供的记忆管理服务，支持将用户对话自动提取为结构化记忆片段，并基于语义相似度进行检索。该功能通过 REST API 提供完整的 CRUD 操作及用户画像管理能力，适用于需要跨会话保持用户上下文的智能体应用场景。

详细的接口定义和参数说明请参考 [长期记忆（新）API 参考](../../raw/application-api-reference/long-term-memory-new/long-term-memory-api-reference.md)。

## 公共请求信息

| 参数 | 说明 |
|------|------|
| Base URL | `https://dashscope.aliyuncs.com/api/v2/apps/memory/` |
| 认证方式 | Header 中添加 `Authorization: Bearer $DASHSCOPE_API_KEY`，获取方式参见 [[get-api-key]] |
| Content-Type | `application/json` |

## 接口概览

长期记忆（新）共提供 11 个 API 接口，分为**记忆片段管理**和**画像模板管理**两大类：

### 记忆片段管理

| 接口 | 方法 | 路径 | 说明 |
|------|------|------|------|
| AddMemory | POST | `/add` | 添加记忆片段 |
| SearchMemory | POST | `/memory_nodes/search` | 语义搜索记忆片段 |
| ListMemory | GET | `/memory_nodes` | 分页列出记忆片段 |
| DeleteMemory | DELETE | `/memory_nodes/{memory_node_id}` | 删除记忆片段 |
| UpdateMemory | PATCH | `/memory_nodes/{memory_node_id}` | 更新记忆片段 |

### 画像模板管理

| 接口 | 方法 | 路径 | 说明 |
|------|------|------|------|
| CreateProfileSchema | POST | `/profile_schemas` | 创建画像模板 |
| ListProfileSchemas | GET | `/profile_schemas` | 获取画像模板列表 |
| DeleteProfileSchema | DELETE | `/profile_schemas/{profile_schema_id}` | 删除画像模板 |
| UpdateProfileSchema | PATCH | `/profile_schemas/{profile_schema_id}` | 更新画像模板 |
| GetProfileSchema | GET | `/profile_schemas/{profile_schema_id}` | 获取画像模板详情 |
| GetUserProfile | GET | `/profile_schemas/{profile_schema_id}/user_profile` | 获取用户画像 |

## 核心接口详解

### AddMemory - 添加记忆片段

将用户对话存储为记忆片段，系统会自动提取关键信息。支持两种输入方式（互斥）：

- **messages**：传入对话消息列表（最多 50 条），每条包含 `role`（user/assistant）和 `content`。
- **custom_content**：传入自定义内容字符串，最大 512 字符。

关键参数：

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `user_id` | string | 是 | 记忆实体 ID，最大 64 字符 |
| `messages` | array | 与 `custom_content` 互斥 | 对话消息列表 |
| `custom_content` | string | 与 `messages` 互斥 | 自定义内容 |
| `profile_schema` | string | 否 | 画像模板 ID |
| `memory_library_id` | string | 否 | 记忆库 ID，不传则使用默认记忆库 |
| `project_id` | string | 否 | 记忆片段规则 ID |
| `meta_data` | object | 否 | 用户自定义元信息 |

返回的 `memory_nodes` 数组中，每个节点包含 `event` 字段标识操作类型：`ADD`（创建）、`UPDATE`（更新）、`DELETE`（删除）。

### SearchMemory - 搜索记忆片段

基于语义相似度搜索记忆片段，支持多种增强选项。据 [长期记忆（新）API 参考](../../raw/application-api-reference/long-term-memory-new/long-term-memory-api-reference.md) 描述，搜索支持以下可选能力：

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `top_k` | integer | 10 | 最大召回数，范围 1~100 |
| `min_score` | double | 0.3 | 最小相似度阈值，范围 [0,1] |
| `enable_rerank` | boolean | false | 是否开启重排序 |
| `enable_judge` | boolean | false | 是否开启意图判别回调 |
| `enable_rewrite` | boolean | false | 是否开启 query 重写 |
| `project_ids` | list | - | 支持传入多个记忆片段规则 ID 进行混合检索 |

### ListMemory / DeleteMemory / UpdateMemory

- **ListMemory**：GET 请求，支持 `page_num` 和 `page_size` 分页参数。
- **DeleteMemory**：DELETE 请求，通过路径参数 `memory_node_id` 指定要删除的片段。
- **UpdateMemory**：PATCH 请求，通过 `custom_content` 更新内容（最大 512 字符），`meta_data` 为增量更新。

## 使用方式

### cURL 调用

直接通过 HTTP 请求调用，示例：

```bash
curl -X POST https://dashscope.aliyuncs.com/api/v2/apps/memory/add \
  --header "Authorization: Bearer $DASHSCOPE_API_KEY" \
  --header 'Content-Type: application/json' \
  --data '{
    "messages": [
      {"role": "user", "content": "每天上午11点提醒我点外卖。"},
      {"role": "assistant", "content": "没问题"}
    ],
    "user_id": "user_001",
    "memory_library_id": "xxx"
  }'
```

### Python SDK

需安装 `agentscope-runtime`（版本 ≥ 1.1.5）：

```bash
pip install agentscope-runtime>=1.1.5
```

SDK 提供 `AddMemory`、`SearchMemory`、`ListMemory`、`DeleteMemory` 等异步封装类，均通过 `arun()` 方法调用。

> **注意**：根据 [长期记忆（新）API 参考](../../raw/application-api-reference/long-term-memory-new/long-term-memory-api-reference.md)，`UpdateMemory` 接口的 Python SDK 封装暂未提供，需通过 `requests` 库直接调用 REST API。

## 限制和注意事项

### 限流（阿里云账号级别）

| API 接口 | 限流 |
|----------|------|
| 全部接口合计 | 3000 QPM |
| AddMemory（add） | 120 QPM |
| SearchMemory（search） | 300 QPM |

### 其他限制

- `user_id` 最大 64 字符，`memory_library_id` 最大 32 字符。
- `messages` 最多支持 50 条对话记录（一问一答算 2 条）。
- `custom_content` 最大 512 字符。
- 生成的记忆片段与用户画像**暂无失效日期**。
- 不传 `memory_library_id` 时，系统自动使用默认记忆库；不传 `project_id` 时，自动使用该记忆库的默认记忆片段规则。
- 记忆库 ID 可在 [[bailian-console]] 的记忆库页面获取。

## 来源文档

- [长期记忆（新）API 参考](../../raw/application-api-reference/long-term-memory-new/long-term-memory-api-reference.md)

