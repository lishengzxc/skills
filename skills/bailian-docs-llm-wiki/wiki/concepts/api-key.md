# API Key 管理与安全

API Key 是调用阿里云百炼平台模型和应用的核心鉴权凭证。正确地创建、配置、分发和保护 API Key，是保障服务安全和稳定运行的基础。

## 基本概念

百炼平台中存在**三类 API Key**，格式和用途各不相同，**切勿混用**：

| 类型 | 格式 | 用途 | 获取方式 |
|------|------|------|----------|
| 百炼通用 API Key | `sk-xxxxx` | 按量计费调用模型和应用 | [密钥管理](https://bailian.console.aliyun.com/?tab=app#/api-key) |
| Token Plan / Coding Plan 专属 API Key | `sk-sp-xxxxx` | 订阅套餐内的模型调用 | Token Plan 管理后台或 Coding Plan 页面 |
| 临时 API Key | `st-xxxxx` | 浏览器、移动端等不可信环境 | 后端 API 动态生成 |

每个 API Key 只能归属**一个地域**内的**一个业务空间**和**一个用户**，不可转移。

## 权限与业务空间

API Key 的权限由其所属业务空间决定，不受用户控制台权限影响：

- **默认业务空间的 API Key**：可调用所有标准模型及该空间内的 [[application-introduction]]。
- **子业务空间的 API Key**：仅可调用已授权的标准模型及该空间内的应用，适用于**权限管控**和**费用分账**场景。
- 同一空间内的 API Key 权限相同，无需为不同模型类型分别创建。

> **生产环境建议**：按环境（dev / test / prod）划分独立业务空间，各自使用专属 API Key，实现环境隔离与配额管理。详见 [[security-and-compliance]]。

## 创建限制

| 地域 | 每个主账号最大数量 |
|------|--------------------|
| 华北2（北京）/ 新加坡 / 德国（法兰克福） | 各 **50** 个 |
| 美国（弗吉尼亚） | 每个归属账号 **20** 个 |

- API Key 无失效日期，手动删除后即失效。
- RAM 用户被禁用或删除后，其创建的所有 API Key 均失效。
- 自 2026 年 3 月 25 日起，华北2（北京）地域所有新创建的 API Key 均归属主账号。

## 安全配置

### 环境变量配置

为避免在代码中硬编码 API Key 导致泄露风险，**必须**将其配置为环境变量 `DASHSCOPE_API_KEY`：

```bash
# Linux / macOS（永久）
echo 'export DASHSCOPE_API_KEY="sk-xxx"' >> ~/.bashrc && source ~/.bashrc

# Windows CMD（永久）
setx DASHSCOPE_API_KEY "sk-xxx"
```

常见问题：仅设置临时变量对已启动的 IDE 不生效；设置永久变量后需重启 IDE；使用 `sudo` 时需加 `-E` 传递环境变量。详见 [[preparations]]。

### IP 访问白名单

目前仅**华北2（北京）**地域支持为 API Key 设置 IP 访问白名单，限制仅允许指定 IP 地址发起调用。

### 临时 API Key

在浏览器或移动 App 等不可信环境中，应通过后端服务生成临时 API Key，避免暴露永久凭证：

```bash
curl -X POST "https://dashscope.aliyuncs.com/api/v1/tokens?expire_in_seconds=1800" \
  -H "Authorization: Bearer $DASHSCOPE_API_KEY"
```

| 参数 | 说明 |
|------|------|
| `expire_in_seconds` | 有效期，范围 [1, 1800] 秒，默认 60 秒 |

关键特性：
- **继承**生成它的永久 API Key 的全部权限（包括模型和 [[knowledge-base]] 的访问限制）。
- 到期自动失效，**无法手动删除**。
- 不同地域的 Endpoint 不同，需根据实际地域替换 URL。

详见 [[more-about-models]] 和 [[more]]。

### 传输加密与私网访问

- **加密调用**：对请求中的 `input` 字段进行 AES+RSA 混合加密传输，DashScope SDK 设置 `enableEncrypt=true` 即可启用。
- **私网访问（PrivateLink）**：通过 VPC 终端节点直接调用百炼 API，流量不经过公网。支持华北2（北京）和新加坡地域。

详见 [[security-and-compliance]]。

## 在不同场景中的使用

### 模型调用

通过 DashScope SDK 或 OpenAI 兼容 SDK 调用模型时，从环境变量读取 API Key：

```python
import os
from openai import OpenAI

client = OpenAI(
    api_key=os.getenv("DASHSCOPE_API_KEY"),
    base_url="https://dashscope.aliyuncs.com/compatible-mode/v1"
)
```

不同地域需使用对应的 Base URL，详见 [[preparations]]。

### 应用调用

调用智能体或工作流应用时，需同时提供 API Key 和 APP ID。若应用位于子业务空间，还需提供 Workspace ID。详见 [[application-call]]。

### Token Plan / Coding Plan

订阅套餐使用专属的 API Key（`sk-sp-xxx`）和独立的 Base URL，与通用 API Key 完全隔离：

| 套餐 | Base URL（OpenAI 兼容） |
|------|------------------------|
| Token Plan 团队版 | `https://token-plan.cn-beijing.maas.aliyuncs.com/compatible-mode/v1` |
| Coding Plan | `https://coding.dashscope.aliyuncs.com/v1` |

详见 [[token-plan-guide]]

## 关联主题页

- [[preparations|preparations]] — `../api/[[preparations|preparations]].md`
- [[more-about-models|[[more|more]] about models]] — `../api/[[more|more]]-about-models.md`
- [[more|more]] — `../api/[[more|more]].md`
- [[application-call|application call]] — `../api/application-call.md`
- [[security-and-compliance|security and compliance]] — `../guides/security-and-compliance.md`
- [[token-plan-guide|token plan guide]] — `../guides/token-plan-guide.md`

