# knowledge base

知识库是阿里云百炼基于 RAG（检索增强生成）技术为大模型补充私有数据和最新信息的核心组件。大模型在生成回答前会先从知识库中检索相关内容，通过语义检索找出与查询意图相关的文本切片，从而提升回答的准确性。知识库支持通过控制台或 API 创建和管理，可集成到智能体应用、工作流应用或外部应用中。

## 支持的模型

知识库可与以下模型配合使用（详见 [知识库](../../raw/application-user-guide/knowledge-base/rag-knowledge-base.md)）：

**预置模型：**
- 千问系列：QwQ / Long / Max / Plus / Turbo / Coder / Deep-Research
- 千问VL系列：Max / Plus / Flash / OCR
- 千问开源版：Qwen3、Qwen2.5、Qwen2 等
- 第三方模型：DeepSeek-R1、DeepSeek-V3.1、Llama3.1、Yi-Large 等

**自定义模型（经 [[model-training]] 调优）：**
- 千问-Plus / Turbo
- 千问VL-Max / Plus
- 千问开源版（Qwen3、Qwen2.5、Qwen2 等）

> **注意**：支持的模型列表随时可能更新，请以控制台应用管理页面实际可选的模型为准。

## 知识库类型与使用场景

创建知识库时需选择类型，**创建后不可更改**：

| 类型 | 适用场景 | 数据格式 |
|------|---------|---------|
| **文档搜索** | 企业文档、产品手册等非结构化数据检索 | PDF、DOCX、TXT、Markdown、HTML、XLSX、图片、音视频 |
| **数据查询** | 结构化数据的查询 | XLSX、XLS |
| **图片问答** | 基于图片内容的问答 | XLSX、XLS |
| **音视频搜索** | 音视频内容检索 | MP4、MP3、WAV 等 |

文档搜索类型还可进一步选择使用场景：
- **基础文档问答**：纯文本语义检索
- **图文并茂回复**：返回图文混排内容
- **视觉理解（富文本文档）**：使用多模态向量模型（qwen3-vl-embedding）处理复杂排版、图表、公式
- **极速问答**：针对 FAQ、参数表等结构化文档优化低延迟检索

## 关键参数

### 索引配置

| 参数 | 说明 | 注意事项 |
|------|------|---------|
| **切片方式** | 智能切分（推荐）或按长度切分 | 创建后不可更改 |
| **Meta 信息抽取** | 为文本切片附加元数据（key-value），提升检索精度 | 创建后无法再配置 |
| **多轮对话改写** | 根据历史对话自动补全用户查询 | 创建后无法再开启 |
| **向量模型** | text-embedding-v4（512 维）、text-embedding-v3（512 维） | 向量维度不可更改 |

### 检索参数

| 参数 | 说明 | 建议 |
|------|------|------|
| **相似度阈值** | 仅语义相似度高于此值的切片才被召回 | 过高会导致相关内容被丢弃，需通过命中测试反复调试 |
| **权重** | 多知识库场景下干预召回顺序，仅同类型知识库间生效 | 按信息源重要程度分配 |
| **TopK（召回片段数）** | 返回给大模型的知识片段数量（1-20） | 增大可提升准确性但增加 Token 消耗 |
| **初步向量检索 TopK** | 基于语义相似性的初步召回数量（默认 50） | 降低可减少 Rerank 费用 |
| **初步关键词检索 TopK** | 基于文本匹配的初步召回数量（默认 50） | 降低可减少 Rerank 费用 |

## 使用方式

### 控制台创建

1. 进入知识库页面，选择**标准版**或**旗舰版**，点击创建知识库
2. 填写基础信息、选择知识库类型
3. 上传文件或从 OSS 导入数据，配置解析方式
4. 设置索引参数（切片方式、Meta 信息等）

### 集成方式

- **智能体应用**：在应用配置中添加文档知识库，设置相似度阈值和权重
- **工作流应用**：拖入知识库节点，支持固定知识库或动态引入（`CodeList` 变量）
- **外部应用**：通过 [[bailian-sdk]] 调用知识库检索能力

### API 调用

通过阿里云百炼 SDK 可实现知识库的全流程自动化操作，包括上传文件、创建知识库、提交索引任务和检索。完整示例代码和前置步骤详见 [知识库API指南](../../raw/application-user-guide/knowledge-base/rag-knowledge-base-api-guide.md)。

核心 API 流程：
1. `ApplyFileUploadLease` — 申请文件上传租约
2. 上传文件到临时存储（PUT 请求）
3. `AddFile` — 将文件添加到类目
4. `CreateIndex` — 初始化知识库
5. `SubmitIndexJob` — 提交索引任务
6. `Retrieve` — 检索知识库

