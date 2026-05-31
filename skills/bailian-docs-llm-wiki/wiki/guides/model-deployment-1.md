# model deployment 1

百炼平台的模型部署功能允许开发者将预置模型或经过 [[model-training-overview]] 的自定义模型部署为独立的、资源专享的推理服务，以满足高并发、低延迟等业务需求。部署流程涵盖模型导入、服务创建、推理调用和服务管理等环节，支持多种计费方式和模型类型。本文汇总了模型部署相关的核心概念、操作步骤和注意事项。

## 支持的模型

### 可部署的预置模型

根据 [模型部署简介](../../raw/model-user-guide/model-deployment-1/model-deployment-introduction.md)，平台支持部署以下系列的预置模型：

- **千问系列**：千问3.7-Max、千问3.6-Flash/Plus、千问3.5-Plus、千问3-Max、千问-Flash、千问-Plus 等
- **DeepSeek 系列**：DeepSeek-v4-Pro/Flash、DeepSeek-v3.2、DeepSeek-v3
- **千问 VL（视觉语言）系列**：千问3-VL-Plus、千问3-VL-8B/32B/235B 等
- **千问 Omni 系列**：千问3.5-Omni-Flash/Plus
- **其他模型**：GLM-5/4.7、MiniMax-M2.5、Kimi-K2.5 等
- **图片/视频生成**：万相文生图、悦动人像 EMO、舞动人像 AnimateAnyone 等
- **语音合成**：CosyVoice-v3-flash

### 可导入的 LoRA 模型

根据 [模型导入](../../raw/model-user-guide/model-deployment-1/model-import.md)，当前支持从 OSS 导入以下基础模型的 LoRA 微调版本：

| 模型系列 | 模型名称 |
|---------|---------|
| 千问3 | 千问3-32B、千问3-14B、千问3-8B、千问3-4B-Instruct-2507 |
| 千问3-VL | 千问3-VL-8B-Instruct |
| 千问2.5 | 千问2.5-72B/32B/14B/7B-Instruct |
| 千问2.5-VL | 千问2.5-VL-72B/7B-Instruct |

> **注意**：当前仅支持导入 LoRA 模型，不支持导入全参微调模型。

## 计费方式

平台提供四种计费方式，创建后**不可更改**，需下线重新部署才能切换：

| 计费方式 | 适用场景 | 特点 |
|---------|---------|------|
| **预置吞吐（PTU）** | 高负载生产环境，需要稳定吞吐保障 | 按使用时长 × TPM 计费；TPS 通常提升 1.5~2.0 倍；超出购买量自动降级为按量付费 |
| **模型单元（MU）** | 需自定义性能指标，资源独占 | 按使用时长 × 单元数量计费；支持 [[pd-separation]] 分离模式；支持包月 |
| **Token 用量** | 调优后模型效果验证 | 按实际 Token 消耗计费，不使用不计费；仅支持部分 LoRA 调优后模型 |
| **算力单元（CU）** | 图片/视频生成模型 | 按实例占用时长计费 |

费用计算公式：
- PTU：`费用 = 使用时长 × (输入 TPM 单价 × 输入 TPM + 输出 TPM 单价 × 输出 TPM)`
- MU：`费用 = 使用时长(小时) × 模型单元数量 × 模型单元单价`
- Token：`费用 = 输入 Token 数 × 输入单价 + 输出 Token 数 × 输出单价`

## 模型导入流程

从 OSS 导入 LoRA 模型的前置条件和步骤：

### 前置条件

1. 已创建 OSS Bucket 并添加 `bailian-datahub-access` 标签（标签值为 `read`）
2. 不支持归档、冷归档或深度冷归档存储类型
3. 不支持访问 Bucket 根目录下的文件，需放在子目录中
4. 首次导入需完成 OSS 服务关联角色授权

### 模型文件要求

- **必需文件**：`adapter_model.safetensors`（权重文件）和 `adapter_config.json`（配置文件）
- **rank 参数**：必须为 8、16、32 或 64 之一，所有 LoRA 层需使用相同 rank 值
- **词汇表**：不可修改，必须与基础模型一致
- **chat_template**：不可修改，必须与基础模型默认配置一致
- **VL 模型**：必须冻结 VIT 部分，adapter 中不能包含 `visual` 相关权重

## 使用 API 部署模型

根据 [使用 API或命令行进行模型部署](../../raw/model-user-guide/model-deployment-1/model-deployment-quick-start.md)，API 部署的完整流程如下：

### 前提条件

- 已获取 [[api-key]] 并配置到环境变量 `DASHSCOPE_API_KEY`
- API Key 所在业务空间拥有模型部署权限

### 1. 创建部署

