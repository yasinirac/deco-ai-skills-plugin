---
name: agent-browser
description: Fast Rust browser automation CLI for AI agents. Drives Chrome via CDP, returns compact accessibility-tree snapshots with stable @eN element refs. Use when the user says "open URL", "browse to", "navigate to", "fill out form", "click button", "take screenshot", "scrape data", "extract from page", "test login", "QA this page", "automate website", "automate Slack/VS Code/Figma/Notion/Discord", "check Slack unreads", "drive Electron app", "run in sandbox", or any task needing programmatic interaction with a rendered web page or Electron window. Skip when a JSON API works (prefer API over UI scraping), when the user wants pure HTML (use WebFetch), or when a dedicated browser MCP is already attached. Requires the `agent-browser` CLI installed globally (`npm i -g agent-browser && agent-browser install`).
tags: [browser, automation, qa, scraping, testing, cdp, chrome, electron, slack, sandbox, e2e]
metadata:
  requires-cli: true
  runtime: claude-code-or-terminal
---

# agent-browser

Fast browser automation CLI for AI agents. Drives Chrome/Chromium via CDP, returns accessibility-tree snapshots with compact `@eN` element refs that stay stable across actions in the same tab.

## When to use

Trigger when the user asks for any of:

- **Navigation / inspection**: "open https://deco.ae", "what does the WETEX site show right now?", "screenshot the homepage", "what's on the bid portal login page?"
- **Form filling**: "log into the client portal with this username/password", "fill out the bid registration form on TenderBoard", "submit the contact form on deco.ae as a smoke test"
- **Click + flow**: "click the 'Apply Now' button", "go through the WETEX exhibitor signup flow and tell me where it breaks", "test the 3-step proposal request wizard"
- **Scraping**: "extract the project list from this Etihad Rail tender page", "pull the contact emails from this exhibitor directory", "read the SCADA spec sheet and grab the key parameters"
- **Visual / regression QA**: "screenshot every page in the deco.ae sitemap", "compare today's homepage to yesterday's", "record a video of the contact form being submitted"
- **Login / auth flows**: "test that SSO still works on the client portal", "verify the password reset email link still works"
- **Slack/Discord/Notion/Figma desktop automation**: "check my Slack unreads in the DECO workspace", "send a message to #bizdev via the Slack desktop app", "open the Figma board and screenshot the latest design"
- **Cross-page workflows**: tab management, multi-step flows that span login -> dashboard -> action -> verify
- **Sandbox / cloud execution**: "run this browser flow in a Vercel Sandbox microVM" or "use AgentCore cloud browser so the user's session isn't touched"
- **Dogfooding / exploratory testing**: "click around the new landing page and tell me what feels broken"

## When NOT to use

