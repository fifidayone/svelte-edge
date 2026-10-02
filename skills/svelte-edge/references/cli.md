# Official Svelte CLI Commands

Use the `sv` CLI as the public, current command surface.
Do not present `svelte-migrate` as the primary user-facing command.
Treat `sv <command> --help` as authoritative over the examples below — they show shape and intent, not the exhaustive flag/add-on list. Run `--help` before asserting a command can't do something.

## Core commands

```bash
npx sv create my-app
npx sv add eslint prettier tailwindcss vitest playwright
npx sv migrate sveltekit-3 --tasks all --confirm
npx sv check
```

## `sv create`

Create a new Svelte or SvelteKit project.

With **sv 1.0+**, new projects target **SvelteKit 3** and keep Svelte/SvelteKit configuration in the `sveltekit({...})` call in `vite.config` (no `svelte.config.js` is created), and templates use the **`#lib`** subpath import instead of `$lib`. Preserve the scaffolded layout. Do not move config back to the older file out of habit.

On the legacy SvelteKit 2 line, **sv 0.16+–0.17.x** targeted SvelteKit 2.62+ with the same Vite-plugin config layout; a kit-2 project must not be scaffolded or re-scaffolded with `sv` 1.0+ expecting kit-2 output — check `npm ls @sveltejs/kit` after any scaffold. Automatic config discovery in `vite.config.js/ts` requires `svelte-language-server` **0.18.2+** and `svelte-check` **4.6.0+**. Upgrade stale tooling before blaming the new config layout.

The demo template also uses Svelte 5.56 declaration tags. If a generated project shows `{const ...}`, upgrade stale editor/checker tooling instead of rewriting it to legacy `{@const}`.

`path` accepts `.` to scaffold into the current directory — it does not require a new subfolder.

Non-interactive scaffold (skips every prompt):

```bash
npx sv create . --template minimal --types ts --no-add-ons --no-dir-check --install npm
```

`--no-dir-check` skips the "directory not empty" prompt. The scaffold leaves existing directories (`.git`, `.agents`) untouched but overwrites same-named files — a pre-existing `.gitignore` is silently replaced — so scaffold into a clean or empty target directory. `--add <addon...>` selects add-ons instead of prompting for them; see `sv add` below for add-on syntax. (`--no-download-check` skips download-confirmation prompts; it is accepted by both `sv create` and `sv add`.)

Fold add-ons into the initial `--add` call at creation rather than scaffolding plain and adding them after — `sv create` resolves every `--add` add-on before its one install, so there's no earlier install for a later `sv add` to fight.

Styling stack is a scaffold-time decision, not a silent omission. For app/product scaffolds (e.g. dashboards, SaaS, admin, UI-heavy sites) — and whenever the project will use Tailwind-based component stacks (see `references/libraries.md`) — add `tailwindcss="plugins:none"`:

```bash
npx sv create . --template minimal --types ts --add tailwindcss="plugins:none" --no-dir-check --install npm
```

- `tailwindcss` takes one option: `plugins` with values `typography`, `forms` (multiselect, comma-separated; default `none`). Set it explicitly to skip prompts, e.g. `tailwindcss="plugins:typography,forms"`.
- The add-on adds `@tailwindcss/vite` to the Vite config and creates `src/routes/layout.css` with `@import 'tailwindcss';`, imported by a root `src/routes/+layout.svelte` (v4 is CSS-first: no `tailwind.config.js`, no `@tailwind` directives). Adding it later with `npx sv add tailwindcss` also works.
- Exception: component libraries, headless packages, design-system primitives, and minimal content sites keep scoped CSS plus CSS custom properties — no add-on needed (swap `--add tailwindcss=...` for `--no-add-ons` in the scaffold commands; deleting `--add` outright re-prompts for add-ons). Marketing and showcase sites straddle the line: Tailwind when marketing blocks or a Tailwind-based component stack is planned, scoped CSS for bespoke one-off brand design.

