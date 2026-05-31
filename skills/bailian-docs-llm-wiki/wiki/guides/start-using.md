# start using

阿里云百炼平台提供了从零代码到全代码的多种应用构建方式，帮助开发者快速搭建基于大语言模型的智能应用。本页面汇总了平台入门所需的核心流程、关键功能模块以及最新功能动态，方便开发者快速上手。

## 核心应用构建流程

根据 [0代码构建私有知识问答应用](../../raw/application-user-guide/start-using/build-knowledge-base-qa-assistant-without-coding.md) 的指引，构建一个完整的知识问答应用主要包含三个步骤：

1. **创建智能体应用**：在 [应用管理](https://bailian.console.aliyun.com/?tab=app#/app-center) 页面创建空白应用，选择模型、设计 Prompt（System Prompt）、配置欢迎语和预设问题。
2. **构建知识库**：通过数据连接器上传知识文档，创建知识库并等待平台完成解析和切分。
3. **关联知识库并发布**：将知识库绑定到应用的"技能"中，测试验证后发布应用。

> **注意**：使用大模型会产生计费。百炼提供了限时免费额度，可在 [模型广场](https://bailian.console.aliyun.com/cn-beijing?tab=model#/model-market/all) 查询目标模型的免费额度详情。

## 支持的模型与功能模块

### 推荐模型

- **通用对话**：千问-Max（推荐用于智能体应用）、DeepSeek 系列
- **深度推理**：QwQ 系列（[[deep-thinking]]），先输出思考过程再给出回答
- **多模态**：qwen-vl-plus / qwen-vl-max，支持图文理解
- **Embedding**：text-embedding-v4（推荐）、text-embedding-v3

### 主要功能模块

根据 [应用功能动态](../../raw/application-user-guide/start-using/application-release-notes.md)，平台当前支持以下应用类型和关键能力：

| 功能模块 | 说明 |
|---------|------|
| [[single-agent-application]] | 支持知识库、MCP 工具、长期记忆、文件问答、音视频实时互动 |
| [[workflow-application]] | 支持批量节点、异步运行、多模态生成节点、Dify 工作流导入 |
| [[rich-code-application]] | 基于 Python 项目结构部署 AI 后端服务，内置运维与可观测能力 |
| [[rag-knowledge-base]] | 支持文档/数据/图片/音视频类型，支持智能切分、图文检索、权重设置 |
| [[mcp]] | 支持官方预置 MCP 服务和自定义 MCP 服务，可外部调用 |

## 关键参数与配置项

### Prompt 设计

System Prompt 用于定义应用的角色和任务，直接影响回答质量。建议明确角色定位和任务边界，例如：

```
你是一位XX领域的专家，任务是帮助用户完成XX。
```

可结合 [[prompt-sample-optimization]]（Prompt 样例库）使用 FewShot 方法提升回答准确性。

### 知识库配置

- **切分策略**：推荐使用"智能切分"，经评测对多数文档可获得最佳检索效果
- **检索参数**：可调整初步向量检索 TopK 和关键词检索 TopK，降低数值可减少排序模型 Token 消耗
- **多知识库权重**：关联多个知识库时，可按重要性设置权重，系统优先召回高权重知识库内容
- **检索配置**：可设置大模型回答范围、是否展示回答来源、多模态回复增强等

### 知识库计费

> **注意**：根据 [应用功能动态](../../raw/application-user-guide/start-using/application-release-notes.md)，阿里云百炼知识库自 2026 年 1 月 4 日起正式计费，费用由规格费用和模型调用费用两部分组成。支持后付费和资源包两种计费方式，详见 [[billing-for-knowledge-base]]。

## 使用方式

### 控制台（零代码）

适合快速验证和非技术用户，通过控制台可完成应用创建、知识库构建、调试和发布的全流程。

### API 调用

- **智能体应用调用**：参见 [[call-single-agent-application]]
- **工作流应用调用**：参见 [[invoke-workflow-application]]
- **Responses API**：兼容 OpenAI 接口规范，支持同步和异步两种模式
- **异步模式**：请求中设置 `background: true`，立即返回 Task ID，可在任务中心查询结果

### 应用发布渠道

支持通过微信公众号、钉钉 AI 机器人、音视频 SDK（H5/iOS/Android）等渠道分享应用，详见 [[share-an-application]]。

## 限制和注意事项

- 文档解析时间取决于文件大小，上传后通常需要 1~6 分钟完成解析
- 知识库创建后的切分解析通常需要 1~2 分钟
- QwQ 系列模型在智能体应用中不支持插件、流程和音视频交互能力
- 智能体编排应用已下线（2025 年 8 月），相关能力已整合至工作流应用
- 新版智能体应用（Agent 2.0，2025 年 12 月上线）将知识库和 MCP 统一为工具，由智能体自主规划调用

## 后续步骤

- 深入了解 Prompt 撰写与应用配置：[[single-agent-application]]
- 构建复杂业务流程：[[workflow-application]]
- 了解各应用类型的功能对比：[[application-introduction]]
- 全代码开发定制化 RAG 应用：[[assistantapi]]

## 来源文档

- [0代码构建私有知识问答应用](../../raw/application-user-guide/start-using/build-knowledge-base-qa-assistant-without-coding.md)
- [应用功能动态](../../raw/application-user-guide/start-using/application-release-notes.md)

