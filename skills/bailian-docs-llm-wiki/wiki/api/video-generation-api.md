# video generation api

百炼平台提供多系列视频生成 API，覆盖文生视频、图生视频、参考生视频、视频编辑、数字人等场景。所有视频生成 API 均采用**异步调用**模式（创建任务 → 轮询获取结果），支持 HTTP 和部分模型的 DashScope SDK 调用方式。开发者需确保模型、Endpoint URL 和 API Key 属于同一地域，跨地域调用将失败。

---

## 支持的模型与功能矩阵

### 万相系列（Wan）

| 功能 | 推荐模型 | 旧版模型 |
|------|----------|----------|
| 文生视频 | `wan2.7-t2v-*` | `wan2.6-t2v` / `wan2.5-*` / `wanx2.1-*` |
| 图生视频（首帧/首尾帧/视频续写） | `wan2.7-i2v-*` | `wan2.6-i2v-*` / `wan2.2-kf2v-*` |
| 参考生视频 | `wan2.7-r2v` | `wan2.6-r2v-flash` |
| 视频编辑 | `wan2.7-videoedit` | `wanx2.1-vace-plus` |
| 图生动作 | `wan2.2-animate-move` | — |
| 视频换人 | `wan2.2-animate-mix` | — |
| 数字人 | `wan2.2-s2v` | — |
| 视频风格重绘 | `video-style-transform` | — |

> **注意**：wan2.7 模型使用**新版协议**（`media` 数组传入多模态素材），wan2.6 及更早模型使用**旧版协议**（`img_url`/`reference_urls` 等独立字段）。新项目推荐使用 wan2.7，详见 [万相2.7-图生视频API参考](../../raw/model-api-reference/video-generation-api/image-to-video-general-api-reference.md) 和 [万相2.7-文生视频API参考](../../raw/model-api-reference/video-generation-api/text-to-video-api-reference.md)。

### 第三方模型

| 模型系列 | 支持功能 | 模型名称示例 |
|----------|----------|-------------|
| **HappyHorse** | 文生视频、图生视频、参考生视频、视频编辑 | `happyhorse-1.0-t2v`、`happyhorse-1.0-i2v`、`happyhorse-1.0-r2v`、`happyhorse-1.0-video-edit` |
| **PixVerse（爱诗）** | 文生视频、图生视频、首尾帧生视频、参考生视频 | `pixverse/pixverse-c1-t2v`、`pixverse/pixverse-c1-it2v` 等 |
| **Kling（可灵）** | 文生视频、图生视频、首尾帧、参考生视频、视频编辑 | `kling/kling-v3-video-generation`、`kling/kling-v3-omni-video-generation` |
| **Vidu** | 文生视频、图生视频、首尾帧、参考生视频 | `vidu/viduq3-pro_text2video`、`vidu/viduq3-turbo_img2video` 等 |

> **注意**：PixVerse、Kling、Vidu 等第三方模型需先在百炼控制台搜索并**立即开通**后才能调用。部分第三方模型仅支持北京地域。

### 人像/动作类专项模型

| 模型 | 功能 | 调用流程 |
|------|------|----------|
| 舞动人像 AnimateAnyone | 基于人物图片+动作模板生成舞蹈视频 | detect → template → 生成（三步） |
| 悦动人像 EMO | 基于肖像图片+音频生成说唱/唱歌视频 | detect → 生成（两步） |
| 灵动人像 LivePortrait | 基于肖像图片+音频快速生成播报视频 | detect → 生成（两步） |
| 数字人 wan2.2-s2v | 基于单张图片+音频生成说话/唱歌视频，支持全身 | detect → 生成（两步） |
| 声动人像 VideoRetalk | 基于视频+音频替换口型 | 一步异步调用 |
| 表情包 Emoji | 基于肖像+预设模板生成表情包视频 | detect → 生成（两步） |

详见 [万相-数字人](../../raw/model-api-reference/video-generation-api/wan-s2v-overview.md) 和 [图生唱演视频-悦动人像EMO](../../raw/model-api-reference/video-generation-api/emo-quick-start.md)。

---

## 调用流程

所有视频生成 API 均为**异步调用**，分两步完成：

### 步骤 1：创建任务

```bash
curl -X POST 'https://dashscope.aliyuncs.com/api/v1/services/aigc/video-generation/video-synthesis' \
  -H 'X-DashScope-Async: enable' \
  -H 'Authorization: Bearer $DASHSCOPE_API_KEY' \
  -H 'Content-Type: application/json' \
  -d '{
    "model": "<模型名称>",
    "input": { "[[prompt|prompt]]": "描述文本", ... },
    "parameters": { "resolution": "720P", "duration": 5, ... }
  }'
```

