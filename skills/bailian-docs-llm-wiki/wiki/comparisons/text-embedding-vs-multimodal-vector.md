# 文本Embedding与多模态向量对比

## 概述

在构建语义搜索、推荐系统或内容理解等 AI 应用时，开发者通常需要将非结构化数据转换为向量表示。百炼平台提供了两类向量化能力：**通用文本向量（General Text Embedding）** 和 **多模态向量（Multimodal Embedding）**。两者在输入模态、模型架构、API 设计和适用场景上存在显著差异。本文从多个关键维度进行系统对比，帮助开发者根据业务需求做出合理的技术选型。

---

## 核心维度对比

| 对比维度 | 通用文本向量（General Text Embedding） | 多模态向量（Multimodal Embedding） |
|---------|---------------------------------------|-----------------------------------|
| **输入格式** | 纯文本（字符串、字符串列表、文本文件） | 文本、图片（URL/Base64）、视频（URL）、多图列表，支持混合输入 |
| **输出格式** | 每条文本对应一个浮点数向量（`float` 数组） | 每个输入对应一个独立向量，或所有输入融合为一个向量 |
| **向量类型** | 仅独立向量（每条文本→一个向量） | 独立向量 + 融合向量（多模态内容融合为统一表征），具体取决于模型 |
| **代表模型** | text-embedding-v4（Qwen3-Embedding）、text-embedding-v3、v2、v1 | qwen3-vl-embedding、qwen2.5-vl-embedding、tongyi-embedding-vision 系列、multimodal-embedding-v1 |
| **向量维度** | 64–2048 可选（v4/v3 支持灵活维度），v1/v2 固定 1536 | 64–2560 可选（因模型而异），部分旧模型固定维度 |
| **最大输入长度** | 单行最大 8,192 Token（v3/v4）；2,048 Token（v1/v2） | 文本最大 32K Token（qwen3/2.5-vl）；1K Token（tongyi 系列）；图片≤5–10MB；视频≤50MB |
| **批量能力** | 同步：10–25 行/次；批处理：最多 100,000 行/次 | 单次请求包含多个 content 对象，无批处理异步接口 |
| **API 端点** | 同步：`https://dashscope.aliyuncs.com/compatible-mode/v1`（OpenAI 兼容）；批处理：DashScope 异步接口 | `POST https://dashscope.aliyuncs.com/api/v1/services/embeddings/multimodal-embedding/multimodal-embedding`（DashScope 原生） |
| **SDK 兼容性** | 同步接口兼容 OpenAI SDK（Python/Java/curl） | 使用 DashScope SDK 或直接 HTTP 调用，不兼容 OpenAI SDK |
| **计费方式** | 按 Token 计费；同步 v3/v4 各 100 万免费 Token，v1/v2 各 50 万；批处理各 2000 万免费 Token（开通后 90 天有效） | 按 Token/图片/视频时长等多维度计费（具体参见各模型定价页） |
| **语种支持** | v4：100+ 语种及编程语言；v3：50+ 语种；v1/v2：6–10 种语种 | 主要支持中英文及常见语种，侧重视觉内容理解 |
| **典型场景** | 文本语义搜索、RAG 知识库构建、文本聚类/分类、推荐系统 | 以文搜图、以图搜视频、跨模态检索、商品图文融合检索、视频内容理解 |

---

## 输入与输出对比详解

### 输入差异

| 特性 | 文本Embedding | 多模态向量 |
|------|--------------|-----------|
| 纯文本 | ✅ 核心支持 | ✅ 支持 |
| 图片 | ❌ 不支持 | ✅ URL 或 Base64 Data URI |
| 视频 | ❌ 不支持 | ✅ 公开可访问 URL |
| 多图列表 | ❌ 不支持 | ✅ 部分模型支持 `multi_images` |
| 文件批量输入 | ✅ 批处理接口支持（HTTP URL 文本文件） | ❌ 不支持文件批量模式 |
| query/document 区分 | ✅ 批处理接口及 DashScope 原生接口支持 `text_type` | ❌ 无此区分，可通过 `instruct` 参数引导 |

### 输出差异

