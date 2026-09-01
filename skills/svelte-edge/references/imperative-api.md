# Imperative Component API

Use this file for Svelte 5 roots, widgets, tests, and integrations that mount components outside normal Svelte composition.

## Current target

Svelte 5 components are functions, not classes. Use `mount`, `hydrate`, and `unmount` from `svelte` instead of `new Component(...)` and component instance methods.

```ts
import { mount, unmount } from 'svelte';
import App from './App.svelte';

const app = mount(App, {
	target: document.querySelector('#app')!,
	props: { name: 'Ada' }
});

await unmount(app, { outro: true });
```

## API selection

- `mount(Component, { target, props })`: create a new client-rendered root.
- `hydrate(Component, { target, props })`: reuse server-rendered HTML.
- `unmount(app)`: remove a mounted/hydrated root.
- `unmount(app, { outro: true })`: await outro transitions before removal.

`mount` and `hydrate` do not run effects synchronously. Use `flushSync()` only when a test or imperative integration must immediately observe pending effects or DOM changes.

`bind:this` on a component returns its exported API, not a Svelte 4 class instance. Do not call `$set`, `$on`, or `$destroy`; update reactive props/callbacks and use `unmount` for teardown.

## Server-side render results

- `render(Component)` from `svelte/server` returns `{ body, head }` — the `html` property is deprecated (`html_deprecated`); use `body`. Since **Svelte 5.57+**, `svelte/server` also exports the `RenderOutput`, `SyncRenderOutput`, `Csp`, and `Sha256Source` types for typing server render output and CSP nonce/hash sources.

## Migration boundary

Avoid `asClassComponent` and other legacy class bridges in new code. Keep one only as a temporary adapter when an external API still requires Svelte 4-style `$set`, `$on`, or `$destroy`; isolate it at that boundary and do not spread class-component assumptions into new components.

For component-value typing, use `Component<Props>` from `svelte`. Do not introduce deprecated `SvelteComponent` or `ComponentType` types.

## Hard reminders

- New imperative root -> function API
- SSR markup reuse -> `hydrate`, not `mount`
- Outro-aware teardown -> await `unmount(..., { outro: true })`
- Do not expect synchronous effects after `mount`/`hydrate`
