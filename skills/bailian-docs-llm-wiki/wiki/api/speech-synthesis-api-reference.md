# speech synthesis api reference

百炼平台提供多种语音合成（TTS）模型和接口，支持将文本转换为高质量语音。平台涵盖 Qwen-TTS、CosyVoice、Sambert 和 MiniMax 等模型系列，提供 HTTP API、WebSocket API 以及多语言 SDK（Python、Java、Android、iOS）等接入方式，满足实时与非实时语音合成的不同场景需求。

## 支持的模型

### Qwen-TTS 系列

| 模型 | 场景 | 接口协议 |
|------|------|----------|
| qwen3-tts-flash | 非实时合成 | HTTP (MultiModalConversation) |
| qwen3-tts-instruct-flash | 非实时合成（指令控制） | HTTP (MultiModalConversation) |
| qwen3-tts-flash-realtime | 实时合成 | WebSocket |
| qwen3-tts-instruct-flash-realtime | 实时合成（指令控制） | WebSocket |
| qwen3-tts-vc-realtime | 实时合成（声音复刻音色） | WebSocket |
| qwen3-tts-vd-realtime | 实时合成（声音设计音色） | WebSocket |

### CosyVoice 系列

| 模型 | 场景 | 接口协议 |
|------|------|----------|
| cosyvoice-v3.5-plus | 实时/非实时 | WebSocket / HTTP |
| cosyvoice-v3.5-flash | 实时/非实时 | WebSocket / HTTP |
| cosyvoice-v3-plus | 实时/非实时 | WebSocket / HTTP |
| cosyvoice-v3-flash | 实时/非实时 | WebSocket / HTTP |
| cosyvoice-v2 | 实时/非实时 | WebSocket / HTTP |

### Sambert 系列

Sambert 模型（如 `sambert-zhichu-v1`）仅支持在北京地域使用，不支持流式输入（所有文本需一次性提交）。

### MiniMax 系列

| 模型 | 说明 |
|------|------|
| MiniMax/speech-2.8-hd | 高清音质 |
| MiniMax/speech-02-hd | 高清音质 |
| MiniMax/speech-2.8-turbo | 低延迟 |
| MiniMax/speech-02-turbo | 低延迟 |

## 接入方式

### HTTP API

适用于非实时合成场景。根据模型不同，端点有所区别：

- **Qwen-TTS 非实时**：通过 `MultiModalConversation` 接口调用，详见 [非实时语音合成（Qwen-TTS）API参考](../../raw/model-api-reference/speech-synthesis-api-reference/qwen-tts-api.md)
- **CosyVoice 非实时**：`POST https://dashscope.aliyuncs.com/api/v1/services/audio/tts/SpeechSynthesizer`，详见 [非实时语音合成CosyVoice HTTP API参考](../../raw/model-api-reference/speech-synthesis-api-reference/non-realtime-cosyvoice-api/cosyvoice-tts-http-api.md)
- **MiniMax**：`POST https://dashscope.aliyuncs.com/api/v1/services/aigc/multimodal-generation/generation`

> **注意**：CosyVoice 非实时 HTTP API 仅在中国内地部署范围（北京地域）下可用。

### WebSocket API

适用于实时合成场景，支持双向流式交互：

- **CosyVoice 实时**：`wss://dashscope.aliyuncs.com/api-ws/v1/inference`，采用 `run-task` → `continue-task` → `finish-task` 交互流程
- **Qwen-TTS 实时**：`wss://dashscope.aliyuncs.com/api-ws/v1/realtime?model=<model_name>`，采用 `session.update` → `input_text_buffer.append` → `input_text_buffer.commit` 交互流程，详见 [Qwen-TTS-Realtime WebSocket API 参考](../../raw/model-api-reference/speech-synthesis-api-reference/qwen-tts-realtime-api-reference/interactive-process-of-qwen-tts-realtime-synthesis.md)
- **Sambert**：`wss://dashscope.aliyuncs.com/api-ws/v1/inference`，仅支持单向[[streaming|流式输出]]（`streaming: "out"`）

### SDK 支持

| SDK | CosyVoice 实时 | CosyVoice 非实时 | Qwen-TTS 实时 | Qwen-TTS 非实时 | Sambert |
|-----|-------|-------|-------|-------|---------|
| Python | ✅ | ✅ | ✅ | ✅ | ✅ |
| Java | ✅ | ✅ | ✅ | ✅ | ✅ |
| Android | ✅ | - | - | - | ✅ |
| iOS | ✅ | - | - | - | ✅ |

## 关键参数

### 通用音频参数

