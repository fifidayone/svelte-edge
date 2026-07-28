# Declaration Tags

Use this file for modern local declarations inside Svelte markup.

## Modern target

In new runes-mode components on **Svelte 5.56+**, use declaration tags instead of legacy `{@const}`:

```svelte
<script lang="ts">
	let boxes = $state([
		{ id: 1, width: 10, height: 20 },
		{ id: 2, width: 15, height: 12 }
	]);
</script>

{#each boxes as box (box.id)}
	{const area = box.width * box.height}
	<p>{box.width} × {box.height} = {area}</p>
{/each}
```

Declaration tags can appear anywhere in markup. Their bindings follow lexical scope: they can read outer bindings and are visible to later siblings and descendants in the same scope. Inner declarations may shadow outer names.

## Reactive declarations

In runes mode, use `$state` for writable local state and `$derived` for reactive computed values. A plain `let` or `const` is not a replacement for a rune when reactivity is required.

```svelte
<script lang="ts">
	let user = $state({ name: 'Svelte' });
	let editing = $state(false);
</script>

{#if editing}
	{let draft = $state(user.name)}
	{const greeting = $derived(`Hello ${draft}`)}

	<input bind:value={draft} />
	<p>{greeting}</p>
	<button onclick={() => ((user.name = draft), (editing = false))}>Save</button>
{/if}
```

Keep declarations close to the markup that consumes them, but do not move reusable business logic out of normal modules merely to use this syntax.

## Legacy `{@const}`

Official docs now classify `{@const ...}` as legacy. Do not generate it for new Svelte 5.56+ components.

```svelte
<!-- legacy target -->
{@const area = box.width * box.height}

<!-- modern target -->
{const area = box.width * box.height}
```

Preserve `{@const}` only for a tiny edit in a coherent legacy file or when the project must target Svelte below 5.56. During an explicit migration, convert the whole affected component coherently.

## Toolchain gate

Declaration tags require **Svelte 5.56+**. The first matching official language-tool releases were:

- `svelte-check` **4.5.0+**
- `svelte-language-server` **0.18.1+**
- `svelte2tsx` **0.7.56+**

If the compiler accepts the syntax but the editor, formatter, or linter rejects it, upgrade the toolchain rather than falling back to legacy syntax.

## Hard reminders

- New Svelte 5.56+ markup -> `{let ...}` / `{const ...}`
- Reactive writable declaration -> `$state`
- Reactive computed declaration -> `$derived`
- Do not emit `{@const}` as a compatibility reflex
- Do not hide broadly reusable logic inside markup declarations
