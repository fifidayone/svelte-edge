# Svelte 5 Libraries, Tools & Components Reference

This file is an **ecosystem discovery map**.
It is useful when the user asks for UI libraries, editors, tables, forms, tooling, dev tools, real-time packages, or community components.

It is **not** a canonical source of truth for Svelte or SvelteKit semantics.
If a community package conflicts with official framework guidance, prefer the official framework guidance.

## Discovery sources beyond this file

This file is curated but not exhaustive. For broader Svelte ecosystem discovery:

- **https://madewithsvelte.com** — comprehensive catalog of Svelte components, libraries, sites, and tools
- official Svelte / SvelteKit docs for first-party tooling and packages

Apply the recommendation rubric below to anything discovered externally before recommending it.

## Recommendation rubric

Before recommending a package from this file, quickly check:
- does it clearly support Svelte 5 or current SvelteKit?
- is it actively maintained?
- are the docs still alive and readable?
- is the release history recent enough for production trust?
- is the source/repo credible for the requested use case?

## Official-tool reminder

For testing setup, remember that the official docs and CLI support modern tooling like **Vitest** and **Playwright** directly via `sv add`.
Use `references/testing.md` for canonical test guidance.
Use this file for ecosystem discovery, not to override official testing guidance.

---

Curated list of community libraries, tools, and components for Svelte 5 development (Nov 2024 - May 2026).

## Component Libraries

| Library | Description | Link |
|---|---|---|
| **shadcn-svelte v1** | Official v1.0 with Svelte 5 support — copy-and-paste components (Jul 2025) | https://shadcn-svelte.com/ |
| **shadcn-svelte-extras** | Extends shadcn-svelte with 20+ extra UI components like chat interfaces, file drop zones, and image croppers. | https://shadcn-svelte-extras.com/ |
| **Origin UI - Svelte** | Extensive collection of copy-and-paste components for quickly building app UIs | https://originui-svelte.pages.dev/ |
| **Skeleton v5** | Svelte-native component library with Svelte 5 support, Tailwind 4, modular structure (Nov 2025) | https://www.skeleton.dev/ |
| **SVAR Svelte v2.3** | Feature-rich Svelte UI components including DataGrid and Gantt charts with TypeScript defs (Oct 2025) | https://svar.dev/svelte/ |
| **FlyonUI** | Open-source Tailwind CSS Components Library with semantic classes and powerful JS plugins | https://github.com/themeselection/flyonui |
| **SaapsUI** | Comprehensive Svelte component library for responsive, accessible, and beautiful UIs | https://github.com/sappsdev/sappsui |
| **cnblocks** | 50+ UI & Marketing blocks using Svelte 5, Tailwind CSS v4 and Shadcn Svelte | https://github.com/SikandarJODD/cnblocks |
| **fox ui** | UI components built with Tailwind 4 and Svelte 5, includes rich text editor | https://flo-bit.dev/ui-kit/ |
| **Quaff** | Component library following Material Design 3 guidelines | https://github.com/quaffui/quaff |
| **Svelte Polaris** | Port of Shopify's design system to build Shopify apps | https://svelte-polaris-docs.storebud.workers.dev/ |
| **Ark UI** | Headless UI library with Svelte version | https://ark-ui.com/ |
| **Bits UI v2** | Headless components with attachments, $props.id(), Shadow DOM support (Jun 2025) | https://www.bits-ui.com |
| **Flowbite Svelte** | UI components with datatable and WYSIWYG text editor plugins | https://flowbite-svelte.com/ |
| **RetroUI** | Copy/pastable component library built for Svelte with shadcn-svelte | https://retroui-svelte.netlify.app/ |
| **Tark UI** | Beautiful UI components built with Ark UI and Tailwind | https://www.tarkui.com/ |
| **Uniface Element** | Enterprise-grade UI component library built with Svelte 5 | https://github.com/ticatec/uniface-element |
| **svelte-ui-kit** | Easy-to-use, customizable button component inspired by shadcn/ui style | https://github.com/ChulkovDanila/svelte-ui-kit |
| **Svelte AI Elements** | Component registry built on shadcn-svelte for AI-powered applications | https://github.com/SikandarJODD/ai-elements |
| **AgnosticUI Local v2** | CLI-based UI component library that copies Lit web components into your project | https://github.com/AgnosticUI/agnosticui |
| **Motion Core** | Animated Svelte components powered by GSAP and OGL with a small bundle size | https://github.com/motion-core/motion-core |
| **Zag** | State machine-based UI components, now supports Svelte 5 | https://zagjs.com/overview/introduction |
| **Storybook 9** | Svelte CSF story format with Svelte 5 support | https://storybook.js.org/blog/storybook-9/ |
| **Threlte 8** | 3D framework for Svelte — more performant, flexible, aligned with Svelte 5 | https://threlte.xyz/blog/threlte-8 |
| **Konva.js** | Declarative 2D Canvas for Svelte apps | https://konvajs.org/docs/svelte/index.html |

