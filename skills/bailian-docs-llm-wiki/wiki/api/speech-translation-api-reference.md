# speech translation api reference

百炼平台提供两类语音翻译 API：一是基于 [[openai-compatible-api|OpenAI 兼容接口]]的**音视频文件翻译**（`qwen3-livetranslate-flash`），适用于对已有音频或视频文件进行翻译；二是基于 WebSocket 的**实时音视频翻译**（`qwen3.5-livetranslate-flash-realtime`），适用于流式音频输入的低延迟场景。两类 API 均支持文本和音频双模态输出，并可配置目标语种、音色、热词等参数。

---

## 支持的模型

| 类型 | 模型名称 | 接口协议 | 说明 |
|------|----------|----------|------|
| 文件翻译 | `qwen3-livetranslate-flash` | OpenAI 兼容（HTTP） | 支持音频/视频文件输入 |
| 文件翻译 | `qwen3-livetranslate-flash-2025-12-01` | OpenAI 兼容（HTTP） | 日期快照版本 |
| 实时翻译 | `qwen3.5-livetranslate-flash-realtime` | WebSocket | 推荐使用，默认音色 `Tina` |
| 实时翻译（旧版） | `qwen3-livetranslate-flash-realtime` | WebSocket | 旧版模型，默认音色 `Cherry` |

> **注意**：文件翻译模型不支持通过 DashScope 接口调用，仅支持 [[openai-compatible-api|OpenAI 兼容接口]]。实时翻译模型则通过 DashScope WebSocket 接口调用。

---

## 文件翻译 API（OpenAI 兼容）

详见 [音视频翻译-通义千问 API 参考](../../raw/model-api-reference/speech-translation-api-reference/qwen3-livetranslate-flash-api.md)。

### 接入地址

| 地域 | base_url |
|------|----------|
| 北京 | `https://dashscope.aliyuncs.com/compatible-mode/v1` |
| 新加坡 | `https://dashscope-intl.aliyuncs.com/compatible-mode/v1` |

### 关键请求参数

| 参数 | 类型 | 必选 | 说明 |
|------|------|------|------|
| `model` | string | 是 | 模型名称 |
| `messages` | array | 是 | 仅支持一个 User Message，内容为 `input_audio` 或 `video_url` |
| `stream` | boolean | 是 | 必须设为 `true`，仅支持[[streaming|流式输出]] |
| `modalities` | array | 否 | `["text","audio"]` 或 `["text"]`，默认仅文本 |
| `audio` | object | 否 | 输出音频配置：`voice`（音色）和 `format`（仅支持 `wav`） |
| `translation_options` | object | 是 | 翻译配置，包含 `source_lang`（可选，自动识别）和 `target_lang`（必选） |

> **注意**：`translation_options`、`top_k`、`repetition_penalty` 为非 OpenAI 标准参数。Python SDK 调用时需放入 `extra_body`；Node.js SDK 或 HTTP 调用时作为顶层参数传递。

### 输入格式

- **音频输入**：`type` 设为 `input_audio`，通过 `data` 字段传入音频 URL 或 Base64 Data URL，`format` 指定格式（如 `mp3`、`wav`）
- **视频输入**：`type` 设为 `video_url`，通过 `url` 字段传入视频文件的公网 URL 或 Base64 Data URL

### 响应格式

[[streaming|流式输出]]包含三类 chunk：

1. **文本 chunk**：`delta.content` 包含增量翻译文本
2. **音频 chunk**：`delta.audio.data` 包含 Base64 编码的增量音频数据
3. **Token 消耗 chunk**：`usage` 包含 `[[prompt|prompt]]_tokens`、`completion_tokens` 及其明细（文本/音频 Token 分项）

### 采样参数

`temperature`（默认 0.000001）、`top_p`（默认 0.8）、`top_k`（默认 1）、`presence_penalty`（默认 0）、`repetition_penalty`（默认 1.05）等参数均可配置，但**为保证翻译准确性，不建议修改**。

---

## 实时翻译 API（WebSocket）

实时翻译通过 WebSocket 连接进行，支持 DashScope Python SDK 和 Java SDK，也可直接构造 WebSocket 消息。

### 接入地址

| 地域 | WebSocket URL |
|------|---------------|
| 中国内地 | `wss://dashscope.aliyuncs.com/api-ws/v1/realtime` |
| 国际 | `wss://dashscope-intl.aliyuncs.com/api-ws/v1/realtime` |

### 交互流程

1. 建立 WebSocket 连接，服务端返回 `session.created` 事件
2. 发送 `session.update` 事件配置会话参数（音色、目标语种、热词等）
3. 持续发送 `input_audio_buffer.append` 追加音频数据（可选发送 `input_image_buffer.append` 追加图像）
4. 服务端自动检测语音起止，通过事件流返回翻译文本和音频
5. 发送 `session.finish` 结束会话，等待 `session.finished` 后断开连接

### 客户端事件

详见 [客户端事件](../../raw/model-api-reference/speech-translation-api-reference/live-translator-api/live-translator-client-events.md)。

| 事件类型 | 说明 |
|----------|------|
| `session.update` | 更新会话配置（模态、音色、语种、热词、声音复刻等） |
| `input_audio_buffer.append` | 追加 Base64 编码的音频数据 |
| `input_image_buffer.append` | 追加 Base64 编码的图像数据（JPG/JPEG，≤500KB，≤2帧/秒） |
| `session.finish` | 通知服务端结束会话 |

