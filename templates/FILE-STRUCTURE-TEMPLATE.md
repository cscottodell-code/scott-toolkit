# File Structure Reference

## Metadata
- Last updated: 2026-02-25
- Version: 1.0
- Changelog:
  - v1.0: Initial template with three project variants

<!--
WHAT THIS IS: A reference document showing the standard file/folder organization
for projects in Scott's stack. Three variants: web-only, desktop-only (Tauri), and
both (shared codebase, two targets).

WHY IT MATTERS: Consistent project structure means Claude Code always knows where
to find things and where to put new files. No guessing, no inconsistency between
projects.

HOW TO USE: When starting a new project, pick the variant that matches your deployment
target. Use it as the starting point for your project repo.
-->

---

## Web-Only Project

For apps that run in the browser. SurrealDB runs as a separate server.
Nuxt handles routing, SSR (optional), and data fetching.

```
project-name/
├── CLAUDE.md               # Claude Code's instruction manual for this project
├── PRD.md                  # Product Requirements Document
├── nuxt.config.ts          # Nuxt configuration
├── package.json            # npm dependencies
├── tsconfig.json           # TypeScript configuration
│
├── app/                    # All Nuxt application code lives here
│   ├── app.vue             # Root Vue component (wraps everything)
│   ├── pages/              # Route-based pages (each file = a URL)
│   │   │                   #   index.vue → /
│   │   │                   #   claims/index.vue → /claims
│   │   │                   #   claims/[id].vue → /claims/abc123
│   │   └── ...
│   ├── components/         # Reusable Vue components
│   │   │                   #   Used across multiple pages
│   │   │                   #   Import automatically (no import statement needed)
│   │   └── ...
│   ├── composables/        # Shared logic (reusable functions)
│   │   │                   #   Think of these like utility functions that can
│   │   │                   #   use Vue features (reactive data, watchers, etc.)
│   │   │                   #   The most important one is useDatabase - it handles
│   │   │                   #   all communication with SurrealDB.
│   │   └── useDatabase.ts  # Database connection and query helper
│   ├── layouts/            # Page layouts (header, sidebar, footer arrangements)
│   │   │                   #   default.vue is used unless a page specifies another
│   │   └── ...
│   └── assets/             # Static assets (images, fonts, CSS files)
│       └── ...
│
├── server/                 # Nuxt server-side code
│   └── api/                # API endpoints (server routes)
│       │                   #   Each file is an API endpoint:
│       │                   #   claims.get.ts → GET /api/claims
│       │                   #   claims.post.ts → POST /api/claims
│       └── ...
│
├── database/               # SurrealDB schema and seed files
│   ├── schema.surql        # Table definitions, indexes, permissions
│   │                       #   Run this to set up a fresh database
│   └── seed.surql          # Development seed data
│       │                   #   Sample records for testing
│
├── tasks/                  # Project management files (maintained by Claude Code)
│   ├── todo.md             # Current milestone task checklist
│   └── lessons.md          # Mistakes and corrections log
│                           #   Claude Code reviews this at the start of each session
│
└── public/                 # Public static files (served as-is, no processing)
    └── ...                 #   favicon.ico, robots.txt, etc.
```

### Key Points for Web Projects:
- `useDatabase.ts` connects to SurrealDB server via WebSocket or HTTP
- `server/api/` routes can also talk to SurrealDB (useful for server-side operations)
- SSR is available (Nuxt renders pages on the server for faster initial load)

---

## Desktop-Only Project (Tauri)

For apps that run as a native desktop window. SurrealDB runs embedded inside
the app (no separate server needed). Nuxt is compiled as static HTML/JS (SSG mode).