## Form & Input Libraries

| Library | Description | Link |
|---|---|---|
| **Svelte Form Builder V2** | Drag-and-drop form builder with Shadcn-Svelte, Superforms and schema support | https://svelte-form-builder.vercel.app/ |
| **Formisch** | Schema-based, headless form library for JS frameworks | https://github.com/fabian-hiller/formisch |
| **Conformal** | Type-safe form submissions working with native FormData | https://github.com/marcomuser/conformal |
| **formshape** | Type-safe form validation for SvelteKit Remote Functions using Standard Schema | https://www.npmjs.com/package/formshape |
| **sveltekit-discriminated-fields** | Type-safe discriminated union support for SvelteKit remote function form fields | https://www.npmjs.com/package/sveltekit-discriminated-fields |
| **chain-enhance** | Sequentially chain multiple SvelteKit form actions with deep-merged data | https://github.com/michaelcuneo/chain-enhance |
| **svelte-o-phone** | Flexible, headless phone number input component powered by libphonenumber-js | https://github.com/kevwpl/svelte-o-phone |
| **Cancellable** | Building block adding reactive attributes to button and anchor elements | https://choco-ui.com/blocks/cancellable |
| **svelte-number-format v1** | Lightweight and reactive number input component (stable v1.0) | https://www.npmjs.com/package/svelte-number-format |
| **svelte-image-input** | Component for loading, scaling and adjusting profile pictures | https://github.com/saabi/svelte-image-input |

## Tables, Data Grids & Lists

| Library | Description | Link |
|---|---|---|
| **Tzezar's datagrid** | Easy to use, easy to customize datagrid made in Svelte 5 | https://github.com/tzezar/datagrid |
| **svelte-virtuallists** | Keeps tables and lists efficient — only render visible items | https://github.com/orefalo/svelte-virtuallists |
| **svelte-tablecn** | Powerful data grid and port of tablecn.com | https://github.com/itisyb/svelte-tablecn |
| **TableCraft Engine** | Define table configurations and auto-generate APIs with filtering, sorting, pagination | https://jacksonkasi.gitbook.io/tablecraft |
| **Svelte Sortable List** | Accessible, sortable lists for Svelte applications | https://github.com/rodrigodagostino/svelte-sortable-list |

## Charts, Maps & Visualization

| Library | Description | Link |
|---|---|---|
| **svisx** | Port of Airbnb's visx to Svelte — D3 visualizations for the Svelte ecosystem | https://github.com/xGEMINIx/svisx |
| **LayerChart** | Charting library with "vanilla CSS" mode, Svelte REPL/playground support | https://github.com/techniq/layerchart |
| **Svelte MapLibre GL** | Interactive web maps with MapLibre GL JS and Svelte 5 | https://github.com/MIERUNE/svelte-maplibre-gl |
| **mapcn-svelte** | Svelte port of mapcn built on MapLibre GL, styled with Tailwind | https://github.com/MariusLang/mapcn-svelte |

