# 流式输出

流式输出（Streaming）是指服务端在生成完整响应之前，将结果以增量片段的形式逐步返回给客户端的数据传输模式。相比非流式调用需要等待全部内容生成完毕后一次性返回，流式输出可显著降低首字（首包）延迟，提升用户感知的响应速度。

## 适用场景

在百炼平台中，流式输出广泛应用于以下场景：

| 场景 | 说明 |
|------|------|
| **文本生成（LLM 对话）** | 模型逐 token 输出文本，实现"打字机"效果 |
| **应用调用（智能体/工作流）** | 智能体和工作流应用的对话响应逐步返回 |
| **语音合成（TTS）** | 实时合成场景下音频数据分片返回，边合成边播放 |
| **音乐生成** | 音频片段以 Base64 编码逐段返回，无需等待完整歌曲生成 |
| **实时多模态交互** | WebSocket 双向通信本身即为流式架构，音频/文本持续推送 |

## 启用方式

不同 API 体系的流式输出启用方式有所差异：

### 文本生成与应用调用

| API 体系 | 启用方式 | 传输协议 |
|----------|----------|----------|
| DashScope SDK（Python） | `stream=True` | SSE |
| DashScope SDK（Java） | `streamCall()` 方法 | SSE |
| DashScope HTTP | 请求头 `X-DashScope-SSE: enable` | SSE |
| [[openai-compatible-api|OpenAI 兼容接口]] | `stream=true` | SSE |

**DashScope SDK 示例（Python）：**

```python
import os
from dashscope import Application

responses = Application.call(
    api_key=os.getenv("DASHSCOPE_API_KEY"),
    app_id='YOUR_APP_ID',
    [[prompt|prompt]]='请介绍量子计算',
    stream=True  # 启用流式输出
)
for response in responses:
    print(response.output.text, end='', flush=True)
```

**HTTP 示例（curl）：**

```bash
curl -X POST 'https://dashscope.aliyuncs.com/api/v1/apps/{APP_ID}/completion' \
  -H "Authorization: Bearer $DASHSCOPE_API_KEY" \
  -H "Content-Type: application/json" \
  -H "X-DashScope-SSE: enable" \
  -d '{"input": {"[[prompt|prompt]]": "请介绍量子计算"}}'
```

### 音乐生成

通过 HTTP 请求头启用 SSE 流式输出：

```
X-DashScope-SSE: enable
```

流式响应中，中间消息的 `output.audio.data` 包含 Base64 编码的音频片段（`finish_reason` 为 `null`），最终消息包含完整音频 URL 和元信息（`finish_reason` 为 `stop`）。

### 语音合成（实时）

实时 TTS 通过 WebSocket 协议天然支持流式输出，音频数据以分片形式持续推送：

- **CosyVoice 实时**：`run-task` → `continue-task` → `finish-task` 交互流程
- **Qwen-TTS 实时**：`session.update` → `input_text_buffer.append` → `input_text_buffer.commit` 交互流程
- **Sambert**：仅支持单向流式输出（`streaming: "out"`）

### 实时多模态交互

[[omni-realtime-api]] 基于 WebSocket 双向流式通信，音频和文本响应通过服务端事件持续推送，无需额外配置流式参数。

## 关键参数与配置

| 参数/配置 | 适用范围 | 说明 |
|-----------|----------|------|
| `stream` | SDK 调用 | 设为 `True`/`true` 启用流式 |
| `X-DashScope-SSE` | HTTP 调用 | 设为 `enable` 启用 SSE 流式 |
| `incremental_output` | 文本生成（部分接口） | 设为 `true` 时每次仅返回增量内容，否则返回累积内容 |
| `streaming` | 语音合成 WebSocket | 如 `"out"` 表示仅输出流式 |

## 注意事项

- **工作流应用的额外配置**：工作流应用使用流式输出时，需在百炼控制台的**结束节点**或**流程输出节点**中启用「流式输出」开关，并**重新发布**应用后才能生效。
- **SSE 数据格式**：HTTP 流式响应遵循 Server-Sent Events 规范，每条消息以 `data:` 前缀发送，消息之间以空行分隔。客户端需按 SSE 协议逐条解析。
- **错误处理**：流式传输过程中若发生错误，错误信息将以最后一条 SSE 事件的形式返回，客户端应监听错误事件并妥善处理。
- **非流式与流式字符限制可能不同**：如 [[music-generation-references]] 中，流式模式下的歌词字符要求（中文 300~350 字）比非流式模式（中文 5~350 字符）更严格。

## 相关主题

- [[qwen-api-reference]] — 文本生成模型 API 接口总览
- [[application-call]] — 应用调用 API 参考
- [[bailian-application-calling]] — 应用调用实践指南
- [[speech-synthesis-api-reference]] — 语音合成 API 参考
- [[music-generation-references]] — 音乐生成 API 参考
- [[omni-realtime-api]] — 实时多模态交互 API 参考

## 关联主题页

- [[qwen-api-reference|qwen api reference]] — `../api/qwen-api-reference.md`
- [[application-call|application call]] — `../api/application-call.md`
- [[bailian-application-calling|bailian [[application-call|application call]]ing]] — `../guides/bailian-application-calling.md`
- [[omni-realtime-api|omni realtime api]] — `../api/omni-realtime-api.md`
- [[speech-synthesis-api-reference|speech synthesis api reference]] — `../api/speech-synthesis-api-reference.md`
- [[music-generation-references|music generation references]] — `../api/music-generation-references.md`

