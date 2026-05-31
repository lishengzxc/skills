# image generation

百炼平台提供丰富的图像生成与编辑能力，涵盖文生图、图像编辑、图像翻译、风格迁移、背景生成等多种场景。平台支持千问（Qwen-Image）、万相（Wan/Wanx）、Z-Image、可灵（Kling）等多个模型系列，开发者可通过 HTTP API 或 DashScope SDK 进行调用。

---

## 支持的模型与功能

### 文生图模型

| 模型系列 | 代表模型 | 特点 | 输出规格 |
|---------|---------|------|---------|
| **千问-文生图** | `qwen-image-2.0-pro`（推荐）、`qwen-image-2.0`、`qwen-image-max` | 擅长复杂文本渲染、多行布局、图文混合设计 | 512×512 ~ 2048×2048，PNG |
| **万相2.7** | `wan2.7-image-pro`、`wan2.7-image` | 文生图支持4K输出，支持图像编辑和组图生成 | PNG，支持2K/4K |
| **万相2.6文生图** | `wan2.6-t2i` | 自由选尺寸，支持图文混排输出 | 1280×1280 ~ 1440×1440，PNG |
| **万相2.5及以下** | `wan2.5-t2i-preview`、`wan2.2-t2i-flash`、`wanx2.1-t2i-turbo` 等 | 多版本可选，速度与质量各有侧重 | PNG |
| **万相V1** | `wanx-v1` | 支持参考图内容/风格迁移 | 仅限北京地域 |
| **Z-Image** | `z-image-turbo` | 轻量快速，支持中英文字渲染 | 512×512 ~ 2048×2048，PNG |
| **可灵** | `kling/kling-v3-image-generation`、`kling/kling-v3-omni-image-generation` | 支持文生图和参考图生图，omni版支持组图模式 | 1K/2K/4K，PNG |

详细参数请参见 [千问-文生图API参考](../../raw/model-api-reference/image-generation/qwen-image-api.md) 和 [万相-文生图V2版API参考](../../raw/model-api-reference/image-generation/text-to-image-v2-api-reference.md)。

### 图像编辑模型

| 模型 | 功能 | 说明 |
|------|------|------|
| `qwen-image-2.0-pro` / `qwen-image-edit-max` | 多图输入输出、文字修改、物体增删、风格迁移 | 千问系列，生成与编辑共用部分模型名 |
| `wan2.7-image-pro` / `wan2.6-image` | 图像编辑、交互式编辑、图文混排 | 万相新版，支持同步调用 |
| `wan2.5-i2i-preview` | 单图编辑、多图融合 | 异步调用 |
| `wanx2.1-imageedit` | 风格化、指令编辑、局部重绘、去水印、扩图、超分、上色、线稿生图等 | 功能最丰富的编辑模型 |

详情请参见 [千问-图像编辑API参考](../../raw/model-api-reference/image-generation/qwen-image-edit-api.md)。

### 专项模型

| 模型 | 用途 | 状态 |
|------|------|------|
| `qwen-mt-image` | 图像文字翻译，保留原始排版 | 仅北京地域 |
| `wanx-style-repaint-v1` | 人像风格重绘 | 付费（0.12元/张） |
| `image-out-painting` | 图像画面扩展（扩图） | 付费（0.18元/张） |
| `wanx-background-generation-v2` | 图像背景生成 | 付费（0.08元/张） |
| `wanx-sketch-to-image-lite` | 涂鸦作画 | 付费（0.06元/张） |
| `wanx-poster-generation-v1` | 创意海报生成 | 仅免费体验 |
| `wanx-virtualmodel` / `virtualmodel-v2` | 虚拟模特 | 仅免费体验 |
| `shoemodel-v1` | 鞋靴模特试穿 | 仅免费体验 |
| `image-instance-segmentation` | 人物实例分割 | 仅免费体验 |
| `image-erase-completion` | 图像擦除补全 | 仅免费体验 |
| `wanx-x-painting` | 图像局部重绘 | 仅免费体验 |
| AI试衣 OutfitAnyone | 服装试穿（基础版/Plus版/精修/分割） | 参见 [[outfitanyone]] |
| FaceChain | 人物写真生成 | 参见 [[facechain-portrait-generation]] |
| WordArt 锦书 | 创意文字变形与纹理生成 | 参见 [[wordart-quick-start]] |

