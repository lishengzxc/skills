# 流式输出

流式输出（Streaming Output）是指模型在生成过程中将结果以增量方式逐步返回给客户端的通信模式，而非等待完整结果生成后一次性返回。这种方式显著降低了用户感知到的首次响应延迟（TTFT），适用于实时对话、语音合成、语音识别等对响应速度有要求的场景。

## 在百炼平台中的使用场景

### 文本生成模型

通过 [OpenAI 兼容接口](openai-compatible-api.md)、Anthropic 兼容接口或 DashScope 原生接口调用 Qwen 系列文本生成模型时，均可开启流式输出。模型会以增量 token 的形式逐步返回生成内容，客户端可边接收边展示。

### 应用调用（智能体 / 工作流）

调用百炼平台的智能体应用和工作流应用时，可通过设置 `stream=True` 启用流式输出。对于**工作流应用**，还需在控制台中对结束节点或流程输出节点启用**流式输出开关**并重新发布应用，否则流式参数不会生效。

DashScope API 和 OpenAI 兼容 Responses API 均支持该参数。

### 实时多模态交互（Omni Realtime API）

基于 WebSocket 协议的 Omni Realtime API 天然采用双向流式通信。服务端通过增量事件（如 `response.audio.delta`、`response.text.delta`、`response.audio_transcript.delta`）将音频、文本结果逐帧推送至客户端，实现低延迟的实时对话体验。

### 语音合成（TTS）

实时语音合成模型（如 `qwen3-tts-flash-realtime`、CosyVoice 系列）通过 WebSocket 实现流式合成——客户端可以流式发送待合成文本，服务端同步返回增量音频数据，无需等待全部文本输入完毕。

### 语音识别（ASR）

实时语音识别模型（如 `qwen3-asr-flash-realtime`、`paraformer-realtime-v2`、`fun-asr-realtime`）通过 WebSocket 接收流式音频输入，并实时返回识别结果（包含中间结果和最终结果），适用于边说边转写的场景。

## 关键参数和配置

| 参数 / 配置 | 适用场景 | 说明 |
|------------|---------|------|
| `stream` | 文本生成、应用调用 | 布尔值，设为 `true` 开启流式输出，默认 `false` |
| 流式输出开关（控制台） | 工作流应用 | 需在结束节点/流程输出节点中手动启用并重新发布 |
| WebSocket 连接 | Omni Realtime、实时 TTS、实时 ASR | 建立持久连接后通过事件驱动收发增量数据 |
| `incremental_output`（DashScope） | 文本生成 | 控制增量输出行为，`true` 时每次仅返回新增部分 |

## 典型代码示例

### 应用调用流式输出（Python SDK）

```python
import os
from dashscope import Application

responses = Application.call(
    api_key=os.getenv("DASHSCOPE_API_KEY"),
    app_id='YOUR_APP_ID',
    prompt='请介绍量子计算',
    stream=True
)

for response in responses:
    print(response.output.text, end='', flush=True)
```

### [OpenAI 兼容接口](openai-compatible-api.md)流式输出

```python
from openai import OpenAI

client = OpenAI(
    api_key=os.getenv("DASHSCOPE_API_KEY"),
    base_url="https://dashscope.aliyuncs.com/compatible-mode/v1"
)

stream = client.chat.completions.create(
    model="qwen-plus",
    messages=[{"role": "user", "content": "你好"}],
    stream=True
)

for chunk in stream:
    if chunk.choices[0].delta.content:
        print(chunk.choices[0].delta.content, end='', flush=True)
```

## 注意事项

- 流式模式下，客户端需逐块拼接内容以获得完整响应；若需完整结果可在流结束后统一处理。
- 工作流应用必须在控制台启用流式输出开关后才能通过 API 获取流式响应，否则行为等同于同步调用。
- WebSocket 场景下的流式通信是双向的——客户端可流式发送输入（如音频帧），服务端也可流式返回结果。
- 流式输出不影响最终结果的完整性和准确性，仅改变结果的交付方式。

## 关联主题页

- [omni realtime api](../api/omni-realtime-api.md)
- [qwen api reference](../api/qwen-api-reference.md)
- [speech synthesis api reference](../api/speech-synthesis-api-reference.md)
- [speech recognition api reference](../api/speech-recognition-api-reference.md)
- [application call](../api/application-call.md)
- [bailian application calling](../guides/bailian-application-calling.md)