## Animation, Effects & Transitions

| Library | Description | Link |
|---|---|---|
| **number-flow** | Component to transition, format, and localize numbers | https://github.com/barvian/number-flow |
| **moving icons** | Collection of animated icons based on the lucide icon library | https://www.movingicons.dev/ |
| **svelte-audio-waveform** | Renders customizable waveforms from peak data for music players, podcasts | https://github.com/Catsvilles/svelte-audio-waveform |
| **Svader** | GPU-rendered Svelte components with WebGL and WebGPU fragment shaders | https://github.com/sockmaster27/svader |
| **Svelte Multitone Image** | Simple image renderer to apply multitone effects | https://stephane-vanraes.github.io/svelte-multitoneimage/ |
| **Anime.js v4** | Popular JS animation library — v4 released with Svelte support | https://animejs.com/ |
| **Svelte Interval** | Comprehensive utility for managing intervals with reactive durations | https://github.com/PuruVJ/svelte-interval |
| **SSGOI** | Native app-like page transitions to the web | https://github.com/meursyphus/ssgoi |
| **Tilt Svelte** | Smooth 3D tilt attachment based on vanilla-tilt.js | https://github.com/Savy011/tilt-svelte |
| **Rune Scroller** | "Enchanting" scroll animations for Svelte 5, native performance, zero deps | https://runescroller.lelab.dev/ |
| **scratch-to-reveal-svelte** | Simple and customizable scratch-to-reveal component | https://www.npmjs.com/package/@dellamora/scratch-to-reveal-svelte |
| **USAL JS** | "Ultimate Scroll Animation Library" | https://usal.dev/ |
| **Motion GPU** | Easy way for writing WGSL shaders in Svelte | https://www.motion-gpu.dev/ |
| **svelte-textcircle** | Displays text in a circular layout with customizable animations | https://github.com/LoStis-World/svelte-textcircle |
| **Svelte Spell UI** | Port of the original Spell UI you can copy-paste into any project | https://sv-animations.vercel.app/spell |

## Drag & Drop

| Library | Description | Link |
|---|---|---|
| **sveltednd** | Lightweight, flexible drag and drop library for Svelte 5 | https://github.com/thisuxhq/SvelteDnD |
| **dnd-kit-svelte** | Svelte 5 port of dnd-kit | https://github.com/HanielU/dnd-kit-svelte |
| **Flexiboards** | Headless, reactive drag and drop components for Svelte 5 | https://github.com/Blakintosh/svelte-flexiboards |
| **fluid-dnd** | Drag and drop library for Vue, React and Svelte | https://github.com/carlosjorger/fluid-dnd |
| **@horuse/svelte-dnd** | Drag-and-drop with animated drop previews, auto-scroll, pointer & touch support | https://svelte-dnd.vercel.app/ |
| **@dnd-kit** | Framework-agnostic modern toolkit for drag and drop (Svelte support) | https://dndkit.com/ |

## Icons

| Library | Description | Link |
|---|---|---|
| **svelicon** | Converts Iconify SVG icons to type-safe components with one command | https://github.com/friendofsvelte/svelicon |
| **Monicon** | All-in-one icon library with 200,000+ icons from popular sets | https://github.com/oktaysenkan/monicon |
| **magicons** | Fast, typesafe icon wrapper fixing bundling issues with large barrel exports | https://github.com/propolies/magicons |
| **heroicons-animated** | Open-source collection of 316 smooth animated icons | https://svelte.heroicons-animated.com/ |

## Routers & Navigation

