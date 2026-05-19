# Testing: Vitest, Component Tests, Storybook, and Playwright

Use this file when the user asks about testing strategy, test setup, component tests, Storybook testing, or end-to-end testing.

This file owns:
- test tool choice
- official-first testing defaults
- component testing patterns
- async UI testing expectations

It does not override framework semantics owned by other files.
For async behavior, boundaries, and await semantics, trust `async-svelte.md`.
For SvelteKit routing/form/server behavior, trust `sveltekit.md`.

## Official defaults

Svelte is unopinionated overall, but for modern Svelte/Vite/SvelteKit projects the practical official-first defaults are:
- **Vitest** for unit and component tests
- **Playwright** for end-to-end tests
- **Storybook** when you want component documentation plus interaction testing

CLI setup examples:

```bash
npx sv add vitest
npx sv add playwright
npx sv add storybook
```

## Testing doctrine for this skill

When the user asks for tests:
- default to TypeScript test files for new SvelteKit projects unless the task or codebase clearly calls for JavaScript
- default to Vitest for logic, stores, utilities, and isolated component tests
- default to Playwright for route flows, form submissions, auth flows, and navigation behavior
- test behavior and visible output, not Svelte internals
- avoid tests that assert whether a rune exists internally

## Unit and component tests with Vitest

If the project uses Vite or SvelteKit, Vitest is the default recommendation.

Prefer this order of attack:
1. test extracted logic in plain `.ts` / `.js` modules when possible
2. test components in isolation when rendering and interaction matter
3. reach for E2E only when whole-app behavior matters

### Using runes in test files

Vitest processes test files like source files.
If you want to use runes directly inside a test file, the filename should include `.svelte`, such as:
- `thing.svelte.test.ts`
- `component.svelte.spec.ts`

If the code under test uses effects, wrap the test in `$effect.root(...)` so cleanup works correctly.

## Component testing patterns

Prefer user-visible assertions:
- text content
- ARIA roles
- interaction results
- loading/error states
- callback effects visible in UI

Avoid brittle tests that depend on component internals or DOM shape that is likely to change.

### Wrapper components

When testing components that use:
- two-way bindings
- context
- snippet props

it is usually better to create a small wrapper component for the test and interact with that wrapper, rather than forcing low-level setup directly in the test body.

### Testing Library

`@testing-library/svelte` is a strong default helper for component tests because it pushes tests toward user-observable behavior.

## Storybook

Use Storybook when you want both:
- component documentation / visual states
- interactive tests in a browser-like environment

Storybook can run interaction tests using Vitest browser mode.
Treat it as a useful companion for UI-heavy component libraries, not a replacement for all unit or E2E coverage.

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

For server-only logic, prefer testing the server helper or action/handler logic directly when possible instead of overrelying on slow E2E coverage.

## Hard reminders

- Vitest first for unit/component coverage
- Playwright first for E2E coverage
- Storybook when interactive component docs are valuable
- test behavior, not rune existence
- use wrapper components for bindings, context, and snippet-heavy cases
- async UI tests must cover pending and failure states, not just success
