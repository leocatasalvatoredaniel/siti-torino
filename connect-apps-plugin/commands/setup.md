---
description: Set up connect-apps - let Claude perform real actions in 500+ apps
allowed-tools: [Bash, AskUserQuestion]
---

# Connect Apps Setup

Set up the Composio MCP server so Claude can take real actions in external apps (Gmail, Slack, GitHub, Notion, and 500+ more).

## Instructions

### Step 1: Add the Composio MCP server

Run this single command:

```bash
claude mcp add --scope user --transport http composio https://connect.composio.dev/mcp
```

This registers Composio as a user-level MCP server available in all Claude Code sessions.

### Step 2: Confirm

Tell the user:

```
Setup complete!

To activate: exit and run `claude` again (or `claude --plugin-dir ./connect-apps-plugin`)

Then try asking Claude to:
- "Send me a test email at your@email.com"
- "Create a GitHub issue in my repo"
- "Post a message to my Slack channel"

First use of each app will prompt you to authorize via OAuth — just follow the link.
```

## Notes

- `--scope user` means the server is available in all your Claude Code sessions
- Auth is handled automatically per app via OAuth on first use
- No API key or pip install needed
