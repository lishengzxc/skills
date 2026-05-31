# 函数调用（Function Calling）

函数调用（Function Calling）是大模型与外部工具交互的核心机制，允许模型根据用户输入和工具定义，自主判断是否需要调用外部函数，并生成符合规范的调用参数，从而弥补大模型在实时信息获取、精确计算、外部系统操作等方面的不足。

## 概述

函数调用的基本流程为：

1. 开发者在请求中定义一组可用工具（函数名称、描述、参数 schema）
2. 模型根据用户输入内容、工具名称和工具描述判断是否需要调用工具
3. 若需要调用，模型输出工具名称和结构化参数（而非直接执行）
4. 应用侧执行实际调用，将结果回传模型
5. 模型结合工具返回结果生成最终回复

若模型判断无需调用工具，则直接生成文本回复。

## 在百炼平台的使用场景

### 模型推理（直接 API 调用）

所有通用文本生成模型均支持 Function Calling，部分视觉理解模型（如 `qwen3.6-plus`）和全模态模型也支持。开发者通过 Chat Completions 等接口的 `tools` 参数定义可用函数，模型在生成过程中按需触发调用。

与 Function Calling 不同，百炼还提供**内置工具**（联网搜索、代码解释器等），无需额外定义即可使用，仅 Qwen3.6 和 Qwen3.5 系列的 plus/flash 版本支持。

### 智能体应用（Agent）

- **新版智能体（Agent 2.0）**：通过 MCP 协议统一接入外部工具，由智能体自主规划调用顺序，支持"规划-执行-反思"链路。可配置 ReAct 最大轮次（1-50）限制单次会话中工具调用的最大次数。
- **旧版智能体（Agent 1.0）**：通过插件机制调用工具，自定义插件超时限制为 5 秒。每个智能体应用最多支持 10 个工具。

模型根据用户输入内容、工具名称和工具描述判断是否需要调用工具。为确保多步规划效果，推荐选用具备强工具调用能力的模型（如千问-Max 系列）。

### 实时多模态交互（Omni Realtime API）

Qwen-Omni-Realtime API 基于 WebSocket 协议支持实时场景下的 Function Calling。通过 `session.update` 事件注册工具定义，模型在语音对话过程中可触发工具调用。相关事件：

| 事件 | 方向 | 说明 |
|------|------|------|
| `session.update`（含 tools 定义） | 客户端→服务端 | 注册可用工具 |
| `response.function_call_arguments.delta` | 服务端→客户端 | 增量返回工具调用参数 |
| `response.function_call_arguments.done` | 服务端→客户端 | 工具调用参数生成完毕 |
| `conversation.item.create` | 客户端→服务端 | 回传工具函数执行结果 |

仅 `qwen3.5-omni-realtime` 系列支持工具调用。

### 插件机制

百炼平台的插件本质上是对 Function Calling 的封装。官方插件、三方插件和自定义插件均通过工具描述让模型决策调用时机。自定义插件的输入参数支持**大模型识别**（模型从用户输入中提取）和**业务透传**（外部主动传入）两种传参方式。

## 关键参数和配置

### 工具定义（tools 参数）

通过 API 调用时，在请求体中传入 `tools` 数组，每个工具包含：

| 字段 | 说明 |
|------|------|
| `type` | 固定为 `"function"` |
| `function.name` | 函数名称，需具备语义以帮助模型理解功能 |
| `function.description` | 功能和使用场景描述，直接影响模型的调用判断 |
| `function.parameters` | JSON Schema 格式的参数定义 |

### 支持的 API 接口

| 接口类型 | 是否支持 Function Calling |
|---------|--------------------------|
| OpenAI 兼容 Chat Completions | ✅ |
| OpenAI 兼容 Responses | ✅（内置工具 + 自定义工具） |
| Anthropic 兼容 Messages | ✅ |
| DashScope 原生接口 | ✅（功能覆盖最全） |

### 支持的模型

- **文本生成**：所有通用文本模型（qwen3.7-max、qwen3.6-plus、qwen3.6-flash、qwen-plus、qwen-max 等）
- **视觉理解**：qwen3.6-plus（视觉模式）等
- **实时多模态**：qwen3.5-omni-realtime 系列

### 相关配置建议

| 配置项 | 建议 |
|--------|------|
| 函数描述 | 清晰描述功能和适用场景，直接影响模型调用准确率 |
| 参数 schema | 使用明确的类型和描述，必填字段标注 `required` |
| ReAct 最大轮次 | 智能体场景下设置合

## 关联主题页

- [model inference](../guides/model-inference.md)
- [omni realtime api](../api/omni-realtime-api.md)
- [plug in](../guides/plug-in.md)
- [llm application](../guides/llm-application.md)
- [qwen api reference](../api/qwen-api-reference.md)
- [toolkits and frameworks](../api/toolkits-and-frameworks.md)
- [more about models](../api/more-about-models.md)

