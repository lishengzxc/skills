# fine tuning

阿里云百炼平台的模型调优（Fine-Tuning）功能支持对文本生成、视觉理解、视频生成和语音合成等多种模型进行定制化训练，以提升模型在特定行业或业务场景下的表现。调优方式包括监督微调（SFT）、继续预训练（CPT）和直接偏好优化（DPO），三者可递进组合使用。所有调优功能仅适用于中国内地部署模式（北京地域）。

## 调优方式与适用场景

百炼提供三种调优方式，推荐按 `CPT（可选）→ SFT → DPO（可选）` 的顺序递进使用（详见 [模型调优简介](../../raw/model-user-guide/fine-tuning/fine-tune-text-generation-model/model-training-overview.md)）：

| 方式 | 核心目标 | 数据要求 | 典型场景 |
|------|----------|----------|----------|
| **CPT（继续预训练）** | 注入领域知识 | 1000 万+ Token 无标签文本 | 金融术语、医疗病理、法律判例 |
| **SFT（监督微调）** | 学会遵循指令 | 1000+ 条高质量"问-答"对 | 客服流程、代码编写、工具调用 |
| **DPO（直接偏好优化）** | 对齐人类偏好 | 100+ 组"更好-更差"回答对 | 抑制幻觉、安全合规、风格对齐 |

每种方式又分为**全参训练**和**高效训练（LoRA）**两种模式。全参训练效果更好但耗时更长，LoRA 收敛更快、适合对成本敏感的场景。

> **注意**：在考虑模型调优之前，建议先尝试 [[prompt-engineering]] 和 [[function-calling]]，调优通常作为改进模型表现的"最后手段"。前期 Prompt 迭代的工作可复用于构建调优数据集。

## 支持的模型

### 文本生成（Qwen 系列）

支持 Qwen3.6-Flash、Qwen3.5、Qwen3、Qwen2.5 等系列模型，不同模型对 CPT/SFT/DPO 的支持情况有所差异。例如：

- **Qwen3-32B**：支持全部五种训练类型（CPT、SFT 全参、SFT 高效、DPO 全参、DPO 高效）
- **Qwen3-8B**：支持 SFT 和 DPO（全参+高效），不支持 CPT
- **Qwen3.5-27B / 9B**：仅支持 SFT（全参+高效）

> **注意**：[模型调优简介](../../raw/model-user-guide/fine-tuning/fine-tune-text-generation-model/model-training-overview.md) 中列出的支持模型列表（含 Qwen3.6-Flash-2026-04-16、Qwen3.5-Flash-2026-02-23）比 [在控制台进行模型调优](../../raw/model-user-guide/fine-tuning/fine-tune-text-generation-model/model-training-on-console.md) 中的列表更全，部分模型可能仅支持通过 API 方式发起训练，请以实际控制台/API 可选项为准。

### 视觉理解（Qwen-VL 系列）

支持 Qwen3-VL-8B/4B、Qwen2.5-VL-72B/32B/7B 等，仅支持 SFT（全参+高效），暂不支持 CPT 和 DPO。

### 视频生成（万相系列）

仅支持 SFT-LoRA 高效微调，适用模型包括：
- **图生视频-基于首帧**：wan2.5-i2v-preview、wan2.2-i2v-flash
- **图生视频-基于首尾帧**：wan2.2-kf2v-flash

详见 [微调视频生成模型](../../raw/model-user-guide/fine-tuning/wan-video-generation-finetune-guide.md)。

### 语音合成（CosyVoice）

仅支持 `cosyvoice-v3-flash` 模型的 SFT 高效微调（`efficient_sft`），且仅支持通过 API 发起，控制台暂不支持。详见 [CosyVoice模型调优](../../raw/model-user-guide/fine-tuning/fine-tune-speech-synthesis-model/fine-tune-speech-synthesis-model-by-api.md)。

## 训练数据格式

### SFT 数据集（文本生成）

采用 ChatML 格式的 `.jsonl` 文件，支持多轮对话：

```json
{"messages": [
  {"role": "system", "content": "系统输入"},
  {"role": "user", "content": "用户输入"},
  {"role": "assistant", "content": "期望的模型输出"}
]}
```

- **思考模型（Thinking）**：仅最后一个 assistant 输出中可包含 `<think>` 标签
- **视觉理解（VL）**：content 使用数组格式 `[{"text":"..."}, {"image":"xxx.jpg"}]`，训练集需打包为 ZIP（含 `data.jsonl` 和图片/视频文件）
- 不支持 OpenAI 的 `name`、`weight` 参数

### DPO 数据集