| Library | Description | Link |
|---|---|---|
| **svelte-simple-router** | Client-side router made for Svelte 5 | https://github.com/dvcol/svelte-simple-router |
| **svelte5-router** | SPA router with nested routers support | https://github.com/mateothegreat/svelte5-router |
| **Svelte Mini Router** | Declarative, minimal SPA router for Svelte 5, without SvelteKit | https://github.com/rodrigocfd/svelte-mini-router |
| **sv-router** | Type-safe SPA router with file-based or code-based routing | https://sv-router.vercel.app/ |
| **@hvniel/svelte-router** | Svelte 5 port of React Router | https://github.com/HanielU/svelte-router |
| **cross-router** | Framework-agnostic router wiring navigation state into Svelte reactivity | https://codeberg.org/Bricklou/cross-router |
| **svelte-crumbs** | Automatic, SSR-ready breadcrumbs for SvelteKit via route metadata | https://svelte-crumbs.vercel.app/ |

## State Management

| Library | Description | Link |
|---|---|---|
| **svelte-firebase-state** | Reactive state classes for Firebase Firestore and Realtime Database | https://github.com/pierregoutheraud/svelte-firebase-state |
| **SyncroState** | Multiplayer state like $state but synchronized in realtime, built on Yjs | https://github.com/beynar/syncrostate |
| **youva** | Pagination, debounced search, sorting, filtering and caching for SvelteKit | https://github.com/SikandarJODD/youva |
| **nuqs-svelte** | Type-safe search params state manager (Svelte port of nuqs) | https://github.com/rtrampox/nuqs-svelte |
| **svstate** | Deep reactive proxy with validation, snapshot/undo, and side effects | https://github.com/BCsabaEngine/svstate |
| **rune-sync** | Synchronizes reactive state across various storage backends | https://github.com/antepodeum/rune-sync |
| **Svelte persistent runes** | Reactive rune that keeps its value through pages and reloads | https://github.com/MacFJA/svelte-persistent-runes |
| **Reddo.js** | Tiny undo/redo utility package for JavaScript, React, Vue, and Svelte | https://github.com/eihabkhan/reddojs |
| **svelte-synk** | Tab data synchronisation with leader election | https://github.com/RussBaz/svelte-synk |
| **Tanstack Query Svelte v6** | Runes-based data fetching and caching (Nov 2025) | https://tanstack.com/query/latest/docs/framework/svelte/migrate-from-v5-to-v6 |
| **Stately** | Pinia-inspired state management library providing a structured way to define shared state, mutate it directly and observe changes | https://stately.self.agency |

## Dev Tools & VS Code Extensions

