<div align="center">

<h1>svelte-edge</h1>

<p>
  <b>Models don't write bad Svelte. They write old Svelte.</b>
</p>

<p>
  <a href="https://github.com/fifidayone/svelte-edge/releases"><img src="https://img.shields.io/badge/version-v0.1.0-ff3e00?style=flat-square&labelColor=09090b&logo=github&logoColor=ffffff" alt="version"></a>
  <a href="https://svelte.dev"><img src="https://img.shields.io/badge/Svelte-5.56+-27272a?style=flat-square&labelColor=09090b&logo=svelte&logoColor=FF3E00" alt="Svelte 5.56+"></a>
  <a href="https://kit.svelte.dev"><img src="https://img.shields.io/badge/SvelteKit-2.70+-27272a?style=flat-square&labelColor=09090b&logo=svelte&logoColor=FF3E00" alt="SvelteKit 2.70+"></a>
  <a href="https://kit.svelte.dev"><img src="https://img.shields.io/badge/SvelteKit_3-RC-ff3e00?style=flat-square&labelColor=09090b&logo=svelte&logoColor=FF3E00" alt="SvelteKit 3 RC"></a>
  <a href="https://github.com/fifidayone/svelte-edge/blob/main/LICENSE"><img src="https://img.shields.io/badge/license-MIT-27272a?style=flat-square&labelColor=09090b" alt="license"></a>
</p>

</div>

---

**svelte-edge** is the correction layer between a capable model and a moving framework. It assumes the model already knows Svelte — it just pins every answer to verified version floors, keeps one syntax generation per component, and tracks everything that changed since the model's training data froze.

## What it enforces

- **One generation per component** — Svelte 4 syntax never leaks into Svelte 5 code, and legacy files stay coherent when edited
- **Verified version floors** — every gate checked against real releases and security advisories, never against training-data memory
- **Both SvelteKit lines** — stable SvelteKit 2 defaults, plus a complete SvelteKit 3 reference so next-line questions don't get improvised from SvelteKit 2 habits
- **Security patch floors** — XSS, origin-check, and dev-server bypass gates, each with the minimum safe version
- **Stable first** — experimental flags are opt-in and never the default recommendation

## It maintains itself

Monthly refresh and deep-audit protocols are embedded in the skill itself. Version gates move when upstream moves, major versions rotate preview → stable → legacy (and legacy retires when its era ends), dead APIs get purged, and the ecosystem shortlist is re-verified live against the registry.

It ships as a snapshot. It doesn't stay one.

## Install

```bash
npx skills add fifidayone/svelte-edge
```

Or clone into your agent skills directory:

```bash
git clone https://github.com/fifidayone/svelte-edge.git .agents/skills/svelte-edge
```

Works with any agent, IDE, or CLI that supports `SKILL.md`.

## Baseline

**Validated August 21, 2026** — Svelte `5.56.10` · SvelteKit `2.70.3` / `3.0.0-next.25` (RC) · `sv` `0.17.0` / `1.0.0-next.4` · `svelte-check` `4.7.6` · Vite `8.2.2` — refreshed monthly against official changelogs, release notes, and security advisories.

## License

MIT — see [LICENSE](LICENSE).
