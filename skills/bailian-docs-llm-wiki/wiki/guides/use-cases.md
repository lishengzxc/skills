# use cases

百炼平台提供了丰富的使用场景和最佳实践，涵盖 Prompt 工程、多模态内容生成、RAG 应用构建、模型调优部署、第三方模型集成以及生产环境下的性能优化等方面。本文汇总了各类使用场景的核心要点，帮助开发者快速定位并参考适合自身业务的实践方案。

---

## Prompt 工程

### 文生文 Prompt

设计清晰、具体、无歧义的 Prompt 是发挥 LLM 能力的关键。[文生文Prompt指南](../../raw/model-user-guide/use-cases/[[prompt|prompt]]-engineering-guide.md) 推荐使用 **Prompt 框架**来系统化地组织输入，框架包含六个要素：

- **背景**：任务相关的上下文信息
- **目的**：期望模型完成的具体任务
- **风格**：输出的写作风格（如某专家、某流派）
- **语气**：正式、诙谐、温馨等
- **受众**：目标读者群体
- **输出**：期望的输出格式（列表、JSON、报告等）

百炼控制台还提供 Prompt 一键优化工具，可对输入进行自动扩写和细节补充（会消耗 [[token]] 并按推理费用计费）。

### 文生图 Prompt

[文生图Prompt指南](../../raw/model-user-guide/use-cases/text-to-image-[[prompt|prompt]].md) 适用于万相-文生图 V1/V2，提供两级提示词公式：

| 公式级别 | 结构 | 适用用户 |
|---------|------|---------|
| 基础公式 | 主体 + 场景 + 风格 | 初次尝试 AI 创作的新用户 |
| 进阶公式 | 主体描述 + 场景描述 + 定义风格 + 镜头语言 + 氛围词 + 细节修饰 | 有一定经验的用户 |

关键参数：

- `[[prompt|prompt]]`：正向提示词，支持中英文
- `negative_prompt`：反向提示词，描述不希望出现的内容
- `prompt_extend`（仅 V2）：是否开启大模型智能改写，默认 `true`

文档还包含景别（特写/近景/中景/远景）、视角（平视/俯视/仰视）等提示词词典供参考。

### 文生视频 / 图生视频 Prompt

[文生视频/图生视频Prompt指南](../../raw/model-user-guide/use-cases/text-to-video-prompt.md) 适用于万相系列视频模型，提供多种提示词公式：

- **基础公式**：主体 + 场景 + 运动
- **进阶公式**：主体描述 + 场景描述 + 运动描述 + 美学控制 + 风格化
- **图生视频公式**：运动 + 运镜（图像已确定主体和风格）
- **声音公式**（wan2.5/2.6/2.7）：增加人声/音效/背景音乐描述
- **多镜头公式**（wan2.6/2.7）：总体描述 + 镜头序号 + 时间戳 + 分镜内容
- **参考生视频公式**（wan2.6/2.7）：支持通过"图n"/"视频n"指代参考文件

> **注意**：wan2.7 模型不再支持 `shot_type` 参数指定单/多镜头，改为由模型结合提示词自动判断。如需控制一镜到底，需在提示词中写明"生成单镜头"。

### Vidu 视频生成 Prompt

Vidu 模型的提示词公式为 **主体/场景 + 场景描述 + 环境描述 + 艺术风格/媒介**。关键特性包括：

- 通过"大动态"、"小动态"等关键词控制运动幅度
- 支持特殊拍摄手法：延时摄影、微距、第一人称、航拍等
- 支持参考生视频的多主体一致性保持
- 提供 AI 漫剧提示词结构：风格/景别/机位/构图/运镜 + 画面描述 + 图片强调

---

## RAG 应用

百炼支持通过 LlamaIndex 构建 [[rag]] 应用。核心流程：

1. **文件解析**：使用 `DashScopeParse` 解析 `.doc`/`.docx`/`.pdf` 文件（单文件 ≤100MB，≤1000 页）
2. **创建知识库**：通过 `DashScopeCloudIndex.from_documents()` 创建
3. **检索与问答**：通过 `index.as_retriever()` 获取 retriever，或通过 `index.as_query_engine(llm=dashscope_llm)` 获取 query engine

前提条件：Python 3.8~3.12，需安装 `llama-index-core`、`llama-index-llms-dashscope`、`llama-index-indices-managed-dashscope`。

