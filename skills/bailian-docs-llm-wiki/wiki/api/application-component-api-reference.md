# Application Component API Reference

阿里云百炼平台（`bailian/2023-12-29`）提供了一套基于 ROA 签名风格的 OpenAPI，涵盖数据连接、知识库和 Prompt 工程三大功能模块。本文汇总了各 API 的功能分类、关键参数、使用方式及限制条件，帮助开发者快速查阅和集成。所有 API 均需通过 [[access-key]] 进行身份认证，推荐使用官方 SDK 简化调用流程。

---

## 服务接入与认证

### 接入点

当前支持两个地域，详见 [服务接入点](../../raw/application-api-reference/application-component-api-reference/api-bailian-2023-12-29-endpoint.md)：

| 地域 | 地域 ID | 公网接入地址 | VPC 接入地址 |
|------|---------|-------------|-------------|
| 华北2（北京） | cn-beijing | `bailian.cn-beijing.aliyuncs.com` | `bailian-vpc.cn-beijing.aliyuncs.com` |
| 新加坡 | ap-southeast-1 | `bailian.ap-southeast-1.aliyuncs.com` | `bailian-vpc.ap-southeast-1.aliyuncs.com` |

### 认证与授权

- **签名方式**：ROA 签名风格，推荐使用官方 SDK 而非自签名（自签名耗时约 5 个工作日）。
- **RAM 权限**：RAM 代码为 `sfm`，授权粒度为**操作级**。RAM 用户需获取对应权限策略并加入 [[workspace]] 后方可调用。
  - 数据类操作通常需要 `AliyunBailianDataFullAccess`
  - 只读操作（如 `DescribeFile`、`GetParseSettings`）可使用 `AliyunBailianDataReadOnlyAccess`
- **安全建议**：避免使用阿里云主账号的 AccessKey，应创建 RAM 用户并遵循最小权限原则。

---

## API 分类总览

根据 [API概览](../../raw/application-api-reference/application-component-api-reference/api-bailian-2023-12-29-overview.md)，全部 API 分为以下三大模块：

### 一、数据连接（原应用数据）

管理文件、类目和连接器，为知识库构建提供数据基础。

#### 类目管理

| API | 说明 | HTTP 方法 | 幂等性 | 限流 |
|-----|------|-----------|--------|------|
| `AddCategory` | 新增类目（每空间最多 500 个） | POST | 否 | 5 次/秒 |
| `ListCategory` | 查询类目列表（支持分页） | POST | 是 | 5 次/秒 |
| `DeleteCategory` | 永久删除类目 | DELETE | 是 | 5 次/秒 |

> **注意**：不支持通过 API 新增或查询数据表，需通过控制台操作。

#### 文件管理

| API | 说明 | HTTP 方法 | 幂等性 | 限流 |
|-----|------|-----------|--------|------|
| `ApplyFileUploadLease` | 申请文件上传租约 | POST | 否 | 10 次/秒 |
| `AddFile` | 将临时存储文件导入数据连接 | PUT | 否 | 10 次/秒 |
| `AddFilesFromAuthorizedOss` | 从已授权 OSS 导入文件 | POST | 否 | 5 次/秒 |
| `ListFile` | 查询文件列表（支持分页） | GET | 是 | 5 次/秒 |
| `DescribeFile` | 查询文件基本信息与状态 | GET | 是 | 10 次/秒 |
| `UpdateFileTag` | 更新单个文件标签 | PUT | — | 5 次/秒 |
| `BatchUpdateFileTag` | 批量更新文件标签 | PUT | — | — |
| `DeleteFile` | 永久删除文件（仅支持已解析/解析失败的文件） | DELETE | 是 | 10 次/秒 |

**文件解析相关**：

| API | 说明 |
|-----|------|
| `GetAvailableParserTypes` | 根据文件扩展名获取支持的解析器列表 |
| `ChangeParseSetting` | 修改类目的文件解析配置 |
| `GetParseSettings` | 查询类目的解析设置 |

