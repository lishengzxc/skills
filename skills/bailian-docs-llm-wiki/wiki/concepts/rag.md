# 检索增强生成（RAG）

检索增强生成（Retrieval-Augmented Generation, RAG）是一种在大模型生成回答前，先从外部知识源中检索相关内容并注入上下文的技术范式，用于解决大模型在私有知识、实时信息等方面的不足，提升回答的准确性和可靠性。

## 核心原理

RAG 的基本流程分为三个阶段：

1. **索引（Indexing）**：将文档切片并通过向量模型（如 `text-embedding-v4`）转为数值向量，存入向量数据库
2. **检索（Retrieval）**：用户提问时，将查询同样向量化，通过语义相似度从知识库中召回相关文本切片
3. **生成（Generation）**：将召回的切片作为上下文注入 Prompt，由大模型基于这些参考信息生成回答

在检索与生成之间，通常还会引入 **Rerank（重排序）** 步骤，使用排序模型（如 `qwen3-rerank`）对初步召回结果做二次精排，提升最终送入模型的内容质量。

## 在百炼平台中的使用场景

### 1. 知识库集成（云端 RAG）

百炼平台的 [[knowledge-base]] 是 RAG 的核心载体。通过控制台或 API 创建知识库后，可集成到不同类型的应用中：

| 集成方式 | 说明 |
|---------|------|
| **智能体应用** | 在 [[llm-application]] 配置中添加知识库。新版 Agent 2.0 将知识库统一为工具，由智能体自主决策何时调用 |
| **工作流应用** | 在工作流中拖入知识库节点，支持固定知识库或通过 `CodeList` 变量动态引入 |
| **外部应用** | 通过 [[bailian-sdk]] 调用 `Retrieve` API 检索知识库，自行拼装 Prompt |

### 2. 框架集成

通过 [[frameworks]] 快速构建 RAG 应用：

- **LlamaIndex（Python）**：使用 `DashScopeCloudIndex.from_documents()` 上传文档创建云端知识库，通过 `query_engine` 实现检索问答
- **Spring AI Alibaba（Java）**：通过 `DashScopeDocumentRetriever` 检索百炼云端知识库，结合 `ChatClient` 和 `DocumentRetrievalAdvisor` 实现 RAG 问答

### 3. 本地 RAG

使用本地嵌入模型和向量存储构建本地知识库，适合对数据安全有严格要求或需要自定义切分策略的场景。支持 PDF、DOCX、TXT、XLSX、CSV 等格式。详见 [[application-use-cases]]。

### 4. 文件问答

智能体应用中的 **切片检索模式** 本质上是会话级 RAG：上传的文件被实时切片和向量化，用户针对文件内容提问时通过检索定位相关片段。适合长文档问答和精确信息定位。

## 关键参数与配置

### 索引阶段

| 参数 | 说明 | 注意事项 |
|------|------|---------|
| **向量模型** | `text-embedding-v4`（支持 64–2048 维）、`text-embedding-v3`（支持 64–1024 维） | 创建后维度不可更改。详见 [[general-text-embedding]] |
| **切片方式** | 智能切分（推荐）或按长度切分，最大 6,000 Token/片 | 创建后不可更改 |
| **Meta 信息抽取** | 为切片附加元数据（key-value），提升检索精度 | 创建后无法再配置 |
| **多轮对话改写** | 根据历史对话自动补全用户查询，解决指代消解问题 | 创建后无法再开启 |
| **text_type** | 区分 `query`（查询文本）和 `document`（底库文本），用于非对称检索场景 | 仅部分接口支持 |

### 检索阶段

| 参数 | 说明 | 建议值 |
|------|------|--------|
| **相似度阈值** | 仅语义相似度高于此值的切片才被召回 | 需通过命中测试反复调试，过高会丢弃相关内容 |
| **TopK（召回片段数）** | 最终返回给大模型的知识片段数量 | 1–20，增大可提升准确性但增加 Token 消耗 |
| **初步向量检索 TopK** | 语义相似性初步召回数量 | 默认 50，降低可减少 Rerank 费用 |
| **初步关键词检索 TopK** | 文本匹配初步召回数量 | 默认 50 |
| **权重** | 多知识库场景下干预召回顺序 | 按信息源重要程度分配，仅同类型知识库间生效 |

### 重排序（Rerank）

使用 [[more-models]] 中的排序模型对初步召回结果做精排：

| 模型 | 适用场景 | 最大文档数 |
|------|---------|-----------|
| `qwen3-rerank` | 文本语义检索、RAG | 500 |
| `qwen3-vl-rerank` | 跨模态搜索（文本+图片+视频） | 文本 100 / 图片 40 / 视频 4 |

关键参数：`top_n`（返回前

## 关联主题页

- [[knowledge-base|knowledge base]] — `../guides/knowledge-base.md`
- [[frameworks|frameworks]] — `../api/[[frameworks|frameworks]].md`
- [[llm-application|llm application]] — `../guides/llm-application.md`
- [[application-use-cases|application use cases]] — `../guides/application-use-cases.md`
- [[use-cases|use cases]] — `../guides/use-cases.md`
- [[more-models|[[more|more]] models]] — `../api/[[more|more]]-models.md`
- [[general-text-embedding|general text embedding]] — `../api/general-text-embedding.md`


