# OMEGATAB — Conceptual Distillation

## The Product: What It Is

OmegaTab replaces the browser's empty new-tab page with a **personalized launch hub**. Every time you open a new tab, instead of a blank void, you land on your own curated dashboard of links, search, and tools. It transforms an otherwise wasted moment into a productivity entry point.

---

## Core Ideas (The "Soul")

### 1. The Universal Omni-Box

The search bar is not a search bar. It is a **single unified input surface** that handles six different actions without the user needing to switch modes:

- **Fuzzy search through saved links** (title, URL, description) — surfaced first, because they're your stuff
- **Web search** via your currently selected engine (Google, Brave, Bing, Perplexity)
- **Search history** with smart scoring (recency × frequency × match quality)
- **Auto-suggestions** from an external API (Brave), deduplicated against history
- **Direct URL navigation** — type an address, go there
- **Multi-line input** — Shift+Enter for compound queries

The omni-box automatically expands as you type (textarea, not input), and the dropdown shows the most relevant result type at any moment. If a link matches, show that. If not, show suggestions. If the user typed a URL, just navigate. **The user doesn't switch contexts — the system adapts.**

### 2. Categorized Columns as Mental Model

Links are not a flat list. They are organized into **user-named columns** (categories). This creates a spatial, visual organization. Example columns: "Dev Tools," "Social," "Finance," "Work."

Each column has:
- A bold title with `//` comment-style prefix (terminal aesthetic)
- Links within it, ordered by user preference
- Its own keyboard modifier for quick-launch (`Ctrl` for column 1, `Ctrl+Alt` for column 2)

This maps to **muscle memory**. Over time, opening "GitHub" becomes `Ctrl+1` without thinking. The spatial layout and keyboard bindings reinforce each other.

### 3. Keyboard-First, Progressive Power

The entire application is navigable without a mouse, but the shortcuts are **progressively disclosed**:

- **Visible when relevant**: Shortcut hints appear on link cards only on hover
- **Discoverable on demand**: `Ctrl+/` opens the full keyboard shortcut reference modal
- **Always available**: `Ctrl+K` opens the command palette, a searchable interface to every link, action, and setting

The philosophy: shortcuts empower power users without intimidating newcomers. New users click. Experienced users fly.

**Shortcut summary:**

| Action | Shortcut |
|---|---|
| Open link in column 1, position N | `Ctrl+N` |
| Open link in column 2, position N | `Ctrl+Alt+N` |
| Command palette | `Ctrl+K` |
| Add new link | `Alt+N` |
| Cycle search engine | `Ctrl+Up/Down` |
| Show shortcuts reference | `Ctrl+/` |
| Arrow-key navigate between links | `Up/Down/Left/Right` |
| Edit focused link | `E` |
| Delete focused link | `D` |

### 4. Command Palette as Universal Navigator

Borrowed directly from code editors (VS Code, Sublime). The command palette (`Ctrl+K`) is a modal search interface that finds and executes **anything**:

- Open any saved link by typing its name
- Switch search engines
- Navigate to Settings
- Trigger actions like "Add New Link"

It's a fallback for everything. If a user forgets a shortcut or can't find something, the command palette finds it. **Low ceiling, high floor.**

### 5. Zero-Typing Link Creation (Metadata Fetching)

When adding a link, the user only needs to paste a URL. The backend:

1. Fetches the page
2. Parses `<title>`, `<meta name="description">`, OpenGraph tags
3. Extracts the favicon (from link tags, or tries common paths like `/favicon.ico`, falls back through multiple URLs)
4. Converts the favicon to a base64 data URI
5. Returns title, description, and icon — auto-populating the link card

The user can override any field, but the default behavior is **"just paste a URL and it works."** This removes the friction of bookmark creation. Links feel like they materialize, not like they're manually constructed.

### 6. Local-First with Background Sync

Data is cached aggressively in localStorage (versioned with `omegatab_v1_` prefix). On page load:

1. Render from cache immediately (instant)
2. Fetch from server in parallel (background refresh)
3. Merge server data if newer

This means the page loads **instantly** — critical for a new tab page that the user might see dozens of times per day. The cached render happens before the first network request completes.

### 7. Optimistic UI with Rollback

Mutations are applied to the UI immediately, then persisted to the server:

