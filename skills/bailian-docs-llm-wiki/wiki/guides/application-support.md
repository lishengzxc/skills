# application [[support|support]]

百炼平台为开发者提供应用构建、数据管理及合规备案等方面的支持。本页面汇总了应用中心常见问题、数据管理注意事项以及相关服务协议，帮助开发者快速排查问题并了解平台使用规范。

## 应用中心功能支持

### 插件能力

根据[常见问题](../../raw/application-user-guide/application-[[support|support]]/application-faq.md)，平台目前提供以下系统插件：

- Python 代码解释器
- 计算器
- 图片生成
- 夸克搜索
- 生成二维码
- GitHub 搜索

部分插件需申请后才可使用。自定义插件服务当前免费，但配置 [[agent]] API 时涉及 [[prompt|prompt]] 优化、应用调用及测试窗测试会产生费用。

### Agent 与 Assistant API 的区别

- **Agent**：用户可自行调整插件模型、基于上下文理解进行开发。
- **Assistant API**：提供各类封装能力，方便调优。自定义 API 插件遵循协议传给大模型进行理解，函数参数会被学习并返回完整结果。

> **注意**：自定义插件调用时不支持自定义 header 透传，仅支持 `authorization`。

## 关键参数与使用方式

### [[streaming|流式输出]]配置

如需增量式回复（而非全量），设置以下参数：

```python
stream=True              # [[streaming|流式输出]]
incremental_output=True  # 增量式[[streaming|流式输出]]
```

### RAG 检索机制

- 检索顺序为**并行**：根据每个 [[knowledge-base]] 的用户配置并行检索，再按得分选取 TopN。
- 若模型回复不准确，可点击回复下方的问题反馈按钮，或复制 `RequestId` 通过阿里云工单反馈。

## 数据管理限制

| 限制项 | 说明 |
|--------|------|
| 文件格式 | 支持 pdf/doc/docx，PDF 文件后缀必须为小写 `pdf`（错误码 140010） |
| 文档数量上限 | 每个业务空间最多 10 万个文档，超出需提交工单申请扩容 |
| 上传接口 MD5 参数 | 用于验证上传文件的完整性 |
| 结构化数据导入 | 表格中出现空行后，后续数据不会被识别；首行为空则视为空文件 |

## 合规备案

接入通义千问大模型并上架应用市场或小程序平台时：

1. 参考 [应用合规备案](https://help.aliyun.com/zh/model-studio/compliance-and-launch-filing-guide-for-ai-apps-powered-by-the-tongyi-model) 完成备案流程。
2. 通过 [提交工单](https://smartservice.console.aliyun.com/service/create-ticket) 申请通义千问系列模型的合作协议。

## 相关协议

根据[相关协议](../../raw/application-user-guide/application-[[support|support]]/application-related-agreements.md)文档，使用百炼平台需遵守以下条款：

- [阿里云百炼服务协议](https://terms.alicdn.com/legal-agreement/terms/common_platform_service/20230728213935489/20230728213935489.html)
- [阿里云百炼服务特别说明](https://help.aliyun.com/zh/model-studio/bailian-service-notes)
- [开源模型协议条款说明](https://help.aliyun.com/zh/model-studio/open-source-model-terms)

## 注意事项

- AI 输出中的 `**xxx**` 是 Markdown 加粗标识，前端渲染时需解析 MD 语法做对应展示。
- 如遇到应用回复质量问题，建议先检查 [[prompt]] 配置和知识库内容质量，再通过工单反馈。

## 来源文档

- [常见问题](../../raw/application-user-guide/application-support/application-faq.md)
- [相关协议](../../raw/application-user-guide/application-support/application-related-agreements.md)

