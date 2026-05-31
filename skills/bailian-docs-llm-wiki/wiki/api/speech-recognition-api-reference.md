# speech recognition api reference

百炼平台提供多种语音识别（ASR）模型和 API，覆盖实时语音识别和录音文件识别两大场景。开发者可根据业务需求选择 Qwen-ASR、Fun-ASR 或 Paraformer 系列模型，并通过 OpenAI 兼容协议、DashScope SDK 或 WebSocket 等方式接入。本页面汇总各模型的接入方式、关键参数和使用限制。

## 模型与功能总览

平台语音识别能力按场景分为**实时语音识别**和**录音文件识别**两大类，涉及三个模型系列：

### 实时语音识别

| 模型系列 | 推荐模型 | 采样率 | 主要语种 | 接入协议 |
|---------|---------|--------|---------|---------|
| **Qwen-ASR Realtime** | qwen3-asr-flash-realtime | 16000/8000 Hz | 中文、英文、日语、韩语、德语、法语、俄语等 30+ 语种 | WebSocket (Realtime API) |
| **Fun-ASR** | fun-asr-realtime | 16000 Hz（8k 模型为 8000 Hz） | 中文（普通话及多种方言）、英文、日语 | WebSocket / DashScope SDK |
| **Paraformer** | paraformer-realtime-v2 | 任意（v2）/ 16000 Hz（v1） | 中文（含方言）、英文、日语、韩语、德语、法语、俄语 | WebSocket / DashScope SDK |

### 录音文件识别

| 模型系列 | 推荐模型 | 接入方式 | 关键特性 |
|---------|---------|---------|---------|
| **Qwen-ASR** | qwen3-asr-flash / qwen3-asr-flash-filetrans | OpenAI 兼容、DashScope 同步/异步 | 支持音频 URL 和 Base64 输入 |
| **Fun-ASR** | fun-asr | DashScope 异步调用 / RESTful API | 支持说话人分离、批量处理（最多100个URL） |
| **Paraformer** | paraformer-v2 | DashScope 异步调用 / RESTful API | 支持说话人分离、语气词过滤、时间戳校准 |

> **注意**：Qwen-ASR 的 `qwen3-asr-flash-filetrans` 仅支持 DashScope 异步调用，而 `qwen3-asr-flash` 支持 OpenAI 兼容和 DashScope 同步调用两种方式。详见 [录音文件识别（Qwen-ASR）API参考](../../raw/model-api-reference/speech-recognition-api-reference/qwen-asr-api-reference.md)。

## 接入方式与协议

### OpenAI 兼容协议

仅 Qwen-ASR 的 `qwen3-asr-flash` 模型支持，使用标准的 `chat/completions` 端点：

- **中国内地**：`https://dashscope.aliyuncs.com/compatible-mode/v1`
- **国际**：`https://dashscope-intl.aliyuncs.com/compatible-mode/v1`

### WebSocket 协议

实时语音识别的主要接入方式。不同模型系列使用不同的 WebSocket 端点：

| 模型系列 | 中国内地端点 | 协议特点 |
|---------|------------|---------|
| Qwen-ASR Realtime | `wss://dashscope.aliyuncs.com/api-ws/v1/realtime?model=<model_name>` | 通过 URL 查询参数指定模型；支持 VAD 和 Manual 两种模式 |
| Fun-ASR / Paraformer | `wss://dashscope.aliyuncs.com/api-ws/v1/inference` | 通过 `run-task` 消息体中的 `model` 字段指定模型 |

所有 WebSocket 连接均需在请求头中设置 `Authorization: Bearer <your_api_key>` 进行鉴权。详见 [Qwen-ASR实时语音识别WebSocket API](../../raw/model-api-reference/speech-recognition-api-reference/qwen-asr-realtime-api/qwen-asr-realtime-interaction-process.md)。

### DashScope SDK

提供 Python 和 Java SDK，封装了 WebSocket 交互细节：

- **实时识别**：通过 `Recognition` 类（Fun-ASR/Paraformer）或 `OmniRealtimeConversation` 类（Qwen-ASR）调用
- **录音文件识别**：通过 `Transcription` 类调用，支持异步提交 + 同步等待或异步查询两种模式

