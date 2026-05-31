# 语音合成、语音识别与语音翻译对比

## 概述

百炼平台提供三大类语音 API 服务：**语音合成（TTS）**、**语音识别（ASR）** 和 **语音翻译（Live Translate）**。三者在输入输出格式、支持模型、通信协议和应用场景等方面各有侧重。本文从开发者技术选型的角度，对这三类服务进行系统对比，帮助开发者根据业务需求快速确定最合适的 API 方案。

---

## 核心维度对比

| 维度 | 语音合成（TTS） | 语音识别（ASR） | 语音翻译（Live Translate） |
|------|----------------|----------------|--------------------------|
| **核心功能** | 文本 → 语音 | 语音 → 文本 | 语音 → 翻译文本 / 翻译语音 |
| **输入格式** | 文本（支持流式文本输入） | 音频流或音频文件（pcm/wav/mp3/opus/speex/aac/amr） | 音频流、音频文件或视频文件 |
| **输出格式** | 音频流（pcm/wav/mp3/opus 等） | 文本（含时间戳、标点等结构化信息） | 翻译文本 + 可选翻译音频（wav/pcm） |
| **支持模型系列** | Qwen-TTS、CosyVoice、Sambert、MiniMax | Qwen-ASR、Fun-ASR、Paraformer | qwen3-livetranslate-flash、qwen3.5-livetranslate-flash-realtime |
| **通信协议** | HTTP、WebSocket | HTTP（OpenAI 兼容）、WebSocket、DashScope 异步 | HTTP（OpenAI 兼容）、WebSocket |
| **实时模式** | ✅ WebSocket 双向流式 | ✅ WebSocket 双向流式 | ✅ WebSocket 双向流式 |
| **离线/非实时模式** | ✅ HTTP 同步调用 | ✅ HTTP 同步/异步调用 | ✅ OpenAI 兼容流式调用 |
| **多语种支持** | 中文、英文等（依音色而定） | 20+ 语种（Qwen-ASR 最丰富） | 多语种互译（需指定源语言和目标语言） |
| **热词/定制能力** | 声音复刻、声音设计 | 定制热词（vocabulary_id） | 热词映射表（corpus.phrases）、声音复刻 |
| **移动端 SDK** | CosyVoice / Sambert 支持 Android/iOS | Fun-ASR / Paraformer 支持 Android/iOS | 暂无移动端 SDK |

---

## 服务端点对比

| 接口类型 | 中国内地（北京） | 国际（新加坡） |
|---------|-----------------|---------------|
| **HTTP（通用）** | `https://dashscope.aliyuncs.com/api/v1` | `https://dashscope-intl.aliyuncs.com/api/v1` |
| **OpenAI 兼容** | `https://dashscope.aliyuncs.com/compatible-mode/v1` | `https://dashscope-intl.aliyuncs.com/compatible-mode/v1` |
| **WebSocket（ASR/TTS 传统）** | `wss://dashscope.aliyuncs.com/api-ws/v1/inference` | `wss://dashscope-intl.aliyuncs.com/api-ws/v1/inference` |
| **WebSocket（Realtime）** | `wss://dashscope.aliyuncs.com/api-ws/v1/realtime` | `wss://dashscope-intl.aliyuncs.com/api-ws/v1/realtime` |

> **说明**：三类服务共享统一的鉴权方式（`Authorization: Bearer <api_key>`），WebSocket 在握手阶段验证。不同地域需使用对应地域的 API Key。

---

## 模型与计费对比

| 服务 | 代表模型 | 计费单位 | 参考价格 |
|------|---------|---------|---------|
| 语音合成 | MiniMax/speech-2.8-hd | 每万字符 | 3.5 元 |
| 语音合成 | MiniMax/speech-2.8-turbo | 每万字符 | 2 元 |
| 语音合成 | CosyVoice / Qwen-TTS | 按字符数 | 详见计费文档 |
| 语音识别 | Qwen-ASR / Fun-ASR / Paraformer | 按音频时长 | 详见计费文档 |
| 语音翻译 | qwen3-livetranslate-flash | 按 Token 消耗 | 详见计费文档 |

---

## SDK 支持对比

