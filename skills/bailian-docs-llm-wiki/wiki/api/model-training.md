# model training

百炼平台提供模型调优（Fine-tuning）能力，支持对文本生成、视觉理解、语音合成、视频生成以及语音识别等多种模型进行训练和定制。开发者可通过 HTTP API 或 SDK 完成训练数据上传、调优任务创建、任务状态查询等全流程操作。本文汇总了模型训练相关的核心概念、支持的模型类型、关键参数及使用方式。

## 支持的模型与训练类型

百炼平台的模型训练覆盖以下场景：

| 场景 | 支持模型（示例） | 训练类型 |
|------|------------------|----------|
| 文本生成 / 视觉理解 | Qwen3 系列、Qwen-VL 系列 | `sft`、`efficient_sft`、`cpt`、`dpo_full`、`dpo_lora` |
| 语音合成 | `cosyvoice-v3-flash` | `efficient_sft` |
| 视频生成（图生视频） | `wan2.5-i2v-preview`、`wan2.2-i2v-flash`、`wan2.2-kf2v-flash` | `efficient_sft` |
| 语音识别热词 | Paraformer 系列 | `compile_asr_phrase` |

详细的基础模型 ID 及训练方式组合，请参阅 [模型调优 API 参考](../../raw/model-api-reference/model-training/model-training-api-reference.md) 中的说明。视频生成模型的微调流程与参数有所不同，具体见 [视频生成模型微调API参考](../../raw/model-api-reference/model-training/wan-video-generation-finetune-api-reference.md)。

## 整体工作流程

1. **上传训练数据** — 通过文件管理 API 上传训练文件，获取 `file_id`。
2. **创建调优任务** — 指定基础模型、训练文件 ID、超参数等，发起训练。
3. **查询任务状态** — 轮询任务状态，直到 `SUCCEEDED` 或其他终态。
4. **使用/部署模型** — 训练完成后，通过返回的 `finetuned_output` 模型 ID 进行推理或部署。

## 文件管理

训练数据通过 [百炼文件管理 API](../../raw/model-api-reference/model-training/model-customization-file-management-service.md) 统一管理，一次上传可在多个任务中复用。

**核心接口：**

| 操作 | 方法 | 端点 |
|------|------|------|
| 上传文件 | `POST` | `/api/v1/files` |
| 列举文件 | `GET` | `/api/v1/files` |
| 查询文件 | `GET` | `/api/v1/files/{file_id}` |
| 删除文件 | `DELETE` | `/api/v1/files/{file_id}` |

上传时 `purpose` 字段设为 `fine-tune` 即用于模型调优。也支持 `file-extract`（[[qwen-long]] 长上下文分析）和 `batch`（[[batch]] 批量任务）等用途。

**使用限制：**
- 单个文件最大 1GB
- 有效文件（未删除）总空间配额 5GB
- 有效文件（未删除）总数量配额 100 个

## 关键超参数

### 文本生成 / 视觉理解模型

以下参数中 `n_epochs`、`batch_size`、`max_length` 为**必填**项，直接影响训练费用：

| 参数 | 类型 | 说明 |
|------|------|------|
| `n_epochs` | Integer | 训练循环次数。数据量 < 10,000 推荐 3~5 次；> 10,000 推荐 1~2 次 |
| `batch_size` | Integer | 批次大小，不同模型默认值不同 |
| `max_length` | Integer | 单条数据最大 token 长度，超出的数据将被丢弃，推荐 8192 |
| `learning_rate` | Float | 学习率，推荐使用默认值 |
| `lr_scheduler_type` | String | 学习率调整策略，推荐 `linear` 或 `inverse_sqrt` |
| `split` | Float | 训练集占比（未指定验证集时生效），默认 0.8~0.9 |

**高效微调（LoRA）参数**（适用于 `efficient_sft`、`dpo_lora`）：

| 参数 | 说明 | 推荐值 |
|------|------|--------|
| `lora_rank` | 低秩矩阵秩值 | 64 |
| `lora_alpha` | LoRA 缩放系数 | 使用默认值 |
| `lora_dropout` | 丢弃率 | 使用默认值 |

> **注意**：对已高效微调的模型进行二次微调时，`lora_rank`、`lora_alpha`、`lora_dropout` 三个参数**必须**与首次保持一致。

**混合训练参数**（适用于 `efficient_sft`、`sft`）：开启 `data_augmentation` 后，训练数据将与百炼提供的通用数据集混合，可提升效果、避免能力退化，但混合数据计入总训练 Token 按标准计费。

