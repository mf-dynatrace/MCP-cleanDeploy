# Dynatrace MCP Connection - Clean Deploy Template

> **Version:** 2.0  
> **Created:** 30 January 2026  
> **Updated:** 6 March 2026  
> **Purpose:** Reusable template for Dynatrace MCP connections with cost optimization and self-learning capabilities

---

## � MCP Server Setup

### Prerequisites
- Node.js 18+ installed
- VS Code with GitHub Copilot
- Dynatrace Platform access with API token

### Step 1: Create Your .env File

Copy `.env.example` to `.env` and configure:

```bash
cp .env.example .env
```

Edit `.env` with your Dynatrace credentials:
```dotenv
# Your Dynatrace Platform URL (use apps.dynatrace.com format)
DT_ENVIRONMENT=https://abc12345.apps.dynatrace.com

# Platform Token with required scopes
DT_PLATFORM_TOKEN=dt0s16.XXXXXXXX.XXXXXXXXXXXXXXXX...

# Feature flags (yes/no) - see Feature Flags section below
MCP_GRAIL_ONLY=yes
MCP_USE_USER_VARIABLE=yes
MCP_SEND_TRACKING_EVENTS=yes
```

### Step 2: Required Token Scopes

Create a Platform Token in Dynatrace with these scopes:

**Core Data Access:**
- `app-engine:apps:run`
- `storage:logs:read`
- `storage:events:read`
- `storage:spans:read`
- `storage:bizevents:read`
- `storage:metrics:read`
- `storage:entities:read`

**RUM/Session Analytics (if using Real User Monitoring):**
- `storage:user.sessions:read` - Gen3 session analytics (cheaper than user.events!)
- `storage:user.events:read` - RUM events (JS errors, navigations, interactions)

**Security/Problems (if needed):**
- `storage:security.vulnerabilities:read` - Vulnerability data
- `storage:problems:read` - Davis problem data

**Note:** Use `user.sessions` (dot-notation) for session-level aggregates. It's much cheaper than `user.events` for device/geo/engagement analysis.

### Step 3: Verify MCP Connection

1. Open VS Code in this workspace
2. The MCP server starts automatically (see `.vscode/mcp.json`)
3. Test with Copilot: "What Dynatrace environment am I connected to?"

---

## �🚀 Quick Start

### Step 1: Configure for Your Client

Replace these placeholders throughout all files:

| Placeholder | Replace With | Example |
|-------------|--------------|---------|
| `[CLIENT_NAME]` | Customer name | "Acme Corporation" |
| `[TENANT_ID]` | Dynatrace tenant ID | "abc12345" |
| `[DATE]` | Current date | "30 January 2026" |
| `[INDUSTRY]` | Customer industry | "E-commerce" |
| `[WEBSITE_URL]` | Customer website | "www.acme.com" |

Or ask AI to Prompt for all and replace the placeholders where required

### Feature Flags

Three `.env` flags control AI assistant behaviour:

| Flag | Default | What It Does |
|------|---------|-------------|
| `MCP_GRAIL_ONLY=yes` | `yes` | **Gen 3 only** — AI uses only Grail DQL via MCP tools. Set to `no` to also enable Gen 2 USQL and classic API calls (requires `DT_GEN2_API_TOKEN`). |
| `MCP_USE_USER_VARIABLE=yes` | `yes` | **User tracking** — AI resolves `MCP_USER_ID` at session start and includes `user.id` on all tracking events. Set to `no` to skip user identity entirely. |
| `MCP_SEND_TRACKING_EVENTS=yes` | `yes` | **Query telemetry** — AI sends a CUSTOM_INFO event to Dynatrace after every MCP query. Set to `no` to disable all tracking events. |

### Step 2: Copy to New Location

```bash
# Copy this folder to your new client workspace
cp -r cleanDeploy /path/to/new/client/workspace/

# Or create a new workspace and copy files
mkdir -p /path/to/new/client/workspace/.github
cp cleanDeploy/.github/copilot-instructions.md /path/to/new/client/workspace/.github/
cp cleanDeploy/*.md /path/to/new/client/workspace/
```

### Step 3: Initial Data Discovery

Start your first session by running these FREE queries:
1. `find_entity_by_name` - Discover entities
2. `list_problems` - Check active problems
3. BizEvents summary query - Discover event types
4. Metrics discovery query - Find available metrics

### Step 4: Populate Reference Files

As you discover data, update the reference files:
- Add entities to `Entities_Reference.md`
- Add event types to `BizEvents_Reference.md`
- Add span patterns to `Spans_Reference.md`
- Add error patterns to `Logs_Reference.md`
- Add metrics to `Metrics_Reference.md`

---

## 📁 File Structure

```
cleanDeploy/
├── .env.example                   # Environment template (copy to .env)
├── .gitignore                     # Excludes .env from version control
├── .github/
│   └── copilot-instructions.md    # GitHub Copilot instructions (auto-loaded)
├── .vscode/
│   └── mcp.json                   # MCP server configuration
├── CLAUDE.md                      # Claude/Anthropic AI instructions (auto-loaded)
├── DATA_REFERENCE_INDEX.md        # Central index - START HERE
├── MCP_Query_Optimization_Guide.md # Query cost optimization
├── mcp_query_tracking_schema.md   # MCP telemetry event schema
├── Entities_Reference.md          # Cached entity IDs
├── BizEvents_Reference.md         # Business event types
├── Spans_Reference.md             # Trace/span patterns
├── Logs_Reference.md              # Log and error patterns
├── Metrics_Reference.md           # Available metrics (FREE)
├── AI_Prompt.md                   # Task templates
├── example/
│   ├── example_dashboard.json     # Sample dashboard template
│   └── MCP_Query_Usage_Dashboard.json # MCP usage monitoring dashboard
└── README.md                      # This file
```

