# Attachments and Modern DOM Behavior

For new DOM behavior in Svelte 5, prefer attachments when composition or reactivity matters.

Attachments are the modern option for many `use:` action scenarios.
They provide a cleaner way to attach lifecycle-aware DOM behavior, but they are not an instruction to rewrite every simple existing action immediately.

## Contents

- [Basic and parameterized attachments](#basic-attachment)
- [Cleanup and reactive reruns](#cleanup-capable-attachment)
- [Forwarding through components](#passing-attachments-through-components)
- [Action interop](#bridging-from-actions-with-fromaction)
- [When to keep actions](#when-to-keep-using-legacy-actions)

## Basic attachment

```svelte
<script lang="ts">
	function autofocus(node: HTMLElement) {
		node.focus();
	}
</script>

<input {@attach autofocus} />
```

## Parameterized attachment

```svelte
<script lang="ts">
	function tooltip(text: string) {
		return (node: HTMLElement) => {
			node.setAttribute('title', text);
		};
	}

	let label = $state('Save changes');
</script>

<button {@attach tooltip(label)}>Save</button>
```

Attachment expressions are reactive. `{@attach tooltip(label)}` reruns when `tooltip`, `label`, or state read during the attachment setup changes.

## Cleanup-capable attachment

```svelte
<script lang="ts">
	function clickOutside(node: HTMLElement, onoutside: () => void) {
		function handle(event: MouseEvent) {
			if (!node.contains(event.target as Node)) onoutside();
		}

		document.addEventListener('click', handle, true);
		return () => document.removeEventListener('click', handle, true);
	}

	let open = $state(true);
</script>

{#if open}
	<div {@attach clickOutside(() => (open = false))}>menu</div>
{/if}
```

## Factories, inline attachments, and reruns

Factories that return attachments are the usual shape for parameterized behavior:

```ts
function tooltip(text: string) {
	return (node: HTMLElement) => {
		node.title = text;
		return () => node.removeAttribute('title');
	};
}
```

Inline attachments are fine for tiny one-off DOM work. If setup is expensive, keep setup in the outer attachment and put changing data in a nested `$effect` or getter so the whole attachment is not destroyed and recreated on every value change.

Falsy values such as `false` or `undefined` mean "no attachment", which is useful for conditional behavior.

## Passing attachments through components

When `{@attach ...}` is used on a component, Svelte passes it as a symbol-keyed prop.
Wrapper components should spread remaining props onto the actual element to forward attachments:

```svelte
<script lang="ts">
	import type { Snippet } from 'svelte';
	import type { HTMLButtonAttributes } from 'svelte/elements';

	let { children, ...props }: HTMLButtonAttributes & { children?: Snippet } = $props();
</script>

<button {...props}>{@render children?.()}</button>
```

## Bridging from actions with `fromAction`

If you already have a legacy action and need to keep using it while moving toward attachments, use `fromAction` from `svelte/attachments`.

```svelte
<script lang="ts">
	import { fromAction } from 'svelte/attachments';
	import { tooltip } from '$lib/legacy-actions';

	let text = $state('Helpful text');
</script>

<button {@attach fromAction(tooltip, () => text)}>Hover me</button>
```

The second argument to `fromAction` should return the action argument itself.
Do not wrap that return value in an extra function layer by mistake.

## `createAttachmentKey`

Use `createAttachmentKey` when you need to apply attachments programmatically through spread patterns or helper composition.
This is an advanced API. Reach for it only when normal `{@attach ...}` syntax is not enough.

## When to keep using legacy actions

Keep `use:` actions when:
- the task is migration work
- the user explicitly needs compatibility with older Svelte versions
- you are touching a legacy file and not modernizing it yet
- the existing action is simple and already fits the codebase well
- you are interoperating with an ecosystem package that still exposes an action API

For new code, prefer attachments when they make the DOM behavior more composable or reactive.

- `use:`, `{@attach}`, `bind:`, `on:`, `transition:`, and `animate:` on the same element run in the order they're written in the markup, not by directive type. This is current compiler behavior, not a documented API contract.

## Hard reminders

- New DOM behavior -> prefer attachments when composition or reactivity matters
- Attachment expressions rerun reactively; avoid accidental expensive teardown/setup loops
- Component wrappers must spread props if they are expected to forward attachments
- Existing legacy file without migration request -> preserve simple actions if needed
- Ecosystem interop -> keeping `use:` can be the right choice
- Migration path -> `fromAction(...)` is the safe bridge
- Do not claim attachments work below **Svelte 5.29+**
