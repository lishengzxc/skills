# application monitoring

应用观测是阿里云百炼平台提供的端到端应用调用链路追踪功能，支持查看业务空间内应用的处理流程（如向量生成、向量检索和大模型调用），并获取延时、Token 量等性能指标，更新频率为分钟级。该功能帮助开发者追踪应用内部调用链路、查看模型响应延时和思考过程，解决多节点架构带来的调试与运维挑战。

## 支持的应用类型

根据[应用观测](../../raw/application-user-guide/application-monitoring/application-observation.md)文档，当前支持以下应用类型：

- [[agent-application]]（智能体应用）
- [[workflow-application]]（工作流应用）
- [[rich-code-application]]（高代码应用）

> **注意**：应用观测目前暂不支持通过 Assistant API 创建的智能体应用，也暂无对外 API。

## 核心功能

### 调用链路追踪

应用观测基于节点（Span）机制记录每个操作单元的名称、类型、起止时间等信息，节点之间支持嵌套关系。

### Span 筛选模式

| 模式 | 说明 |
|------|------|
| Root Span | 仅显示根节点（默认模式） |
| All Span | 显示所有 Span，平铺展示 |
| Model Span | 仅显示包含模型调用的 Span |

### 过滤器支持的筛选条件

支持按状态、Span Name、输入/输出关键词、延时（毫秒）、Token 总量/输入 Token/输出 Token、标签等字段筛选。

### 监控统计

包括调用次数、失败次数与失败率、Token 总量、平均单次请求 Token 量、平均首 Token 耗时、平均调用时长等指标，支持按分钟/小时/天聚合，最长可查看 30 天数据。

### 数据导出

支持导出为 JSONL 或 EXCEL 格式。

### 数据标注

支持对 Span 数据添加布尔值、分类、数字、文本类型的标签标注，与[[application-evaluation]]的标签管理功能共享。

### 添加到评测集

支持将 Span 数据批量添加到评测集，用于构建贴近实际业务场景的评测样本。每个评测集最多支持 50 个字段映射。

## 支持的节点类型

如[应用观测](../../raw/application-user-guide/application-monitoring/application-observation.md)所述，节点仅在被触发或调用时展示：

| 节点类型 | 说明 |
|----------|------|
| CHAIN | 将大模型节点与其他节点连接，根节点时名称为 AgentApp 或 WorkflowApp |
| AGENT | 智能体调用 |
| RETRIEVER | 检索操作（TextRetriever / VectorRetriever），默认返回 100 个文本切片 |
| REWRITER | 基于上下文自动调整 Prompt 以提升检索效果 |
| EMBEDDING | 将输入 Prompt 转化为向量 |
| RERANKER | 计算文本切片相似度分数并降序排列 |
| LLM | 大模型推理/生成，延时包含输出回复过程 |
| TOOL | 插件调用（官方/自定义） |
| GUARDRAIL | 阿里绿网内容安全检测 |

工作流应用额外支持：START、API、CLASSIFIER、TEXT_CONVERTER、SCRIPT、CONDITION、FUNCTION_COMPUTE、APP_FLOW、END 等节点。

> **注意**：高代码应用目前不支持追踪其内部调用链路，仅展示 FullCodeApp 节点。

## 使用前提条件

首次使用需完成三步配置（建议使用主账号操作）：

1. 授权可观测链路 [[opentelemetry]] 服务角色权限
2. 开通可观测链路 OpenTelemetry 服务
3. 初始化 OpenTelemetry 存储 LogStore

开通后通常分钟级生效，高峰期可能延迟。

### 子账号权限配置

需要同时配置：
- `AliyunBailianFullAccess` 权限
- `应用观测-操作`（或`管理员`）页面权限
- 创建并授予 `ram:CreateServiceLinkedRole` 系统策略

## 限制与注意事项

- 应用必须已发布且属于当前业务空间才会出现在可选列表中
- 关闭观测后追踪数据停止同步，重新添加仅同步新增数据
- 目前暂不支持观测[[long-term-memory]]中的检索过程
- 高代码应用需在代码中使用 AgentScope-AI 的 Tracing 模块定义上报信息，并在部署时添加 `--telemetry enable` 参数

## 计费说明

根据[应用观测](../../raw/application-user-guide/application-monitoring/application-observation.md)说明，应用观测功能本身不收费，但产生的数据存储在可观测链路 OpenTelemetry 服务中，需支付相关存储费用。

## 来源文档

- [应用观测](../../raw/application-user-guide/application-monitoring/application-observation.md)

