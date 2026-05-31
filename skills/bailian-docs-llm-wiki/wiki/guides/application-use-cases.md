# application [[use-cases|use cases]]

百炼平台提供多种应用场景，帮助开发者将大模型能力快速集成到网站、即时通讯平台或本地环境中。核心流程通常包括创建百炼智能体应用、获取 API 凭证、通过 AppFlow 或代码对接目标渠道，并可选地配置知识库（RAG）以增强回答精准度。

## 支持的集成渠道

百炼应用目前支持以下渠道的快速集成，均可在约 10 分钟内完成基本部署：

| 渠道 | 集成方式 | 是否需要编码 | 关键依赖 |
|------|----------|-------------|----------|
| 网站（Web） | AppFlow AI 助手 + 悬浮挂件脚本 | 需少量 HTML 代码 | AppFlow |
| 微信公众号（订阅号） | AppFlow 连接流模板 | 否 | AppFlow、微信公众号后台 |
| 企业微信 | AppFlow 连接流模板 | 否 | AppFlow、企业微信开发者中心 |
| 钉钉 | AppFlow 连接流模板 | 否 | AppFlow、钉钉开放平台 |
| 本地部署 | Python + Gradio | 需要编码 | Python 3.8–3.12、LlamaIndex |

详细操作请参考 [在网站上增加一个AI助手](../../raw/application-user-guide/application-use-cases/add-an-ai-assistant-to-your-website-in-10-minutes.md)、[10分钟在钉钉上增加一个AI机器人](../../raw/application-user-guide/application-use-cases/add-an-ai-assistant-to-your-dingtalk-in-10-minutes.md) 和 [基于本地知识库构建RAG应用](../../raw/application-user-guide/application-use-cases/build-rag-application-based-on-local-retrieval.md)。

## 通用流程

无论目标渠道是什么，基本步骤一致：

### 1. 创建百炼智能体应用

在百炼控制台 [应用管理](https://bailian.console.aliyun.com/?tab=app#/app-center) 中创建 **智能体应用**，选择模型并配置 Prompt（角色设定）。

### 2. 获取 API 凭证

- **应用 ID**：在应用管理页面查看。
- **API Key**：在 [密钥管理](https://bailian.console.aliyun.com/?tab=app#/api-key) 页面创建。

这两个凭证用于后续 AppFlow 连接或 API 调用。

### 3. 对接目标渠道

- **AppFlow 方式**（网站 / 微信 / 企业微信 / 钉钉）：通过预置模板创建连接流，填入 API Key、应用 ID 及渠道侧凭证即可完成关联，无需编写代码。
- **本地部署方式**：通过 Python 代码调用通义千问 API，结合本地嵌入模型和向量检索构建 RAG 应用。

### 4. 配置知识库（可选）

为应用添加私有知识（[[knowledge-base]]），让 AI 能够精准回答业务相关问题。云端知识库通过百炼控制台上传文档并关联到应用；本地知识库则使用 LlamaIndex 进行文档切分与向量化存储。

## 关键参数与模型选择

### 推荐模型

| 模型 | 特点 | 适用场景 |
|------|------|----------|
| qwen-max | 性能最优 | 对回答质量要求高的场景 |
| qwen-plus / 千问-Plus | 效果、速度、成本均衡 | 通用客服问答 |
| qwen-turbo | 速度快、价格低 | 对延迟敏感的场景（如未认证微信公众号 5 秒超时限制） |

> **注意**：[在网站上增加一个AI助手](../../raw/application-user-guide/application-use-cases/add-an-ai-assistant-to-your-website-in-10-minutes.md) 中推荐使用 **Qwen3.5-Plus**，而 [10分钟让微信公众号成为智能客服](../../raw/application-user-guide/application-use-cases/add-an-ai-assistant-to-your-wechat-in-10-minutes.md) 等其他文档推荐 **千问-Plus**。请根据实际需求和模型可用性选择。

### RAG 参数（本地部署）

- **召回片段数**：控制检索返回的文本段数量，值越大参考信息越多，但噪声也可能增加。
- **相似度阈值**：剔除低相关性文本段，值为 0 时不做过滤。
- **温度参数**：控制生成随机性，客服场景建议使用较低值。
- **携带上下文轮数**：控制多轮对话中模型参考的历史轮数。

## 知识库配置

知识库（RAG）是让 AI 助手回答私域问题的关键能力。两种部署方式：

- **云端知识库**：通过百炼控制台上传文档 → 创建知识库 → 在应用中引用并选择"必定调用"。支持标准版和 ADB-PG 向量存储。
- **本地知识库**：支持 pdf、docx、txt、xlsx、csv 文件，使用本地或云端嵌入模型进行向量化，灵活控制切分策略。

## 限制和注意事项

- **微信公众号未认证**：只能使用被动回复消息功能，响应超时限制为 **5 秒**。建议在 Prompt 中添加"请总是给出简短的回答"或选择 qwen-turbo 以加速响应。
- **钉钉机器人消息接收模式**：必须选择 **HTTP 模式**，AppFlow 不支持 Stream 模式。
- **企业微信域名主体校验**：如果域名备案主体与企业主体不一致，需配置自有域名或使用 Nginx 代理转发。
- **企业微信可信 IP**：同一 IP 仅能用于一个企业，重复使用会被认定为服务商。可通过 ECS 或托管实例进行请求转发解决。
- **本地 RAG 文件限制**：受嵌入模型 API 限流影响，不建议传入超过 100 MB 的文件。
- **新用户免费额度**：百炼提供的 [[new-free-quota]] 可覆盖教程级别的资源消耗，超出后按 token 计费。

## 上线前建议

- 使用百炼的 [[evaluate-application]] 功能组织业务人员进行人工评测，确保回答效果符合预期。
- 通过 [[prompt-engineering-guide]] 优化提示词、完善私有知识、调整文档切分策略来改进效果。
- 可在 AppFlow 中添加 SLS 日志节点，记录 AI 助手对话日志用于后续分析。

## 来源文档

- [在网站上增加一个AI助手](../../raw/application-user-guide/application-use-cases/add-an-ai-assistant-to-your-website-in-10-minutes.md)
- [10分钟让微信公众号成为智能客服](../../raw/application-user-guide/application-use-cases/add-an-ai-assistant-to-your-wechat-in-10-minutes.md)
- [10分钟在企业微信中集成一个 AI 助手](../../raw/application-user-guide/application-use-cases/add-an-ai-assistant-to-your-work-wechat-in-10-minutes.md)
- [10分钟在钉钉上增加一个AI机器人](../../raw/application-user-guide/application-use-cases/add-an-ai-assistant-to-your-dingtalk-in-10-minutes.md)
- [基于本地知识库构建RAG应用](../../raw/application-user-guide/application-use-cases/build-rag-application-based-on-local-retrieval.md)

