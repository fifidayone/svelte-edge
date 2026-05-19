# Runes, Modern Reactivity, Context, and Bindings

Use this file for modern Svelte component semantics.
For new Svelte 5 components, default to runes mode.

## Default runes

### `$state`
Use `$state(...)` for reactive component state.

```svelte
<script lang="ts">
	let count = $state(0);
	let user = $state({ name: 'Ada', online: true });

	function increment() {
		count += 1;
	}
</script>

<button onclick={increment}>{count}</button>
<p>{user.name}</p>
```

### `$derived`
Use `$derived(...)` or `$derived.by(...)` for computed values.
Prefer this over using `$effect` to keep another piece of state in sync.

```svelte
<script lang="ts">
	let items = $state([{ price: 4 }, { price: 7 }]);
	let total = $derived(items.reduce((sum, item) => sum + item.price, 0));
</script>

<p>Total: {total}</p>
```

```svelte
<script lang="ts">
	let search = $state('');
	let products = $state(['Svelte', 'Kit', 'Rune']);
	let filtered = $derived.by(() => {
		let q = search.toLowerCase();
		return products.filter((p) => p.toLowerCase().includes(q));
	});
</script>
```

`$derived` tracks synchronous state reads inside its expression or callback.
You may temporarily override a derived value for optimistic UI, but do not use that as a substitute for real source state when the value is genuinely writable.

### `$effect`
Treat `$effect` as an escape hatch for browser-only side effects, subscriptions, imperative APIs, and synchronization with systems outside Svelte.

Use `$effect.pre(...)` only when you explicitly need pre-update timing.
Use `$effect.root(...)` only for advanced manual effect lifetimes.
Use `$effect.tracking()` only inside helpers that need to know whether they are running in a tracking context.
Use `$effect.pending()` only with async mode when you need pending state for later async updates.

**Important:** generally do **not** update one piece of state from another inside `$effect`. Prefer `$derived` instead.

```svelte
<script lang="ts">
	let canvas: HTMLCanvasElement;
	let width = $state(300);
	let height = $state(150);

	$effect(() => {
		let ctx = canvas.getContext('2d');
		if (!ctx) return;
		ctx.clearRect(0, 0, width, height);
		ctx.fillRect(10, 10, 100, 40);
	});
</script>

<canvas bind:this={canvas} {width} {height}></canvas>
```

### `$props`
Use `$props()` for component props.
Assume TypeScript in examples unless the task strongly suggests otherwise.

```svelte
<script lang="ts">
	interface Props {
		label: string;
		disabled?: boolean;
	}

	let { label, disabled = false }: Props = $props();
</script>

<button {disabled}>{label}</button>
```

Do not mutate props you do not own. Use callback props for one-way communication, or `$bindable()` only when the parent and child intentionally share writable state.

### `$props.id()`
Available in **Svelte 5.20+**.
Use `$props.id()` for SSR/hydration-stable component-instance IDs, especially `for`, `aria-labelledby`, and related attributes.

```svelte
<script lang="ts">
	const uid = $props.id();
</script>

<label for="{uid}-email">Email</label>
<input id="{uid}-email" />
```

### `$bindable`
Use `$bindable()` only when component consumers genuinely need two-way binding.
Do not make everything bindable by reflex.

```svelte
<script lang="ts">
	let { value = $bindable('') } = $props();
</script>

<input bind:value />
```

### `$inspect`
Use `$inspect(...)` for development-time debugging.
Do not make production logic depend on it.

### `$host`
Use `$host()` only inside custom element components to access the host element, commonly to dispatch custom DOM events.
For normal Svelte component communication, prefer callback props or bindable props instead.

## Reactivity control

Use these sparingly. If ordinary `$derived` / `$effect` expresses the behavior, prefer that.

- `untrack(() => value)` reads state inside `$derived` or `$effect` without making it a dependency. Use it when a value is metadata for the side effect, not a trigger.
- `flushSync(() => work())` synchronously flushes pending updates and returns the callback result. Use it mostly in tests, low-level integrations, or imperative code that must observe updated DOM immediately.

```ts
import { untrack } from 'svelte';

$effect(() => {
	save(data, { timestamp: untrack(() => now) });
});
```

## Lifecycle and scheduling