`AddFile` 支持的解析器类型包括：`DOCMIND`（智能文档解析）、`DOCMIND_DIGITAL`（电子文档解析）、`DOCMIND_LLM_VERSION`（大模型文档解析）、`DASH_QWEN_VL_PARSER`（Qwen VL 解析）、`DOCMIND_LLM_VERSION_MEDIA`（音视频解析）、`AUTO_SELECT`（自动选择）。

#### 表格与连接器管理

| API | 说明 |
|-----|------|
| `AddTable` | 为表格连接器添加表格 |
| `UpdateTableFromAuthorizedOss` | 从 OSS 文件更新表格 |
| `AddConnector` | 创建连接器（当前仅支持文件类型） |
| `GetConnector` | 获取连接器信息 |

### 二、知识库

管理 RAG 知识库的完整生命周期，包括创建、文件管理、检索和切片操作。

#### 知识库生命周期

| API | 说明 | 幂等性 | 限流 |
|-----|------|--------|------|
| `CreateIndex` | 创建知识库（文档/音视频/结构化） | 否 | 10 次/秒 |
| `SubmitIndexJob` | 提交知识库创建任务 | 否 | 10 次/秒 |
| `GetIndexJobStatus` | 查询创建/追加任务状态 | 是 | 建议间隔 ≥5 秒 |
| `ListIndices` | 查询知识库列表 | 是 | 10 次/秒 |
| `UpdateIndex` | 更新知识库配置 | 是 | — |
| `DeleteIndex` | 永久删除知识库 | 是 | 10 次/秒 |
| `GetIndexMonitor` | 获取知识库监控数据（存储/QPS） | 是 | — |

**典型调用流程**：`CreateIndex` → `SubmitIndexJob` → 轮询 `GetIndexJobStatus` 直至完成。详见 [创建知识库](../../raw/application-api-reference/application-component-api-reference/api-bailian-2023-12-29-dir/api-bailian-2023-12-29-dir-knowledge-base/api-bailian-2023-12-29-createindex.md) 文档。

> **注意**：`CreateIndex` 仅初始化知识库，必须后续调用 `SubmitIndexJob` 才能完成创建，否则将得到一个空知识库。

#### 知识库文件操作

| API | 说明 |
|-----|------|
| `SubmitIndexAddDocumentsJob` | 向已有知识库追加文件（不支持数据查询/图片问答类） |
| `ListIndexDocuments` | 查询知识库中的文件概要 |
| `ListIndexFileDetails` | 查询知识库中的文件详情 |
| `DeleteIndexDocument` | 永久删除知识库中的文件（不可逆） |

#### 检索与切片

| API | 说明 |
|-----|------|
| `Retrieve` | 检索知识库，支持通过 SDK 或 [[spring-ai-alibaba]] 调用 |
| `ListChunks` | 查询指定文件的文本切片列表 |
| `UpdateChunk` | 修改切片内容/标题，设置是否参与检索（仅文档搜索类知识库） |
| `DeleteChunk` | 永久删除切片（硬删除，不可恢复） |

`UpdateIndex` 支持的关键检索参数：
- `DenseSimilarityTopK`：向量检索 Top K，取值 0-100，默认 100
- `SparseSimilarityTopK`：关键词检索 Top K，取值 0-100，默认 100
- 两者之和不超过 200
- `RerankMinScore`：排序最低分数，取值 0-1

### 三、Prompt 工程

| API | 说明 | HTTP 方法 |
|-----|------|-----------|
| `CreatePromptTemplate` | 创建 Prompt 模板（不支持文生图） | POST |
| `GetPromptTemplate` | 获取指定模板详情 | GET |
| `UpdatePromptTemplate` | 增量更新模板 | PATCH |
| `DeletePromptTemplate` | 删除模板 | DELETE |
| `ListPromptTemplates` | 查询模板列表（支持按名称/类型过滤） | GET |

模板内容支持 `${variable}` 变量语法，模板类型分为 `System`（系统预置）和 `Custom`（用户自定义）。

---

## 通用参数与约定

### 路径参数

几乎所有 API 的路径中都包含 `WorkspaceId`（业务空间 ID），这是操作的基本作用域。获取方式参见 [[workspace]] 相关文档。

