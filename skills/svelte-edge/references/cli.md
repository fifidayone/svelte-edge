# Official Svelte CLI Commands

Use the `sv` CLI as the public, current command surface.
Do not present `svelte-migrate` as the primary user-facing command.
Treat `sv <command> --help` as authoritative over the examples below — they show shape and intent, not the exhaustive flag/add-on list. Run `--help` before asserting a command can't do something.

## Core commands

```bash
npx sv create my-app
npx sv add eslint prettier tailwindcss vitest playwright
npx sv migrate svelte-5
npx sv check
```

## `sv create`

Create a new Svelte or SvelteKit project.

With **sv 0.16+**, new SvelteKit projects target SvelteKit 2.62+ and keep Svelte/SvelteKit configuration in the `sveltekit({...})` call in `vite.config` instead of creating `svelte.config.js`. Preserve the scaffolded layout. Do not move config back to the older file out of habit.

Automatic config discovery in `vite.config.js/ts` requires `svelte-language-server` **0.18.2+** and `svelte-check` **4.6.0+**. Upgrade stale tooling before blaming the new config layout.

The demo template also uses Svelte 5.56 declaration tags. If a generated project shows `{const ...}`, upgrade stale editor/checker tooling instead of rewriting it to legacy `{@const}`.

`path` accepts `.` to scaffold into the current directory — it does not require a new subfolder.

Non-interactive scaffold (skips every prompt):

```bash
npx sv create . --template minimal --types ts --no-add-ons --no-dir-check --install npm
```

`--no-dir-check` skips the "directory not empty" prompt — but it does not clear existing files, so scaffolding into a directory that already has files mixes new content into old; prefer a clean or empty target directory. `--add <addon...>` selects add-ons instead of prompting for them; see `sv add` below for add-on syntax. (`--no-download-check` skips download-confirmation prompts; it is accepted by both `sv create` and `sv add`.)

Fold add-ons into the initial `--add` call at creation rather than scaffolding plain and adding them after — `sv create` resolves every `--add` add-on before its one install, so there's no earlier install for a later `sv add` to fight.

Styling stack is a scaffold-time decision, not a silent omission. For app/product scaffolds (e.g. dashboards, SaaS, admin, UI-heavy sites) — and whenever the project will use Tailwind-based component stacks (see `references/libraries.md`) — add `tailwindcss="plugins:none"`:

```bash
npx sv create . --template minimal --types ts --add tailwindcss="plugins:none" --no-dir-check --install npm
```

- `tailwindcss` takes one option: `plugins` with values `typography`, `forms` (multiselect, comma-separated; default `none`). Set it explicitly to skip prompts, e.g. `tailwindcss="plugins:typography,forms"`.
- The add-on adds `@tailwindcss/vite` to the Vite config and creates `src/routes/layout.css` with `@import 'tailwindcss';`, imported by a root `src/routes/+layout.svelte` (v4 is CSS-first: no `tailwind.config.js`, no `@tailwind` directives). Adding it later with `npx sv add tailwindcss` also works.
- Exception: component libraries, headless packages, design-system primitives, and minimal content sites keep scoped CSS plus CSS custom properties — no add-on needed.

For a brand-new **SvelteKit 3** project, scaffold on the `sv@next` line — it creates SvelteKit 3 directly (`@sveltejs/kit@3.0.0-next.*` and `#lib` subpath imports):

```bash
npx -y sv@next create . --template minimal --types ts --add tailwindcss="plugins:none" --no-dir-check --install npm
```

After scaffolding, verify what actually got installed: the `@next` dist-tag floats and template pins can lag the newest preview — a fresh scaffold may resolve an older `3.0.0-next.*`, or an `@sveltejs/adapter-*` line whose Kit 2 peer range conflicts with Kit 3 and fails `npm install` with `ERESOLVE`. Move the preview packages explicitly when needed — `npm install @sveltejs/kit@next @sveltejs/adapter-auto@next` (same dist-tag mechanics as `references/migration.md`).

For an existing SvelteKit 2 project, convert it with `npx -y sv@next migrate sveltekit-3 --tasks all` (see `sv migrate` below). `npx -y` auto-confirms installing the package and `@next` is a floating tag that resolves at run time — verify the concrete version it pulls before relying on it in an existing project.

## `sv add`

Use official add-ons by their current names.
Examples that matter often in this skill:

```bash
npx sv add tailwindcss
npx sv add vitest
npx sv add playwright
```

Bare `--add vitest` still prompts for its options; to stay fully non-interactive, set them explicitly — e.g. `npx sv add vitest="usages:unit,component"`.

Add-on options attach with `=`: a single option is `<addon>=<opt>:<val>`, multiple options combine with `+` (`<addon>=<opt1>:<val1>+<opt2>:<val2>`), and multiselect options take comma-separated values or `none` to clear them. Run `sv add --help` for the current add-on list rather than assuming it is fixed.

