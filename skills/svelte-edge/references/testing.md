# Testing: Vitest, Component Tests, Storybook, and Playwright

Use this file when the user asks about testing strategy, test setup, component tests, Storybook testing, or end-to-end testing.

This file owns:
- Svelte-specific test tooling and setup (`sv add vitest`, SSR/client-aware config)
- component testing patterns specific to runes, snippets, and bindings
- async UI testing expectations
- rune-usage gotchas inside test files

It does not restate generic cross-framework testing philosophy except where a rule has a Svelte-specific failure mode attached to it.
For async behavior, boundaries, and await semantics, trust `async-svelte.md`.
For SvelteKit routing/form/server behavior, trust `sveltekit.md`.

## Contents

- [Svelte-specific setup](#svelte-specific-setup)
- [Testing doctrine for this skill](#testing-doctrine-for-this-skill)
- [Using runes in test files](#using-runes-in-test-files)
- [Component testing patterns](#component-testing-patterns)
- [Storybook](#storybook)
- [End-to-end tests with Playwright](#end-to-end-tests-with-playwright)
- [What to test in async-first UI](#what-to-test-in-async-first-ui)
- [SvelteKit-specific testing reminders](#sveltekit-specific-testing-reminders)
- [Hard reminders](#hard-reminders)

## Svelte-specific setup

Use `npx sv add vitest` rather than manually wiring Vitest into a Svelte/SvelteKit project.
Unlike a generic Vitest install, this scaffolds **client/server-aware** testing config in the Vite
config file and wires the Svelte Vite plugin into the test pipeline.

```bash
npx sv add vitest
npx sv add playwright
npx sv add storybook
```

`sv add playwright` and `sv add storybook` scaffold the equivalent baseline setup for E2E and
component-documentation workflows respectively; treat them as the Svelte-aware starting point.

## Testing doctrine for this skill

When the user asks for tests:
- default to TypeScript test files for new SvelteKit projects unless the task or codebase clearly calls for JavaScript
- default to Vitest for logic, stores, utilities, and isolated component tests
- default to Playwright for route flows, form submissions, auth flows, and navigation behavior
- test behavior and visible output, not Svelte internals
- avoid tests that assert whether a rune exists internally

Prefer this order of attack:
1. test extracted logic in plain `.ts` / `.js` modules when possible
2. test components in isolation when rendering and interaction matter
3. reach for E2E only when whole-app behavior matters

## Using runes in test files

Vitest processes test files like source files, but Svelte runes are only compiler syntax inside
`.svelte`, `.svelte.js`, and `.svelte.ts` files.

### File naming rule

Rename a test file to include `.svelte` only when the test file itself contains rune syntax such as
`$state`, `$derived`, `$effect`, or `$props` directly in the file body.

Examples:
- `counter.svelte.test.ts` — needed if the test file itself writes `let count = $state(0)`
- `component.svelte.spec.ts` — needed if the test file itself creates reactive test scaffolding with runes
- `component.test.ts` — fine if the test only imports a `.svelte` component or a `.svelte.ts` helper and does not write runes directly

Do **not** rename every test file to `.svelte.test.ts` by default. The suffix is required for rune
syntax in the test file itself, not merely because the thing being imported was written with Svelte.

### `$effect.root` in test files

`$effect.root(...)` is not a wrapper for “anything that uses runes.” It is only needed when the code
under test creates `$effect(...)` or `$effect.pre(...)` outside a normal component lifecycle.

`$state` and `$derived` are usable in `.svelte.ts` modules without this problem. The runtime error
`effect_orphan` is specifically about effect creation without a parent effect/root, not about rune
usage in general.

Use `$effect.root(...)` in tests for classes, factories, or helpers that register effects internally.
It creates an effect scope and returns a cleanup function that **must** be called to dispose the
scope.

```ts
// logger.svelte.test.ts
import { test, expect } from 'vitest';
import { Logger } from './logger.svelte.ts';

test('logger sync effect can run outside a component when rooted', () => {
	const stop = $effect.root(() => {
		const logger = new Logger();
		logger.write('hello');
		expect(logger.history).toContain('hello');
	});

	stop();
});
```

A realistic shape for the code under test:

```ts
// logger.svelte.ts
export class Logger {
	history = $state<string[]>([]);
	lastMessage = $state('');

	constructor() {
		$effect(() => {
			if (this.lastMessage) {
				this.history = [...this.history, this.lastMessage];
			}
		});
	}

	write(message: string) {
		this.lastMessage = message;
	}
}
```

If that class is instantiated outside component initialization and outside `$effect.root(...)`, the
internal `$effect(...)` has no parent effect scope and throws `effect_orphan`.

If the tested module only uses `$state`/`$derived` and never creates an effect, do **not** require
`$effect.root(...)` in the test. Avoid teaching a blanket rule that all rune-based helpers must be
wrapped.

## Component testing patterns

Prefer user-visible assertions:
- text content
- ARIA roles
- interaction results
- loading/error states
- callback effects visible in UI

Avoid brittle tests that depend on component internals or DOM shape that is likely to change.
Do not write tests that assert whether a specific rune exists internally — assert the resulting
behavior instead.

### Wrapper components

When testing components that use:
- two-way bindings
- context
- snippet props

it is usually better to create a small wrapper component for the test and interact with that
wrapper, rather than forcing low-level setup directly in the test body.

A wrapper is the clean pattern because `bind:`, context, and snippet props are component-tree
mechanisms, not plain values that can be fully constructed as loose objects from outside a Svelte
component.

```svelte
<!-- TestWrapper.svelte -->
<script lang="ts">
	import Child from './Child.svelte';

	let value = $state('start');
</script>

<Child bind:value>
	{#snippet children()}
		<span data-testid="content">wrapped content</span>
	{/snippet}
</Child>
```

Use this pattern when you need to verify:
- two-way binding updates
- context-dependent rendering
- optional or required snippet content

### Testing Library

`@testing-library/svelte` is a strong default helper for component tests because it pushes tests
toward user-observable behavior and handles Svelte-specific rendering/cleanup for you.

## Storybook

Use Storybook when you want both:
- component documentation / visual states
- interactive tests in a browser-like environment

Storybook can run interaction tests using Vitest browser mode.
Treat it as a useful companion for UI-heavy component libraries, not a replacement for all unit or
E2E coverage.

## End-to-end tests with Playwright

Use Playwright for:
- route-level behavior
- navigation flows
- auth flows
- progressive enhancement / forms
- submissions, redirects, and error handling
- SSR/CSR integration behavior that component tests cannot validate in isolation

When using Playwright, prefer user-journey tests over microscopic DOM assertions.

## What to test in async-first UI

When components use direct `await`, `<svelte:boundary>`, or async-first UI patterns, test:
- pending UI
- success UI
- failed UI
- retry / recovery behavior when applicable

Do not only test the success path.

## SvelteKit-specific testing reminders

For SvelteKit apps, prioritize tests around:
- form actions and enhanced forms
- `load`-driven route output
- redirects and error boundaries
- shallow routing and snapshot-backed UI state when used

For server-only logic, prefer testing the server helper or action/handler logic directly when
possible instead of over-relying on slow E2E coverage.

## Hard reminders

- `sv add vitest` sets up client/server-aware config for Svelte projects
- Only rename a test file to `.svelte.test.ts` / `.svelte.spec.ts` when that file itself contains rune syntax
- `$effect.root(...)` is for code that creates `$effect`/`$effect.pre` outside component initialization, not for all rune usage
- Always call the cleanup function returned by `$effect.root(...)`
- Playwright for E2E, Storybook when interactive component docs add value
- Test behavior, not rune existence
- Use wrapper components for bindings, context, and snippet-heavy cases
- Async UI tests must cover pending and failure states, not just success