| Library | Description | Link |
|---|---|---|
| **svelte-bundle** | Bundle Svelte components into single HTML files with SSR | https://github.com/uhteddy/svelte-bundle |
| **Svelte-Next** | Automates Svelte version updates | https://svelte-next.codewithshin.com/ |
| **Sveltick** | Lightweight traffic-tracking library for Svelte apps | https://www.npmjs.com/package/sveltick |
| **Svelte Radar** | VS Code extension for visual overview of project routing structure | https://marketplace.visualstudio.com/items?itemName=HarshKothari.svelte-radar |
| **Svelte 5 Snippets** | Reusable code templates for VS Code | https://marketplace.visualstudio.com/items?itemName=thonymg.svelte-5-kit-snippets |
| **SvelteDoc** | VS Code extension showing Svelte component props on hover | https://marketplace.visualstudio.com/items?itemName=burke-development.sveltedoc |
| **svelte-inspect-value** | "JSON tree"-like value inspector with Panels support | https://github.com/ampled/svelte-inspect-value |
| **Svelte 5 MCP Server** | MCP server for Svelte 5 frontend development | https://github.com/StudentOfJS/svelte5-mcp |
| **mcp-svelte-docs** | MCP server to search and access Svelte docs with caching | https://github.com/spences10/mcp-svelte-docs |
| **SV Floating Console** | Floating console for Svelte apps, dev mode only | https://www.npmjs.com/package/sv-console |
| **svelte-grab** | Dev tool capturing component context for LLM coding agents | https://github.com/HeiCg/svelte-grab |
| **Svelte Agentation** | Turns UI annotations into structured context that AI coding agents can understand and act on | https://sv-agentation.com |
| **svelte-fast-check** | Type and Svelte compiler warning checker, up to 24x faster than svelte-check | https://github.com/astralhpi/svelte-fast-check |
| **svelte-breakpoint-badge** | Displays current Tailwind CSS breakpoint during development | https://github.com/AnakKucingTerbang/svelte-breakpoint-badge |
| **Svelte runtime components** | Compile Svelte components from text at runtime | https://github.com/MrGentle/svelte-runtime-components |
| **svelte-runtime-template** | Handle templates at runtime with curly brace substitutions | https://www.npmjs.com/package/svelte-runtime-template |
| **SVG to Svelte** | Quickly converts SVG strings directly into Svelte components | https://github.com/JLAcostaEC/svgtosvelte |
| **svelte-asciiart** | Render ASCII art as scalable SVG with optional grid overlay | https://github.com/xl0/svelte-asciiart |
| **svelte-mainloop** | Wrapper for MainLoop.js handling function registration and cleanup | https://github.com/retrotheft/svelte-mainloop |
| **SVAR Svelte Filter** | Complex filtering UI and logic for data-heavy apps | https://svar.dev/svelte/filter/ |
| **svelte-check-native** | Rust/tsgo drop-in replacement for `svelte-check` with the same flags, output formats and exit codes | https://github.com/harshmandan/svelte-check-native |

## SvelteKit Tools & Adapters

| Library | Description | Link |
|---|---|---|
| **Frizzante** | Minimalistic web server framework using Svelte to render pages | https://github.com/razshare/frizzante |
| **PrevelteKit** | Lightweight framework with Server-Side Pre Rendering using Rsbuild | https://github.com/tbocek/preveltekit |
| **kit-on-lambda** | Adapter for running SvelteKit on AWS Lambda (Node.js and Bun) | https://github.com/beesolve/kit-on-lambda |
| **fastify-svelte-view** | Fastify plugin rendering Svelte components with SSR/CSR support | https://github.com/matths/fastify-svelte-view |
| **warpkit** | Standalone Svelte 5 SPA framework with state-based routing | https://github.com/upstat-io/warpkit |
| **@edgeone/sveltekit** | Deploy SvelteKit to Tencent Cloud EdgeOne Pages | https://pages.edgeone.ai/document/framework-sveltekit |
| **EXE** | Build tool to distribute full-stack web app as a single executable | https://github.com/Hugo-Dz/exe |
| **Vite Plugin Svelte Anywhere** | Embed Svelte components as Custom Elements in any HTML context | https://github.com/vidschofelix/vite-plugin-svelte-anywhere |
| **vite-plugin-svelte-inline-component** | Write tiny Svelte components inside JS/TS tests via tagged-template literals | https://github.com/hanielu/vite-plugin-svelte-inline-component |
| **vite-plugin-sveltekit-decorators** | Auto-decorate SvelteKit functions with wrappers for logging, analytics, etc. | https://github.com/KiraPC/vite-plugin-sveltekit-decorators |
| **Vite Static Assets Plugin** | Auto-scan static assets, generate type-safe TypeScript module | https://www.npmjs.com/package/vite-static-assets-plugin |
| **Composably** | Content processing plugin for Vite/SvelteKit with typed content | https://github.com/kompismoln/composably |
| **Mode Watcher** | Simple light/dark mode management, rewritten with Svelte 5 support | https://github.com/svecosystem/mode-watcher |
| **Sveltepress** | Content-centered site build tool on top of SvelteKit | https://github.com/SveltePress/sveltepress |
| **ptero** | Docusaurus for Svelte | https://ptero.yaoke.pro/ |
| **microfolio** | Static portfolio generator with file-based CMS using Markdown | https://github.com/aker-dev/microfolio |
| **better-svelte-email** | Render emails in Svelte with first-class Tailwind support | https://github.com/Konixy/better-svelte-email |
| **sveltekit-password-protect** | Simple utility to add password protection to websites | https://github.com/humanshield-sidepack/sveltekit-password-protect |
| **sveltekit-image-optimize** | Create an endpoint that optimizes images | https://github.com/humanshield-sidepack/sveltekit-image-optimize |
| **gositemap** | Fast, test-driven sitemap.xml generator for static SvelteKit sites | https://github.com/lelabdev/gositemap |
| **SvelteKit Auto OpenAPI** | Type-safe OpenAPI generation and runtime validation for SvelteKit | https://github.com/SaaSTEMLY/sveltekit-auto-openapi |
| **sveltekit-api-gen** | Auto-generates OpenAPI 3.0 specs from SvelteKit server endpoints | https://github.com/Michael-Obele/sveltekit-api-gen |
| **Apollo Runes** | Apollo Client for Svelte 5 | https://apollo-runes-docs.vercel.app/ |
| **Atom Forge** | Full-stack TypeScript toolkit with Svelte 5 UI components, type-safe RPC and a battle-tested architecture pattern that scales | https://atom-forge.eu/ |

