---
name: bailian-docs-llm-wiki
description: >-
  阿里云百炼平台技术文档知识库（LLM Wiki）。当用户查询百炼模型列表、API 参数、错误码、
  应用开发（智能体/RAG/知识库/记忆/插件）、模型对比与定价、SDK/OpenAI 兼容接口、
  多模态能力（语音/图片/视频）、Token 计费等百炼相关问题时激活。
  包含 models 结构化模型市场数据（含 contextWindow/QPM/价格/sample code）、
  wiki 合成层（主题页/概念页/对比页）和 raw 原始文档层；
  模型规格类问题先查 models/index.md，文档类问题先查 wiki/index.md。
---

# 百炼文档知识库

> 阿里云百炼平台的完整技术文档知识库，涵盖模型使用、应用开发、API 参考等内容。

## 何时使用

当用户涉及以下场景时激活此 Skill：

- 查询百炼平台的模型列表、模型参数、调用方式（含具体模型的 contextWindow、QPM、定价、sample code 等结构化字段 → 查 `models/`）
- 查阅百炼 API 参数、请求/响应格式、错误码
- 了解百炼应用开发（智能体、RAG、知识库、记忆、插件等）
- 选择模型、比较模型能力、了解模型定价与限流
- 使用百炼 SDK / OpenAI 兼容接口
- 语音识别、语音合成、图片生成、视频生成等多模态能力
- Token Plan、计费、免费额度等商务问题

## 何时**不要**使用

- 询问 OpenAI / Anthropic / Google 等其他厂商的 API、模型或定价
- 通用编程问题、框架问题（React、Vue、Spring 等）
- `bl` CLI 命令本身的用法（那是 `bailian-cli` skill 的职责）
- 与百炼无关的阿里云产品（OSS、ECS、RDS 等）

## 文档层级

### 1. Wiki 层（合成层） — `wiki/`

由 LLM 自动合成的结构化 Wiki，包含三类页面：

- **主题页**：按功能领域聚合的综合文档（`wiki/guides/*.md`、`wiki/api/*.md`）
- **概念页**：跨主题的横切概念（`wiki/concepts/*.md`，如 RAG、流式输出、Function Calling 等）
- **对比页**：同类方案的结构化对比分析（`wiki/comparisons/*.md`，含对比表格）

**优先查阅 `wiki/index.md`，它是全部 wiki 页面的索引入口。**

每篇主题页末尾都有 `## 来源文档` 段落，列出对应的 raw 原文路径，可用于追溯和深度阅读。

### 2. Models 层（模型市场结构化数据） — `models/`

直接从百炼 console 网关 `listFoundationModels` 抓取的**结构化模型元数据**，对应「模型广场」页面。涉及具体模型的能力、上下文长度、QPM 限流、定价、官方 sample code 等问题时，优先查这里，比 wiki 更新更准。

**三层产物，单一数据源**（`families.jsonl` / `models.jsonl` 是机器查询入口，`index.md` 是 `families.jsonl` 的人类可读视图）：

- `models/index.md` — 按主能力分桶的家族索引（中文标签 + 模型代码 + 链接到明细 JSON）
- `models/families.jsonl` — **每行一个家族**，含 `slug` / `name` / `description` / `primaryCapability` /
  `capabilities`（家族下所有 item 的并集）/ `providers` / `itemCount` / `maxContextWindow` / 轻量 `items[]` 摘要 / `detailPath`。例如：
  - 列出推理类家族：`jq -c 'select(.primaryCapability=="Reasoning") | {slug,name,itemCount}' families.jsonl`
  - 找包含某 provider 的家族：`grep '"moonshot-ai"' families.jsonl | jq -c .slug`
- `models/models.jsonl` — **每行一个主干模型**（跨家族扁平），含 `model` / `family` / `capabilities` /
  `features` / `contextWindow` / `maxInputTokens` / `maxOutputTokens` / `prices`（精简）/ `qpmInfo`（精简）/
  `docUrl` / `detailPath`。例如：
  - 列出 contextWindow ≥ 1M 的模型：`jq -c 'select(.contextWindow>=1000000)' models.jsonl`
  - 找支持 function-calling 的文本模型：`grep '"function-calling"' models.jsonl | jq -c '{model,family,contextWindow}'`

两份 JSONL 的 **join 字段**：`models.jsonl[].family == families.jsonl[].slug`。命中后按 `detailPath` 打开 `groups/<slug>.json` 取完整字段（`samples`、`predictConfig`）。

- `models/groups/<slug>.json` — 单个模型家族的完整明细：家族层 `name` / `description`；`items[]` 为该家族主干模型版本（已裁剪账号/UI 字段），保留 `model`（API 调用名）、`contextWindow`、`maxInputTokens`、`maxOutputTokens`、`capabilities`、`features`、`provider`、`docUrl`、`prices`、`qpmInfo`、`samples`（扁平化调用示例 `samples.<sdk>.<api>.<lang>`）、`predictConfig`（入参定义，对齐 Playground）
- `models/meta.json` — 增量爬取的指纹缓存，**不要给用户引用**

#### 能力代码（capability）映射

| 代码 | 中文 | 代码 | 中文 |
| --- | --- | --- | --- |
| `TG` | 文本生成 | `TTS` | 语音合成 |
| `Reasoning` | 推理 | `ASR` | 语音识别 |
| `VU` | 视觉理解 | `Multimodal-Omni` | 全模态 |
| `IG` | 图像生成 | `ME` | 多模态嵌入 |
| `VG` | 视频生成 | `TR` | 翻译 |

`index.md` 按 `capabilities[0]`（主能力）分桶，查找时按中文标签定位章节。

### 3. Raw 层（原始层） — `raw/`

从 help.aliyun.com 爬取的原始文档，按分类存放：

- `raw/model-user-guide/` — 模型使用指南
- `raw/application-user-guide/` — 应用使用指南
- `raw/model-api-reference/` — 模型 API 参考
- `raw/application-api-reference/` — 应用 API 参考

当 wiki / models 层信息不足时，从 raw 层获取完整原文。

## 强约束

- **回答必须基于 `wiki/` 或 `raw/` 下的实际文件内容**，不得凭印象编造 API、参数名、错误码或定价数字。
- **引用时附带相对路径**（如 `wiki/api/qwen-api.md`），方便用户验证。
- 如果在 `wiki-metadata.json` 中查到某页面 `qualityScore <= 2`，绕过 wiki 该页，回退到 `raw/` 原文回答。

## 查阅流程

1. **模型规格类问题先查 models**：具体模型 → `index.md` 定位家族 → `groups/<slug>.json`；跨家族筛选 → grep `models.jsonl`；按家族筛选 → grep `families.jsonl`
2. **概念/使用方式查 wiki 索引**：读 `wiki/index.md` 找主题页/概念页/对比页
3. **回溯 raw 原文**：从主题页末尾 `## 来源文档` 进 `raw/.../*.md`
4. **全文索引**：`llms.txt` 含目录树，可快速定位
5. **特定参数/错误码**：直接在 `raw/` 下 grep

## 注意事项

- 文档持续更新，如遇矛盾以 `models/` > `raw/` > `wiki/` 的顺序为准
- `wiki/eval-report.md` 是最近一次评测报告

**以下文件是增量缓存，不要引用给用户**：`metadata.json`、`wiki-metadata.json`、`crawl-report.json`、`models/meta.json`

> 数据刷新：`npm run wiki`（文档+合成）、`npm run models`（模型市场）。