**模型参数快照发布**（适用于 `efficient_sft`、`sft`）：通过 `save_strategy`、`save_steps`、`save_total_limit` 控制 Checkpoint 保存策略。

### 视频生成模型

视频模型使用独立的超参数集，部分参数含义与文本模型相同但默认值和取值范围不同：

| 参数 | 说明 | wan2.5 默认 | wan2.2 默认 |
|------|------|-------------|-------------|
| `n_epochs` | 训练轮数 | 400 | 400 |
| `batch_size` | 批次大小 | 2 | 4 |
| `learning_rate` | 学习率 | 2e-5 | 2e-5 |
| `max_pixels` | 最大视频分辨率（宽×高像素数） | 36864 | 262144 |
| `eval_epochs` | 验证间隔（epoch） | 50 | 50 |

> **注意**：视频模型使用 `eval_epochs` 作为验证间隔参数，而文本模型使用 `eval_steps`（按步数）。两者不可混用。

### CosyVoice 语音合成模型

仅适用于 `cosyvoice-v3-flash`，8 个 LM/FM 超参**全部必填**，与文本模型的 `n_epochs`、`batch_size`、`max_length` 不可混用。详见 [模型调优 API 参考](../../raw/model-api-reference/model-training/model-training-api-reference.md) 中的 CosyVoice 章节。

### Paraformer 热词

热词是语音识别场景的特殊"训练"方式，通过 `AsrPhraseManager` SDK 管理，支持创建、查询、更新、删除和列举热词。热词列表最大 500 个，权重取值 `[1, 5]`（提高识别概率）或 `[-6, -1]`（降低识别概率）。详见 [paraformer热词](../../raw/model-api-reference/model-training/paraformer-asr-phrase-manager.md)。

## 调优任务 API

### 创建任务

```
POST https://dashscope.aliyuncs.com/api/v1/fine-tunes
Content-Type: application/json
Authorization: Bearer ${DASHSCOPE_API_KEY}
```

请求体核心字段：

| 字段 | 必选 | 说明 |
|------|------|------|
| `model` | 是 | 基础模型 ID，或已调优模型 ID（支持二次调优） |
| `training_file_ids` | 是 | 训练集文件 ID 数组 |
| `validation_file_ids` | 否 | 验证集文件 ID 数组，未提供时自动从训练集按 `split` 比例划分 |
| `training_type` | 否 | 训练方式：`sft`、`efficient_sft`、`cpt`、`dpo_full`、`dpo_lora` |
| `hyper_parameters` | 否 | 超参数配置（不同模型参数集合不同） |

### 查询任务

```
GET https://dashscope.aliyuncs.com/api/v1/fine-tunes/{job_id}
```

### 任务状态

| 状态 | 含义 |
|------|------|
| `PENDING` | 待开始 |
| `QUEUING` | 排队中（同时仅一个任务可执行） |
| `RUNNING` | 进行中 |
| `SUCCEEDED` | 训练成功 |
| `FAILED` | 训练失败 |
| `CANCELING` | 取消中 |
| `CANCELED` | 已取消 |

训练成功后，返回的 `output.finetuned_output` 即为可用于推理或 [[model-deployment]] 的模型 ID。

## 限制和注意事项

- **地域限制**：模型调优 API 仅适用于中国大陆版（北京地域），需使用该地域的 [[api-key]]。
- **并发限制**：同一时间仅允许一个训练任务运行，其余任务排队等待。
- **文件配额**：有效文件数量上限 100 个、总空间 5GB。
- **数据丢弃**：单条训练数据 token 超过 `max_length` 时会被直接丢弃。
- **费用相关**：`n_epochs`、`batch_size`、`max_length` 影响训练费用；混合训练的增补数据也计入计费 Token。
- **视觉模型特殊参数**：`freeze_vit` 设置为 `true` 时，千问-VL 模型才支持按 Token 用量计费。
- **视频模型训练时长**：视频生成模型微调需**数小时**，具体耗时取决于基础模型和数据量。

## 来源文档

- [百炼文件管理 API](../../raw/model-api-reference/model-training/model-customization-file-management-service.md)
- [模型调优 API 参考](../../raw/model-api-reference/model-training/model-training-api-reference.md)
- [视频生成模型微调API参考](../../raw/model-api-reference/model-training/wan-video-generation-finetune-api-reference.md)
- [paraformer热词](../../raw/model-api-reference/model-training/paraformer-asr-phrase-manager.md)

