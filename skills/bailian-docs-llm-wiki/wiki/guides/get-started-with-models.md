# get started with models

阿里云百炼是一站式大模型开发与应用平台，提供兼容 OpenAI 接口规范的 API，开发者只需几行代码即可调用千问（Qwen）全系列模型及 DeepSeek、Kimi、GLM 等第三方模型。本文汇总了模型调用前需要了解的核心内容，包括支持的模型、接入方式、地域选择和限流规则。

## 支持的模型与能力

百炼提供开箱即用的模型服务，无需自行部署或运维。根据 [选择模型](../../raw/model-user-guide/get-started-with-models/models.md)，当前覆盖以下模态：

| 能力类别 | 代表模型 | 说明 |
|---|---|---|
| 文本生成 | `qwen3.7-max`、`qwen3.6-plus`、`qwen3.6-flash` | 千问旗舰系列，从效果最优到高性价比依次排列 |
| 文本生成（三方） | `deepseek-v4-pro`、`kimi-k2.6`、`glm-5.1` 等 | API 格式与千问一致，DeepSeek 仅支持北京地域 |
| 视觉理解 | `qwen3.6-plus`、`qwen3.5-omni-plus` | 分析图片/视频，返回文本或结构化结果 |
| 图像/视频生成 | `wan2.7-image-pro`、`happyhorse-1.0-t2v` | 文生图、图生视频、视频编辑等 |
| 语音合成与识别 | `cosyvoice-v3.5-plus`、`fun-asr-realtime` | TTS、ASR 及端到端语音对话 |
| 向量与重排序 | `text-embedding-v4`、`qwen3-rerank` | 文本向量化及检索精度提升 |
| 全模态 | `qwen3.5-omni-plus-realtime` | 融合文本、图像、音频、视频的理解与生成 |

千问旗舰模型的定位：
- **Max**：效果最强，适合复杂多步骤任务
- **Plus**：效果、速度、成本均衡，多数场景推荐
- **Flash**：低延迟高性价比，适合简单任务快速响应

