# model data overview

百炼平台的模型数据功能提供对大模型训练集和评测集的统一管理，涵盖数据集创建、格式规范、数据清洗与增强等完整流程。通过 [模型数据控制台](https://bailian.console.aliyun.com/#/efm/model_data) 可以集中管理业务空间下所有与大模型相关的数据集。本文汇总了数据集类型、格式要求及数据处理能力的核心信息。

> **注意**：以下内容仅适用于中国大陆版（北京地域）。

## 数据集分类

模型数据分为两大类：

| 类型 | 用途 | 支持的子类型 |
|------|------|-------------|
| **训练集** | 用于 [[model-training-overview]]，通过有监督训练提升模型在特定任务上的表现 | 文本生成（SFT / SFT-Thinking / DPO / CPT）、多模态理解（千问VL）、图生视频（首帧）、图生视频（首尾帧） |
| **评测集** | 用于 [[model-evaluation-overview]]，评估调优后模型的泛化能力 | 文本生成 |

详细格式说明见 [训练集与评测集](../../raw/model-user-guide/model-data-overview/training-set-and-evaluation-set.md)。

## 训练集格式规范

### SFT 训练集（文本生成）

采用 ChatML 格式，支持多轮对话和 system / user / assistant 角色。每行一条 JSON 数据：

```json
{"messages": [
  {"role": "system", "content": "系统输入"},
  {"role": "user", "content": "用户输入1"},
  {"role": "assistant", "content": "期望的模型输出1"}
]}
```

- 不支持 OpenAI 的 `name`、`weight` 参数，所有 assistant 输出都会被训练。
- 支持 `loss_weight` 参数（`0.0~1.0`）设置 assistant 行的训练权重（邀测功能）。
- xls/xlsx 格式仅支持单轮对话。

### SFT 思考模型（Thinking）

仅对**最后一个** assistant 输出进行训练，思考内容使用 `<think>` 标签包裹：

```json
{"role": "assistant", "content": "<think>\n思考内容\n</think>\n\n输出内容"}
```

> **注意**：思考标签前后的 `\n` 必须保留。如果训练样本中不包含 `<think>` 标签，训练后不建议再开启思考模式调用。

### SFT 视觉理解（千问VL）

content 字段使用数组格式，支持图片和视频输入。需以 ZIP 压缩包形式上传，包含 `data.jsonl` 和媒体文件。关键约束：

- ZIP 最大 2 GB，文件名仅支持 ASCII 字符
- `data.jsonl` 必须位于压缩包根目录
- 图片单张宽高均不超过 1024px，最大 10MB
- 图片文件名全局唯一（跨文件夹）
- 视频文件路径模式仅 qwen3.5 及以后的 VL 模型支持

### DPO 训练集

在标准 ChatML 对话基础上增加 `chosen` 和 `rejected` 字段，用于训练模型的正负反馈偏好。

### CPT 训练集

纯文本格式：`{"text":"文本内容"}`

### 图生视频训练集

分为**基于首帧**和**基于首尾帧**两种模式，均以 ZIP 包上传，包含 `data.jsonl`、图像和视频文件。验证集为可选，无需提供视频。详细结构参见 [训练集与评测集](../../raw/model-user-guide/model-data-overview/training-set-and-evaluation-set.md)。

## 评测集格式

文本生成评测集为 Excel 格式的单轮对话数据，包含 `Prompt` 和 `Completion` 两列。评测时模型基于 Prompt 推理，评分员参考 Completion 进行评分。

## 数据量建议

| 训练方式 | 最低数据量要求 |
|----------|--------------|
| CPT | 一千万 Token 优质预训练数据 |
| SFT | 上千条优质微调数据 |
| DPO | 上百条人类偏好数据 |

如果数据不足，可考虑构建 [[agent-application]] 并使用知识库索引增强模型能力。

## 数据清洗与增强

当训练集质量较低或数量不足时，可使用平台的数据处理功能进行清洗和增强。详细操作流程见 [数据清洗或增强](../../raw/model-user-guide/model-data-overview/data-processing.md)。

### 适用范围

数据处理目前**仅支持 SFT 文本生成训练集**（ChatML 格式），不支持 SFT 图片理解训练集和 DPO 训练集。

> **注意**：如果训练集包含法律文件、医学记录、文学作品、方言汇总等特殊内容，不建议使用数据清洗与增强功能。

### 数据清洗

修正训练数据的规范性、合规性、一致性及重复问题。支持的算子包括：

- 特殊内容移除（URL、特殊字符等）
- 敏感信息打码（脱敏处理）
- 文章相似度去重
- 敏感词过滤
- 毒性消除等（共十种清洗算子）

### 数据增强

通过 Few-Shot 策略调用千问-Max 模型生成新数据，支持四种场景：

| 场景 | 说明 |
|------|------|
| 数据增强-通用 | 通用场景，适用于 SFT 训练集增强 |
| 数据增强-文本分类 | 针对分类任务优化 |
| 数据增强-文本抽取 | 针对抽取任务优化 |
| 数据增强-文本创作 | 针对创作任务优化 |

每次最多生成 2000 条样本。增强后的训练集自动生成新版本，不覆盖原数据。

### 操作流程

1. **创建数据流**：在数据管理页面搭建自定义数据流，编排清洗和增强节点
2. **创建数据流任务**：选择已发布的数据流和目标训练集，启动处理任务
3. **查看结果**：处理完成后自动生成训练集新版本，可在任务列表查看执行过程

> **注意**：建议先清洗再增强，确保增强操作在干净的数据集上进行。当前暂不提供 API 进行数据处理，仅支持控制台操作。

## 限制与注意事项

- 数据处理执行期间不支持手动终止
- 增强输出中的 `foreignKey` 字段为系统标识，无需删除，不影响 [[model-training-overview]] 效果
- 清洗后建议检查训练集，确保数据完整性和真实性未被破坏
- 数据流需先发布才能创建任务；编辑已发布的数据流会使其回到草稿状态

## 来源文档

- [训练集与评测集](../../raw/model-user-guide/model-data-overview/training-set-and-evaluation-set.md)
- [数据清洗或增强](../../raw/model-user-guide/model-data-overview/data-processing.md)

