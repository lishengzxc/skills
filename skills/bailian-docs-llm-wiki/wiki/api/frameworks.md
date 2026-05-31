# frameworks

阿里云百炼平台支持通过主流开发框架集成其大模型服务和数据管理能力，当前主要支持 **LlamaIndex**（Python）和 **Spring AI Alibaba**（Java）两个框架。开发者可以根据技术栈选择合适的框架，快速构建 RAG 应用、集成智能体应用或检索云端知识库。

## 支持的框架与适用场景

| 框架 | 语言 | 主要场景 | 环境要求 |
|------|------|----------|----------|
| LlamaIndex | Python | 构建云端 RAG 应用（知识库问答） | Python 3.9+ |
| Spring AI Alibaba | Java | 集成智能体/工作流应用、检索云端知识库 | JDK 17+, Spring Boot 3.x |

## LlamaIndex（Python）

LlamaIndex 方案通过 `DashScopeCloudIndex` 将本地文件上传至百炼云端，构建知识库并提供检索引擎。详细步骤参见 [通过LlamaIndex API构建RAG应用](../../raw/application-api-reference/frameworks/llamaindex.md)。

### 核心流程

1. **构建云端知识库**：使用 `DashScopeParse` 解析本地文件（支持 `.txt`、`.docx`、`.pdf`），通过 `DashScopeCloudIndex.from_documents()` 上传并创建知识库。
2. **构建检索引擎**：基于已有知识库创建 `query_engine`，结合相似度过滤和重排（`DashScopeRerank`）返回高质量结果。

### 关键参数

```python
Settings.llm = DashScope(model_name="qwen-max")  # 生成回答的模型
similarity_top_k = 5        # 检索结果数量
similarity_cutoff = 0.4     # 最低相似度阈值
top_n = 1                   # 重排后返回的结果数
```

`model_name` 支持 `qwen-max` 等[[models]]，具体可用模型请参考文本生成模型列表。

### 限制

- 使用默认的智能文档切分与官方向量模型，**不支持**自定义文档切分方式或自定义嵌入模型。
- 如需本地知识库或自定义嵌入模型，需采用其他方案。

## Spring AI Alibaba（Java）

Spring AI Alibaba 提供两类集成能力：调用百炼大模型应用（智能体/工作流）和检索百炼云端知识库。

### 集成大模型应用

通过 `DashScopeAgent` 调用百炼平台上创建的[[agent-application]]或[[workflow-application]]，支持非流式和流式两种调用方式。详见 [使用Spring AI Alibaba集成阿里云百炼大模型应用](../../raw/application-api-reference/frameworks/spring-ai-alibaba/spring-ai-alibaba-integrate-llm-application.md)。

**核心配置**（`application.yml`）：

```yaml
spring:
  ai:
    dashscope:
      agent:
        app-id: ${APP_ID}
      api-key: ${DASHSCOPE_API_KEY}
      # workspace-id: ${WORKSPACE_ID}  # 子业务空间时需要
```

**调用方式**：
- **非流式**：`agent.call(new Prompt(message, options))` → 返回 `ChatResponse`
- **流式**：`agent.stream(new Prompt(message, options))` → 返回 `Flux<ChatResponse>`，需设置 `withIncrementalOutput(true)`

> **注意**：仅支持集成智能体应用和工作流应用，不支持其他类型的百炼应用。

### 检索云端知识库

通过 `DashScopeDocumentRetriever` 检索百炼平台上已有的知识库，结合 `ChatClient` 实现 RAG 问答。详见 [通过Spring AI Alibaba检索阿里云百炼知识库](../../raw/application-api-reference/frameworks/spring-ai-alibaba/spring-ai-alibaba-integrate-knowledge-base.md)。

核心代码：

```java
DocumentRetriever retriever = new DashScopeDocumentRetriever(dashscopeApi,
    DashScopeDocumentRetrieverOptions.builder().withIndexName("知识库名称").build());

ChatClient chatClient = builder
    .defaultAdvisors(new DocumentRetrievalAdvisor(retriever, systemTemplate))
    .build();
```

知识库需提前在百炼控制台创建，默认使用 `qwen-max` 模型生成回答，可通过 `DashScopeChatOptions` 切换为其他模型。

> **注意**：两篇 Spring AI Alibaba 文档中推荐的 API Key 环境变量名不同：集成应用文档使用 `DASHSCOPE_API_KEY`，检索知识库文档使用 `AI_DASHSCOPE_API_KEY`。业务空间 ID 的环境变量名同样不一致（`WORKSPACE_ID` vs `AI_DASHSCOPE_WORKSPACE_ID`）。请根据各自 `application.yml` 中的实际引用保持一致即可。

## 通用前提条件

无论使用哪个框架，都需要：

1. 开通百炼服务并获取 [[api-key]]
2. 将 API Key 配置到环境变量，避免硬编码泄露
3. 确保本地可以访问公网

## 计费说明

框架本身不收费，但通过框架调用模型会产生[[billing]]相关费用。

## 来源文档

- [通过LlamaIndex API构建RAG应用](../../raw/application-api-reference/frameworks/llamaindex.md)
- [使用Spring AI Alibaba集成阿里云百炼大模型应用](../../raw/application-api-reference/frameworks/spring-ai-alibaba/spring-ai-alibaba-integrate-llm-application.md)
- [通过Spring AI Alibaba检索阿里云百炼知识库](../../raw/application-api-reference/frameworks/spring-ai-alibaba/spring-ai-alibaba-integrate-knowledge-base.md)

