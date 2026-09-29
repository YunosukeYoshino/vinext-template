# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Core Commands

```bash
pnpm dev          # vite dev server
pnpm build        # production build → .cloudflare/output/
pnpm start        # preview the built Worker locally
pnpm deploy       # deploy to Cloudflare Workers (vinext-cloudflare deploy)
pnpm lint         # oxlint
pnpm format       # oxfmt (format and write)
pnpm format:check # oxfmt (check only)
```

## Project

Next.js App Router application running on **Cloudflare Workers via vinext 1.0** — a Vite-based reimplementation of Next.js that replaces the webpack/Node.js runtime with Vite and deploys directly to the edge.

## Architecture

```
src/app/              ← Next.js App Router (pages, layouts, CSS)
vite.config.ts        ← Vite config: vinext() + cloudflare() plugins
cloudflare.config.ts  ← Worker config via cf (ASSETS + IMAGES bindings)
next.config.ts        ← Next.js-style config (read by vinext at build time)
.cloudflare/types     ← Generated Worker types (gitignored)
.cloudflare/output    ← Build output (gitignored — worker bundle + client assets)
```

### How vinext works

- `next/*` imports are shimmed by vinext — the `next` package is NOT a dependency
- `next/image` is routed through `imagesOptimizer()` declared in `vite.config.ts`
- `next/font/google` loads from CDN at runtime, not self-hosted at build time
- `vinext()` auto-registers the RSC plugin — do not add `rsc()` manually
- Worker types land in `.cloudflare/types` during dev/build; regenerate standalone with `pnpm exec cf workers types`

### Cloudflare bindings

Declared in `cloudflare.config.ts` via `bindings.*` helpers:

| Binding      | Purpose                                        |
| ------------ | ---------------------------------------------- |
| `env.ASSETS` | Static asset delivery (built client assets)    |
| `env.IMAGES` | Image transformation via Cloudflare Images API |

## Git Workflow

- Main branch: `main`
- Conventional commits: `feat:`, `fix:`, `refactor:`, `docs:`, `chore:`

## Available Skills

| Skill               | Description                                                                            |
| ------------------- | -------------------------------------------------------------------------------------- |
| `migrate-to-vinext` | Migrates a Next.js project to vinext (package swap, config generation, ESM conversion) |
