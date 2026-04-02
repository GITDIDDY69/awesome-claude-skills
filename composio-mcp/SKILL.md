---
name: composio-mcp
description: Set up and use Composio as a remote MCP server in Claude Code to connect to 1000+ apps. Use when users want to integrate Claude with external services like Gmail, Slack, GitHub, Jira, Notion, Google Drive, and more — with managed OAuth and no local server setup.
---

# Composio MCP Integration

## Overview

Composio provides a hosted MCP server at `https://connect.composio.dev/mcp` that connects Claude to 1000+ SaaS applications through the Model Context Protocol. It handles OAuth flows, token refresh, sandboxed execution, and rate limiting automatically.

This skill covers setup, configuration, authentication, and usage patterns.

---

# Quick Start

## Step 1: Get API Key

1. Sign up at [platform.composio.dev](https://platform.composio.dev)
2. Go to **Settings → API Keys** → Create key
3. Export it:

```bash
export COMPOSIO_API_KEY="your-key-here"
```

## Step 2: Configure Claude Code

Add to your project's `.claude/settings.json`:

```json
{
  "mcpServers": {
    "composio": {
      "url": "https://connect.composio.dev/mcp",
      "headers": {
        "x-api-key": "${COMPOSIO_API_KEY}"
      }
    }
  }
}
```

Merge with existing settings if the file already exists.

## Step 3: Authenticate Apps

On first use of any app tool, Composio provides an OAuth URL. Open it, authorize, and you're connected.

## Step 4: Use Tools

Composio tools use the pattern `{APP}_{ACTION}`:

- `GMAIL_SEND_EMAIL` — Send emails
- `SLACK_SEND_MESSAGE` — Post to Slack
- `GITHUB_CREATE_ISSUE` — Create issues
- `GOOGLE_SHEETS_ADD_ROW` — Add spreadsheet data
- `JIRA_CREATE_ISSUE` — Create Jira tickets
- `NOTION_CREATE_PAGE` — Create Notion pages

---

# Supported Apps (1000+)

| Category | Examples |
|----------|----------|
| **Email** | Gmail, Outlook, SendGrid, Mailchimp |
| **Calendar** | Google Calendar, Outlook Calendar, Calendly |
| **Messaging** | Slack, Discord, Microsoft Teams, Telegram |
| **Project Management** | Jira, Asana, Linear, Monday, Trello, Notion, ClickUp |
| **Code & DevOps** | GitHub, GitLab, Bitbucket, Vercel, Netlify, CircleCI |
| **Storage** | Google Drive, Dropbox, OneDrive, Box |
| **Spreadsheets** | Google Sheets, Airtable, Excel Online |
| **CRM** | HubSpot, Salesforce, Pipedrive, Zoho CRM |
| **Social Media** | LinkedIn, Twitter/X, Instagram, Facebook |
| **Payments** | Stripe, PayPal, Square |
| **Support** | Zendesk, Intercom, Freshdesk |
| **E-commerce** | Shopify, WooCommerce |
| **Analytics** | Google Analytics, Mixpanel, Amplitude |

---

# Configuration Patterns

## Project-Level (Recommended)

Add to `.claude/settings.json` in your repo. The `${COMPOSIO_API_KEY}` syntax reads from the user's environment, keeping secrets out of version control.

## User-Level (Global)

For access across all projects, add the same config to `~/.claude/settings.json`.

## Custom Server (Advanced)

For restricted tool subsets or multi-user setups, create a custom MCP server via the Composio SDK:

```typescript
import { Composio } from '@composio/core';

const composio = new Composio({ apiKey: process.env.COMPOSIO_API_KEY });
const server = await composio.mcpServer.create({
  name: 'project-server',
  toolkits: ['GMAIL', 'SLACK', 'GITHUB'],
  allowedTools: ['GMAIL_SEND_EMAIL', 'SLACK_SEND_MESSAGE', 'GITHUB_CREATE_ISSUE']
});
// Use returned URL in settings.json
```

Endpoint format: `https://backend.composio.dev/v3/mcp/{SERVER_ID}?user_id={USER_ID}`

---

# Best Practices

1. **Least privilege** — Only authenticate the apps you need
2. **Env vars for secrets** — Always use `${COMPOSIO_API_KEY}`, never hardcode keys
3. **Manage connections** — Review and revoke at platform.composio.dev → Connections
4. **Rate awareness** — For bulk operations, add delays between tool calls
5. **Scope tools** — Use custom MCP servers to restrict available tools per project

---

# Troubleshooting

| Issue | Fix |
|-------|-----|
| MCP server not connecting | Ensure `COMPOSIO_API_KEY` is exported. Restart Claude Code. |
| Tool not found | Enable the toolkit in your Composio dashboard |
| OAuth expired | Re-authenticate at platform.composio.dev → Connections |
| Permission denied | Grant additional OAuth scopes during re-auth |
| Rate limited | Wait for backoff period; Composio returns retry headers |

---

# Integration with Existing Skills

Composio MCP works alongside other Claude Code skills and MCP servers. Common combinations:

- **composio-mcp + mcp-builder**: Use Composio for managed integrations, build custom MCP servers for proprietary APIs
- **composio-mcp + connect-apps**: Composio MCP provides the server-side execution; connect-apps plugin provides the Claude Code UI integration
- **composio-mcp + workflow-orchestrator**: Chain Composio tool calls into multi-step business workflows
