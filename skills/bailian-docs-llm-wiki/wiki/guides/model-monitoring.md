# model monitoring

百炼平台提供模型监控与用量统计功能，帮助开发者追踪模型调用情况、监控性能指标、管理成本并设置异常告警。该功能覆盖调用记录查看、Token 消耗统计、性能指标（延时/RPM/TPM/失败率）监控以及主动告警配置等场景。

## 功能概览

模型监控体系包含两大模块：

| 模块 | 核心能力 | 数据延迟 |
|------|---------|---------|
| **用量统计** | 按业务空间查看各模型的调用量和 Token 消耗 | 约 1 小时 |
| **模型监控（普通）** | 调用次数、失败率、调用时长等指标 | 小时级 |
| **模型监控（高级）** | 分钟级指标采集、告警、Prometheus API 接入 | 分钟级 |
| **模型日志** | 查看每次调用的输入/输出及 Token 消耗明细 | 分钟级 |

## 支持的模型与地域

根据 [模型监控](../../raw/model-user-guide/model-monitoring/model-telemetry.md) 文档，不同功能的模型和地域支持范围有所不同：

- **普通监控**：支持[模型列表](https://help.aliyun.com/zh/model-studio/models)中的所有模型，包括基于它们调优后的自定义模型。
- **高级监控**：支持北京、新加坡、弗吉尼亚地域下的所有模型。
- **告警功能**：仅支持北京、新加坡地域。
- **日志功能**：仅支持北京、新加坡地域下的部分模型（如 qwen3-max、qwen-plus、deepseek-v3 系列等）。

> **注意**：[模型用量](../../raw/model-user-guide/model-monitoring/model-usage-statistics.md) 中提到用量统计支持"所有模型"，而模型日志功能仅覆盖部分模型。如需查看某次调用的详细 Token 消耗，需确认目标模型在日志支持列表中。

## 用量统计

### 查看方式

在控制台 [用量统计](https://bailian.console.aliyun.com/?tab=model#/model-usage/usage-statistics) 页面查看，数据按 [[workspace]]（业务空间）维度统计，不支持按阿里云账号维度直接统计。如需账号级数据，可通过 [账单详情](https://usercenter2.aliyun.com/finance/expense-report/expense-detail) 导出。

### 用量统计单位

根据 [模型用量](../../raw/model-user-guide/model-monitoring/model-usage-statistics.md)，不同模型类型的统计单位如下：

| 模型类型 | 统计单位 |
|---------|---------|
| 大语言模型（文本生成/深度思考/视觉理解） | Token |
| 图像生成 | 张 |
| 视频生成 | 秒 |
| 语音模型 | 秒、字符或 Token（视模型而定） |
| 全模态模型 | Token（各模态分别计算） |
| 向量模型 | Token |

### 免费额度管理

在 [免费额度](https://bailian.console.aliyun.com/?tab=model#/model-usage/free-quota) 页面可管理免费额度。**免费额度用完即停**开关开启后，额度用尽时服务将返回 `403 AllocationQuota.FreeTierOnly` 错误，避免产生额外费用。

### 关键限制

- 用量统计**不支持查看 30 天以前**的数据，更早的记录需通过 [费用与成本](https://billing-cost.console.aliyun.com/) 查询。
- 仅「大语言模型」页签支持按推理类型（[[real-time-inference]] 或 [[batch-inference]]）筛选。

## 监控指标与查看

系统自动采集主账号下所有业务空间的模型调用数据，按"模型 + 业务空间"维度生成监控记录。监控指标分为四类：

- **安全**：内容安全错误次数等
- **成本**：平均单次请求调用量等
- **性能**：调用时长、首 Token 延时、RPM、TPM 等
- **错误**：失败次数、失败率、限流错误次数等

支持按 [[api-key]]、推理类型和时间范围进行筛选。

## 模型日志（历史对话）

开通步骤：

1. 使用主账号登录目标业务空间的模型监控页面
2. 点击**模型监控配置** → 依次开通**审计日志**和**推理日志**
3. 在模型监控列表中点击目标模型的**日志**操作

日志记录每次调用的输入、输出、Token 消耗和耗时，适用于故障排查和内容审计。如需停止记录，在模型监控配置中关闭推理日志即可。

## 告警配置

在模型告警页面创建告警规则，选择监控模型和模板即可。支持的通知方式：

| 告警等级 | 通知渠道 |
|---------|---------|
| 紧急（CRITICAL） | 电话、短信、邮件 |
| 错误（ERROR） | 短信、邮件 |
| 警告（WARNING） | 短信、邮件 |
| 普通（INFO） | 邮件 |

还支持钉钉群机器人、企业微信机器人及 Webhook。

## 接入 Grafana 与自建应用

高级监控数据存储在私有 Prometheus 实例中，支持标准 Prometheus HTTP API。关键监控指标包括：

| 指标名称 | 描述 |
|---------|------|
| `model_call_count` | 调用次数总和 |
| `model_call_duration` | 调用时长均值 |
| `model_first_token_duration` | 首包时长均值 |
| `model_usage` | 模型用量总和 |

查询示例：

```
GET {HTTP_API}/api/v1/query_range?query=model_usage{workspace_id="llm-xxx",model="qwen-plus"}&start=2025-11-20T00:00:00Z&end=2025-11-20T23:59:59Z&step=60s
Authorization: Basic base64Encode(AccessKey:AccessKeySecret)
```

支持的过滤条件（LabelKey）：`user_id`、`apikey_id`、`workspace_id`、`model`、`protocol`、`status_code`、`error_code`、`usage_type` 等。

## 生产环境建议

根据 [模型用量](../../raw/model-user-guide/model-monitoring/model-usage-statistics.md) 的建议：

- 通过 `max_tokens` 参数和[[deep-thinking]]的思考长度限制控制单次生成成本
- 简单任务优先使用轻量模型（如 `qwen-turbo`），避免过度使用高价模型
- 配置模型监控告警，及时发现 Token 消耗突增或静默失败
- 非实时任务使用 [[batch-inference]] 降低成本
- 优化 Prompt 减少不必要的输入 Token 消耗

## 来源文档

- [模型用量](../../raw/model-user-guide/model-monitoring/model-usage-statistics.md)
- [模型监控](../../raw/model-user-guide/model-monitoring/model-telemetry.md)

