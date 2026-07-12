# MCP Dependencies

Use this reference when a skill depends on an MCP server, connector, or tool provider.

## agents/openai.yaml Pattern

```yaml
dependencies:
  tools:
    - type: "mcp"
      value: "github"
      description: "GitHub MCP server for repository, issue, and pull request operations."
      transport: "streamable_http"
      url: "https://api.githubcopilot.com/mcp/"
```

Replace:

- `value`: stable MCP server or connector identifier.
- `description`: what the dependency is used for in this skill.
- `transport`: MCP transport type, usually `streamable_http` when using a remote MCP endpoint.
- `url`: MCP server URL, when applicable.

## SKILL.md Guidance

In the skill body, include:

- Which operations require the MCP dependency.
- What local or manual fallback to use if the dependency is missing.
- Any authentication, organization, repository, tenant, or workspace assumption.
- Whether the skill may read, write, create, delete, or publish data through the MCP tool.

## Common Examples

GitHub:

```yaml
dependencies:
  tools:
    - type: "mcp"
      value: "github"
      description: "Inspect repositories, issues, pull requests, checks, and reviews."
      transport: "streamable_http"
      url: "https://api.githubcopilot.com/mcp/"
```

Internal company MCP:

```yaml
dependencies:
  tools:
    - type: "mcp"
      value: "company-knowledge"
      description: "Access internal company knowledge, policies, and approved operational data."
      transport: "streamable_http"
      url: "https://mcp.example-company.com/mcp"
```
