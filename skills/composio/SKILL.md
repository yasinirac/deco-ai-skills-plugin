---
name: composio
description: |
  Connect AI agents to 1000+ external SaaS apps via Composio's tool router and MCP server. Provides managed OAuth, API-key vaults, scoped permissions, automatic token refresh, retries, and rate-limit handling so the agent never sees raw credentials. Covers communication (Slack, Discord, MS Teams, WhatsApp, Telegram), email (Gmail, Outlook), engineering (GitHub, GitLab, Linear, Jira, Sentry), productivity (Notion, Asana, Trello, ClickUp, Monday), CRM (HubSpot, Salesforce, Pipedrive, Zoho), data (Airtable, Google Sheets, Postgres, Supabase), storage (Google Drive, Dropbox, Box, OneDrive), calendar (Google Calendar, Outlook Calendar, Calendly), marketing (Mailchimp, Brevo, Klaviyo, ConvertKit), and 850+ more. Use when the user says any of: "send Slack message", "post to Slack channel", "search Slack", "send email", "send Gmail", "draft email", "read inbox", "create GitHub issue", "open GitHub PR", "comment on PR", "merge PR", "create Linear issue", "update Linear ticket", "update Notion page", "create Notion database row", "search Notion", "add Airtable record", "update Airtable", "fetch HubSpot contact", "create HubSpot deal", "update Salesforce lead", "search Salesforce", "create calendar event", "schedule meeting", "upload to Google Drive", "fetch from Drive", "post to Trello", "create Asana task", "update Jira issue", "send Discord message", "send Telegram message", "send WhatsApp", "send SMS via Twilio", "trigger n8n workflow", "automate cross-app workflow", "build agent that integrates X with Y", or any other "do X in <SaaS app>" request. Use when the user wants per-user multi-tenant connections (each end user authorizes their own account). Use when building production agents that must NOT leak credentials to the LLM. Skip when a first-class MCP for the same app is already attached in this session (prefer the dedicated MCP), when the task is a single action on a tool the user has already authenticated locally via that tool's own CLI, or when the action is read-only public web data (use WebFetch/WebSearch instead). Requires the Composio CLI installed and authenticated (`composio login`); for SDK use, also requires `composio init` inside the project to write a `COMPOSIO_API_KEY` to `.env`.
tags: [composio, tool-router, agents, mcp, tools, api, automation, cli, slack, gmail, github, notion, linear, hubspot, salesforce, airtable, calendar, crm, integrations]
metadata:
  requires-cli: true
  runtime: claude-code-or-terminal
---

# Composio

Composio is a managed integration layer that gives AI agents secure, brokered access to 1000+ external SaaS apps. Composio holds the OAuth tokens and API keys in an encrypted vault, exposes each app's API as a set of structured tools, and executes those tools on the agent's behalf so credentials never reach the LLM context window. It removes the work of writing per-app OAuth flows, handling token refresh, retrying failed calls, and managing rate limits.

There are two distinct modes:

1. **CLI mode** - the agent (you) calls `composio search`, `composio link`, and `composio execute` from the shell. No code is written. Best for one-off automations the user asks for in chat.
2. **SDK / MCP mode** - you write code that imports the Composio SDK or you connect the Composio MCP server, and your code/agent runtime calls tools. Best for productizing a workflow into an app, multi-tenant systems, or anything the user wants running on a server.

---

## When to use

Trigger this skill when the user asks for any of these in natural language:

