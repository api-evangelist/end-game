---
name: Endgame
description: Use when building AI-powered sales intelligence workflows, connecting CRM and conversation data to AI assistants, creating reusable skills for sales teams, querying account and pipeline data, or embedding Endgame into custom applications via API or MCP.
metadata:
    mintlify-proj: endgame
    version: "1.0"
---

# Endgame

## Product summary

Endgame is a revenue intelligence platform that builds a context graph from your CRM, emails, calls, Slack, and uploaded documents, then exposes that graph to AI assistants via MCP (Model Context Protocol), REST API, or CLI. Agents use Endgame to search accounts, analyze conversations, surface pipeline risks, and execute repeatable workflows (called skills). The primary integration point is the MCP server at `https://app.endgame.io/api/v1/mcp`, which connects to Claude, ChatGPT, Claude Code, Codex, and Claude Managed Agents. For direct terminal access, use the Endgame CLI (`endgame` command). For programmatic access, use the REST API at `https://app.endgame.io/api/v1/threads`. See the [primary docs site](https://docs.endgame.io) for full reference.

## When to use

Reach for Endgame when you need to:

- **Query sales data across multiple sources** — Ask about accounts, opportunities, contacts, and recent interactions without switching between CRM, email, and call recordings
- **Prepare for customer meetings** — Generate pre-call briefs, stakeholder maps, and competitive intelligence from your organization's data
- **Analyze pipeline health** — Identify stalled deals, coverage gaps, and at-risk opportunities with structured reports
- **Create repeatable sales workflows** — Build skills (reusable instruction sets) for deal inspections, meeting prep, follow-up emails, or pipeline reviews
- **Embed intelligence into your own tools** — Use the REST API or MCP to add Endgame's context graph to custom applications, agents, or automations
- **Schedule automated reports** — Set up daily, weekly, or monthly email digests that run prompts on a schedule
- **Integrate with AI assistants** — Connect Claude, ChatGPT, or your own agents to Endgame so they can access your full sales context

## Quick reference

### Connection methods

| Method | Use case | Setup |
|--------|----------|-------|
| **MCP (Claude)** | Claude users who want Endgame tools in conversations | Admin adds connector at `https://app.endgame.io/api/v1/mcp`, users connect individually |
| **MCP (ChatGPT)** | ChatGPT users who want Endgame tools in conversations | Admin creates custom app, publishes to org, users enable |
| **MCP (Claude Code/Codex)** | Terminal-based workflows with Endgame tools | Run `claude mcp add` or `codex mcp add` with Endgame URL |
| **REST API** | Embed threads in custom apps or automations | Create user or service account API key, POST to `/api/v1/threads` |
| **CLI** | Direct terminal access to Endgame tools | Install Go binary, run `endgame auth login`, invoke tools with `endgame tools` |
| **Web app** | Interactive exploration and chat | Log in at `https://app.endgame.io` |

### Core CLI commands

```bash
endgame auth login                                    # Authenticate (browser or device-code mode)
endgame auth status                                   # Check authentication state
endgame tools --help                                  # List available tools
endgame tools search_graph_entities --json '{...}'   # Search accounts, people, deals
endgame tools list_my_accounts --json '{}'           # List your accounts
endgame tools get_graph_entities --json '{...}'      # Get full record for entity
```

### REST API endpoints

| Method | Path | Purpose |
|--------|------|---------|
| `POST` | `/api/v1/threads` | Create a thread (start a conversation) |
| `GET` | `/api/v1/threads/{id}` | Get thread and all messages (poll for completion) |
| `GET` | `/api/v1/threads` | List threads (with pagination) |
| `PATCH` | `/api/v1/threads/{id}` | Rename a thread |
| `DELETE` | `/api/v1/threads/{id}` | Soft-delete a thread |

### API authentication

- **User-scoped key** (`eak_*`): Tied to a person, can create/rename/delete own threads, scoped to visible data
- **Service account key** (`eak_*`): Org-wide, user-agnostic, for automated workflows and Claude Managed Agents
- **M2M credentials**: OAuth client_credentials grant for short-lived tokens (production server-to-server)

Create keys at `https://app.endgame.io/settings/api-keys` (admins only).

### Key file paths and settings

| Location | Purpose |
|----------|---------|
| `https://app.endgame.io/settings/integrations` | Connect CRM, transcripts, Slack, knowledge sources |
| `https://app.endgame.io/settings/skills` | Create and manage reusable skills |
| `https://app.endgame.io/settings/api-keys` | Create API keys for REST API and MCP |
| `https://app.endgame.io/settings/rules` | Define terminology mappings and data presentation rules |
| `~/.endgame-auth.json` | CLI auth token (auto-managed, do not edit) |

## Decision guidance

### When to use MCP vs REST API vs CLI

| Scenario | Use MCP | Use REST API | Use CLI |
|----------|---------|-------------|---------|
| Claude/ChatGPT user asking conversational questions | ✅ | — | — |
| Embed Endgame in a custom web app | — | ✅ | — |
| Automated backend workflow (cron, webhook) | — | ✅ | — |
| Developer scripting from terminal | — | — | ✅ |
| Claude Code or Codex session | ✅ | — | ✅ |
| Claude Managed Agents (automated agents) | ✅ (service account) | — | — |

### When to create a skill vs use ad hoc chat

| Situation | Create a skill | Use ad hoc chat |
|-----------|----------------|-----------------|
| Task is repeatable and done weekly+ | ✅ | — |
| Output format must be consistent | ✅ | — |
| Multiple team members do the same task | ✅ | — |
| One-off research or exploration | — | ✅ |
| Task is novel or changes frequently | — | ✅ |
| Need to onboard new reps on a process | ✅ | — |

### When to upload knowledge vs rely on CRM

| Content type | Upload to knowledge | Rely on CRM |
|--------------|-------------------|-----------|
| Sales methodology, playbooks, battlecards | ✅ | — |
| Competitive positioning, win/loss analysis | ✅ | — |
| Account plans, strategy docs | ✅ | — |
| CRM records, opportunity data | — | ✅ |
| Call transcripts, emails | — | ✅ (auto-ingested) |
| Custom pricing or product guides | ✅ | — |

## Workflow

### Typical task: Create a skill for deal inspection

1. **Identify the repeatable workflow** — Ask your team: What do managers check during every deal review? What questions come up repeatedly? Pick one that wastes time or produces inconsistent results.

2. **Navigate to skill creation** — Go to `https://app.endgame.io/settings/skills`, click **+ Add Skill**, select **Write a Skill**.

3. **Structure the skill content** — Use this template:
   - **Goal**: One sentence describing the outcome (e.g., "Run a health check on an opportunity and surface risks")
   - **Inputs needed**: List the data sources (Salesforce fields, call transcripts, Slack threads)
   - **Instructions**: Numbered steps with specific thresholds (e.g., "Flag if no activity in 14+ days")
   - **Output format**: Specify structure, length, and tone (e.g., "Lead with health rating: Strong/At Risk/Needs Attention")

4. **Fill in metadata** — Title (100 chars max), description (500 chars, lead with outcome), example prompt (optional, 500 chars).

5. **Test against 3–5 accounts** — Run the skill in chat, check accuracy, iterate on wording. Small phrasing changes improve output quality.

6. **Publish and share** — Once tested, the skill is visible to your team in the skill library. Users invoke it by name in chat or via MCP.

### Typical task: Query account data via MCP in Claude

1. **Ensure Endgame connector is added** — Admin navigates to Claude's custom connector setup, adds `https://app.endgame.io/api/v1/mcp`, authenticates.

2. **Individual user connects** — Go to **Customize** → **Connectors** in Claude, find Endgame, click **Connect**, follow auth prompts.

3. **Ask a question** — In Claude, ask naturally: "What accounts does my team own?" or "Show me all open opportunities over $500K with no activity in 45 days." Claude automatically selects the right Endgame tools.

4. **Claude chains tools if needed** — For complex queries, Claude may call multiple tools (search account, get relationships, query data warehouse) and synthesize the answer.

### Typical task: Create a thread via REST API

1. **Get an API key** — Go to `https://app.endgame.io/settings/api-keys`, create a user-scoped or service account key, copy immediately.

2. **POST a prompt** — Send a request:
   ```bash
   curl -X POST https://app.endgame.io/api/v1/threads \
     -H "Authorization: Bearer eak_your_key" \
     -H "Content-Type: application/json" \
     -d '{"prompt": "What accounts does my team own?"}'
   ```

3. **Poll for completion** — Use the returned thread ID to poll `GET /api/v1/threads/{id}` until `status.state` is `idle` or `error`.

4. **Parse the response** — Extract the assistant's message from the `messages` array where `role` is `assistant` and `status` is `completed`.

## Common gotchas

- **Skill content is too vague** — "Include everything important" produces generic output. Be explicit: list exact fields, thresholds, and edge cases. Example: "Flag if close date has been pushed more than once" beats "flag risky deals."

- **Forgetting to test skills before publishing** — Always run a skill against 3–5 real accounts before sharing. Output quality varies based on data completeness; iteration is normal.

- **API key scope mismatch** — User-scoped keys can only create/rename/delete their own threads. Service account keys are org-wide but can't be used for personal thread operations. Choose the right scope upfront.

- **Not uploading knowledge documents** — Endgame's responses feel generic if it only has CRM data. Upload your sales methodology, battlecards, and account plans so the context graph reflects how your team actually sells.

- **Ignoring rules and terminology mapping** — If your team says "ARR" but Salesforce calls it `Annual_Recurring_Revenue__c`, create a rule so Endgame uses the right field. Without rules, responses miss obvious context.

- **Rate limit surprises** — REST API has a 10,000 request/day org-wide limit (open beta). If you're building high-volume automations, monitor usage and contact support to adjust.

- **CLI token expiry** — Tokens in `~/.endgame-auth.json` refresh automatically, but if you see auth errors, run `endgame auth login` again.

- **MCP tool list stale** — Endgame periodically adds tools. If Claude doesn't see a new tool, go to the connector settings and click **Refresh Tools List**.

- **Skill references other skills but they don't exist** — Skills can reference other skills by name, but if the referenced skill is deleted or renamed, the reference breaks silently. Test after renaming.

- **Digests not sending** — Digests run on a schedule but require at least one data source connected (CRM, transcripts, Slack). If a digest produces no output, check that your integrations are active.

## Verification checklist

Before submitting work with Endgame:

- [ ] **Skill tested** — Ran the skill against 3+ real accounts and verified output accuracy and format
- [ ] **Skill description clear** — Description leads with outcome, mentions trigger conditions, and is under 500 characters
- [ ] **Skill content explicit** — Instructions include specific field names, thresholds, and edge case handling (not vague wishes)
- [ ] **API key scoped correctly** — User-scoped for personal threads, service account for automated workflows
- [ ] **Knowledge uploaded** — Sales methodology, battlecards, and account plans are in the knowledge section
- [ ] **Rules created** — Terminology mappings and CRM field mappings are defined (not relying on Endgame to guess)
- [ ] **MCP connector refreshed** — If adding new tools, refreshed the tool list in Claude/ChatGPT settings
- [ ] **Rate limits monitored** — For high-volume API usage, confirmed org is not approaching 10K request/day limit
- [ ] **Digests configured** — If using scheduled digests, verified at least one integration is active and preview sent successfully

## Resources

- **Full page navigation**: See [https://docs.endgame.io/llms.txt](https://docs.endgame.io/llms.txt) for comprehensive page-by-page listing
- **MCP Server & AI Integration**: [https://docs.endgame.io/features/mcp-server](https://docs.endgame.io/features/mcp-server) — Connect Claude, ChatGPT, Claude Code, or Codex
- **Skills: Composing Skills**: [https://docs.endgame.io/skills/skills-101](https://docs.endgame.io/skills/skills-101) — How to write effective skill instructions
- **REST API Reference**: [https://docs.endgame.io/api-reference/endpoints](https://docs.endgame.io/api-reference/endpoints) — Thread CRUD operations and error handling

---

> For additional documentation and navigation, see: https://docs.endgame.io/llms.txt