API 端点：`POST https://dashscope.aliyuncs.com/api/v1/deployments`

不同计费方式的请求示例：

**PTU 模式：**
```bash
curl "https://dashscope.aliyuncs.com/api/v1/deployments" \
--header "Authorization: Bearer $DASHSCOPE_API_KEY" \
--header 'Content-Type: application/json' \
--data '{
    "name": "my_qwen_flash",
    "model_name": "qwen-flash-2025-07-28",
    "plan": "ptu",
    "ptu_capacity": {
        "input_tpm": 10000,
        "output_tpm": 1000
    }
}'
```

**MU 模式：**
```bash
curl "https://dashscope.aliyuncs.com/api/v1/deployments" \
--header "Authorization: Bearer $DASHSCOPE_API_KEY" \
--header 'Content-Type: application/json' \
--data '{
    "name": "my_qwen_plus",
    "model_name": "qwen-plus-2025-12-01",
    "plan": "mu",
    "deploy_spec": "MU1",
    "enable_thinking": true,
    "capacity": 4,
    "max_context_length": 10000,
    "rpm_limit": 500,
    "tpm_limit": 1000
}'
```

**Token 用量（LoRA）模式：**
```bash
curl "https://dashscope.aliyuncs.com/api/v1/deployments" \
--header "Authorization: Bearer $DASHSCOPE_API_KEY" \
--header 'Content-Type: application/json' \
--data '{
    "model_name": "qwen3-8b-ft-202511132025-0260",
    "plan": "lora",
    "capacity": 1,
    "name": "qwen3-8b-ft"
}'
```

> **注意**：LoRA 模式下 `capacity` 参数设置无效但必须填写。扩缩容需在控制台提交申请。

### 2. 查询状态

```bash
curl "https://dashscope.aliyuncs.com/api/v1/deployments/{deployed_model}" \
    --header "Authorization: Bearer $DASHSCOPE_API_KEY"
```

当返回 `"status": "RUNNING"` 时，部署完成。

### 3. 调用推理

```python
from dashscope import Generation
response = Generation.call(
    model='qwen3-8b',
    [[prompt|prompt]]='你是谁？',
    enable_thinking=False,
    api_key=os.getenv('DASHSCOPE_API_KEY'),
)
```

部署后支持通过 [[openai-compatible-api]]、[[dashscope-api]] 及 [[assistant-sdk]] 进行调用。调用时 `model` 参数取值为部署后的模型 `code`。

### 4. 删除服务

```bash
curl --request DELETE \
  'https://dashscope.aliyuncs.com/api/v1/deployments/{deployed_model}' \
    --header "Authorization: Bearer $DASHSCOPE_API_KEY"
```

删除后不可恢复，服务立即停止计费。

## 关键参数说明

### MU 模式部署配置

| 配置项 | 说明 |
|-------|------|
| `enable_thinking` | 推理模式：`true` 为思考模式（Thinking），`false` 为非思考模式（Instruct） |
| `max_context_length` | 最长上下文长度，基于模型类型 |
| `rpm_limit` / `tpm_limit` | 服务限流配置 |
| `deploy_spec` | 模型单元规格（如 MU1-MU9） |

### 推理参数对齐（导入模型适用）

导入的模型推理效果可能与本地 vLLM/SGLang 不一致，建议调整以下参数：

| 参数 | vLLM 默认值对应 |
|-----|---------------|
| `temperature` | 1.0 |
| `top_p` | 1.0 |
| `top_k` | None 或 >100（不启用） |
| `presence_penalty` | 0 |
| `repetition_penalty` | 1.0 |

## 限制和注意事项

- **地域限制**：本功能仅适用于中国大陆版（北京地域）
- **计费即时性**：PTU、MU、CU 模式部署成功后即开始计费，即使未调用模型
- **Token 用量模式**：一个月内不使用将自动释放
- **PTU 溢出**：超出购买吞吐量时自动降级为按量付费，API 返回 Header 包含 `x-dashscope-ptu-overflow:true`
- **MU 后付费**：算力资源先到先得，购买不成功全额退款
- **预付费退订**：MU 包月首月内提前退订，日单价按 1.2 倍计费；PTU 预付费无法提前终止
- **权限问题**：API 调用需确保 API Key 归属业务空间拥有部署权限，且账号在该空间有操作权限
- **欠费处理**：后付费账户欠费后资源保留 24 小时后自动释放

## 来源文档

- [模型导入](../../raw/model-user-guide/model-deployment-1/model-import.md)
- [使用 API或命令行进行模型部署](../../raw/model-user-guide/model-deployment-1/model-deployment-quick-start.md)
- [模型部署简介](../../raw/model-user-guide/model-deployment-1/model-deployment-introduction.md)