| 参数 | 说明 | CosyVoice | Qwen-TTS Realtime | Sambert |
|------|------|-----------|-------------------|---------|
| format | 音频格式 | pcm/wav/mp3/opus | pcm/wav/mp3/opus | pcm/wav/mp3 |
| sample_rate | 采样率(Hz) | 8000-48000，默认22050 | 8000/16000/24000(默认)/48000 | 8000-24000，默认16000 |
| volume | 音量 | [0,100]，默认50 | [0,100]，默认50 | [0,100]，默认50 |
| rate/speech_rate | 语速 | [0.5,2.0]，默认1.0 | [0.5,2.0]，默认1.0 | [0.5,2.0]，默认1.0 |
| pitch | 音调 | [0.5,2.0]，默认1.0 | - | [0.5,2.0]，默认1.0 |

> **注意**：`cosyvoice-v1` 不支持 opus 格式和 seed 参数。Qwen-TTS-Realtime 的旧版模型（qwen-tts-realtime）仅支持 pcm 格式和 24000 采样率。

### 音色（voice）

音色来源包括：
- **系统音色**：各模型预置音色，参见 [[cosyvoice-voice-list]]
- **声音复刻音色**：通过上传音频创建自定义音色，参见 [[voice-clone-design-http-api]]
- **声音设计音色**：通过文本描述生成音色，参见 [[voice-design-api-references]]

每个模型仅支持一组特定音色，不能跨模型混用。复刻/设计音色时 `target_model` 必须与后续合成使用的模型一致。

### 文本输入限制

| 接口 | 单次文本限制 | 累计限制 |
|------|-------------|---------|
| CosyVoice 非流式 | 20000 字符 | - |
| CosyVoice 流式 | 单次 20000 字符 | 累计 20万字符 |
| Qwen-TTS Realtime | 通过 append 分段提交 | - |
| Sambert | 一次性提交 | - |
| MiniMax | 10000 字符 | - |

### 特色功能参数

- **SSML**：CosyVoice 支持，需设置 `enable_ssml: true`，开启后仅允许发送一次 `continue-task`
- **Instruct**：CosyVoice 部分音色支持通过 `instruction` 参数控制情感/场景
- **字级别时间戳**：CosyVoice 和 Sambert 支持，通过 `word_timestamp_enabled` 开启
- **音素时间戳**：仅 Sambert 支持，通过 `phoneme_timestamp_enabled` 开启
- **指令控制**（Qwen-TTS）：使用 `qwen3-tts-instruct-flash` 模型，通过 `instructions` 参数描述语音风格
- **seed**：CosyVoice 随机种子，相同参数+seed 可复现合成结果，范围 [0, 65535]

## 交互模式

### CosyVoice WebSocket

1. 建立连接 → 2. `run-task`（设置参数）→ 3. 收到 `task-started` → 4. 多次 `continue-task`（发送文本）→ 5. 收到 `result-generated` + binary 音频 → 6. `finish-task` → 7. 收到 `task-finished`

同一任务中所有事件必须使用相同的 `task_id`（UUID 格式）。建议复用 WebSocket 连接处理多个任务。

### Qwen-TTS Realtime WebSocket

支持两种模式：
- **server_commit**（默认）：服务端自动判断合成时机
- **commit**：客户端通过 `input_text_buffer.commit` 手动触发

流程：连接 → `session.created` → `session.update` → 多次 `input_text_buffer.append` → `input_text_buffer.commit`（或自动触发）→ `response.audio.delta`（base64 音频）→ `session.finish`

## 服务地域

| 地域 | HTTP 端点 | WebSocket 端点 |
|------|-----------|---------------|
| 中国内地（北京） | `https://dashscope.aliyuncs.com/api/v1` | `wss://dashscope.aliyuncs.com/api-ws/v1/inference` |
| 国际（新加坡） | `https://dashscope-intl.aliyuncs.com/api/v1` | `wss://dashscope-intl.aliyuncs.com/api-ws/v1/inference` |

> **注意**：不同地域的 API Key 不通用，请确保使用对应地域的 Key。Sambert 仅支持北京地域。

## 鉴权

所有接口统一使用 `Authorization: Bearer <your_api_key>` 请求头。WebSocket 鉴权在握手阶段完成，Key 无效时返回 HTTP 401/403。

建议在移动端（Android/iOS）使用[[generate-temporary-api-key|临时 API Key]]，默认 60 秒有效期，降低泄露风险。

## 限制和注意事项

- 待合成文本必须在所选音色支持的语言范围内，否则可能出现发音错误
- CosyVoice `continue-task` 中的文本会被服务端自动分句，完整语句立即合成，不完整语句缓存等待
- Sambert 不支持 `continue-task` 和 `finish-task`，所有文本必须在 `run-task` 中一次性提交
- MiniMax 文本超过 3000 字符时推荐使用[[streaming|流式输出]]
- DashScope Python SDK 中 `SpeechSynthesizer` 接口（用于 Qwen-TTS 非实时）已统一为 `MultiModalConversation`，使用方法和参数保持一致

## 来源文档

