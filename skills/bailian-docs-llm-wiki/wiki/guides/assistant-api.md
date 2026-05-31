# assistant api

Assistant API 是百炼平台提供的大模型应用开发接口，内置多轮对话管理和工具调用组件，帮助开发者快速构建个人助理、智能导购等应用。相比直接使用[[text-generation]] API，Assistant API 封装了上下文管理和流程控制，降低了开发复杂度。

> **注意**：Assistant API 当前处于**下线中**状态，建议迁移至 [[responses-api]]（Responses API），后者同样内置多种工具并支持多轮上下文管理，可作为替代方案。

## 核心概念与使用流程

根据 [Assistant API（下线中）](../../raw/application-user-guide/assistant-api.md) 的描述，构建一个 Assistant 应用通常需要四个步骤：

1. **创建 Assistant**：配置大模型、系统指令和工具列表，定义 Assistant 的能力边界。
2. **创建 Thread**：Thread 记录用户和 Assistant 之间的所有消息，实现多轮对话上下文管理。
3. **创建 Message**：Message 是承载每一轮用户输入或 Assistant 回复的容器。
4. **创建 Run**：Run 代表 Assistant 对当前对话的一次完整响应过程，包括模型推理和工具调用。支持启用[[streaming-output]]（[[streaming|流式输出]]）。

## 支持的模型

Assistant API 支持以下千问系列模型：

| 模型系列 | 模型标识符 |
|---------|-----------|
| 千问-Turbo | `qwen-turbo` |
| 千问-Plus | `qwen-plus` |
| 千问-Max | `qwen-max` |

> **注意**：千问-Turbo、千问-Plus、千问-Max 的快照版本（例如 `qwen-plus-1220`）仅兼容"函数调用"及"知识检索增强"工具，其他工具的兼容性以实际运行结果为准。

## 支持的工具

如 [Assistant API（下线中）](../../raw/application-user-guide/assistant-api.md) 所列，支持以下工具：

| 工具 | 标识符 | 用途 |
|------|--------|------|
| 代码解释器 | `code_interpreter` | 执行 Python 代码，适用于编程、数学计算、数据分析 |
| 夸克搜索 | `quark_search` | 实时检索网络信息 |
| 文生图 | `text_to_image` | 将文字描述转为图像 |
| 计算器 | `calculator` | 精确数学运算 |
| 生成二维码 | `generate_qrcode` | 文本转二维码 |
| GitHub搜索 | `github_search` | 搜索 GitHub 项目实时信息 |
| 函数调用 | `function` | 本地设备执行自定义功能（[[function-calling]]） |
| 知识检索增强 | `rag` | 检索外部知识，增强回答准确性 |
| 自定义插件 | `${plugin_id}` | 连接自定义业务接口 |

## 关键能力

- **内置对话管理**：Thread 自动维护对话历史，开发者无需手动拼接上下文。
- **工具调用流程**：Run 执行过程中可能触发 `thread.run.requires_action` 事件，此时需要提交工具输出（`submit_tool_outputs`）后继续执行。
- **多智能体编排**：可基于 Assistant 的封装快速搭建 Multi Agent 系统，通过规划 Agent 分配任务顺序，依次调用不同 Assistant 处理。

## 限制和注意事项

- Assistant API 正在下线，不建议新项目使用。迁移方案参见 [[responses-api]]。
- [[agent-application]]（智能体应用）与 Assistant 功能相互独立：智能体应用通过控制台创建并使用应用调用 API 调用，Assistant 仅通过 Assistant API 操作。
- 自定义插件的兼容性以实际执行结果为准。
- [[streaming|流式输出]]通过 Run 的 `stream=True` 参数启用，事件类型包括 `thread.message.delta`、`thread.message.completed`、`thread.run.step.delta`、`thread.run.completed` 等。

更多细节请参考 [Assistant API（下线中）](../../raw/application-user-guide/assistant-api.md) 原始文档。

## 来源文档

- [Assistant API（下线中）](../../raw/application-user-guide/assistant-api.md)