- The data is reachable via a public REST/GraphQL API. Use `WebFetch` or write a one-shot fetch script.
- A Composio toolkit covers the action (e.g., "send a Slack message" - prefer Composio's `SLACK_SEND_MESSAGE` over driving the Slack web UI).
- The user only wants to read static HTML / no JS rendering required. Use `WebFetch`.
- The task is a search-the-public-web question. Use `WebSearch`.
- A `Claude in Chrome` or `Claude Preview` MCP is already loaded and the user wants to test their own app's visual artifact. Those have richer screenshot + DOM tooling.
- The user wants automated unit/integration tests for their own product. Recommend Playwright/Vitest/Cypress in their repo, not agent-browser.
- Anything involving real money movement, real client production accounts, or destructive actions on a UAE-government client portal without explicit confirmation. Always escalate first.

## Prerequisites

```bash
npm i -g agent-browser
agent-browser install            # downloads managed Chrome
agent-browser --version          # confirm install
```

For sandboxed / cloud execution, additionally:
- **Vercel Sandbox**: Vercel account + sandbox token in env.
- **AWS Bedrock AgentCore**: AWS creds + AgentCore region enabled.

For the Electron / Slack specialized skills, the target desktop app must already be installed and ideally already signed in.

**Always load the live workflow content from the CLI before running commands** - it matches the installed version exactly and won't go stale:

```bash
agent-browser skills get core            # workflows, common patterns, troubleshooting
agent-browser skills get core --full     # full command reference + templates
agent-browser skills list                # see all specialized skills available
```

Specialized skill packs:

```bash
agent-browser skills get electron        # Electron desktop apps (VS Code, Slack, Discord, Figma, Notion, Spotify)
agent-browser skills get slack           # Slack workspace automation patterns
agent-browser skills get dogfood         # exploratory testing / QA / bug hunts
agent-browser skills get vercel-sandbox  # run in Vercel Sandbox microVMs
agent-browser skills get agentcore       # AWS Bedrock AgentCore cloud browsers
```

## Workflow (canonical loop)

The basic loop for any web task:

1. **Open the page** -> `agent-browser open <url>`
2. **Snapshot** -> `agent-browser snapshot -i` returns the accessibility tree with refs (`@e1`, `@e2`, ...) for every interactive element. Read the snapshot, not the raw HTML.
3. **Act** by ref, not selector:
   - `agent-browser click @e3`
   - `agent-browser fill @e7 "yasin@deco.ae"`
   - `agent-browser type @e7 "..."` (slower, simulates keystrokes; use only when fill fails)
   - `agent-browser select @e9 "United Arab Emirates"`
   - `agent-browser check @e2` / `uncheck @e2`
   - `agent-browser hover @e5`, `focus`, `press`, `keydown/keyup`
   - `agent-browser scroll`, `scrollintoview @e12`
   - `agent-browser drag @e1 @e2`
   - `agent-browser upload @e4 ./proposal.pdf`
4. **Re-snapshot** after each action. Refs stay valid within a tab until the DOM changes meaningfully; re-snapshot when in doubt.
5. **Verify / extract** -> `agent-browser screenshot ./out.png`, `agent-browser snapshot --text-only`, or `agent-browser eval "..."` for one-off JS.
6. **Close** -> `agent-browser close` (single tab) or `agent-browser quit` (whole browser).

## Common command patterns

```bash
# Tabs
agent-browser tab new --label docs https://docs.example.com
agent-browser tab list
agent-browser tab switch docs
agent-browser tab close docs

# Sessions / state persistence (keep cookies + auth across runs)
agent-browser session save deco-portal
agent-browser session load deco-portal

# Auth vault (store creds outside the LLM context)
agent-browser auth set deco-portal --user yasin --pass-from-env DECO_PORTAL_PASS
agent-browser auth use deco-portal

# Recording (great for QA bug reports)
agent-browser record start ./bug-repro.webm
# ... do the actions ...
agent-browser record stop

# Run JS in the page when CLI primitives aren't enough
agent-browser eval "JSON.stringify([...document.querySelectorAll('a.tender')].map(a => ({text: a.innerText, href: a.href})))"

# iframes
agent-browser iframes list
agent-browser snapshot --frame <frame-id>
```

## Examples (DECO scenarios)

**Scenario 1 - Tender portal smoke test.**
User: "Verify our login on TenderBoard.ae still works and screenshot the dashboard."
1. `agent-browser open https://tenderboard.ae/login`
2. `agent-browser snapshot -i` -> identify username/password/submit refs.
3. `agent-browser auth use tenderboard-deco` (creds from vault, never in chat).
4. `agent-browser fill @e1 "$DECO_TB_USER"` (or via auth use); `fill @e2 "..."`; `click @e3`.
5. `agent-browser snapshot --text-only` to confirm "Welcome, DECO" text present.
6. `agent-browser screenshot ./tb-dashboard-$(date +%Y%m%d).png`.

**Scenario 2 - Scrape competitor exhibitor list from WETEX.**
User: "Pull the list of EPC exhibitors from the WETEX 2026 directory."
1. `agent-browser open https://wetex.ae/exhibitors?category=EPC`
2. Loop: `snapshot -i` -> find next-page ref -> extract via `eval` -> click next -> repeat.
3. Save to `./wetex-epc-exhibitors.json`.
4. Hand the file to the deco-content-prioritizer skill for outreach scoring.

**Scenario 3 - Visual regression on deco.ae.**
User: "Screenshot every page in the deco.ae sitemap so I can review before tomorrow's launch."
1. Fetch sitemap.xml (use WebFetch, not agent-browser, for the XML).
2. For each URL, `agent-browser open <url>` -> `screenshot ./regression/<slug>.png`.
3. Optionally `record start` for a flythrough video for the client demo.

**Scenario 4 - Slack desktop check.**
User: "What unreads are in #bizdev right now?"
1. Load the slack specialized skill: `agent-browser skills get slack`.
2. Follow its pattern: open the desktop Slack app via Electron entry point, navigate to channel, snapshot the unread message list, return summarized text.

## Common pitfalls

- **Don't guess CSS selectors.** Always work from the snapshot's `@eN` refs. Selectors break across renders; refs are computed from the accessibility tree and stay aligned.
- **Always re-snapshot after navigation.** Refs from the prior page are invalid after a route change.
- **Token bloat.** A full snapshot of a complex page can be huge. Use `snapshot --text-only`, `snapshot --max-depth 5`, or scope to a frame to keep context lean.
- **Don't paste credentials into chat.** Use `agent-browser auth set ... --pass-from-env` or load from a `.env` file. Never echo passwords into the conversation.
- **Headless vs headed.** Default is headed for visual debugging. Pass `--headless` for CI / agents running unattended. Some sites detect headless mode and behave differently - have a headed fallback.
- **Cookie / login persistence.** Without `session save`, every run starts cold. For multi-step flows that need a logged-in state, save the session on first auth and load it on subsequent runs.
- **Rate limits / bot detection.** Some UAE government portals (and many SaaS apps) flag rapid automated access. Throttle with `sleep` between actions, set a realistic user-agent, and use `record` to prove the flow is benign if you get blocked.
- **iframes are common.** Sign-in widgets, embedded forms, and chat widgets often live in iframes - explicitly list and target frames; snapshots default to the top frame only.
- **Don't run on real client production accounts.** Use staging or read-only views. Confirm with the user before any destructive action on a UAE-client production system.

## Why agent-browser

- Fast native Rust CLI; no Node runtime overhead at execution.
- ~4x more token-efficient than Playwright MCP because the snapshot is a compact accessibility tree, not raw HTML.
- Works with any AI agent (Claude Code, Cursor, Codex, Continue, Windsurf).
- Element refs are stable across actions within a tab - safer than CSS selectors.
- First-class support for tabs, iframes, sessions, auth vault, state persistence, video recording.
- Specialized skill packs for Electron desktop apps, Slack, exploratory QA, and cloud browsers (Vercel Sandbox, AWS Bedrock AgentCore).

## Related skills

- **composio** - prefer Composio's API toolkits when the action is achievable via API (sending a Slack message, creating a GitHub issue). Use agent-browser only when you need the actual UI.
- **deploy-to-vercel** / **vercel-cli-with-tokens** - separate concern; agent-browser can verify the deployed page after a Vercel deploy.
- **accesslint-audit** / **web-design-guidelines** / **design-review** - pair with agent-browser screenshots for accessibility and design QA.
- **graphify** - feed scraped page structures into a knowledge graph for later querying.
- **schedule** / **loop** - for recurring monitoring (daily smoke test of deco.ae, weekly tender scrape).

> Always run `agent-browser skills get core` at the start of a session to load the version-matched workflow guide before issuing commands. The CLI's content is the source of truth and updates with each release.
