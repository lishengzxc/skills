# qwen api reference

百炼平台提供多种接口协议用于调用通义千问系列文本生成模型。开发者可根据现有技术栈和业务需求选择最合适的接入方式，各接口在兼容性、功能覆盖度方面各有侧重。详见 [文本生成模型API参考](../../raw/model-api-reference/qwen-api-reference.md)。

## 支持的接口协议

根据 [文本生成模型API参考](../../raw/model-api-reference/qwen-api-reference.md)，百炼目前提供以下四种调用方式：

| 接口 | 适用场景 | 特点 |
|------|----------|------|
| OpenAI 兼容 Chat Completions | 迁移现有 OpenAI 应用、接入第三方工具 | 与 OpenAI 客户端库直接兼容，迁移成本最低 |
| OpenAI 兼容 Responses | 需要内置工具能力的场景 | 内置联网搜索、代码解释器、网页内容提取；自动管理对话历史 |
| Anthropic 兼容 Messages | 使用 Anthropic SDK 的项目 | 兼容 Anthropic Messages API，支持思考和工具调用 |
| DashScope | 需要完整功能集的场景 | 百炼原生接口，参数支持最全面 |

## 接口选择建议

- **已有 OpenAI 代码**：优先使用 [[openai-chat-completions]] 接口，改动最小。
- **需要内置工具（搜索/代码执行）**：使用 [[openai-responses]] 接口，无需手动维护多轮对话上下文。
- **已有 Anthropic 代码**：使用 [[anthropic-messages]] 接口，可直接复用现有 SDK。
- **需要平台独有功能或最新参数**：使用 [[dashscope]] 原生接口。

## 关键差异

如 [文本生成模型API参考](../../raw/model-api-reference/qwen-api-reference.md) 所述，DashScope 作为百炼原生接口提供"最完整的功能集和参数支持"。这意味着部分高级参数或新上线功能可能仅在 DashScope 接口率先可用，兼容接口会存在一定的功能滞后。

> **注意**：OpenAI 兼容 Responses 接口会自动管理对话历史，与 Chat Completions 接口的无状态设计不同。如果你的应用已自行维护上下文，切换到 Responses 接口时需注意避免重复传递历史消息。

## 使用方式

所有接口均需通过百炼平台获取 API Key 进行鉴权。具体的请求格式、参数说明和返回结构请参考各接口的独立文档。通用流程为：

1. 在百炼控制台创建 API Key
2. 根据选择的协议安装对应 SDK（OpenAI SDK / Anthropic SDK / DashScope SDK）
3. 将 base URL 指向百炼平台端点
4. 按照对应协议的规范构造请求

## 来源文档

- [文本生成模型API参考](../../raw/model-api-reference/qwen-api-reference.md)

