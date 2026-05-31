# application permission management

阿里云百炼平台提供基于控制台页面级和模型级的多维度权限控制，支持多地域、多用户的复杂组织架构管理。权限管理的最小管理单元是**业务空间**，通过角色划分（超级管理员、业务空间管理员、普通用户）实现精细化的资源访问控制。

## 身份与角色体系

百炼的权限管理基于三种角色，详见 [权限管理](../../raw/application-user-guide/application-permission-management/application-permission-management-overview.md) 原文：

| 角色 | 说明 |
|------|------|
| **超级管理员** | 阿里云主账号或拥有 `AliyunBailianFullAccess` 策略的 RAM 用户，可跨空间统一管理权限 |
| **业务空间管理员** | 拥有特定业务空间"权限管理"页面访问权限的 RAM 用户 |
| **普通用户** | 根据分配的权限使用资源，无管理能力 |

## 权限矩阵

| 业务空间权限 | 超级管理员 | 业务空间管理员 | 普通用户 |
|---|---|---|---|
| 模型调用 & 限流配置 | ✅ | ❌ | ❌ |
| 模型调优/部署配置 | ✅ | ❌ | ❌ |
| 用户管理 | ✅ | ✅ | ❌ |
| 用户可用页面管理 | ✅ | ✅ | ❌ |
| API Key 管理 | ✅ | ✅ | ❌ |
| 访问被授权的资源 | ✅ | ✅ | ✅ |
| OpenAPI 接口权限 | 需主账号在 RAM 控制台单独配置 | — | — |

## 业务空间与模型管理

业务空间按地域划分，**单个业务空间不能跨地域存在**。根据 [权限管理](../../raw/application-user-guide/application-permission-management/application-permission-management-overview.md) 文档，非默认业务空间可管理：

- **模型调用限制**：控制模型是否可在该空间调用，设置请求数限流和 Token 限流
- **模型训练限制**：控制模型是否可在该空间进行调优和部署
- **模型部署限制**：控制模型是否可在该空间直接部署

> **注意**：默认业务空间无法设置上述限制，所有模型均可调用且无法限流。

## API Key 权限

API Key 的关键特性：

- 单个 API Key 只能归属**一个地域**内的**一个业务空间**和**一个用户**，不可转移
- 可调用的功能和模型限流与归属业务空间一致，不受用户控制台权限影响
- 无需为不同模型类型（文生文、文生图、语音合成等）创建不同 API Key
- 自 2026年3月25日起，华北2（北京）地域新建的 API Key 均归属主账号

API Key 状态变化规则：
- 主动删除：失效且不可恢复
- 用户移出业务空间：失效（重新加入后恢复）
- RAM 控制台删除账号/角色：失效且不可恢复

## OpenAPI 接口权限

RAM 用户默认无权调用百炼**应用**类 OpenAPI（[[knowledge-base]]、Prompt工程、长期记忆等）。需由主账号在 RAM 控制台添加以下权限之一：

- `AliyunBailianDataFullAccess`：可调用应用 API 目录下所有 API
- `AliyunBailianDataReadOnlyAccess`：仅可调用只读类 API

## 常用配置步骤

### 设置超级管理员

需主账号或具备 `AliyunRAMFullAccess` 的 RAM 用户操作：

1. 在 RAM 控制台为 RAM 用户添加 `AliyunBailianFullAccess` 和 `AliyunBSSOrderAccess` 权限
2. 完成后可通过全局管理菜单管理所有地域和空间

### 设置模型调用权限

1. 非默认空间需超级管理员开通模型调用权限
2. 控制台调用需添加：**模型体验-操作**、**批量推理-操作**、**模型观测-操作**
3. API 调用需在对应空间创建或分配 API Key

### 设置模型调优权限

控制台调优需添加：模型体验、模型调优、我的模型、模型部署、模型评测、数据管理、模型观测的操作权限。API 调优仅需 API Key。

## 生产环境最佳实践

根据 [权限管理](../../raw/application-user-guide/application-permission-management/application-permission-management-overview.md) 建议的空间规划策略：

**按环境划分（推荐）：**
- `project-dev-workspace` / `project-test-workspace` / `project-prod-workspace`

**限流策略示例**（账号总配额 1000 QPM）：
- 生产：600 QPM (60%)
- 测试：200 QPM (20%)
- 开发：100 QPM (10%)
- 预留缓冲：100 QPM (10%)

## 账单与预付费权限

RAM 用户默认无法查看账单或购买预付费产品，需额外授权：

- 查看账单：添加 `AliyunBSSReadOnlyAccess`
- 购买预付费产品：添加 `AliyunBSSOrderAccess`

> **注意**：这些权限作用于阿里云**所有产品**，并非百炼专属，请谨慎授权。

## 相关概念

- [[api-key-management]]
- [[ram-user]]
- [[workspace]]
- [[model-fine-tuning]]
- [[batch-inference]]

## 来源文档

- [权限管理](../../raw/application-user-guide/application-permission-management/application-permission-management-overview.md)