- **Delete a link**: Removed from the DOM instantly. If the server call fails, the link is restored from a snapshot.
- **Update a link**: Updated in state immediately. If the server call fails, revert.
- **Reorder links**: Drag-and-drop updates `order_index` locally, then pushes to the server.

This makes every interaction feel **instant and solid**, not network-dependent.

### 8. Search History with Smart Scoring

History items are scored on three axes:

- **Match quality** (0-1): exact match > prefix match > substring > fuzzy
- **Frequency** (normalized): how often the user has searched this term
- **Recency** (0-1): decay over 30 days, recent searches score higher

Final score: `match × 0.5 + freq × 0.3 + recency × 0.2`

History is **merged with API suggestions** and deduplicated (history wins ties), so the dropdown feels personalized rather than generic.

### 9. Multi-Engine Search Cycling

The user isn't locked into one search engine. `Ctrl+Up/Down` cycles through a configurable list (Brave, Perplexity, Google, Bing). The visual indicator (icon + placeholder text) changes to reflect the active engine. The selection persists across sessions via cache.

This acknowledges that different searches call for different engines — and makes switching frictionless.

### 10. Tool Integrations as First-Class Features

The settings panel includes toggles for Jira, Confluence, and Linear API integrations. The search bar has special handling: typing a pattern like `PROJ-123` can be detected as a Jira issue ticket and navigated directly. The product vision is a **work hub** — not just bookmarks, but deep links into the tools you actually use at work.

(These integrations were scoped as premium/paid features.)

### 11. Desktop-Native Experience in a Web App

OmegaTab fights the browser's natural behavior to feel like a desktop application:

- **Aggressive focus capture**: On page load, the search input is focused with a cascade of `requestAnimationFrame` and `setTimeout` calls at multiple delays to beat the browser's URL bar
- **Visibility detection**: When the tab becomes visible again (user switches to it), the search bar is re-focused
- **System tray integration**: The Rust backend creates a native system tray icon (Windows/macOS/Linux) for quit/manage actions
- **Single binary**: The Rust server compiles to one executable that serves both API and SPA assets

### 12. Draggable Link Ordering

Links within a column can be reordered via drag-and-drop. Order matters because keyboard shortcuts (`Ctrl+1`, `Ctrl+2`, etc.) map to ordinal positions. Reordering is persisted to the server as `order_index` updates on each affected link.

### 13. Full Product Surface

OmegaTab was conceived as a **real product**, not a side project:

- **Landing page** with hero, feature sections, screenshots, CTAs
- **VitePress documentation** with 11 guides covering every feature
- **Authentication** (email/password registration and login with JWT)
- **Pricing tiers** (Free, Plus, Pro, Team) with Stripe integration
- **Feature gating**: Boolean toggles per feature per plan (analytics, team features, custom domains)
- **Feedback system**: In-app feedback form with rate limiting (once per 24 hours) and email notification to support
- **Privacy Policy and ToS**: Comprehensive legal pages
- **Error monitoring**: Sentry integration with user context tracking
- **Staging environment**: Password-gated staging login with separate deployment

### 14. The Icon Ecosystem

Link icons are a priority, not an afterthought:

- **Auto-fetch**: Favicon is scraped during metadata fetch, tried from multiple common paths, converted to base64
- **Dashboard Icons integration**: Built-in search panel that queries dashboardicons.com for recognized app icons (Slack, GitHub, Docker, etc.)
- **Upload**: Users can upload custom images (max 500KB, converted to data URI)
- **URL input**: Users can paste an icon URL directly
- **Preview**: Live icon preview with image analysis that detects dark icons and adds a background

### 15. Cache Strategy as Architecture

All persistent state uses a versioned, namespaced localStorage cache. Search history has its own bespoke storage format (frequency + timestamp per entry). This separation of concerns means the cache layer can evolve independently of the data format.

### 16. Design Ethos: "Minimal Terminal"

The aesthetic philosophy: **"What if a beautifully typeset newspaper met a 1980s terminal?"**

- **Monochromatic foundation**: Nearly all UI is grayscale (cream whites, soft grays, deep blacks)
- **Single bold accent**: Vibrant mint green (`#06d6a0`) used sparingly — like a terminal cursor or LED
- **Retro-tech details**: Cursor blink on the logo, LED pulse animations, ticker transitions
- **High contrast, low noise**: Strong text/background contrast, minimal decoration, generous whitespace
- **Warm undertones**: Off-white background (`#f8f7f4`) with cream/paper warmth