> **注意**：API 指南目前仅适用于文档搜索类知识库。子账号需获取 `AliyunBailianDataFullAccess` 策略。

## 配额与限制

关键限制如下（完整信息见 [知识库配额与限制](../../raw/application-user-guide/knowledge-base/rag-knowledge-base-specifications.md)）：

| 类别 | 上限 |
|------|------|
| 知识库数量（RDS 数据源） | 100 个/主账号 |
| 知识库存储（标准版/旗舰版） | 100 GB / 9,999 GB |
| 每个业务空间文件数量 | 100,000 |
| 每个业务空间类目数量 | 500 |
| 文本切片长度 | 6,000 Token |
| 单次召回切片数量 | 20 |
| 检索并发（标准版/旗舰版） | 1 QPS / 50-10,000 QPS |

**文件格式限制：**
- PDF/DOCX/PPT：最大 100 MB，≤ 1,000 页
- TXT/Markdown/HTML：最大 10 MB
- XLSX/XLS：最大 10 MB，10 万行
- 图片：最大 20 MB
- 音视频：最大 512 MB

## 计费

知识库自 2026 年 1 月 4 日起正式计费，费用由**规格费用**和**模型调用费用**两部分构成：

| 规格 | 并发 | 存储 | 价格 |
|------|------|------|------|
| 标准版 | 1 QPS | ≤ 100 GB | 0.03 元/知识库/小时 |
| 旗舰版 | 50-10,000 QPS（1-200 RCU） | ≤ 9,999 GB | 0.2 元/RCU/小时 |

**模型调用费用**独立计费，包括向量化（text-embedding-v4 等）和排序（Rerank）产生的 Token 消耗。挂载 N 个知识库时 Token 消耗量按 N 倍计算。

**免费额度**：所有用户享一次性 720 小时标准版额度（新用户 30 天有效）。

**费用优化**：可关闭 Rerank 或降低初步召回 TopK 以减少排序模型费用。

## RAG 效果优化

当遇到知识召回不完整或内容不准确时，可按以下方向优化（详见 [RAG效果优化](../../raw/application-user-guide/knowledge-base/rag-optimization.md)）：

| 问题 | 优化方向 |
|------|---------|
| **检索无效：未找到相关知识** | 补充知识、优化源文件排版（推荐 Markdown）、消除实体歧义、启用多轮对话改写 |
| **检索无效：召回不相关** | 添加标签过滤、配置 Meta 信息抽取 |
| **切片不完整** | 使用智能切分、人工检查修正切片内容 |
| **重排不佳** | 调整相似度阈值、增加召回片段数（推荐按拼装长度策略） |
| **模型理解有误** | 更换更强的生成模型（如千问 Max）、优化提示词模板（限定输出、添加示例、分隔标记） |

建议在优化前通过 [[application-auto-evaluation]] 建立包含至少 100 组问题的评测基线。

## 日志与监控

知识库检索调用日志通过日志服务（SLS）投递，支持调用审计、问题排查和用量统计。开通后自动创建 LogStore，按 SLS 存储与流量计费。

核心日志字段包括：`request_id`、`pipeline_id`（知识库 ID）、`latency`（耗时 ms）、`response_status_code`、`request_body`、`response_body`（含召回切片 `data.nodes[]`）。

建议搭建的监控项：
- 调用量趋势（按小时/天）
- 业务错误率（`response_code != Success`）
- HTTP 5xx 错误率
- TopN 知识库调用排名

## 注意事项

- 知识库**类型**、**切片方式**、**Meta 信息抽取配置**在创建后均不可更改，请提前规划
- 删除知识库会**永久清除**数据且无法恢复
- 欠费后知识库暂停服务，平台存储数据保留 14 天，自购 ADB-PG 仅保留 7 天
- 命中测试会产生向量模型和排序模型的调用费用
- 提示词模板中变量 `${documents}` 应**只出现一次**

## 来源文档

- [知识库](../../raw/application-user-guide/knowledge-base/rag-knowledge-base.md)
- [RAG效果优化](../../raw/application-user-guide/knowledge-base/rag-optimization.md)
- [知识库日志与监控](../../raw/application-user-guide/knowledge-base/rag-knowledge-base-log-monitoring.md)
- [知识库配额与限制](../../raw/application-user-guide/knowledge-base/rag-knowledge-base-specifications.md)
- [知识库计费说明](../../raw/application-user-guide/knowledge-base/billing-for-knowledge-base.md)
- [知识库API指南](../../raw/application-user-guide/knowledge-base/rag-knowledge-base-api-guide.md)