---

## 模型调优与部署

自定义模型的创建涉及三个主要步骤：

1. **模型调优**：基于预置模型，使用 Prompt-Completion 格式的训练数据进行微调（建议至少 500 条数据）
2. **模型部署**：将调优后的模型部署到独占实例（完成调优的模型**必须部署后才能调用和评测**）
3. **模型评测**：使用评测数据验证模型效果

训练数据准备要点：
- 数据来源多样化，质量控制优先
- 支持训练集和评测集两种数据类型
- 平台提供数据清洗和数据增强工具

---

## 第三方模型集成

百炼平台支持多个第三方模型供应商，统一通过 [[openai-compatible-api|OpenAI 兼容接口]]或 DashScope SDK 调用。所有模型均使用 `https://dashscope.aliyuncs.com/compatible-mode/v1` 作为 base URL。

### 支持的模型一览

| 模型系列 | 供应商 | 代表模型 | 思考模式 | 地域限制 |
|---------|--------|---------|---------|---------|
| DeepSeek | 百炼 | deepseek-v4-pro | `enable_thinking` | - |
| DeepSeek | 硅基流动 | siliconflow/deepseek-v3.2 | `enable_thinking` | 北京 |
| DeepSeek | 快手万擎 | vanchin/deepseek-v4-pro | `enable_thinking` | 北京 |
| Kimi | 月之暗面 | kimi/kimi-k2.6 | `enable_thinking`（默认开启） | 北京 |
| Kimi | 百炼 | kimi-k2-thinking | 默认开启 | 北京/弗吉尼亚/法兰克福 |
| GLM | 百炼 | glm-5.1 | `enable_thinking` | - |
| GLM | 智谱 | ZHIPU/GLM-5.1 | `enable_thinking` | 北京 |
| MiniMax | 百炼 | MiniMax-M2.5 | 默认开启 | 中国内地 |
| MiniMax | 稀宇科技 | MiniMax/MiniMax-M2.7 | 默认开启 | 中国内地 |
| Step | 阶跃星辰 | stepfun/step-3.7-flash | `enable_thinking`（默认关闭） | 北京 |
| MiMo | 小米 | xiaomi/mimo-v2.5-pro | 默认开启 | 北京 |

> **注意**：同一模型系列的不同供应商版本在功能上可能有差异。例如硅基流动版 DeepSeek 支持更长上下文，而百炼版限流更宽松且支持联网搜索和上下文缓存；智谱版 GLM 支持更长回复长度，百炼版提供免费额度和阶梯计费。

### 通用调用模式

`enable_thinking` 是各模型共有的非标准参数：
- **Python SDK**：通过 `extra_body={"enable_thinking": True}` 传入
- **Node.js SDK**：作为顶层参数直接传入
- **HTTP**：直接在 JSON body 中添加

思考模式开启后，模型输出包含 `reasoning_content`（思考过程）和 `content`（最终回复）两部分。

---

## 多模态内容生成

### 文档转视频

借助 LLM 和多模态技术，可以将文档自动转换为包含图文、语音、字幕的完整视频。流程为：

1. 文档切片（LLM 总结标题并分段）
2. 生成演示文稿（整合标题、正文、图片）
3. 生成讲解语音与字幕（多模态模型）
4. 合成视频（FFmpeg + Marp）

依赖工具：FFmpeg、Marp CLI、Python 3.x 及相关库。

---

## 生产环境最佳实践

### 限流应对

百炼 API 按主账号维度、模型独立计算限流，存在三种限流规则：

| 规则 | 说明 |
|------|------|
| RPM / TPM | 每分钟最大请求数 / Token 用量 |
| RPS / TPS | 每秒最大请求数 / Token 用量 |
| Traffic Burst | 短时间内请求量激增时触发 |

应对方案（按改动成本递增）：

1. **服务端排队等待**（推荐首选）：添加 `X-DashScope-Wait-Timeout` 请求头，仅适用于 Traffic Burst
2. **提升限流额度**：在控制台直接提升临时额度，提交后立即生效
3. **PTU 预留算力**：独立专享算力，保障 SLA
4. **Batch API**：离线批处理任务
5. **客户端流控**：令牌桶、信号量、平滑限速器、自适应拥塞控制
6. **架构兜底**：模型降级 Fallback、MQ 削峰填谷

