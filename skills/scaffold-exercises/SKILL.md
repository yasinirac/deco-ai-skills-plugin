---
name: scaffold-exercises
description: NICHE skill for AI Hero / ai-hero-cli course-authoring repos only. Creates exercise directory structures with sections (`XX-section-name/`), exercises (`XX.YY-exercise-name/`), and variant subfolders (`problem/`, `solution/`, `explainer/`) with stub files that pass `pnpm ai-hero-cli internal lint`. Use ONLY inside a Matt Pocock / AI Hero course repo. Trigger when the user says "scaffold exercises", "create exercise stubs", "set up new course section", "add exercises X.Y-X.Z", "create problem/solution/explainer folders", "renumber exercise", "add explainer.1 variant", or "scaffold next cohort module". Skip when the project does NOT have `ai-hero-cli` as a dependency, when the user wants generic scaffolding (use `pnpm create next-app`, etc.), or when authoring non-AI-Hero courses. Requires `pnpm` and `ai-hero-cli` dev dependency. Final validation: `pnpm ai-hero-cli internal lint`.
metadata:
  tags: ai-hero, course-authoring, scaffolding, exercises, niche
  requires-cli: true
  runtime: claude-code-or-terminal
---

# Scaffold Exercises (AI Hero / ai-hero-cli)

Create exercise directory structures that pass `pnpm ai-hero-cli internal lint`, then stage with `git add` and commit.

## When to use

- The repo's `package.json` declares `ai-hero-cli` as a dependency (or the project is a known AI Hero course repo).
- The user asks to "scaffold", "stub out", "set up", or "renumber" exercises matching the AI Hero `XX-section/XX.YY-exercise/<variant>/readme.md` convention.
- The user has a plan / outline naming the sections + exercises and wants the directories created without writing the lesson content yet.

Concrete user requests:
- "Add exercises 05.04 through 05.08 to the memory section, all explainer-only."
- "Scaffold a new section called 07-evals-and-tracing with three exercises - intro, problem+solution, explainer."
- "Move 03.07 to 03.04 and renumber the rest accordingly."
- "Stub out the long-term memory section so the lint passes - I'll fill the readmes later."

## When NOT to use

- The repo doesn't use `ai-hero-cli`. Use whatever scaffolding tool that project ships with.
- The user wants to scaffold a Next.js / Remotion / Vite app - use the framework's `create-*` CLI.
- The user wants to write exercise content (the lesson narrative, the code, the tests). This skill only sets up directories + stub readmes; switch to the relevant content/coding skill afterwards.
- The user wants generic test-suite scaffolding. Use `tdd` or the framework-specific test generator.

## Prerequisites

- `pnpm` installed and in PATH.
- Inside the AI Hero repo root with `pnpm install` already run (so `ai-hero-cli` is available).
- Confirm: `pnpm ai-hero-cli --help` should print the CLI's commands. If it errors with "command not found," this skill is not applicable - bail and suggest the user check they're in the right repo.

## Directory naming

- **Sections**: `XX-section-name/` inside `exercises/` (e.g., `01-retrieval-skill-building`).
- **Exercises**: `XX.YY-exercise-name/` inside a section (e.g., `01.03-retrieval-with-bm25`).
- Section number = `XX`, exercise number = `XX.YY`.
- Names are dash-case, lowercase only, hyphens between words.

## Exercise variants

Each exercise needs at least one of these subfolders:

- `problem/` - student workspace with `// TODO:` markers.
- `solution/` - reference implementation.
- `explainer/` - conceptual material with no TODOs.
- `explainer.1/`, `explainer.2/`, ... - sequential explainer variants.

When stubbing, default to a single `explainer/` unless the plan specifies otherwise.

## Required files

Each variant subfolder (`problem/`, `solution/`, `explainer/`) needs a `readme.md` that:

- Is **not empty** (must have real content - even a single title line works).
- Has no broken links.

Minimal stub:

```md
# Exercise Title

Description here
```

If the subfolder ships code (problem/solution mostly), it also needs a `main.ts` with more than 1 line. For pure stubs, a readme-only variant folder is fine.

## Workflow

1. **Parse the plan** - extract section names, exercise names, and which variants each exercise needs.
2. **Create directories** - one `mkdir -p` per leaf variant path.
3. **Create stub readmes** - one `readme.md` per variant folder with a `# Title` and a one-line description.
4. **Run lint** - `pnpm ai-hero-cli internal lint` to validate.
5. **Fix any errors** - common ones below; iterate until lint passes.
6. **Commit** - use `git add exercises/<paths> && git commit -m "..."`.