- [非实时语音合成（Qwen-TTS）API参考](../../raw/model-api-reference/speech-synthesis-api-reference/qwen-tts-api.md)
- [声音设计API参考](../../raw/model-api-reference/speech-synthesis-api-reference/voice-design-api-references.md)
- [CosyVoice服务端事件](../../raw/model-api-reference/speech-synthesis-api-reference/cosyvoice-large-model-for-speech-synthesis/cosyvoice-server-events.md)
- [CosyVoice WebSocket API参考](../../raw/model-api-reference/speech-synthesis-api-reference/cosyvoice-large-model-for-speech-synthesis/cosyvoice-websocket-api.md)
- [CosyVoice客户端事件](../../raw/model-api-reference/speech-synthesis-api-reference/cosyvoice-large-model-for-speech-synthesis/cosyvoice-client-events.md)
- [实时语音合成CosyVoice Python SDK](../../raw/model-api-reference/speech-synthesis-api-reference/cosyvoice-large-model-for-speech-synthesis/cosyvoice-python-sdk.md)
- [实时语音合成CosyVoice Java SDK](../../raw/model-api-reference/speech-synthesis-api-reference/cosyvoice-large-model-for-speech-synthesis/cosyvoice-java-sdk.md)
- [语音合成CosyVoice Android SDK](../../raw/model-api-reference/speech-synthesis-api-reference/cosyvoice-large-model-for-speech-synthesis/cosyvoice-android-sdk.md)
- [语音合成CosyVoice iOS SDK](../../raw/model-api-reference/speech-synthesis-api-reference/cosyvoice-large-model-for-speech-synthesis/cosyvoice-ios-sdk.md)
- [客户端事件](../../raw/model-api-reference/speech-synthesis-api-reference/qwen-tts-realtime-api-reference/qwen-tts-realtime-client-events.md)
- [Qwen-TTS-Realtime WebSocket API 参考](../../raw/model-api-reference/speech-synthesis-api-reference/qwen-tts-realtime-api-reference/interactive-process-of-qwen-tts-realtime-synthesis.md)
- [CosyVoice音色列表](../../raw/model-api-reference/speech-synthesis-api-reference/cosyvoice-large-model-for-speech-synthesis/cosyvoice-voice-list.md)
- [服务端事件](../../raw/model-api-reference/speech-synthesis-api-reference/qwen-tts-realtime-api-reference/qwen-tts-realtime-server-events.md)
- [Python SDK](../../raw/model-api-reference/speech-synthesis-api-reference/qwen-tts-realtime-api-reference/qwen-tts-realtime-python-sdk.md)
- [Sambert WebSocket API 参考](../../raw/model-api-reference/speech-synthesis-api-reference/sambert-speech-synthesis/sambert-websocket-api.md)
- [Java SDK](../../raw/model-api-reference/speech-synthesis-api-reference/qwen-tts-realtime-api-reference/qwen-tts-realtime-java-sdk.md)
- [Sambert客户端事件](../../raw/model-api-reference/speech-synthesis-api-reference/sambert-speech-synthesis/sambert-client-events.md)
- [Sambert服务端事件](../../raw/model-api-reference/speech-synthesis-api-reference/sambert-speech-synthesis/sambert-server-events.md)
- [语音合成Sambert Java SDK](../../raw/model-api-reference/speech-synthesis-api-reference/sambert-speech-synthesis/sambert-java-sdk.md)
- [语音合成Sambert Python SDK](../../raw/model-api-reference/speech-synthesis-api-reference/sambert-speech-synthesis/sambert-python-sdk.md)
- [语音合成Sambert Android SDK](../../raw/model-api-reference/speech-synthesis-api-reference/sambert-speech-synthesis/sambert-android-sdk.md)
- [语音合成Sambert iOS SDK](../../raw/model-api-reference/speech-synthesis-api-reference/sambert-speech-synthesis/sambert-ios-sdk.md)
- [非实时语音合成CosyVoice HTTP API参考](../../raw/model-api-reference/speech-synthesis-api-reference/non-realtime-cosyvoice-api/cosyvoice-tts-http-api.md)
- [非实时语音合成CosyVoice Java SDK参考](../../raw/model-api-reference/speech-synthesis-api-reference/non-realtime-cosyvoice-api/cosyvoice-tts-java-sdk.md)
- [非实时语音合成CosyVoice Python SDK参考](../../raw/model-api-reference/speech-synthesis-api-reference/non-realtime-cosyvoice-api/cosyvoice-tts-python-sdk.md)
- [MiniMax同步语音合成API参考](../../raw/model-api-reference/speech-synthesis-api-reference/minimax-speech-synthesis/minimax-synchronous-speech-synthesis-api.md)
- [声音复刻HTTP API参考](../../raw/model-api-reference/speech-synthesis-api-reference/sound-reengraving/voice-clone-design-http-api.md)
- [声音复刻Java SDK参考](../../raw/model-api-reference/speech-synthesis-api-reference/sound-reengraving/voice-clone-design-java-sdk.md)
- [声音复刻Python SDK参考](../../raw/model-api-reference/speech-synthesis-api-reference/sound-reengraving/voice-clone-design-python-sdk.md)

