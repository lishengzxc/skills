# token plan guide

Token Plan 是阿里云百炼推出的 AI 大模型订阅服务，包含面向团队/企业的 **Token Plan 团队版**和面向个人开发者的 **Coding Plan** 两种套餐形态。两者均兼容主流 AI 编程与智能体工具，通过专属 API Key 和 Base URL 接入，但在计费方式、支持模型、数据策略等方面存在显著差异。

## 产品对比：Token Plan 团队版 vs Coding Plan

| 维度 | Token Plan 团队版 | Coding Plan |
|------|-------------------|-------------|
| 适用场景 | 团队/企业日常办公 | 个人开发场景 |
| 计费方式 | 按 Token 消耗抵扣 Credits | 按模型调用次数 |
| 支持模型 | 文本生成 + 图像生成 | 仅文本生成 |
| 使用频次 | 无每 5 小时/每周限额 | 每 5 小时/每周/每月限额 |
| 高峰期性能 | 多租户隔离，不排队 | 可能排队 |
| 数据安全 | **不**使用对话数据训练模型 | 数据用于服务改进与模型优化 |

详细对比参见 [常见问题](../../raw/model-user-guide/token-plan-guide/token-plan-faq.md)。

## 支持的模型

### Token Plan 团队版

模型 ID 为**精确字符串白名单**，必须逐字符完全匹配，版本号/子型号任何差异均视为不支持。

| 品牌 | 模型 ID | 模型能力 |
|------|---------|----------|
| 千问 | qwen3.7-max（限时活动：Credits 消耗减半） | 推理、文本生成 |
| 千问 | qwen3.6-plus | 推理、视觉理解、文本生成 |
| 千问 | qwen3.6-flash | 推理、视觉理解、文本生成 |
| 千问 | qwen-image-2.0 / qwen-image-2.0-pro | 图像生成 |
| 万相 | wan2.7-image / wan2.7-image-pro | 图像生成 |
| DeepSeek | deepseek-v4-pro / deepseek-v4-flash / deepseek-v3.2 | 推理、文本生成 |
| 月之暗面 | kimi-k2.6 / kimi-k2.5 | 推理、视觉理解、文本生成 |
| 智谱 AI | glm-5.1 / glm-5 | 文本生成 |
| MiniMax | MiniMax-M2.5 | 推理、文本生成 |

### Coding Plan

推荐模型：qwen3.6-plus（视觉）、kimi-k2.5（视觉）、glm-5、MiniMax-M2.5。更多模型包括 qwen3.5-plus、qwen3-max-2026-01-23、qwen3-coder-next、qwen3-coder-plus、glm-4.7 等。

> **注意**：两个套餐支持的模型列表不同。例如 Token Plan 团队版支持 glm-5.1 但 Coding Plan 文档中未列出；Coding Plan 支持 qwen3-coder-next/qwen3-coder-plus 但 Token Plan 团队版不支持。请以各自文档中的精确白名单为准。

完整模型列表参见 [Token Plan（团队版）概述](../../raw/model-user-guide/token-plan-guide/token-plan-overview.md) 和 [Coding Plan概述](../../raw/model-user-guide/token-plan-guide/coding-plan-guide/coding-plan.md)。

## 关键参数：API Key 与 Base URL

两个套餐的 API Key 和 Base URL **互不相通**，与百炼按量计费的通用 API Key（`sk-xxx` + `dashscope.aliyuncs.com`）也不通用，**切勿混用**。

### Token Plan 团队版

| 协议 | Base URL |
|------|----------|
| OpenAI 兼容 | `https://token-plan.cn-beijing.maas.aliyuncs.com/compatible-mode/v1` |
| Anthropic 兼容 | `https://token-plan.cn-beijing.maas.aliyuncs.com/apps/anthropic` |

- API Key 格式：`sk-sp-xxx`，由管理员在管理后台为成员生成。

### Coding Plan

| 协议 | Base URL |
|------|----------|
| OpenAI 兼容 | `https://coding.dashscope.aliyuncs.com/v1` |
| Anthropic 兼容 | `https://coding.dashscope.aliyuncs.com/apps/anthropic` |

- API Key 格式：`sk-sp-xxx`，在 Coding Plan 页面获取。

## 快速接入

### Token Plan 团队版三步接入

