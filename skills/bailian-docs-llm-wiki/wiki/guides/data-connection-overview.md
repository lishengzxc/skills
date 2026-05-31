# data connection overview

数据连接是阿里云百炼平台管理外部数据源的统一入口。通过创建数据连接器，百炼应用可以安全地访问企业数据库、文档系统和对象存储中的数据，在对话中实时查询和引用这些数据。数据连接器按存储和访问方式分为**平台托管**和**流处理**两大类。

## 连接器类型

根据 [数据连接](../../raw/application-user-guide/data-connection-overview/data-connection.md) 文档，平台支持以下连接器类型：

### 平台托管类型

| 连接器类型 | 数据存储方式 | 适用场景 |
|-----------|-------------|---------|
| 文件 | 百炼平台或自有 OSS | 管理非结构化文档（PDF、Word、Markdown 等） |
| 表格 | 百炼平台或自有 OSS | 导入和查询结构化表格数据（CSV、Excel 等） |

### 流处理类型

| 连接器类型 | 数据存储方式 | 适用场景 |
|-----------|-------------|---------|
| MySQL | 数据保留在原数据库，实时访问 | 连接 MySQL 数据库，执行 SQL 查询（仅 DMS 导入方式支持） |
| PostgreSQL | 数据保留在原数据库，实时访问 | 连接 PostgreSQL 数据库，执行 SQL 查询（仅 DMS 导入方式支持） |
| PolarDB-X 2.0 | 数据保留在原数据库，实时访问 | 连接阿里云 PolarDB-X 2.0 分布式数据库 |
| 语雀 | 数据保留在语雀，实时访问 | 访问语雀文档和知识库 |
| OSS | 数据保留在 OSS，实时访问 | 访问对象存储中的文件 |

## 前置条件

- **账号权限**：主账号或具有数据连接管理权限的 RAM 用户。RAM 用户需主账号授权，详见 [[permission-management]]。
- **数据源准备**（按类型）：
  - **MySQL**：已有 RDS 或自建实例，网络可达
  - **PostgreSQL**：已有实例，`wal_level` 设为 `logical`
  - **PolarDB-X 2.0**：已有阿里云实例，所在地域支持私网访问
  - **语雀**：仅支持公网版本语雀，需获取个人访问 Token
  - **OSS**：已创建 Bucket，已开通向量检索服务

## 关键配置参数

### 数据库连接器对比

| 差异项 | MySQL | PostgreSQL | PolarDB-X 2.0 |
|--------|-------|------------|----------------|
| 默认端口 | 3306 | 5432 | 自动获取 |
| 网络类型 | 公网/私网 | 公网/私网 | 仅私网 |
| 额外必填字段 | 无 | dbName | 无 |
| 连通性检测服务 | EventBridge | DTS | EventBridge |
| 特殊配置 | 无 | `wal_level=logical` | 需 DTS + PolarDB-X SLR 授权 |
| 支持自建数据库 | 是 | 是 | 否 |

### OSS 相关标签配置

根据 [数据连接](../../raw/application-user-guide/data-connection-overview/data-connection.md) 中的说明，不同场景 Bucket 需添加不同标签：

- **文件/表格连接器使用自有 OSS**：添加 `bailian-connector-access` 标签，值为 `ReadAndWrite`
- **OSS 连接器**：添加 `bailian-datahub-access` 标签，值为 `read`

## 数据导入

### 文件导入

文件连接器支持多种解析方式：

- **电子文档解析**：不支持解析插图与图表
- **文档智能解析**：识别插图中文本并生成摘要
- **大模型文档解析**：支持对插图和图表内容提问，需使用支持的模型，详见 [[knowledge-base]]
- **Qwen VL 解析**：仅图片格式，可通过 Prompt 指定识别内容
- **音视频解析**：语音识别 + 视频帧提取 + 剧情解析

### 表格导入

支持两种建表方式：
- **直接上传 Excel**：自动识别表头创建表结构
- **自定义表头**：手动配置列名、描述、类型

> **注意**：数据表结构（列名、描述、类型）一旦确定无法修改。上传文件的列数和列名必须与数据表结构完全一致。

若字段类型为 `image_url`，链接必须是公开可访问的图片 URL，平台会抓取图片生成向量索引用于以图搜图场景。

## 限制和注意事项

- 文件连接器平台存储：最大 100,000 个文件，1 TB 存储额度（限时免费）
- 表格连接器平台存储：1 TB 免费额度，超出后按量付费
- SQL 查询执行：仅通过 **DMS 导入数据源** 方式创建的数据库连接器支持，自定义数据源方式不支持直接执行 SQL
- OSS 连接器不支持归档、冷归档或深度冷归档存储类型的 Bucket
- 语雀连接器仅支持公网版本语雀
- 导入的文件仅支持查看最近 **90 天**内的记录（不会被删除，仅无法查看）
- 不支持直接导入 JSON、CSV、YAML 格式文件到文件连接器，需先转换为 XLSX/XLS 格式

如需通过 API 调用应用时利用标签筛选知识库文件，可在请求参数 `tags` 中指定，详见 [[application-calling-guide]]。

更多细节参考 [数据连接](../../raw/application-user-guide/data-connection-overview/data-connection.md) 原始文档。

## 来源文档

- [数据连接](../../raw/application-user-guide/data-connection-overview/data-connection.md)

