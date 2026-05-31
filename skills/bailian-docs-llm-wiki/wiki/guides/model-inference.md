# model inference

百炼平台提供覆盖文本、图像、视频、音频、3D 等多种模态的模型推理服务，开发者可通过 [[openai-compatible-api|OpenAI 兼容接口]]、DashScope API 或 WebSocket 协议调用。本文汇总各模态的模型选型建议、核心能力、关键参数与使用限制，帮助开发者快速定位适合自身场景的模型。

## 支持的模态与模型概览

百炼模型推理按输入/输出模态可分为以下几大类：

| 模态 | 典型场景 | 推荐入门模型 |
|------|----------|-------------|
| 文本生成 | 聊天机器人、摘要、代码生成 | `qwen3.6-plus` |
| 视觉理解 | 图像分析、视频理解、OCR | `qwen3.6-plus`（多模态） |
| 图片生成与编辑 | 文生图、图片编辑 | `wan2.7-image-pro` |
| 视频生成与编辑 | 文生视频、图生视频、视频编辑 | `happyhorse-1.0-t2v` |
| 3D 模型生成 | 文生3D、图生3D | `Tripo/Tripo-P1.0` |
| 语音合成（TTS） | 标准合成、声音复刻、声音设计 | `cosyvoice-v3.5-plus` |
| 语音识别（ASR） | 实时/非实时语音转文本 | `fun-asr-realtime` / `fun-asr` |
| 语音转语音（S2S） | 语音对话、同声传译 | `qwen3.5-omni-plus-realtime` |
| 向量与重排序 | 语义搜索、RAG 检索 | `text-embedding-v4`、`qwen3-rerank` |
| 全模态 | 音视频分析、多模态对话 | `qwen3.5-omni-plus` |
| 音乐生成 | 歌曲创作 | `fun-music-v1`（邀测中） |

## 文本生成

详见 [文本生成](../../raw/model-user-guide/model-inference/text-generation-model.md)。

### 模型选型

按能力档位选择：

- **旗舰**：`qwen3.7-max` — 最强推理能力，百万 Token 上下文，成本较高
- **平衡**：`qwen3.6-plus` — 能力与成本均衡，100 万上下文，完整工具链，推荐作为默认选择
- **轻量**：`qwen3.6-flash` — 效果接近旗舰，成本更低，上下文同为 100 万

第三方模型可选 `deepseek-v4-pro`、`glm-5.1`、`kimi-k2.6`、`MiniMax-M2.5` 等，但部分不支持内置工具和结构化输出。

### 核心能力

- **思考模式**：通过 `enable_thinking` 参数开启逐步推理，适用于数学计算、代码调试等场景。所有 Qwen3 及以上模型支持。详见 [[deep-thinking]]。
- **Function Calling 与内置工具**：所有通用模型支持 Function Calling；`qwen3.6-plus` / `qwen3.6-flash` 还支持联网搜索、代码解释器等内置工具。详见 [[tool-calls]]。
- **结构化输出**：获取有效 JSON 返回。Qwen 系列支持，DeepSeek 系列不支持。详见 [[structured-output]]。
- **批量推理**：适用于大量请求且对延迟不敏感的场景，可降低成本。仅部分模型支持。详见 [[batch-inference]]。

### 上下文窗口

100 万 Token 约相当于 70 万汉字。`qwen3.6-plus` / `qwen3.6-flash` 支持 1M 上下文；常规任务 128k–256k 已足够。

## 视觉理解

详见 [视觉理解](../../raw/model-user-guide/model-inference/vision-model.md)。

- **图像分析**：推荐 `qwen3.6-plus`，支持最多 256 张图片，每张最高 1600 万像素。Token 消耗公式：`h × w / (32 × 32) + 2`。
- **视频理解**：`qwen3.6-plus` / `qwen3.6-flash` 支持最长 2 小时 / 2GB 视频，最多 64 个视频。
- **OCR / 文档提取**：`qwen-vl-ocr` 专为文档、表格、手写内容优化。

视觉模型同样支持 Function Calling、内置工具和结构化输出（Qwen3.6 / Qwen3.5 系列）。

## 图片生成与编辑

详见 [图片生成与编辑](../../raw/model-user-guide/model-inference/image-model.md)。

| 模型 | 适用场景 | 最大分辨率 |
|------|----------|-----------|
| `wan2.7-image-pro` | 文生图 + 编辑 + 文字渲染 + 角色一致性 | 4096×4096（文生图）/ 2048×2048（编辑） |
| `z-image-turbo` | 快速生成、写实人像，速度快 10x、成本约 1/5 | 2048×2048 |
| `qwen-image-2.0-pro` | 负向提示词、最多 6 张变体 | 2048×2048 |

## 视频生成与编辑

- **文生视频**：推荐 `happyhorse-1.0-t2v`（有声视频，最长 15 秒，1080P）
- **图生视频**：首帧推荐 `happyhorse-1.0-i2v`；首尾帧推荐 `wan2.7-i2v-2026-04-25`
- **参考生视频**：`happyhorse-1.0-r2v`（图片参考）/ `wan2.7-r2v`（视频参考）
- **视频编辑**：`happyhorse-1.0-video-edit`（通用编辑）/ `wan2.7-videoedit`（特效/运镜复刻）
- **角色动画**：`wan2.2-animate-move`（动作迁移）/ `wan2.2-animate-mix`（人物替换）

如需自定义音频文件输入，选择 Wan 2.7 系列模型。

## 3D 模型生成

Tripo 模型支持文生3D、单图生3D 和多图生3D 三种模式，通过异步任务 API 调用。