- **Communication**: "post in #general on Slack", "DM Yasin on Slack saying...", "search Slack for messages about WETEX", "send a Discord ping when the deploy finishes"
- **Email**: "send an email to client@deco.ae with the proposal attached", "summarize unread Gmail in the marketing inbox", "draft a follow-up to last week's pitch thread"
- **Engineering**: "open a GitHub issue titled X in repo deco-ai/site", "list open PRs assigned to me", "comment on Linear DCO-142 saying we'll ship Friday", "close the Sentry issue once we deploy"
- **Project / docs**: "add a row to the Notion campaigns database with these fields", "create an Asana task in the WETEX project", "log this site visit as a Trello card"
- **CRM**: "log this lead in HubSpot with stage = qualified", "find all Salesforce contacts at Etihad Rail", "update the deal value on Pipedrive opportunity 4521"
- **Spreadsheets / data**: "append a row to the Airtable bid tracker", "pull last quarter's KPIs from Google Sheets", "update the SCADA status column"
- **Calendar / scheduling**: "book a 30-min meeting with john@etisalat.ae next Tuesday", "what's on my calendar today?", "send a Calendly link"
- **Files**: "upload this proposal to the client's Google Drive folder", "share the latest project deck via Dropbox"
- **Marketing**: "add this contact to the WETEX 2026 segment in Mailchimp", "trigger the Klaviyo welcome flow"
- **Multi-step automations**: "when a new lead comes into HubSpot, create a Slack thread in #sales and a Linear ticket in the bizdev project"
- **Building an agent / app**: "I'm writing a Python agent that needs Gmail + Calendar + Notion - set me up"

## When NOT to use

