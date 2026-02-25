# Vinext

Next.js App Router application deployed to **Cloudflare Workers** via [vinext](https://github.com/nicholasgasior/vinext) — a Vite-based reimplementation of Next.js that runs at the edge.

## Features

- Next.js 16 App Router with React 19
- Edge-first deployment on Cloudflare Workers
- Image optimization via Cloudflare Images API (SVG served directly from ASSETS)
- Tailwind CSS v4
- oxlint + oxfmt for linting and formatting

## Prerequisites

- [Node.js](https://nodejs.org) 20+
- [Wrangler CLI](https://developers.cloudflare.com/workers/wrangler/) — `pnpm install -g wrangler`
- A Cloudflare account with Workers and Images enabled

## Getting started

```bash
pnpm install
pnpm dev
```

Open [http://localhost:3000](http://localhost:3000) to view the app.

> [!NOTE]
> Local image optimization uses a mock binding. To test the full Cloudflare Images pipeline, use `wrangler dev` directly.

## Commands

```bash
pnpm dev          # Start dev server (http://localhost:3000)
pnpm build        # Production build → dist/
pnpm lint         # Run oxlint
pnpm format       # Format files with oxfmt
pnpm format:check # Check formatting without writing
pnpm deploy       # Build and deploy to Cloudflare Workers
pnpm cf-typegen   # Regenerate cloudflare-env.d.ts from wrangler bindings
```

## Project structure

```
src/app/          Next.js App Router (pages, layouts, styles)
worker/index.ts   Cloudflare Worker entry — image optimization + SVG bypass
public/           Static assets
vite.config.ts    Vite config: vinext + rsc + cloudflare plugins
wrangler.jsonc    Wrangler deploy config (ASSETS and IMAGES bindings)
next.config.ts    Next.js config (read by vinext at build time)
dist/             Build output (generated — client/ and server/)
```

## Deployment

```bash
pnpm deploy
```

This runs `vinext build` followed by `wrangler deploy`. The worker is published to Cloudflare Workers with the following bindings:

| Binding | Purpose |
|---------|---------|
| `ASSETS` | Static asset delivery (`dist/client/`) |
| `IMAGES` | Image transformation via Cloudflare Images API |

Add local binding overrides in `.dev.vars`.

## How vinext works

vinext replaces the Next.js webpack/Node.js runtime with Vite and compiles directly to Cloudflare Workers:

- `next/*` imports are shimmed automatically — no application code changes needed
- `next/image` is replaced by `@unpic/react`; local images are routed through `/_vinext/image?url=...`
- `next/font/google` loads fonts from CDN at runtime instead of self-hosting at build time
- `next` must remain in `dependencies` as vinext uses it for shims and types

> [!IMPORTANT]
> Use `vinext({ rsc: false })` in `vite.config.ts` to disable auto-registration, then register `rsc()` explicitly. Using plain `vinext()` alongside a manual `rsc()` causes a duplicate plugin error.