完整模型列表请前往 [[models]] 或[模型广场](https://bailian.console.aliyun.com/cn-beijing?tab=model#/model-market/all)查看。

## 快速开始：发起第一个 API 请求

详细步骤参见 [首次调用千问API](../../raw/model-user-guide/get-started-with-models/first-api-call-to-qwen.md)，核心流程如下：

### 1. 获取 API Key

1. 使用阿里云主账号前往[百炼控制台](https://bailian.console.aliyun.com/?tab=model#/model-market)开通服务
2. 进入 [API Key 页面](https://bailian.console.aliyun.com/?tab=model#/api-key) 创建密钥

### 2. 配置环境变量

将 API Key 设置为环境变量 `DASHSCOPE_API_KEY`，避免在代码中硬编码：

```bash
# Linux/macOS
export DASHSCOPE_API_KEY="YOUR_DASHSCOPE_API_KEY"

# Windows CMD
set DASHSCOPE_API_KEY=YOUR_DASHSCOPE_API_KEY

# Windows PowerShell
$env:DASHSCOPE_API_KEY = "YOUR_DASHSCOPE_API_KEY"
```

永久生效请写入 `~/.bashrc`（Linux）、`~/.zshrc`（macOS Zsh）或通过 `setx` / 系统属性设置（Windows）。

### 3. 调用模型

百炼兼容 OpenAI 接口，只需调整 `api_key`、`base_url` 和 `model` 即可迁移现有代码。

**Python（OpenAI SDK）**

```python
import os
from openai import OpenAI

client = OpenAI(
    api_key=os.getenv("DASHSCOPE_API_KEY"),
    base_url="https://dashscope.aliyuncs.com/compatible-mode/v1",
)
completion = client.chat.completions.create(
    model="qwen-plus",
    messages=[
        {'role': 'system', 'content': 'You are a helpful assistant.'},
        {'role': 'user', 'content': '你是谁？'}
    ]
)
print(completion.choices[0].message.content)
```

**curl**

```bash
curl -X POST https://dashscope.aliyuncs.com/compatible-mode/v1/chat/completions \
  -H "Authorization: Bearer $DASHSCOPE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "qwen-plus",
    "messages": [{"role": "user", "content": "你是谁？"}]
  }'
```

除 OpenAI SDK 外，也支持 DashScope SDK 和 Anthropic 兼容接口。安装方式：

```bash
pip install -U openai      # OpenAI Python SDK
pip install -U dashscope   # DashScope Python SDK
```

> **注意**：文档 1 的示例代码使用模型名 `qwen3.6-plus`，文档 2 使用 `qwen-plus`。两者均为有效模型，`qwen-plus` 会自动指向当前稳定版本，带版本号的名称（如 `qwen3.6-plus`）则指向特定版本。建议根据是否需要锁定版本来选择。

## 地域与服务部署范围

根据 [选择地域和服务部署范围](../../raw/model-user-guide/get-started-with-models/regions.md)，调用前需确定两个维度：

- **地域**：决定接入点（Base URL）和数据存储位置，就近选择可降低延迟
- **服务部署范围**：决定推理执行位置，有数据合规需求时选择特定地理边界

### 各地域 Base URL（OpenAI 兼容）

| 地域 | Base URL |
|---|---|
| 华北2（北京） | `https://dashscope.aliyuncs.com/compatible-mode/v1` |
| 新加坡 | `https://dashscope-intl.aliyuncs.com/compatible-mode/v1` |
| 美国（弗吉尼亚） | `https://dashscope-us.aliyuncs.com/compatible-mode/v1` |
| 德国（法兰克福） | `https://{WorkspaceId}.eu-central-1.maas.aliyuncs.com/compatible-mode/v1` |

**关键约束**：
- 不同地域的 **API Key 不通用**，Base URL 也不能跨地域混用
- 支持的模型、功能和价格因地域而异
- 法兰克福需先创建业务空间（Workspace）并获取 WorkspaceId
- 弗吉尼亚使用 `-us` 后缀模型名（如 `qwen-plus-us`）可限定美国境内推理

### 地域功能差异

| 功能 | 北京 | 新加坡 | 弗吉尼亚 | 法兰克福 |
|---|---|---|---|---|
| 批量推理 | ✅ | ✅ | ❌ | ❌ |
| 模型调优 | ✅ | ❌ | ❌ | ❌ |
| 模型告警 | ✅ | ✅ | ❌ | ❌ |

## 限流规则

根据 [限流](../../raw/model-user-guide/get-started-with-models/rate-limit.md)，百炼按**主账号维度**对模型调用设置限流，所有子账号、业务空间和 API Key 的调用量合并计算。

### 限流指标

- **RPM**（Requests Per Minute）：每分钟请求数
- **TPM**（Tokens Per Minute）：每分钟 Token 消耗（含输入和输出）
- 实际可能按秒级 RPS/TPS（RPM/60、TPM/60）执行，短时间请求爆发也可能触发限流

### 典型限流额度（中国内地 / 北京）

| 模型 | RPM | TPM |
|---|---|---|
| `qwen3.7-max` | 30,000 | 5,000,000 |
| `qwen3.6-plus` | 30,000 | 5,000,000 |
| `qwen3.6-flash` | 30,000 | 10,000,000 |
| `qwen-plus`（稳定版） | 30,000 | 5,000,000 |

带日期后缀的快照版本限流通常更低（如 RPM 60~600），优先使用稳定版可获得更宽松的额度。

### 避免限流的策略

1. **优先选用稳定版模型**（如 `qwen-plus` 而非 `qwen-plus-2025-07-28`）
2. **平滑请求速率**：采用匀速调度、指数退避，避免瞬时高峰
3. **添加备选模型**：主模型触发限流后自动切换到备用模型
4. **使用[[batch-inference]]**：无实时要求时，批量推理不受实时限流约束
5. **提升临时额度**：在控制台 [限流提额](https://bailian.console.aliyun.com/?tab=model#/efm/temp_limit_raise) 页面申请，提交后立即生效，有效期 30 天

### 常见限流错误

| 错误信息 | 含义 | 处理方式 |
|---|---|---|
| `Requests rate limit exceeded` | RPM 超限 | 降低调用频率 |
| `Allocated quota exceeded` | TPM 超限 | 缩短输入或限制输出长度 |
| `Request rate increased too quickly` | 请求频率激增触发保护 | 平滑请求速率 |

超限请求被拒绝后通常在**一分钟内**自动恢复。

## 计费概要

- 开通百炼免费，调用模型按量计费（按分钟出账）
- 新用户可获得北京地域的免费额度，用完后自动转为按量付费
- 可开启「免费额度用完即停」功能避免意外扣费
- 详情参见 [[billing-for-model-studio]]

## 相关概念

- [[models]] — 完整模型列表与能力说明
- [[regions]] — 地域与数据合规详情
- [[rate-limit]] — 各模型详细限流额度
- [[api-reference]] — API 参考文档
- [[error-code]] — 错误码与排查指南
- [[model-deployment-introduction]] — 模型部署（专享推理服务）
- [[model-training-overview]] — 模型调优（SFT/CPT/DPO）

## 来源文档

- [什么是阿里云百炼](../../raw/model-user-guide/get-started-with-models/what-is-model-studio.md)
- [首次调用千问API](../../raw/model-user-guide/get-started-with-models/first-api-call-to-qwen.md)
- [选择模型](../../raw/model-user-guide/get-started-with-models/models.md)
- [选择地域和服务部署范围](../../raw/model-user-guide/get-started-with-models/regions.md)
- [限流](../../raw/model-user-guide/get-started-with-models/rate-limit.md)

