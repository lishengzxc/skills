# Token 计量与上下文窗口

Token 是百炼平台衡量模型输入与输出文本量的基本单位，也是计费和资源限制的核心度量。上下文窗口（Context Window）则定义了模型在单次请求中能处理的最大 Token 总量，直接影响模型可参考的信息范围和对话轮次深度。

## 什么是 Token

Token 是模型处理文本时的最小语义单元。中文环境下，1 个 Token 大约对应 1.5 个汉字（即 100 万 Token 约等于 70 万汉字）。英文中 1 个 Token 大约对应 4 个字符或 0.75 个单词。每次模型调用产生的 Token 分为两部分：

- **输入 Token**：用户发送给模型的内容，包括系统提示词（System Prompt）、对话历史、工具定义等。
- **输出 Token**：模型生成的回复内容，包括思考过程（若开启思考模式）和最终回答。

## 在百炼平台中的使用场景

### 计费与成本管理

百炼平台按输入 Token 和输出 Token 分别计价。部分模型实行阶梯计费——单价取决于单次请求的输入 Token 总量。例如 `qwen3.6-plus` 在输入 ≤256K Token 时为 2 元/百万 Token，超出后适用更高单价。

主要成本优化手段：

| 方式 | 说明 |
|------|------|
| Batch 调用 | 输入和输出单价按实时推理的 50% 计费 |
| 上下文缓存 | 仅输入 Token 享有折扣，与 Batch 不叠加 |
| 节省计划 / 资源包 | 承诺消费额度或预购 Token 数量获取折扣 |
| Token Plan 团队版 | 以 Credits 统一计量，坐席制按月分配额度 |

计费抵扣顺序为：免费额度 → 资源包 → 其他模型节省计划 → AI 通用型节省计划 → 按量付费。

### 监控与观测

- **模型监控**：通过控制台的用量统计页面查看各模型的 Token 消耗，支持按业务空间维度统计，延迟约 1 小时。大语言模型以 Token 为统计单位，图像模型按张、视频模型按秒。
- **应用观测**：追踪每次应用调用的 Token 总量（输入 + 输出），支持按 Token 总量/输入 Token/输出 Token 进行过滤和告警。
- **Grafana 接入**：通过 `model_usage` 等 PromQL 指标获取 Token 用量数据，支持按模型、API Key、业务空间等维度过滤。

### 向量模型中的 Token 限制

文本向量模型对单行输入有明确的 Token 上限：`text-embedding-v4` 和 `text-embedding-v3` 单行最大 8,192 Token，`text-embedding-v1/v2` 单行最大 2,048 Token。

## 上下文窗口

上下文窗口是模型单次请求可处理的最大 Token 容量。输入 Token 与输出 Token 之和不能超过上下文窗口大小。

### 主流模型的上下文窗口

| 上下文窗口大小 | 代表模型 |
|--------------|---------|
| 1M Token（约 70 万汉字） | `qwen3.7-max`、`qwen3.6-plus`、`qwen3.6-flash`、`deepseek-v4-pro` |
| 256K Token | `qwen3.6-max-preview`、`kimi-k2.6` |
| 192K~198K Token | `glm-5.1`（198K）、`MiniMax-M2.5`（192K） |
| 128K Token | `qwen-plus`、`deepseek-v3.2` |

### 思考预算（Thinking Budget）

开启 `enable_thinking` 参数后，模型在输出中增加思考过程，思考内容也计入 Token 消耗。不同模型的思考预算上限不同：

| 模型 | 思考预算上限 |
|------|------------|
| `qwen3.7-max` | 256K Token |
| `qwen3.6-plus` | 128K Token |
| `qwen3.6-flash` | 128K Token |

## 关键参数与配置

| 参数 | 作用 | 建议 |
|------|------|------|
| `max_tokens` | 限制模型输出的最大 Token 数 | 根据任务合理设置，避免不必要的长输出 |
| `enable_thinking` | 开启思考模式，增加推理深度 | 仅在复杂推理场景使用，会显著增加输出 Token |
| `thinking_budget` | 控制思考过程的最大 Token 数 | 与 `enable_thinking` 配合使用 |

## 最佳实践

- **控制输出长度**：合理设置 `max_tokens` 和思考预算，避免 Token 浪费。
-

## 关联主题页

- [token plan guide](../guides/token-plan-guide.md)
- [test 1](../guides/test-1.md)
- [model monitoring](../guides/model-monitoring.md)
- [application monitoring](../guides/application-monitoring.md)
- [model inference](../guides/model-inference.md)
- [general text embedding](../api/general-text-embedding.md)
- [qwen api reference](../api/qwen-api-reference.md)

