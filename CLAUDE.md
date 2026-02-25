# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Core Commands

```bash
pnpm dev          # vinext dev server (http://localhost:3000)
pnpm build        # production build → dist/
pnpm deploy       # build + deploy to Cloudflare Workers
pnpm lint         # oxlint via vinext
pnpm format       # oxfmt (format and write)
pnpm format:check # oxfmt (check only)
pnpm cf-typegen   # regenerate cloudflare-env.d.ts from wrangler bindings
```

## Project

Next.js App Router application running on **Cloudflare Workers via vinext** — a Vite-based reimplementation of Next.js that replaces the webpack/Node.js runtime with Vite and deploys directly to the edge.

## Architecture

```
src/app/          ← Next.js App Router (pages, layouts, CSS)
worker/index.ts   ← Cloudflare Worker entry point (image optimization + SVG bypass)
vite.config.ts    ← Vite config: vinext({ rsc: false }) + rsc() + cloudflare()
wrangler.jsonc    ← Wrangler deploy config (ASSETS + IMAGES bindings)
next.config.ts    ← Next.js config (read by vinext at build time)
dist/             ← Build output (client/ and server/ subdirs)
```

### How vinext works

- `next/*` imports are shimmed automatically — no application code changes needed
- `next/image` is replaced by `@unpic/react`; local images are routed through `/_vinext/image?url=...`
- `next/font/google` loads from CDN at runtime, not self-hosted at build time
- `next` package must remain in `dependencies` as vinext uses it for shims and types

### vite.config.ts — critical pattern

Use `vinext({ rsc: false })` to disable auto-registration, then register `rsc()` explicitly with entry points. Using plain `vinext()` alongside a manual `rsc()` causes a duplicate plugin error.

### worker/index.ts — SVG handling

Cloudflare Images API cannot process SVG files. The worker bypasses image optimization for `.svg` URLs and fetches them directly from `env.ASSETS`. See `worker/index.ts` for the implementation.

### Cloudflare bindings

| Binding      | Purpose                                        |
| ------------ | ---------------------------------------------- |
| `env.ASSETS` | Static asset delivery (`dist/client/`)         |
| `env.IMAGES` | Image transformation via Cloudflare Images API |

Add local binding overrides in `.dev.vars`.

## Git Workflow

- Main branch: `main`
- Conventional commits: `feat:`, `fix:`, `refactor:`, `docs:`, `chore:`

## Available Skills

| Skill               | Description                                                                            |
| ------------------- | -------------------------------------------------------------------------------------- |
| `migrate-to-vinext` | Migrates a Next.js project to vinext (package swap, config generation, ESM conversion) |
