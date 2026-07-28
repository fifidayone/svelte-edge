# Svelte 5 ecosystem shortlist

Last curated: **2026-07-16**.

Use this file when a user asks for third-party Svelte libraries, components, or tooling. It is a compact decision aid, not an ecosystem archive and not a source for Svelte or SvelteKit semantics.

## Contents

- [Selection contract](#selection-contract)
- [Evidence boundary](#evidence-boundary)
- [Maturity and fit](#maturity-and-fit)
- [UI systems and primitives](#ui-systems-and-primitives)
- [Forms and focused inputs](#forms-and-focused-inputs)
- [Data display and visualization](#data-display-and-visualization)
- [Interaction, motion, and icons](#interaction-motion-and-icons)
- [State, data, realtime, and routing](#state-data-realtime-and-routing)
- [Editors, content, and media](#editors-content-and-media)
- [Internationalization and app integrations](#internationalization-and-app-integrations)
- [Development tooling](#development-tooling)
- [Watchlist](#watchlist)
- [Discovery beyond this file](#discovery-beyond-this-file)

## Selection contract

1. Prefer Svelte/SvelteKit built-ins when they solve the problem cleanly.
2. Recommend at most two packages for one job and explain why each fits.
3. Recheck the live release, Svelte 5/current Kit compatibility, documentation, license, provenance, and migration warnings immediately before recommending.
4. State maturity, fit, and any experimental or platform coupling separately.
5. Measure bundle impact in the target build or a package analyzer. Never invent a bundle-size number.
6. Treat monthly Community Showcase posts as discovery indexes, not endorsements.

First-party CLI, adapters, Svelte MCP, Vitest, and Playwright do not belong in this community shortlist. Use official documentation and the canonical CLI/testing references.

## Evidence boundary

- The shortlist is a human curation decision; it is not generated from popularity alone.
- This file contains only the current shortlist and watchlist. Use Git history for previous selections.
- Absence from this file is a scope decision, not evidence that a project is defective or unavailable.
- An unreachable URL describes that exact reference at that audit time. It does not prove that a project is unavailable elsewhere.
- A missing peer dependency, old commit, low download count, or 404 is a review signal, never an automatic viability verdict.

## Maturity and fit

| Axis | Values | Meaning |
|---|---|---|
| **Maturity** | established / current / experimental / unverified | Strength of current release, maintenance, documentation, and adoption evidence |
| **Fit** | broad / exact / unverified | Broadly reusable choice versus a package for a specific stack or product surface |
| **Provenance** | official-svelte / official-other / vendor / community / unverified | Who owns or publishes the project; this is independent from maturity |

Established means worth normal evaluation, not guaranteed production suitability. Current means current enough to evaluate after a live recheck. Experimental includes prereleases, unpublished packages, or especially unstable integration seams. Keep unverified discoveries out of this file until they have been checked live.

## UI systems and primitives

| Package | Maturity | Fit | Best use and guardrail | Link |
|---|---|---|---|---|
| **shadcn-svelte v1** | established | broad | Copy-owned application UI when the team wants source control over components | [docs](https://shadcn-svelte.com/) |
| **Bits UI v2** | established | broad | Headless accessible primitives and custom design systems | [docs](https://www.bits-ui.com/) |
| **Ark UI** | established | broad | Headless state-machine primitives when cross-framework behavior parity matters | [docs](https://ark-ui.com/docs/overview/getting-started) |
| **Skeleton v4** | established | broad | Themed Svelte application suite; v4 was the stable tag at the audit date | [docs](https://www.skeleton.dev/) |
| **Flowbite Svelte** | established | broad | Broad Tailwind-oriented component suite with Svelte 5 support | [docs](https://flowbite-svelte.com/) |
| **SVAR Svelte** | established | exact | Data-heavy business UI; check product packaging and free-versus-PRO boundaries | [docs](https://svar.dev/svelte/) |

## Forms and focused inputs

Start with native SvelteKit form actions and progressive enhancement. Add a library only when schema coordination, reusable field composition, or difficult input behavior justifies it.

| Package | Maturity | Fit | Best use and guardrail | Link |
|---|---|---|---|---|
| **SvelteKit Superforms 2** | established | broad | Schema-backed server/client forms, validation, and progressive enhancement | [docs](https://superforms.rocks/) |
| **Formsnap 2** | established | broad | Accessible form primitives, especially with Superforms and shadcn-svelte | [docs](https://formsnap.dev/docs/quick-start) |
| **svelte-number-format 2** | current | exact | Locale-aware numeric input; server validation remains separate | [docs](https://pitis.github.io/svelte-number-format/) |
| **svelte-o-phone** | current | exact | Headless international phone input backed by libphonenumber-js | [repo](https://github.com/kevwpl/svelte-o-phone) |
| **Huey** | current | exact | Accessible, composable color-picker primitives with a Svelte 5 package | [docs](https://hueycolor.pages.dev/) |

## Data display and visualization

| Package | Maturity | Fit | Best use and guardrail | Link |
|---|---|---|---|---|
| **TanStack Virtual for Svelte** | established | broad | Large virtualized lists and grids with project-owned semantic markup | [docs](https://tanstack.com/virtual/latest/docs/framework/svelte/svelte-virtual) |
| **LayerChart 2** | established | broad | Svelte-native composable charts with D3 building blocks | [docs](https://www.layerchart.com/) |
| **Svelte MapLibre GL** | established | exact | Interactive MapLibre maps with explicit Svelte 5 support | [repo](https://github.com/MIERUNE/svelte-maplibre-gl) |
| **Threlte 8** | established | exact | Three.js and 3D experiences built around Svelte 5 | [docs](https://threlte.xyz/) |
| **svelte-konva** | established | exact | Retained-mode interactive 2D canvas scenes rather than ordinary DOM UI | [docs](https://konvajs.org/docs/svelte/index.html) |
| **Svelte Event Calendar** | current | exact | Scheduling UI with drag/drop and multiple calendars; check paid feature boundaries | [docs](https://svar.dev/demos/calendar/) |

## Interaction, motion, and icons

| Package | Maturity | Fit | Best use and guardrail | Link |
|---|---|---|---|---|
| **svelte-dnd-action** | established | broad | Drag-and-drop for lists and zones | [repo](https://github.com/isaacHagoel/svelte-dnd-action) |
| **@dnd-kit/svelte** | current | broad | First-party dnd-kit Svelte integration; verify accessibility in the target interaction | [docs](https://dndkit.com/svelte/quickstart) |
| **NumberFlow** | established | exact | Animated, formatted, localized number transitions | [repo](https://github.com/barvian/number-flow) |
| **SSGOI** | current | exact | Svelte page transitions; always respect reduced-motion preferences | [repo](https://github.com/meursyphus/ssgoi) |
| **@lucide/svelte** | established | broad | Small, tree-shakeable general icon set; replace deprecated lucide-svelte | [docs](https://lucide.dev/guide/packages/lucide-svelte) |
| **@iconify/svelte** | established | broad | Very broad icon catalog; verify how selected icon data is bundled | [docs](https://iconify.design/docs/icon-components/svelte/) |

## State, data, realtime, and routing

Use runes and context for ordinary client state, SvelteKit load/form primitives for server data, and SvelteKit routing by default.

| Package | Maturity | Fit | Best use and guardrail | Link |
|---|---|---|---|---|
| **TanStack Query for Svelte v6** | established | broad | Client-side server-state caching and invalidation when Kit primitives are insufficient | [docs](https://tanstack.com/query/latest/docs/framework/svelte/overview) |
| **nuqs-svelte** | current | exact | Typed URL search-parameter state for SvelteKit | [repo](https://github.com/rtrampox/nuqs-svelte) |
| **svelte-realtime** | current | exact | Realtime RPC and reactive subscriptions; verify the deployment runtime | [docs](https://svelte-realtime.dev/) |
| **Apollo Runes** | current | exact | Apollo GraphQL client when Apollo is already the chosen stack | [docs](https://apollo-runes-docs.vercel.app/) |
| **sv-router** | current | exact | Type-safe routing for a deliberately non-Kit Svelte SPA | [docs](https://sv-router.vercel.app/) |

## Editors, content, and media

| Package | Maturity | Fit | Best use and guardrail | Link |
|---|---|---|---|---|
| **Edra** | current | exact | Headless and shadcn-oriented rich-text editing | [repo](https://github.com/Tsuzat/Edra) |
| **Tipex** | current | exact | Tiptap/ProseMirror rich-text editor with Svelte 5 support | [docs](https://tipex.pages.dev/) |
| **EmbedPDF for Svelte** | current | exact | Headless PDF viewer built on PDFium | [docs](https://www.embedpdf.com/svelte-pdf-viewer) |
| **Svelte Streamdown** | established | exact | Streaming Markdown for AI/chat output; keep sanitization and link policy explicit | [repo](https://github.com/beynar/svelte-streamdown) |
| **better-svelte-email** | current | exact | Render email templates with Svelte; verify the installable package and current peer support | [repo](https://github.com/Konixy/better-svelte-email) |
| **Aphex CMS** | current | exact | CMS embedded in SvelteKit; evaluate storage adapters and migration path | [docs](https://getaphex.com/) |
| **keystatic-sveltekit** | current | exact | Keystatic/Markdoc content when Keystatic is the selected backend | [repo](https://github.com/Greenheart/keystatic-sveltekit) |

## Internationalization and app integrations

| Package | Maturity | Fit | Best use and guardrail | Link |
|---|---|---|---|---|
| **Paraglide JS 2** | established | broad | Compile-time, type-safe application internationalization | [docs](https://inlang.com/m/gerre34r/library-inlang-paraglideJs) |
| **wuchale** | current | exact | Compile-time i18n with minimal code changes; verify translator workflow | [repo](https://github.com/wuchalejs/wuchale) |
| **Sveltia I18n** | current | exact | Runes-based Unicode MessageFormat 2 workflows | [repo](https://github.com/sveltia/sveltia-i18n) |
| **sveltekit-openapi-generator** | current | exact | Generate OpenAPI from Kit server endpoints and check generated contracts in CI | [docs](https://oapi.svelte-apps.me/) |
| **oRPC** | established | exact | End-to-end typed APIs when the project deliberately wants an RPC layer | [repo](https://github.com/unnoq/orpc) |
| **Laravel Svelte Starter Kit** | established | exact | Laravel-maintained Inertia starting point for Laravel backends | [repo](https://github.com/laravel/svelte-starter-kit) |

## Development tooling

Canonical checking remains npx sv check; use the testing reference for Vitest and Playwright guidance.

| Package | Maturity | Fit | Best use and guardrail | Link |
|---|---|---|---|---|
| **Storybook 10** | established | broad | Isolated component development and interaction testing | [docs](https://storybook.js.org/docs/get-started/frameworks/svelte-vite) |
| **jscpd** | established | broad | Framework-generic duplicate-code detection with Svelte parsing | [docs](https://jscpd.dev/) |
| **svelte-grab** | current | exact | Capture component context for coding agents during development | [repo](https://github.com/HeiCg/svelte-grab) |
| **Svelte Agentation** | current | exact | Turn visual UI annotations into structured agent context | [site](https://sv-agentation.com/) |
| **svelte-inspect-value** | current | exact | Development-only structured value inspection | [repo](https://github.com/ampled/svelte-inspect-value) |

## Watchlist

Do not recommend these normally. Consider one only for an exact fit, then recheck its canonical documentation, repository, registry metadata, and target-project behavior.

| Package | Maturity | Fit | Dated review signal | Link |
|---|---|---|---|---|
| **Formisch for Svelte** | experimental | exact | npm latest was 1.0.0-rc.0 on 2026-07-16 | [docs](https://formisch.dev/svelte/guides/introduction/) |
| **Svaul** | current | exact | npm latest was 1.1.0; validate drawer accessibility in the target UI | [docs](https://harshmandan.github.io/svaul/) |
| **TabSpot** | current | exact | npm latest was 0.4.0; validate keyboard and assistive-technology behavior | [repo](https://github.com/JLAcostaEC/tabspot) |
| **Svelte Video Editor** | current | exact | npm latest was 1.1.0; validate complex editing behavior and performance | [docs](https://svelte-video-editor.ariefsn.dev/) |
| **svelte-ws** | experimental | exact | Release-channel evidence was incomplete; verify installation and runtime support live | [repo](https://github.com/sowahq/svelte-ws) |
| **sveltekit-cloudflare-do** | current | exact | npm latest was 0.2.1; it is a fork that modifies generated adapter output | [repo](https://github.com/The-LukeZ/sveltekit-cloudflare-do) |
| **pottz** | experimental | exact | npm latest was 0.1.6; validate platform packaging in the target OS matrix | [repo](https://github.com/lucaletizia/pottz) |
| **svelte-docinfo** | current | exact | npm latest was 0.5.4; compare against current TypeScript/language-tool APIs | [docs](https://svelte-docinfo.fuz.dev/) |
| **svelte-check-native** | current | exact | npm latest was 1.0.2; official sv check remains canonical until parity is validated | [repo](https://github.com/harshmandan/svelte-check-native) |

## Discovery beyond this file

Search [Made with Svelte](https://madewithsvelte.com/), recent official monthly Svelte posts, and canonical package sources. Treat every discovery as unverified until it passes the selection contract.