```
project-name/
├── CLAUDE.md
├── PRD.md
├── nuxt.config.ts          # MUST have ssr: false (Tauri can't do server rendering)
├── package.json
├── tsconfig.json
│
├── app/
│   ├── app.vue
│   ├── pages/
│   ├── components/
│   ├── composables/
│   │   └── useDatabase.ts  # Calls Tauri commands for all data operations
│   │                       #   Instead of connecting to a server, this calls
│   │                       #   Rust functions via Tauri's IPC bridge
│   ├── layouts/
│   └── assets/
│
├── database/
│   ├── schema.surql        # Same schema format as web projects
│   └── seed.surql          # Loaded by Rust backend on first run
│
├── tasks/
│   ├── todo.md
│   └── lessons.md
│
├── src-tauri/              # Rust backend (this is the native app part)
│   ├── Cargo.toml          # Rust dependencies (includes surrealdb crate)
│   ├── tauri.conf.json     # Tauri configuration (window size, title, permissions)
│   ├── src/
│   │   ├── main.rs         # App entry point + SurrealDB embedded initialization
│   │   │                   #   Sets up the database, opens the window
│   │   ├── lib.rs          # Tauri command registrations
│   │   │                   #   Lists all commands the frontend can call
│   │   ├── commands/       # Tauri command handlers (one file per domain)
│   │   │   │               #   Each file defines Rust functions that the
│   │   │   │               #   frontend can call via @tauri-apps/api
│   │   │   └── ...         #   e.g., claims.rs, photos.rs
│   │   └── db/             # SurrealDB connection and query helpers
│   │       │               #   Manages the embedded database connection
│   │       │               #   Shared across all command handlers
│   │       └── ...
│   └── capabilities/       # Tauri permission definitions
│       │                   #   Controls what the app is allowed to do
│       │                   #   (file access, network, notifications, etc.)
│       └── ...
│
└── public/
```

### Key Points for Desktop Projects:
- **ssr: false is mandatory** — Tauri wraps static HTML, not a Node server
- **No `/server/` directory** — there's no Nuxt server in Tauri apps
- `useDatabase.ts` calls Tauri commands (Rust functions) instead of HTTP/WebSocket
- **Build process:** `nuxi build` (generates static files) → `cargo tauri build` (wraps in native app)
- SurrealDB data is stored locally on the user's machine (offline-capable)

---

## Both (Shared Codebase, Two Targets)

For apps that work as both desktop AND web. Uses an adapter pattern so the
same components work with either embedded or server SurrealDB.

```
project-name/
├── CLAUDE.md
├── PRD.md
├── nuxt.config.ts          # Conditional config based on build target
├── package.json
├── tsconfig.json
│
├── app/
│   ├── app.vue
│   ├── pages/
│   ├── components/
│   ├── composables/
│   │   ├── useDatabase.ts      # Abstract interface — components call this
│   │   │                       #   This is the KEY FILE. It detects whether
│   │   │                       #   the app is running in Tauri or a browser,
│   │   │                       #   then delegates to the right adapter.
│   │   └── adapters/
│   │       ├── tauri-adapter.ts # Desktop: calls Tauri commands → embedded SurrealDB
│   │       └── web-adapter.ts   # Web: calls SurrealDB server via WebSocket
│   ├── layouts/
│   └── assets/
│
├── server/                 # Only used in web mode (ignored by Tauri build)
│   └── api/
│
├── database/
│   ├── schema.surql
│   └── seed.surql
│
├── tasks/
│   ├── todo.md
│   └── lessons.md
│
├── src-tauri/              # Only exists for desktop builds
│   ├── Cargo.toml
│   ├── tauri.conf.json
│   ├── src/
│   │   ├── main.rs
│   │   ├── lib.rs
│   │   ├── commands/
│   │   └── db/
│   └── capabilities/
│
└── public/
```

### Key Points for Dual-Target Projects:
- **The adapter pattern is critical** — `useDatabase.ts` is the abstraction layer
  that makes components work on both desktop and web without changes
- Components NEVER talk to SurrealDB directly. Always through `useDatabase.ts`
- `server/` directory is only used in web mode
- `src-tauri/` directory is only used in desktop mode
- Same schema.surql works for both modes (SurrealQL is SurrealQL)

### How the Adapter Pattern Works:
```
Component calls useDatabase().getClaims()
  ↓
useDatabase detects environment:
  ├── In Tauri? → tauri-adapter.ts → invoke('get_claims') → Rust → embedded SurrealDB
  └── In browser? → web-adapter.ts → WebSocket → SurrealDB server
  ↓
Same data comes back to the component either way
```

---

## Directory Explanations

| Directory | What Goes Here | Think of It Like... |
|-----------|---------------|-------------------|
| `pages/` | One file per URL/route in your app | Tabs in a spreadsheet — each one is a different view |
| `components/` | Reusable UI pieces used across pages | Reusable cell formatting templates |
| `composables/` | Shared logic and data-fetching functions | Shared formulas used across sheets |
| `layouts/` | Page frame (header, sidebar, footer) | The print layout template |
| `assets/` | Images, fonts, CSS files | The media folder |
| `server/api/` | Backend endpoints (web only) | The "backend" that processes requests |
| `database/` | SurrealDB schema and seed data | The database design document |
| `src-tauri/` | Rust backend code (desktop only) | The native app wrapper |
| `tasks/` | Project management files | Your project tracking spreadsheet |
| `public/` | Static files served as-is | Files dropped in a public folder |
