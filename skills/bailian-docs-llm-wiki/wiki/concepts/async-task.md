# 异步任务调用

异步任务调用是百炼平台中用于处理耗时较长的模型推理任务的调用模式。开发者先提交请求创建任务并获取 `task_id`，再通过轮询或事件通知获取任务结果，适用于视频生成、3D 模型生成、批量文本向量化、图像生成等场景。

---

## 适用场景

百炼平台中以下类型的 API 采用异步任务调用：

| 场景 | 典型模型 | 说明 |
|------|----------|------|
| [[video-generation-api]] | `wan2.7-t2v-*`、`wan2.7-i2v-*`、数字人等 | 所有视频生成 API 均为异步调用 |
| [[3d-generation]] | `Tripo/Tripo-H3.1`、`Tripo/Tripo-P1.0` | 文生3D、图生3D |
| [[image-generation]] | `wan2.5-i2i-preview`、`wanx2.1-imageedit` 等 | 部分图像生成/编辑模型 |
| [[general-text-embedding]]（批处理） | `text-embedding-async-v2` 等 | 大规模文本向量化（单次最多 10 万行） |

---

## 调用流程

所有异步任务遵循统一的两步流程：

### 步骤 1：创建任务

向对应的模型 API 端点发送 POST 请求，**必须**在请求头中设置：

```
X-DashScope-Async: enable
```

请求成功后返回 `task_id`。

```bash
curl -X POST '<模型对应的API端点>' \
  -H 'X-DashScope-Async: enable' \
  -H 'Authorization: Bearer $DASHSCOPE_API_KEY' \
  -H 'Content-Type: application/json' \
  -d '{
    "model": "<模型名称>",
    "input": { ... },
    "parameters": { ... }
  }'
```

> **重要**：请勿对同一请求重复创建任务。创建成功后使用返回的 `task_id` 轮询即可。

### 步骤 2：查询任务结果

```bash
GET https://dashscope.aliyuncs.com/api/v1/tasks/{task_id}
```

建议轮询间隔 **15 秒**。当 `task_status` 为 `SUCCEEDED` 时，从 `output` 中获取生成结果。

---

## 任务状态

| 状态值 | 含义 |
|--------|------|
| `PENDING` | 排队中 |
| `RUNNING` | 处理中 |
| `SUCCEEDED` | 成功 |
| `FAILED` | 失败 |
| `CANCELED` | 已取消 |
| `UNKNOWN` | 任务不存在或已过期 |

---

## 任务管理 API

百炼提供三个通用异步任务管理接口，限流 **20 QPS**（按主账号维度）：

| 接口 | 方法 | 说明 |
|------|------|------|
| 查询任务结果 | `GET /api/v1/tasks/{task_id}` | 根据 `task_id` 查询状态和结果 |
| 批量查询任务状态 | `GET /api/v1/tasks/` | 支持按时间、模型、状态等条件筛选 |
| 取消任务 | `POST /api/v1/tasks/{task_id}/cancel` | 仅支持取消 `PENDING` 状态的任务 |

---

## 任务完成通知（替代轮询）

相比频繁轮询，通过 [[eventbridge]] 事件总线接收完成通知更为高效。百炼支持两种通知方式：

| 方式 | 特点 | 适用场景 |
|------|------|----------|
| **HTTP 回调 URL** | 需公网或 VPC 可访问的 POST 接口 | 通用场景 |
| **RocketMQ** | 消息无丢失，支持失败重试 | 对可靠性要求高的场景 |

**事件关键字段**：

- 事件源：`acs.dashscope`
- 事件类型：`dashscope:System:AsyncTaskFinish`
- `data.task_id`：已完成的任务 ID
- `data.task_status`：任务最终状态（`SUCCEEDED` 或 `FAILED`）

配置方式：在 EventBridge 控制台（北京地域 `default` 总线）中创建事件规则，指定事件源和类型，配置事件目标即可。

---

## 关键限制

| 限制项 | 说明 |
|--------|------|
| `task_id` 有效期 | **24 小时**，过期后查询返回 `UNKNOWN` |
| 结果文件有效期 | 因模型而异，部分生成结果（如 3D 模型下载链接）仅 **2 小时** 有效，请及时下载 |
| 查询限流 | 20 QPS（按主账号维度） |
| 跨账号隔离 | 跨主账号的任务无法查询或取消 |
| 地域一致性 | 模型、Endpoint URL 和 [[api-key]] 必须属于同一地域，跨地域调用将失败 |

---

## 相关主题

- [[video-generation-api]] — 视频生成全系列异步调用
- [[3d-generation]] — 3D 模型生成异步调用
- [[image-generation]] — 图像生成与编辑
- [[general-text-embedding]] — 批处理向量化的异步调用
- [[more-about-models]] — 异步任务管理与完成通知详细说明
- [[error-code]] — 错误码参考

## 关联主题页

- [[video-generation-api|video generation api]] — `../api/video-generation-api.md`
- [[3d-generation|3d generation]] — `../api/3d-generation.md`
- [[image-generation|image generation]] — `../api/image-generation.md`
- [[more-about-models|[[more|more]] about models]] — `../api/[[more|more]]-about-models.md`
- [[general-text-embedding|general text embedding]] — `../api/general-text-embedding.md`


