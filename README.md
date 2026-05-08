# DECO AI Skills Plugin

> 86 curated skills for AI agents — engineering, marketing, design, and content creation.

A production-ready Claude plugin assembled, audited, and improved specifically for [DECO AI](https://deco.ae) workflows. All skills have been reviewed for trigger reliability, duplicates removed, and weak descriptions deeply rewritten with current 2026 best practices.

---

## Installation

### Option 1: Cowork (recommended for teams)

1. Push this folder to a Git repository (GitHub, GitLab, Bitbucket).
2. In a Cowork session, run:
   ```
   /plugin marketplace add <your-username>/<repo-name>
   /plugin install deco-ai-skills@<your-username>/<repo-name>
   ```
3. All 86 skills become available to every member of the workspace.

### Option 2: Claude Code (local)

```bash
# clone or unzip into a known location
cd /path/to/deco-ai-skills-plugin
/plugin install local
```

### Option 3: Direct copy (simplest)

Copy the `skills/` folder into `~/.claude/skills/`:

```bash
cp -r skills/* ~/.claude/skills/
```

---

## What's inside

### 🎯 Marketing (40 skills)
**Conversion Optimization (CRO):** ab-test-setup, form-cro, onboarding-cro, page-cro, paywall-upgrade-cro, popup-cro, signup-flow-cro

**SEO:** ai-seo, programmatic-seo, schema-markup, seo-audit, site-architecture, competitor-alternatives

**Acquisition:** cold-email, paid-ads, ad-creative, directory-submissions, lead-magnets, free-tool-strategy

**Retention:** churn-prevention, customer-research, email-sequence

**Strategy:** content-strategy, marketing-ideas, marketing-psychology, launch-strategy, pricing-strategy, product-marketing-context, competitor-profiling, co-marketing, community-marketing, referral-program, revops, sales-enablement

**Content:** copywriting, copy-editing, image, social-content, video, analytics-tracking, aso-audit

### 🎨 Design (10 skills)
frontend-design, ui-ux-pro-max, web-design-guidelines, design-flow, design-brief, design-review, design-tokens, information-architecture, awesome-design-md, brief-to-tasks

### 💻 Engineering (20 skills)
**React/Next.js:** react-best-practices, react-native-skills, react-view-transitions, composition-patterns

**Workflow:** tdd, diagnose, prototype, improve-codebase-architecture, triage, to-issues, to-prd, agent-orchestration

**Tooling:** deploy-to-vercel, vercel-cli-with-tokens, setup-pre-commit, git-guardrails-claude-code, migrate-to-shoehorn, scaffold-exercises, accesslint-audit, find-skills, setup-matt-pocock-skills

### ✍️ Writing (5 skills)
writing-shape, writing-beats, writing-fragments, grill-me, grill-with-docs

### 🤖 Automation (5 skills)
agent-browser, composio (1000+ SaaS integrations), graphify, remotion-best-practices, obsidian-vault

### 🛠️ Meta/Utility (6 skills)
skill-creator, caveman (token compression), handoff, zoom-out

---

## Audit & improvements applied

| Action | Count | Details |
|---|---|---|
| **Skills audited** | 90 → 86 | Full frontmatter + body review |
| **Consolidated** | 4 archived | bencium-controlled, bencium-innovative, write-a-skill, edit-article |
| **Frontmatter fixed** | 5 | vercel-* prefix mismatches and graphify-windows |
| **Deeply rewritten** | 4 | composio, agent-browser, remotion-best-practices, scaffold-exercises |
| **Disambiguated** | 4 | IA pair (app vs site) + grill pair (greenfield vs codebase-aware) |
| **Metadata-tagged** | 32 | `runtime: claude-code-only` or `requires-cli: true` |

The 4 archived skills are preserved at `~/Desktop/skills-archive/` if you want to restore any.

---

## Skill metadata flags

Skills with these tags need extra context:

- `runtime: claude-code-only` — won't work in Claude.ai chat (needs filesystem)
- `requires-cli: true` — needs an external CLI tool installed (graphify, agent-browser, vercel CLI, composio, etc.)

Chat-mode Claude will auto-skip these when they don't apply.

---

## Compatibility

| Environment | Status |
|---|---|
| **Claude Code** (CLI) | ✅ Full (all 86 skills work) |
| **Cowork** (workspace) | ✅ Full |
| **Claude.ai chat** (web/desktop) | ⚠️ ~74 of 86 work (12 CLI-dependent skills will load but their commands won't run) |
| **GitHub Copilot, Gemini CLI, Antigravity** | ✅ Universal skills supported |

---

## License

Mixed. Each skill carries its original authorship and license — see individual `SKILL.md` files. The packaging and curation work is by DECO AI.

Original sources include: Anthropic, Vercel, Corey Haines (marketingskills), Matt Pocock (claude-code-skills), Julian Onofrei (designer-skills), Composio, Remotion, Jamie BitFlight (claude_skills), and the agent-browser team.

---

## Maintenance

- **Audit report:** `~/Desktop/skills-audit-report.md`
- **Archive of removed skills:** `~/Desktop/skills-archive/`
- **Re-audit anytime:** `npx skillfish list` to see installed skills