- A dedicated, first-class MCP server for the same target app is already loaded in this Claude session (e.g., a native Slack MCP, native Notion MCP). Prefer the native one - it usually returns richer types and lower latency.
- The user only needs read-only public web data ("scrape this page", "what does deco.ae say about EPC services"). Use `WebFetch`, `WebSearch`, or `agent-browser` instead.
- The user is using the agent-browser skill to drive a real browser session for the same app (e.g., they want to QA the Slack web UI). Don't double up.
- A single-app local CLI is faster and the user already authenticated it (e.g., `gh` for GitHub when they're already logged in via `gh auth login`). Use the local CLI unless the user wants the action logged in Composio's audit trail.
- The user wants a shell command, not an integration (e.g., "run npm install"). Use Bash.

## Prerequisites

1. **CLI installed.** Check: `composio whoami`. If "command not found":
   ```bash
   curl -fsSL https://composio.dev/install | bash
   ```
   Then restart the shell or `source ~/.bashrc` (or `~/.zshrc`).

2. **Authenticated.** First time only:
   ```bash
   composio login           # opens browser, picks org/project
   composio whoami          # confirm org_id, project_id, user_id
   ```
   For headless agents that can't open a browser, use the share-link flow:
   ```bash
   composio login --no-wait | jq    # prints URL + cli_key
   # share URL with the user, wait for them to complete in their browser
   composio login --key <cli_key> --no-wait
   ```

3. **For SDK/code use** (mode 2), inside the project root:
   ```bash
   composio init            # writes COMPOSIO_API_KEY to .env
   ```
   Then `pip install composio` (Python) or `npm i composio-core` (Node), depending on stack.

4. **App-level connection.** Each external app the user wants to use must be linked once per user:
   ```bash
   composio link gmail                  # interactive
   composio link slack --no-wait        # for headless agents - returns URL to share
   ```

## Workflow

The canonical pattern is **search -> link (if needed) -> execute**.

1. **Search for the right tool by use case**, not by guessing slugs:
   ```bash
   composio search "send a message to a slack channel"
   composio search "create a row in airtable"
   composio search "list calendar events for the next 7 days"
   ```
   Returns ranked tool slugs with parameter docs.

2. **Verify the user has connected that app.** If not, `composio link <toolkit>` first.

3. **Execute with structured JSON input**:
   ```bash
   composio execute "SLACK_SEND_MESSAGE" -d '{"channel":"#bizdev","text":"WETEX prospect call confirmed for Wed 2pm GST"}'
   composio execute "GMAIL_SEND_EMAIL" -d '{"to":"client@etihadrail.ae","subject":"Proposal v2","body":"Hi, attaching the revised SCADA scope...","attachments":["./proposals/v2.pdf"]}'
   composio execute "GITHUB_CREATE_ISSUE" -d '{"owner":"deco-ai","repo":"deco-site","title":"Add WETEX 2026 landing page","body":"See linear.app/deco/DCO-201","labels":["marketing","p1"]}'
   ```

4. **For triggers / event listeners** (e.g., "ping me when a new email arrives"):
   ```bash
   composio listen           # streams real-time events from connected apps
   ```
   Configure trigger filters via `composio triggers enable <slug>`.

5. **For SDK code**, the pattern is:
   ```python
   from composio import Composio
   composio = Composio()
   tools = composio.tools.get(user_id="user_123", toolkits=["GMAIL", "SLACK", "NOTION"])
   # pass tools to your LLM (Anthropic, OpenAI, LangChain, etc.) - Composio handles auth + execution
   ```

## Common toolkits (one-line use cases)

DECO will most often touch these. Slug names are stable; use them directly with `composio execute <SLUG>` after `composio search` confirms.

| Toolkit | Common DECO use case |
|---|---|
| **Slack** | Drop campaign updates into `#bizdev` or `#marketing`, post weekly KPI digests, search threads for past client convos. |
| **Gmail** | Send proposal emails to UAE clients (Etihad Rail, ADNOC, DEWA), draft cold outreach to RTA project leads, summarize unread bid invitations. |
| **Outlook** | Same as Gmail, for clients on Microsoft 365 (most UAE government and semi-gov entities). |
| **GitHub** | Open issues for the deco.ae site repo, list PRs awaiting review, comment on engineering tasks. |
| **GitLab** | Same as GitHub, for clients/contractors using self-hosted GitLab on UAE infrastructure. |
| **Linear** | Create / update Linear issues in the marketing or bizdev projects, log content backlog from a brainstorm session. |
| **Jira** | For engineering teams (DECO field engineers, EPC project managers) tracking SCADA/EI&I deliverables. |
| **Notion** | Update the WETEX 2026 campaign tracker, add case study drafts to the project storytelling DB, search the brand guideline pages. |
| **Asana** | Track multi-week marketing campaigns and approval workflows. |
| **Trello** | Lightweight content calendar boards (LinkedIn posts, social schedule). |
| **HubSpot** | Push new leads from website forms, update lifecycle stage, log calls and emails against contacts. |
| **Salesforce** | For larger UAE enterprise pursuits - log opportunities, update stage, fetch contact lists for ABM campaigns. |
| **Pipedrive** | Lighter CRM for boutique pursuits and SME clients. |
| **Airtable** | Bid tracker, content calendar, project portfolio - DECO's "small-but-structured" data store of choice. |
| **Google Sheets** | Quick KPI dashboards, sponsorship ROI rollups, partner/vendor lists. |
| **Google Drive** | Upload final proposals and case studies to client-shared folders, fetch brand assets. |
| **Dropbox / Box / OneDrive** | Same as Drive, for clients on those stacks. |
| **Google Calendar / Outlook Calendar** | Schedule client calls, find availability across the team, book WETEX booth meetings. |
| **Calendly** | Generate booking links for prospect intro calls, sync availability. |
| **Discord** | Community / partner channels, internal team pings. |
| **Telegram / WhatsApp** | UAE B2B reality - many vendors and clients respond on WhatsApp first. |
| **Mailchimp / Brevo / Klaviyo** | Campaign sends, segment management, automation flows for newsletter and product launches. |
| **Sentry** | Track production errors on deco.ae or client portals; auto-create Linear issues on new error groups. |
| **Stripe** | Pull invoice/payment status, reconcile against Airtable bid tracker. |
| **Twilio** | Programmatic SMS for OTP / event reminders. |

Run `composio search` for anything not listed - the catalog is 1000+ apps.

## Examples (DECO scenarios)

**Scenario 1 - WETEX outreach blast.**
User: "Send the WETEX 2026 booth invite to the 12 contacts in our Airtable 'WETEX Targets' base, and log each send as a HubSpot activity."

Plan:
1. `composio search "list records from airtable base"` -> `AIRTABLE_LIST_RECORDS`.
2. `composio execute AIRTABLE_LIST_RECORDS -d '{"base_id":"appXXXX","table":"WETEX Targets"}'`.
3. For each record, `composio execute GMAIL_SEND_EMAIL` with personalized subject/body.
4. For each send, `composio execute HUBSPOT_LOG_EMAIL_ACTIVITY -d '{"contact_id":"...","subject":"...","body":"..."}'`.

**Scenario 2 - LinkedIn campaign sync to internal trackers.**
User: "Whenever we publish a LinkedIn post via the deco-linkedin-content-engine skill, also create a Notion page in the 'Published Posts' DB and post a confirmation in #marketing on Slack."

Plan:
1. After the LinkedIn post is drafted/scheduled, in one chained call:
   - `composio execute NOTION_CREATE_DATABASE_ROW -d '{"database_id":"...", "properties": {...}}'`
   - `composio execute SLACK_SEND_MESSAGE -d '{"channel":"#marketing","text":"Posted: <link>"}'`

**Scenario 3 - EPC project status digest.**
User: "Every Monday morning, pull this week's tasks from Linear's 'EI&I Substations' project, the open Sentry alerts on the SCADA dashboard, and the upcoming meetings from my Google Calendar, and DM me a summary in Slack."

This is a recurring agent. Use the SDK + the `schedule` skill or a cron, fetching from Linear / Sentry / Calendar via Composio tools and posting via SLACK_SEND_DIRECT_MESSAGE.

## Common pitfalls

- **Forgetting to link the toolkit.** `composio execute` returns an auth error if the user hasn't run `composio link <toolkit>` for that app yet. Always check `composio whoami` and `composio connections list` first if errors say "no active connection."
- **Wrong slug.** Tool slugs are stable but spelling matters - always confirm via `composio search` before guessing. Don't make up slugs from training data.
- **Over-broad scopes.** When linking OAuth apps, use the most restrictive scope set the task needs. Don't link Gmail with full mailbox scope just to send one email.
- **Headless mode for agents.** When the agent has no browser (this Claude session), always pass `--no-wait` to `composio link` and `composio login`. Otherwise the CLI will block forever waiting for a browser handshake.
- **Multi-tenant code paths.** In SDK mode, always pass `user_id` to `composio.tools.get()`. Otherwise tools execute under the developer's account, which is wrong for any user-facing app.
- **Rate limits.** Composio retries automatically but has caps. For bulk operations (>50 calls), batch with delays or use the SDK's `bulk_execute` rather than looping single executes.
- **Don't expose tokens.** Never `cat ~/.composio/...` into the chat. Composio's whole value is keeping tokens out of the LLM context.
- **Production OAuth apps.** For multi-user SaaS, set up your own OAuth developer app per toolkit (your client_id / client_secret in Composio's auth config) rather than Composio's default app. Better scope control and brand-correct consent screen.

## Related skills

- **agent-browser** - when you need to drive the actual web UI of an app (e.g., Slack web client) instead of its API. Use Composio first; fall back to agent-browser only if the API doesn't expose what you need.
- **deco-master-marketing-agent** / **deco-linkedin-content-engine** / **deco-email-pr-communications** - upstream skills that produce content; Composio is the delivery channel.
- **schedule** / **loop** - for recurring Composio-driven automations (Monday digests, daily lead syncs).
- **mcp-builder** - if you outgrow Composio for a specific app and want to write a custom MCP server.
- **graphify** - record cross-app workflows as a knowledge graph for later reuse.

> Full reference: see `rules/composio-cli.md` (CLI deep-dive) and `rules/building-with-composio.md` (SDK / MCP code patterns).
