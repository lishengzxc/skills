# omni realtime api

Qwen-Omni-Realtime API 是百炼平台提供的实时多模态交互接口，基于 WebSocket 协议，支持音频、视频、文本的实时双向通信。该 API 支持语音活动检测（VAD）、工具调用（Function Calling）、联网搜索、声音复刻等能力，适用于语音助手、实时客服等场景。

## 支持的模型

| 模型系列 | 默认音色 | 特性说明 |
|---------|---------|---------|
| qwen3.5-omni-realtime | Tina | 支持 semantic_vad、联网搜索、工具调用 |
| qwen3-omni-flash-realtime | Cherry | 支持 smooth_output 参数 |
| qwen-omni-turbo-realtime | Chelsie | 多数生成参数不支持修改 |

声音复刻场景还支持 `qwen3.5-omni-plus-realtime` 和 `qwen3.5-omni-flash-realtime` 作为驱动模型，详见 [声音复刻API参考](../../raw/model-api-reference/omni-realtime-api/qwen-omni-voice-cloning.md)。

## 交互模式

根据 [实时多模态交互流程](../../raw/model-api-reference/omni-realtime-api/omni-realtime-interaction-process.md)，API 支持两种交互模式：

### VAD 模式（默认）

将 `session.turn_detection` 设为 `"server_vad"` 或 `"semantic_vad"`。服务端自动检测语音起止并触发模型响应，支持语音打断。适用于持续音频流输入场景。

### Manual 模式

将 `session.turn_detection` 设为 `null`。客户端需手动发送 `input_audio_buffer.commit` 提交音频，再发送 `response.create` 触发响应。适用于按下即说场景。

## 客户端事件

基于 [客户端事件](../../raw/model-api-reference/omni-realtime-api/client-events.md) 文档，主要事件如下：

| 事件类型 | 用途 |
|---------|------|
| `session.update` | 更新会话配置（模态、音色、VAD、工具等） |
| `input_audio_buffer.append` | 追加 Base64 编码的音频数据 |
| `input_audio_buffer.commit` | 提交音频缓冲区（Manual 模式必需） |
| `input_audio_buffer.clear` | 清除音频缓冲区 |
| `input_image_buffer.append` | 追加 Base64 编码的图像数据 |
| `response.create` | 触发模型生成响应 |
| `response.cancel` | 取消正在进行的响应 |
| `conversation.item.create` | 回传工具调用结果 |

## 关键参数

### 音频格式

- **输入音频**：PCM，16 kHz 采样率，单声道 16bit
- **输出音频**：PCM，24 kHz 采样率，单声道 16bit

### VAD 配置

| 参数 | 说明 | 默认值 |
|------|------|--------|
| `type` | `server_vad`（声学检测）或 `semantic_vad`（语义检测，仅 qwen3.5-omni-realtime 支持） | `server_vad` |
| `threshold` | 灵敏度，范围 [-1.0, 1.0] | 0.5 |
| `silence_duration_ms` | 静音触发时间（ms），范围 [200, 6000] | 800 |

### 生成参数

| 参数 | qwen3.5-omni-realtime | qwen3-omni-flash-realtime | qwen-omni-turbo-realtime |
|------|----------------------|--------------------------|-------------------------|
| temperature | 0.7 | 0.9 | 1.0（不可改） |
| top_p | 0.8 | 1.0 | 0.01（不可改） |
| top_k | 20 | 50 | 20（不可改） |
| repetition_penalty | 1.0 | 1.05 | 1.05（不可改） |
| presence_penalty | 1.5 | 0.0 | 0.0（不可改） |

> **注意**：`qwen-omni-turbo-realtime` 系列不支持修改 temperature、top_p、top_k、max_tokens、repetition_penalty、presence_penalty、seed 等参数。

### 图像输入限制

- 格式：JPG/JPEG，建议 480p~720p，最高 1080p
- 单张 Base64 编码后不超过 256KB（建议原始图片 ≤ 190KB）
- 建议发送频率：1 张/秒
- 发送图像前需至少发送过一次音频数据

## 工具调用与联网搜索

- **工具调用**：通过 `session.update` 的 `tools` 字段定义工具，模型自主判断是否调用。触发工具调用后，客户端执行函数并通过 `conversation.item.create` 回传结果。
- **联网搜索**：仅 qwen3.5-omni-realtime 支持，通过 `enable_search: true` 启用。

> **注意**：`tools` 和 `enable_search` 不兼容，不可同时开启。

## SDK 使用

### Python SDK

```python
from dashscope.audio.qwen_omni import OmniRealtimeConversation, OmniRealtimeCallback, MultiModality

conv = OmniRealtimeConversation(model="qwen3-omni-flash-realtime", callback=callback)
conv.connect()
conv.update_session(
    output_modalities=[MultiModality.AUDIO, MultiModality.TEXT],
    voice="Cherry",
    enable_turn_detection=True
)
# 持续发送音频
conv.append_audio(base64_audio_data)
```

SDK 版本要求：Python ≥ 1.25.17，Java ≥ 2.22.15。

详细接口参见 [Python SDK](../../raw/model-api-reference/omni-realtime-api/omni-realtime-python-sdk.md) 和 [[omni-realtime-java-sdk]]。

## 声音复刻

声音复刻通过 `qwen-voice-enrollment` 模型，上传 10~20 秒音频即可创建专属音色，无需训练。创建时需指定 `target_model`，后续对话时使用的模型必须与之一致。

音频要求：WAV/MP3/M4A，采样率 ≥ 24 kHz，单声道，≤ 10 MB，最长 60 秒。

## 限制和注意事项

- 音频格式当前仅支持 PCM，不支持自定义输出采样率
- `semantic_vad` 仅 `qwen3.5-omni-realtime` 模型支持
- `smooth_output` 参数仅 `qwen3-omni-flash-realtime` 系列生效
- 联网搜索仅 `qwen3.5-omni-realtime` 支持
- 音频缓冲区最大 15 MiB（Manual 模式下单次）
- 语音转录模型固定为 `qwen3-asr-flash-realtime`，不可修改
- 推荐使用耳机播放音频，避免回声触发语音打断

## 来源文档

- [客户端事件](../../raw/model-api-reference/omni-realtime-api/client-events.md)
- [服务端事件](../../raw/model-api-reference/omni-realtime-api/server-events.md)
- [Python SDK](../../raw/model-api-reference/omni-realtime-api/omni-realtime-python-sdk.md)
- [Java SDK](../../raw/model-api-reference/omni-realtime-api/omni-realtime-java-sdk.md)
- [实时多模态交互流程](../../raw/model-api-reference/omni-realtime-api/omni-realtime-interaction-process.md)
- [声音复刻API参考](../../raw/model-api-reference/omni-realtime-api/qwen-omni-voice-cloning.md)