> **注意**：标记"仅免费体验"的模型，免费额度用完后不可调用且不支持付费。官方推荐使用千问图像编辑或万相2.1作为替代方案。

---

## 关键参数

### 通用参数

| 参数 | 说明 | 适用模型 |
|------|------|---------|
| `model` | 模型名称（必选） | 全部 |
| `[[prompt|prompt]]` / `messages[].content[].text` | 正向提示词 | 全部（新版协议使用 messages 格式） |
| `negative_[[prompt|prompt]]` | 反向提示词，排除不希望出现的元素 | 万相系列、千问系列 |
| `size` | 输出图像分辨率，格式为 `宽*高`，如 `1024*1024` | 大部分模型 |
| `n` | 输出图像数量 | 大部分模型（部分固定1张） |
| `[[prompt|prompt]]_extend` | 是否启用提示词智能扩展/优化 | 万相2.6+、Z-Image、千问系列 |
| `watermark` | 是否添加水印 | 万相系列 |
| `seed` | 随机种子，用于结果复现 | 千问、万相部分模型 |

### 分辨率说明

不同模型对分辨率的支持方式有所不同：

- **千问系列**（qwen-image-2.0-pro/2.0）：总像素在 512×512 至 2048×2048 之间自由设置宽高
- **万相2.6文生图**：总像素在 1280×1280 至 1440×1440 之间
- **万相2.7**：支持 `1K`、`2K`、`4K` 等预设值
- **可灵**：通过 `resolution`（1k/2k/4k）和 `aspect_ratio`（16:9/9:16/1:1）组合指定
- **Z-Image**：总像素在 512×512 至 2048×2048 之间

---

## 使用方式

### 调用协议

平台图像生成 API 有两种调用模式：

| 模式 | 说明 | 支持的模型 |
|------|------|-----------|
| **同步调用** | 一次请求直接返回结果，流程简单 | 千问系列、万相2.6+、万相2.7、Z-Image |
| **异步调用** | 先创建任务获取 `task_id`，再轮询查询结果 | 所有模型均支持；万相2.5及以下、可灵、V1版模型仅支持异步 |

### 请求地址

| 地域 | 地址 |
|------|------|
| 北京 | `https://dashscope.aliyuncs.com/api/v1/services/aigc/...` |
| 新加坡 | `https://dashscope-intl.aliyuncs.com/api/v1/services/aigc/...` |
| 弗吉尼亚 | `https://dashscope-us.aliyuncs.com/api/v1/services/aigc/...` |

