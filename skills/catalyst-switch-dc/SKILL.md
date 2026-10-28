---
name: catalyst-switch-dc
description: "Select or change the regional Catalyst Zoho MCP endpoint used by the Codex plugin. Trigger on 'switch Catalyst DC', 'change MCP region', 'connect to IN DC', 'use EU data center', 'configure Catalyst MCP region', 'regional server', 'switch-dc.js', or 'MCP REGIONAL BLOCKER'."
metadata:
  version: "2.0.0"
---

## How It Works

1. The plugin defaults to `IN`. Never infer any other DC from location, project name, or prior sessions; the user must explicitly choose one of `US`, `EU`, `IN`, `AU`, `CA`, `SA`, `JP`, `UAE`.
2. Load `references/switch-dc-basics.md` for the endpoint table and the helper procedure.
3. Run `scripts/switch-dc.js` from the installed plugin only after the user confirms the exact endpoint. It edits `~/.codex/config.toml` only.
4. After switching, make no further `ZohoMCP_*` calls; the user must restart Codex and complete OAuth.

## Triggers

Use this skill for: "switch Catalyst DC", "change MCP region", "connect to IN DC", "use EU data center", "configure Catalyst MCP region", "MCP REGIONAL BLOCKER", or `switch-dc.js`.

## References

| Reference | Load when the query is about… |
|-----------|-------------------------------|
| `references/switch-dc-basics.md` | Endpoint table, helper procedure, status check, errors |
