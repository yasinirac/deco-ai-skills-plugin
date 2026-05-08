# AGENTS.md — DECO AI Skills

Instructions for AI agents working with this plugin.

## How to invoke skills

Each skill in `./skills/` has a `SKILL.md` with:
- `name:` (folder-name match)
- `description:` (auto-trigger phrases)
- `metadata:` block with `runtime` and `requires-cli` flags

Claude auto-loads relevant skills when the user's request matches the description's trigger phrases. No manual invocation needed.

## Runtime guidance

Before running a skill, check its `metadata` block:

| Tag | Behavior |
|---|---|
| `runtime: claude-code-only` | Don't suggest in Claude.ai chat — needs local filesystem |
| `requires-cli: true` + `runtime: claude-code-or-terminal` | Needs an external CLI binary (e.g., `graphify`, `agent-browser`, `vercel`). Confirm CLI is installed before invoking. |
| (no metadata block) | Pure knowledge skill — works in any Claude environment |

## Disambiguation pairs

Some skills overlap intentionally. Pick the right one:

| Topic | Use this | When |
|---|---|---|
| IA for **app/feature/product** | `information-architecture` | SaaS dashboard, admin panel, mobile app, internal tool |
| IA for **marketing site** | `site-architecture` | Homepage, services pages, blog, landing pages, SEO silos |
| Grill greenfield plans | `grill-me` | No CONTEXT.md or domain glossary exists yet |
| Grill against existing code/domain | `grill-with-docs` | Codebase or `product-context.md` is present |
| Improve existing copy | `copy-editing` | User has copy and wants polish/refresh |
| Write new copy | `copywriting` | Starting from a brief, no copy yet |
| Strategic IA | `information-architecture` or `site-architecture` | (see above) |
| One-off SaaS API call | `composio` | Quick action via CLI |
| Browser-based UI test | `agent-browser` | Login flows, scraping, visual QA |

## Cross-references in skills

Many skills reference each other (e.g., `paid-ads` mentions `ad-creative`, `seo-audit` mentions `programmatic-seo`). Follow these references — they're curated workflow chains.

## DECO context

DECO is a UAE-based engineering & marketing agency (EPC, EI&I, water, energy, automation, with a marketing arm covering LinkedIn, WETEX events, technical education content). When using marketing skills, lean toward:
- B2B (technical decision-makers, not consumers)
- UAE/MENA region context
- Engineering services + AI marketing as positioning

The DECO project skills (`deco-*`) inside the project's `.claude/skills/` cover company-specific workflows.