The column titles use `//` comment-style prefixes. The logo has a blinking cursor. The overall feel is **terminal-inspired but warm and readable**, not cold or clinical.

---

## What Made It Work (The Deeper Patterns)

### Pattern A: "Don't make me choose a mode"

The search bar is the clearest example. The user doesn't toggle between "search my links" and "search the web" modes. They just type, and the system figures out the most relevant thing to show. Mode-free interaction.

### Pattern B: "Show me what I need, when I need it"

Keyboard shortcut hints appear on hover. The command palette is hidden behind `Ctrl+K`. The help menu is a small floating button. Complexity is available on demand, not forced on first encounter.

### Pattern C: "Feel instant, sync later"

Every data operation follows the pattern: render from cache, mutate optimistically, persist in the background, reconcile on conflict. The user never waits for the network.

### Pattern D: "One input, many outputs"

The omni-box pattern is repeated. The command palette is the same idea — one text input that searches across links, commands, and settings simultaneously. Type once, find anything.

### Pattern E: "Own your data, own your tool"

Search history is local-first. Settings are toggled per-user. The server is self-hostable (single binary). The user controls which features are enabled. Privacy-forward design where local storage is the source of truth and the server is persistence, not surveillance.

---

## Conceptual Feature Map

```
OmegaTab
├── Omni-Box (SearchBar)
│   ├── Fuzzy link search (Fuse.js)
│   ├── Web search (multi-engine, cyclable)
│   ├── Search history (smart-scored, local)
│   ├── Auto-suggestions (Brave API, premium)
│   └── Direct URL navigation
├── Link Columns (LinkColumns)
│   ├── User-named categories
│   ├── Keyboard shortcuts (Ctrl+N, Ctrl+Alt+N)
│   ├── Arrow-key navigation (2D grid)
│   ├── Drag-and-drop reorder
│   └── Optimistic CRUD with rollback
├── Command Palette (CommandPalette)
│   ├── Universal search (links + commands + settings)
│   ├── Ctrl+K activation
│   └── Keyboard-driven selection
├── Link Cards (LinkCard)
│   ├── Custom icons (auto-fetch, upload, URL, icon search)
│   ├── Context menu (Edit, Copy URL, Delete)
│   └── Progressive shortcut disclosure (hover)
├── Settings
│   ├── Feature toggles (search history, autosuggest, metadata, integrations)
│   ├── New-tab behavior toggle
│   └── Cache clearing
├── Auth
│   ├── Email/password registration + login
│   ├── JWT tokens
│   └── Staging password gate
├── Monetization
│   ├── Tiered plans (Free/Plus/Pro/Team)
│   ├── Feature gating per plan
│   └── Stripe payment integration
├── Feedback System
│   ├── Cancellation reason collection
│   ├── Rate-limited email submission
│   └── Resend email delivery
├── Documentation
│   ├── VitePress site with 11 guides
│   ├── Feature-specific walkthroughs
│   └── Keyboard shortcut reference
├── Landing Page
│   ├── Hero with screenshots
│   ├── Feature showcase sections
│   └── Legal pages (Privacy, ToS)
├── Backend (Rust)
│   ├── Self-contained binary (API + static assets)
│   ├── SQLite (local) / PostgreSQL (cloud)
│   ├── System tray integration
│   └── Sentry error monitoring
└── Desktop Experience
    ├── Aggressive focus capture on load
    ├── Tab visibility re-focus
    └── System tray quit/manage
```

---

## The Elevator Pitch

**OmegaTab is a browser new-tab replacement that turns every new tab into a personalized launchpad. A universal omni-box handles search, history, suggestions, and link navigation without mode-switching. Links are organized into keyboard-shortcut-mapped columns for spatial muscle memory. A command palette gives power users instant access to everything. The design is "minimal terminal" — warm paper tones with a single mint accent, evoking a typeset newspaper meeting a 1980s terminal. It's local-first, self-hostable, and feels like a desktop app in a browser tab.**

---

*This document captures the ideas and design philosophy of OmegaTab. It intentionally omits implementation details, code patterns, and technical debt. The goal is to preserve the conceptual DNA for a ground-up rebuild.*