## Real-time & WebSocket

| Library | Description | Link |
|---|---|---|
| **svelte-openai-realtime-api** | Svelte component for using the OpenAI realtime API | https://github.com/flo-bit/svelte-openai-realtime-api |
| **svelte-realtime** | Realtime RPC and reactive subscriptions for SvelteKit | https://svelte-realtime.dev |
| **itty-sockets** | Ultra-tiny WebSocket client with optional public relay server | https://ittysockets.com/ |
| **svelte-socket** | WebSocket wrapper for Svelte 5 using runes | https://www.npmjs.com/package/@hardingjam/svelte-socket |

## Audio Components

| Library | Description | Link |
|---|---|---|
| **Svelte Audio UI** | Accessible, composable audio UI components | https://svelte-audio-ui.vercel.app |

## Text Editors & Content

| Library | Description | Link |
|---|---|---|
| **Edra (ex-ShadEditor)** | Headless & ShadCN-powered rich text editor for Svelte | https://github.com/Tsuzat/Edra |
| **Tipex** | Advanced rich text editor based on Tiptap and Prosemirror | https://www.npmjs.com/package/@friendofsvelte/tipex |
| **svelte-drawer** | Drawer component for Svelte 5, inspired by Vaul | https://github.com/AbhiVarde/svelte-drawer |
| **Markdown UI** | Turn static docs into interactive experiences | https://github.com/BlueprintLabIO/markdown-ui |
| **PDJsonEditor** | JSON visualization and editing with code editor and graph views | https://github.com/podosoft-dev/pdjsoneditor |
| **Show & Svelte** | Create fully interactive presentations with Svelte | https://github.com/retrotheft/show-and-svelte |
| **SvelTTY** | Render and interact with Svelte apps in the terminal | https://github.com/miunau/sveltty |
| **Nabu** | Modular, local-first Svelte block editor engine built on a Single ContentEditable architecture | https://github.com/aionbuilders/nabu |

## Internationalization

| Library | Description | Link |
|---|---|---|
| **Paraglide 2.0** | i18n library for supporting multiple languages in Svelte apps | https://inlang.com/m/gerre34r/library-inlang-paraglideJs |
| **wuchale** | Compile-time i18n toolkit requiring zero code changes | https://github.com/wuchalejs/wuchale |
| **Sveltia I18n** | Internationalization library powered by runes and the messageformat library, formatting messages using Unicode MessageFormat 2 (MF2) | https://github.com/sveltia/sveltia-i18n |

## CLI, Build Tools & Scaffolding

