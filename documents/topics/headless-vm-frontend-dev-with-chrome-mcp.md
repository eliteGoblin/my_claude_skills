# Headless VM Frontend Dev with Chrome MCP

## The Problem

Claude Code runs on a headless Ubuntu VM (no display). Frontend work (React, etc.) needs visual verification. The host Mac has Chrome with the Chrome DevTools MCP server connected to Claude.

## Network Setup

```
Ubuntu VM (192.168.4.27)  <-->  Mac Host (192.168.4.25)

Mac resolves:   myubuntu  → 192.168.4.27
Ubuntu resolves: mymac    → 192.168.4.25
```

## How It Works

```
┌─────────────────┐          HTTP           ┌──────────────────┐
│  Ubuntu VM      │ ◄────────────────────── │  Mac Chrome      │
│                 │                          │                  │
│  Vite dev       │  serves index.html + JS  │  Downloads JS,   │
│  server :5173   │ ────────────────────────►│  renders React   │
│  (host: true)   │                          │  client-side     │
│                 │                          │                  │
│  Claude Code    │  Chrome MCP (snapshot,   │  MCP Server      │
│  (CLI)          │  screenshot, click, etc) │  (chrome-devtools)│
└─────────────────┘ ────────────────────────►└──────────────────┘
```

### Vite Config Requirement

Vite must bind to all interfaces so the Mac can reach it:

```typescript
// vite.config.ts
server: {
  host: true,  // binds 0.0.0.0 instead of localhost
}
```

### Why This Works (CSR vs SSR)

Vite + React is **client-side rendering (CSR)**:
1. Mac Chrome requests `http://myubuntu:5173`
2. Vite serves `index.html` (empty `<div id="root">`) + JS bundles over HTTP
3. Chrome downloads JS and **renders everything locally in the browser**
4. No server-side rendering — Vite is just a file server + HMR websocket

This is different from **Next.js SSR**, where the server pre-renders HTML before sending it. For CSR, the server only needs to be network-reachable — all rendering happens in the browser.

## Claude's UI Verification Workflow

1. Start dev server in background: `npm run dev` (Vite on `:5173`)
2. Open page on Mac Chrome via MCP:
   ```
   mcp__chrome-devtools__new_page({ url: "http://myubuntu:5173" })
   ```
3. Verify UI:
   - `take_screenshot` — see rendered page as image
   - `take_snapshot` — read DOM/a11y tree as text
   - `click` / `fill` — interact with elements
   - `navigate_page` — switch between routes
4. Iterate: fix code → Vite HMR auto-reloads → re-screenshot

## Constraints

- Chrome MCP operates on the **selected tab** — the Vite page must be the active tab
- `new_page` brings it to front by default, so this is handled automatically
- HMR websocket also needs network access (same port, handled by `host: true`)
