# Svelte Edge

Future-first guidance for writing **modern Svelte 5 and SvelteKit** code without drifting into mixed-generation answers.

This skill is built for AI agents and maintainers who want:
- modern Svelte 5 answers
- stable SvelteKit architecture by default
- correct topic routing across references
- coherent migration decisions
- explicit handling of experimental features

## Installation

You can install this skill into your AI agent environment (such as Claude Code, Cursor, Windsurf, or Gemini) using the Vercel **`skills`** CLI:

```bash
npx skills add fifidayone/svelte-edge --all
```

Alternatively, you can clone this repository directly into your local workspace's agent configuration directory (e.g., `<project-root>/.agents/skills/svelte-edge`):

```bash
git clone https://github.com/fifidayone/svelte-edge.git .agents/skills/svelte-edge
```

## Design goals

- **Future-first** for new code
- **Stable-first** for SvelteKit production architecture
- **TypeScript-first where modern workflows benefit**
- **No mixed-generation components**
- **Canonical ownership per topic**
- **Explicit opt-in for experimental features**
- **Repeatable audit output**
- **Low redundancy across reference files**

## What this skill is for

Use it when the user asks about:
- Svelte 5 component authoring
- runes and reactivity
- snippets and component composition
- attachments and DOM behavior
- transitions and animations
- async component patterns
- SvelteKit routing, forms, state, navigation, and server code
- migrations from Svelte 4 / legacy Svelte to modern Svelte 5
- testing strategy for modern Svelte projects
- ecosystem libraries and tooling for Svelte 5
- official `sv` CLI workflows

## What this skill is not for

- broad backward compatibility by default
- teaching legacy Svelte 3/4 patterns unless explicitly requested
- treating experimental features as mandatory defaults
- using ecosystem packages to override official framework guidance

## Operating model

The skill uses **canonical topic ownership** with two tiers:

**Mandatory (always consulted when the topic comes up):**
- `references/runes.md` — runes, reactivity, lifecycle, scheduling, reactive classes, context, bindings
- `references/snippets.md` — snippets and composition
- `references/attachments.md` — modern DOM behavior and action replacement
- `references/animations-transitions.md` — motion, transitions, animations, easing
- `references/async-svelte.md` — direct `await`, boundaries, abort-aware async patterns, `fork(...)`, hydratable data
- `references/sveltekit.md` — SvelteKit architecture, routing, forms, auth, server safety, navigation, remote functions
- `references/testing.md` — testing strategy and tool choice
- `references/best-practices.md` — practical guardrails and pitfalls
- `references/cli.md` — official `sv` commands

**On-demand only (read only when the user explicitly asks about the topic):**
- `references/migration.md` — migration-only work
- `references/libraries.md` — ecosystem discovery

If references overlap, prefer the canonical semantic owner defined in `SKILL.md`.

## Repository structure

| File | Purpose | Load mode |
|---|---|---|
| `SKILL.md` | Policy, working modes, routing, version gates, audit contract | always |
| `references/runes.md` | Modern Svelte semantics: runes, reactivity, lifecycle, scheduling, reactive classes, context, bindings | mandatory |
| `references/snippets.md` | Snippets, render props, typing, modern composition | mandatory |
| `references/attachments.md` | `{@attach}`, `fromAction(...)`, modern DOM behavior | mandatory |
| `references/animations-transitions.md` | `svelte/motion`, `transition:`, `in:`/`out:`, `animate:`, easing, custom motion | mandatory |
| `references/async-svelte.md` | Direct `await`, `<svelte:boundary>`, `getAbortSignal()`, `fork(...)`, `hydratable(...)` | mandatory |
| `references/sveltekit.md` | Stable SvelteKit defaults, state safety, routing, auth, forms, env/server-only safety, remote functions | mandatory |
| `references/testing.md` | Vitest, Playwright, Storybook | mandatory |
| `references/best-practices.md` | Guardrails, anti-mixing, hydration caveats, raw HTML safety | mandatory |
| `references/cli.md` | Official `sv` commands and add-on guidance | mandatory |
| `references/migration.md` | Migration guidance from legacy code | on-demand |
| `references/libraries.md` | Community ecosystem discovery map | on-demand |

## Quick start

```bash
npx sv create my-app
npx sv add eslint prettier tailwindcss vitest playwright
npx sv migrate svelte-5
npx sv check
```

## Scope decisions

Testing has its own canonical file so tool choice and testing doctrine stay first-class instead of being buried in general best practices.

`libraries.md` is on-demand because most Svelte questions are about framework semantics, not package selection. Pull it only when the user explicitly asks about ecosystem packages.

`migration.md` is on-demand because most new code does not need migration guidance. Pull it only for migration work.

Examples are inlined into their canonical owners rather than living in a separate patterns file — this avoids drift between doctrine and examples.

## Version and freshness

Current baseline: **May 2026**.

Detailed version gates live in `SKILL.md`. Keep `README.md` lightweight and use it to explain scope, structure, and maintenance expectations.

## Maintenance model

Update this skill when official docs materially change:
- recommended modern Svelte syntax
- stable SvelteKit architecture defaults
- experimental feature status or required flags
- public `sv` workflows
- important version gates

Framework semantics age slower than ecosystem package lists. `libraries.md` should be reviewed more often than core doctrine files.

## Audit note

Audit and review workflows should follow the output contract defined in `SKILL.md` so findings stay repeatable and actionable.

## Goal

Accurate, modern, coherent, production-safe guidance that an AI agent can apply quickly and consistently.