### 分页机制

列表类接口（如 `ListCategory`、`ListFile`、`ListIndices`）采用两种分页方式：
- **Token 分页**：使用 `MaxResults` + `NextToken`，适用于 `ListCategory`、`ListFile`
- **页码分页**：使用 `PageNumber` + `PageSize`，适用于 `ListIndices`、`ListIndexDocuments`

### 返回结构

所有接口返回统一包含以下字段：
- `RequestId`：请求唯一标识
- `Success`：布尔值，表示是否成功
- `Code`/`Status`：状态码
- `Message`：错误描述信息
- `Data`：业务数据

---

## 限制与注意事项

1. **限流**：各接口限流阈值在 5-15 次/秒之间，遇到限流请增加重试间隔。
2. **OSS 导入限制**：OSS Bucket 需与百炼同属一个主账号，不支持归档/冷归档/深度冷归档存储类型。
3. **删除操作不可逆**：`DeleteIndex`、`DeleteIndexDocument`、`DeleteChunk` 均为硬删除。删除知识库文件不影响数据连接中的原始文件，反之亦然。
4. **数据表操作限制**：数据表的新增、查询和删除均不支持 API，需通过控制台操作。
5. **知识库类型限制**：`SubmitIndexAddDocumentsJob`、`UpdateChunk` 不支持数据查询/图片问答类知识库。
6. **标签限制**：单文件最多 100 个标签，所有标签总字符长度不超过 700，单个标签最多 32 个字符。
7. **版本变更**：API 持续更新中，如 `UpdateIndex`（2026-01-19 新增）、`GetIndexMonitor`（2026-01-14 新增），建议关注 [[api-changelog]] 获取最新变更。

## 来源文档