- `sveltekit-adapter=` takes one of six choices: `auto` (default), `node`, `static`, `vercel`, `cloudflare`, `netlify` — run `sv add --help` for the current set.
- `sveltekit-adapter="adapter:static"` installs `@sveltejs/adapter-static`. A project scaffolded with it fails `vite build` with `Encountered dynamic routes` until you add `export const prerender = true` to the root `src/routes/+layout.ts` (or configure the adapter's `fallback` for an SPA) — this is standard adapter-static behavior (documented, and unchanged for years), not a SvelteKit 3 regression.
- Every `sv add` run also prompts for a package manager unless you pass `--install <npm|pnpm|yarn|bun|deno>` or `--no-install`, and — if the working directory has uncommitted changes — adds an extra `Verifications failed. Do you wish to continue?` gate (defaults to **No**) unless you pass `--no-git-check`. Both apply no matter which add-on you're installing; see the Experimental feature add-on below for a fully non-interactive example combining these with addon-specific options.

## `sv migrate`

Public migration entry point.

```bash
npx sv migrate
npx sv migrate svelte-5
```

`sv migrate` delegates to migration tooling under the hood, but the public guidance should use `sv`. Run `sv migrate --help` to see which migrations exist in the installed version before assuming one is available or missing. On `sv@latest` it proxies to a separately versioned package, so its migration list doesn't necessarily match what you'd expect from `sv`'s own version number. On the `sv@next` line, migrations are built into `sv` as task-based migrations (`sveltekit-3`, `$app/state`) selectable with `--tasks` — run `npx -y sv@next migrate <name> --tasks` to list a migration's tasks, or `--tasks all` to run every task. `sv migrate` also prompts interactively (dirty-tree gate, package manager, and a final confirmation) — the official non-interactive form is `npx sv@next migrate sveltekit-3 --tasks all --confirm`; add `--no-git-check` and `--install <pm>` / `--no-install` to suppress the remaining prompts.

This section covers command mechanics only. For which migrations to run, in what order, and what to verify afterward, see `references/migration.md` — it owns the workflow.

## `sv check`

Use this to typecheck and surface Svelte diagnostics.
It is a good final validation step after significant edits. Scaffolded projects wire the same check into `npm run check` (`svelte-kit sync && svelte-check --tsconfig ./tsconfig.json`) — treat the two as equivalent, not competing tools.

For direct `svelte-check` use:

- **svelte-check 4.7+** supports `--config <path>` for a non-standard `svelte.config` or `vite.config` location.
- **svelte-check 4.7+** offers experimental `--tsgo`; install `@typescript/native-preview` and expect the same limitations as incremental mode.
- Do not recommend `--tsgo` as the default until the project accepts experimental TypeScript-Go behavior.
- `--ignore` only applies with `--no-tsconfig`. With `--tsconfig` (the default for `sv check`/`npm run check`), exclude paths in `tsconfig.json` itself — e.g. add `"build"` if a build output folder starts appearing in check results after `vite build`.

## Experimental feature add-on (`sv@latest`)

On `sv@latest`, this add-on toggles experimental features (`async`, `remoteFunctions`, `explicitEnvironmentVariables`, `handleRenderingErrors`, `forkPreloads`). For SvelteKit 3 itself, use the `sv@next` line instead — `sv@next create` scaffolds SvelteKit 3 directly and `sv@next migrate sveltekit-3` converts existing projects (see `sv create` / `sv migrate` above). If you do stay on `sv@latest`, set `versions:none` explicitly — `versions` defaults to `kit`, so omitting it silently moves the whole project onto the SvelteKit 3 line.

```bash
npx sv add experimental="versions:none+features:async,remoteFunctions" --no-git-check --install npm
```

- `versions` and `features` are independent multiselect options; `none` clears either one. Always set both explicitly when running non-interactively — leaving one out prompts for it instead of skipping cleanly.
- `--no-git-check`/`--install`/`--no-install` are `sv add`'s own prompts — see `sv add` above.
- On the `sv@next` line this add-on is features-only (`versions:kit` is removed) with features `async`, `remoteFunctions`, `forkPreloads` — run `npx -y sv@next add --help` before writing its flags on that line.

Name every selected flag and preserve the stable-first policy; never run this add-on merely because a project lacks experimental configuration.

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

## `sv` and `@sveltejs/sv-utils` separation (`sv@0.15+`, `@sveltejs/sv-utils@0.2+`)

The CLI is split into separate `sv` and `@sveltejs/sv-utils` packages with an explicit public API.
When scripting around `sv`, prefer the documented public surface; do not rely on private internals.

For add-on authors on `@sveltejs/sv-utils@0.3+`, use `svelteConfig` to find/read/edit either Vite-plugin or legacy-file configuration, and `defineEnv` for version-aware environment imports. Do not hand-roll config-file detection.
