# 插件开发

CyberStrikeAI 当前仓库中的插件主要位于 `plugins/`，示例是 Burp Suite 扩展。插件通常通过 HTTP API、MCP 或本地文件与主应用集成。

## 目录

```text
plugins/
  README.md
  burp-suite/
    cyberstrikeai-burp-extension/
      src/main/java/burp/
      README.md
      README.zh-CN.md
      build.gradle
      pom.xml
```

## 插件类型

常见集成方式：

- 浏览器或安全工具扩展：调用 CyberStrikeAI API。
- MCP Server：向 CyberStrikeAI 暴露新工具。
- 文件型扩展：提供 tools、roles、skills、agents。
- Webhook/机器人：通过平台回调与 CyberStrikeAI 对话。

## Burp Suite 扩展

Burp 插件目录包含 Java 源码和构建脚本。典型能力：

- 读取 Burp 中的 HTTP 请求/响应。
- 格式化消息。
- 调用 CyberStrikeAI API。
- 在 Burp 标签页展示 AI 分析结果。

构建前确认：

- JDK 可用。
- Gradle 或 Maven 可用。
- CyberStrikeAI 服务地址和认证配置正确。

## API 对接建议

插件调用主应用时：

- 先登录或使用受控认证方式。
- 优先调用 `/api/eino-agent` 或 `/api/eino-agent/stream`。
- 大文件通过 `/api/chat-uploads` 上传，再在消息中引用。
- 查询结果或漏洞可写入 `/api/vulnerabilities`。
- 项目信息可写入 `/api/projects/:id/facts`。

完整接口以 `/api-docs` 为准。

## MCP 插件

如果插件的目标是给 Agent 增加工具，优先实现 MCP Server。然后在外部 MCP 管理中接入：

- stdio：本机启动。
- HTTP/SSE：长期服务。

MCP 工具设计建议：

- schema 明确。
- 参数最小化。
- 输出结构稳定。
- 错误信息可读。
- 高风险动作拆成独立工具，方便 HITL 审批。

## 文件型扩展

插件也可以交付：

- `tools/*.yaml`
- `roles/*.yaml`
- `skills/<name>/SKILL.md`
- `agents/*.md`

这种方式简单可靠，适合内部方法论或工具链沉淀。

## 发布检查

发布插件前确认：

- 不包含 API Key、Cookie、目标信息。
- README 有安装、配置、卸载说明。
- 错误提示清晰。
- 与当前 CyberStrikeAI API 版本兼容。
- 高风险能力有明显说明。

## 版本兼容

插件应避免依赖未公开的前端内部实现。优先依赖：

- `/api/openapi/spec`
- 稳定 HTTP API。
- MCP 协议。
- 文件目录规范。

如果必须依赖内部接口，插件 README 中应标注兼容版本。

## 插件设计的三个层次

| 层次 | 例子 | 优点 | 代价 |
| --- | --- | --- | --- |
| API 插件 | Burp 扩展调用 `/api/eino-agent` | 易实现，适合 UI 集成 | 依赖认证和 API 稳定性 |
| MCP 插件 | 提供新工具给 Agent | Agent 可主动调用 | 需要 schema 和安全设计 |
| 资源包插件 | 交付 tools/roles/skills/agents | 最简单，可版本化 | 交互能力弱 |

插件一开始不必做成 MCP。如果只是“把 Burp 请求交给 AI 分析”，API 插件更直接；如果要让 Agent 主动调用 Burp 扫描或查询结果，再做 MCP。

## API 插件请求设计

发送给 Agent 的内容应包含：

- 来源工具和上下文。
- 目标 URL、方法、关键 header。
- 请求体和响应体的截断策略。
- 用户希望 AI 做什么。
- 授权边界。

不要把完整大响应直接塞进消息。大文件应走上传接口或做摘要。

## MCP 插件 schema 设计

坏 schema：

```json
{"cmd":{"type":"string"}}
```

好 schema：

```json
{
  "target_url": {"type":"string","description":"授权目标 URL"},
  "scan_profile": {"type":"string","enum":["passive","active-safe"]},
  "max_requests": {"type":"integer","description":"最大请求数"}
}
```

schema 越具体，HITL 越容易判断风险，Agent 也越不容易发散。

## 插件安全边界

插件不要绕过平台安全控制：

- 不要直接执行本机高风险命令而不暴露给 HITL。
- 不要在插件内保存明文长期凭证。
- 不要默认把目标数据发给第三方服务。
- 不要依赖浏览器本地状态绕过登录。

## 源码锚点

- Burp 插件 Java 代码：`plugins/burp-suite/cyberstrikeai-burp-extension/src/main/java/burp/`
- OpenAPI：`internal/handler/openapi.go`
- 外部 MCP：`internal/handler/external_mcp.go`