- [API概览](../../raw/application-api-reference/application-component-api-reference/api-bailian-2023-12-29-overview.md)
- [服务接入点](../../raw/application-api-reference/application-component-api-reference/api-bailian-2023-12-29-endpoint.md)
- [授权信息](../../raw/application-api-reference/application-component-api-reference/api-bailian-2023-12-29-ram.md)
- [版本说明](../../raw/application-api-reference/application-component-api-reference/api-bailian-2023-12-29-changeset.md)
- [AddCategory - 新增类目](../../raw/application-api-reference/application-component-api-reference/api-bailian-2023-12-29-dir/api-bailian-2023-12-29-dir-data-connection-original-application-data/api-bailian-2023-12-29-addcategory.md)
- [ListCategory - 类目列表](../../raw/application-api-reference/application-component-api-reference/api-bailian-2023-12-29-dir/api-bailian-2023-12-29-dir-data-connection-original-application-data/api-bailian-2023-12-29-listcategory.md)
- [DeleteCategory - 删除类目](../../raw/application-api-reference/application-component-api-reference/api-bailian-2023-12-29-dir/api-bailian-2023-12-29-dir-data-connection-original-application-data/api-bailian-2023-12-29-deletecategory.md)
- [ApplyFileUploadLease - 申请文件上传租约](../../raw/application-api-reference/application-component-api-reference/api-bailian-2023-12-29-dir/api-bailian-2023-12-29-dir-data-connection-original-application-data/api-bailian-2023-12-29-applyfileuploadlease.md)
- [AddFile - 添加文件](../../raw/application-api-reference/application-component-api-reference/api-bailian-2023-12-29-dir/api-bailian-2023-12-29-dir-data-connection-original-application-data/api-bailian-2023-12-29-addfile.md)
- [AddFilesFromAuthorizedOss - 从已授权OSS Bucket中导入文件](../../raw/application-api-reference/application-component-api-reference/api-bailian-2023-12-29-dir/api-bailian-2023-12-29-dir-data-connection-original-application-data/api-bailian-2023-12-29-addfilesfromauthorizedoss.md)
- [ListFile - 文件列表](../../raw/application-api-reference/application-component-api-reference/api-bailian-2023-12-29-dir/api-bailian-2023-12-29-dir-data-connection-original-application-data/api-bailian-2023-12-29-listfile.md)
- [UpdateFileTag - 更新文件标签](../../raw/application-api-reference/application-component-api-reference/api-bailian-2023-12-29-dir/api-bailian-2023-12-29-dir-data-connection-original-application-data/api-bailian-2023-12-29-updatefiletag.md)
- [DescribeFile - 查询文件状态](../../raw/application-api-reference/application-component-api-reference/api-bailian-2023-12-29-dir/api-bailian-2023-12-29-dir-data-connection-original-application-data/api-bailian-2023-12-29-describefile.md)
- [BatchUpdateFileTag - 批量更新文档标签](../../raw/application-api-reference/application-component-api-reference/api-bailian-2023-12-29-dir/api-bailian-2023-12-29-dir-data-connection-original-application-data/api-bailian-2023-12-29-batchupdatefiletag.md)
- [DeleteFile - 删除文件](../../raw/application-api-reference/application-component-api-reference/api-bailian-2023-12-29-dir/api-bailian-2023-12-29-dir-data-connection-original-application-data/api-bailian-2023-12-29-deletefile.md)
- [GetAvailableParserTypes - 获取文件支持的解析器类型](../../raw/application-api-reference/application-component-api-reference/api-bailian-2023-12-29-dir/api-bailian-2023-12-29-dir-data-connection-original-application-data/api-bailian-2023-12-29-getavailableparsertypes.md)
- [ChangeParseSetting - 修改类目解析设置](../../raw/application-api-reference/application-component-api-reference/api-bailian-2023-12-29-dir/api-bailian-2023-12-29-dir-data-connection-original-application-data/api-bailian-2023-12-29-changeparsesetting.md)
- [GetParseSettings - 获取类目解析设置](../../raw/application-api-reference/application-component-api-reference/api-bailian-2023-12-29-dir/api-bailian-2023-12-29-dir-data-connection-original-application-data/api-bailian-2023-12-29-getparsesettings.md)
- [AddTable - 添加表格](../../raw/application-api-reference/application-component-api-reference/api-bailian-2023-12-29-dir/api-bailian-2023-12-29-dir-data-connection-original-application-data/api-bailian-2023-12-29-addtable.md)
- [UpdateTableFromAuthorizedOss - 从已授权OSS Bucket中选择文件更新表格](../../raw/application-api-reference/application-component-api-reference/api-bailian-2023-12-29-dir/api-bailian-2023-12-29-dir-data-connection-original-application-data/api-bailian-2023-12-29-updatetablefromauthorizedoss.md)
- [GetConnector - 获取连接器信息](../../raw/application-api-reference/application-component-api-reference/api-bailian-2023-12-29-dir/api-bailian-2023-12-29-dir-data-connection-original-application-data/api-bailian-2023-12-29-getconnector.md)
- [AddConnector - 新增连接器](../../raw/application-api-reference/application-component-api-reference/api-bailian-2023-12-29-dir/api-bailian-2023-12-29-dir-data-connection-original-application-data/api-bailian-2023-12-29-addconnector.md)
- [GetIndexJobStatus - 查询知识库创建任务状态](../../raw/application-api-reference/application-component-api-reference/api-bailian-2023-12-29-dir/api-bailian-2023-12-29-dir-knowledge-base/api-bailian-2023-12-29-getindexjobstatus.md)
- [CreateIndex - 创建知识库](../../raw/application-api-reference/application-component-api-reference/api-bailian-2023-12-29-dir/api-bailian-2023-12-29-dir-knowledge-base/api-bailian-2023-12-29-createindex.md)
- [SubmitIndexJob - 提交知识库创建任务](../../raw/application-api-reference/application-component-api-reference/api-bailian-2023-12-29-dir/api-bailian-2023-12-29-dir-knowledge-base/api-bailian-2023-12-29-submitindexjob.md)
- [SubmitIndexAddDocumentsJob - 提交知识库追加任务](../../raw/application-api-reference/application-component-api-reference/api-bailian-2023-12-29-dir/api-bailian-2023-12-29-dir-knowledge-base/api-bailian-2023-12-29-submitindexadddocumentsjob.md)
- [Retrieve - 检索知识库](../../raw/application-api-reference/application-component-api-reference/api-bailian-2023-12-29-dir/api-bailian-2023-12-29-dir-knowledge-base/api-bailian-2023-12-29-retrieve.md)
- [ListIndexDocuments - 查询知识库下的文件列表](../../raw/application-api-reference/application-component-api-reference/api-bailian-2023-12-29-dir/api-bailian-2023-12-29-dir-knowledge-base/api-bailian-2023-12-29-listindexdocuments.md)
- [ListIndexFileDetails - 查询知识库下的文件详情](../../raw/application-api-reference/application-component-api-reference/api-bailian-2023-12-29-dir/api-bailian-2023-12-29-dir-knowledge-base/api-bailian-2023-12-29-listindexfiledetails.md)
- [UpdateIndex - 更新知识库](../../raw/application-api-reference/application-component-api-reference/api-bailian-2023-12-29-dir/api-bailian-2023-12-29-dir-knowledge-base/api-bailian-2023-12-29-updateindex.md)
- [DeleteIndexDocument - 删除知识库下的文件](../../raw/application-api-reference/application-component-api-reference/api-bailian-2023-12-29-dir/api-bailian-2023-12-29-dir-knowledge-base/api-bailian-2023-12-29-deleteindexdocument.md)
- [ListIndices - 查询知识库列表](../../raw/application-api-reference/application-component-api-reference/api-bailian-2023-12-29-dir/api-bailian-2023-12-29-dir-knowledge-base/api-bailian-2023-12-29-listindices.md)
- [DeleteIndex - 删除知识库](../../raw/application-api-reference/application-component-api-reference/api-bailian-2023-12-29-dir/api-bailian-2023-12-29-dir-knowledge-base/api-bailian-2023-12-29-deleteindex.md)
- [UpdateChunk - 修改切片](../../raw/application-api-reference/application-component-api-reference/api-bailian-2023-12-29-dir/api-bailian-2023-12-29-dir-knowledge-base/api-bailian-2023-12-29-updatechunk.md)
- [ListChunks - 查询索引下的分片列表](../../raw/application-api-reference/application-component-api-reference/api-bailian-2023-12-29-dir/api-bailian-2023-12-29-dir-knowledge-base/api-bailian-2023-12-29-listchunks.md)
- [GetIndexMonitor - 获取知识库监控数据](../../raw/application-api-reference/application-component-api-reference/api-bailian-2023-12-29-dir/api-bailian-2023-12-29-dir-knowledge-base/api-bailian-2023-12-29-getindexmonitor.md)
- [DeleteChunk - 删除切片](../../raw/application-api-reference/application-component-api-reference/api-bailian-2023-12-29-dir/api-bailian-2023-12-29-dir-knowledge-base/api-bailian-2023-12-29-deletechunk.md)
- [CreatePromptTemplate - 创建Prompt模板](../../raw/application-api-reference/application-component-api-reference/api-bailian-2023-12-29-dir/api-bailian-2023-12-29-dir-[[prompt|prompt]]-engineering/api-bailian-2023-12-29-create[[prompt|prompt]]template.md)
- [GetPromptTemplate - 获取Prompt模板](../../raw/application-api-reference/application-component-api-reference/api-bailian-2023-12-29-dir/api-bailian-2023-12-29-dir-[[prompt|prompt]]-engineering/api-bailian-2023-12-29-getprompttemplate.md)
- [DeletePromptTemplate - 删除Prompt模板](../../raw/application-api-reference/application-component-api-reference/api-bailian-2023-12-29-dir/api-bailian-2023-12-29-dir-prompt-engineering/api-bailian-2023-12-29-deleteprompttemplate.md)
- [UpdatePromptTemplate - 更新Prompt模板](../../raw/application-api-reference/application-component-api-reference/api-bailian-2023-12-29-dir/api-bailian-2023-12-29-dir-prompt-engineering/api-bailian-2023-12-29-updateprompttemplate.md)
- [ListPromptTemplates - 获取Prompt模板列表](../../raw/application-api-reference/application-component-api-reference/api-bailian-2023-12-29-dir/api-bailian-2023-12-29-dir-prompt-engineering/api-bailian-2023-12-29-listprompttemplates.md)
- [ApplyTempStorageLease - 申请临时文件上传许可](../../raw/application-api-reference/application-component-api-reference/api-bailian-2023-12-29-dir/api-bailian-2023-12-29-dir-others/api-bailian-2023-12-29-applytempstoragelease.md)
- [GetAlipayUrl - 获取支付宝打赏URL](../../raw/application-api-reference/application-component-api-reference/api-bailian-2023-12-29-dir/api-bailian-2023-12-29-dir-others/api-bailian-2023-12-29-getalipayurl.md)
- [GetAlipayTransferStatus - 查询支付宝打赏状态](../../raw/application-api-reference/application-component-api-reference/api-bailian-2023-12-29-dir/api-bailian-2023-12-29-dir-others/api-bailian-2023-12-29-getalipaytransferstatus.md)
- [GetMemory - 获取长期记忆体](../../raw/application-api-reference/application-component-api-reference/api-bailian-2023-12-29-dir/api-bailian-2023-12-29-dir-others/api-bailian-2023-12-29-dir-long-term-memory/api-bailian-2023-12-29-getmemory.md)
- [CreateMemory - 创建长期记忆体](../../raw/application-api-reference/application-component-api-reference/api-bailian-2023-12-29-dir/api-bailian-2023-12-29-dir-others/api-bailian-2023-12-29-dir-long-term-memory/api-bailian-2023-12-29-creatememory.md)
- [DeleteMemory - 删除长期记忆体](../../raw/application-api-reference/application-component-api-reference/api-bailian-2023-12-29-dir/api-bailian-2023-12-29-dir-others/api-bailian-2023-12-29-dir-long-term-memory/api-bailian-2023-12-29-deletememory.md)
- [UpdateMemory - 更新长期记忆体](../../raw/application-api-reference/application-component-api-reference/api-bailian-2023-12-29-dir/api-bailian-2023-12-29-dir-others/api-bailian-2023-12-29-dir-long-term-memory/api-bailian-2023-12-29-updatememory.md)
- [ListMemories - 获取长期记忆体列表](../../raw/application-api-reference/application-component-api-reference/api-bailian-2023-12-29-dir/api-bailian-2023-12-29-dir-others/api-bailian-2023-12-29-dir-long-term-memory/api-bailian-2023-12-29-listmemories.md)
- [GetMemoryNode - 获取记忆片段](../../raw/application-api-reference/application-component-api-reference/api-bailian-2023-12-29-dir/api-bailian-2023-12-29-dir-others/api-bailian-2023-12-29-dir-long-term-memory/api-bailian-2023-12-29-getmemorynode.md)
- [CreateMemoryNode - 创建记忆片段](../../raw/application-api-reference/application-component-api-reference/api-bailian-2023-12-29-dir/api-bailian-2023-12-29-dir-others/api-bailian-2023-12-29-dir-long-term-memory/api-bailian-2023-12-29-creatememorynode.md)
- [DeleteMemoryNode - 删除记忆片段](../../raw/application-api-reference/application-component-api-reference/api-bailian-2023-12-29-dir/api-bailian-2023-12-29-dir-others/api-bailian-2023-12-29-dir-long-term-memory/api-bailian-2023-12-29-deletememorynode.md)
- [UpdateMemoryNode - 更新记忆片段](../../raw/application-api-reference/application-component-api-reference/api-bailian-2023-12-29-dir/api-bailian-2023-12-29-dir-others/api-bailian-2023-12-29-dir-long-term-memory/api-bailian-2023-12-29-updatememorynode.md)
- [ListMemoryNodes - 获取记忆片段列表](../../raw/application-api-reference/application-component-api-reference/api-bailian-2023-12-29-dir/api-bailian-2023-12-29-dir-others/api-bailian-2023-12-29-dir-long-term-memory/api-bailian-2023-12-29-listmemorynodes.md)