### 会话配置参数（session.update）

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `modalities` | array | `["text","audio"]` | 输出模态 |
| `voice` | string | `Tina`（3.5）/ `Cherry`（3） | 音色，启用声音复刻时需设为 `default` 或复刻音色 ID |
| `enable_voice_clone` | boolean | `false` | 是否启用声音复刻 |
| `voice_clone_options.frequency` | string | — | 复刻频率：`never`/`once`/`always` |
| `sample_rate` | integer | 16000 | 输入采样率（Hz），可选 8000 或 16000 |
| `input_audio_format` | string | `pcm` | 输入格式：`pcm` 或 `opus` |
| `output_audio_format` | string | `pcm` | 输出格式，当前仅支持 `pcm` |
| `input_audio_transcription.model` | string | — | 设为 `qwen3-asr-flash-realtime` 可获取源语言识别结果 |
| `input_audio_transcription.language` | string | `en` | 源语种 |
| `translation.language` | string | `en` | 目标语种 |
| `translation.corpus.phrases` | object | — | 热词映射表，如 `{"人工智能": "Artificial Intelligence"}` |

### 服务端事件

详见 [服务端事件](../../raw/model-api-reference/speech-translation-api-reference/live-translator-api/live-translator-server-events.md)。

| 事件类型 | 说明 |
|----------|------|
| `session.created` | 连接建立后返回默认配置 |
| `session.updated` | 配置更新成功 |
| `session.finished` | 会话结束 |
| `error` | 错误信息 |
| `response.created` | 新响应开始 |
| `response.done` | 响应完成，含 Token 用量 |
| `response.text.text` | 纯文本模态下的增量文本（含 `stash` 临时文本） |
| `response.text.done` | 纯文本模态下文本生成完成 |
| `response.audio.delta` | 音频模态下的增量音频（Base64） |
| `response.audio.done` | 音频生成完成（不含完整音频） |
| `response.audio_transcript.text` | 音频模态下的增量翻译文本（含 `stash`） |
| `response.audio_transcript.done` | 音频模态下翻译文本完成 |
| `conversation.item.input_audio_transcription.text` | 源语言识别中间结果 |
| `conversation.item.input_audio_transcription.completed` | 源语言识别最终结果 |

> **注意**：`text` 与 `stash` 的关系——`text` 为已确认文本，`stash` 为临时文本，两者拼接构成当前最佳结果；最终以 `done` 事件中的完整文本为准。

---

## SDK 使用方式

### Python SDK

- SDK 版本要求：DashScope SDK ≥ 1.25.6
- 核心类：`OmniRealtimeConversation`、`OmniRealtimeCallback`、`TranslationParams`
- 关键方法：`connect()`、`update_session()`、`append_audio()`、`end_session()`、`close()`

详见 [实时音视频翻译（Qwen-LiveTranslate）Python SDK-API参考](../../raw/model-api-reference/speech-translation-api-reference/live-translator-api/qwen-livetranslate-python-sdk.md)。

### Java SDK

- SDK 版本要求：DashScope SDK ≥ 2.22.5
- 核心类：`OmniRealtimeConversation`、`OmniRealtimeCallback`、`OmniRealtimeParam`、`OmniRealtimeConfig`
- 关键方法：`connect()`、`updateSession()`、`appendAudio()`、`endSession()`、`close()`
- 额外提供 `getFirstTextDelay()` 和 `getFirstAudioDelay()` 用于获取首包延迟

详见 [实时音视频翻译（Qwen-LiveTranslate）Java SDK-API参考](../../raw/model-api-reference/speech-translation-api-reference/live-translator-api/qwen-livetranslate-java-sdk.md)。

---

## 限制与注意事项

- **文件翻译仅支持流式调用**：`stream` 必须设为 `true`，不支持非流式响应
- **图像输入限制**（实时翻译）：仅支持 JPG/JPEG，单张 ≤ 500KB，频率 ≤ 2帧/秒，且需先发送过音频数据
- **声音复刻与音色互斥**：启用 `enable_voice_clone` 时不可使用系统预设音色，需设为 `default` 或预先复刻的音色 ID
- **地域隔离**：北京和新加坡地域的 API Key 不同，接入地址也不同，需注意区分
- **非标准参数传递**：`translation_options`（文件翻译）、`top_k`、`repetition_penalty` 等参数在 Python SDK 中需通过 `extra_body` 传递

---

## 相关概念

- [[qwen-omni]]：通义千问全模态模型
- [[speech-recognition]]：语音识别相关 API
- [[voice-cloning]]：声音复刻 API
- [[api-key-configuration]]：API Key 获取与配置

## 来源文档

- [音视频翻译-通义千问 API 参考](../../raw/model-api-reference/speech-translation-api-reference/qwen3-livetranslate-flash-api.md)
- [客户端事件](../../raw/model-api-reference/speech-translation-api-reference/live-translator-api/live-translator-client-events.md)
- [服务端事件](../../raw/model-api-reference/speech-translation-api-reference/live-translator-api/live-translator-server-events.md)
- [实时音视频翻译（Qwen-LiveTranslate）Python SDK-API参考](../../raw/model-api-reference/speech-translation-api-reference/live-translator-api/qwen-livetranslate-python-sdk.md)
- [实时音视频翻译（Qwen-LiveTranslate）Java SDK-API参考](../../raw/model-api-reference/speech-translation-api-reference/live-translator-api/qwen-livetranslate-java-sdk.md)

