# Vinext

Next.js App Router application deployed to **Cloudflare Workers** via [vinext](https://github.com/cloudflare/vinext) — a Vite-based reimplementation of Next.js that runs at the edge.

## Features

- Next.js-style App Router with React 19 (App Router code runs on Vite 8 + RSC)
- Edge-first deployment on Cloudflare Workers
- Image optimization via the built-in `imagesOptimizer()` adapter (Cloudflare Images binding)
- Tailwind CSS v4
- oxlint + oxfmt for linting and formatting

## Prerequisites

- [Node.js](https://nodejs.org) 20+
- A Cloudflare account with Workers and Images enabled

## Getting started

```bash
pnpm install
pnpm dev
```

Open [http://localhost:5173](http://localhost:5173) to view the app.

> [!NOTE]
> Worker types are generated into `.cloudflare/types` during dev/build. Run `pnpm exec cf workers types` to regenerate them standalone.

## Commands

```bash
pnpm dev          # Start the Vite dev server (workerd-backed)
pnpm build        # Production build → .cloudflare/output/
pnpm start        # Preview the built Worker locally
pnpm lint         # Run oxlint
pnpm format       # Format files with oxfmt
pnpm format:check # Check formatting without writing
pnpm deploy       # Deploy to Cloudflare Workers (vinext-cloudflare deploy)
```

## Project structure

```
src/app/              Next.js App Router (pages, layouts, styles)
public/               Static assets
vite.config.ts        Vite config: vinext() + cloudflare() plugins
cloudflare.config.ts  Worker config (cf): ASSETS + IMAGES bindings
next.config.ts        Next.js-style config (read by vinext at build time)
.cloudflare/types     Generated Worker types (gitignored)
.cloudflare/output    Build output (gitignored — worker bundle + client assets)
```

## Deployment

```bash
pnpm deploy
```

This builds the app and deploys it to Cloudflare Workers with `vinext-cloudflare deploy`, using the bindings declared in `cloudflare.config.ts`:

| Binding  | Purpose                                        |
| -------- | ---------------------------------------------- |
| `ASSETS` | Static asset delivery (built client assets)    |
| `IMAGES` | Image transformation via Cloudflare Images API |

## How vinext works

vinext replaces the Next.js webpack/Node.js runtime with Vite and compiles directly to Cloudflare Workers:

- `next/*` imports are shimmed by vinext — the `next` package itself is not installed
- `next/image` is routed through the configured `imagesOptimizer()` (Cloudflare Images)
- `next/font/google` loads fonts from CDN at runtime instead of self-hosting at build time
- `vinext()` registers the RSC plugin automatically — no manual `rsc()` setup needed
