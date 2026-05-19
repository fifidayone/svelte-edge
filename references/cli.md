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

## Community add-ons (`sv@0.1.0+`, experimental)

Community add-ons are discoverable via the official CLI as an experimental opt-in.
Recommend a specific community add-on only when it clearly benefits the project — do not promote unvetted add-ons by reflex.

## `sv` and `sv-utils` separation (`sv@0.2.0+`)

The CLI is split into separate `sv` and `sv-utils` packages with an explicit public API.
When scripting around `sv`, prefer the documented public surface; do not rely on private internals.
