# 文本Embedding与多模态向量对比

## 概述

在基于百炼平台构建语义搜索、推荐系统或内容理解应用时，开发者通常面临一个核心选型问题：**应该使用通用文本向量（Text Embedding）还是多模态向量（Multimodal Embedding）？** 两者都将输入转换为数值向量表示，但在输入模态、模型架构、API 设计、成本结构和适用场景上存在显著差异。

本页面从多个技术维度对两类方案进行系统对比，帮助开发者根据业务需求做出合理的技术选型。

---

## 关键维度对比

### 基础能力对比

| 维度 | 文本 Embedding | 多模态向量 |
|------|---------------|-----------|
| **输入格式** | 纯文本（字符串、字符串列表、文本文件） | 文本、图片（URL/Base64）、视频（URL）及其组合 |
| **输出格式** | 浮点向量（`float` 数组），每条文本对应一个向量 | 浮点向量，支持**独立向量**（每个输入各一个）和**融合向量**（多模态合并为一个） |
| **语义空间** | 纯文本语义空间 | 统一的跨模态语义空间（文本、图像、视频共享） |
| **支持语种** | v4: 100+ 语种及编程语言；v3: 50+；v2: 10 种；v1: 6 种 | qwen3-vl-embedding: 33 种语言；2026-03-06 快照版: 30+ 语言 |
| **调用模式** | 同步调用 + 批处理（异步） | 同步调用（HTTP） |

### 模型与向量维度对比

| 维度 | 文本 Embedding | 多模态向量 |
|------|---------------|-----------|
| **代表模型** | `text-embedding-v4`（推荐）、`text-embedding-v3`、`v2`、`v1` | `qwen3-vl-embedding`（推荐）、`qwen2.5-vl-embedding`、`tongyi-embedding-vision-plus/flash` 系列、`multimodal-embedding-v1` |
| **默认向量维度** | v4: 1024；v3: 1024；v2/v1: 1536 | qwen3-vl: 2560；qwen2.5-vl: 1024；tongyi-plus: 1152；tongyi-flash: 768；v1: 1024 |
| **维度可调** | v3/v4 支持（64–2048），v1/v2 固定 1536 | 大部分模型支持 `dimension` 参数，`multimodal-embedding-v1` 和旧版 tongyi 系列固定 |
| **批处理支持** | ✅ 支持（async-v1/v2，单次最多 10 万行） | ❌ 不支持批处理接口 |

### API 端点与调用方式对比

| 维度 | 文本 Embedding | 多模态向量 |
|------|---------------|-----------|
| **API 端点** | 同步：`POST /compatible-mode/v1/embeddings`<br>批处理：`POST /api/v1/services/embeddings/text-embedding/text-embedding`（异步） | `POST /api/v1/services/embeddings/multimodal-embedding/multimodal-embedding` |
| **OpenAI SDK 兼容** | ✅ 同步接口完全兼容 | ❌ 需使用 DashScope SDK 或原生 HTTP |
| **SDK 支持** | OpenAI SDK + DashScope SDK | DashScope SDK |
| **认证方式** | [[api-key]]（环境变量 `DASHSCOPE_API_KEY`） | [[api-key]]（环境变量 `DASHSCOPE_API_KEY`） |

### 输入限制对比

| 维度 | 文本 Embedding | 多模态向量 |
|------|---------------|-----------|
| **单条文本长度上限** | v3/v4: 8,192 Token；v1/v2: 2,048 Token | qwen3-vl/qwen2.5-vl: 32,000 Token；tongyi 系列/v1: 512–1,024 Token |
| **单次请求行数/条目** | 同步: 10–25 行；批处理: 100,000 行 | 总数 ≤ 20 条（含文本/图片/视频混合） |
| **图片/视频限制** | 不适用 | 图片: ≤ 3–10 MB；视频: ≤ 10–50 MB（因模型而异） |

### 计费方式对比