Prefer runes for reactive work. Use lifecycle/scheduling APIs for their narrow jobs:

- `onMount(...)`: browser-only work that runs once after mount; not SSR.
- `onDestroy(...)`: cleanup before unmount; it is the lifecycle hook that can also run during SSR.
- `tick()`: await pending DOM updates after state changes.
- `settled()`: **Svelte 5.36+**, await state changes plus async work caused by them.
- `flushSync(...)`: force pending updates synchronously, mostly for tests and imperative integration.

Do not migrate old `beforeUpdate` / `afterUpdate` patterns directly; use `$effect.pre` / `$effect` instead.

## Bindings

Use `bind:` for deliberate two-way data flow, not as the default for every prop.

- Function bindings `bind:value={get, set}` are available in **Svelte 5.9+** and are useful for validation/transforms.
- For readonly bindings such as dimensions, the getter should be `null`: `bind:clientWidth={null, redraw}`.
- `bind:this` is `undefined` until mount; read it in an effect or event handler, not during component initialization.
- `bind:files` needs a `FileList`; create one with `DataTransfer` when clearing or replacing files, and avoid constructing it during SSR.
- Component bindings require the child prop to use `$bindable()`. A fallback on a bound prop still expects the parent to provide a non-`undefined` value when bound.

## State helpers

### `$state.raw(...)`
Use this for non-proxied values when proxy behavior is not wanted.

### `$state.snapshot(...)`
Use this to capture plain immutable snapshots for logging, serialization, or interop.

### `$state.eager(...)`
Available in **Svelte 5.41+**.
Use sparingly when immediate UI feedback matters more than waiting for coordinated async updates.

## Reactive classes

Class instances are not proxied. Use `$state` directly on class fields (public or private), or assign it as the first thing inside the constructor:

```ts
class Todo {
	done = $state(false);
	text = $state('');

	constructor(text: string) {
		this.text = text;
	}

	// Arrow method binds `this` correctly when passed as an event handler
	reset = () => {
		this.text = '';
		this.done = false;
	};
}
```

**`this` gotcha:** regular class methods lose `this` when used directly as event handlers. Either define the method as an arrow on the class (as above) or wrap with an inline function at the call site:

```svelte
<!-- breaks — `this` becomes the button element -->
<button onclick={todo.reset}>reset</button>

<!-- correct -->
<button onclick={() => todo.reset()}>reset</button>
```

`$derived` and `$state.raw` work the same way as class fields:

```ts
interface Item {
	price: number;
}

class Cart {
	items = $state<Item[]>([]);
	total = $derived(this.items.reduce((sum, item) => sum + item.price, 0));
}
```

The compiler transforms `$state` fields into private fields with `get` / `set` methods on the prototype. The properties are **not enumerable** — `Object.keys(instance)` will not list them.

Use reactive classes when:
- related state and behavior want to live together
- you prefer method-based mutation over free-form assignment
- the state has invariants worth enforcing through getters/setters

## Sharing state across modules

In `.svelte.ts` / `.svelte.js` files, you can declare reactive state, but you **cannot export a directly reassignable `$state`**. The compiler rewrites references file-locally; another module importing the binding would see a stale value. Two valid patterns:

```ts
// option 1 — export an object whose properties you mutate
export const counter = $state({ count: 0 });

export function increment() {
	counter.count += 1;
}
```

```ts
// option 2 — keep state private, expose accessors
let count = $state(0);

export function getCount() {
	return count;
}
export function increment() {
	count += 1;
}
```

