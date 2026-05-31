# music generation references

百炼平台提供音乐生成能力，当前支持 Fun-Music 模型，可根据歌词或提示词自动创作并生成完整歌曲音频。该功能支持流式与非流式两种输出模式，适用于背景音乐生成、AI 作曲等场景。

## 支持的模型

| 模型名称 | 模型 ID | 状态 | 部署地域 |
|---------|---------|------|---------|
| Fun-Music | `fun-music-v1` | 邀测中 | 中国内地（北京） |

> **注意**：根据 [音乐生成Fun-Music API参考](../../raw/model-api-reference/music-generation-references/fun-music-api.md)，该模型目前处于邀测阶段，需在模型广场申请开通后方可使用。

## 服务端点

```
POST https://dashscope.aliyuncs.com/api/v1/services/audio/music/generation
```

通信协议：HTTPS，[[streaming|流式输出]]支持 SSE（Server-Sent Events）。

## 关键参数

### 请求头

| 参数 | 必填 | 说明 |
|------|------|------|
| `Authorization` | 是 | `Bearer {api-key}`，参见 [[get-api-key]] |
| `Content-Type` | 是 | `application/json` |
| `X-DashScope-SSE` | 否 | 设为 `enable` 启用[[streaming|流式输出]] |

### 输入参数（input 对象）

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `lyrics` | string | 条件必选 | 歌词内容，与 `[[prompt|prompt]]` 二选一 |
| `[[prompt|prompt]]` | string | 条件必选 | 提示词，模型据此自动创作歌词并生成歌曲 |
| `gender` | string | 否 | 演唱声音性别：`male` / `female`（默认） |
| `format` | string | 否 | 音频格式：`mp3`（默认）/ `wav` |
| `enable_aigc_watermark` | boolean | 否 | AIGC 水印开关，默认 `false` |

如 [音乐生成Fun-Music API参考](../../raw/model-api-reference/music-generation-references/fun-music-api.md) 所述，当同时传入 `lyrics` 和 `[[prompt|prompt]]` 时，仅 `lyrics` 生效，`prompt` 将被忽略。

### 字符限制

| 参数 | 非流式模式 | 流式模式 |
|------|-----------|---------|
| `lyrics` | 中文 5~350 字符，英文 5~2000 字符 | 中文 300~350 字，英文 200~250 词 |
| `prompt` | 1~2000 字符 | 5~1000 个中文汉字或英文单词 |

## 使用方式

### 非流式调用

```bash
curl -X POST 'https://dashscope.aliyuncs.com/api/v1/services/audio/music/generation' \
-H "Authorization: Bearer $DASHSCOPE_API_KEY" \
-H "Content-Type: application/json" \
-d '{
    "model": "fun-music-v1",
    "input": {
        "prompt": "夏日清新民谣，木吉他与口琴伴奏，轻快节奏，适合旅行Vlog背景音乐",
        "gender": "female"
    }
}'
```

非流式模式直接返回完整音频的 OSS URL。

### 流式调用

添加请求头 `X-DashScope-SSE: enable`，响应以 SSE 事件流返回：
- **中间消息**：`audio.data` 包含 Base64 编码的音频片段，`finish_reason` 为 `null`
- **最终消息**：包含完整音频 URL、歌词、采样率等元信息，`finish_reason` 为 `stop`

## 返回结构

| 字段 | 说明 |
|------|------|
| `output.audio.url` | 完整音频 OSS URL，有效期 24 小时 |
| `output.audio.data` | 流式模式下的 Base64 音频片段 |
| `output.extra_info.lyrics` | 生成的歌词内容 |
| `output.extra_info.channels` | 声道数（2 = 立体声） |
| `output.extra_info.sample_rate` | 采样率（如 48000） |
| `output.finish_reason` | `null` 生成中 / `stop` 生成结束 |
| `usage.duration` | 音乐时长（秒），用于计费 |

## 限制和注意事项

- **地域限制**：仅在中国内地（北京地域）可用
- **访问权限**：邀测阶段，需申请开通
- **输入互斥**：`lyrics` 和 `prompt` 同时存在时以 `lyrics` 为准
- **流式模式字符限制更严格**：歌词在流式模式下要求中文 300~350 字，远高于非流式模式的最低 5 字符
- **URL 有效期**：音频下载链接 24 小时后过期
- **AIGC 水印**：开启后会在音频末尾追加摩尔斯电码信号（`·— ··`），增加音频时长

详细参数说明和完整返回示例请参阅 [音乐生成Fun-Music API参考](../../raw/model-api-reference/music-generation-references/fun-music-api.md)。

## 相关概念

- [[get-api-key]] - 获取 API Key
- [[audio-generation-references]] - 音频生成相关 API
- [[billing]] - 计费说明

## 来源文档

- [音乐生成Fun-Music API参考](../../raw/model-api-reference/music-generation-references/fun-music-api.md)

