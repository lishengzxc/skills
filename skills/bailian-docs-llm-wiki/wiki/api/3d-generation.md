# 3d generation

百炼平台通过 Tripo 模型提供 3D 模型生成能力，支持文生 3D、单图生 3D 和多图生 3D 三种输入模式。API 采用异步调用机制，需先创建任务再轮询获取结果，生成产物为 GLB 格式的 3D 模型文件。

## 支持的模型

根据 [Tripo-3D模型生成](../../raw/model-api-reference/3d-generation/tripo-3d-generation-api-reference.md) 文档，当前平台提供两个模型：

| 模型名称 | 说明 | 最高面数 | 对应 Tripo 官方版本 |
|---------|------|---------|-------------------|
| `Tripo/Tripo-H3.1` | 高精度 3D 模型生成 | 200 万面 | v3.1-20260211 |
| `Tripo/Tripo-P1.0` | 专业 3D 模型生成，速度更快 | 2 万面 | P1-20260311 |

## 功能模式

- **文生 3D**：通过 `[[prompt|prompt]]` 文本描述生成 3D 模型
- **单图生 3D**：通过 `image` 单张图片 URL 生成 3D 模型
- **多图生 3D**：通过 `images` 数组（固定 4 个位置：前、左、后、右）传入 2~4 张图片生成 3D 模型，不需要的视角传入空对象 `{}`

> `[[prompt|prompt]]`、`image`、`images` 三者互斥，同时传入多个将报错。

## 关键参数

### 输入参数（input）

| 参数 | 类型 | 说明 |
|-----|------|------|
| `[[prompt|prompt]]` | string | 文本提示词，最大 1024 字符，支持多语言 |
| `image` | string | 单张图片 URL（JPEG/PNG，分辨率 20~6000px，≤20MB） |
| `images` | array[object] | 多图数组，固定长度 4，每项含 `type` 和 `file_token` |

### 生成参数（parameters）

| 参数 | 类型 | 默认值 | 说明 |
|-----|------|-------|------|
| `texture_quality` | string | `standard` | 贴图质量：`standard`（标清）/ `detailed`（高清） |
| `geometry_quality` | string | `standard` | 几何精度（仅 Tripo-H3.1）：`standard`（150 万面）/ `ultra`（200 万面） |
| `pbr` | boolean | `true` | 是否生成 PBR 材质，设为 true 会强制启用贴图 |
| `texture` | boolean | `true` | 是否生成贴图。无贴图模型需同时设 `texture` 和 `pbr` 为 false |

## 使用方式

该 API 采用异步调用，流程为 **创建任务 → 轮询获取结果**，详见 [Tripo-3D模型生成](../../raw/model-api-reference/3d-generation/tripo-3d-generation-api-reference.md)。

### 步骤 1：创建任务

```
POST https://dashscope.aliyuncs.com/api/v1/services/aigc/video-generation/3d-generation
```

必需请求头：
- `Content-Type: application/json`
- `Authorization: Bearer $DASHSCOPE_API_KEY`
- `X-DashScope-Async: enable`（**必须设置**，否则报错）

### 步骤 2：轮询查询结果

```
GET https://dashscope.aliyuncs.com/api/v1/tasks/{task_id}
```

- 建议轮询间隔：15 秒
- `task_id` 有效期：24 小时
- 查询接口默认 RPS：20

### 返回结果

任务成功后，`results` 中返回：
- `pbr_model_url`：PBR 材质模型（GLB），`pbr=true` 时返回
- `base_model_url`：无贴图基础模型（GLB），`texture=false` 且 `pbr=false` 时返回
- `rendered_image_url`：3D 模型预览渲染图

> **注意**：模型下载链接有效期仅 **2 小时**，请及时下载保存。

## 限制和注意事项

1. **地域限制**：仅适用于"中国内地（北京）"地域，必须使用该地域的 [[api-key]]。
2. **开通服务**：需在[百炼控制台](https://bailian.console.aliyun.com/cn-beijing/?tab=model#/model-market/all)搜索"Tripo"并开通。
3. **请勿重复创建任务**：创建成功后使用返回的 `task_id` 轮询即可。
4. **任务状态流转**：PENDING → RUNNING → SUCCEEDED / FAILED。
5. **`geometry_quality` 参数仅 `Tripo/Tripo-H3.1` 支持**，`Tripo-P1.0` 不适用。
6. 如需更高频查询或事件通知，可配置 [[async-task-callback]]。
7. 错误处理参见 [[error-code]]。

更多使用细节请参考 [Tripo-3D模型生成](../../raw/model-api-reference/3d-generation/tripo-3d-generation-api-reference.md) 完整 API 文档。

## 来源文档

- [Tripo-3D模型生成](../../raw/model-api-reference/3d-generation/tripo-3d-generation-api-reference.md)

