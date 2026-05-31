# deploy dedicated services

百炼平台的专属服务部署涵盖两个核心环节：**模型导入**和**模型部署**。用户可通过 API 将自定义微调模型从 OSS 导入平台，随后选择合适的计费方案将模型部署为在线服务。整个流程均可通过 HTTP API 完成，无需依赖控制台操作。

## 核心流程

专属服务部署的完整流程为：

1. **模型导入**：将 OSS 中的模型文件导入百炼平台（仅自定义模型需要）
2. **查询可部署模型**：获取支持部署的模型列表及方案
3. **创建部署任务**：选择计费方案并创建部署
4. **管理部署**：查询状态、缩扩容、删除部署

详细的导入流程请参考 [模型导入API参考](../../raw/model-api-reference/deploy-dedicated-services/model-import-api-reference.md)。

## 支持的模型与部署方案

根据 [模型部署API参考](../../raw/model-api-reference/deploy-dedicated-services/model-deployment-api.md)，平台支持以下部署方案（`plan` 参数）：

| 部署方案 | plan 值 | 适用场景 | 计费方式 |
|---------|---------|---------|---------|
| 模型单元 (MU) | `mu` | 通用文本模型 | 按使用时长 × 单元数 |
| 算力单元 (CU) | `cu` | 图片/视频生成模型 | 按使用时长 × 单元数 |
| 预置吞吐量 (PTU) | `ptu` | 需要稳定吞吐保障 | 按 TPM × 时长 |
| LoRA 共享部署 | `lora` | 微调模型按量使用 | 按 Token 用量 |

支持的模型包括千问系列（qwen3-max、qwen-plus、qwen-flash 等）、DeepSeek 系列、千问 VL 多模态模型及 GLM 等。可通过 `GET /api/v1/deployments/models` 接口动态获取完整列表。

## 模型导入

### 前提条件

- 已配置 [[api-key]]
- 已创建 OSS Bucket 并完成百炼平台授权
- 模型文件符合导入格式要求

### 关键参数

| 参数 | 说明 |
|------|------|
| `model_name` | 基础模型名称，如 `qwen3-32b` |
| `weight_type` | `full`（全参微调）或 `lora`（LoRA 微调）|
| `source` | 当前仅支持 `oss` |
| `storage_info.bucket_name` | OSS Bucket 名称 |
| `storage_info.object_key` | 模型路径前缀，需以 `/` 结尾 |

### 任务状态流转

```
PENDING → RUNNING → SUCCESSED / FAILED
```

> **注意**：状态值 `SUCCESSED` 为平台实际返回值（非标准英语拼写 `SUCCEEDED`），编码时请使用该值进行判断。

## 模型部署

### 创建部署请求示例（模型单元方式）

```bash
curl "https://dashscope.aliyuncs.com/api/v1/deployments" \
--header "Authorization: Bearer $DASHSCOPE_API_KEY" \
--header 'Content-Type: application/json' \
--data '{
    "name": "my_qwen_plus",
    "model_name": "qwen-plus-2025-12-01",
    "plan": "mu",
    "deploy_spec": "MU1",
    "capacity": 4,
    "billing_method": "POST_PAY"
}'
```

### 关键请求参数

| 参数 | 必选 | 说明 |
|------|------|------|
| `model_name` | 是 | 模型 ID，系统模型或导入后生成的标识 |
| `plan` | 是 | 部署方案：`mu`/`cu`/`ptu`/`lora` |
| `name` | 是 | 控制台显示名称 |
| `capacity` | MU 必填 | 资源单元数，须为 `base_capacity` 整数倍 |
| `deploy_spec` | MU 必填 | 模型单元规格，如 `MU1`、`MU2` |
| `ptu_capacity` | PTU 时生效 | 包含 `input_tpm` 和 `output_tpm` |
| `enable_thinking` | 否 | 是否启用思考模式 |
| `max_context_length` | 否 | 最长上下文长度 |
| `suffix` | 否 | 部署后模型名称后缀，最大 8 字符，需全局唯一 |

### 模板与 PD 分离

使用 `version=v1.0` 查询模型列表时，响应包含模板信息。模板类型分为：
- **COUPLED**：非 PD 分离，使用 `capacity` 参数
- **SEPERATED**：PD 分离，需分别配置 `prefill_capacity` 和 `decode_capacity`

创建部署时通过 `template_id` 指定模板。

## 限制与注意事项

- **计费即时生效**：部署任务创建成功后即开始计费，即使尚未调用模型。
- **LoRA 部署的 capacity**：`plan=lora` 时 `capacity` 参数设置无效但必须填写，扩缩容需通过控制台申请。
- **CosyVoice 等调优模型**：不接受 `MU1`/`MU2` 等缩写，必须使用具体规格 ID（形如 `dps-20260521172224-1vabse`），可通过模型列表接口的 `deploy_specs` 字段获取。
- **PTU 溢出**：当调用超过购买的 TPM 量时，自动切换为按量付费模式，响应 Header 中包含 `x-dashscope-ptu-overflow:true`。
- **删除限制**：导入任务仅在 `SUCCESSED` 或 `FAILED` 状态下可删除，`RUNNING` 状态删除会返回 `OperationDenied` 错误。

## 公共配置

所有接口共用以下请求头，详见 [模型导入API参考](../../raw/model-api-reference/deploy-dedicated-services/model-import-api-reference.md) 中的公共请求头部分：

```
Authorization: Bearer ${DASHSCOPE_API_KEY}
Content-Type: application/json
```

## 相关概念

- [[model-import]] - 从 OSS 导入自定义模型
- [[model-deployment]] - 模型部署管理
- [[api-key]] - API 密钥获取与配置
- [[rate-limit]] - 服务限流说明
- [[model-pricing]] - 模型计费详情

## 来源文档

- [模型导入API参考](../../raw/model-api-reference/deploy-dedicated-services/model-import-api-reference.md)
- [模型部署API参考](../../raw/model-api-reference/deploy-dedicated-services/model-deployment-api.md)