| SDK | 语音合成 | 语音识别 | 语音翻译 |
|-----|---------|---------|---------|
| Python SDK | ✅ 全系列 | ✅ 全系列 | ✅ |
| Java SDK | ✅ 全系列 | ✅ 全系列 | ✅ |
| Android SDK | ✅（CosyVoice、Sambert） | ✅（Fun-ASR、Paraformer） | ❌ |
| iOS SDK | ✅（CosyVoice、Sambert） | ✅（Fun-ASR、Paraformer） | ❌ |
| HTTP/REST | ✅ | ✅ | ✅（OpenAI 兼容） |
| WebSocket 原生 | ✅ | ✅ | ✅ |

---

## 实时交互协议对比

三类服务的实时（WebSocket）交互模式均采用事件驱动，但协议细节有所不同：

| 特性 | 语音合成（TTS） | 语音识别（ASR） | 语音翻译 |
|------|----------------|----------------|---------|
| **会话初始化** | `run-task` 事件（CosyVoice）/ `session.update`（Qwen-TTS Realtime） | `run-task`（Fun-ASR/Paraformer）/ `session.update`（Qwen-ASR Realtime） | `session.update` |
| **数据发送** | `continue-task` 发送文本片段 | Binary 帧发送音频流 | `input_audio_buffer.append`（Base64 音频） |
| **数据接收** | Binary 帧接收音频流 | 文本事件接收识别结果 | `response.text.text`（文本）+ `response.audio.delta`（音频） |
| **会话结束** | `finish-task` / 客户端关闭 | `finish-task` / 客户端关闭 | `session.finish` |
| **连接复用** | ✅ 支持 | ✅ 支持 | ✅ 支持 |

---

## 适用场景建议

### 语音合成（TTS）

- **智能客服/语音助手**：将 AI 回复转为自然语音输出，推荐使用 CosyVoice 或 Qwen-TTS 实时模式以降低首帧延迟
- **有声内容生产**：将文章、小说等长文本批量转换为音频，适合非实时模式
- **个性化播报**：通过声音复刻或声音设计打造品牌专属音色
- **移动端集成**：推荐 CosyVoice（支持 Android/iOS SDK）

### 语音识别（ASR）

- **实时字幕/会议纪要**：使用实时识别 + 语义断句，推荐 Qwen-ASR Realtime 或 Paraformer-realtime-v2
- **电话/呼叫中心录音转写**：8kHz 采样率场景，推荐 Fun-ASR-flash-8k 或 Paraformer-realtime-8k-v2
- **多语种场景**：推荐 Qwen-ASR（支持 20+ 语种）
- **批量录音文件处理**：使用异步文件识别接口（qwen3-asr-flash-filetrans），单次最多 100 个文件
- **特定行业术语识别**：配合定制热词功能提升专业词汇识别准确率

### 语音翻译（Live Translate）

- **实时同声传译**：多人会议实时翻译，推荐 qwen3.5-livetranslate-flash-realtime + 声音复刻（`always` 模式）
- **视频/音频本地化**：将已有音视频内容翻译为目标语言，使用离线翻译接口
- **跨语言客服**：客户说一种语言、客服听另一种语言的实时双向翻译
- **保留说话人音色**：启用声音复刻功能，让翻译后的语音保持原始说话人的声音特征

---

## 技术选型决策指引

```
需要将文本转为语音？
  └─ 是 → 语音合成（TTS）
       ├─ 需要极低延迟？→ Qwen-TTS Realtime / CosyVoice WebSocket
       ├─ 需要移动端？→ CosyVoice（Android/iOS SDK）
       └─ 需要第三方音色？→ MiniMax 系列

需要将语音转为文本？
  └─ 是 → 语音识别（ASR）
       ├─ 实时场景？→ Qwen-ASR Realtime / Paraformer Realtime
       ├─ 多语种？→ Qwen-ASR（20+ 语种）
       └─ 批量文件？→ qwen3-asr-flash-filetrans（异步）

需要语音跨语言转换？
  └─ 是 → 语音翻译（Live Translate）
       ├─ 实时流式？→ qwen3.5-livetranslate-flash-realtime
       ├─ 离线文件？→ qwen3-livetranslate-flash
       └─ 需要保留音色？→ 启用声音复刻
```

---

## 被对比主题页

- [speech synthesis api reference](../api/speech-synthesis-api-reference.md)
- [speech recognition api reference](../api/speech-recognition-api-reference.md)
- [speech translation api reference](../api/speech-translation-api-reference.md)

