# security and compliance

阿里云百炼平台提供多层次的安全与合规能力，涵盖权限管理、内容安全审核、传输加密、私网访问、安全存储以及合规备案等方面。开发者可根据业务场景灵活组合这些能力，构建满足企业级安全要求的 AI 应用。本文汇总了百炼平台在安全与合规方面的核心功能、配置方式和注意事项。

## 权限管理

百炼通过**业务空间**作为精细化权限管理的最小单元，支持基于角色的多维度权限控制。详细内容参见 [权限管理](../../raw/model-user-guide/security-and-compliance/permission-management-overview.md)。

### 角色体系

| 角色 | 范围 | 核心能力 |
|------|------|----------|
| **超级管理员** | 跨空间 | 模型授权与限流、用户管理、API Key 管理 |
| **业务空间管理员** | 单空间 | 用户管理、页面权限、API Key 管理 |
| **普通用户** | 单空间 | 使用被授权的资源 |

超级管理员包括阿里云主账号以及拥有 `AliyunBailianFullAccess` 系统策略的 RAM 用户。

### API Key 权限

- 单个 API Key 只能归属一个地域内的一个业务空间和一个用户，不可转移。
- API Key 的可调用模型和限流与**归属业务空间**一致，不受用户控制台权限影响。
- 华北2（北京）地域的 API Key 支持设置 **IP 访问白名单**。

> **注意**：自 2026 年 3 月 25 日起，华北2（北京）地域所有新创建的 API Key 均归属主账号。

### OpenAPI 接口权限

RAM 用户默认无权调用百炼应用的知识库、Prompt 工程等 OpenAPI。需由主账号在 RAM 控制台添加 `AliyunBailianDataFullAccess` 或 `AliyunBailianDataReadOnlyAccess` 权限。

### 生产环境建议

- **按环境划分空间**（推荐）：为 dev/test/prod 创建独立业务空间实现环境隔离。
- **限流策略**：按比例分配配额并预留缓冲，例如总配额 1000 QPM 中分配 60% 给生产、20% 测试、10% 开发、10% 缓冲。

## 内容安全（AI 安全护栏）

百炼支持接入 AI 安全护栏服务，对模型输入输出中的涉黄、涉政、广告等违规内容进行识别。详见 [输入输出AI安全护栏](../../raw/model-user-guide/security-and-compliance/content-security.md)。

### 接入方式

1. **开通内容审核服务**：在购买页创建服务关联角色并开通。
2. **授权内容安全设置**：在百炼控制台安全管理页面完成授权。
3. **设置请求头**：在 API 调用时添加 `X-DashScope-DataInspection` 请求头：

```json
{
    "X-DashScope-DataInspection": {
       "input": "cip",
       "output": "cip"
    }
}
```

当输入内容触发安全检查时，API 返回错误码 `data_inspection_failed`（HTTP 400）。支持 OpenAI 兼容模式、DashScope SDK 等多种调用方式。

> **注意**：目前仅支持文本和图片类型的模型，具体模型与 AI 安全护栏服务的对应关系请参见官方文档。

## 传输安全

### 加密调用

当请求涉及敏感信息时，可对请求体中的 `input` 字段进行加密传输。详见 [以加密的方式接入模型推理功能](../../raw/model-user-guide/security-and-compliance/transmission-security/encrypted-access-to-model-inference.md)。

百炼采用**混合加密机制**：数据由 AES 对称算法加密，AES 密钥通过 RSA 非对称加密进行安全传输。

**两种使用方式：**

| 方式 | 适用场景 | 语言支持 |
|------|----------|----------|
| DashScope SDK（自动加密） | 开箱即用，设置 `enableEncrypt=true` | Java、Python |
| HTTP 手动密钥管理 | 需自定义密钥或使用其他语言 | 任意语言 |

SDK 方式仅需一行配置即可启用加密，响应自动解密返回明文。HTTP 方式需要手动生成 AES 密钥、通过 [[model-interface-aes-encryption]] 接口获取 RSA 公钥、加密 `input` 字段并构建 `X-DashScope-EncryptionKey` 请求头。

> **注意**：HTTP 手动加密仅适用于 DashScope 的 Endpoint，[[openai-compatible-api|OpenAI 兼容接口]]（Chat Completions API 和 Responses API）不支持此加密机制。

### 私网访问（PrivateLink）

通过创建接口终端节点，可在 VPC 内通过私网直接调用百炼 API，流量不经过公网。支持的地域：

- **公共云**：华北2（北京）、新加坡
- 美国（弗吉尼亚）暂不支持

**关键配置要点：**
- 终端节点服务选择 `com.aliyuncs.dashscope`
- 安全组需允许 80/443 端口入方向访问
- 建议至少选择两个可用区的交换机以实现高可用
- 默认服务域名仅支持 HTTP，HTTPS 需使用自定义服务域名