成功后返回 `task_id`。

### 步骤 2：轮询获取结果

```bash
curl -X GET 'https://dashscope.aliyuncs.com/api/v1/tasks/<task_id>' \
  -H 'Authorization: Bearer $DASHSCOPE_API_KEY'
```

任务状态为 `SUCCEEDED` 时，`output` 中返回视频 URL。

> **注意**：`task_id` 有效期为 **24 小时**，过期后查询将返回 `UNKNOWN`。请勿对同一请求重复创建任务。

### 地域 Endpoint

| 地域 | Endpoint |
|------|----------|
| 华北2（北京） | `https://dashscope.aliyuncs.com` |
| 新加坡 | `https://dashscope-intl.aliyuncs.com` |
| 美国（弗吉尼亚） | `https://dashscope-us.aliyuncs.com` |
| 德国（法兰克福） | `https://{WorkspaceId}.eu-central-1.maas.aliyuncs.com` |

> **注意**：不同模型支持的地域范围不同。wan2.7 目前仅支持北京和新加坡；HappyHorse 支持四个地域；部分第三方模型（PixVerse、Kling、Vidu）仅支持北京。

---

## 关键参数

### 通用请求头

| 参数 | 必选 | 说明 |
|------|------|------|
| `Content-Type` | 是 | 固定 `application/json` |
| `Authorization` | 是 | `Bearer <API_KEY>` |
| `X-DashScope-Async` | 是 | 固定 `enable`，缺失将报错 |

### 常见请求体参数

| 参数 | 说明 | 备注 |
|------|------|------|
| `model` | 模型名称 | 不同任务对应不同模型 ID |
| `input.[[prompt|prompt]]` | 文本提示词 | 最大 5000 字符（中文约 2500 字） |
| `input.media` | 多模态素材数组（新版协议） | 支持 `first_frame`、`last_frame`、`video`、`reference_image`、`reference_video`、`driving_audio` 等 type |
| `parameters.resolution` | 视频分辨率 | 常见值：`480P`、`540P`、`720P`、`1080P`（因模型而异） |
| `parameters.duration` | 视频时长（秒） | 常见值：5、10、15（因模型而异） |
| `parameters.ratio` / `parameters.aspect_ratio` / `parameters.size` | 画面比例/尺寸 | 不同模型使用不同参数名 |
| `parameters.[[prompt|prompt]]_extend` | 是否开启提示词智能改写 | 布尔值，部分模型默认开启 |
| `parameters.watermark` | 是否添加水印 | 布尔值 |

> **注意**：参数名称在不同模型间存在差异。例如画面比例在万相 wan2.7 中用 `ratio`（如 `"16:9"`），在可灵 Kling 中用 `aspect_ratio`，在 PixVerse/Vidu 中用 `size`（如 `"1280*720"`）。请以各模型的 API 参考文档为准。

---

## 多镜头（Multi-Shot）支持

部分模型支持在单次调用中生成多镜头叙事视频：

- **wan2.7**：在 `prompt` 中自然描述镜头结构（如使用时间戳 `[0-3秒]`），无需额外参数
- **wan2.6**：需设置 `"shot_type": "multi"` 和 `"prompt_extend": true`
- **Kling**：通过 `multi_shot: true` + `shot_type` + `multi_prompt` 数组实现自定义分镜
- **PixVerse (c1)**：在 `prompt` 中描述多镜头场景即可

---

## 限制和注意事项

- **异步调用**：所有视频生成接口仅支持异步模式。缺少 `X-DashScope-Async: enable` 请求头将返回错误。
- **处理时间**：通常 1-5 分钟，视频编辑统一模型（wanx2.1-vace）约 5-10 分钟。
- **地域隔离**：模型、Endpoint URL、[[api-key]] 必须属于同一地域，跨地域调用将直接失败。
- **并发限制**：不同模型的并发任务数和 QPS 限制不同，人像类模型通常同时仅允许 1 个任务运行，其余排队。详见各模型的 [[rate-limit]] 说明。
- **计费**：各模型计费方式不同（按时长/按张数），部分模型提供 [[free-quota]]。AnimateAnyone 和 EMO 还支持模型独立部署（预付费）模式。
- **输入图像**：人像类模型要求先调用对应的 detect 接口进行图像合规检测，检测通过后再提交生成任务。
- **输出有效期**：生成的视频 URL 有时效限制，建议及时下载保存。

---

## 相关概念

