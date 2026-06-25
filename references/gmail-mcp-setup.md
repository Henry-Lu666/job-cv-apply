# Gmail MCP Server Setup

> 2026-06-24: Installed and configured
> 2026-06-25: OAuth completed ✅, MCP tools verified working (list_labels returned 14 labels, read_email full body extraction confirmed)

## Package

```
npm install -g @gongrzhe/server-gmail-autoauth-mcp
```

Binary: `~/.hermes/node/lib/node_modules/@gongrzhe/server-gmail-autoauth-mcp/dist/index.js`

## Hermes Config (`~/.hermes/config.yaml`)

```yaml
mcp_servers:
  gmail:
    command: node
    args:
      - [USER_HOME]/.hermes/node/lib/node_modules/@gongrzhe/server-gmail-autoauth-mcp/dist/index.js
    connect_timeout: 60
    env:
      GMAIL_OAUTH_PATH: [USER_HOME]/.gmail-mcp/gcp-oauth.keys.json
      GMAIL_CREDENTIALS_PATH: [USER_HOME]/.gmail-mcp/credentials.json
      PATH: [USER_HOME]/.hermes/node/bin:/usr/local/bin:/usr/bin:/bin
    timeout: 30
```

## OAuth Setup (user manual steps)

1. Google Cloud Console → Create project → Enable Gmail API
2. Create OAuth client ID → Desktop app → Download JSON
3. Save JSON as `~/.gmail-mcp/gcp-oauth.keys.json`
4. Run: `npx @gongrzhe/server-gmail-autoauth-mcp auth`
5. Browser opens → Login → Authorize → Credentials saved
6. Restart Hermes

## Verified Tools (2026-06-25)

Actual tool names after MCP connection (prefix: `mcp_gmail_`):
- `mcp_gmail_search_emails` — search with Gmail query syntax (maxResults param)
- `mcp_gmail_read_email` — read full message by ID (returns thread ID, subject, from, to, date, body)
- `mcp_gmail_list_email_labels` — list all labels (system + user)
- `mcp_gmail_send_email` — send with attachments
- `mcp_gmail_modify_email` — add/remove labels (move to folder)
- `mcp_gmail_delete_email` — permanently delete
- `mcp_gmail_create_label` / `update_label` / `delete_label`
- `mcp_gmail_download_attachment`
- `mcp_gmail_create_filter` / `list_filters` / `delete_filter`
- `mcp_gmail_draft_email`
- `mcp_gmail_batch_modify_emails` / `batch_delete_emails`

⚠️ Tool names differ from original npm package docs. Always use `mcp_gmail_` prefix.

## Token Impact

~15-20 tools → ~8-12K tokens per system prompt injection. Acceptable given 99K compression threshold.

## Pitfalls

- **npx adds 10-15s startup** — use global install + full binary path instead
- **WSL OAuth callback** — OAuth callback uses localhost:3000, which works in WSL if browser opens on Windows host
- **Credential file location** — must be `~/.gmail-mcp/credentials.json`, not in project directory
