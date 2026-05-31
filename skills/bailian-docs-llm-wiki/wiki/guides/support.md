# support

阿里云百炼平台提供多渠道的技术支持与服务保障，涵盖常见问题解答、计费说明、API/SDK 使用指导以及相关法律协议。本页面汇总了开发者在使用百炼平台过程中最常遇到的问题和资源入口，帮助快速定位并解决问题。

## 计费与付费方式

百炼平台采用**按分钟级出账、按月结算**的后付费模式，部分模型还支持预付费（详见[节省计划与资源包](https://help.aliyun.com/zh/model-studio/savings-plan-and-resource-package)）。开通服务要求阿里云账户余额不小于 0 元。

关键计费资源：

| 计费项 | 参考文档 |
|--------|----------|
| 模型调用 | [模型调用计费](https://help.aliyun.com/zh/model-studio/model-pricing) |
| 模型部署 | [模型部署计费](https://help.aliyun.com/zh/model-studio/model-training-and-deployment-billing) |
| 模型训练 | [模型训练计费](https://help.aliyun.com/zh/model-studio/model-training-and-deployment-billing) |

扣款明细可在[费用与成本控制台](https://usercenter2.aliyun.com/finance/expense-report/expense-detail)查看，发票在[发票管理](https://usercenter2.aliyun.com/invoice/list/aliyun)页面申请。

> **注意**：万相会员与百炼 API 的计费体系相互独立，万相会员权益不适用于百炼 API 调用。

## API/SDK 使用要点

百炼平台当前支持 Java 和 Python SDK，安装方式详见 [安装SDK](https://help.aliyun.com/zh/model-studio/install-sdk)。API 调用错误码及解决方案请参见 [错误码文档](https://help.aliyun.com/zh/model-studio/error-code)。

根据 [常见问题](../../raw/model-user-guide/support/faq-about-alibaba-cloud-model-studio.md) 中的说明，以下是开发者常遇到的 API 问题：

- **Completion API 报错 100004**：通常是缺少必填参数或参数格式不正确，需检查 `AppId`、`Prompt` 等字段。
- **Assistant API 不支持多函数串行调用**：如需调用两个本地函数，需创建两个 Assistant 分别处理。
- **Assistant API 暂不支持 memory 配置**。
- **`doc_reference_type` 参数**：仅在旧版应用中生效，新版应用需在操作页面开启"展示答案来源"开关。

## 模型选型与训练

### 千问系列模型对比

| 模型 | 特点 | 适用场景 |
|------|------|----------|
| [[qwen-turbo]] | 速度快、费用低 | 对响应速度和部署便捷性有较高要求 |
| [[qwen-max]] | 顶级性能、全面知识 | 对精度和复杂任务能力有严格要求 |

千问系列支持 **14 种语言**：中文、英文、阿拉伯语、西班牙语、法语、葡萄牙语、德语、意大利语、俄语、日语、韩语、越南语、泰语、印度尼西亚语。

### 模型训练注意事项

- 已支持图片训练（[[qwen-vl-plus]] 支持训练微调）。
- 本地训练的模型**不支持上传**，"自定义模型"仅指平台上已训练完成的模型用于二次训练。
- 训练完成的开源模型**不支持导出**。
- 千问基模型升级后，企业模型不一定需要重新训练，建议用效果评估工具自行判断。

### 训练数据最佳实践

据 [常见问题](../../raw/model-user-guide/support/faq-about-alibaba-cloud-model-studio.md) 中的总结，高质量 SFT 数据应满足：

1. **任务定义清晰**：避免同一 [[prompt|prompt]] 对应模棱两可的答案。
2. **数据质量高**：答案准确、简洁，避免冗余。
3. **数据多样性**：同一语义用多种 [[prompt|prompt]] 表达，避免模型只学会单一模式。

超参数（如循环次数）无固定规律，需通过实验确定。评估模型效果应以**人工评估**为准，不应仅依赖 loss 值判断过拟合。

## 降低模型幻觉

模型幻觉指 LLM 生成看似合理但无事实依据的内容。可通过以下方式缓解：

1. **选择更强模型**：Max > Plus > Turbo。
2. **[[prompt-engineering]]**：添加约束指令，如"信息不足请说'我不知道'"。
3. **[[rag]]（检索增强生成）**：限制模型在检索到的知识范围内回答。
4. **插件 / [[mcp]]**：将数值计算等任务委托给外部工具。
5. **参数调优**：降低 `temperature`、`top_k`、`top_p` 等随机性参数。
6. **后处理验证**：用 AI 二次校验回复内容（会增加成本和延迟）。

## 模型限流

生成速度不固定，受服务负载和请求并发影响。触发限流后的等待时间取决于具体 RPS/RPM 限额。例如 120 RPM 限额下，0.2 秒内连发 2 次后第 3 次将被限流，需等待约 0.8 秒。

## 数据安全与隐私

- 阿里云**不会**将用户数据用于模型训练。
- 传输数据经 AES-256 加密。
- 根据法律法规要求，百炼会存储模型与应用调用时产生的数据。
- 可通过主账号为不同子账号分配不同 [[业务空间]] 权限实现数据隔离。
- 控制台最多展示 **100 条历史对话记录**，不设时间限制；未登录状态和推理报错的对话不会保存。

## 服务开通与关闭

- **开通**：使用阿里云主账号访问百炼控制台，阅读并同意协议后自动开通（需已完成实名认证）。
- **关闭**：开通后暂不支持关闭，但可通过删除 API-Key 避免后续调用和计费。

## 相关协议

根据 [相关协议](../../raw/model-user-guide/support/related-agreements.md)，使用百炼平台需遵守以下协议：

- [阿里云百炼服务协议](https://terms.alicdn.com/legal-agreement/terms/common_platform_service/20230728213935489/20230728213935489.html)
- [阿里云百炼模型推理服务等级协议（SLA）](https://terms.alicdn.com/legal-agreement/terms/b_end_product_protocol/20250923215800868/20250923215800868.html)
- [阿里云百炼服务特别说明](https://help.aliyun.com/zh/model-studio/bailian-service-notes)
- [开源模型协议条款说明](https://help.aliyun.com/zh/model-studio/open-source-model-terms)
- [三方模型服务协议和使用条款清单](https://terms.alicdn.com/legal-agreement/terms/common_product_agreement/20260207131114217/20260207131114217.html)

## 联系方式

| 场景 | 渠道 |
|------|------|
| 业务合作 / 售前咨询 | 热线 4008013260 或 [官网售前咨询](https://smartservice.console.aliyun.com/service/pre-sales-chat) |
| 产品使用问题 / 售后 | [官网售后服务](https://smartservice.console.aliyun.com/service/robot-chat) |
| 合作协议申请 | 提交 [阿里云工单](https://smartservice.console.aliyun.com/service/create-ticket) |

## 来源文档

- [常见问题](../../raw/model-user-guide/support/faq-about-alibaba-cloud-model-studio.md)
- [相关协议](../../raw/model-user-guide/support/related-agreements.md)

