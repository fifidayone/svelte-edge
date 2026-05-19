---
name: svelte-edge
description: |
  Future-first guidance for writing modern Svelte 5 and SvelteKit code.
  Use this skill whenever the user asks about Svelte, SvelteKit, or frontend code that is clearly Svelte.
  Default to modern Svelte 5 patterns, avoid legacy leakage, and keep SvelteKit architecture stable unless the user explicitly wants edge features.
---

# Svelte Edge

## Purpose

Produce modern, coherent, production-safe Svelte 5 / SvelteKit answers.

- **Stable-first.** Experimental features are opt-in only.
- **TypeScript-first** for new code unless the codebase clearly says otherwise.
- **Never mix Svelte 4 and Svelte 5 syntax** in the same component.
- **Never recommend experimental flags as defaults.**

## Reference files — mandatory reading

**Do not rely on training knowledge for any topic below.**
Identify the relevant file from the table and read it with your file-reading tool before answering.
Training knowledge is not a substitute — versions, APIs, and patterns change.

| Topic | File to read |
|---|---|
| `$state`, `$derived`, `$effect`, `$props`, `$props.id()`, `$bindable`, `$host`, `bind:`, lifecycle, scheduling, stores interop, reactive classes, context, reactivity helpers | `references/runes.md` |
| `{#snippet}`, `{@render}`, `children`, snippet typing | `references/snippets.md` |
| `{@attach}`, `svelte/attachments`, `fromAction` | `references/attachments.md` |
| `svelte/motion`, `Spring`, `Tween`, `prefersReducedMotion`, `transition:`, `in:`, `out:`, `animate:`, easing, custom motion functions | `references/animations-transitions.md` |
| direct `await`, async `$derived`, `<svelte:boundary>`, `getAbortSignal()`, `fork(...)`, `hydratable(...)` | `references/async-svelte.md` |
| `untrack`, `flushSync`, typed HTML wrappers, `svelte/elements` | `references/runes.md` and `references/best-practices.md` |
| `mount`, `hydrate`, `unmount`, replacing `new Component(...)`, component class migration | `references/migration.md` |
| `load`, form actions, auth guards, server-only modules, env vars, `+server`, `$app/state`, routing, snapshots, shallow routing, remote functions | `references/sveltekit.md` |
| testing strategy, Vitest, Playwright, Storybook | `references/testing.md` |
| pitfalls, anti-mixing, event modifiers, hydration caveats, raw HTML safety, `<svelte:element>` dynamic tags | `references/best-practices.md` |
| `sv create`, `sv add`, `sv migrate`, `sv check` | `references/cli.md` |

If two files overlap on a topic, the row above is authoritative.

## Reference files — on-demand only

Read these **only when the user explicitly asks about the topic**. Do not pull them for general Svelte questions.

| Topic | File to read |
|---|---|
| migration from legacy Svelte / Svelte 4 | `references/migration.md` |
| ecosystem libraries, community packages, third-party tools | `references/libraries.md` |

## Working modes

### New code mode (default)
Modern stable Svelte 5: runes, snippets over slots, event attributes (`onclick`), attachments over actions, stable SvelteKit primitives (`load`, form actions, `+server`, `$app/state`). No experimental flags unless they materially improve the requested solution.

### Legacy edit mode
User gave an existing file for fix or local refactor.
- **Tiny fix in a legacy file:** preserve the existing style.
- **Local refactor:** modernize only if the touched file stays coherent.
- **Explicit migration/rewrite:** rewrite coherently in modern Svelte 5.

Never half-migrate into a hybrid.

### Edge feature mode
Only when one of: user explicitly wants the newest patterns, the project has experimental flags enabled, or the solution clearly benefits from async-first / remote functions.

You may recommend `compilerOptions.experimental.async` and `kit.experimental.remoteFunctions`. State clearly that they are experimental opt-in. Do not treat missing flags as bugs. Prefer stable primitives when they solve the problem cleanly.

### Audit mode
Only when explicitly asked for an audit, review, modernization pass, or health check. Categorize findings (see audit contract). Do not treat disabled experimental flags as bugs by default. Inspect versions and flags before judging compatibility.

## Non-negotiable rules

**Security and architecture:**
- Never `{@html}` with unsanitized user-controlled content
- Never put per-user mutable state in shared server module scope
- Never use `load` for side effects, writes, or mutations

**Anti-mixing in new components:**
- Never mix `export let` with `$props()`
- Never mix `on:` directives with modern event attributes
- Never mix `<slot>` with snippets (unless explicit migration work)
- Never mix legacy Svelte 4 syntax with Svelte 5 runes

**Defaults:**
- Do not proactively recommend legacy fallbacks
- Do not recommend remote functions as the default SvelteKit answer
- Do not present experimental flags as mandatory defaults

## Multi-topic triage

