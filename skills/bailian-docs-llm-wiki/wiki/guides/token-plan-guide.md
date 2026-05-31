# token plan guide

Token Plan 团队版是阿里云百炼推出的 AI 大模型订阅服务，以 Credits 统一计量，支持文本生成与图像生成模型，兼容主流 AI 编程和智能体工具。它面向团队/企业场景，提供多租户隔离、席位管理、数据安全保障等能力，目前仅支持华北2（北京）地域。

---

## 支持的模型

Token Plan 团队版采用精确字符串白名单匹配，模型 ID 必须逐字符完全一致，不做版本兼容推理。详见 [Token Plan（团队版）概述](../../raw/model-user-guide/token-plan-guide/token-plan-overview.md)。

| 品牌 | 模型 ID | 模型能力 |
|------|---------|----------|
| 千问 | qwen3.7-max（限时活动：Credits 消耗减半） | 推理模型、文本生成 |
| 千问 | qwen3.6-plus | 推理模型、视觉理解、文本生成 |
| 千问 | qwen3.6-flash | 推理模型、视觉理解、文本生成 |
| 千问 | qwen-image-2.0 / qwen-image-2.0-pro | 图像生成 |
| 万相 | wan2.7-image / wan2.7-image-pro | 图像生成 |
| DeepSeek | deepseek-v4-pro / deepseek-v4-flash / deepseek-v3.2 | 推理模型、文本生成 |
| 月之暗面 | kimi-k2.6 / kimi-k2.5 | 推理模型、视觉理解、文本生成 |
| 智谱 AI | glm-5.1 / glm-5 | 文本生成 |
| MiniMax | MiniMax-M2.5 | 推理模型、文本生成 |

> **注意**：Token Plan 团队版与 Coding Plan 支持的模型列表不同。例如 Coding Plan 支持 `qwen3-coder-next`、`qwen3-coder-plus` 等编程专用模型，但 Token Plan 团队版不支持这些模型。反之，Token Plan 团队版支持 `deepseek-v4-pro`、`kimi-k2.6` 等模型，Coding Plan 不支持。

---

## 套餐与定价

| 坐席类型 | 价格 | 月额度 | 适用场景 |
|----------|------|--------|----------|
| 标准坐席 | ¥198/坐席/月 | 25,000 Credits | 轻度使用 |
| 高级坐席 | ¥698/坐席/月 | 100,000 Credits | 日常高频 |
| 尊享坐席 | ¥1,398/坐席/月 | 250,000 Credits | 重度依赖 AI |
| 共享用量包 | ¥5,000/个 | 625,000 Credits | 跨坐席弹性补充 |

**Credits 抵扣顺序**：坐席月度额度 → 共享用量包（优先抵扣最近到期的） → 全部用尽则服务暂停。

---

## 快速开始

完整步骤请参见 [快速开始](../../raw/model-user-guide/token-plan-guide/token-plan-quickstart.md)。

### 1. 订阅

