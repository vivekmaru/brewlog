# BrewLog — Architecture Document

> A personal homebrew tracking web app for beer, mead, and cider.

---

## Table of Contents

1. [Overview](#overview)
2. [Tech Stack](#tech-stack)
3. [Database Schema](#database-schema)
4. [Route / Page Structure](#route--page-structure)
5. [Component Architecture](#component-architecture)
6. [API Layer](#api-layer)
7. [Authentication & Access Model](#authentication--access-model)
8. [QR Code Flow](#qr-code-flow)
9. [Offline Support & Local Caching](#offline-support--local-caching)
10. [Project File / Folder Structure](#project-file--folder-structure)
11. [Deployment Architecture](#deployment-architecture)
12. [Key npm Dependencies](#key-npm-dependencies)
13. [Architectural Decisions & Trade-offs](#architectural-decisions--trade-offs)

---

## Overview

BrewLog is a **single-user** web application for tracking homebrew batches of beer, mead, and cider. The owner creates brews, logs ingredients, records fermentation events over time, and generates QR codes to attach to fermenters for quick mobile access.

All measurements use the **metric system** — liters (L) for volume, grams (g) / kilograms (kg) for weight, and Celsius (°C) for temperature.

### Core Workflow

```mermaid
flowchart LR
    A[Create Brew] --> B[Add Ingredients]
    B --> C[Log Fermentation Events]
    C --> D[Generate QR Code]
    D --> E[Scan QR on Fermenter]
    E --> F[Quick-Add Log Entry]
    C --> G[Dashboard View]
```

---

## Tech Stack

| Layer | Technology | Rationale |
|-------|-----------|-----------|
| Frontend | React 19 SPA (Vite) | Fast builds, modern React features, static deployment |
| Language | TypeScript | Type safety across frontend and API |
| Routing | React Router v7 | Client-side routing for SPA |
| Server State | TanStack Query (React Query) | Caching, background refetching, optimistic updates |
| Local State | Zustand | Lightweight, minimal boilerplate |
| Styling | Tailwind CSS + shadcn/ui | Rapid UI development with accessible components |
| Backend API | Cloudflare Workers / Vercel Functions / Netlify Functions | Serverless, free-tier friendly |
| Database | PostgreSQL (self-hosted, always-on) | Owner-provided Postgres server via connection string |
| ORM | Drizzle ORM (with `postgres-js` adapter) | Type-safe, lightweight, serverless-compatible |
| Auth | API key (simple header-based) | Single-user app — no OAuth/JWT complexity needed |
| Offline | Service Worker + IndexedDB | Local caching and sync-later for spotty WiFi |
| QR Codes | `qrcode.react` | Client-side QR generation as React component |

---

## Database Schema

### Entity Relationship Diagram

```mermaid
erDiagram
    BREWS ||--o{ INGREDIENTS : has
    BREWS ||--o{ FERMENTATION_EVENTS : has

    BREWS {
        text id PK
        text user_id
        text name
        text type
        real batch_size_liters
        real target_og
        real target_fg
        text status
        text start_date
        text end_date
        text notes
        timestamptz created_at
        timestamptz updated_at
    }

    INGREDIENTS {
        text id PK
        text brew_id FK
        text user_id
        text category
        text name
        real quantity
        text unit
        text date_added
        text notes
        timestamptz created_at
    }

    FERMENTATION_EVENTS {
        text id PK
        text brew_id FK
        text user_id
        text event_type
        real gravity
        real temperature_celsius
        text notes
        text event_date
        timestamptz created_at
    }
```

### Table Definitions

#### `brews`

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | `text` | PK, nanoid | Unique brew identifier |
| `user_id` | `text` | NOT NULL, DEFAULT `'default'` | Owner identifier (single user, for data hygiene and future-proofing) |
| `name` | `text` | NOT NULL | Brew name |
| `type` | `text` | NOT NULL | `beer`, `mead`, or `cider` |
| `batch_size_liters` | `real` | NOT NULL | Batch size in liters (L) |
| `target_og` | `real` | | Target original gravity |
| `target_fg` | `real` | | Target final gravity |
| `status` | `text` | NOT NULL, DEFAULT `fermenting` | One of: `planning`, `fermenting`, `conditioning`, `bottled`, `done` |
| `start_date` | `text` | NOT NULL | ISO 8601 date |
| `end_date` | `text` | | ISO 8601 date, nullable |
| `notes` | `text` | | Free-form notes |
| `created_at` | `timestamptz` | NOT NULL, DEFAULT now() | ISO 8601 timestamp |
| `updated_at` | `timestamptz` | NOT NULL, DEFAULT now() | ISO 8601 timestamp |

#### `ingredients`

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | `text` | PK, nanoid | Unique ingredient identifier |
| `brew_id` | `text` | FK → brews.id, NOT NULL | Parent brew |
| `user_id` | `text` | NOT NULL, DEFAULT `'default'` | Owner identifier (for data hygiene) |
| `category` | `text` | NOT NULL | `grain`, `hop`, `honey`, `fruit`, `yeast`, `adjunct`, `other` |
| `name` | `text` | NOT NULL | Ingredient name |
| `quantity` | `real` | NOT NULL | Amount (metric) |
| `unit` | `text` | NOT NULL | `g`, `kg`, `ml`, `L` |
| `date_added` | `text` | | ISO 8601 date — when added to the brew |
| `notes` | `text` | | e.g. boil time, dry hop duration |
| `created_at` | `timestamptz` | NOT NULL, DEFAULT now() | ISO 8601 timestamp |

#### `fermentation_events`

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | `text` | PK, nanoid | Unique event identifier |
| `brew_id` | `text` | FK → brews.id, NOT NULL | Parent brew |
| `user_id` | `text` | NOT NULL, DEFAULT `'default'` | Owner identifier (for data hygiene) |
| `event_type` | `text` | NOT NULL | `gravity_reading`, `temperature`, `racking`, `addition`, `tasting`, `note`, `bottling`, `other` |
| `gravity` | `real` | | Specific gravity reading |
| `temperature_celsius` | `real` | | Temperature in °C |
| `notes` | `text` | | Free-form notes |
| `event_date` | `text` | NOT NULL | ISO 8601 datetime |
| `created_at` | `timestamptz` | NOT NULL, DEFAULT now() | ISO 8601 timestamp |

### Drizzle Schema Notes

- All IDs use `nanoid` — short, URL-safe, collision-resistant.
- All tables include a `user_id` column defaulting to `'default'` for data hygiene and future-proofing, even though this is a single-user app.
- Timestamps use PostgreSQL `timestamptz` for native timezone-aware date handling.
- Foreign keys enforced via Drizzle relations and PostgreSQL constraints.
- Indexes on `ingredients.brew_id` and `fermentation_events.brew_id` for query performance.
- Drizzle ORM is used with the `drizzle-orm/postgres-js` adapter for serverless compatibility.
- The database is the owner's self-hosted, always-on PostgreSQL server, connected via a standard `DATABASE_URL` connection string.
- All units are strictly metric: `g`, `kg`, `ml`, `L` for quantity; `°C` for temperature; `L` for volume.

---

## Route / Page Structure

Routes are defined using React Router v7 in the client-side SPA:

```
src/routes/
├── root.tsx                      # Root layout with nav, theme provider
├── dashboard.tsx                 # Dashboard — all brews overview
├── brews/
│   ├── new.tsx                   # Create new brew form
│   └── $brewId/
│       ├── layout.tsx            # Brew detail layout with tabs
│       ├── index.tsx             # Brew overview/summary
│       ├── ingredients.tsx       # Ingredients list + add form
│       ├── log.tsx               # Fermentation event timeline + add form
│       ├── qr.tsx                # QR code display + download
│       └── edit.tsx              # Edit brew details
├── quick-log/
│   └── $brewId.tsx               # Mobile-optimized quick-add entry (QR target)
└── not-found.tsx                 # 404 page
```

### Route Summary

| Route | Purpose | Access | Key Features |
|-------|---------|--------|-------------|
| `/` | Dashboard | Authenticated | Brew cards grouped by status, search/filter |
| `/brews/new` | Create brew | Authenticated | Form with type selector, batch params (L) |
| `/brews/:brewId` | Brew summary | Public (read-only) | Overview stats, latest readings, ABV calc |
| `/brews/:brewId/ingredients` | Ingredients | Public (read-only) | Table of ingredients |
| `/brews/:brewId/log` | Fermentation log | Public (read-only) | Timeline of events |
| `/brews/:brewId/qr` | QR code | Authenticated | Generate, display, download QR |
| `/brews/:brewId/edit` | Edit brew | Authenticated | Update brew metadata and status |
| `/quick-log/:brewId` | Quick-add entry | Authenticated | Mobile-first form reached via QR scan |

---

## Component Architecture

### Layout Components

| Component | File | Responsibility |
|-----------|------|---------------|
| `RootLayout` | `src/layouts/root-layout.tsx` | HTML shell, font loading, theme provider |
| `AppNav` | `src/components/app-nav.tsx` | Top navigation bar with logo and links |
| `BrewDetailLayout` | `src/layouts/brew-detail-layout.tsx` | Tab navigation between brew sub-pages |

### Dashboard Components

| Component | File | Responsibility |
|-----------|------|---------------|
| `BrewDashboard` | `src/components/brew-dashboard.tsx` | Fetches and displays all brews via TanStack Query |
| `BrewCard` | `src/components/brew-card.tsx` | Single brew summary card with status badge |
| `StatusFilter` | `src/components/status-filter.tsx` | Filter brews by status |
| `BrewTypeIcon` | `src/components/brew-type-icon.tsx` | Icon for beer/mead/cider |

### Brew Detail Components

| Component | File | Responsibility |
|-----------|------|---------------|
| `BrewSummary` | `src/components/brew-summary.tsx` | Overview stats: OG, FG, ABV, batch size (L), age |
| `GravityChart` | `src/components/gravity-chart.tsx` | Line chart of gravity readings over time |
| `BrewStatusBadge` | `src/components/brew-status-badge.tsx` | Colored badge for brew status |

### Form Components

| Component | File | Responsibility |
|-----------|------|---------------|
| `BrewForm` | `src/components/forms/brew-form.tsx` | Create/edit brew form |
| `IngredientForm` | `src/components/forms/ingredient-form.tsx` | Add/edit ingredient (metric units: g, kg, ml, L) |
| `EventForm` | `src/components/forms/event-form.tsx` | Add fermentation event (°C) |
| `QuickLogForm` | `src/components/forms/quick-log-form.tsx` | Simplified mobile event form |

### Ingredient Components

| Component | File | Responsibility |
|-----------|------|---------------|
| `IngredientTable` | `src/components/ingredient-table.tsx` | Table of all ingredients for a brew |
| `IngredientRow` | `src/components/ingredient-row.tsx` | Single ingredient with edit/delete |

### Fermentation Log Components

| Component | File | Responsibility |
|-----------|------|---------------|
| `EventTimeline` | `src/components/event-timeline.tsx` | Chronological list of events |
| `EventCard` | `src/components/event-card.tsx` | Single event display |
| `EventTypeIcon` | `src/components/event-type-icon.tsx` | Icon per event type |

### QR Components

| Component | File | Responsibility |
|-----------|------|---------------|
| `QrCodeDisplay` | `src/components/qr-code-display.tsx` | Renders QR code using `qrcode.react` |
| `QrDownloadButton` | `src/components/qr-download-button.tsx` | Download QR as PNG |
| `QrPrintButton` | `src/components/qr-print-button.tsx` | Print-friendly QR with brew name |

### Shared / UI Components

All shadcn/ui primitives live in `src/components/ui/` — Button, Card, Input, Select, Badge, Dialog, Table, Tabs, Toast, etc.

---

## API Layer

The backend is a serverless API deployed to one of the supported free-tier providers (Cloudflare Workers, Vercel Functions, or Netlify Functions). It exposes RESTful endpoints consumed by the React frontend via TanStack Query.

### API Endpoints

#### Brews — `/api/brews`

| Method | Endpoint | Auth Required | Description |
|--------|----------|---------------|-------------|
| `GET` | `/api/brews` | No | List all brews, ordered by updated_at desc |
| `GET` | `/api/brews/:brewId` | No | Get single brew with ingredient + event counts |
| `POST` | `/api/brews` | **Yes** | Create a new brew |
| `PUT` | `/api/brews/:brewId` | **Yes** | Update brew metadata |
| `PATCH` | `/api/brews/:brewId/status` | **Yes** | Change brew status |
| `DELETE` | `/api/brews/:brewId` | **Yes** | Delete brew and cascade ingredients/events |

#### Ingredients — `/api/brews/:brewId/ingredients`

| Method | Endpoint | Auth Required | Description |
|--------|----------|---------------|-------------|
| `GET` | `/api/brews/:brewId/ingredients` | No | List all ingredients for a brew |
| `POST` | `/api/brews/:brewId/ingredients` | **Yes** | Add ingredient to brew |
| `PUT` | `/api/brews/:brewId/ingredients/:id` | **Yes** | Edit ingredient |
| `DELETE` | `/api/brews/:brewId/ingredients/:id` | **Yes** | Remove ingredient |

#### Fermentation Events — `/api/brews/:brewId/events`

| Method | Endpoint | Auth Required | Description |
|--------|----------|---------------|-------------|
| `GET` | `/api/brews/:brewId/events` | No | List all events for a brew, ordered by event_date |
| `POST` | `/api/brews/:brewId/events` | **Yes** | Log a fermentation event |
| `DELETE` | `/api/brews/:brewId/events/:id` | **Yes** | Remove an event |

### Validation

All request bodies are validated using Zod schemas on the serverless API before database operations. The same Zod schemas can be shared between frontend and backend via a shared package or copy.

---

## Authentication & Access Model

BrewLog is a **single-user personal app**. Authentication is intentionally simple — no OAuth, no JWT, no user registration flows.

### Write Operations — API Key

All write operations (`POST`, `PUT`, `PATCH`, `DELETE`) require a static API key sent as a request header:

```
X-API-Key: <your-secret-api-key>
```

The API key is set as an environment variable on the serverless API (`API_KEY`). The auth middleware checks for this header on all mutating requests and returns `401 Unauthorized` if it is missing or incorrect.

The frontend stores the API key in the browser's `localStorage` after the user enters it once (a simple "unlock" prompt on first visit). All subsequent API requests from the frontend include the key automatically via the fetch wrapper.

### Read Operations — Public

All `GET` endpoints are **public and unauthenticated**. This enables the QR code access model: anyone who scans a QR code on a fermenter can view the brew details, ingredients, and fermentation log without needing credentials.

### Access Summary

| Operation | Auth Required | Use Case |
|-----------|---------------|----------|
| Read brew details | No | QR code scanning, public viewing |
| Read ingredients | No | QR code scanning, public viewing |
| Read fermentation log | No | QR code scanning, public viewing |
| Create/edit/delete brews | **Yes** (API key) | Owner only |
| Create/edit/delete ingredients | **Yes** (API key) | Owner only |
| Log/delete fermentation events | **Yes** (API key) | Owner only |

---

## QR Code Flow

### Generation

1. When a brew is created, a unique `id` is assigned via `nanoid`.
2. On the `/brews/:brewId/qr` page, the QR code is generated **client-side** using the `qrcode.react` library.
3. The QR encodes the URL: `{APP_BASE_URL}/brews/{brewId}`
4. `APP_BASE_URL` is set via the `VITE_APP_URL` environment variable — defaults to `http://localhost:5173` in development.

### URL Format

```
https://your-domain.com/brews/abc123nanoid
```

QR codes link to the **read-only brew detail page**, not the quick-log page. This means anyone scanning the QR can view brew details without authentication. The owner can navigate to the quick-log form from the brew detail page when authenticated.

### QR Display Page Features

- Large QR code image rendered as SVG via `qrcode.react`
- Brew name and type displayed above the QR
- **Download as PNG** button — saves a print-ready image
- **Print** button — opens print dialog with a label layout including brew name, type, start date, and QR code

### Scanning Experience

```mermaid
flowchart TD
    A[User scans QR on fermenter] --> B[Phone opens /brews/brewId]
    B --> C{Brew exists?}
    C -->|Yes| D[Show brew detail page - read-only]
    C -->|No| E[Show 404 with link to dashboard]
    D --> F{Owner authenticated?}
    F -->|Yes| G[Show quick-log button]
    F -->|No| H[Read-only view only]
    G --> I[Navigate to quick-log form]
    I --> J[User fills in reading/notes]
    J --> K[Submit with API key]
    K --> L[Toast: Event logged]
```

### Quick-Log Page Design

The `/quick-log/:brewId` page is optimized for mobile and **requires authentication** (API key):

- **Header**: Brew name and current status
- **Event type selector**: Large tap targets — Gravity, Temperature, Tasting, Racking, Addition, Note
- **Dynamic fields**: Based on selected type
  - Gravity → gravity input with numeric keyboard
  - Temperature → temp input in °C
  - Tasting → notes textarea
  - Racking → notes textarea
  - Addition → name + quantity (g/kg/ml/L) + notes
  - Note → notes textarea
- **Date/time**: Defaults to now, editable
- **Submit button**: Large, prominent
- **Success state**: Toast notification, form resets, link to full log

---

## Offline Support & Local Caching

Since the app is used at the fermenter where WiFi may be spotty, BrewLog includes offline support via Service Worker and IndexedDB.

### Architecture

```mermaid
flowchart TD
    A[User opens app] --> B{Online?}
    B -->|Yes| C[Fetch from API]
    C --> D[Cache response in IndexedDB]
    B -->|No| E[Serve from IndexedDB cache]
    
    F[User submits form offline] --> G[Save to IndexedDB outbox]
    G --> H{Online?}
    H -->|Yes| I[Sync outbox to API]
    H -->|No| J[Queue for later sync]
    
    K[Connection restored] --> L[Background sync triggers]
    L --> I
    I --> M[Clear synced items from outbox]
```

### Implementation Details

| Concern | Technology | Details |
|---------|-----------|---------|
| **Service Worker** | Vite PWA plugin (`vite-plugin-pwa`) | Caches static assets (HTML, JS, CSS) for offline shell loading |
| **Data Cache** | IndexedDB (via `idb` library) | Stores API responses locally — brew list, brew details, ingredients, events |
| **Offline Writes** | IndexedDB outbox queue | When offline, write operations are saved to an "outbox" store in IndexedDB |
| **Background Sync** | Service Worker Background Sync API | When connectivity is restored, queued writes are replayed to the API in order |
| **Conflict Resolution** | Last-write-wins | Simple strategy — the most recent write wins. Acceptable for single-user app |
| **Sync Status UI** | Zustand store + toast notifications | Shows pending sync count, notifies on successful sync |

### Offline Capabilities

| Feature | Offline Support |
|---------|----------------|
| View dashboard / brew list | ✅ Cached data from last online visit |
| View brew details, ingredients, events | ✅ Cached data |
| Log a fermentation event | ✅ Queued in outbox, synced when online |
| Create a new brew | ✅ Queued in outbox, synced when online |
| Add ingredients | ✅ Queued in outbox, synced when online |
| Generate/view QR code | ✅ Client-side generation, works offline |
| Delete operations | ⚠️ Queued but may conflict — user warned |

### Cache Freshness

- TanStack Query's `staleTime` and `gcTime` settings control how long cached data is considered fresh.
- On reconnection, a full refetch is triggered for any stale queries.
- IndexedDB cache is updated on every successful API response.

---

## Project File / Folder Structure

```
brewlog/
├── frontend/                       # React 19 SPA (Vite)
│   ├── src/
│   │   ├── main.tsx                # App entry point
│   │   ├── App.tsx                 # Router setup
│   │   ├── index.css               # Global styles (Tailwind)
│   │   ├── routes/                 # Route components (React Router v7)
│   │   │   ├── root.tsx
│   │   │   ├── dashboard.tsx
│   │   │   ├── brews/
│   │   │   │   ├── new.tsx
│   │   │   │   └── $brewId/
│   │   │   │       ├── layout.tsx
│   │   │   │       ├── index.tsx
│   │   │   │       ├── ingredients.tsx
│   │   │   │       ├── log.tsx
│   │   │   │       ├── qr.tsx
│   │   │   │       └── edit.tsx
│   │   │   ├── quick-log/
│   │   │   │   └── $brewId.tsx
│   │   │   └── not-found.tsx
│   │   ├── components/
│   │   │   ├── ui/                 # shadcn/ui primitives
│   │   │   ├── forms/
│   │   │   │   ├── brew-form.tsx
│   │   │   │   ├── ingredient-form.tsx
│   │   │   │   ├── event-form.tsx
│   │   │   │   └── quick-log-form.tsx
│   │   │   ├── app-nav.tsx
│   │   │   ├── brew-card.tsx
│   │   │   ├── brew-dashboard.tsx
│   │   │   ├── brew-status-badge.tsx
│   │   │   ├── brew-summary.tsx
│   │   │   ├── brew-type-icon.tsx
│   │   │   ├── event-card.tsx
│   │   │   ├── event-timeline.tsx
│   │   │   ├── event-type-icon.tsx
│   │   │   ├── gravity-chart.tsx
│   │   │   ├── ingredient-row.tsx
│   │   │   ├── ingredient-table.tsx
│   │   │   ├── qr-code-display.tsx
│   │   │   ├── qr-download-button.tsx
│   │   │   ├── qr-print-button.tsx
│   │   │   ├── status-filter.tsx
│   │   │   └── sync-status.tsx     # Shows offline queue status
│   │   ├── layouts/
│   │   │   ├── root-layout.tsx
│   │   │   └── brew-detail-layout.tsx
│   │   ├── lib/
│   │   │   ├── api-client.ts       # Fetch wrapper with API key header
│   │   │   ├── queries.ts          # TanStack Query hooks
│   │   │   ├── mutations.ts        # TanStack Query mutation hooks
│   │   │   ├── store.ts            # Zustand store(s)
│   │   │   ├── offline.ts          # IndexedDB cache + outbox logic
│   │   │   ├── sync.ts             # Background sync orchestration
│   │   │   ├── utils.ts            # Shared utilities (ABV calc, date formatting)
│   │   │   ├── types.ts            # Shared TypeScript types/enums
│   │   │   └── validators.ts       # Zod schemas for form validation
│   │   ├── sw.ts                   # Service Worker registration
│   │   └── assets/                 # Static assets (icons, images)
│   ├── public/
│   │   └── icons/                  # App icons, favicon
│   ├── .env                        # VITE_APP_URL, VITE_API_URL
│   ├── vite.config.ts              # Vite configuration (with PWA plugin)
│   ├── tailwind.config.ts          # Tailwind configuration
│   ├── tsconfig.json
│   └── package.json
│
├── api/                            # Serverless API (Cloudflare Workers / Vercel Functions / Netlify Functions)
│   ├── src/
│   │   ├── index.ts                # Entry point / router
│   │   ├── routes/
│   │   │   ├── brews.ts            # Brew CRUD handlers
│   │   │   ├── ingredients.ts      # Ingredient CRUD handlers
│   │   │   └── events.ts           # Event CRUD handlers
│   │   ├── db/
│   │   │   ├── index.ts            # Drizzle client + PostgreSQL connection
│   │   │   ├── schema.ts           # Drizzle table definitions
│   │   │   └── migrations/         # Generated migration files
│   │   ├── middleware/
│   │   │   └── auth.ts             # API key validation middleware
│   │   ├── validators.ts           # Zod schemas for request validation
│   │   └── types.ts                # Shared TypeScript types
│   ├── wrangler.toml               # Cloudflare Workers config (if using CF Workers)
│   ├── tsconfig.json
│   └── package.json
│
├── ARCHITECTURE.md                 # This document
└── README.md
```

---

## Deployment Architecture

The application is split into three independently deployed tiers:

```mermaid
flowchart TB
    subgraph Client
        A[React 19 SPA<br/>Static Files<br/>+ Service Worker]
    end

    subgraph Static Host
        B[Cloudflare Pages<br/>OR Netlify<br/>OR Vercel Static]
    end

    subgraph Serverless API
        C[Cloudflare Workers<br/>OR Vercel Functions<br/>OR Netlify Functions]
    end

    subgraph Data Layer
        D[PostgreSQL<br/>Self-hosted, always-on]
    end

    A -->|Deployed to| B
    B -->|API requests<br/>+ API key header| C
    C -->|Drizzle ORM<br/>via connection string| D
```

### Free-Tier Provider Options

| Tier | Provider | Free Tier Limits |
|------|----------|-----------------|
| **Static Frontend** | Cloudflare Pages | Unlimited requests, 500 builds/month |
| | Netlify | 100 GB bandwidth/month, 300 build minutes/month |
| | Vercel (static) | 100 GB bandwidth/month |
| **Serverless API** | Cloudflare Workers | 100,000 requests/day, 10 ms CPU time |
| | Vercel Functions | 100 GB-hours/month, 100,000 invocations/day |
| | Netlify Functions | 125,000 requests/month, 100 hours/month |
| **Database** | Self-hosted PostgreSQL | Owner-provided, always-on server |

### Environment Variables

**Frontend** (`.env`):
```
VITE_APP_URL=https://brewlog.pages.dev
VITE_API_URL=https://api.brewlog.workers.dev
```

**API** (environment/secrets):
```
DATABASE_URL=postgresql://user:password@your-server:5432/brewlog
API_KEY=your-secret-api-key
```

---

## Key npm Dependencies

### Frontend — Production

| Package | Purpose |
|---------|---------|
| `react` / `react-dom` | React 19 UI library |
| `react-router` | Client-side routing (v7) |
| `@tanstack/react-query` | Server state management, caching, mutations |
| `zustand` | Lightweight local state management |
| `qrcode.react` | Client-side QR code generation as React component |
| `zod` | Schema validation for forms |
| `date-fns` | Date formatting and manipulation |
| `recharts` | Charting library for gravity/temperature graphs |
| `lucide-react` | Icon library — used by shadcn/ui |
| `class-variance-authority` | Component variant styling — shadcn/ui dependency |
| `clsx` / `tailwind-merge` | Conditional class name utilities |
| `idb` | Lightweight IndexedDB wrapper for offline caching |
| `vite-plugin-pwa` | Service Worker generation and PWA support |

### Frontend — Development

| Package | Purpose |
|---------|---------|
| `vite` | Build tool and dev server |
| `@vitejs/plugin-react` | React support for Vite |
| `typescript` | Type checking |
| `tailwindcss` / `postcss` / `autoprefixer` | CSS toolchain |
| `eslint` | Linting |

### API — Production

| Package | Purpose |
|---------|---------|
| `drizzle-orm` | Type-safe ORM for PostgreSQL |
| `postgres` | PostgreSQL driver (serverless-compatible) |
| `nanoid` | Short unique ID generation |
| `zod` | Request body validation |

### API — Development

| Package | Purpose |
|---------|---------|
| `typescript` | Type checking |
| `drizzle-kit` | Migration generation and DB studio |
| `wrangler` | Cloudflare Workers local dev (if using CF Workers) |

---

## Architectural Decisions & Trade-offs

### 1. React 19 SPA + Serverless API Over Next.js

**Decision**: Use a React 19 SPA (Vite) with a separate serverless API instead of a Next.js monolith.

**Rationale**: Decoupling frontend and backend allows independent deployment, scaling, and technology choices. The static frontend can be hosted for free on any CDN. The serverless API scales to zero and stays within free tiers. No vendor lock-in to a specific framework's deployment model.

**Trade-off**: No server-side rendering — the app is fully client-rendered. Acceptable for a personal tool where SEO is irrelevant. Requires managing CORS between frontend and API.

### 2. Self-Hosted PostgreSQL Over Managed Services

**Decision**: Use the owner's self-hosted, always-on PostgreSQL server instead of a managed database service.

**Rationale**: The owner already has an always-on Postgres server available. Direct connection via a standard `DATABASE_URL` connection string keeps things simple. No vendor lock-in, no managed service limitations, no free-tier constraints. Drizzle ORM connects via the `postgres-js` driver, which works well in serverless environments.

**Trade-off**: The owner is responsible for database backups, updates, and availability. Acceptable since this is a personal app and the server is already maintained.

### 3. Simple API Key Auth Over OAuth/JWT

**Decision**: Use a static API key header for write operations instead of OAuth, JWT, or any user registration system.

**Rationale**: This is a single-user personal app. There is no need for user registration, login flows, token refresh logic, or session management. A simple `X-API-Key` header on mutating requests is sufficient. Read operations are public to support the QR code access model.

**Trade-off**: The API key must be kept secret. It's stored in the browser's `localStorage` and sent over HTTPS. If compromised, it can be rotated by changing the environment variable on the API.

### 4. Metric Units Only

**Decision**: All measurements use metric units exclusively — liters (L), grams (g), kilograms (kg), milliliters (ml), Celsius (°C).

**Rationale**: Metric is the international standard and simplifies the data model by eliminating unit conversion logic and dual-unit storage. No `temperature_unit` column needed; temperature is always Celsius. The unit enum is strictly `g`, `kg`, `ml`, `L` — no imperial units.

**Trade-off**: None for this user. A client-side display toggle could be added later without changing the database schema if needed.

### 5. TanStack Query for Server State

**Decision**: Use TanStack Query (React Query) for all API data fetching and mutations.

**Rationale**: Provides caching, background refetching, optimistic updates, and request deduplication out of the box. Eliminates the need for manual loading/error state management. Pairs well with a REST API and integrates with the offline caching strategy.

**Trade-off**: Adds a dependency. The alternative — raw `fetch` with `useEffect` — would require significantly more boilerplate for the same functionality.

### 6. nanoid for IDs Instead of Auto-Increment

**Decision**: Use `nanoid` — 21-character URL-safe strings — for all primary keys.

**Rationale**: URL-safe without encoding, no sequential enumeration, safe for use in QR code URLs, and avoids integer overflow concerns. Generated client-side or server-side without DB coordination. Enables offline ID generation for the outbox queue.

**Trade-off**: Slightly larger storage than integers. No natural ordering — use `created_at` for ordering instead.

### 7. Client-Side QR Generation

**Decision**: Generate QR codes in the browser using `qrcode.react` rather than on the server.

**Rationale**: No need to store QR images. The QR is deterministic — same URL always produces the same QR. Client-side generation avoids server load and storage. The `qrcode.react` library renders QR codes as React components (SVG or Canvas), making download/print workflows straightforward. Works offline.

**Trade-off**: Requires JavaScript. Acceptable since QR display is not a critical path.

### 8. Recharts for Gravity Visualization

**Decision**: Use Recharts for the gravity-over-time chart.

**Rationale**: React-native charting library, good TypeScript support, responsive, and handles line charts well. Lighter than D3 for this use case.

**Trade-off**: Adds bundle size. Can be lazy-loaded with `React.lazy()` and only loaded on the brew detail page.

### 9. Zod for Validation

**Decision**: Use Zod schemas for both client-side form validation and server-side request validation.

**Rationale**: Single source of truth for validation rules. Integrates well with React Hook Form if needed later. The API must validate input regardless of client-side checks.

### 10. Mobile-First Quick-Log

**Decision**: The `/quick-log/:brewId` route is a separate, mobile-optimized page rather than a modal or the same page as the full log.

**Rationale**: QR scanning happens on a phone. The quick-log page needs large tap targets, minimal scrolling, and fast load times. Keeping it separate allows optimizing the layout and bundle independently.

### 11. Service Worker + IndexedDB for Offline Support

**Decision**: Use a Service Worker for asset caching and IndexedDB for data caching and offline write queuing.

**Rationale**: The app is used at the fermenter where WiFi may be spotty. The user needs to log gravity readings and temperature even without a reliable connection. Service Worker caches the app shell for instant loading. IndexedDB stores API data locally and queues writes in an outbox that syncs when connectivity is restored. This is a proven PWA pattern that works well for single-user apps.

**Trade-off**: Adds complexity to the frontend (Service Worker lifecycle, sync logic, conflict handling). Acceptable because reliable logging at the fermenter is a core use case.

---

## Future Considerations

These are explicitly out of scope for v1 but worth noting:

- **Recipe templates**: Save and reuse ingredient lists
- **Batch cloning**: Duplicate a brew as a starting point
- **Photo attachments**: Add photos to events (use Cloudflare R2 or S3-compatible storage)
- **Export**: Export brew data as JSON or CSV
- **PWA install prompt**: Home screen install for mobile
- **Notifications**: Reminders for gravity checks or dry hop schedules
- **tRPC migration**: Replace REST with tRPC for end-to-end type safety between frontend and API
