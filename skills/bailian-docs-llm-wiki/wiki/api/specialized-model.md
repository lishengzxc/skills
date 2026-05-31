# specialized model

百炼平台提供多个专用模型（Specialized Model），针对特定任务场景进行优化，包括机器翻译、深度研究、文字提取（OCR）和界面交互自动化。这些模型通过 [[openai-compatible-api|OpenAI 兼容接口]]或 DashScope API 调用，各自有独立的参数体系和使用方式。开发者可根据业务场景选择合适的专用模型，获得优于通用模型的任务表现。

## 支持的模型

| 模型 | 用途 | 支持的接口 | 多地域部署 |
|------|------|-----------|-----------|
| `qwen-mt-plus` | 机器翻译 | OpenAI 兼容 / DashScope | 北京、新加坡、弗吉尼亚 |
| `qwen-deep-research` | 深度研究报告生成 | DashScope（仅 Python SDK） | 仅北京 |
| `qwen-vl-ocr-latest` | 图像文字提取 | OpenAI 兼容 / DashScope | 北京、新加坡、弗吉尼亚 |
| `gui-plus-2026-02-26` | GUI 界面交互自动化 | OpenAI 兼容 / DashScope | 仅北京 |

> **注意**：Qwen-Deep-Research 当前仅支持 Python DashScope SDK 调用，暂不支持 Java SDK 与 [[openai-compatible-api|OpenAI 兼容接口]]。详见 [Qwen-Deep-Research API 参考](../../raw/model-api-reference/specialized-model/qwen-deep-research-api.md)。

## 各模型功能与关键参数

### Qwen-MT（机器翻译）

Qwen-MT 是专用翻译模型，通过 `translation_options` 参数控制翻译行为。详细参数与示例参见 [Qwen-MT API 参考](../../raw/model-api-reference/specialized-model/qwen-mt-api.md)。

核心参数（通过 `translation_options` 传入）：

- **`source_lang`**：源语言，支持 `"auto"` 自动检测
- **`target_lang`**：目标语言（必选）
- **`terms`**：术语干预列表，每项包含 `source`/`target` 对，强制模型使用指定译法
- **`tm_list`**：翻译记忆列表，提供参考译文帮助模型保持风格一致
- **`domains`**：领域提示文本，引导模型按特定领域风格翻译

使用 OpenAI SDK 时，`translation_options` 通过 `extra_body` 传入：

```python
completion = client.chat.completions.create(
    model="qwen-mt-plus",
    messages=[{"role": "user", "content": "待翻译文本"}],
    extra_body={
        "translation_options": {
            "source_lang": "Chinese",
            "target_lang": "English",
            "terms": [{"source": "石墨烯", "target": "graphene"}]
        }
    }
)
```

### Qwen-Deep-Research（深度研究）

Qwen-Deep-Research 采用**两阶段调用流程**：先由模型反问确认研究方向，再进行深入研究并生成报告。响应通过[[streaming|流式输出]]，包含多个阶段（`ResearchPlanning` → `WebResearch` → `KeepAlive` → `answer`）。

关键参数：

- **`output_format`**：控制报告详细程度
  - `model_detailed_report`（默认）：约 6000 Token 的完整报告
  - `model_summary_report`：约 1500-2000 Token 的摘要报告

响应中的 `phase` 字段标识当前阶段，`extra.deep_research` 包含搜索查询、参考来源等元信息。

### Qwen-OCR（文字提取）

Qwen-OCR 专用于从图像中提取文字，支持自定义 Prompt 控制提取行为。详细用法参见 [Qwen-OCR API参考](../../raw/model-api-reference/specialized-model/qwen-vl-ocr-api-reference.md)。

关键参数：

- **`image_url.url`**：图片 URL 或 Base64 编码
- **`min_pixels`** / **`max_pixels`**：控制输入图像像素阈值，影响图像缩放行为
  - 默认 `min_pixels`: `3072`（即 32×32×3）
  - 默认 `max_pixels`: `8388608`（即 32×32×8192）
- **`text`**（可选）：自定义提取 Prompt。未传入时使用默认 Prompt：`Please output only the text content from the image without any additional descriptions or formatting.`

支持流式和非流式两种输出模式。

### GUI-Plus（界面交互）

GUI-Plus 是界面交互专用模型，用于自动化操控桌面 GUI。模型接收屏幕截图和用户指令，输出具体的交互操作（点击、输入、滚动等）。

使用要点：

- 需通过 `system` 消息注入 [[tool-calling]] 的工具定义（`computer_use` 函数）
- 用户消息同时包含截图（`image_url`）和任务描述（`text`）
- 建议开启 `vl_high_resolution_images: true`（通过 `extra_body` 传入）
- 模型以 `<tool_call>` XML 标签返回操作指令

## 通用使用方式

### 接口地址

大部分专用模型支持 [[openai-compatible-api|OpenAI 兼容接口]]，不同地域的 `base_url` 如下：

| 地域 | base_url |
|------|----------|
| 北京 | `https://dashscope.aliyuncs.com/compatible-mode/v1` |
| 新加坡 | `https://dashscope-intl.aliyuncs.com/compatible-mode/v1` |
| 弗吉尼亚 | `https://dashscope-us.aliyuncs.com/compatible-mode/v1` |

### 认证

所有模型调用前需要：
1. 获取 [[api-key]]
2. 将 API Key 配置到环境变量 `DASHSCOPE_API_KEY`
3. 安装对应 SDK（OpenAI SDK 或 DashScope SDK）

> **注意**：不同地域的 API Key 不通用，新加坡和北京地域需分别获取。

## 限制和注意事项

- **Qwen-Deep-Research** 仅适用于中国大陆版（北京），必须使用北京地域的 API Key，且当前仅支持 Python DashScope SDK。
- **Qwen-MT** 的 `messages` 仅支持 `user` 角色，不支持 `system` 消息；翻译内容放在 `content` 字段中。
- **Qwen-OCR** 的图像像素参数会影响识别精度和 Token 消耗，需根据图片实际分辨率合理设置 `min_pixels` 和 `max_pixels`。
- **GUI-Plus** 默认屏幕分辨率假设为 1000×1000，截图需匹配此分辨率或在 system [[prompt|prompt]] 中调整。
- 各专用模型均不支持通用 [[chat-completions]] 的全部参数（如 `temperature`、`top_p` 等），具体支持情况请参考各模型的 API 参考文档。

## 来源文档

- [Qwen-MT API 参考](../../raw/model-api-reference/specialized-model/qwen-mt-api.md)
- [Qwen-Deep-Research API 参考](../../raw/model-api-reference/specialized-model/qwen-deep-research-api.md)
- [Qwen-OCR API参考](../../raw/model-api-reference/specialized-model/qwen-vl-ocr-api-reference.md)
- [GUI-Plus API参考](../../raw/model-api-reference/specialized-model/gui-plus-interface-interaction-model.md)