- [[api-key]] — API Key 获取与配置
- [[dashscope-sdk]] — DashScope SDK 安装
- [[image-generation-api]] — 图像生成 API
- [[rate-limit]] — 限流说明
- [[free-quota]] — 免费额度

## 来源文档

- [HappyHorse-文生视频API参考](../../raw/model-api-reference/video-generation-api/happyhorse-text-to-video-api-reference.md)
- [HappyHorse-图生视频-基于首帧API参考](../../raw/model-api-reference/video-generation-api/happyhorse-image-to-video-api-reference.md)
- [HappyHorse-参考生视频API参考](../../raw/model-api-reference/video-generation-api/happyhorse-reference-to-video-api-reference.md)
- [万相2.7-图生视频API参考](../../raw/model-api-reference/video-generation-api/image-to-video-general-api-reference.md)
- [HappyHorse-视频编辑API参考](../../raw/model-api-reference/video-generation-api/happyhorse-video-edit-api-reference.md)
- [万相2.7-文生视频API参考](../../raw/model-api-reference/video-generation-api/text-to-video-api-reference.md)
- [万相2.7-参考生视频API参考](../../raw/model-api-reference/video-generation-api/wan-video-to-video-api-reference.md)
- [万相2.7-视频编辑API参考](../../raw/model-api-reference/video-generation-api/wan-video-editing-api-reference.md)
- [万相-图生动作API参考](../../raw/model-api-reference/video-generation-api/wan-animate-move-api.md)
- [万相-视频换人API参考](../../raw/model-api-reference/video-generation-api/wan-animate-mix-api.md)
- [万相-数字人](../../raw/model-api-reference/video-generation-api/wan-s2v-overview.md)
- [图生舞蹈视频-舞动人像AnimateAnyone](../../raw/model-api-reference/video-generation-api/animateanyone-quick-start.md)
- [图生唱演视频-悦动人像EMO](../../raw/model-api-reference/video-generation-api/emo-quick-start.md)
- [图生播报视频-灵动人像LivePortrait](../../raw/model-api-reference/video-generation-api/liveportrait-quick-start.md)
- [图生表情包视频-表情包Emoji](../../raw/model-api-reference/video-generation-api/emoji-quick-start.md)
- [视频口型替换-声动人像VideoRetalk](../../raw/model-api-reference/video-generation-api/videoretalk.md)
- [视频风格重绘API参考](../../raw/model-api-reference/video-generation-api/video-style-transform-api-reference.md)
- [爱诗-图生视频-基于首帧API参考](../../raw/model-api-reference/video-generation-api/pixverse-image-to-video-api-reference.md)
- [爱诗-文生视频API参考](../../raw/model-api-reference/video-generation-api/pixverse-text-to-video-api-reference.md)
- [爱诗-图生视频-基于首尾帧API参考](../../raw/model-api-reference/video-generation-api/pixverse-keyframe-to-video-api-reference.md)
- [爱诗-参考生视频API参考](../../raw/model-api-reference/video-generation-api/pixverse-reference-to-video-api-reference.md)
- [可灵-视频生成API文档](../../raw/model-api-reference/video-generation-api/kling-video-generation-api-reference.md)
- [Vidu-图生视频-基于首帧API参考](../../raw/model-api-reference/video-generation-api/vidu-image-to-video-api-reference.md)
- [Vidu-图生视频-基于首尾帧API参考](../../raw/model-api-reference/video-generation-api/vidu-keyframe-to-video-api-reference.md)
- [Vidu-文生视频API参考](../../raw/model-api-reference/video-generation-api/vidu-text-to-video-api-reference.md)
- [Vidu-参考生视频 API 参考](../../raw/model-api-reference/video-generation-api/vidu-reference-to-video-api-reference.md)
- [万相-参考生视频API参考（2.6）](../../raw/model-api-reference/video-generation-api/legacy-video-models/legacy-wan-reference-to-video-api-reference.md)
- [万相-文生视频API参考（2.1-2.6）](../../raw/model-api-reference/video-generation-api/legacy-video-models/legacy-wan-text-to-video-api-reference.md)
- [万相-图生视频-基于首帧API参考（2.1-2.6）](../../raw/model-api-reference/video-generation-api/legacy-video-models/legacy-image-to-video-api-reference.md)
- [万相-视频编辑API参考（2.1）](../../raw/model-api-reference/video-generation-api/legacy-video-models/legacy-wanx-vace-api-reference.md)
- [万相-首尾帧生视频API参考（2.2）](../../raw/model-api-reference/video-generation-api/legacy-video-models/legacy-image-to-video-by-first-and-last-frame-api-reference.md)

