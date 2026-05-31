# 模型微调、模型训练与模型部署对比

## 概述

在阿里云百炼平台上，**模型微调**、**模型训练**和**模型部署**是模型定制化全链路中的三个核心环节。模型微调侧重于调优策略的选择与数据准备，模型训练聚焦于通过 API 执行训练任务的技术实现，模型部署则负责将训练产出的模型上线为可调用的推理服务。三者在功能定位、操作方式和计费模型上各有不同，但在实际业务中相互衔接、缺一不可。

本文旨在帮助开发者快速理解三者的差异与关联，从而在技术选型和工作流设计时做出更高效的决策。

## 核心维度对比

### 功能定位与输入输出

| 维度 | 模型微调 | 模型训练 | 模型部署 |
|------|---------|---------|---------|
| **核心定位** | 调优策略规划与数据准备 | 训练任务的 API 执行与管理 | 将模型上线为推理服务 |
| **主要输入** | 训练数据（JSONL / ZIP）、调优方式与超参数配置 | `file_id`、`model`、`training_type`、`hyper_parameters` | 模型 ID / 模型名称、计费方式、部署配置 |
| **主要输出** | 调优后的模型（`fine_tuned_model`） | 任务状态与训练产出模型 ID | 可调用的推理端点（`deployed_model`） |
| **操作方式** | 控制台（可视化）+ API / 命令行 | HTTP API / DashScope CLI | 控制台 + HTTP API |
| **地域限制** | 仅中国大陆版（北京地域） | 仅中国大陆版（北京地域） | 仅中国内地（北京）地域 |

### 支持的模型范围

| 维度 | 模型微调 | 模型训练 | 模型部署 |
|------|---------|---------|---------|
| **文本生成** | Qwen3、Qwen3.5、Qwen2.5 系列 | qwen3-14b 等千问系列 | 千问系列、DeepSeek、GLM、MiniMax、Kimi 等 |
| **视觉理解** | Qwen-VL 系列（SFT） | 千问-VL 系列（SFT） | 千问 VL 系列、千问 Omni 系列 |
| **语音合成** | cosyvoice-v3-flash | cosyvoice-v3-flash | CosyVoice |
| **视频生成** | wan2.5-i2v-preview、wan2.2 系列 | wan2.5-i2v-preview、wan2.2 系列 | 万相、悦动人像 EMO、舞动人像 |
| **语音识别** | — | paraformer 系列（热词编译） | — |
| **模型导入** | — | — | 支持从 OSS 导入 LoRA 模型 |

### 训练方式与方法

| 维度 | 模型微调 | 模型训练 | 模型部署 |
|------|---------|---------|---------|
| **调优方式** | SFT、CPT、DPO 三种方式 | `sft`、`efficient_sft`、`cpt`、`dpo_full`、`dpo_lora` | 不涉及训练 |
| **训练模式** | 全参训练 / 高效训练（LoRA） | 通过 `training_type` 参数指定 | 不涉及训练 |
| **推荐流程** | CPT（可选）→ SFT → DPO（可选） | 由调用方按需组合 | 训练完成后部署 |
| **混合训练** | 支持（SFT / efficient_SFT） | 通过 `data_augmentation` 参数启用 | 不涉及 |
| **二次调优** | 支持 | 支持（`model` 填已调优模型 ID） | 不涉及 |

### API 端点与接口

| 维度 | 模型微调 | 模型训练 | 模型部署 |
|------|---------|---------|---------|
| **上传数据** | `POST /api/v1/files` | `POST /api/v1/files`（`purpose=fine-tune`） | — |
| **创建任务/服务** | `POST /api/v1/fine-tunes` | `POST /api/v1/fine-tunes` | `POST /api/v1/deployments` |
| **查询状态** | `GET /api/v1/fine-tunes/{job_id}` | `GET /api/v1/fine-tunes/{job_id}` | `GET /api/v1/deployments/{deployed_model}` |
| **删除/取消** | — | 支持取消任务 | `DELETE /api/v1/deployments/{deployed_model}` |
| **推理调用** | 部署后使用 `deployed_model` 调用 | 部署后使用产出模型 ID 调用 | [OpenAI 兼容接口](../concepts/openai-compatible-api.md) / DashScope 接口 / Assistant SDK |

### 数据格式要求

| 维度 | 模型微调 | 模型训练 | 模型部署 |
|------|---------|---------|---------|
| **SFT 数据** | ChatML 格式 JSONL（`messages` 数组含 system/user/assistant） | 同左，通过 `file_id` 引用 | 不涉及 |
| **DPO 数据** | 在 messages 基础上增加 `chosen` 和 `rejected` 字段 | 同左 | 不涉及 |
| **CPT 数据** | 纯文本格式 `{"text": "..."}` | 同左 | 不涉及 |
| **CosyVoice 数据** | `.wav` 音频 + `data.jsonl` 打包为 ZIP | 同左 | 不涉及 |
| **视觉理解数据** | `content` 使用数组格式，压缩为 ZIP 上传 | 同左 | 不涉及 |
| **导入模型** | — | — | `adapter_model.safetensors` + `adapter_config.json` |
| **文件大小限制** | 单文件 300MB，总空间 5GB，最多 100 个 | 单文件 1GB，总空间 5GB，最多 100 个 | 仅 LoRA 模型，rank 限定 8/16/32/64 |

### 计费方式

| 维度 | 模型微调 | 模型训练 | 模型部署 |
|------|---------|---------|---------|
| **文本模型费用** | 按训练 Token 数 × 循环次数 × 单价（如 qwen3-8b ¥0.006/千Token） | 同左（API 仅支持按 Token 计费） | 预置吞吐（PTU）/ 模型单元（MU）/ Token 用量 |
| **CosyVoice 费用** | 训练按 Token 计费（¥0.2/千Token），部署按模型单元时长 | 同左 | 按模型单元使用时长 |
| **视频/图片模型** | — | — | 按实例时长（后付费/包月） |
| **付费方式** | 控制台支持训练单元（预付费/后付费），API 仅按 Token | 仅按 Token 计费 | 随用随付 / 包天 / 包月 |
| **计费起点** | 任务开始训练时计费 | 同左 | 部署命令执行后即开始计费（即使未调用） |

### 并发与限制

| 维度 | 模型微调 | 模型训练 | 模型部署 |
|------|---------|---------|---------|
| **并发限制** | 同一时间仅一个训练任务执行 | 同一时间仅一个训练任务执行，其余排队 | 可同时运行多个部署服务 |
| **任务状态** | — | PENDING → QUEUING → RUNNING → SUCCEEDED/FAILED/CANCELED | 部署中 → 运行中 / 失败 |
| **权限要求** | 子账号需额外授予训练和部署权限 | 同左 | 子账号需授予部署权限 |
| **扩缩容** | — | — | PTU/MU 自助调整；Token 用量需提交申请 |

## 适用场景建议

### 模型微调（Fine-tuning Guide）

适合以下场景的开发者参考：

- **初次接触模型调优**，需要了解 SFT、CPT、DPO 三种方法的原理和选择策略
- **数据准备阶段**，需要明确不同调优方式对数据格式、数据量的要求
- **偏好控制台操作**，通过可视化界面完成从数据上传到模型部署的完整流程
- **需要全参训练**以追求最优效果的场景

>

## 被对比主题页

- [fine tuning](../guides/fine-tuning.md)
- [model training](../api/model-training.md)
- [model deployment 1](../guides/model-deployment-1.md)

