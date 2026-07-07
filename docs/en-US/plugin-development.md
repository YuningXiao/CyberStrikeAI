# Plugin Development

[中文](../zh-CN/plugin-development.md)

Plugins can integrate with CyberStrikeAI through HTTP APIs, MCP servers, or resource packs such as tools, roles, Skills, and agents.

## Plugin Layers

| Layer | Example | Benefit | Cost |
| --- | --- | --- | --- |
| API plugin | Burp extension calls `/api/eino-agent` | simple UI integration | depends on API/auth |
| MCP plugin | exposes tools to Agent | Agent can call it | needs schema and safety design |
| Resource pack | ships tools/roles/skills/agents | simple and versionable | less interactive |

Do not start with MCP unless the Agent must actively call your capability.

## API Plugin Payload

Include:

- source tool and context;
- target URL, method, key headers;
- truncation policy for request/response bodies;
- user intent;
- authorization boundary.

Large responses should be uploaded or summarized, not pasted whole into the prompt.

## MCP Schema Design

Bad:

```json
{"cmd":{"type":"string"}}
```

Better:

```json
{
  "target_url": {"type":"string","description":"authorized target URL"},
  "scan_profile": {"type":"string","enum":["passive","active-safe"]},
  "max_requests": {"type":"integer","description":"request limit"}
}
```

Specific schemas make HITL and Agent behavior safer.

## Security Boundaries

Plugins should not bypass platform controls:

- no hidden destructive local commands;
- no plaintext long-lived credentials;
- no default third-party data exfiltration;
- no dependency on browser state to bypass login.

## Source Anchors

- Burp plugin: `plugins/burp-suite/cyberstrikeai-burp-extension/src/main/java/burp/`
- OpenAPI: `internal/handler/openapi.go`
- External MCP: `internal/handler/external_mcp.go`
