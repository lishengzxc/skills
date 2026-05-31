# 语音合成、语音识别与语音翻译对比

百炼平台围绕语音处理提供了三大核心能力：**语音合成（TTS）**、**语音识别（ASR）** 和 **语音翻译（Live Translate）**。三者在输入输出形式、模型选择、接入协议和适用场景上各有差异。本文从开发者技术选型的角度，对三类能力进行系统性对比，帮助您快速定位适合自身业务的 API 方案。

> 各能力的完整 API 参考请分别查阅：[[speech-synthesis-api-reference]]、[[speech-recognition-api-reference]]、[[speech-translation-api-reference]]。

---

## 核心维度对比

### 基本能力对比

| 维度 | 语音合成（TTS） | 语音识别（ASR） | 语音翻译（Live Translate） |
|------|----------------|----------------|--------------------------|
| **核心功能** | 文本 → 语音 | 语音 → 文本 | 语音 → 翻译文本 + 翻译语音 |
| **输入格式** | 文本（纯文本 / SSML） | 音频流或音频文件（pcm、wav、mp3、opus 等） | 音频文件（mp3、wav 等）/ 视频文件 / 实时音频流（pcm、opus） |
| **输出格式** | 音频（pcm / wav / mp3 / opus） | 文本（JSON 结构化结果，含时间戳等） | 文本 + 音频（双模态输出），音频格式为 wav（文件）或 pcm（实时） |
| **处理模式** | 实时流式合成 / 非实时合成 | 实时流式识别 / 录音文件异步识别 | 文件翻译（[[streaming|流式输出]]） / 实时翻译（WebSocket） |
| **多语种支持** | 中文、英文等（因模型而异） | 中文、英文、日语、韩语等 30+ 语种 | 多语种互译，支持自动语种识别 |

### 模型与接入协议对比

| 维度 | 语音合成（TTS） | 语音识别（ASR） | 语音翻译（Live Translate） |
|------|----------------|----------------|--------------------------|
| **主要模型系列** | Qwen-TTS、CosyVoice、Sambert、MiniMax | Qwen-ASR、Fun-ASR、Paraformer | qwen3-livetranslate-flash、qwen3.5-livetranslate-flash-realtime |
| **推荐模型** | qwen3-tts-flash（非实时）/ cosyvoice-v3.5-plus（实时） | qwen3-asr-flash（文件）/ qwen3-asr-flash-realtime（实时） | qwen3-livetranslate-flash（文件）/ qwen3.5-livetranslate-flash-realtime（实时） |
| **HTTP API** | ✅ DashScope 专有端点 | ✅ OpenAI 兼容（仅 Qwen-ASR）/ DashScope 异步 | ✅ OpenAI 兼容（仅文件翻译） |
| **WebSocket API** | ✅ CosyVoice / Qwen-TTS Realtime / Sambert | ✅ Qwen-ASR Realtime / Fun-ASR / Paraformer | ✅ 实时翻译 |
| **OpenAI 兼容协议** | ❌ 不支持 | ✅ 仅 `qwen3-asr-flash` | ✅ 仅文件翻译模型 |
| **DashScope SDK** | ✅ Python / Java / Android / iOS | ✅ Python / Java / Android / iOS | ✅ Python / Java |

### API 端点对比

| 能力 | 场景 | 端点 |
|------|------|------|
| **语音合成** | 非实时（CosyVoice） | `POST https://dashscope.aliyuncs.com/api/v1/services/audio/tts/SpeechSynthesizer` |
| **语音合成** | 非实时（MiniMax） | `POST https://dashscope.aliyuncs.com/api/v1/services/aigc/multimodal-generation/generation` |
| **语音合成** | 实时（CosyVoice） | `wss://dashscope.aliyuncs.com/api-ws/v1/inference` |
| **语音合成** | 实时（Qwen-TTS） | `wss://dashscope.aliyuncs.com/api-ws/v1/realtime?model=<model_name>` |
| **语音识别** | 文件识别（Qwen-ASR） | `https://dashscope.aliyuncs.com/compatible-mode/v1/chat/completions` |
| **语音识别** | 实时识别（Qwen-ASR） | `wss://dashscope.aliyuncs.com/api-ws/v1/realtime?model=<model_name>` |
| **语音识别** | 实时识别（Fun-ASR / Paraformer） | `wss://dashscope.aliyuncs.com/api-ws/v1/inference` |
| **语音翻译** | 文件翻译 | `https://dashscope.aliyuncs.com/compatible-mode/v1/chat/completions` |
| **语音翻译** | 实时翻译 | `wss://dashscope.aliyuncs.com/api-ws/v1/realtime` |