在 SFT 格式基础上增加 `chosen`（正样本）和 `rejected`（负样本）字段。

### CPT 数据集

纯文本格式：`{"text":"文本内容"}`

### 视频生成数据集

ZIP 格式，包含视频文件和对应的描述信息，具体结构参见视频微调文档。

### CosyVoice 数据集

ZIP 格式，包含 `data.jsonl`（文本标注）和 `train/` 目录下的 `.wav` 音频文件。推荐训练音频总时长 1~10 小时、不少于 150 条。

## 关键超参数

### 文本生成模型

| 参数 | 推荐值 | 说明 |
|------|--------|------|
| `learning_rate` | 高效训练 1e-4；全参训练 1e-5 | 过高导致效果不稳定，过低导致变化不明显 |
| `n_epochs` | 数据<1万条：3~5；数据>1万条：1~2 | 循环次数，直接影响训练时间和费用 |
| `batch_size` | 16 或 32 | 每次更新参数的数据步长 |
| `max_length` | 设为模型支持的最大值 | SFT 超长数据会被丢弃；DPO 会截断 |
| `lr_scheduler_type` | linear 或 inverse_sqrt | 学习率调整策略 |
| `lora_rank` | 设为最大值 | 仅高效训练，秩越大效果越好但训练略慢 |

### CosyVoice 模型

涉及 LM 和 FM 两个子网络，分别以 `lm_*` 和 `fm_*` 前缀区分，推荐初始值：`lm_max_epoch=60, fm_max_epoch=100`。

## 使用方式

### 控制台（文本生成/VL 模型）

通过百炼控制台的可视化界面操作，无需编码。流程：创建训练任务 → 选择模型和训练方式 → 上传数据 → 配置参数 → 开始训练 → [[model-deployment]] → [[model-evaluation]]。

具体操作参见 [在控制台进行模型调优](../../raw/model-user-guide/fine-tuning/fine-tune-text-generation-model/model-training-on-console.md)。

### API / 命令行

通过 HTTP API 或 DashScope Shell 命令操作，适用于所有模型类型。核心流程：

1. **上传数据集**：`POST /api/v1/files`，获取 `file_id`
2. **创建调优任务**：`POST /api/v1/fine-tunes`，获取 `job_id` 和 `finetuned_output`
3. **轮询任务状态**：`GET /api/v1/fine-tunes/<job_id>`，等待 `status` 变为 `SUCCEEDED`
4. **部署模型**：`POST /api/v1/deployments`，等待 `status` 变为 `RUNNING`
5. **调用模型**：使用 `deployed_model` 作为 model 参数发起推理请求

> **注意**：通过 API 创建的训练任务仅支持按 Token 计费，暂不支持使用模型训练单元（预付费或后付费）。如需使用训练单元，请通过控制台创建任务。

## 计费

训练费用按 `（训练数据 Token 总数 + 混合训练数据 Token 总数）× 循环次数 × 训练单价` 计算。不同模型单价差异较大，例如：

- Qwen3-8B：¥0.006/千Token
- Qwen2.5-72B-Instruct：¥0.15/千Token
- CosyVoice：¥0.2/千Token

调优后的模型需要 [[model-deployment]] 才能使用，部署产生额外费用。

## 限制和注意事项

- **地域限制**：所有调优功能仅适用于中国内地部署模式下的北京地域，需使用该地域的 [[api-key]]
- **权限要求**：RAM 子账号需授予模型调用、训练和部署权限
- **文件限制**：单个文件最大 300MB，有效文件总空间 5GB，总数量 100 个；VL 训练集 ZIP 最大 2GB
- **训练耗时**：文本模型训练通常需数小时，视频模型训练需更长时间，CosyVoice 约 30~60 分钟
- **部署要

## 来源文档

- [微调视频生成模型](../../raw/model-user-guide/fine-tuning/wan-video-generation-finetune-guide.md)
- [模型调优简介](../../raw/model-user-guide/fine-tuning/fine-tune-text-generation-model/model-training-overview.md)
- [在控制台进行模型调优](../../raw/model-user-guide/fine-tuning/fine-tune-text-generation-model/model-training-on-console.md)
- [使用 API 或命令行进行模型调优](../../raw/model-user-guide/fine-tuning/fine-tune-text-generation-model/fine-tuning-api-guide.md)
- [0 代码强化大模型安全合规能力](../../raw/model-user-guide/fine-tuning/fine-tune-text-generation-model/enhance-the-security-compliance-of-large-models.md)
- [CosyVoice模型调优](../../raw/model-user-guide/fine-tuning/fine-tune-speech-synthesis-model/fine-tune-speech-synthesis-model-by-api.md)

