# Model Context Protocol

模型上下文协议（Model Context Protocol, MCP）是一种开源标准协议，旨在搭建大模型与外部工具之间的信息传递通道。通过 MCP，开发者无需为每个外部工具编写复杂的接口，即可在阿里云百炼的[[agent]]或[[workflow]]应用中接入海量第三方工具。MCP 协议由 Anthropic 提出，详细规范可参考 [MCP 官网](https://modelcontextprotocol.io/)。

## 支持的接入方式

根据 [模型上下文协议（MCP）](../../raw/application-user-guide/model-context-protocol/mcp-introduction.md)，百炼平台支持两类 MCP 服务：

### 官方 MCP 服务

百炼官方部署的云端 MCP 服务，开通后即可使用，无需自行部署。当前可用的官方服务包括 Amap Maps（地理信息）、Sequential Thinking（逐步推理）、QuickChart（图表生成）、Firecrawl（网页爬取）、联网搜索等。

### 自定义 MCP 服务

根据 [自定义 MCP 服务](../../raw/application-user-guide/model-context-protocol/custom-mcp.md)，百炼支持三种自定义部署方式：

| 部署方式 | 适用场景 | 说明 |
|---------|---------|------|
| **使用脚本部署**（npx/uvx/http） | 拥有遵循 MCP 协议的代码包 | 通过函数计算 FC 托管，支持 Node.js（npx）、Python（uvx）和远程服务（http） |
| **从 AI 网关导入** | 将现有 RESTful API 封装为 MCP 服务 | 需先在 AI 网关托管 MCP 服务 |
| **从阿里云 OpenAPI 导入** | 操作阿里云云资源（OSS、ECS 等） | 需先在 OpenAPI 开发者门户创建 MCP 服务 |

## 使用方式

### 在智能体中使用

智能体应用中，大模型根据输入对话自动判断是否调用 MCP 服务，支持同时添加最多 **5 个** MCP 服务。配置流程：创建智能体 → 添加 MCP 服务 → 测试对话。

### 在工作流中使用

工作流中每个 MCP 节点只能使用一个工具，需手动指定输入参数并传递输出到下游节点。典型模式为：开始节点 → 大模型节点（提取参数）→ MCP 节点（调用工具）→ 大模型节点（整理结果）→ 结束节点。

### 外部调用

根据 [外部调用](../../raw/application-user-guide/model-context-protocol/mcp-external-calls.md)，百炼 MCP 服务支持集成到第三方应用或个人项目：

- **第三方应用集成**：支持一键配置至 Cherry Studio、Cursor 等客户端
- **SDK 开发集成**：通过 MCP SDK（如 Qwen Agent）编码调用，使用 Streamable HTTP 协议连接

外部调用的 MCP 端点格式为：
```
https://dashscope.aliyuncs.com/api/v1/mcps/{service-name}/mcp
```
需在请求头中携带百炼 API Key 进行鉴权。

> **注意**：百炼 MCP 服务已从旧版 SSE 协议升级为新版 Streamable HTTP 协议。已开通的用户需先取消再重新开通以完成升级。

## 关键配置参数

### 脚本部署配置模板

```json
{
  "mcpServers": {
    "本地 MCP 服务": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@your_acc_name/your_pkg_name"],
      "env": { "YOUR_ENV_KEY": "YOUR_ENV_VALUE" }
    },
    "远程 MCP 服务": {
      "type": "sse/streamableHttp",
      "url": "https://your-mcp-server/sse"
    }
  }
}
```

### 部署模式选择

| 模式 | 部署费 | 调用费率 | 部署费率 | 适用场景 |
|------|--------|---------|---------|---------|
| 基础模式 | 无 | 0.000156 元/秒 | — | 偶尔调用，可接受冷启动延迟 |
| 极速模式 | 有 | 0.000156 元/秒 | 0.000036 元/秒 | 频繁调用，需低延迟响应 |

官方云部署 MCP 服务限时免部署费用。联网搜索 MCP 服务免费额度 2000 次，超出后 29 元/千次，限流 15 QPS（主账号与 RAM 子账号共享）。

## 限制和注意事项

- **MCP 服务不能直接在千问 API 调用时接入**，必须集成在智能体或工作流应用中使用
- **不支持访问本地资源**：云端 MCP 服务无法访问用户本地数据库或文件系统
- **远程资源访问**：MCP 服务托管在函数计算 FC，无固定出口公网 IP，访问云数据库等需配置 IP 白名单或 VPC 打通
- **版本更新不自动同步**：通过 npx/uvx 部署的服务版本更新后需手动重新部署
- **私有仓库不支持**：npm 私有仓库的 MCP Server 暂不支持部署，需发布到公共仓库或改用 SSE/HTTP 连接
- **Token 消耗增加**：调用 MCP 会增加模型输入和输出 Token 数量
- **安全提醒**：部署自定义 MCP 服务时务必核实来源和源代码，防止钓鱼攻击
- **智能体调用失败时**：优化提示词明确工具名称和能力，或更换更强的推理模型（如[[qwen]]3 系列）

## 常见错误排查

关键错误码速查（完整列表参见 [MCP 常见问题](../../raw/application-user-guide/model-context-protocol/mcp-faq.md)）：

| 错误码 | 含义 | 处理建议 |
|--------|------|---------|
| `MCP_CONNECTION_REFUSED` | 连接被拒绝 | 检查服务是否启动、端口和防火墙配置 |
| `MCP_REQUEST_TIMEOUT` | 响应超时 | 重试 2-3 次；拆分业务或切换极速模式 |
| `MCP_SERVER_HTTP_UNAUTHORIZED` | 401 鉴权失败 | 检查 Token/API Key 是否正确 |
| `MCP_PROTOCOL_ERROR` | 协议解析失败 | 确认 `type` 与端点路径匹配：`sse` → `/sse`，`streamableHttp` → `/mcp` |
| `MCP_INIT_TIMEOUT` | 初始化超时 | 基础模式有冷启动延迟，可切换极速模式 |

## 来源文档

- [模型上下文协议（MCP）](../../raw/application-user-guide/model-context-protocol/mcp-introduction.md)
- [官方 MCP 服务](../../raw/application-user-guide/model-context-protocol/official-and-third-party-mcp.md)
- [自定义 MCP 服务](../../raw/application-user-guide/model-context-protocol/custom-mcp.md)
- [外部调用](../../raw/application-user-guide/model-context-protocol/mcp-external-calls.md)
- [MCP 常见问题](../../raw/application-user-guide/model-context-protocol/mcp-faq.md)

