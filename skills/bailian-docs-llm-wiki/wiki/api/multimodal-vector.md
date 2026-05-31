# multimodal vector

多模态向量（Multimodal Embedding）是百炼平台提供的一类模型能力，可将文本、图片和视频转换为同一语义空间中的向量表示，从而支持跨模态检索（以文搜图、以图搜视频等）、语义相似度计算和内容分类聚类。所有模态生成的向量位于统一空间，可直接通过余弦相似度等方法进行跨模态匹配。

## 支持的模型

根据 [Multimodal-Embedding API详情](../../raw/model-api-reference/multimodal-vector/multimodal-embedding-api-reference.md)，百炼平台提供以下多模态向量模型：

| 模型 | 默认维度 | 向量类型 | 特点 |
|------|---------|---------|------|
| `qwen3-vl-embedding` | 2560 | 独立 / 融合 | 支持 33 种语言，通过 `enable_fusion=true` 开启融合模式 |
| `qwen2.5-vl-embedding` | 1024 | 仅融合 | 始终返回 1 个融合向量，不支持独立向量和多图输入 |
| `tongyi-embedding-vision-plus-2026-03-06` | 1152 | 独立 / 融合 | 基于 Qwen3 底座，支持多分辨率、30+ 语言 |
| `tongyi-embedding-vision-flash-2026-03-06` | 768 | 独立 / 融合 | 同上，轻量版 |
| `tongyi-embedding-vision-plus` | 1152 | 仅独立 | 支持 `multi_images`（最多 8 张） |
| `tongyi-embedding-vision-flash` | 768 | 仅独立 | 轻量版 |
| `multimodal-embedding-v1` | 1024 | 独立 | 固定维度，不支持 `dimension` 参数 |

## 向量类型：独立向量与融合向量

多模态向量模型支持两种生成方式：

- **独立向量**：为 `contents` 中每个输入分别生成向量。输入 1 段文本和 1 张图片，返回 2 个向量。适用于以图搜图、以文搜图等逐项对比场景。
- **融合向量**：将所有输入融合编码为 1 个向量，实现跨模态综合语义表征。适用于将商品图片和描述文本融合为统一表征进行检索等场景。

融合向量的开启方式因模型而异：

- `qwen3-vl-embedding`：设置 `enable_fusion=true`
- `tongyi-embedding-vision-plus-2026-03-06` / `flash-2026-03-06`：将 text、image、video 放在**同一个 content 对象**中
- `qwen2.5-vl-embedding`：默认且仅支持融合

融合支持的组合包括：文本+图片、文本+视频、多图+文本、图片+视频+文本混合。

## 关键参数

根据 [Multimodal-Embedding API详情](../../raw/model-api-reference/multimodal-vector/multimodal-embedding-api-reference.md) 中的参数说明：

| 参数 | 类型 | 说明 | 适用模型 |
|------|------|------|---------|
| `dimension` | integer | 指定输出向量维度，不同模型支持的值不同 | 除 `multimodal-embedding-v1`、`tongyi-embedding-vision-plus/flash` 外均支持 |
| `enable_fusion` | bool | 是否生成融合向量，默认 `false` | 仅 `qwen3-vl-embedding` |
| `fps` | float | 控制视频抽帧比例，范围 [0,1]，默认 1.0 | 全部 |
| `instruct` | string | 自定义任务说明，指导模型理解查询意图（建议英文） | 全部 |
| `res_level` | integer | 输入分辨率档位 (0/1/2/3)，默认 1 | 仅 `2026-03-06` 快照版本 |
| `max_video_frames` | integer | 视频最大采样帧数上限，最大 64，默认 8 | 仅 `2026-03-06` 快照版本 |

## 使用方式

### HTTP 调用

请求端点：

```
POST https://dashscope.aliyuncs.com/api/v1/services/embeddings/multimodal-embedding/multimodal-embedding
```

请求体核心结构：

```json
{
    "model": "模型名称",
    "input": {
        "contents": [
            {"text": "文本内容"},
            {"image": "图片URL或Base64"},
            {"video": "视频URL"}
        ]
    },
    "parameters": {
        "dimension": 1024,
        "enable_fusion": false
    }
}
```

`contents` 支持四种模态类型：`text`、`image`、`video`、`multi_images`。图片支持 URL 和 Base64 Data URI 两种传入方式；视频仅支持 URL。

### 前提条件

调用前需获取 [[api-key]] 并配置到环境变量。通过 SDK 调用还需安装 [[dashscope-sdk]]。

## 输入限制

| 模型 | 文本长度 | 图片大小 | 视频大小 | 单次请求限制 |
|------|---------|---------|---------|------------|
| `qwen3-vl-embedding` | 32,000 Token | ≤5 MB | ≤50 MB | 总数≤20，图片≤5，视频≤1 |
| `qwen2.5-vl-embedding` | 32,000 Token | ≤5 MB | ≤50 MB | 每种类型最多 1 次 |
| `tongyi-embedding-vision-plus-2026-03-06` | 1,024 Token | 建议≤5 MB，最大 10 MB，最多 64 张 | ≤50 MB (H.264/H.265) | 总数≤20，图片≤64，视频≤8 |
| `tongyi-embedding-vision-plus` | 1,024 Token | ≤3 MB，最多 8 张 | ≤10 MB | 按 Token 上限控制 |
| `multimodal-embedding-v1` | 512 Token | ≤3 MB | ≤10 MB | 总数≤20 |

## 注意事项

> **注意**：`multi_images` 类型仅 `tongyi-embedding-vision-plus`、`tongyi-embedding-vision-flash` 及其 `2026-03-06` 快照版本支持，`qwen3-vl-embedding` 通过传入多个 `image` 条目实现多图输入，而 `qwen2.5-vl-embedding` 不支持多图。

- 所有模型均支持 text、image、video 三种输入类型及组合。
- `res_level` 参数对 IPC/自驾/视觉文字等分辨率敏感场景，设为 3 可提升 5%-10% 效果。
- `instruct` 参数建议使用英文撰写，通常带来约 1%-5% 效果提升。
- 各模型支持的图片格式差异较大：新版模型支持 JPEG/PNG/WEBP/BMP/TIFF 等 9 种格式，旧版仅支持 JPG/PNG/BMP，详见 [Multimodal-Embedding API详情](../../raw/model-api-reference/multimodal-vector/multimodal-embedding-api-reference.md)。

## 相关概念

- [[text-embedding]]：纯文本向量化模型
- [[vector-search]]：基于向量的语义搜索
- [[dashscope-sdk]]：百炼 SDK 安装与使用

## 来源文档

- [Multimodal-Embedding API详情](../../raw/model-api-reference/multimodal-vector/multimodal-embedding-api-reference.md)