After scaffolding, verify what actually got installed (`npm ls @sveltejs/kit`): templates can lag the newest framework release, and an adapter line whose peer range conflicts with the installed Kit major fails `npm install` with `ERESOLVE` — align framework, adapter, and tooling majors together (see `references/sveltekit.md` → Requirements).

For an existing SvelteKit 2 project, convert it with `npx sv migrate sveltekit-3 --tasks all --confirm` (see `sv migrate` below).

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

- `sveltekit-adapter=` takes one of six choices: `auto` (default), `node`, `static`, `vercel`, `cloudflare`, `netlify` — run `sv add --help` for the current set. Current adapter majors peer on SvelteKit 3 (`adapter-auto` 8, `adapter-node` 6, `adapter-static` 4, `adapter-vercel` 7, `adapter-cloudflare` 8, `adapter-netlify` 7).
- `ai-tools` (**sv 0.17.0+**; replaces the retired `mcp` add-on) wires Svelte tooling into AI clients. Options: `ide` (`claude-code`, `cursor`, `gemini`, `opencode`, `vscode`, `other`), `delivery` (`plugin`, `tools`), `tools` (`mcp`, `svelte-code-writer`, `svelte-core-bestpractices`, `svelte-file-editor`), `mcpSetup` (`local`, `remote`) — set every option explicitly (e.g. `ai-tools=ide:claude-code+delivery:tools+tools:mcp+mcpSetup:local`) to stay non-interactive.
- `sveltekit-adapter="adapter:static"` installs `@sveltejs/adapter-static`. A project scaffolded with it fails `vite build` with `Encountered dynamic routes` until you add `export const prerender = true` to the root `src/routes/+layout.ts` (or configure the adapter's `fallback` for an SPA) — this is standard adapter-static behavior (documented, and unchanged for years), not a SvelteKit 3 regression.
- `adapter-auto` (the default) ends a fully successful build with the warning `Could not detect a supported production environment` when no platform is detected — the build is green; it is the cue to pick a real adapter when deploying, not a failure to fix.
- `enhanced-img` (**sv 1.0+**) adds `@sveltejs/enhanced-img` build-time image optimization.
- The prettier add-on writes a `format` script (`prettier --write .`) that formats the whole project root — if the root hosts non-project files (e.g. an `.agents/` directory), add a `.prettierignore` for them or the format run rewrites those files too.
- Every `sv add` run also prompts for a package manager unless you pass `--install <npm|pnpm|yarn|bun|deno>` or `--no-install`, and — if the working directory has uncommitted changes — adds an extra `Verifications failed. Do you wish to continue?` gate (defaults to **No**) unless you pass `--no-git-check`. Both apply no matter which add-on you're installing; see the Experimental feature add-on below for a fully non-interactive example combining these with addon-specific options.

## `sv migrate`

Public migration entry point.

```bash
npx sv migrate
npx sv migrate sveltekit-3 --tasks all --confirm
```

With **sv 1.0+**, migrations are built into `sv` as task-based migrations — `sveltekit-3` (the SvelteKit 2→3 conversion, with prerequisite tasks running automatically and non-automated steps written to `MIGRATION_TASKS.md` plus `@migration-task` comments) and `$app/state`. Run `npx sv migrate <name> --tasks` to list a migration's tasks, or `--tasks all` to run every task. `sv migrate` also prompts interactively (dirty-tree gate, package manager, and a final confirmation) — the official non-interactive form is `npx sv migrate sveltekit-3 --tasks all --confirm`; add `--no-git-check` and `--install <pm>` / `--no-install` to suppress the remaining prompts.

Legacy migrations (`svelte-5`, `svelte-4`, `self-closing-tags`, `sveltekit-2`, `package`, `routes`) still run through `npx sv migrate <name>`, but on sv 1.0+ they delegate to `svelte-migrate@1` and are not task-based — `--tasks` does not apply to them. (On sv 0.17.x and earlier these ran as the older `sv migrate` implementations.)

Run `sv migrate --help` to see which migrations exist in the installed version before assuming one is available or missing.

This section covers command mechanics only. For which migrations to run, in what order, and what to verify afterward, see `references/migration.md` — it owns the workflow.

## `sv check`

Use this to typecheck and surface Svelte diagnostics.
It is a good final validation step after significant edits. Scaffolded projects wire the same check into `npm run check` (`svelte-kit sync && svelte-check --tsconfig ./tsconfig.json`) — treat the two as equivalent, not competing tools.

For direct `svelte-check` use:

- **svelte-check 4.7+** supports `--config <path>` for a non-standard `svelte.config` or `vite.config` location.
- **svelte-check 4.7+** offers experimental `--tsgo`; install `@typescript/native-preview` and expect the same limitations as incremental mode.
- Do not recommend `--tsgo` as the default until the project accepts experimental TypeScript-Go behavior.
- `--ignore` only applies with `--no-tsconfig`. With `--tsconfig` (the default for `sv check`/`npm run check`), exclude paths in `tsconfig.json` itself — e.g. add `"build"` if a build output folder starts appearing in check results after `vite build`.

## Experimental feature add-on (`sv@1.0+`)

On **sv 1.0+**, this add-on enables experimental features only — the `versions` option (moving `@sveltejs/kit` and its adapter to a prerelease line) is removed, and the old `explicitEnvironmentVariables` / `handleRenderingErrors` feature entries are gone (explicit env vars are standard on SvelteKit 3; rendering-error handling is unconditional). Available features: `async` (→ `compilerOptions.experimental.async`), `remoteFunctions` (→ `kit.experimental.remoteFunctions`), `forkPreloads` (→ `kit.experimental.forkPreloads`, off by default).

```bash
npx sv add experimental="features:async,remoteFunctions"
```

- `features` is a multiselect; set it explicitly when running non-interactively — leaving it out prompts for it instead of skipping cleanly.
- `--no-git-check`/`--install`/`--no-install` are `sv add`'s own prompts — see `sv add` above.
- On **sv 0.17.x** (the last 0.x line, for SvelteKit 2-era projects), this add-on still had `versions` (`none|kit`) and features `async`, `remoteFunctions`, `explicitEnvironmentVariables`, `handleRenderingErrors`, `forkPreloads` — omitting `versions` defaulted it to `kit`, silently moving the whole project onto the SvelteKit 3 prerelease line; if you must script against a pinned 0.x, set `versions:none` explicitly.

Name every selected flag and preserve the stable-first policy; never run this add-on merely because a project lacks experimental configuration.

## Declaration-tag toolchain

For Svelte 5.56 declaration tags, require at least:

- `svelte-check` 4.5.0
- `svelte-language-server` 0.18.1
- `svelte2tsx` 0.7.56

## TypeScript 6 toolchain

For TypeScript 6 projects, require at least `svelte-language-server` **0.18.0**, `svelte2tsx` **0.7.55**, `svelte-check` **4.4.8**, and — when used — `svelte-preprocess` **6.0.4**. Upgrade this set together when diagnostics disagree across the editor and CI.

## Community add-ons (`sv@0.14+`)

Community add-ons are discoverable via the official CLI. Since **sv 1.0** the community add-on API is official (no longer experimental), community add-ons no longer require scoped package names, and an `addon` template is selectable in `sv create` for authoring them.
Recommend a specific community add-on only when it clearly benefits the project — do not promote unvetted add-ons by reflex.
Svelte maintainers do not review community add-ons for malicious code; do not use them in production without an independent source and maintenance review.

## `sv` and `@sveltejs/sv-utils` separation (`sv@0.15+`, `@sveltejs/sv-utils@0.2+`)

The CLI is split into separate `sv` and `@sveltejs/sv-utils` packages with an explicit public API.
When scripting around `sv`, prefer the documented public surface; do not rely on private internals.

For add-on authors on `@sveltejs/sv-utils@0.3+`, use `svelteConfig` to find/read/edit either Vite-plugin or legacy-file configuration, and `defineEnv` for version-aware environment imports. Do not hand-roll config-file detection.