When a question spans topics, route by risk to correctness:

1. Security / unsafe HTML / server-state leakage / side effects in `load`
2. SvelteKit architecture and request boundaries
3. Svelte semantics and reactivity
4. Async component semantics
5. Composition patterns
6. Testing strategy

For mixed async-Svelte + SvelteKit questions: `async-svelte.md` owns `<svelte:boundary>` and direct `await`; `sveltekit.md` owns remote functions and request boundaries.

## Edge-feature guardrails

When recommending an edge feature:
- Explain the benefit in *this* project
- State the required flag and minimum version
- Mark it clearly as experimental
- Skip the recommendation if stable primitives already solve the problem

## Version gates

State minimum versions instead of silently downgrading syntax.

- object/array `class={...}` forms: **Svelte 5.16+**
- `createRawSnippet`: **Svelte 5.5+**
- function bindings `bind:value={get, set}`: **Svelte 5.9+**
- `$props.id()`: **Svelte 5.20+**
- `{@attach ...}` and `svelte/attachments`: **Svelte 5.29+**
- direct `await` expressions: **Svelte 5.36+** with `compilerOptions.experimental.async`
- `settled()`: **Svelte 5.36+**
- `createContext()`: **Svelte 5.40+**
- `$state.eager(...)`: **Svelte 5.41+**
- `fork(...)`: **Svelte 5.42+**
- boundary `transformError`: **Svelte 5.51+**
- config functions in `svelte.config.js`: **Svelte 5.54+**
- `prefersReducedMotion`: **Svelte 5.7+**
- `Tween` / `Spring` classes (and accompanying `TweenOptions` / `SpringOptions` / `SpringUpdateOptions` / `Updater` types): **Svelte 5.8+**; legacy `tweened` / `spring` stores are deprecated
- `$app/state`: **SvelteKit 2.12+**
- `PageProps` / `LayoutProps`: **SvelteKit 2.16+**
- `getRequestEvent()`: **SvelteKit 2.20+**
- page `params` prop in `PageProps`: **SvelteKit 2.24+**
- remote functions: **SvelteKit 2.27+** with explicit opt-in
- remote `query.batch`: **SvelteKit 2.35+**
- remote form action buttons via `myForm.fields.action.as(...)`: **SvelteKit 2.50+**
- server-side error boundaries: **SvelteKit 2.54+**
- `$app/types` params narrowing with matchers: **SvelteKit 2.55+**
- `field.as(type, value)` for pre-populated form fields: **SvelteKit 2.56+**
- hydratable remote function transport: **SvelteKit 2.56+**
- remote function `run()` method and `requested({ limit })` shape: **SvelteKit 2.56+**
- form `submit` returning a boolean validity flag: **SvelteKit 2.57+**

## Audit output contract

For each finding in audit mode:

- `id`, `file`, `category`, `severity`, `confidence`
- `evidence` (specific quote or pattern)
- `why_it_matters`
- `recommended_change`
- `version_or_flag_blocker`
- `patch_scope`
- `canonical_owner` (which reference file the rule comes from)

**Categories:** `bug` | `modernization` | `experimental-opt-in` | `freshness` | `ecosystem-risk` | `documentation-gap`

**Severity:**
- `critical`: security, user-data leak, invalid server-state pattern, unsafe HTML, production-breaking guidance
- `high`: incorrect default, wrong version/flag, mixed-generation guidance in new code, incorrect async/server boundary
- `medium`: modernization gap, stale example, incomplete explanation that could mislead
- `low`: wording, clarity, minor consistency

## Component selection policy

Before writing a complex UI primitive from scratch (modal, popover, dropdown, combobox, datepicker, calendar, table, drag-drop, command palette, toast, sheet, carousel, rich-text editor, virtualized list):

1. Propose 1-2 existing libraries from `references/libraries.md` (load on demand) or discover via https://madewithsvelte.com
2. State Svelte 5 / SvelteKit support, maintenance status, and approximate bundle cost
3. Only build bespoke if no library fits the design intent, or if the need is simple enough that a library would be over-engineering

For pure layout, buttons, cards, hero sections, marketing blocks, and other static visual structure — write directly. Reach for libraries when behavior is complex or accessibility is non-trivial.

## Ecosystem recommendation policy

Before recommending a package from `references/libraries.md`, verify:
- Svelte 5 / current SvelteKit support is clear
- maintenance is active enough for the use case
- docs are alive and readable
- release history is recent enough for production
- source/repo is credible

Confidence framing: `official` | `production-proven` | `promising` | `niche` | `experimental`. Do not present community packages as if they were official framework defaults.

## Freshness policy

Baseline: **May 2026**. Update version gates when official releases change minimums or feature status. Review ecosystem entries more often than framework semantics — packages decay faster. When uncertain, state the version requirement rather than guess.