| 特性 | 文本Embedding | 多模态向量 |
|------|--------------|-----------|
| 独立向量 | ✅ 每条文本一个向量 | ✅ 每个 content 对象一个向量 |
| 融合向量 | ❌ 不支持 | ✅ 多模态内容融合为单一向量（qwen3-vl、qwen2.5-vl、2026-03-06 版本） |
| 维度可调 | ✅ v3/v4 支持 `dimensions` 参数 | ✅ 多数新模型支持 `dimension` 参数 |
| 跨模态可比 | — 仅文本间可比 | ✅ 所有模态向量位于同一语义空间，可直接计算相似度 |

---

## 适用场景建议

### 选择文本Embedding的场景

- **纯文本语义搜索**：构建 RAG（检索增强生成）知识库，对文档、FAQ、客服语料等进行向量化检索。
- **大规模文本批处理**：需要一次性处理数万至十万条文本，批处理接口（单次 10 万行）是更高效的选择。
- **多语种文本处理**：text-embedding-v4 支持 100+ 语种及编程语言，适合国际化和代码搜索场景。
- **OpenAI 生态迁移**：同步接口完全兼容 OpenAI SDK，已有 OpenAI Embedding 调用代码可低成本迁移。
- **成本敏感的纯文本任务**：文本 Embedding 模型在纯文本场景下通常成本更低、速度更快。

### 选择多模态向量的场景

- **跨模态检索**：以文搜图、以图搜视频、以图搜图等涉及多种模态的语义搜索。
- **图文融合表征**：电商商品（图片 + 描述）、新闻内容（配图 + 标题）等需要将多模态信息融合为统一向量的场景。
- **视频内容理解**：对视频进行语义向量化，支持视频检索、分类、推荐。
- **视觉内容分类与聚类**：基于图片或视频的语义进行智能分组和标签分析。
- **多模态 RAG**：检索增强生成中需要同时检索文本和图像资料的场景。

---

## 技术选型参考

```
是否涉及图片/视频输入？
├── 否 → 使用文本Embedding
│   ├── 数据量 ≤ 数十条，实时响应 → 同步接口（text-embedding-v4 推荐）
│   ├── 数据量达万级以上，离线处理 → 批处理接口（text-embedding-async-v2）
│   └── 需兼容 OpenAI SDK → 同步接口（OpenAI 兼容模式）
│
└── 是 → 使用多模态向量
    ├── 需要融合向量（图文合一表征） → qwen3-vl-embedding（enable_fusion=true）或 qwen2.5-vl-embedding
    ├── 仅需独立向量（逐项对比） → tongyi-embedding-vision-plus/flash 或 qwen3-vl-embedding
    ├── 需处理视频内容 → qwen3-vl-embedding 或 tongyi-embedding-vision 系列
    └── 追求性价比 → tongyi-embedding-vision-flash 系列
```

### 关键决策因素总结

| 决策因素 | 推荐方案 |
|---------|---------|
| 仅处理文本，追求高质量多语种向量 | text-embedding-v4 |
| 仅处理文本，需大规模批量处理 | text-embedding-async-v2（批处理接口） |
| 需要跨模态检索能力 | qwen3-vl-embedding 或 tongyi-embedding-vision-plus |
| 需要图文融合为单一向量 | qwen3-vl-embedding（融合模式）或 qwen2.5-vl-embedding |
| 已有 OpenAI SDK 代码需迁移 | text-embedding-v4（同步接口，OpenAI 兼容） |
| 纯文本场景，控制成本 | text-embedding-v3 或 v2（免费额度较充足） |
| 视频语义理解与检索 | qwen3-vl-embedding |

---

## 注意事项

1. **向量空间不互通**：文本Embedding和多模态向量生成的向量位于不同的语义空间，不可混合使用或直接比较相似度。即使维度相同，也不应将两类模型的输出放入同一向量索引。

2. **维度一致性**：同一向量索引中的所有向量必须来自同一模型、同一维度设置。切换模型或维度后需重新生成全部向量。

3. **API 风格差异**：文本Embedding 同步接口兼容 OpenAI 格式（参数名为 `dimensions`），多模态向量使用 DashScope 原生格式（参数名为 `dimension`），注意参数命名差异。

4. **批处理能力**：仅文

## 被对比主题页

- [general text embedding](../api/general-text-embedding.md)
- [multimodal vector](../api/multimodal-vector.md)

