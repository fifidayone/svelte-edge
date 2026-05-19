# Animations and Transitions

Use this file for `svelte/motion`, `transition:`, `in:`, `out:`, and `animate:` directives, built-in transitions/animations, easing, and custom motion functions.

## Three directives — pick the right one

| Directive | When element enters/exits DOM | When reordered in keyed each |
|---|---|---|
| `transition:` | bidirectional, reversible mid-flight | — |
| `in:` / `out:` | separate enter/exit, not reversible | — |
| `animate:` | — | runs on index change |

`transition:` reverses smoothly if interrupted. `in:` / `out:` lets you mix different enter/exit effects but treats them independently — an `in` continues alongside an interrupting `out` rather than reversing.

`animate:` only runs when the *index* of an existing item changes in a **keyed** `{#each}` block. It does not fire on add/remove.

## Built-in transitions (`svelte/transition`)

Imports: `blur`, `crossfade`, `draw`, `fade`, `fly`, `scale`, `slide`.

```svelte
<script lang="ts">
	import { fade, fly } from 'svelte/transition';
	let visible = $state(false);
</script>

{#if visible}
	<div transition:fade={{ duration: 200 }}>fades both ways</div>
	<div in:fly={{ y: 20 }} out:fade>flies in, fades out</div>
{/if}
```

Common parameters across most transitions: `delay`, `duration`, `easing`. Per-transition extras:

- `blur`: `amount`, `opacity`
- `fade`: base three only
- `fly`: `x`, `y`, `opacity`
- `slide`: `axis: 'x' | 'y'`
- `scale`: `start`, `opacity`
- `draw`: `speed`, `duration` — SVG `<path>` / `<polyline>` only (element must have `getTotalLength`)
- `crossfade`: factory that returns a `[send, receive]` pair for moving elements between containers

## Local vs global

Transitions are **local** by default — they only play when the block they belong to is created or destroyed, not when a parent block is. Append `|global` to opt into outer-block triggering:

```svelte
{#if x}
	{#if y}
		<p transition:fade>plays only when y changes</p>
		<p transition:fade|global>plays when x or y change</p>
	{/if}
{/if}
```

## Animations (`svelte/animate`)

Only `flip` ships built in. Use it for list reorder animations:

```svelte
<script lang="ts">
	import { flip } from 'svelte/animate';

	let list = $state([{ id: 1, name: 'a' }, { id: 2, name: 'b' }]);
</script>

{#each list as item (item.id)}
	<li animate:flip={{ duration: 200 }}>{item.name}</li>
{/each}
```

`animate:` requires:

- a **keyed** `{#each}` block
- the directive on an element that is the **immediate child** of the each block

`flip` stands for First-Last-Invert-Play. The animation function receives `{ from, to }` `DOMRect`s describing the element's start and end positions.

## Easing (`svelte/easing`)

All easing functions are `(t: number) => number` and interchangeable with motion classes (`Tween`, `Spring`) and any custom transition/animation:

- `linear`
- `backIn`, `backOut`, `backInOut`
- `bounceIn`, `bounceOut`, `bounceInOut`
- `circIn`, `circOut`, `circInOut`
- `cubicIn`, `cubicOut`, `cubicInOut`
- `elasticIn`, `elasticOut`, `elasticInOut`
- `expoIn`, `expoOut`, `expoInOut`
- `quadIn`, `quadOut`, `quadInOut`
- `quartIn`, `quartOut`, `quartInOut`
- `quintIn`, `quintOut`, `quintInOut`
- `sineIn`, `sineOut`, `sineInOut`

```svelte
<script lang="ts">
	import { fly } from 'svelte/transition';
	import { cubicOut } from 'svelte/easing';
</script>

<div in:fly={{ y: 20, easing: cubicOut }}>...</div>
```

## Motion (`svelte/motion`)

Prefer the modern class-based APIs. `spring` and `tweened` stores are deprecated; use `Spring` and `Tween`.