- `Tripo/Tripo-P1.0`：快速预览，最高 2 万面，适合游戏/AR
- `Tripo/Tripo-H3.1`：高精度，最高 200 万面，适合影视级渲染

> **注意**：Tripo 3D 模型生成**仅适用于中国内地（北京）地域**，必须使用该地域的 API Key。

## 语音合成（TTS）

- **标准合成**：`cosyvoice-v3-plus` 或 `MiniMax/speech-2.8-hd`，选择内置音色即可使用
- **声音复刻**：`cosyvoice-v3.5-plus`，从音频样本复刻音色
- **声音设计**：`cosyvoice-v3.5-plus`，用文字描述创建全新音色
- **指令控制**：CosyVoice 系列和 Qwen3-TTS-Instruct 系列支持用自然语言控制语速、情绪和风格

接入方式：**WebSocket**（双向流式，延迟最低，适合实时交互）和 **HTTP**（完整文本输入，支持流式返回）。

## 语音识别（ASR）

按场景选择：

| 需求 | 推荐模型 | 协议 |
|------|----------|------|
| 实时识别 + 热词 | `fun-asr-realtime` | WebSocket |
| 非实时识别 + 说话人分离 | `fun-asr` | HTTP |
| 实时 + Prompt 上下文注入 | `qwen3.5-omni-plus-realtime` | WebSocket |
| 情感识别 | `qwen3-asr-flash-realtime` / `qwen3-asr-flash-filetrans` | WebSocket / HTTP |

> **注意**：仅 Fun-ASR 系列的非实时模型（`fun-asr`、`fun-asr-mtl`）支持说话人分离。Qwen3.5-Omni 不是传统 ASR，而是通过 Prompt 注入上下文实现自适应识别。

## 语音转语音（S2S）与全模态

S2S 单模型路线使用 Qwen3.5-Omni 或 Livetranslate 系列，端到端处理语音输入输出，延迟低且能感知语调/情绪。Pipeline 路线（ASR + LLM + TTS 串行）可为每个环节选择最优模型，但延迟较高。

- **实时语音对话**：`qwen3.5-omni-plus-realtime`（WebSocket）
- **同声传译**：`qwen3.5-livetranslate-flash-realtime`（60 种语言）
- **视频配音/播客翻译**：`qwen3-livetranslate-flash`（HTTP）

全模态模型 Qwen3.5-Omni 同时支持 Function Calling 和联网搜索（两者不可同时开启）。

## 向量与重排序

- **文本 Embedding**：`text-embedding-v4`，支持 64–2048 维，默认 1024 维
- **多模态 Embedding**：`qwen3-vl-embedding`（融合+独立向量）/ `tongyi-embedding-vision-plus`（独立向量）
- **重排序**：`qwen3-rerank`（纯文本，100+ 语言，最多 500 文档）/ `qwen3-vl-rerank`（多模态）

维度选择建议：大规模搜索选 256/512 维；通用场景选 1024 维；高精度选 1536/2048 维。

## 关键参数速查

| 参数/概念 | 说明 | 适用模型 |
|-----------|------|----------|
| `enable_thinking` | 开启思考模式（逐步推理） | Qwen3 及以上文本/视觉模型 |
| `reasoning.effort` | Responses API 控制思考深度 | Qwen3 及以上 |
| `texture_quality` | 3D 模型贴图质量（`standard` / `detailed`） | Tripo 系列 |
| `geometry_quality` | 3D 几何精度（`standard` / `ultra`） | 仅 Tripo-H3.1 |
| `X-DashScope-Async: enable` | 异步任务模式 | 视频生成、3D 生成 |
| `X-DashScope-SSE: enable` | [[streaming|流式输出]] | 音乐生成等 |

## 通用限制与注意事项

- 所有异步任务（视频生成、3D 生成）的 `task_id` 查询有效期为 **24 小时**，建议轮询间隔 15 秒或配置 [[async-task-callback]]。
- 3D 生成的输出文件（`pbr_model_url`、`rendered_image_url`）有效期 **2 小时**，需及时下载。
- 音乐生成模型 `fun-music-v1` 目前处于**邀测阶段**，仅在中国内地（北京）地域可用。
- 旧版模型（Qwen2.5、Qwen3 早期版本、Paraformer 等）不再作为首选推荐，新项目建议使用 Qwen3.6 / Qwen3.5 系列。
- 第三方模型（DeepSeek、GLM、Kimi、MiniMax）在内置工具、结构化输出、批量推理等功能上的支持有限，选型时注

## 来源文档

- [文本生成](../../raw/model-user-guide/model-inference/text-generation-model.md)
- [图片生成与编辑](../../raw/model-user-guide/model-inference/image-model.md)
- [视觉理解](../../raw/model-user-guide/model-inference/vision-model.md)
- [视频生成与编辑](../../raw/model-user-guide/model-inference/video-generate-edit-model.md)
- [Tripo 3D模型生成](../../raw/model-user-guide/model-inference/tripo-3d-generation-guide.md)
- [语音合成](../../raw/model-user-guide/model-inference/tts-model.md)
- [音乐生成](../../raw/model-user-guide/model-inference/fun-music.md)
- [语音识别](../../raw/model-user-guide/model-inference/asr-model.md)
- [语音转语音](../../raw/model-user-guide/model-inference/s2s-model.md)
- [向量与重排序](../../raw/model-user-guide/model-inference/embedding-rerank-model.md)
- [全模态](../../raw/model-user-guide/model-inference/omni.md)