> **注意**：各地域的 **API Key** 和**请求地址**独立，不可混用，跨地域调用会导致鉴权失败。不同模型支持的地域不同，请在[百炼控制台](https://bailian.console.aliyun.com/cn-beijing?tab=model#/model-market/all)查看。

### SDK 支持

- **DashScope Python SDK** 和 **Java SDK**：支持大部分模型
- 部分专项模型（如人像风格重绘、创意海报）仅提供 HTTP API

### 前提条件

调用前需完成：
1. [获取 API Key](https://help.aliyun.com/zh/model-studio/get-api-key)
2. [配置 API Key 到环境变量](https://help.aliyun.com/zh/model-studio/configure-api-key-through-environment-variables)
3. 如使用 SDK，需 [安装 DashScope SDK](https://help.aliyun.com/zh/model-studio/install-sdk)

### 快速示例（文生图-同步调用）

```bash
curl --location 'https://dashscope.aliyuncs.com/api/v1/services/aigc/multimodal-generation/generation' \
--header 'Content-Type: application/json' \
--header "Authorization: Bearer $DASHSCOPE_API_KEY" \
--data '{
    "model": "qwen-image-2.0-pro",
    "input": {
        "messages": [
            {
                "role": "user",
                "content": [
                    {"text": "一间有着精致窗户的花店，漂亮的木质门，摆放着花朵"}
                ]
            }
        ]
    },
    "parameters": {
        "size": "1024*1024"
    }
}'
```

---

## 计费与限流

- **免费额度**：开通后自动发放，一般为 500 张，有效期 90 天，主账号与 RAM 子账号共享
- **计费项**：仅对成功生成的输出图片计费，输入图片及失败任务不收费
- **限流**：主账号与 RAM 子账号共享。典型限制为任务下发 QPS 2、同时处理中任务数 1-5（因模型而异）
- 调用情况可在[模型观测](https://bailian.console.aliyun.com/#/model-telemetry)页面查看

详细计费信息请参见 [常见问题](../../raw/model-api-reference/image-generation

## 来源文档

- [千问-文生图API参考](../../raw/model-api-reference/image-generation/qwen-image-api.md)
- [千问-图像编辑API参考](../../raw/model-api-reference/image-generation/qwen-image-edit-api.md)
- [千问-图像翻译API参考](../../raw/model-api-reference/image-generation/qwen-mt-image-api.md)
- [Z-Image API参考](../../raw/model-api-reference/image-generation/z-image-api-reference.md)
- [万相-文生图V2版API参考](../../raw/model-api-reference/image-generation/text-to-image-v2-api-reference.md)
- [万相-文生图V1版API参考](../../raw/model-api-reference/image-generation/text-to-image-api-reference.md)
- [万相-图像生成与编辑2.7 API参考](../../raw/model-api-reference/image-generation/wan-image-generation-and-editing-api-reference.md)
- [万相-图像生成与编辑2.6 API参考](../../raw/model-api-reference/image-generation/wan-image-generation-api-reference.md)
- [万相-通用图像编辑2.5](../../raw/model-api-reference/image-generation/wan2-5-image-edit-api-reference.md)
- [万相-通用图像编辑API参考](../../raw/model-api-reference/image-generation/wanx-image-edit-api-reference.md)
- [万相-涂鸦作画API参考](../../raw/model-api-reference/image-generation/wanx-sketch-to-image-api-reference.md)
- [万相-图像局部重绘API参考](../../raw/model-api-reference/image-generation/vary-region-api-reference.md)
- [人像风格重绘API参考](../../raw/model-api-reference/image-generation/portrait-style-redraw-api-reference.md)
- [图像画面扩展API参考](../../raw/model-api-reference/image-generation/image-scaling-api.md)
- [虚拟模特API参考](../../raw/model-api-reference/image-generation/virtual-model-api-details.md)
- [鞋靴模特API参考](../../raw/model-api-reference/image-generation/shoe-model-api.md)
- [创意海报生成API参考](../../raw/model-api-reference/image-generation/creative-poster-generation-api.md)
- [人物实例分割API参考](../../raw/model-api-reference/image-generation/image-instance-segmentation-api-reference.md)
- [图像背景生成API参考](../../raw/model-api-reference/image-generation/wanx-background-generation-api-reference.md)
- [图像擦除补全API参考](../../raw/model-api-reference/image-generation/image-erase-completion-api-reference.md)
- [AI试衣OutfitAnyone](../../raw/model-api-reference/image-generation/outfitanyone.md)
- [人物写真生成FaceChain](../../raw/model-api-reference/image-generation/facechain-portrait-generation.md)
- [创意文字WordArt锦书](../../raw/model-api-reference/image-generation/wordart-quick-start.md)
- [可灵-图像生成API参考](../../raw/model-api-reference/image-generation/kling-image-generation-api-reference.md)
- [常见问题](../../raw/model-api-reference/image-generation/image-faq.md)