- `prefersReducedMotion` is available in **Svelte 5.7+** and exposes `.current`.
- `Spring` / `Tween` classes and option types are available in **Svelte 5.8+**.
- `Spring.of(() => value)` and `Tween.of(() => value)` bind motion to a reactive value and must be created inside an effect root, such as component initialization.
- `Spring#set(value, options)` and `Tween#set(value, options)` return promises that resolve when `current` catches up.
- `SpringUpdateOptions` supports `instant` and `preserveMomentum`; legacy `hard` / `soft` only apply to the deprecated store API.
- `TweenOptions` supports `delay`, `duration`, `easing`, and `interpolate`.

```svelte
<script lang="ts">
	import { Spring, prefersReducedMotion } from 'svelte/motion';

	let { target }: { target: number } = $props();
	const x = Spring.of(() => target);

	function snap() {
		x.set(0, { instant: prefersReducedMotion.current });
	}
</script>

<button onclick={snap}>reset</button>
<div style={`transform: translateX(${x.current}px)`}>box</div>
```

## Custom transitions

A custom transition is a function returning a config object:

```ts
type Transition = (
	node: HTMLElement,
	params: unknown,
	options: { direction: 'in' | 'out' | 'both' }
) => {
	delay?: number;
	duration?: number;
	easing?: (t: number) => number;
	css?: (t: number, u: number) => string;
	tick?: (t: number, u: number) => void;
};
```

Prefer `css` over `tick` when possible — web animations run off the main thread:

```svelte
<script lang="ts">
	import { elasticOut } from 'svelte/easing';

	function whoosh(node: HTMLElement, params: { duration?: number; easing?: (t: number) => number }) {
		const existingTransform = getComputedStyle(node).transform.replace('none', '');
		return {
			duration: params.duration ?? 400,
			easing: params.easing ?? elasticOut,
			css: (t: number) => `transform: ${existingTransform} scale(${t})`
		};
	}

	let visible = $state(false);
</script>

{#if visible}
	<div in:whoosh>whooshes in</div>
{/if}
```

`t` runs `0 → 1` for `in`, `1 → 0` for `out`; `u = 1 - t`. `1` is the element's natural state.

## Custom animations

```ts
type Animation = (
	node: HTMLElement,
	geometry: { from: DOMRect; to: DOMRect },
	params: unknown
) => {
	delay?: number;
	duration?: number;
	easing?: (t: number) => number;
	css?: (t: number, u: number) => string;
	tick?: (t: number, u: number) => void;
};
```

The animation function receives the start and end `DOMRect`s so you can compute deltas. Same `css`-over-`tick` preference applies.

## Transition events

Elements with a transition dispatch these in addition to DOM events:

- `onintrostart`
- `onintroend`
- `onoutrostart`
- `onoutroend`

```svelte
<p
	transition:fly={{ y: 200 }}
	onintroend={() => (status = 'in done')}
	onoutroend={() => (status = 'out done')}
>
	flies in and out
</p>
```

## Reduced-motion respect

Use `prefersReducedMotion` from `svelte/motion` (covered in `references/runes.md`) and short-circuit transitions when the user opts out:

```svelte
<script lang="ts">
	import { prefersReducedMotion } from 'svelte/motion';
	import { fly } from 'svelte/transition';

	let visible = $state(false);
</script>

{#if visible}
	<div transition:fly={{ y: prefersReducedMotion.current ? 0 : 20 }}>...</div>
{/if}
```

## Hard reminders

- `transition:` is bidirectional and reversible. `in:` / `out:` is one-way each direction and does not reverse mid-flight.
- `animate:` requires a keyed `{#each}` and must sit on the immediate child element.
- Prefer `css` over `tick` in custom transitions — keeps animation off the main thread.
- Transitions are local by default — use `|global` to play on parent block changes.
- Respect `prefersReducedMotion.current` for accessibility.
- Reach for `svelte/transition` built-ins before writing custom ones.
- Prefer `Spring` / `Tween` over deprecated motion stores.
