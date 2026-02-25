# Tech Stack Overview

## Metadata
- Last updated: 2026-02-25
- Version: 1.0

## The Stack (February 2026)

| Layer | Technology | Version | Role |
|-------|-----------|---------|------|
| Frontend framework | Nuxt | 4.x | Application framework (routing, data fetching, build) |
| UI components | Nuxt UI | v4.3.0 | 125+ pre-built components (installed as a Nuxt module) |
| Styling | Tailwind CSS | v4.0 | Utility-first CSS (CSS-first config, no tailwind.config.js) |
| Frontend language | TypeScript | latest | Type safety across all frontend code |
| State management | Pinia | latest | Reactive state management for Vue/Nuxt |
| Desktop shell | Tauri | 2.x | Wraps Nuxt app as native desktop app (Windows, Mac, Linux, mobile) |
| Backend runtime | Rust (via Tauri) | latest stable | Native backend logic, system access, IPC bridge |
| Database | SurrealDB | v3.0.0 | Multi-model DB (document, graph, vector, time-series, KV, geospatial) |
| Query language | SurrealQL | - | SurrealDB's query language for all data operations |
| Automation | n8n | latest | Workflow automation, used ONLY when truly the best option |

## How the Pieces Connect

### Nuxt 4 + Nuxt UI v4
Nuxt UI is a Nuxt module. Install it, and its 125+ components are auto-imported
throughout the app. Nuxt handles routing, data, and build. Nuxt UI handles what
things look like. They share Tailwind CSS v4 as their styling foundation.

### Tauri 2 + Nuxt 4
Tauri wraps the Nuxt frontend (compiled in SSG mode, ssr: false) in a native desktop
window. The Rust backend communicates with the Nuxt frontend via IPC (Inter-Process
Communication) called "Tauri commands." Nuxt's server-side features (SSR, /server/api/)
are NOT available inside Tauri.

### SurrealDB v3 Deployment Modes

**Embedded (desktop apps):**
SurrealDB runs inside Tauri's Rust backend. No separate server. Local-first, offline-capable.
Data stored on the user's machine.

**Server (web apps):**
SurrealDB runs as a standalone server. Nuxt connects via WebSocket or HTTP.

### The Adapter Pattern (for apps that target both)
```
Component calls useDatabase().getClaims()
  ↓
useDatabase detects environment:
  ├── In Tauri? → tauri-adapter.ts → invoke('get_claims') → Rust → embedded SurrealDB
  └── In browser? → web-adapter.ts → WebSocket → SurrealDB server
  ↓
Same data comes back to the component either way
```

### n8n Integration
n8n is NOT part of every project. It's used only for multi-system orchestration,
scheduled tasks, or webhook receivers from external services. When needed:
- Tauri apps call n8n webhooks via Rust HTTP client
- Web apps call n8n webhooks via Nuxt server routes

## Version Pinning

**Stay on these stable versions:**
- SurrealDB v3.0.0 (GA as of Feb 17, 2026)
- Nuxt 4.x
- Nuxt UI v4.3.0
- Tailwind CSS v4.0
- Tauri 2.x

**Do NOT upgrade to:**
- SurrealDB v3.1+ (wait for stability)
- Nuxt 5 (not released)
- Tauri 3 (not released)

## Key Architecture Decisions

| Decision | Reason |
|----------|--------|
| SurrealDB over PostgreSQL | Multi-model (document + graph), SurrealQL, embedded mode for Tauri |
| Nuxt over plain Vue | File-based routing, auto-imports, SSR/SSG flexibility, server routes |
| Nuxt UI over custom components | 125+ production components, consistent design, Tailwind integration |
| Tauri over Electron | 10x smaller bundles, Rust backend, better performance, embedded SurrealDB |
| Pinia over Vuex | Official Vue 3 state management, simpler API, TypeScript-first |
| TypeScript over JavaScript | Catch errors at build time, better IDE support, self-documenting |
| Tailwind over custom CSS | Utility-first, consistent spacing/colors, works with Nuxt UI |
| n8n only when needed | Avoid over-engineering; most app logic belongs in the app, not in a workflow tool |
