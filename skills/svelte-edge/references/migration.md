# Migration: Legacy Svelte to Modern Svelte 5

Use this file only for migration work.
Do not use it as the default source for brand-new code.

## Contents

- [CLI-first migration](#cli-first-migration)
- [Manual cleanup after CLI](#manual-cleanup-after-cli)
- [Migration doctrine](#migration-doctrine)
- [SvelteKit migration](#sveltekit-migration)
- [Remote-function compatibility timeline](#remote-function-compatibility-timeline)
- [SvelteKit 3 preview migration](#sveltekit-3-preview-migration)
- [Post-migration reading](#post-migration-reading)

## CLI-first migration

Use the official migration CLI as the primary tool. Do not manually rewrite what the CLI handles.

```bash
npx sv migrate svelte-5
```

For SvelteKit projects, also run:

```bash
npx sv migrate app-state
```

The `svelte-5` migration bumps core dependencies, converts `let`→`$state`, `$:`→`$derived`/`$effect`, `on:click`→`onclick`, `<slot>`→snippets+`{@render}`, and `new Component(...)`→`mount(...)`. The `app-state` migration converts `$app/stores` to `$app/state`.

Per-file migration is also available via VS Code: **"Migrate Component to Svelte 5 Syntax"** in the Command Palette.

After running, verify with `npx sv check` and the project's test suite.

## Manual cleanup after CLI

The CLI inserts `@migration` comments at points requiring human judgment. Address each annotation, then remove it.

### `run()` from `svelte/legacy`

When the CLI cannot determine whether a `$:` statement is a derivation or a side effect, it wraps it in `run()` imported from `svelte/legacy`. Review each instance:

- If the code computes a value from other state → replace with `$derived` or `$derived.by`.
- If the code performs a browser-only side effect → replace with `$effect`.
- `run()` mimics `$:` behavior (runs once on server, runs as `$effect.pre` on client). It is a stopgap, not a target.

### Event modifier wrappers from `svelte/legacy`

The CLI replaces `on:click|preventDefault={handler}` with a `preventDefault(handler)` wrapper from `svelte/legacy`. Inline the behavior into the handler instead:

```svelte
<script lang="ts">
	function handleSubmit(event: SubmitEvent) {
		event.preventDefault();
		// ...
	}
</script>

<form onsubmit={handleSubmit}></form>
```

### `createEventDispatcher`

The CLI does not convert `createEventDispatcher` because it cannot determine the impact on external callers. Replace with typed callback props manually. See `references/runes.md` for callback-prop patterns.

### `beforeUpdate` / `afterUpdate`

The CLI does not convert these because the intended behavior is ambiguous. As a rule of thumb:

- `beforeUpdate` → `$effect.pre(() => { ... })`
- `afterUpdate` → `$effect(() => { ... })`, or use `tick` from `svelte` to wait for DOM updates within an effect.

## Migration doctrine

When migrating, rewrite coherently.
Do not half-migrate a component into a mixed-generation hybrid.

- Migrate a whole component to runes mode.
- Keep a legacy file legacy if only a tiny fix is required.
- Modernize in coherent chunks, not one random rune at a time.

## SvelteKit migration

Do not treat remote functions as the automatic end state of every SvelteKit migration.
Stable `load`, form actions, and `+server` files are still the normal baseline.

For Svelte 5 projects on SvelteKit 2.12+, replace deprecated `$app/stores` imports with `$app/state` and update store-subscription syntax to direct reactive property reads.

### Config location and SvelteKit 3 readiness

SvelteKit **2.62+** can place all Svelte/SvelteKit configuration in `sveltekit({...})` inside `vite.config`, and `sv@0.16+` scaffolds new projects that way. If migrating:

- move the entire config coherently
- flatten former `kit` properties into the `sveltekit({...})` argument
- remove the old config after verifying tooling support
- never leave partial settings in both places, because plugin configuration causes `svelte.config.js` to be ignored

The old file remains supported in SvelteKit 2. Move it early only for a deliberate cleanup or SvelteKit 3 preparation. This subsection covers **SvelteKit 2** readiness steps only — once the project actually upgrades to the `3.0.0-next.*` line, stop following this subsection and read `references/sveltekit-3-preview.md` instead, since SvelteKit 3 makes this config location mandatory rather than optional.

### Environment variables and SvelteKit 3 readiness

SvelteKit **2.63+** offers experimental explicit environment variables. `$env/*` remains the SvelteKit 2 target; do not bulk-migrate it unless the project explicitly opts into `experimental.explicitEnvironmentVariables` or is preparing for SvelteKit 3.

An opted-in migration includes:

- declare variables in `src/env.ts` with `defineEnvVars` from `@sveltejs/kit/env` (moved here in **SvelteKit 2.70+**; older code importing it from `@sveltejs/kit/hooks` still works via a deprecated re-export)
- replace `$env/static|dynamic/private|public` imports with `$app/env/private|public`
- replace `$app/environment` with `$app/env`
- preserve public/private boundaries and validate transformed values

This is the **SvelteKit 2** opt-in flag timeline. It is a separate migration from the SvelteKit 3 preview's own environment-variable breaking changes (e.g. its `next.10` move of `defineEnvVars` and its `schema`/`static`/`building` options) — do not cite a SvelteKit 2 version number as if it applied to SvelteKit 3 preview behavior, or vice versa. See `references/sveltekit-3-preview.md` for the SvelteKit 3 preview's environment-variable model.

## Remote-function compatibility timeline

Remote functions are experimental and changed rapidly. Do not stop at an intermediate migration state.
Each entry below is labeled to show actual impact on existing code:

- **2.56** — ✓ ADDITIVE: briefly added query `.run()`, stable cache-key sorting, richer hydratable transport, and explicit server acceptance for client-requested refreshes. Existing code continues to work unchanged.
- **2.57** — ✓ ADDITIVE: made enhanced form `submit()` return a boolean validity result. Existing callers that ignore the return value are unaffected.
- **2.58** — ⚠ BREAKING: made `requested(query, limit)` require the `limit` argument and yield `{ arg, query }` instead of the previous shape. Calls without `limit`, or destructuring the old shape, will throw or silently break.
- **2.59** — ✓ ADDITIVE: added `query.live` and batch-query support in `requested(...)`. Opt-in only; existing code is unaffected.
- **2.61** — ⚠ BREAKING: removed `.run()` entirely. Any code still calling someQuery(...).run() will fail immediately — must replace with `await someQuery(...)`.
- **2.61** — ⚠ BREAKING: changed `enhance` callbacks to receive a form-instance copy instead of the old `{ form, data, submit }` object. Old destructuring patterns will fail silently or throw depending on usage — this is the single highest-risk step in the timeline for production form flows.

### Current migration target (2.61+)

When upgrading an older remote-functions project to a current release, prioritize the ⚠ BREAKING items above before anything else:

1. **First, fix breaking changes** (in this order, since later ones depend on earlier code being stable):
   - Add the required `limit` parameter to all `requested(...)` calls and update destructuring to `{ arg, query }`.
   - Replace every transitional `someQuery(...).run()` with `await someQuery(...)`.
   - Update old `enhance(({ form, data, submit }) => ...)` callbacks to receive the form instance and call `await form.submit()`.
2. **Then, adopt additive improvements** (lower risk, can be done incrementally):
   - Remove any manual sorting of cache keys — caching now sorts object keys internally.
   - Audit client-triggered refresh logic — it now requires explicit server-side permission.
   - Drop cross-query failure handling that assumed cascading failures; each refresh failure is isolated per-query.
   - For live queries, handle shared connections, `connected`, `reconnect()`, and async iteration instead of treating them as ordinary refreshable queries.

Do not teach or preserve `.run()` as a compatibility fallback. Pinning to SvelteKit 2.56–2.60 is the only reason it should appear in historical code.

Remote functions on the SvelteKit 3 preview line have their own additional breaking changes (e.g. mandatory `form.fields.foo.as(...)`, stricter `event.url`/`event.params`/`event.route` access inside queries). Once a project is on `3.0.0-next.*`, cross-check remote-function code against `references/sveltekit-3-preview.md` in addition to this timeline — the two timelines are not interchangeable.

## SvelteKit 3 preview migration

SvelteKit 3 is still a `3.0.0-next.*` prerelease, not a stable release. Treat it as a distinct generation from SvelteKit 2 — the subsections above prepare a SvelteKit 2 project for an eventual jump; they are not a substitute for the actual migration.

The SvelteKit 3 jump happens on the `sv@next` line. For a **new** project, `npx -y sv@next create` scaffolds SvelteKit 3 directly (`@sveltejs/kit@3.0.0-next.*` and `#lib` imports). For an **existing** SvelteKit 2 project, run `npx -y sv@next migrate sveltekit-3 --tasks all --confirm` (plus `sv migrate app-state`). The two prerequisite tasks (`package-json` — moves SvelteKit, its peers, and every `@sveltejs/adapter-*` to the `next` line — and `tsconfig` — retargets `extends` to `$app/tsconfig`) run automatically; `all` selects every remaining task, including `collect-migration-instructions`, which writes `MIGRATION_TASKS.md` for the non-automated steps (e.g. `$service-worker` replacement, leftover `$app/stores`, `handleValidationError`, cookie v2, adapter-specific changes) plus `@migration-task` comments — the same CLI-first + cleanup-after shape as the `svelte-5` migration, and the CLI recommends one task at a time with a commit after each. The pre-`sv@next` experimental-add-on route is superseded — don't use it for new SvelteKit 3 work. Official guide: https://next.svelte.dev/docs/kit/migrating-to-sveltekit-3. This file owns the CLI workflow; the SvelteKit 3 knowledge surface lives in `references/sveltekit-3-preview.md`.

npm does not re-resolve a dist-tag on a bare `npm install` — a tag is looked up only at install time, and the lockfile records the concrete version (npm/cli#3755). If `npm install` reports 'up to date' while `package-lock.json` still resolves the old line, reinstall the moved packages explicitly: `npm install @sveltejs/kit@next @sveltejs/adapter-<name>@next`.

The migration rewrites import statements, not string literals — test files that reference a moved module by name (e.g. `vi.mock('$app/environment')`, `jest.mock('$app/stores')`) are not touched. After migrating, grep the whole repo (including `*.test.*`/`*.spec.*`, setup files, and fixtures) for the old module names and update them by hand.

Once a project resolves `@sveltejs/kit@3.0.0-next.*`, or the user explicitly asks to migrate from SvelteKit 2 to SvelteKit 3, stop using this file for SvelteKit-specific guidance and read `references/sveltekit-3-preview.md` instead. It owns:

- exact toolchain/version floors for the preview line
- the complete SvelteKit 3 knowledge surface (config, routing, env, navigation, service workers, remote functions, behavior changes)
- code-generation recipes for writing new SvelteKit 3 code
- adapter verification and test checklist

Do not hand-roll a SvelteKit 3 migration from this file's SvelteKit 2 readiness notes alone — they cover *preparation*, not the actual breaking-change surface.

## Post-migration reading

After completing a migration, consult the following references for patterns and gotchas in the migrated code:

- **State architecture and runes gotchas** — `references/runes.md` (destructuring pitfalls, `$effect` async tracking, pass-by-value, `$effect.root` cleanup)
- **SvelteKit patterns** — `references/sveltekit.md` (load functions, form actions, SSR safety, `$app/state`)
- **Anti-mixing and pitfalls** — `references/best-practices.md` (syntax coherence rules, hydration caveats, raw HTML safety)
