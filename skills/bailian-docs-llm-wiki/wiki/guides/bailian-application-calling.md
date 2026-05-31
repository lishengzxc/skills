# bailian [[application-call|application call]]ing

百炼平台支持通过 DashScope SDK 或 HTTP API 将应用集成到业务系统中。目前支持调用**智能体应用**和**工作流应用**两种类型，调用方式统一，均通过 `Application.call` 接口（SDK）或 `/api/v1/apps/{APP_ID}/completion` 端点（HTTP）完成。本文汇总了应用调用的核心流程、关键参数及注意事项。

## 支持的应用类型

| 应用类型 | 说明 |
|---------|------|
| 智能体应用 | 单 Agent 应用，可关联自定义插件，适合对话、问答等场景 |
| 工作流应用 | 基于工作流编排的应用（已替代原智能体编排应用），支持插件节点等复杂流程 |

两种应用的基础调用方式完全一致，详见 [调用智能体应用](../../raw/application-user-guide/bailian-application-calling/call-single-agent-application.md) 和 [调用工作流应用](../../raw/application-user-guide/bailian-application-calling/invoke-workflow-application.md)。

## 前提条件

1. **获取 API Key** 并配置到环境变量 `DASHSCOPE_API_KEY`（推荐，避免硬编码）
2. 在百炼控制台创建应用并获取 `APP_ID`
3. 若使用 SDK 调用，需安装 DashScope SDK（Python / Java）

> **注意**：工作流应用文档明确标注"仅适用于中国大陆版（北京地域）"，智能体应用文档未作此限制，实际使用时请确认所在地域的可用性。

## 调用方式

### API 端点

```
POST https://dashscope.aliyuncs.com/api/v1/apps/{APP_ID}/completion
```

请求头：
- `Authorization: Bearer $DASHSCOPE_API_KEY`
- `Content-Type: application/json`

### SDK 调用（最小示例）

**Python**
```python
import os
from dashscope import Application

response = Application.call(
    api_key=os.getenv("DASHSCOPE_API_KEY"),
    app_id='YOUR_APP_ID',
    [[prompt|prompt]]='你是谁？'
)
print(response.output.text)
```

**Java**（建议 SDK 版本 >= 2.12.0）
```java
ApplicationParam param = ApplicationParam.builder()
    .apiKey(System.getenv("DASHSCOPE_API_KEY"))
    .appId("YOUR_APP_ID")
    .[[prompt|prompt]]("你是谁？")
    .build();
Application application = new Application();
ApplicationResult result = application.call(param);
System.out.println(result.getOutput().getText());
```

支持的语言/方式：Python、Java、curl、PHP、Node.js、C#、Go。

## 关键参数

| 参数 | 位置 | 说明 |
|------|------|------|
| `app_id` | 必填 | 应用 ID，从控制台应用卡片获取 |
| `[[prompt|prompt]]` | input | 用户输入的提示文本 |
| `session_id` | input/output | 多轮对话会话标识，有效期 1 小时，最多 50 轮 |
| `messages` | input | 自行管理的对话历史数组（推荐），优先级高于 `session_id` |
| `biz_params` | input | 业务透传参数，用于自定义插件/节点的参数传递 |
| `parameters` | 顶层 | 模型参数配置（可选） |
| `debug` | 顶层 | 调试配置（可选） |

### 响应结构

```json
{
  "output": {
    "finish_reason": "stop",
    "session_id": "...",
    "text": "模型回复内容"
  },
  "usage": {
    "models": [{"output_tokens": 51, "model_id": "qwen-max", "input_tokens": 121}]
  },
  "request_id": "..."
}
```

## 多轮对话

两种方式实现多轮对话：

1. **`session_id` 云端存储**：首次调用后从响应中获取 `session_id`，后续请求携带该值即可。系统自动加载历史。
2. **`messages` 自行管理（推荐）**：手动维护对话数组，灵活控制上下文，无需传 `prompt`。

> **注意**：若请求中同时包含 `session_id` 和 `messages`，系统将优先使用 `messages`。

工作流应用使用 `messages` 方式时，需在大模型节点配置提示词变量 `historyList` 并重新发布应用。

## 自定义参数传递

当应用关联了自定义插件或包含自定义节点时，可通过 `biz_params` 传递业务参数。详细配置流程参见 [应用的自定义参数传递](../../raw/application-user-guide/bailian-application-calling/pass-through-of-application-parameters.md)。

### 自定义插件参数

```python
biz_params = {
    "user_defined_params": {
        "your_plugin_code": {   # 替换为实际插件 ID
            "article_index": 2   # 插件定义的输入参数
        }
    }
}
response = Application.call(
    api_key=os.getenv("DASHSCOPE_API_KEY"),
    app_id='YOUR_APP_ID',
    prompt='寝室公约内容',
    biz_params=biz_params
)
```

关键步骤：
1. 创建自定义插件时，输入参数的**传参方式**须选择「业务透传」
2. 将插件关联到智能体应用（或通过工作流的插件节点引用）
3. API 调用时在 `biz_params.user_defined_params` 中按插件 ID 传入参数

如果插件需要鉴权，还需在创建插件时配置鉴权信息，并通过 `biz_params` 中的 `user_defined_tokens` 传递鉴权凭证。

## 限制和注意事项

- `session_id` 有效期为 **1 小时**，最多支持 **50 轮**对话
- 不建议在生产环境中硬编码 [[api-key]]，应通过环境变量配置
- 工作流应用仅适用于中国大陆版（北京地域）
- 如需使用 [[openai-responses-api]]，请参阅对应文档
- 错误处理时可参考 `response.status_code` 和 `response.message`，错误码文档：https://help.aliyun.com/zh/model-studio/developer-reference/error-code

## 来源文档

- [调用工作流应用](../../raw/application-user-guide/bailian-application-calling/invoke-workflow-application.md)
- [调用智能体应用](../../raw/application-user-guide/bailian-application-calling/call-single-agent-application.md)
- [应用的自定义参数传递](../../raw/application-user-guide/bailian-application-calling/pass-through-of-application-parameters.md)

