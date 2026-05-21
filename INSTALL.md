# Clean Deploy — Quick Install Guide

> Get a Dynatrace MCP workspace running in 5 minutes.

---

## Prerequisites

- **Node.js 18+** — [nodejs.org](https://nodejs.org)
- **VS Code** with GitHub Copilot (Chat enabled)
- **Dynatrace Platform Token** with required scopes

> Works on **macOS**, **Windows**, and **Linux**. Platform-specific commands are noted below.

---

## 1. Download the Template

Download a clean copy of the working folder, using web download or commands, from:  
**https://github.com/mf-dynatrace/MCP-cleanDeploy**

**macOS / Linux (Terminal):**
```bash
git clone https://github.com/mf-dynatrace/MCP-cleanDeploy.git my-client-workspace
cd my-client-workspace
```

**Windows (PowerShell or CMD):**
```powershell
git clone https://github.com/mf-dynatrace/MCP-cleanDeploy.git my-client-workspace
cd my-client-workspace
```

## 2. Create Your `.env`

**macOS / Linux:**
```bash
cp .env.example .env
```

**Windows (PowerShell):**
```powershell
Copy-Item .env.example .env
```

**Windows (CMD):**
```cmd
copy .env.example .env
```

Open `.env` in any text editor and fill in these three values:

```dotenv
DT_ENVIRONMENT=https://YOUR_TENANT_ID.apps.dynatrace.com
DT_PLATFORM_TOKEN=dt0s16.XXXXXXXX.XXXXXXXX...
MCP_USER_ID=your.email@company.com
```

## 3. Token Scopes

Create a **Platform Token** in Dynatrace with:

| Scope | Required For |
|-------|-------------|
| `app-engine:apps:run` | MCP server |
| `storage:logs:read` | Log queries |
| `storage:events:read` | Event queries |
| `storage:spans:read` | Trace/span queries |
| `storage:bizevents:read` | Business events |
| `storage:metrics:read` | Metric queries |
| `storage:entities:read` | Entity lookups |
| `storage:user.sessions:read` | RUM sessions (optional) |
| `storage:user.events:read` | RUM events (optional) |

## 4. Open in VS Code

**macOS / Linux / Windows:**
```bash
code .
```

> If `code` is not recognised on macOS, open VS Code → `Cmd+Shift+P` → "Shell Command: Install 'code' command in PATH".  
> On Windows, the VS Code installer adds `code` to PATH automatically.

The MCP server auto-starts via `.vscode/mcp.json` — no install step needed.  
It runs: `npx -y @dynatrace-oss/dynatrace-mcp-server@latest`

## 5. Verify Connection

In Copilot Chat, ask:

> "What Dynatrace environment am I connected to?"

If it responds with your tenant ID, you're connected.

---

## 6. Replace Placeholders

Search and replace across all files:

| Placeholder | Replace With |
|-------------|-------------|
| `[CLIENT_NAME]` | Customer name |
| `[TENANT_ID]` | Dynatrace tenant ID |
| `[INDUSTRY]` | Customer industry |
| `[WEBSITE_URL]` | Customer website URL |

> **Tip:** You can ask your LLM to mass-change these by prompting:  
> *"Replace all placeholders with the correct information, prompt with discovered values before writing to files"*

---

## Feature Flags (Optional)

All default to `yes`. Set to `no` in `.env` to disable:

| Flag | Effect when `no` |
|------|-----------------|
| `MCP_GRAIL_ONLY` | Enables Gen 2 USQL/classic APIs |
| `MCP_USE_USER_VARIABLE` | Skips user identity on events |
| `MCP_SEND_TRACKING_EVENTS` | Disables query tracking |

---

## File Structure (Key Files)

```
.env                          ← Your credentials (git-ignored)
.vscode/mcp.json              ← MCP server config (auto-starts)
.github/copilot-instructions.md ← Copilot behaviour rules
CLAUDE.md                     ← Claude/Anthropic instructions
reference/                    ← Self-updating knowledge base
skills/                       ← DQL domain knowledge (12 files)
example/                      ← Sample dashboards & workflows
```

---

## First Session Checklist

- [ ] `.env` configured and connection verified
- [ ] Placeholders replaced in `CLAUDE.md` and `copilot-instructions.md`
- [ ] Run `find_entity_by_name` to discover key services (FREE)
- [ ] Update `reference/Entities_Reference.md` with discovered IDs
- [ ] Run a BizEvents summary to discover event types
- [ ] Update `reference/BizEvents_Reference.md`

---

## Troubleshooting

| Problem | Fix |
|---------|-----|
| MCP not connecting | Check `DT_ENVIRONMENT` URL format (must be `*.apps.dynatrace.com`) |
| Auth errors | Verify token scopes match table above |
| "Not authorized for table" | Add missing `storage:*:read` scope to token |
| Copilot ignoring instructions | Ensure `.github/copilot-instructions.md` exists at workspace root |
| High query costs | Read `reference/MCP_Query_Optimization_Guide.md` |
