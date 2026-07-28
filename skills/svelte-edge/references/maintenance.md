# Maintaining Svelte Edge

Read this file only when refreshing or auditing the skill itself.

## Source priority

Use sources in this order:

1. Current official Svelte, SvelteKit, and CLI docs, including https://svelte.dev/llms-full.txt
2. Official package changelogs and linked Svelte organization pull requests
3. Monthly What's new in Svelte posts as discovery indexes
4. Package registries, canonical repositories, and vendor documentation
5. Community catalogs only for discovery, never framework semantics

If sources disagree, current official docs own framework behavior. Package metadata owns only the facts it exposes.

## Evidence discipline

Keep three layers separate:

1. **Observation** — HTTP status, redirect, dist-tag, peer range, license, release, or commit date.
2. **Interpretation** — compatibility needs review, integration is sensitive, or evidence is incomplete.
3. **Curation** — keep in the shortlist, place on the watchlist, or leave out.

Never turn one observation into a stronger claim without corroboration:

- A 404/410 applies only to the exact URL checked.
- DNS, TLS, timeout, 401/403, 429, and 5xx results are indeterminate.
- Missing peer metadata does not prove incompatibility.
- An old release or commit does not prove a package is unusable.
- Absence from `libraries.md` is a scope decision, not a defect claim.

Use temporary working notes while researching. Do not bundle crawls, registry dumps, generated audit snapshots, or historical package catalogs into the skill. Git history preserves prior curated versions.

## Monthly refresh

1. Record the date and current versions of `svelte`, `@sveltejs/kit`, `sv`, `svelte-check`, `svelte-language-server`, and `svelte2tsx`.
2. Read every monthly blog since the previous baseline, separating framework/tooling changes from Community Showcase discovery.
3. Diff official changelogs through the current releases. Search for `breaking`, `deprecated`, `legacy`, `experimental`, `security`, and `removed`.
4. Verify every framework feature in current docs. Capture its minimum version, required flag, stability, and replacement.
5. Update the canonical reference first, then compact version gates in `SKILL.md`.
6. Count every entry in each monthly Libraries, Tools & Components section before curation so none is skipped accidentally.
7. For packages relevant to the shortlist, inspect the canonical docs, repository, registry metadata, license, Svelte 5/current Kit support, and migration notes.
8. Update `references/libraries.md` with only the current shortlist and watchlist. Treat monthly posts as discovery, never endorsement.
9. Search the whole skill for superseded names, stale links, and contradictory version gates.
10. Run skill validation and fresh-agent forward tests on representative new-code, legacy-edit, ecosystem, and edge-feature prompts.

## Current-versus-history rule

- Canonical references describe the current target.
- Migration references may retain short version timelines.
- Never leave a removed transitional API in current guidance.
- Do not label a stable current API deprecated merely because a future major plans to replace it.
- Use Git history when investigating why a package or recommendation changed.

## Release checklist

- Frontmatter still triggers on Svelte and SvelteKit work
- `SKILL.md` remains concise and every reference is linked directly
- New-code examples use one syntax generation only
- Experimental features name every required flag
- Version gates agree across all files
- Security patch floors remain visible where affected APIs are taught
- Every listed package was rechecked live and factual watchlist signals are dated
- No generated audit data, registry dumps, or historical catalogs are bundled
- Skill validation and forward tests pass