| 维度 | 文本 Embedding | 多模态向量 |
|------|---------------|-----------|
| **计费单位** | 每千 Token | 按模型计费规则（详见各模型文档） |
| **参考单价** | v3/v4: 0.0005 元/千Token；v1/v2: 0.0007 元/千Token | 详见 [Multimodal-Embedding API详情](../../raw/model-api-reference/multimodal-vector/multimodal-embedding-api-reference.md) |
| **免费额度** | 同步: 50 万–100 万 Token；批处理: 2,000 万 Token（开通后 90 天内） | 详见模型文档 |
| **Batch 折扣** | 通过 [[batch-interfaces-compatible-with-openai]] 可享同步价格 50% 折扣 | 不适用 |

---

## 核心差异说明

### 1. 语义空间的本质区别

**文本 Embedding** 将文本映射到纯文本语义空间，生成的向量仅能与其他文本向量进行有意义的相似度计算。

**多模态向量** 将文本、图片和视频映射到**统一的跨模态语义空间**，不同模态的向量可直接通过余弦相似度进行匹配。这是实现"以文搜图""以图搜视频"等跨模态检索的基础。

> ⚠️ 两类模型生成的向量**不在同一语义空间**，不可混合使用进行相似度计算。

### 2. 融合向量能力

多模态向量独有的**融合向量**机制，可将多种模态的输入编码为单一向量。例如将一件商品的图片和文字描述融合为一个向量，用于商品检索或推荐。文本 Embedding 不具备此能力。

### 3. 批处理与吞吐量

文本 Embedding 提供专用的批处理接口（`text-embedding-async-v1/v2`），单次支持最多 10 万行文本，适合大规模语料库的离线向量化。多模态向量目前仅支持同步调用，大规模处理需自行实现并发控制。

---

## 适用场景建议

### 推荐使用文本 Embedding 的场景

| 场景 | 说明 |
|------|------|
| **纯文本语义搜索** | 如文档检索、FAQ 匹配、知识库问答（RAG） |
| **文本聚类与分类** | 对大量文本进行无监督聚类或特征提取 |
| **大规模离线向量化** | 利用批处理接口高效处理百万级文本语料 |
| **推荐系统（文本特征）** | 基于用户查询和物品描述的语义匹配 |
| **多语言文本匹配** | 跨 100+ 语种的语义对齐（v4） |
| **成本敏感场景** | 纯文本计算成本更低，且有 Batch 折扣 |

👉 详见 [[general-text-embedding]]

### 推荐使用多模态向量的场景

| 场景 | 说明 |
|------|------|
| **跨模态检索** | 以文搜图、以图搜视频、以图搜图 |
| **商品/内容理解** | 将商品图片+描述融合为统一向量表征 |
| **视频语义检索** | 通过文本描述检索视频内容片段 |
| **多模态内容审核** | 图文视频的语义相似度分析 |
| **多模态推荐** | 基于图文混合特征的内容推荐 |

👉 详见 [[multimodal-vector]]

### 选型决策流程

```
输入是否包含图片或视频？
├── 是 → 是否需要跨模态检索或融合表征？
│       ├── 是 → 使用多模态向量
│       └── 否（仅处理文本部分） → 使用文本 Embedding
└── 否（纯文本）
    ├── 是否需要大规模批处理（>1万条）？
    │   ├── 是 → 使用文本 Embedding（批处理接口）
    │   └── 否 → 使用文本 Embedding（同步接口）
    └── 选择 text-embedding-v4 获得最佳性能
```

---

## 组合使用建议

在实际项目中，两类向量也可以**组合使用**：

- **混合检索系统**：对文档内容使用文本 Embedding 建立文本索引，对文档中的图表使用多模态向量建立图文索引，查询时并行检索后融合排序。
- **多阶段管线**：先用文本 Embedding 进行粗筛（成本低、吞吐高），再用多模态向量对候选结果进行精排。
- **RAG + 多模态**：知识库的文本部分用 [[general-text-embedding]] 向量化，图片/视频素材用 [[multimodal-vector]] 向量化，分别存入不同的向量索引。

---

## 相关页面

- [[general-text-embedding]] — 通用文本向量详情
- [[multimodal-vector]] — 多模态向量详情
- [[api-key]] — API Key 获取与配置
- [[install-sdk]] — SDK

## 被对比主题页

- [[general-text-embedding|general text embedding]] — `../api/general-text-embedding.md`
- [[multimodal-vector|multimodal vector]] — `../api/multimodal-vector.md`