---

## 📊 MCP Query Tracking

### Overview
When `MCP_SEND_TRACKING_EVENTS=yes` (default), all MCP queries are automatically tracked via `send_event` (CUSTOM_INFO events). This provides:
- Complete visibility into MCP query usage
- Cost tracking and budget monitoring
- User-level consumption analytics *(when `MCP_USE_USER_VARIABLE=yes`)*
- Query optimization insights

**When `MCP_SEND_TRACKING_EVENTS=no`:** Tracking is completely disabled. No events are sent.

### Setup
1. Set `MCP_SEND_TRACKING_EVENTS=yes` in your `.env` file (default)
2. Set `MCP_USER_ID` in your `.env` file to identify your queries *(if `MCP_USE_USER_VARIABLE=yes`)*
3. Import `example/MCP_Query_Usage_Dashboard.json` into Dynatrace
4. AI assistants will automatically send tracking events after each query

### How It Works
After every MCP query (execute_dql, list_problems, find_entity_by_name, etc.), the AI sends a tracking event:
```
event.name: "MCP Query Execution"
event.type: CUSTOM_INFO
properties:
  query.bytes_scanned: "0.84"
  query.cost_usd: "0.042"
  user.id: "your.email@company.com"
  ...
```

### Dashboard Queries
Query tracking events with:
```dql
fetch events
| filter event.type == "CUSTOM_INFO" and event.name == "MCP Query Execution"
```

See [mcp_query_tracking_schema.md](mcp_query_tracking_schema.md) for full event schema.

---

## 🎯 Core Principles

### 1. Cost Optimization
- **Query Priority:** FREE tools first, expensive queries last
- **Timeframes:** Start with 24h, extend only if needed
- **Filters:** Always filter by event.type/entity before other filters
- **Aggregation:** Use summarize, not raw data

### 2. Self-Learning
- **Update Reference Files:** After EVERY discovery, update the relevant file
- **Cache Entity IDs:** Never look up the same entity twice
- **Document Patterns:** Record what works for future sessions

### 3. Session Continuity
- **Read First:** Always read reference files before querying
- **Check Existing Data:** Data may already be documented
- **Incremental Updates:** Add new learnings to existing docs

---

## 📊 Query Cost Reference

| Query Type | Typical Cost | When to Use |
|------------|-------------|-------------|
| `find_entity_by_name` | 0 GB (FREE) | Always - first step |
| `list_problems` | 0 GB (FREE) | Problem investigation |
| `timeseries` metrics | 0 GB (FREE) | Performance trends |
| BizEvents (filtered, 7d) | 0.5-5 GB | Business analysis |
| Logs (loglevel filter, 24h) | 10-15 GB | Error investigation |
| Spans (entity filter, 24h) | 15-20 GB | Trace analysis |
| Spans (entity filter, 7d) | 100-130 GB | **AVOID** |
| Logs/Spans (unfiltered) | 300+ GB | **NEVER** |

---

## ✅ Checklist for New Deployments

- [ ] Copy `.env.example` to `.env`
- [ ] Configure `DT_ENVIRONMENT` with your tenant URL
- [ ] Configure `DT_PLATFORM_TOKEN` with required scopes
- [ ] Set feature flags (`MCP_GRAIL_ONLY`, `MCP_USE_USER_VARIABLE`, `MCP_SEND_TRACKING_EVENTS`)
- [ ] Verify MCP connection works ("What environment am I connected to?")
- [ ] Replace all `[PLACEHOLDER]` values in reference files
- [ ] Copy `.github/copilot-instructions.md` (for Copilot integration)
- [ ] Run initial entity discovery
- [ ] Run BizEvents summary query
- [ ] Populate `Entities_Reference.md` with discovered entities
- [ ] Populate `BizEvents_Reference.md` with event types
- [ ] Test a sample dashboard query
- [ ] (Optional) Copy example dashboards to `example/` folder

---

## 🔧 Customization

### Adding New Reference Categories

If your client has unique data types (e.g., custom metrics, specific integrations):

1. Create a new reference file: `CustomData_Reference.md`
2. Add to `DATA_REFERENCE_INDEX.md`
3. Add to `.github/copilot-instructions.md` file references
4. Follow the same self-updating protocol

### Industry-Specific Templates

Modify `AI_Prompt.md` to include:
- Industry-specific KPIs
- Common use case templates
- Client-specific terminology

---

## 📝 Maintenance

### Regular Updates
- Review and clean up reference files monthly
- Archive outdated patterns
- Update cost baselines as data volumes change

### Version Control
- Commit reference file updates frequently
- Tag stable versions before major changes
- Keep changelog of significant discoveries

---

## 🆘 Troubleshooting

### High Query Costs
1. Check if entity ID is cached in `Entities_Reference.md`
2. Verify using metrics instead of spans where possible
3. Reduce timeframe to 24h for exploration
4. Add more filters before executing

### Missing Data
1. Run discovery queries (event types, metrics)
2. Check semantic dictionary for available fields
3. Verify entity exists with `find_entity_by_name`

### Copilot Not Following Instructions
1. Ensure `.github/copilot-instructions.md` is in workspace root
2. Check file is properly formatted (Markdown)
3. Restart Copilot session

---

## 📚 Additional Resources

- [Dynatrace DQL Documentation](https://docs.dynatrace.com/docs/platform/grail/dynatrace-query-language)
- [Dynatrace Gen 3 Dashboards](https://docs.dynatrace.com/docs/observe-and-explore/dashboards-new)
- [Dynatrace BizEvents](https://docs.dynatrace.com/docs/platform/grail/data-model/business-events)
