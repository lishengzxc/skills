# 函数调用（工具调用）

函数调用（Function Calling），也称工具调用（Tool Calling），是指大模型在推理过程中根据用户输入自主判断是否需要调用外部函数或工具，并生成符合预定义格式的调用参数，由客户端执行后将结果返回模型以生成最终回答的机制。

## 核心原理

函数调用的本质是让大模型充当"决策者"：模型根据**用户输入**、**工具名称**和**工具描述**判断是否需要调用工具。当需要调用时，模型输出结构化的函数名和参数；客户端负责实际执行并将结果回传给模型，模型再基于工具返回结果生成最终响应。

典型流程：
1. 开发者在请求中定义可用工具（`tools` 参数）
2. 模型推理后决定是否调用工具，若调用则返回函数名和参数
3. 客户端执行对应函数，获取结果
4. 将工具输出回传给模型
5. 模型综合工具结果生成最终回答

## 在百炼平台的使用场景

### API 直接调用

百炼平台的多种接口协议均支持函数调用：

| 接口 | 函数调用支持方式 |
|------|-----------------|
| [[openai-chat-completions]] | 通过 `tools` 参数定义函数，模型返回 `tool_calls` |
| [[openai-responses]] | 内置工具 + 自定义函数调用 |
| [[anthropic-messages]] | 兼容 Anthropic 工具调用格式 |
| [[dashscope]] | 百炼原生接口，参数支持最全面 |

### 智能体应用中的工具调用

在 [[single-agent-application]] 中，函数调用以更高层次的形式存在：
- **插件工具**：通过 [[plug-in]] 机制，智能体最多添加 10 个工具，模型自主决定调用时机
- **MCP 工具**：通过 [[mcp]] 协议接入外部工具，智能体在多步推理中动态调用
- 新版智能体（Agent 2.0）将知识库、MCP 统一为工具，支持完整的"规划-执行-反思"链路

### Assistant API 中的函数调用

在 [[assistant-api]] 中，函数调用的工具标识符为 `function`。当 Run 执行过程中触发 `thread.run.requires_action` 事件时，开发者需通过 `submit_tool_outputs` 提交工具执行结果后继续流程。

### 实时多模态交互中的工具调用

在 [[omni-realtime-api]] 中，通过 WebSocket 协议实现实时工具调用：
1. 通过 `session.update` 的 `tools` 字段定义可用工具
2. 模型自主判断是否调用
3. 触发调用后，客户端执行函数并通过 `conversation.item.create` 事件回传结果

> **注意**：实时 API 中 `tools` 和 `enable_search` 不兼容，不可同时开启。

## 关键参数和配置

### 工具定义格式（OpenAI 兼容）

```json
{
  "tools": [
    {
      "type": "function",
      "function": {
        "name": "get_weather",
        "description": "获取指定城市的天气信息",
        "parameters": {
          "type": "object",
          "properties": {
            "city": {
              "type": "string",
              "description": "城市名称"
            }
          },
          "required": ["city"]
        }
      }
    }
  ]
}
```

### 关键字段说明

| 字段 | 说明 | 最佳实践 |
|------|------|---------|
| `name` | 函数名称，模型据此选择工具 | 使用语义明确的命名 |
| `description` | 函数功能描述 | 详细描述功能和使用场景，直接影响模型调用准确性 |
| `parameters` | JSON Schema 格式的参数定义 | 为每个参数提供 description |

### 模型选择建议

推荐选用具备强工具调用能力的模型（如 `qwen-max` 系列、`qwen-plus` 系列）。若调用失败率较高，可尝试：
- 优化工具的 `description`，明确工具名称和能力边界
- 更换更强的推理模型（如 Qwen3 系列）
- 在系统提示词中引导工具使用时机

## 函数调用 vs 插件 vs MCP

| 维度 | 函数调用（API 层） | 插件（Plugin） | MCP |
|------|-------------------|---------------|-----|
| 执行方 | 客户端本地执行 | 百炼平台远程执行 | 云端 MCP 服务执行 |
| 配置方式 | 代码中定义 `tools` | 控制台配置或 API 传入 | 协议标准化接入 |
| 适用场景 | 需要访问本地资源或自定义逻辑 | 使用平台预置或自定义 API | 接入第三方标准化工具 |
| 开发复杂度 | 需自行处理调用流程 | 平台托管执行 | 平台托管执行 |

## 限制和注意事项

- 工具的 `name` 和 `description` 质量直接影响模型的调用准确率
- 单次请求中定义过多工具会增加 Token 消耗并可能降低选择准确性
- 模型可能在不需要工

## 关联主题页

- [[qwen-api-reference|qwen api reference]] — `../api/qwen-api-reference.md`
- [[omni-realtime-api|omni realtime api]] — `../api/omni-realtime-api.md`
- [[plug-in|plug in]] — `../guides/plug-in.md`
- [[model-context-protocol|model context protocol]] — `../guides/model-context-protocol.md`
- [[llm-application|llm application]] — `../guides/llm-application.md`
- [[assistant-api|assistant api]] — `../guides/assistant-api.md`
- [[toolkits-and-[[frameworks|frameworks]]|toolkits and frameworks]] — `../api/toolkits-and-[[frameworks|frameworks]].md`
- [[more-about-models|[[more|more]] about models]] — `../api/[[more|more]]-about-models.md`