For typed shared state with subtree scoping, prefer `createContext()` (see [Context](#context)) over module-scoped state.

In SvelteKit SSR, **never** put per-user mutable state in shared module scope — it leaks between requests. See `references/sveltekit.md`.

## Universal reactivity across modules (`.svelte.ts` / `.svelte.js`)

In Svelte 5, runes can live outside components in files such as `.svelte.ts` and `.svelte.js`.
Use this for shared reactive helpers and app state that should live beyond a single component.

```ts
// counter.svelte.ts
export function createCounter() {
	let count = $state(0);

	return {
		get count() {
			return count;
		},
		increment() {
			count += 1;
		}
	};
}
```

Use this for shared client-side state or non-user-specific shared logic.
In SvelteKit SSR, do **not** put per-user mutable state in a shared server module.
Use context, `locals`, cookies, or other request-scoped data for that.

## Context

Use `createContext()` in **Svelte 5.40+** when you want typed context helpers.
Keep the helper in a normal module like `context.ts`, not inside a `.svelte` component wrapper.

```ts
// context.ts
import { createContext } from 'svelte';

export interface UserContext {
	name: string;
	role: 'admin' | 'user';
}

export const [getUserContext, setUserContext] = createContext<UserContext>();
```

```svelte
<!-- Parent.svelte -->
<script lang="ts">
	import { setUserContext } from './context';

	setUserContext({ name: 'Ada', role: 'admin' });
</script>
```

```svelte
<!-- Child.svelte -->
<script lang="ts">
	import { getUserContext } from './context';

	const user = getUserContext();
</script>

<p>{user.name}</p>
```

If the project version is below 5.40, fall back only when the user explicitly needs compatibility.

Prefer context when shared state should be scoped to one subtree rather than process-wide module state.

## Reactivity utilities (`svelte/reactivity`)

These are official modern helpers and are worth knowing.
They are not legacy extras.

### `MediaQuery`
Use `MediaQuery` when component logic truly depends on a media query.
Prefer pure CSS when CSS alone solves the problem.

**Hydration caveat:** during SSR there is no real browser media state, so content can shift after hydration.

```svelte
<script lang="ts">
	import { MediaQuery } from 'svelte/reactivity';

	const large = new MediaQuery('(min-width: 768px)');
</script>

{#if large.current}
	<DesktopNav />
{:else}
	<MobileNav />
{/if}
```

### Reactive built-ins
Use these when the reactive shape itself is useful:
- `SvelteDate`
- `SvelteMap`
- `SvelteSet`
- `SvelteURL`
- `SvelteURLSearchParams`

They behave like their native counterparts but participate in Svelte's reactivity.

### `createSubscriber`
Use `createSubscriber` when building your own reactive wrapper for external event sources.
This is the right abstraction for observers and browser APIs when you want reactive reads without manual listener wiring everywhere.

```ts
import { createSubscriber } from 'svelte/reactivity';

export function fromEventTarget<T>(target: EventTarget, type: string, read: () => T) {
	const subscribe = createSubscriber((update) => {
		const handler = () => update();
		target.addEventListener(type, handler);
		return () => target.removeEventListener(type, handler);
	});

	return {
		get current() {
			subscribe();
			return read();
		}
	};
}
```

Use patterns like this for observers, browser APIs, or small reactive wrappers around external systems.

## Window reactivity (`svelte/reactivity/window`)

Use this for reactive window values such as:
- `innerWidth`
- `innerHeight`
- `outerWidth`
- `outerHeight`
- `scrollX`
- `scrollY`
- `online`
- `devicePixelRatio`
- `screenLeft`
- `screenTop`

These exports provide a reactive `.current` property and let you avoid manual listeners or `<svelte:window>` bindings for these values.

```svelte
<script lang="ts">
	import { innerWidth, scrollY, online } from 'svelte/reactivity/window';
</script>

<p>{innerWidth.current}px wide</p>
<p>Scroll: {scrollY.current}</p>
<p>{online.current ? 'Online' : 'Offline'}</p>
```

Use `<svelte:window>` when you need actual window event listeners or bindings outside these built-in reactive values.

## Stores interop

For new app state, prefer runes and `.svelte.ts` / `.svelte.js` modules.
Keep `svelte/store` for legacy code, libraries that expose store contracts, Observable/RxJS-style interop, or manual subscription APIs.
Do not introduce stores just to share ordinary Svelte 5 state.

## Hard reminders

- Default to runes mode for new components
- Prefer `$derived` over `$effect` for computed state
- Use lifecycle APIs only for lifecycle jobs; do not recreate Svelte 4 update patterns
- Use `.svelte.ts` / `.svelte.js` modules for shared universal reactivity when module-scoped state is the right fit
- Prefer runes over stores for new app state
- Use `createContext()` for typed, subtree-scoped context when available
- Use `svelte/reactivity/window` for standard reactive window values
- Use `MediaQuery` only when component logic needs it, and remember SSR caveats
- For `Spring`, `Tween`, and transitions, read `references/animations-transitions.md`