### 移动端 SDK

Paraformer 和 Fun-ASR 均提供 Android SDK（AAR 格式）和 iOS SDK（nuisdk.framework），通过 `initialize` → `startDialog` → `stopDialog` → `release` 的生命周期管理识别流程。

## 关键参数

### 通用鉴权参数

| 参数 | 说明 |
|-----|------|
| `Authorization` | `Bearer <api_key>` 格式，中国内地和国际地域使用不同的 API Key |
| `X-DashScope-WorkSpace` | 可选，[[workspace]] 业务空间 ID |
| `X-DashScope-DataInspection` | 可选，数据合规检测开关 |

### 实时识别参数

**Paraformer / Fun-ASR（WebSocket `run-task` 消息）：**

| 参数 | 类型 | 必选 | 说明 |
|-----|------|-----|------|
| `format` | string | 是 | 音频格式：pcm、wav、mp3、opus、speex、aac、amr |
| `sample_rate` | integer | 是 | 采样率（Hz），因模型而异 |
| `vocabulary_id` | string | 否 | [[custom-hot-words]] 热词列表 ID |
| `language_hints` | array | 否 | 指定待识别语种 |
| `semantic_punctuation_enabled` | boolean | 否 | 语义断句开关（默认 false） |
| `disfluency_removal_enabled` | boolean | 否 | 语气词过滤（仅 Paraformer） |

**Qwen-ASR Realtime（`session.update` 事件）：**

| 参数 | 说明 |
|-----|------|
| `input_audio_format` | 支持 pcm 和 opus |
| `sample_rate` | 支持 16000 和 8000 |
| `turn_detection` | VAD 配置，设为 null 切换到 Manual 模式 |
| `turn_detection.threshold` | VAD 检测阈值，推荐 0.0 |
| `turn_detection.silence_duration_ms` | 静音断句阈值（ms），推荐 400 |

### 录音文件识别参数

| 参数 | 说明 |
|-----|------|
| `file_urls` | 公网可访问的音频文件 URL（不支持本地文件直传），单次最多 100 个 |
| `channel_id` | 音轨索引 |
| `diarization_enabled` | 说话人分离开关 |
| `speaker_count` | 说话人数量参考值 |
| `language_hints` | 待识别语种 |

## 交互流程

### 实时识别（Paraformer / Fun-ASR）

采用标准 WebSocket 双工通信：

1. 建立 WebSocket 连接
2. 发送 `run-task` 指令 → 收到 `task-started` 事件
3. 持续发送二进制音频流 → 实时接收 `result-generated` 事件
4. 发送 `finish-task` 指令 → 收到 `task-finished` 事件
5. 关闭连接

详见 [Paraformer实时语音识别WebSocket API](../../raw/model-api-reference/speech-recognition-api-reference/paraformer-real-time-speech-recognition-api-reference/websocket-for-paraformer-real-time-service.md)。

### 实时识别（Qwen-ASR Realtime）

支持两种交互模式：

- **VAD 模式**（默认）：服务端自动检测语音起止点，返回 `speech_started` / `speech_stopped` 事件
- **Manual 模式**：客户端通过 `input_audio_buffer.commit` 主动触发断句

### 录音文件识别

采用"提交-轮询"异步模式：

1. 调用提交任务接口，获取 `task_id`
2. 轮询查询任务接口，直到状态为 `SUCCEEDED` 或 `FAILED`
3. 获取识别结果（结果 URL 有效期 24 小时）

## 定制热词

通过 [[custom-hot-words]] 功能可提升特定词汇的识别准确率。支持 HTTP API、Python SDK 和 Java SDK 三种管理方式，提供创建、查询、更新、删除热词列表的完整 CRUD 操作。

关键约束：
- 热词权重范围 [1, 5]，常用值为 4
- `target_model` 必须与实际调用语音识别时使用的模型一致
- 新加坡地域的子业务空间暂不支持热词功能

## 限制和注意事项

