# use chat client or development tool

阿里云百炼平台兼容 OpenAI 和 Anthropic API 协议，开发者可通过多种主流 AI 编程工具、桌面客户端和开发平台接入百炼上的模型服务。本文汇总了各类工具的接入方式、计费方案选择、关键配置参数及常见限制，帮助开发者快速上手。

## 支持的工具总览

百炼支持的工具大致分为以下几类：

| 类别 | 工具 | 说明 |
|------|------|------|
| AI 编程 IDE | Cursor、Qoder、Qoder CN（原 Lingma） | 图形化 IDE，直接在编辑器中使用 AI |
| 终端 CLI 工具 | Claude Code、Codex、Hermes Agent、OpenCode、Qwen Code、Kilo CLI、OpenClaw | 命令行交互，适合终端工作流 |
| VSCode / JetBrains 插件 | Cline、Kilo Code、Qwen Code、Qoder 插件 | 在已有 IDE 中安装插件使用 |
| 桌面客户端 | Cherry Studio、Chatbox | 通用 AI 对话客户端 |
| 应用开发平台 | Dify | 工作流/Agent 编排平台 |
| API 测试 | Postman、cURL | 快速验证 API 调用 |

## 计费方案与接入凭证

百炼提供三种计费方案，不同方案使用**不同的 API Key 和 Base URL**，不可混用：

