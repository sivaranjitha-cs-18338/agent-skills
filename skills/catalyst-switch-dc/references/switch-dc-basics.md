# Switch the Catalyst MCP Data Center

Use this skill when a Codex user needs to select or change the regional Catalyst MCP endpoint.

## Safety Rules

- The plugin ships with `IN` enabled by default. Catalyst MCP endpoints are region-specific for compliance, so never infer any other DC from the user's location, project name, previous session, or available account data.
- Require the user to explicitly choose one of: `US`, `EU`, `IN`, `AU`, `CA`, `SA`, `JP`, `UAE`.
- Show the exact endpoint before applying the change.
- Do not edit the installed plugin's `.codex-plugin/mcp.json`; it is managed plugin content.
- Do not edit OAuth credentials.
- After changing the configuration, make no further `ZohoMCP_*` calls in the current session. The old connection remains active until Codex restarts.

## Procedure

1. If the user did not specify a supported DC, ask them to choose one. Do not guess a DC; `IN` is only the plugin's bundled default until the user chooses.
2. State the matching endpoint from this table and ask for confirmation if the user's request did not already explicitly authorize the switch:

   | DC | Endpoint |
   |----|----------|
   | IN (default) | `https://catalyst.zohomcp.in/mcp/message` |
   | US | `https://catalyst.zohomcp.com/mcp/message` |
   | EU | `https://catalyst.zohomcp.eu/mcp/message` |
   | AU | `https://catalyst.zohomcp.com.au/mcp/message` |
   | CA | `https://catalyst.zohomcp.ca/mcp/message` |
   | SA | `https://catalyst.zohomcp.sa/mcp/message` |
   | JP | `https://catalyst.zohomcp.jp/mcp/message` |
   | UAE | `https://catalyst.zohomcp.ae/mcp/message` |

3. Run `codex plugin list --json` and find the enabled entry whose `name` is `catalyst-by-zoho`. Use its `source.path` as `<installed-plugin-path>`. If more than one enabled entry matches, ask which installation to configure.
4. Run the bundled helper with permission to update the user-level Codex configuration:

   ```bash
   node "<installed-plugin-path>/scripts/switch-dc.js" <DC>
   ```

   The helper:
   - accepts a code (`eu`), a region name (`europe`, `india`) or a server name (`catalyst-eu`), in any case;
   - with no argument in a terminal, shows a numbered menu with the active DC marked;
   - reports "Already using <DC>" and changes nothing when the DC is already selected;
   - discovers the installed plugin ID using `codex plugin list --json`;
   - updates only `plugins.<plugin-id>.mcp_servers.<regional-server>.enabled` in `~/.codex/config.toml`;
   - disables every other Catalyst regional server;
   - preserves unrelated settings and creates `config.toml.catalyst-dc.bak`;
   - writes the result atomically.

5. If multiple enabled Catalyst plugin installations are reported, ask the user which installation to configure, then rerun:

   ```bash
   node "<installed-plugin-path>/scripts/switch-dc.js" <DC> --plugin-id "<plugin-id>"
   ```

6. Report the selected DC and endpoint. Tell the user to restart Codex and complete the browser OAuth flow for that regional endpoint.
7. Stop. Do not perform any Catalyst MCP operation until a new session confirms that exactly one regional server is enabled and the `ZohoMCP_*` meta-tools are available.

## Check Current Selection

```bash
node "<installed-plugin-path>/scripts/switch-dc.js" --list     # all DCs, active one marked
node "<installed-plugin-path>/scripts/switch-dc.js" --status   # JSON detail
```

The status command exits successfully only when exactly one Catalyst regional server is enabled.

## Common Errors

| Error | Cause | Fix |
|-------|-------|-----|
| `EPERM`/`EACCES` writing `~/.codex/config.toml.catalyst-dc.lock` | Codex's sandbox blocks writes outside the workspace | Run the helper in your own terminal, or approve running it outside the sandbox |
| `Another Catalyst DC update may be running` | Stale `config.toml.catalyst-dc.lock` file | Delete the lock file named in the message and retry |
| `Could not list installed Codex plugins` | `codex plugin list --json` failed | Re-run the helper with `--plugin-id <id>` |
| Multiple enabled Catalyst plugin installations | More than one install matched | Ask the user which one, then pass `--plugin-id` |
| `ZohoMCP_*` tools still point to the old region | Codex was not restarted | Restart Codex and redo the OAuth flow |
| `--status` exits non-zero | Zero or several regional servers enabled | Re-run the helper with one explicit DC |