- **音频格式**：实时识别支持 pcm、wav、mp3、opus、speex、aac、amr；录音文件识别额外支持 flac、ogg、m4a 及多种视频格式
- **音频要求**：实时识别须为**单声道**音频；录音文件不超过 2GB、时长不超过 12 小时
- **文件输入限制**：录音文件识别不支持本地文件直传和 Base64 格式（Fun-ASR/Paraformer），需提供公网可访问的 URL

> **注意**：使用 SDK 时，若录音文件存储在阿里云 OSS，不支持 `oss://` 前缀的临时 URL；使用 RESTful API 时则支持，但临时 URL 有效期仅 48 小时，不建议用于生产环境。

- **视频文件处理**：建议使用 ffmpeg 预处理提取音轨并压缩，以提升转写效率：
  ```shell
  ffmpeg -i input-video -ac 1 -ar 16000 -acodec libopus output.opus
  ```
- **Paraformer 音频编码约束**：opus/speex 必须使用 Ogg 封装；wav 必须为 PCM 编码；amr 仅支持 AMR-NB
- **SDK 版本要求**：Qwen-ASR Realtime 需要 DashScope Python SDK ≥ 1.25.6、Java SDK ≥ 2.22.5
- **安全建议**：API Key 应配置到环境变量，移动端场景建议使用 [[temporary-api-key]]（有效期 60 秒）

## 来源文档

