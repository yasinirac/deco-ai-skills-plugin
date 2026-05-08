---
name: awesome-design-md
description: Reference a brand's exact visual language (colors, typography, spacing, components) when building UI. Local collection of DESIGN.md files for 59 brands — Stripe, Apple, Linear, Claude, Vercel, Notion, Spotify, Figma, Supabase, and more. Use when the user asks to build a page/component "like <brand>", match a site's aesthetic, or generate UI in a specific brand's style.
---

# awesome-design-md

A cloned copy of [VoltAgent/awesome-design-md](https://github.com/VoltAgent/awesome-design-md) lives at:

```
C:\Users\DECO-YASIN\design-md\design-md\<brand>\
```

Each brand folder contains three files:

- `DESIGN.md` — full design system (color tokens, typography scale, spacing, component patterns, motion, voice)
- `preview.html` — visual catalog, light mode
- `preview-dark.html` — same catalog, dark mode

## When to use this skill

Triggers:
- "Build a page like Stripe / Apple / Linear / Claude / <brand>"
- "Make this look like <brand>"
- "Use <brand>'s design system / color palette / typography"
- "Generate a landing page that matches <site>"
- User mentions a specific brand name while asking for UI work

## How to use

1. **Resolve the brand.** Run `ls C:/Users/DECO-YASIN/design-md/design-md/` to list available folders. Folder names are lowercase, usually the brand slug (`stripe`, `apple`, `linear.app`, `claude`, `x.ai`, etc.). If the user's brand isn't in the list, say so and offer the closest matches.

2. **Read the DESIGN.md** for that brand before writing any styling code. Do not guess tokens — the file has exact hex values, font stacks, spacing scales, and component specs.

3. **Apply the tokens faithfully.** Use the exact colors, fonts, radii, shadows, and motion values from the file. If the user's stack is Tailwind/shadcn, map the tokens to Tailwind config or CSS variables; if plain CSS, inline the values.

4. **Optionally copy the DESIGN.md into the project** if the user wants it checked in — typical location is the project root. Ask first.

## Available brands (59)

airbnb · airtable · apple · bmw · cal · claude · clay · clickhouse · cohere · coinbase · composio · cursor · elevenlabs · expo · ferrari · figma · framer · hashicorp · ibm · intercom · kraken · lamborghini · linear.app · lovable · minimax · mintlify · miro · mistral.ai · mongodb · notion · nvidia · ollama · opencode.ai · pinterest · posthog · raycast · renault · replicate · resend · revolut · runwayml · sanity · semrush · sentry · spacex · spotify · stripe · supabase · superhuman · tesla · together.ai · uber · vercel · voltagent · warp · webflow · wise · x.ai · zapier

## Keeping it fresh

The clone was shallow. To pull new brands later:

```
cd C:/Users/DECO-YASIN/design-md && git pull
```

## What this skill is NOT

- Not a CLI — there's nothing to invoke; just read the markdown.
- Not a substitute for the `ui-ux-pro-max` skill, which covers *general* UI style systems (glassmorphism, neumorphism, etc.). `awesome-design-md` is about *specific brand* replication. They compose well: pro-max for stylistic direction, awesome-design-md when a real brand is named.