| 方案 | 计费模式 | API Key 获取 |
|------|----------|-------------|
| **Token Plan 团队版** | 按坐席订阅，按 token 消耗抵扣 Credits | [Token Plan 控制台](https://bailian.console.aliyun.com/?tab=plan#/efm/subscription/overview) |
| **Coding Plan** | 固定月费，按模型调用次数计量 | [Coding Plan 控制台](https://bailian.console.aliyun.com/cn-beijing/?tab=model#/efm/coding_plan) |
| **按量计费** | 按实际调用量后付费 | [百炼 API Key](https://help.aliyun.com/zh/model-studio/get-api-key) |

## 关键配置参数：Base URL

所有工具接入的核心配置都是 **API Key** + **Base URL** + **模型名称**。Base URL 因计费方案、API 协议和地域不同而异。详细的各方案 Base URL 汇总参见 [更多工具](../../raw/model-user-guide/use-chat-client-or-development-tool/[[more|more]]-tools.md)。

### OpenAI 兼容协议

| 方案 | Base URL |
|------|----------|
| Token Plan 团队版 | `https://token-plan.cn-beijing.maas.aliyuncs.com/compatible-mode/v1` |
| Coding Plan | `https://coding.dashscope.aliyuncs.com/v1` |
| 按量计费（北京） | `https://dashscope.aliyuncs.com/compatible-mode/v1` |
| 按量计费（新加坡） | `https://dashscope-intl.aliyuncs.com/compatible-mode/v1` |
| 按量计费（弗吉尼亚） | `https://dashscope-us.aliyuncs.com/compatible-mode/v1` |

### Anthropic 兼容协议

| 方案 | Base URL |
|------|----------|
| Token Plan 团队版 | `https://token-plan.cn-beijing.maas.aliyuncs.com/apps/anthropic` |
| Coding Plan | `https://coding.dashscope.aliyuncs.com/apps/anthropic` |
| 按量计费（北京） | `https://dashscope.aliyuncs.com/apps/anthropic` |
| 按量计费（新加坡） | `https://dashscope-intl.aliyuncs.com/apps/anthropic` |

## 各工具配置方式概要

### 终端 CLI 工具

**Claude Code** 通过 `~/.claude/settings.json` 中的环境变量（`ANTHROPIC_BASE_URL`、`ANTHROPIC_AUTH_TOKEN`）配置，使用 Anthropic 协议。详细配置及 CC Switch GUI 切换工具的用法参见 [Claude Code](../../raw/model-user-guide/use-chat-client-or-development-tool/claude-code.md)。

**Codex** 通过 `~/.codex/config.toml` 配置 `base_url` 和 `wire_api`，并通过环境变量 `OPENAI_API_KEY` 传入密钥。其中 qwen3.7-max 和 qwen3.6-plus 支持 Responses API（最新版 Codex），其他模型需通过 Chat/Completions API 接入（需安装 Codex 0.80.0 等旧版本）。

**Hermes Agent** 通过 `hermes config set` 命令或编辑 `~/.hermes/config.yaml` 配置。注意 `model.provider` 必须设置为 `custom`，否则会默认连接 OpenRouter。

**Qwen Code** 提供可视化 `/auth` 命令配置，也支持通过 `settings.json` 文件手动配置，是接入百炼最便捷的终端工具之一。

**OpenCode** 和 **Kilo CLI** 均通过 JSON 配置文件接入，配置结构类似，支持为每个模型单独设置 thinking 参数。

**OpenClaw** 是个人 AI 助手平台，配置文件为 `~/.openclaw/openclaw.json`，使用 Anthropic 协议接入。参见 [OpenClaw](../../raw/model-user-guide/use-chat-client-or-development-tool/openclaw.md)。

### IDE 与插件

**Cursor** 在 Settings > Models 中开启 OpenAI API Key 和 Override OpenAI Base URL 进行配置。

> **注意**：Cursor 免费版仅支持 Auto 模式，不支持调用自定义模型，需升级至 Cursor Pro 及以上。部分模型名称与 Cursor 内置模型冲突，需使用别名（如 `kimi-k2.6` → `kimi-k2-6`，`glm-5` → `glm-5-0`）。

**Cline**（VSCode 插件）选择 OpenAI Compatible 作为 API Provider，填入 Base URL、API Key 和 Model ID。使用 Qwen3 思考模式或 QwQ 模型时需勾选 **Enable R1 messages format**。

**Qoder** 和 **Qoder CN（原 Lingma）** 提供原生的百炼集成，直接在设置中选择"阿里云百炼 - 国内"作为提供商。Qoder CN 企业版不支持接入百炼。

### 桌面客户端

**Cherry Studio** 和 **Chatbox** 均支持 OpenAI API 兼容模式，在设置中添加模型供应商，填入 API Key 和对应 Base URL 即可。

### 开发平台

**Dify** 通过安装"通义千问"插件（Dify 官方维护）接入百炼模型，包括 DeepSeek 等模型也使用该插件。Dify 支持聊天助手、Agent、Chatflow/工作流、知识库等多种应用类型。万相模型需通过工作流 HTTP 节点接入。

### API 测试

使用 **Postman** 或 **cURL** 可快速验证 API 调用。图像/视频生成 API 采用异步调用机制：先创建任务获取 `task_id`，再轮询查询结果。详见 [使用Postman或cURL调用图像/视频生成API](../../raw/model-user-guide/use-chat-client-or-development-tool/first-call-to-image-and-video-api.md)。

## 限制与注意事项

### 工具使用限制

- **Token Plan 团队版** 和 **Coding Plan** 仅限在 AI 编程工具和 OpenClaw 类 Agent 中使用，**不支持**工作流平台（如 Dify、n8n、Coze）、API 测试工具（Postman）或自定义应用程序调用。违规使用可能导致订阅暂停或 API Key 封禁。
- 部分工具（如 Cursor 免费版、Trae、通义灵码个人版）不支持自定义服务端点，无法直接使用 Token Plan 或 Coding Plan。
- [[coding-plan]] 和 [[token-plan]] 的 API Key 互不相通，与按量计费的 API Key 也不可混用。

### 模型名称兼容性

部分工具对模型名称中的 `.` 有限制，需替换为 `-`。例如在 Cursor 中：
- `kimi-k2.6` → `kimi-k2-6`
- `glm-5.1` → `glm-5-1`
- `glm-5` → `glm-5-0`

### 思考模式

使用支持思考模式的模型（如 Qwen3 系列、Kimi K2 系列、GLM-5 系列）时，不同工具的启用方式不同：
- OpenCode / Kilo CLI：在配置中设置 `thinking.type: "enabled"` 和 `budgetTokens`
- Qwen Code：通过 `extra_body.enable_thinking: true` 启用
- Cline：勾选 **Enable R1 messages format**
- Cherry Studio：在客户端中开启思考模式

### 地域与免费额度

- 按量计费的 API Key 需与 Base URL 的地域对应（北京/新加坡/弗吉尼亚）
- 免费额度仅适用于中国内地版（北京地域），各模型独立计算
- 控制台的额度数据每小时更新，可能存在延迟

### 常见错误排查

| 错误 | 可能原因 |
|------|----------|
| 401 Unauthorized / Incorrect API key | API Key 与 Base URL 不匹配，或 Key 与地域不对应 |
| 400 InvalidParameter | 未启用思考模式（部分模型强制要求） |
| Model does not work with your plan | Cursor 免费版不支持自定义模型 |
| 连接到非预期的提供商 | 工具默认提供商未正确覆盖（如 Hermes Agent 需设置 `provider: custom`） |

错误码详细排查：[[error-code]]、[[coding-plan-faq]]、[[token-plan-faq]]

## 来源文档

- [OpenClaw](../../raw/model-user-guide/use-chat-client-or-development-tool/openclaw.md)
- [Hermes Agent](../../raw/model-user-guide/use-chat-client-or-development-tool/hermes-agent.md)
- [OpenCode](../../raw/model-user-guide/use-chat-client-or-development-tool/opencode.md)
- [Claude Code](../../raw/model-user-guide/use-chat-client-or-development-tool/claude-code.md)
- [Cursor](../../raw/model-user-guide/use-chat-client-or-development-tool/cursor.md)
- [Codex](../../raw/model-user-guide/use-chat-client-or-development-tool/codex.md)
- [Cherry Studio](../../raw/model-user-guide/use-chat-client-or-development-tool/cherry-studio.md)
- [Qwen Code](../../raw/model-user-guide/use-chat-client-or-development-tool/qwen-code.md)
- [Chatbox](../../raw/model-user-guide/use-chat-client-or-development-tool/chatbox.md)
- [Cline](../../raw/model-user-guide/use-chat-client-or-development-tool/cline.md)
- [Qoder](../../raw/model-user-guide/use-chat-client-or-development-tool/qoder-agent.md)
- [Kilo CLI](../../raw/model-user-guide/use-chat-client-or-development-tool/kilo-cli.md)
- [Qoder CN（原 Lingma）](../../raw/model-user-guide/use-chat-client-or-development-tool/lingma-agent.md)
- [使用Postman或cURL调用图像/视频生成API](../../raw/model-user-guide/use-chat-client-or-development-tool/first-call-to-image-and-video-api.md)
- [Dify](../../raw/model-user-guide/use-chat-client-or-development-tool/dify.md)
- [更多工具](../../raw/model-user-guide/use-chat-client-or-development-tool/[[more|more]]-tools.md)