### 关键特性对比

| 特性 | 语音合成（TTS） | 语音识别（ASR） | 语音翻译（Live Translate） |
|------|----------------|----------------|--------------------------|
| **音色选择** | ✅ 系统预置音色 + 声音复刻 + 声音设计 | ❌ 不适用 | ✅ 支持音色配置 + 声音复刻 |
| **热词/自定义词表** | ❌ 不适用 | ✅ 通过 `vocabulary_id` 配置 | ✅ 通过 `translation.corpus.phrases` 配置热词映射 |
| **说话人分离** | ❌ 不适用 | ✅ Fun-ASR / Paraformer 支持 | ❌ 不适用 |
| **SSML 支持** | ✅ CosyVoice 支持 | ❌ 不适用 | ❌ 不适用 |
| **时间戳输出** | ✅ 字级别时间戳（CosyVoice / Sambert） | ✅ 字/句级别时间戳 | ❌ 不适用 |
| **VAD（语音活动检测）** | ❌ 不适用 | ✅ Qwen-ASR Realtime 支持 | ✅ 服务端自动检测语音起止 |
| **视频输入** | ❌ 不支持 | ❌ 不支持 | ✅ 支持视频文件 URL 输入 |
| **图像输入** | ❌ 不支持 | ❌ 不支持 | ✅ 实时翻译支持图像帧输入 |

### 计费方式对比

| 维度 | 语音合成（TTS） | 语音识别（ASR） | 语音翻译（Live Translate） |
|------|----------------|----------------|--------------------------|
| **计量单位** | 按合成字符数计费 | 按音频时长计费 | 按 Token 计费（含 [[prompt|prompt]]_tokens 和 completion_tokens） |
| **Token 明细** | — | — | 区分文本 Token 和音频 Token |
| **免费额度** | 因模型而异，详见各模型定价页 | 因模型而异，详见各模型定价页 | 因模型而异，详见各模型定价页 |

> 具体价格信息请参考百炼平台各模型的定价页面。

---

## 适用场景建议

### 语音合成（TTS） — 适合"文字变声音"场景

推荐在以下场景中使用 [[speech-synthesis-api-reference]]：

- **智能客服 / 语音助手**：将 LLM 生成的文本回复转为语音播放，搭配实时合成（Qwen-TTS Realtime / CosyVoice）实现低延迟交互
- **有声读物 / 内容播报**：批量将文章、新闻转为语音，使用非实时合成 + 高品质音色
- **个性化音色需求**：通过声音复刻（[[voice-clone-design-http-api]]）或声音设计（[[voice-design-api-references]]）创建品牌专属音色
- **IVR / 电话系统**：使用 8000 Hz 采样率的窄带语音输出，适配电话线路

**选型建议**：
- 追求最低延迟 → Qwen-TTS Realtime（WebSocket）
- 需要丰富音色和 SSML 控制 → CosyVoice 系列
- 移动端集成 → CosyVoice / Sambert（提供 Android / iOS SDK）

### 语音识别（ASR） — 适合"声音变文字"场景

推荐在以下场景中使用 [[speech-recognition-api-reference]]：

- **实时字幕 / 会议纪要**：使用实时识别模型，边说边转文字，支持语义断句
- **呼叫中心质检**：对录音文件批量转写，结合说话人分离功能区分坐席和客户
- **语音输入法 / 语音搜索**：实时识别用户语音输入，快速返回文本结果
- **多语种内容处理**：Qwen-ASR 支持 30+ 语种，适合国际化业务

**选型建议**：
- 多语种 + 最新模型能力 → Qwen-ASR 系列
- 中文方言识别 → Fun-ASR（支持多种方言）
- 需要说话人分离 +

## 被对比主题页

- [[speech-synthesis-api-reference|speech synthesis api reference]] — `../api/speech-synthesis-api-reference.md`
- [[speech-recognition-api-reference|speech recognition api reference]] — `../api/speech-recognition-api-reference.md`
- [[speech-translation-api-reference|speech translation api reference]] — `../api/speech-translation-api-reference.md`

