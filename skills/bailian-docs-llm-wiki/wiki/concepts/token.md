# Token 计量与计费

Token 是百炼平台衡量模型输入与输出文本量的基本单位，也是模型推理和训练费用计算的核心依据。百炼平台围绕 Token 构建了完整的计量、统计与计费体系，覆盖模型推理调用、模型训练、模型部署等场景。

---

## 计费场景与方式

| 场景 | 计费方式 | 说明 |
|------|---------|------|
| **模型推理（调用）** | 输入/输出 Token 分别计价，按量后付费 | 大语言模型以 Token 为单位；图像按张、视频按秒、语音按秒/字符/Token |
| **模型训练** | 按训练 Token 总量计费 | 费用 =（训练数据 Token + 混合数据 Token）× 循环次数（`n_epochs`）× 单价 |
| **模型部署** | 按时长 / 按 Token 用量 / 包月预付费 | 部分部署模式仍以 Token 为计量维度 |
| **Token Plan 团队版** | Token 消耗抵扣 Credits | 独立计费体系，与按量付费的 API Key 不互通 |

详细单价参见 [[test-1]]。

---

## 输入与输出 Token 分别计价

调用文本生成模型时，**输入 Token**（Prompt、系统指令、上下文历史）和**输出 Token**（模型生成的回复）按不同单价分别计费。以 `qwen-plus` 中国内地为例：输入 0.8 元/百万 Token，输出另计。同一模型在不同部署地域（中国内地、新加坡、弗吉尼亚等）单价差异显著。

### 阶梯计费

部分模型（如 `qwen3-max`）实行阶梯计费——单价由**单次请求的输入 Token 总量**所在区间决定，该请求所有 Token 按对应阶梯统一结算。例如分 0–32K、32K–128K、128K–256K 三档。

### Batch 调用与上下文缓存的折扣

- **Batch 调用**：输入/输出单价按实时推理的 **50%** 计费。
- **上下文缓存**：仅输入 Token 享有折扣。
- 两者**不可同时生效**。

---

## 训练场景中的 Token 计量

模型调优（Fine-tuning）费用直接与训练 Token 数量挂钩，受以下参数影响：

| 参数 | 影响 |
|------|------|
| `n_epochs` | 训练循环次数，直接乘以 Token 总量计费 |
| `max_length` | 单条数据最大 Token 长度，超出则丢弃该条数据 |
| `data_augmentation` | 开启混合训练后，混合数据的 Token 也计入总量按标准计费 |

不同调优方式对数据量的要求差异很大：CPT（继续预训练）需 1000 万+ Token 的无标签文本，SFT 需 1000+ 条问答对，DPO 需 100+ 组偏好对。详见 [[fine-tuning]] 和 [[model-training]]。

---

## Token 用量监控与统计

百炼通过 [[model-monitoring]] 提供多层级的 Token 用量可观测能力：

| 功能 | 粒度 | 说明 |
|------|------|------|
| **用量统计** | 按业务空间 + 模型 | 查看各模型的调用量和 Token 消耗，数据延迟约 1 小时 |
| **模型日志** | 单次调用 | 查看每次请求的输入/输出 Token 明细，分钟级延迟 |
| **高级监控** | 分钟级采集 | 通过 `model_usage` 指标查询 Token 用量，支持 Prometheus API 接入 Grafana |
| **应用观测** | Span 级别 | 追踪应用内各节点的 Token 总量、输入/输出 Token，支持按 Token 量筛选 |

用量统计按模型类型使用不同单位：大语言模型按 Token，图像按张，视频按秒，语音按秒/字符/Token，全模态模型各模态分别计算 Token。

应用观测（[[application-monitoring]]）中可按 Token 总量、输入 Token、输出 Token 筛选 Span，并查看平均单次请求 Token 量等聚合指标。

---

## 关键参数与配置

### 计费相关 API 响应字段

调用文本生成模型后，API 响应中会返回 `usage` 对象，包含 `[[prompt|prompt]]_tokens`（输入）、`completion_tokens`（输出）和 `total_tokens`（合计），可用于应用侧的 Token 消耗追踪。

### 免费额度与用量控制

- 首次开通百炼（中国内地版）自动获得新人免费额度，有效期 30–90 天，仅抵扣实时推理费用。
- 开启「**免费额度用完即停**」可在额度耗尽时返回错误码 `AllocationQuota.FreeTierOnly`，防止意外扣费。

### 账单字段

账单中 `实例 ID` 字段格式为：`ApiKeyID;业务空间ID;模型名称;输入输出类型;调用渠道;免费额度用完即停标识`，可追溯至具体调用来源。

---

## 成本优化

抵扣优先

## 关联主题页

- [[test-1|test 1]] — `../guides/test-1.md`
- [[token-plan-guide|token plan guide]] — `../guides/token-plan-guide.md`
- [[model-monitoring|model monitoring]] — `../guides/model-monitoring.md`
- [[application-monitoring|application monitoring]] — `../guides/application-monitoring.md`
- [[model-training|model training]] — `../api/model-training.md`
- [[fine-tuning|fine tuning]] — `../guides/fine-tuning.md`
- [[qwen-api-reference|qwen api reference]] — `../api/qwen-api-reference.md`


