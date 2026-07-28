# Official Svelte CLI Commands

Use the `sv` CLI as the public, current command surface.
Do not present `svelte-migrate` as the primary user-facing command.

## Core commands

```bash
npx sv create my-app
npx sv add eslint prettier tailwindcss vitest playwright
npx sv migrate svelte-5
npx sv check
```

## `sv create`

Create a new Svelte or SvelteKit project.

With **sv 0.16+**, new SvelteKit projects target Kit 2.62+ and keep Svelte/Kit configuration in the `sveltekit({...})` call in `vite.config` instead of creating `svelte.config.js`. Preserve the scaffolded layout. Do not move config back to the older file out of habit.

Automatic config discovery in `vite.config.js/ts` requires `svelte-language-server` **0.18.2+** and `svelte-check` **4.6.0+**. Upgrade stale tooling before blaming the new config layout.

The demo template also uses Svelte 5.56 declaration tags. If a generated project shows `{const ...}`, upgrade stale editor/checker tooling instead of rewriting it to legacy `{@const}`.

## `sv add`

Use official add-ons by their current names.
Examples that matter often in this skill:

```bash
npx sv add tailwindcss
npx sv add vitest
npx sv add playwright
```

## `sv migrate`

Public migration entry point.

```bash
npx sv migrate
npx sv migrate svelte-5
```

`sv migrate` delegates to migration tooling under the hood, but the public guidance should use `sv`.

## `sv check`

Use this to typecheck and surface Svelte diagnostics.
It is a good final validation step after significant edits.

For direct `svelte-check` use:

- **svelte-check 4.7+** supports `--config <path>` for a non-standard `svelte.config` or `vite.config` location.
- **svelte-check 4.7+** offers experimental `--tsgo`; install `@typescript/native-preview` and expect the same limitations as incremental mode.
- Do not recommend `--tsgo` as the default until the project accepts experimental TypeScript-Go behavior.

## Experimental feature add-on (`sv@0.16+`)

Use only when the user deliberately opts into edge features:

```bash
npx sv add experimental
```

It can enable `async`, `remoteFunctions`, `explicitEnvironmentVariables`, `handleRenderingErrors`, and `forkPreloads`, and can move Kit/adapters to the `@next` line. Name every selected flag and preserve the stable-first policy; never run this add-on merely because a project lacks experimental configuration.

## Declaration-tag toolchain

For Svelte 5.56 declaration tags, require at least:

- `svelte-check` 4.5.0
- `svelte-language-server` 0.18.1
- `svelte2tsx` 0.7.56

## TypeScript 6 toolchain

For TypeScript 6 projects, require at least `svelte-language-server` **0.18.0**, `svelte2tsx` **0.7.55**, `svelte-check` **4.4.8**, and — when used — `svelte-preprocess` **6.0.4**. Upgrade this set together when diagnostics disagree across the editor and CI.

## Community add-ons (`sv@0.14+`, experimental)

Community add-ons are discoverable via the official CLI as an experimental opt-in.
Recommend a specific community add-on only when it clearly benefits the project — do not promote unvetted add-ons by reflex.
Svelte maintainers do not review community add-ons for malicious code; do not use them in production without an independent source and maintenance review.

## `sv` and `sv-utils` separation (`sv@0.15+`, `sv-utils@0.2+`)

The CLI is split into separate `sv` and `sv-utils` packages with an explicit public API.
When scripting around `sv`, prefer the documented public surface; do not rely on private internals.

For add-on authors on `sv-utils@0.3+`, use `svelteConfig` to find/read/edit either Vite-plugin or legacy-file configuration, and `defineEnv` for version-aware environment imports. Do not hand-roll config-file detection.