**跨地域场景：**
- 同境内/同境外跨地域：启用跨地域端点
- 跨境跨地域：通过云企业网（CEN）实现 VPC 互通

## 安全存储

百炼提供**安全存储业务空间**，将数据存储在用户自有的私有网络资源（ElasticSearch、AnalyticDB、OSS）中，实现数据隔离。此功能需要申请开通。

### 配置流程

1. **创建安全存储业务空间** → 创建反向终端节点 → 确认连接
2. **配置可用区 IP** → 创建 MSE 云原生网关 → 获取可用区 VIP
3. **配置私有网络资源**：
   - **OSS**：创建 Bucket（需设置标签 `bailian-safe-workspace-oss-access: ReadAndWrite`），配置跨域规则
   - **ADB**：购买 AnalyticDB PostgreSQL 6.0 标准版，开启向量引擎优化
   - **ElasticSearch**：购买内核增强版 7.10，将交换机网段加入白名单
4. **配置 MSE 网关** → 创建服务和路由 → 激活安全存储空间

> **注意**：OSS Bucket 或 ES 实例被释放将导致安全存储空间不可用且**无法恢复**，需重新创建。所有资源须位于华北2（北京）地域。

## 合规与隐私

### 合规资质

- 百炼已通过 **SOC 2** 审计（无保留意见），覆盖安全、可用性和保密性。
- 更多资质信息参见 [阿里云合规文档中心](https://security.aliyun.com/compliance-repository)。

### 隐私保护

- 阿里云**不会**将用户数据用于模型训练。
- 传输数据经过 **AES-256** 加密。
- 根据法规要求，百炼会存储模型与应用调用时产生的数据，具体条款参见《阿里云百炼服务协议》。

### 算法备案

根据《生成式人工智能服务管理暂行办法》，使用千问、万相等大模型的应用上架需完成合规备案。备案主体信息如下：

| 大模型 | 备案号 | 备案主体 |
|--------|--------|----------|
| 千问 | 网信算备330110507206401230035号 | 阿里巴巴达摩院(杭州)科技有限公司 |
| 万相（图像） | 网信算备330110507206401230027号 | 阿里巴巴达摩院(杭州)科技有限公司 |
| 万相（视频） | 网信算备330106003156001240091号 | 通义云启（杭州）信息技术有限公司 |

> **注意**：即使使用了阿里云的模型及备案信息，应用开发者仍是法规定义的"服务提供者"，需独立承担内容审核、用户保护、数据安全等全部法定义务。备案信息应以 [互联网信息服务算法备案系统](https://beian.cac.gov.cn/#/index) 实时查询结果为准。

## 限制与注意事项

- **业务空间不能跨地域**：各地域的默认业务空间是独立的。
- **默认业务空间**无法设置模型调用限制和限流。
- 加密调用中 AES 密钥应**单次请求有效，禁止复用**，建议使用密码学安全随机源生成。
- 开通 AI 安全护栏、[[model-telemetry]]、[[application-observation]] 等功能，建议使用**主账号**进行一次性授权。
- 账单查看（`AliyunBSSReadOnlyAccess`）和预付费购买（`AliyunBSSOrderAccess`）权限将授予 RAM 用户查看/购买**所有阿里云产品**的权限，请谨慎授权。

## 来源文档

- [权限管理](../../raw/model-user-guide/security-and-compliance/permission-management-overview.md)
- [输⼊输出AI安全护栏](../../raw/model-user-guide/security-and-compliance/content-security.md)
- [千问大模型应用上架及合规备案](../../raw/model-user-guide/security-and-compliance/compliance-and-launch-filing-guide-for-ai-apps-powered-by-the-tongyi-model.md)
- [合规资质与隐私说明](../../raw/model-user-guide/security-and-compliance/privacy-notice.md)
- [获取RSA的公钥](../../raw/model-user-guide/security-and-compliance/transmission-security/model-interface-aes-encryption.md)
- [以加密的方式接入模型推理功能](../../raw/model-user-guide/security-and-compliance/transmission-security/encrypted-access-to-model-inference.md)
- [通过终端节点私网访问阿里云百炼模型或应用 API](../../raw/model-user-guide/security-and-compliance/transmission-security/access-model-studio-through-privatelink.md)
- [配置终端节点并发起连接](../../raw/model-user-guide/security-and-compliance/secure-storage/configure-an-endpoint-and-initiate-a-connection.md)
- [配置可用区IP](../../raw/model-user-guide/security-and-compliance/secure-storage/configure-zone-ip.md)
- [配置MSE云原生网关](../../raw/model-user-guide/security-and-compliance/secure-storage/configure-mse.md)
- [配置私有网络中的资源](../../raw/model-user-guide/security-and-compliance/secure-storage/configure-resources-in-private-network.md)