### 显式缓存

通过在请求中添加 `cache_control` 标记实现确定性缓存命中，适用于：

- 高频复用相同 Prompt 的场景（首次写入仅产生标准价格 25% 额外开销，命中后节省 90%）
- 工业级 Agent 的长上下文管理
- 需要稳定命中缓存的业务场景

已原生支持显式缓存的工具（通过 Anthropic 协议接入）：

| 工具 | 缓存行为 |
|------|---------|
| Claude Code | 默认对 system/env/最近 user message 挂 `cache_control` |
| Open Code | 默认对 system 与最近非 system 消息注入 |
| OpenClaw | 默认对系统提示词与最近用户消息注入 |
| Hermes | 通过 `hermes config set` 配置 |

> **注意**：Claude Code 默认在 system prompt 中包含动态信息（目录、日期、git 状态等），可能降低跨会话命中率。启动时添加 `--exclude-dynamic-system-prompt-sections` 可优化。

---

## 限制和注意事项

- 第三方模型大多仅适用于特定地域（通常为华北2北京），调用前需确认地域并使用对应的 [[api-key]]
- `enable_thinking` 是非 OpenAI 标准参数，不同 SDK 的传入方式不同
- DashScopeParse 仅支持 `.doc`/`.docx`/`.pdf`，单文件 ≤100MB、≤1000 页
- 自定义模型必须**先部署再评测**，部署到独占实例会产生持续费用
- 显式缓存的排队等待功能仅适用于 Traffic Burst 限流，不适用于 RPM/TPM 绝对值限流
- Prompt 优化工具会消耗

## 来源文档

- [文生文Prompt指南](../../raw/model-user-guide/use-cases/prompt-engineering-guide.md)
- [文生图Prompt指南](../../raw/model-user-guide/use-cases/text-to-image-prompt.md)
- [基于LlamaIndex构建RAG应用](../../raw/model-user-guide/use-cases/build-rag-applications-based-on-llamaindex.md)
- [文生视频/图生视频Prompt指南](../../raw/model-user-guide/use-cases/text-to-video-prompt.md)
- [自定义模型调优、部署与评测](../../raw/model-user-guide/use-cases/model-training-best-practices.md)
- [显式缓存最佳实践](../../raw/model-user-guide/use-cases/explicit-cache-best-practice.md)
- [限流应对最佳实践 ](../../raw/model-user-guide/use-cases/rate-limiting-best-practices.md)
- [借助大模型将文档转换为视频](../../raw/model-user-guide/use-cases/use-llm-to-convert-document-to-video.md)
- [DeepSeek大语言模型](../../raw/model-user-guide/use-cases/third-party-model-integration-tutorial/deepseek-api.md)
- [DeepSeek-硅基流动](../../raw/model-user-guide/use-cases/third-party-model-integration-tutorial/siliconflow-deepseek-api.md)
- [DeepSeek](../../raw/model-user-guide/use-cases/third-party-model-integration-tutorial/deepseek-api-by-vanchin.md)
- [Kimi-月之暗面](../../raw/model-user-guide/use-cases/third-party-model-integration-tutorial/kimi-api-by-moonshot-ai.md)
- [Kimi](../../raw/model-user-guide/use-cases/third-party-model-integration-tutorial/kimi-api.md)
- [GLM](../../raw/model-user-guide/use-cases/third-party-model-integration-tutorial/glm.md)
- [GLM-智谱](../../raw/model-user-guide/use-cases/third-party-model-integration-tutorial/glm-zhipu.md)
- [MiniMax](../../raw/model-user-guide/use-cases/third-party-model-integration-tutorial/minimax-api.md)
- [MiniMax](../../raw/model-user-guide/use-cases/third-party-model-integration-tutorial/minimax-api-by-minimax.md)
- [Vidu视频生成Prompt指南](../../raw/model-user-guide/use-cases/third-party-model-integration-tutorial/vidu-video-generation-prompt-guide.md)
- [Stepfun-阶跃星辰](../../raw/model-user-guide/use-cases/third-party-model-integration-tutorial/stepfun.md)
- [MiMo-小米](../../raw/model-user-guide/use-cases/third-party-model-integration-tutorial/mimo.md)