## Lint rules summary

The linter (`pnpm ai-hero-cli internal lint`) checks:

- Each exercise folder has at least one variant subfolder.
- At least one of `problem/`, `explainer/`, or `explainer.1/` exists per exercise.
- `readme.md` exists and is non-empty in the primary subfolder.
- No `.gitkeep` files anywhere.
- No `speaker-notes.md` files.
- No broken links inside readmes.
- No `pnpm run exercise` commands inside readmes.
- `main.ts` is required per variant subfolder unless the variant is intentionally readme-only.

## Examples (DECO / AI Hero scenarios)

These are typical when the user is preparing course materials. DECO won't usually run this skill for client work - it's for internal training and the AI Hero collab repos.

**Example 1 - Stubbing a memory section.**

User plan:
```
Section 05: Memory Skill Building
- 05.01 Introduction to Memory
- 05.02 Short-term Memory (explainer + problem + solution)
- 05.03 Long-term Memory
```

Commands:
```bash
mkdir -p exercises/05-memory-skill-building/05.01-introduction-to-memory/explainer
mkdir -p exercises/05-memory-skill-building/05.02-short-term-memory/{explainer,problem,solution}
mkdir -p exercises/05-memory-skill-building/05.03-long-term-memory/explainer
```

Then create readmes:
```
exercises/05-memory-skill-building/05.01-introduction-to-memory/explainer/readme.md  -> "# Introduction to Memory"
exercises/05-memory-skill-building/05.02-short-term-memory/explainer/readme.md       -> "# Short-term Memory"
exercises/05-memory-skill-building/05.02-short-term-memory/problem/readme.md         -> "# Short-term Memory"
exercises/05-memory-skill-building/05.02-short-term-memory/solution/readme.md        -> "# Short-term Memory"
exercises/05-memory-skill-building/05.03-long-term-memory/explainer/readme.md        -> "# Long-term Memory"
```

Then `pnpm ai-hero-cli internal lint`.

**Example 2 - Renumbering an exercise.**

User: "Move 01.03-embeddings to 01.04-embeddings to make room for a new 01.03 on tokenization."

```bash
git mv exercises/01-retrieval/01.03-embeddings exercises/01-retrieval/01.04-embeddings
mkdir -p exercises/01-retrieval/01.03-tokenization/explainer
# create stub readme for 01.03
pnpm ai-hero-cli internal lint
```

Always use `git mv` rather than `mv` so history follows the rename.

**Example 3 - Adding an explainer.1 variant.**

User: "The intro to retrieval needs two explainer passes - 01.01-introduction-to-retrieval/explainer.1 and explainer.2."

```bash
mkdir -p exercises/01-retrieval/01.01-introduction-to-retrieval/{explainer.1,explainer.2}
# create stub readme in each
pnpm ai-hero-cli internal lint
```

## Common pitfalls

- **Wrong case / spaces.** Section and exercise names must be dash-case lowercase. `01-Retrieval-Skill-Building/` will fail the lint.
- **Missing primary variant.** An exercise with only a `solution/` folder fails - you need at least `problem/`, `explainer/`, or `explainer.1/` as the primary.
- **Empty readme.** A blank file or whitespace-only file fails the non-empty check. Always include a title line.
- **`.gitkeep` files.** People reflexively add these for empty directories. Remove them - the lint forbids them.
- **`speaker-notes.md`.** Forbidden by lint. If the user has speaker notes, suggest a separate path outside `exercises/`.
- **Broken markdown links.** When the stub readme links to other exercises or external docs, make sure the targets exist or remove the link.
- **`pnpm run exercise` references.** Old course material sometimes had this command in instructions. Don't include it in new readmes.
- **Using `mv` instead of `git mv`** when renumbering. Lose git history. Always `git mv`.
- **Forgetting to run the lint.** Don't claim "done" until `pnpm ai-hero-cli internal lint` exits 0.

## Related skills

- **scaffold-exercises (this)** - directory structure only. Does not write lesson content.
- **tdd** - for writing actual exercise tests once the structure is in place.
- **to-issues** / **to-prd** - if the user wants to break a course outline into trackable issues instead of folders.
- **graphify** - to map the exercise dependency graph after several sections exist.
- **deco-technical-education-posts** - if the user is repackaging an AI Hero exercise as a DECO LinkedIn educational post.

> Note: this skill is **niche**. If the user asks for "exercise scaffolding" outside of an AI Hero / `ai-hero-cli` repo, decline politely and suggest the framework's own generator. Do NOT generalize this convention to other course repos - the lint is project-specific.