| Library | Description | Link |
|---|---|---|
| **Laravel + Svelte Starter Kit** | Official Laravel starter with Svelte frontend via Inertia | https://github.com/laravel/svelte-starter-kit |
| **jetbrains-svelte-templates** | Live Templates for JetBrains IDEs to speed up Svelte development | https://github.com/ruben-sprengel/jetbrains-svelte-templates |

## UI Utilities & Misc Components

| Library | Description | Link |
|---|---|---|
| **monoco-svelte** | Custom (squircle) corners and borders for Svelte components | https://github.com/monokai/monoco-svelte |
| **diaper** | Advanced bottom sheet component for Svelte 5 | https://github.com/devantic/diaper |
| **bsky-comments-svelte** | Add comments to your website using Bluesky | https://github.com/nsarrazin/bsky-comments-svelte/ |
| **svelte-overflow-fade** | Fade effects to overflowing content (action and attachment) | https://github.com/harshmandan/svelte-overflow-fade |
| **svelte-bash** | Fully typed, customizable terminal emulator component with virtual file system | https://github.com/YusufCeng1z/svelte-bash |
| **trioxide** | Customizable components focused on non-trivial UI pieces | https://github.com/ObelusFi/trioxide |
| **Keycloakify** | Create custom Keycloak themes | https://docs.keycloakify.dev/ |
| **mint** | Digital compositing tool to crop, resize, create collages | https://github.com/mosaiq-software/mint |
| **svelte-tiler** | Small, unstyled library for building tiling user interfaces | https://x0k.dev/svelte-tiler/ |
| **Shimmer From Structure** | Structure-aware skeleton loader mirroring rendered UI at runtime | https://shimmer-from-structure-docs.vercel.app/ |
| **pocket-mocker** | In-page HTTP controller to intercept, modify, simulate API responses | https://github.com/tianchangNorth/pocket-mocker |
| **Avatune** | Production-ready avatar system with AI-powered generation | https://github.com/avatune/avatune |
| **Blossom Color Picker** | Flower-style color picker | https://blossom.dayflow.studio/ |
| **phantom-ui** | Structure-aware skeleton loader built with web components | https://aejkatappaja.github.io/phantom-ui/demo/ |
| **Svileo** | Physics-based toast component inspired by Sileo | https://svileo.elyasasmad.com |

## Other Framework Integrations

| Library | Description | Link |
|---|---|---|
| **keystatic-sveltekit** | Integrate Keystatic CMS with SvelteKit, Markdoc content | https://github.com/Greenheart/keystatic-sveltekit |
| **Svelte Streamdown** | Svelte port of Streamdown — markdown renderer for AI streaming | https://github.com/beynar/svelte-streamdown |
| **components-pack** | Photos-related UI components for Svelte 5, Vue 3, Astro 5, vanilla JS | https://github.com/Matb85/components-pack |
| **Svelte MiniApps** | Collection of user-friendly tools rebuilt with Svelte 5 | https://github.com/Michael-Obele/Svelte-MiniApps |
| **Roguelighter Engine** | Free, open-source 2D game engine | https://github.com/roguelighterengine/roguelighter |
| **Davia** | AI coding agents tool for interactive internal documentation | https://github.com/davialabs/davia |
| **@sheepdog/svelte** | Manage async tasks and concurrency with ease | https://github.com/mainmatter/sheepdog |
| **Neodrag v3** | Multi-framework drag library with event delegation | https://www.puruvj.dev/blog/neodrag-v3-alpha |
| **svelte-5-dashboard** | Boilerplate for Svelte 5 dashboards with alerts, avatars, formatting | https://github.com/thomaslappenbusch/svelte-5-dashboard |
| **oRPC** | Type-safe APIs for Svelte and TanStack Svelte Query | https://github.com/unnoq/orpc |

---

**Note**: Always check the official Svelte Society website (https://sveltesociety.dev) for the latest community packages.
