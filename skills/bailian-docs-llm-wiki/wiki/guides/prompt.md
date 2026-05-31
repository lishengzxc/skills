# prompt

Prompt 是大语言模型应用的核心输入，直接决定模型输出的质量与稳定性。阿里云百炼平台围绕 Prompt 提供了模板管理、自动优化、反馈优化等一系列工具，帮助开发者高效构建、迭代和复用高质量的 Prompt。本文汇总了百炼平台中与 Prompt 相关的核心功能、使用方式及注意事项。

> **注意**：以下 Prompt 模板相关功能仅适用于中国大陆版（北京地域）。

## Prompt 模板

Prompt 模板将固定结构与动态变量分离，实现 Prompt 的统一管理和复用。模板分为**预置模板**和**自定义模板**两类，详见 [Prompt模板概述](../../raw/application-user-guide/prompt/prompt-template.md)。

### 预置模板

由阿里云百炼提供，覆盖营销文案、摘要抽取、文案润色等通用场景，已经过优化，效果稳定。预置模板**不支持修改**，但可通过「复制模板」创建自定义副本后再编辑。

### 自定义模板

支持通过控制台或 API（`CreatePromptTemplate`）创建。控制台提供两种输入模式：

| 输入模式 | 适用场景 |
|---|---|
| **自定义创建** | 已有现成 Prompt，快速模板化或简单优化 |
| **基于 Prompt 工程创建** | 构建复杂任务 Prompt，借助 ICIO / CRISPE / RASCEF 等框架进行结构化设计 |

自定义模板还支持**图片生成**场景，可分别定义正向和负向提示词来控制画面内容。更多创建细节参见 [自定义Prompt模板](../../raw/application-user-guide/prompt/prompt-custom-template.md)。

### 模板工作流程

1. **创建模板**：通过控制台或 API 创建并保存，获取模板 ID。
2. **获取模板**：通过 `GetPromptTemplate` API 拉取模板内容。
3. **生成 Prompt**：将业务数据填入模板变量，生成最终 Prompt。
4. **调用模型**：将生成的 Prompt 发送给目标模型获取结果。

使用 `GetPromptTemplate` 接口相比直接在代码中拼接字符串的优势在于：逻辑与内容分离（无需重新部署即可更新 Prompt）、集中管理与团队协作、版本一致性保障。

## Prompt 优化功能

### 自动优化

当手动编写高质量 Prompt 成本较高时，可使用**Prompt 自动优化**功能。该功能利用大模型对原始 Prompt 进行分析和重写，优化策略包括：

- **结构重组**：调整整体结构使其更符合逻辑
- **角色扮演引导**：设定明确的专家角色
- **指令增强**：将模糊指令具体化、步骤化
- **安全与边界注入**：增加输出格式、内容限制等约束

操作路径：**应用开发 > 组件管理 > 提示词 > 自动优化**。优化后的 Prompt 可复制使用或保存为模板。该功能**不计费**，详见 [Prompt自动优化](../../raw/application-user-guide/prompt/optimize-prompt.md)。

### 基于样例的反馈优化

相比自动优化仅对 Prompt 文本本身进行改写，**反馈优化**结合用户提供的输入输出样例进行多轮评估、反思和迭代，使优化后的 Prompt 在实际业务场景中表现更好。其工作流程为：

1. 选择**推理模型**（推荐 [[qwen-max]]）
2. 输入初始 Prompt（描述任务目标即可）
3. 上传**样例数据**（建议 5~10 条，每种场景至少 1 条）
4. 上传**评测数据**（建议至少 20 条，越多效果越好）
5. 系统自动执行多轮优化

优化后的 Prompt 可保存为模板或直接创建 [[agent-application]]。详见 [基于大模型输入输出样例的Prompt自动优化](../../raw/application-user-guide/prompt/prompt-feedback-optimization.md)。

## Prompt 工程框架

百炼内置了三种 Prompt 工程框架，适用于不同复杂度的任务：

| 框架 | 组成要素 | 适用场景 |
|---|---|---|
| **ICIO** | 指令、背景信息、补充数据、输出格式 | 简单明确的任务：数据分析、内容生成、文本摘要 |
| **CRISPE** | 角色与能力、背景信息、任务、输出风格、输出范围 | 需要角色扮演的交互：智能客服、创意写作、面试模拟 |
| **RASCEF** | 角色、行动、步骤、上下文、示例、格式 | 多步骤复杂业务流程：项目规划、战略分析、流程设计 |

可在创建自定义模板时选择「基于 Prompt 工程创建」模式使用这些框架，更多框架说明和优化前后对比示例参见 [自定义Prompt模板](../../raw/application-user-guide/prompt/prompt-custom-template.md)。

## Prompt 样例库

> **注意**：Prompt 样例库功能**已不再维护**，推荐将样例库数据迁移到 [[rag]] 表格库中。

样例库采用少样本学习（Few-shot learning）思路，从预定义的高质量问答对中检索相关样例注入上下文，引导模型生成更准确、风格更一致的回复。适用于智能客服、特定领域知识问答、格式化内容生成等场景。

关键限制：

| 限制项 | 值 |
|---|---|
| 单个样例库最大样例数 | 300 条 |
| 单个应用最多关联样例库数 | 5 个 |
| 单次请求最大召回片段数 | 10 个（可配置，默认 5） |
| 批量导入文件大小 | ≤ 20MB（Excel），单次最多 100 条 |

样例库本身不收取费用，但召回样例会增加输入 Token 消耗：`总输入 Token ≈ 用户查询 Token + 召回样例总 Token + 系统指令 Token`。

## 使用方式

Prompt 模板支持通过以下方式使用：

- **控制台**：在模板卡片上点击「创建应用」，模板内容自动填充到 [[agent-application]] 的提示词中。
- **API**：通过 `GetPromptTemplate` 接口传入 `workspaceId` 和 `promptTemplateId` 获取模板内容。
- **SDK**：在 API 调试界面获取 SDK 示例代码，设置 `accessKeyId` 和 `accessKeySecret` 后运行。

获取 `workspaceId` 和 `promptTemplateId` 的方法参见平台文档中的「获取 APP ID 和 Workspace ID」。

## 限制与注意事项

- Prompt 自动优化可能因输入过长（超出 Token 限制）、触发内容审核策略或网络问题而失败。
- 反馈优化的效果依赖评测数据的质量和数量，建议至少准备 20 条评测数据。
- 样例库超过 300 条时，建议按业务主题拆分为多个独立库（如"产品功能库""售后策略库"），避免检索延迟。
- 提交用于优化的 Prompt 数据不会被存储或用于模型训练。
- 调用失败时可参考平台提供的错误码文档排查问题。

## 来源文档

- [Prompt模板概述](../../raw/application-user-guide/prompt/prompt-template.md)
- [使用Prompt样例库优化模型输出](../../raw/application-user-guide/prompt/prompt-sample-optimization.md)
- [自定义Prompt模板](../../raw/application-user-guide/prompt/prompt-custom-template.md)
- [Prompt自动优化](../../raw/application-user-guide/prompt/optimize-prompt.md)
- [基于大模型输入输出样例的Prompt自动优化](../../raw/application-user-guide/prompt/prompt-feedback-optimization.md)