访问 [Token Plan 团队版购买页面](https://common-buy.aliyun.com/token-plan/)，选择坐席类型和数量，主账号和 RAM 账号均可订阅。

### 2. 获取 API Key 和 Base URL

- **API Key**：在管理后台创建成员账号、分配席位后生成（格式：`sk-sp-xxx`）。
- **Base URL**：

| 协议 | Base URL |
|------|----------|
| OpenAI 兼容 | `https://token-plan.cn-beijing.maas.aliyuncs.com/compatible-mode/v1` |
| Anthropic 兼容 | `https://token-plan.cn-beijing.maas.aliyuncs.com/apps/anthropic` |

> **注意**：Token Plan 团队版、Coding Plan 和百炼按量计费三者的 API Key 和 Base URL 互不相通，请勿混用。

### 3. 接入 AI 工具

支持的工具包括：Claude Code、Cursor、Cline、OpenClaw、Hermes Agent、OpenCode、Codex、Qwen Code、Cherry Studio、Chatbox、Qoder、Lingma、Kilo CLI 等。

---

## 工具调用

Token Plan 团队版提供两种方式扩展模型能力，详见 [工具调用](../../raw/model-user-guide/token-plan-guide/token-plan-best-practice/token-plan-tool.md)。

### 模型内置工具

适用模型：`qwen3.7-max`、`qwen3.6-plus`、`qwen3.6-flash`

通过 Responses API 直接使用，模型会根据问题自动调用：

| 工具 | 说明 |
|------|------|
| 联网搜索 | 检索互联网信息 |
| 代码解释器 | 沙箱环境运行 Python |
| 网页抓取 | 访问指定 URL 提取内容 |
| 以图搜图 | 根据图片搜索相似图片 |
| 文搜图 | 根据文本搜索相关图片 |

内置工具不额外收费，token 消耗从套餐 Credits 中抵扣。

### MCP 服务

其他模型（如 deepseek-v3.2、glm-5 等）通过百炼 MCP 广场接入。需要使用**百炼通用 API Key**（`sk-xxx` 格式）鉴权 MCP 服务，与 Token Plan 专属 API Key 不同。

典型接入命令（以 Claude Code + 联网搜索为例）：

```bash
claude mcp add WebSearch https://dashscope.aliyuncs.com/api/v1/mcps/WebSearch/mcp -t http -H "Authorization: Bearer YOUR_BAILIAN_API_KEY"
```

---

## 接入图像生成模型

图像生成模型（qwen-image-2.0、wan2.7-image 等）使用独立接口，无法通过文本模型的 Base URL 直接调用。需要通过工具的 Skill、Slash Command 或 Agent 机制接入，详见 [接入多模态生成模型](../../raw/model-user-guide/token-plan-guide/token-plan-best-practice/token-plan-multimodal-gen.md)。

API 端点：
```
POST https://token-plan.cn-beijing.maas.aliyuncs.com/api/v1/services/aigc/multimodal-generation/generation
```

各工具的扩展机制：

| 工具 | 机制 | 配置路径 |
|------|------|----------|
| Claude Code | Slash Command | `.claude/commands/text-to-image.md` |
| Codex | Skill | `~/.codex/skills/token-plan-image/SKILL.md` |
| Qwen Code | Skill | `~/.qwen/skills/text-to-image/SKILL.md` |
| OpenCode | Agent | `.opencode/agents/text-to-image.md` |
| OpenClaw | Skill | `~/.openclaw/workspace/skills/token-plan-image/SKILL.md` |

---

## 团队管理

### 角色权限

| 角色 | 权限 |
|------|------|
| 拥有者 | 全部管理权限 |
| 管理员 | 与拥有者相同，由拥有者授予 |
| 成员 | 仅使用分配的 API Key 调用模型 |

### 关键操作

- **添加成员**：手动添加（仅 API 调用）或通过 SSO/钉钉（可登录管理平台）
- **分配席位**：分配后系统自动生成 API Key
- **回收席位**：席位转为未分配，API Key 立即失效
- **重置 API Key**：原 Key 立即失效，需重新分发

详见 [团队管理](../../raw/model-user-guide/token-plan-guide/token-plan-team.md)。

---

## 与 Coding Plan 的区别

| 维度 | Token Plan 团队版 | Coding Plan |
|------|-------------------|-------------|
| 适用场景 | 团队/企业日常办公 | 个人开发 |
| 模型范围 | 文本生成 + 图像生成 | 仅文本生成 |
| 计费方式 | 按 Token 消耗抵扣 Credits | 按模型调用次数 |
| 频次限制 | 无每 5 小时/每周限额 | 有限额 |
| 高峰性能 | 多租户隔离，不排队 | 可能排队 |
| 数据安全 | 不使用数据训练模型 | 数据用于服务改进 |

---

## 限制和注意事项

1. **使用范围**：仅限在兼容的 AI 编程和智能体工具中交互式使用，不可用于自动化脚本或应用后端，违规可能导致订阅暂停或 API Key 封禁。
2. **账号规范**：API Key 仅限已分配席位的成员本人使用，不可共享或泄露。
3. **地域限制**：仅支持华北2（北京）地域，海外调用需确认合规。
4. **额度不累积**：坐席额度每月重置，未用完不累积；共享用量包按月有效期清零。
5. **退订规则**：已有用量消耗的席位不可退订。
6. **每账号限购一个订阅**，同一订阅下可购买多个各类型坐席。

### 常见错误

| 报错 | 原因 | 解决方案 |
|------|------|----------|
| `InvalidApiKey: Invalid API-key provided` | 混用了百炼通用 Key 或 Coding Plan Key | 使用管理员生成的专属 API Key |
| `model 'xxx' not found` | 模型名称拼写错误或不在支持列表 | 核对模型 ID，区分大小写 |
| `invalid access token or token expired` | 使用了错误的 Base URL | 使用 Token Plan 专属 Base URL |
| `Range of input length should be [1, xxx]` | 输入超出上下文长度 | 新建会话或使用 `/compact` 压缩上下文 |

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

