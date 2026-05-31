# more

百炼平台应用 API 参考中的"更多"分类涵盖了一些不直接属于核心模型调用的辅助功能和平台机制，包括临时 API Key 的生成与管理、服务关联角色（SLR）的权限配置，以及知识库检索结果的高级过滤能力。这些功能在实际生产环境中对安全性、权限管控和检索精度起到关键支撑作用。

## 临时 API Key

在浏览器、移动 App 等不可信环境中，直接暴露永久 [[api-key]] 存在泄露风险。百炼提供了通过后端服务生成临时 API Key 的机制，详见 [生成临时API Key](../../raw/application-api-reference/more/application-obtain-temporary-authentication-token.md)。

### 关键参数

| 参数 | 说明 |
|------|------|
| `expire_in_seconds` | 有效期（TTL），范围 [1, 1800] 秒，默认 60 秒 |

### 使用方式

通过 POST 请求生成临时 Key：

```bash
curl -X POST "https://dashscope.aliyuncs.com/api/v1/tokens?expire_in_seconds=1800" \
  -H "Authorization: Bearer $DASHSCOPE_API_KEY"
```

成功后返回 `token`（以 `st-` 开头）和 `expires_at`（UNIX 时间戳）。

### 限制和注意事项

- 临时 API Key **继承**生成它的永久 API Key 的全部权限，包括模型和 [[knowledge-base]] 的访问限制。
- 临时 API Key **无法手动删除**，只能等待到期自动失效。
- 不同地域（北京、新加坡、弗吉尼亚）的 API Key 和 Endpoint 不同，需注意区分。

## 服务关联角色（SLR）

百炼通过服务关联角色获取对其他阿里云服务（如 FC、OSS、ADB-PG 等）的访问权限。首次授权开通相关功能时，系统将自动创建对应角色。完整说明参见 [服务关联角色](../../raw/application-api-reference/more/bailian-service-linked-role.md)。

### 角色清单

| 角色名称 | 用途 | 关联服务 |
|----------|------|----------|
| `AliyunServiceRoleForSFMAccessFC` | [[workflow-application]] 和流程编排访问函数计算 | FC |
| `AliyunServiceRoleForSFMDataHubOSSImport` | 数据管理 OSS 导入 | OSS |
| `AliyunServiceRoleForAccessOSS` | 安全存储空间访问 OSS | OSS |
| `AliyunServiceRoleForSFMAccessADB` | 知识库和安全存储访问向量数据库 | ADB-PG |
| `AliyunServiceRoleForSFMAccessingMNS` | 数据管理监听 OSS 变更 | MNS |
| `AliyunServiceRoleForSFMTelemetry` | [[application-observation]] 访问 OpenTelemetry | ARMS/XTrace |
| `AliyunServiceRoleForSFMAccessingCIP` | 应用访问内容安全服务 | 内容安全 |
| `AliyunServiceRoleForSFMAccessSLS` / `AliyunServiceRoleForSFMAccessCMS` | [[model-telemetry]] 访问日志和监控 | SLS / CMS |
| `AliyunServiceRoleForAccessCusOss` | 平台托管操作用户 OSS 文件 | OSS |
| `AliyunServiceRoleForSFMConnectorAccessDTS` | 通过 DTS 接入外部数据源 | DTS |
| `AliyunServiceRoleForSFMFineTuning` | [[fine-tuning]] 和数据管理访问存储 | CPFS / OSS |

### 删除注意事项

> **注意**：删除服务关联角色前，必须先清理其依赖资源（如断开安全存储空间连接、删除工作流中的函数计算节点等），否则相关功能将不可用。删除方法参见 RAM 控制台中的[服务关联角色文档](https://help.aliyun.com/zh/ram/user-guide/service-linked-roles#section-b9f-8dv-b5q)。

## 知识库 SearchFilters

在调用知识库 [[retrieve]] 接口时，如果语义检索结果包含较多干扰信息，可通过 `searchFilters` 参数对结果进行结构化过滤。该功能尤其适合结构化数据场景，详见 [知识库SearchFilters](../../raw/application-api-reference/more/how-to-use-search-filters.md)。

### 支持的查询类型

| 查询类型 | 字段类型 | 说明 |
|----------|----------|------|
| 单值查询 | long / double / string | 精确匹配单个值 |
| 多值查询 | 纯数值或纯字符串数组 | 匹配数组中的任意值 |
| 范围查询（等值） | long / double / string | 支持 `eq`、`neq` |
| 范围查询（区间） | long / double | 支持 `gt`、`gte`、`lt`、`lte` |
| 模糊查询 | string | 支持 `like`，使用 `%` 通配符 |
| 标签查询 | tags | 仅文档搜索、音视频搜索类知识库 |

### 语法结构

`searchFilters` 是一个数组，包含一个或多个子分组。**子分组之间为 AND 语义且不可更改**。同一子分组内可包含多组 Key-Value 键值对。

```json
{
  "searchFilters": [
    { "姓名": "张三", "性别": "男" },
    { "岗位": "技术员" }
  ]
}
```

### 前置条件

- 子账号需获取 `AliyunBailianDataFullAccess` 策略并加入业务空间。
- 安装阿里云百炼 SDK（`bailian 2023-12-29` 版本）并配置 AccessKey 环境变量。
- 知识库需已创建且字段已设置为参与检索。

## 来源文档

- [生成临时API Key](../../raw/application-api-reference/more/application-obtain-temporary-authentication-token.md)
- [服务关联角色](../../raw/application-api-reference/more/bailian-service-linked-role.md)
- [知识库SearchFilters](../../raw/application-api-reference/more/how-to-use-search-filters.md)