- [录音文件识别（Qwen-ASR）API参考](../../raw/model-api-reference/speech-recognition-api-reference/qwen-asr-api-reference.md)
- [Paraformer实时语音识别WebSocket API](../../raw/model-api-reference/speech-recognition-api-reference/paraformer-real-time-speech-recognition-api-reference/websocket-for-paraformer-real-time-service.md)
- [实时语音识别（Paraformer）客户端事件](../../raw/model-api-reference/speech-recognition-api-reference/paraformer-real-time-speech-recognition-api-reference/paraformer-client-events.md)
- [实时语音识别（Paraformer）服务端事件](../../raw/model-api-reference/speech-recognition-api-reference/paraformer-real-time-speech-recognition-api-reference/paraformer-server-events.md)
- [Paraformer实时语音识别Java SDK](../../raw/model-api-reference/speech-recognition-api-reference/paraformer-real-time-speech-recognition-api-reference/paraformer-real-time-speech-recognition-java-sdk.md)
- [Paraformer实时语音识别Python SDK](../../raw/model-api-reference/speech-recognition-api-reference/paraformer-real-time-speech-recognition-api-reference/paraformer-real-time-speech-recognition-python-sdk.md)
- [Paraformer实时语音识别Android SDK](../../raw/model-api-reference/speech-recognition-api-reference/paraformer-real-time-speech-recognition-api-reference/android-sdk-for-paraformer-real-time-service.md)
- [Paraformer实时语音识别iOS SDK](../../raw/model-api-reference/speech-recognition-api-reference/paraformer-real-time-speech-recognition-api-reference/ios-sdk-for-paraformer-real-time-service.md)
- [Fun-ASR实时语音识别WebSocket API](../../raw/model-api-reference/speech-recognition-api-reference/fun-asr-real-time-speech-recognition-api-reference/fun-asr-realtime-websocket-api.md)
- [实时语音识别（Fun-ASR）客户端事件](../../raw/model-api-reference/speech-recognition-api-reference/fun-asr-real-time-speech-recognition-api-reference/fun-asr-client-events.md)
- [实时语音识别（Fun-ASR）服务端事件](../../raw/model-api-reference/speech-recognition-api-reference/fun-asr-real-time-speech-recognition-api-reference/fun-asr-server-events.md)
- [Python SDK](../../raw/model-api-reference/speech-recognition-api-reference/fun-asr-real-time-speech-recognition-api-reference/fun-asr-realtime-python-sdk.md)
- [Java SDK](../../raw/model-api-reference/speech-recognition-api-reference/fun-asr-real-time-speech-recognition-api-reference/fun-asr-realtime-java-sdk.md)
- [Fun-ASR实时语音识别Android SDK](../../raw/model-api-reference/speech-recognition-api-reference/fun-asr-real-time-speech-recognition-api-reference/android-sdk-for-fun-asr-real-time-service.md)
- [Fun-ASR实时语音识别iOS SDK](../../raw/model-api-reference/speech-recognition-api-reference/fun-asr-real-time-speech-recognition-api-reference/ios-sdk-for-fun-asr-real-time-service.md)
- [Qwen-ASR实时语音识别WebSocket API](../../raw/model-api-reference/speech-recognition-api-reference/qwen-asr-realtime-api/qwen-asr-realtime-interaction-process.md)
- [实时语音识别（Qwen-ASR-Realtime）客户端事件](../../raw/model-api-reference/speech-recognition-api-reference/qwen-asr-realtime-api/qwen-asr-realtime-client-events.md)
- [实时语音识别（Qwen-ASR-Realtime）服务端事件](../../raw/model-api-reference/speech-recognition-api-reference/qwen-asr-realtime-api/qwen-asr-realtime-server-events.md)
- [实时语音识别（Qwen-ASR-Realtime）Python SDK-API参考](../../raw/model-api-reference/speech-recognition-api-reference/qwen-asr-realtime-api/qwen-asr-realtime-python-sdk.md)
- [实时语音识别（Qwen-ASR-Realtime）Java SDK-API参考](../../raw/model-api-reference/speech-recognition-api-reference/qwen-asr-realtime-api/qwen-asr-realtime-java-sdk.md)
- [Paraformer录音文件识别Java SDK](../../raw/model-api-reference/speech-recognition-api-reference/paraformer-recorded-speech-recognition-api-reference/paraformer-recorded-speech-recognition-java-sdk.md)
- [Paraformer录音文件识别Python SDK](../../raw/model-api-reference/speech-recognition-api-reference/paraformer-recorded-speech-recognition-api-reference/paraformer-recorded-speech-recognition-python-sdk.md)
- [Paraformer录音文件识别RESTful API](../../raw/model-api-reference/speech-recognition-api-reference/paraformer-recorded-speech-recognition-api-reference/paraformer-recorded-speech-recognition-restful-api.md)
- [Paraformer录音文件识别Android SDK](../../raw/model-api-reference/speech-recognition-api-reference/paraformer-recorded-speech-recognition-api-reference/paraformer-recorded-speech-recognition-android-sdk.md)
- [Paraformer录音文件识别iOS SDK](../../raw/model-api-reference/speech-recognition-api-reference/paraformer-recorded-speech-recognition-api-reference/paraformer-recorded-speech-recognition-ios-sdk.md)
- [最佳实践](../../raw/model-api-reference/speech-recognition-api-reference/paraformer-recorded-speech-recognition-api-reference/paraformer-best-practices.md)
- [Fun-ASR录音文件识别Python SDK](../../raw/model-api-reference/speech-recognition-api-reference/fun-asr-recorded-speech-recognition-api-reference/funauidio-asr-recorded-speech-recognition-python-sdk.md)
- [Fun-ASR录音文件识别Java SDK](../../raw/model-api-reference/speech-recognition-api-reference/fun-asr-recorded-speech-recognition-api-reference/fun-asr-recorded-speech-recognition-java-sdk.md)
- [Fun-ASR录音文件识别HTTP API参考](../../raw/model-api-reference/speech-recognition-api-reference/fun-asr-recorded-speech-recognition-api-reference/fun-asr-recorded-speech-recognition-http-api.md)
- [Fun-ASR录音文件识别Android SDK](../../raw/model-api-reference/speech-recognition-api-reference/fun-asr-recorded-speech-recognition-api-reference/fun-asr-recorded-speech-recognition-android-sdk.md)
- [Fun-ASR录音文件识别iOS SDK](../../raw/model-api-reference/speech-recognition-api-reference/fun-asr-recorded-speech-recognition-api-reference/fun-asr-recorded-speech-recognition-ios-sdk.md)
- [定制热词HTTP API参考](../../raw/model-api-reference/speech-recognition-api-reference/custom-hot-words/vocabulary-http-api.md)
- [定制热词Java SDK参考](../../raw/model-api-reference/speech-recognition-api-reference/custom-hot-words/vocabulary-java-sdk.md)
- [定制热词Python SDK参考](../../raw/model-api-reference/speech-recognition-api-reference/custom-hot-words/vocabulary-python-sdk.md)