1. **订阅**：访问 [Token Plan 团队版购买页面](https://common-buy.aliyun.com/token-plan/) 选择坐席类型和数量。
2. **获取凭证**：管理员在[管理后台](https://tokenplan-enterprise.bailian.console.aliyun.com)创建成员、分配席位、生成 API Key。
3. **配置工具**：将 API Key 和 Base URL 填入 AI 工具（Claude Code、Cursor、Cline、Cherry Studio 等）。

详细步骤参见 [快速开始](../../raw/model-user-guide/token-plan-guide/token-plan-quickstart.md)。

### Coding Plan 接入

1. 访问 [Coding Plan 购买页](https://common-buy.aliyun.com/coding-plan) 订阅。
2. 在 Coding Plan 页面获取专属 API Key 和 Base URL。
3. 配置到 AI 编程工具中。

## 套餐与定价

### Token Plan 团队版

| 坐席类型 | 价格 | 额度 | 适用场景 |
|---------|------|------|---------|
| 标准坐席 | ¥198/坐席/月 | 25,000 Credits | 轻度使用 |
| 高级坐席 | ¥698/坐席/月 | 100,000 Credits | 日常高频 |
| 尊享坐席 | ¥1,398/坐席/月 | 250,000 Credits | 重度依赖 |
| 共享用量包 | ¥5,000/个 | 625,000 Credits | 弹性溢出 |

**Credits 抵扣顺序**：坐席月度额度 → 共享用量包（优先最近到期的） → 全部用尽则服务暂停。

### Coding Plan

Pro 套餐 ¥200/月，每 5 小时 6,000 次、每周 45,000 次、每月 90,000 次请求限额。

> **注意**：Coding Plan Lite 套餐已于 2026 年 3 月 20 日停止新购，4 月 13 日起停止续费与升级。

## 团队管理（Token Plan 团队版）

Token Plan 团队版提供完整的团队管理能力，包括：

- **角色权限**：拥有者、管理员、成员三级角色。
- **成员管理**：手动添加成员，或通过 SAML SSO / 钉钉登录自动加入。
- **席位操作**：分配、回收、升级席位；每个成员同一时间只能持有一个席位。
- **用量分析**：查看用量趋势、各模型和各成员的 Credits 消耗明细。

详细操作参见 [团队管理](../../raw/model-user-guide/token-plan-guide/token-plan-team.md)。

## 工具调用与扩展能力

### 模型内置工具（Token Plan 团队版）

qwen3.7-max、qwen3.6-plus、qwen3.6-flash 通过 Responses API 内置 5 种工具：

- 联网搜索、代码解释器、网页抓取、以图搜图、文搜图

内置工具不额外收费，token 消耗从套餐 Credits 抵扣。

### MCP 服务

其他模型可通过百炼 MCP 广场接入联网搜索等 MCP 服务。MCP 服务使用百炼通用 API Key（`sk-xxx`），与套餐专属 API Key 不同。部分 MCP 服务限时免费，每月 2,000 次免费额度。

接入方式参见 [工具调用](../../raw/model-user-guide/token-plan-guide/token-plan-best-practice/token-plan-tool.md) 和 [[web-search-for-coding-plan]]。

### 图像生成模型

图像生成模型（qwen-image-2.0、wan2.7-image 等）使用独立接口，无法通过文本模型的 Base URL 直接调用。需通过工具的 Skill / Slash Command / Agent 扩展机制接入，详见 [[token-plan-multimodal-gen]]。

### 视觉理解

- qwen3.6-plus、qwen3.5-plus、kimi-k2.5 原生支持视觉理解，可直接传入图片。
- glm-5、MiniMax-M2.5 等纯文本模型可通过 Skill/Agent 方式调用视觉模型辅助理解图片，详见 [[add-vision-skill]]。

## 使用限制和注意事项

1. **使用范围**：两个套餐均仅限在兼容的 AI 编程和智能体工具中**交互式使用**，不可用于自动化脚本或应用后端。违规可能导致订阅暂停或 API Key 封禁。
2. **API Key 独占**：API Key 仅限已分配席位的成员本人使用，不可共享或公开泄露。
3. **地域限制**：Token Plan 团队版目前仅支持**华北2（北京）**地域。
4. **退订规则**：Token Plan 团队版支持按席位退订（已有用量消耗的席位不可退订）；Coding Plan **不支持退款**。
5. **额度不累积**：坐席额度在每个订阅月到期时重置，未用完的额度不累积。
6. **每个阿里云账号**限购一个 Token Plan 团队版订阅，同一订阅下可购买多个坐席。

## 常见错误排查

| 报错信息 | 常见原因 | 解决方案 |
|---------|---------|---------|
| `InvalidApiKey: Invalid API-key provided` | 混用了通用 API Key 或其他套餐的 Key | 确认使用对应套餐的专属 API Key |
| `model 'xxx' not found` | 模型名称拼写错误或不在支持列表 | 严格使用白名单中的模型 ID，区分大小写 |
| `invalid access token or token expired` | 使用了错误的 Base URL | 检查 Base URL 是否与套餐匹配 |
| `Range of input length should be [1, xxx]` | 输入超出模型上下文长度

## 来源文档

- [快速开始](../../raw/model-user-guide/token-plan-guide/token-plan-quickstart.md)
- [Token Plan（团队版）概述](../../raw/model-user-guide/token-plan-guide/token-plan-overview.md)
- [团队管理](../../raw/model-user-guide/token-plan-guide/token-plan-team.md)
- [常见问题](../../raw/model-user-guide/token-plan-guide/token-plan-faq.md)
- [工具调用](../../raw/model-user-guide/token-plan-guide/token-plan-best-practice/token-plan-tool.md)
- [接入多模态生成模型](../../raw/model-user-guide/token-plan-guide/token-plan-best-practice/token-plan-multimodal-gen.md)
- [Coding Plan概述](../../raw/model-user-guide/token-plan-guide/coding-plan-guide/coding-plan.md)
- [添加视觉理解能力](../../raw/model-user-guide/token-plan-guide/coding-plan-guide/add-vision-skill.md)
- [联网搜索](../../raw/model-user-guide/token-plan-guide/coding-plan-guide/web-search-for-coding-plan.md)
- [常见问题](../../raw/model-user-guide/token-plan-guide/coding-plan-guide/coding-plan-faq.md)

