# BrewLog — Implementation Tasks

> Granular task breakdown derived from [IMPLEMENTATION_PHASES.md](./IMPLEMENTATION_PHASES.md) and [ARCHITECTURE.md](../ARCHITECTURE.md).
> Each task is self-contained, ordered, and designed for an AI coding agent to execute in a single session.

---

## Task ID Format

`P{phase}-T{task}` — e.g., `P1-T1` is Phase 1, Task 1.

---

## Phase 1 — Project Scaffold & Database Schema

### P1-T1: Initialize monorepo root and workspace configuration

**Description**: Create the monorepo root with npm workspaces, shared TypeScript config, and environment scaffolding.

**Files to create**:
- `package.json` — root package with `"workspaces": ["frontend", "api"]`, scripts for running both packages
- `tsconfig.base.json` — shared TypeScript config with `strict: true`, `target: "ES2022"`, `module: "ESNext"`
- `.gitignore` — Node.js defaults, `.env`, `node_modules/`, `dist/`, `.wrangler/`
- `.env.example` — document `DATABASE_URL`, `API_KEY`, `VITE_APP_URL`, `VITE_API_URL`

**Dependencies**: None

**Acceptance criteria**:
- `package.json` has valid `workspaces` config pointing to `frontend` and `api`
- `tsconfig.base.json` exists with strict TypeScript settings
- `.gitignore` covers `node_modules`, `dist`, `.env`, `.wrangler`
- `.env.example` documents all four environment variables

**Complexity**: S

---

### P1-T2: Scaffold frontend with Vite + React 19 + TypeScript

**Description**: Initialize the frontend package with Vite, React 19, TypeScript, and basic entry files.

**Files to create**:
- `frontend/package.json` — dependencies: `react@19`, `react-dom@19`, `typescript`; devDeps: `vite`, `@vitejs/plugin-react`
- `frontend/tsconfig.json` — extends `../tsconfig.base.json`, includes `src`
- `frontend/vite.config.ts` — React plugin, dev server on port 5173
- `frontend/index.html` — Vite entry HTML with `<div id="root">`
- `frontend/src/main.tsx` — React 19 `createRoot` rendering `<App />`
- `frontend/src/App.tsx` — placeholder component with "BrewLog" heading

**Dependencies**: P1-T1

**Acceptance criteria**:
- `cd frontend && npm install && npm run dev` starts Vite dev server on `localhost:5173`
- Browser shows the placeholder "BrewLog" page
- TypeScript compiles without errors

**Complexity**: S

---

### P1-T3: Configure Tailwind CSS and shadcn/ui in frontend

**Description**: Add Tailwind CSS v4 and initialize shadcn/ui component library.

**Files to create/modify**:
- `frontend/src/index.css` — Tailwind directives (`@import "tailwindcss"`) and shadcn/ui CSS variables
- `frontend/src/lib/utils.ts` — `cn()` utility using `clsx` + `tailwind-merge`
- `frontend/components.json` — shadcn/ui configuration file

**Packages to install** (frontend): `tailwindcss`, `@tailwindcss/vite`, `class-variance-authority`, `clsx`, `tailwind-merge`, `lucide-react`

**Dependencies**: P1-T2

**Acceptance criteria**:
- Tailwind utility classes render correctly in the browser (e.g., `className="text-red-500"` shows red text)
- `cn()` utility merges class names correctly
- shadcn/ui CLI can add components (e.g., `npx shadcn@latest add button`)

**Complexity**: S

---

### P1-T4: Set up React Router v7 with placeholder routes

**Description**: Install React Router v7 and configure the client-side route structure with placeholder pages.

**Files to create/modify**:
- `frontend/src/App.tsx` — wrap with `BrowserRouter`, define route tree
- `frontend/src/routes/root.tsx` — root layout with `<Outlet />`
- `frontend/src/routes/dashboard.tsx` — placeholder "Dashboard" page
- `frontend/src/routes/not-found.tsx` — placeholder 404 page

**Packages to install** (frontend): `react-router`

**Dependencies**: P1-T3

**Acceptance criteria**:
- `/` renders the dashboard placeholder
- Unknown routes render the 404 placeholder
- Root layout wraps all routes with `<Outlet />`
- No console errors in the browser

**Complexity**: S

---

### P1-T5: Wire up TanStack Query provider and Zustand store skeleton

**Description**: Add TanStack Query and Zustand to the frontend with provider setup and an initial store.

**Files to create/modify**:
- `frontend/src/App.tsx` — wrap routes with `<QueryClientProvider>`
- `frontend/src/lib/store.ts` — Zustand store with initial state: `{ apiKey: string | null, isOnline: boolean, pendingSyncCount: number }`
- `frontend/src/lib/types.ts` — TypeScript types for `Brew`, `BrewStatus` (`'planning' | 'fermenting' | 'conditioning' | 'bottled' | 'done'`), `BrewType` (`'beer' | 'mead' | 'cider'`), `Ingredient`, `FermentationEvent`

**Packages to install** (frontend): `@tanstack/react-query`, `zustand`

**Dependencies**: P1-T4

**Acceptance criteria**:
- `QueryClientProvider` wraps the app (visible in React DevTools or no runtime errors)
- Zustand store exports `useAppStore` hook with `apiKey`, `isOnline`, `pendingSyncCount`
- `types.ts` exports all core TypeScript types matching the database schema

**Complexity**: S

---

### P1-T6: Scaffold API with Hono and health-check endpoint

**Description**: Initialize the API package with Hono framework, TypeScript, and a health-check endpoint.

**Files to create**:
- `api/package.json` — dependencies: `hono`, `zod`, `nanoid`; devDeps: `typescript`, `wrangler`, `@types/node`
- `api/tsconfig.json` — extends `../tsconfig.base.json`, includes `src`
- `api/src/index.ts` — Hono app with `GET /api/health` returning `{ status: "ok", timestamp: ... }`
- `api/wrangler.toml` — Cloudflare Workers config with `[vars]` section for `API_KEY`

**Dependencies**: P1-T1

**Acceptance criteria**:
- `cd api && npm install && npm run dev` starts the dev server (wrangler or node)
- `GET /api/health` returns `200` with JSON `{ status: "ok" }`
- TypeScript compiles without errors

**Complexity**: S

---

### P1-T7: Define Drizzle ORM schema and generate first migration

**Description**: Create the Drizzle schema file defining all three tables (`brews`, `ingredients`, `fermentation_events`) with columns, indexes, and relations per the architecture.

**Files to create**:
- `api/src/db/schema.ts` — Drizzle table definitions:
  - `brews`: `id` (text PK, nanoid), `user_id` (text, default `'default'`), `name`, `type`, `batch_size_liters` (real), `target_og` (real), `target_fg` (real), `status` (text, default `'fermenting'`), `start_date`, `end_date`, `notes`, `created_at` (timestamptz), `updated_at` (timestamptz)
  - `ingredients`: `id` (text PK), `brew_id` (text FK → brews.id), `user_id`, `category`, `name`, `quantity` (real), `unit`, `date_added`, `notes`, `created_at`
  - `fermentation_events`: `id` (text PK), `brew_id` (text FK → brews.id), `user_id`, `event_type`, `gravity` (real), `temperature_celsius` (real), `notes`, `event_date`, `created_at`
  - Indexes on `ingredients.brew_id` and `fermentation_events.brew_id`
  - Relations defined via Drizzle `relations()`
- `api/src/db/index.ts` — Drizzle client setup using `postgres-js` adapter, reading `DATABASE_URL` from env
- `api/drizzle.config.ts` — Drizzle Kit config pointing to schema and migrations directory

**Packages to install** (api): `drizzle-orm`, `postgres`, `drizzle-kit`

**Dependencies**: P1-T6

**Acceptance criteria**:
- `npx drizzle-kit generate` produces a migration SQL file in `api/src/db/migrations/`
- Schema defines all three tables with correct column types and constraints
- Foreign keys from `ingredients` and `fermentation_events` to `brews` are defined
- Indexes on `brew_id` columns exist
- `drizzle-kit push` (or `migrate`) applies the schema to a Postgres database without errors

**Complexity**: M

---

### P1-T8: Implement API key auth middleware

**Description**: Create middleware that validates the `X-API-Key` header on all mutating requests (`POST`, `PUT`, `PATCH`, `DELETE`) and passes through `GET` requests.

**Files to create**:
- `api/src/middleware/auth.ts` — Hono middleware function:
  - On `GET`/`OPTIONS` requests: call `next()`
  - On mutating requests: check `X-API-Key` header against `API_KEY` env var
  - Return `401 Unauthorized` JSON response if missing or incorrect
  - Call `next()` if valid

**Files to modify**:
- `api/src/index.ts` — apply auth middleware to all `/api/*` routes

**Dependencies**: P1-T6

**Acceptance criteria**:
- `GET /api/health` returns `200` without any API key
- `POST /api/health-auth-test` without `X-API-Key` returns `401`
- `POST /api/health-auth-test` with correct `X-API-Key` returns `200`
- `OPTIONS` requests pass through (for CORS preflight)

**Complexity**: S

---

### P1-T9: Create Zod validation schemas

**Description**: Define Zod schemas for all API request payloads — brews, ingredients, and fermentation events.

**Files to create**:
- `api/src/validators.ts` — Zod schemas:
  - `createBrewSchema`: `name` (string, min 1), `type` (enum: `beer`, `mead`, `cider`), `batch_size_liters` (number, positive), `target_og` (number, optional), `target_fg` (number, optional), `status` (enum: `planning`, `fermenting`, `conditioning`, `bottled`, `done`, optional, default `fermenting`), `start_date` (string, ISO date), `end_date` (string, optional), `notes` (string, optional)
  - `updateBrewSchema`: same as create but all fields optional (partial)
  - `updateBrewStatusSchema`: `status` (enum: `planning`, `fermenting`, `conditioning`, `bottled`, `done`)
  - `createIngredientSchema`: `category` (enum: `grain`, `hop`, `honey`, `fruit`, `yeast`, `adjunct`, `other`), `name` (string, min 1), `quantity` (number, positive), `unit` (enum: `g`, `kg`, `ml`, `L`), `date_added` (string, optional), `notes` (string, optional)
  - `updateIngredientSchema`: partial of create
  - `createEventSchema`: `event_type` (enum: `gravity_reading`, `temperature`, `racking`, `addition`, `tasting`, `note`, `bottling`, `other`), `gravity` (number, optional), `temperature_celsius` (number, optional), `notes` (string, optional), `event_date` (string, ISO datetime)

**Dependencies**: P1-T6

**Acceptance criteria**:
- Each schema correctly validates well-formed payloads (returns parsed data)
- Each schema rejects malformed payloads with descriptive error messages
- Enum fields only accept documented values
- Required fields cause validation failure when missing
- Optional fields are allowed to be absent

**Complexity**: S

---

### P1-T10: Build frontend API client (fetch wrapper)

**Description**: Create a fetch wrapper that attaches the API key from localStorage and targets the configured API URL.

**Files to create**:
- `frontend/src/lib/api-client.ts` — exports:
  - `apiClient.get(path)` — `GET` request to `${VITE_API_URL}${path}`, returns parsed JSON
  - `apiClient.post(path, body)` — `POST` with JSON body and `X-API-Key` header from `localStorage`
  - `apiClient.put(path, body)` — `PUT` with JSON body and `X-API-Key` header
  - `apiClient.patch(path, body)` — `PATCH` with JSON body and `X-API-Key` header
  - `apiClient.del(path)` — `DELETE` with `X-API-Key` header
  - Reads API key from `localStorage` key `brewlog-api-key`
  - Reads base URL from `import.meta.env.VITE_API_URL`
  - Throws on non-2xx responses with error message from response body

**Files to create**:
- `frontend/.env` — `VITE_API_URL=http://localhost:8787` and `VITE_APP_URL=http://localhost:5173`

**Dependencies**: P1-T5

**Acceptance criteria**:
- `apiClient.get('/api/health')` makes a GET request without API key header
- `apiClient.post('/api/brews', data)` includes `X-API-Key` header from localStorage
- Non-2xx responses throw an error with the response message
- Base URL is read from `VITE_API_URL` environment variable

**Complexity**: S

---

## Phase 2 — Brews API & Core Brew UI

### P2-T1: Implement brews API routes (CRUD)

**Description**: Create all brew CRUD endpoints using Hono, Drizzle ORM, and Zod validation.

**Files to create**:
- `api/src/routes/brews.ts` — Hono route group with:
  - `GET /api/brews` — list all brews ordered by `updated_at` desc
  - `GET /api/brews/:brewId` — get single brew by ID, include ingredient count and event count
  - `POST /api/brews` — validate with `createBrewSchema`, generate `nanoid`, insert into DB
  - `PUT /api/brews/:brewId` — validate with `updateBrewSchema`, update brew, set `updated_at`
  - `PATCH /api/brews/:brewId/status` — validate with `updateBrewStatusSchema`, update status only
  - `DELETE /api/brews/:brewId` — delete brew and cascade (delete related ingredients and events)

**Files to modify**:
- `api/src/index.ts` — mount brews routes

**Dependencies**: P1-T7, P1-T8, P1-T9

**Acceptance criteria**:
- `POST /api/brews` with valid body creates a brew and returns `201` with the brew object
- `GET /api/brews` returns an array of brews ordered by `updated_at` desc
- `GET /api/brews/:id` returns a single brew with `ingredientCount` and `eventCount`
- `PUT /api/brews/:id` updates fields and returns the updated brew
- `PATCH /api/brews/:id/status` updates only the status field
- `DELETE /api/brews/:id` removes the brew and returns `204`
- Invalid payloads return `400` with Zod error details
- Missing/invalid API key on mutating requests returns `401`
- Non-existent brew ID returns `404`

**Complexity**: M

---

### P2-T2: Create frontend TypeScript types and utility functions

**Description**: Define all shared types and utility functions needed for the brew UI.

**Files to create/modify**:
- `frontend/src/lib/types.ts` — add/update types:
  - `Brew` — all fields from the `brews` table, plus optional `ingredientCount` and `eventCount`
  - `BrewStatus` — union type of status values
  - `BrewType` — union type of brew types
  - `IngredientCategory` — union type of ingredient categories
  - `Ingredient` — all fields from the `ingredients` table
  - `EventType` — union type of event types
  - `FermentationEvent` — all fields from the `fermentation_events` table
- `frontend/src/lib/utils.ts` — add utility functions:
  - `calculateABV(og: number, fg: number): number` — standard ABV formula: `(og - fg) * 131.25`
  - `formatDate(dateString: string): string` — format ISO date to human-readable using `date-fns`
  - `formatDateTime(dateString: string): string` — format ISO datetime
  - `brewAge(startDate: string): string` — calculate and format brew age (e.g., "12 days")

**Packages to install** (frontend): `date-fns`

**Dependencies**: P1-T5

**Acceptance criteria**:
- All types match the database schema columns exactly
- `calculateABV(1.050, 1.010)` returns approximately `5.25`
- `formatDate` and `formatDateTime` produce readable strings
- `brewAge` returns a human-readable duration string

**Complexity**: S

---

### P2-T3: Create TanStack Query hooks for brews

**Description**: Implement query and mutation hooks for all brew operations.

**Files to create**:
- `frontend/src/lib/queries.ts` — TanStack Query hooks:
  - `useBrews()` — `GET /api/brews`, returns `Brew[]`, query key `['brews']`
  - `useBrew(brewId: string)` — `GET /api/brews/:brewId`, returns `Brew`, query key `['brews', brewId]`
- `frontend/src/lib/mutations.ts` — TanStack Query mutations:
  - `useCreateBrew()` — `POST /api/brews`, invalidates `['brews']` on success
  - `useUpdateBrew(brewId: string)` — `PUT /api/brews/:brewId`, invalidates `['brews']` and `['brews', brewId]`
  - `useUpdateBrewStatus(brewId: string)` — `PATCH /api/brews/:brewId/status`, invalidates `['brews']` and `['brews', brewId]`
  - `useDeleteBrew()` — `DELETE /api/brews/:brewId`, invalidates `['brews']`

All hooks use `apiClient` from `api-client.ts`.

**Dependencies**: P1-T10, P2-T2

**Acceptance criteria**:
- `useBrews()` fetches and caches the brew list
- `useBrew(id)` fetches and caches a single brew
- `useCreateBrew().mutate(data)` posts to the API and invalidates the brew list cache
- `useUpdateBrew(id).mutate(data)` updates and invalidates relevant caches
- `useDeleteBrew().mutate(id)` deletes and invalidates the brew list cache
- All hooks handle loading, error, and success states

**Complexity**: S

---

### P2-T4: Build RootLayout and AppNav components

**Description**: Create the root layout with navigation bar that wraps all pages.

**Files to create**:
- `frontend/src/layouts/root-layout.tsx` — `RootLayout` component: renders `<AppNav />` and `<Outlet />`, wraps with theme/font providers
- `frontend/src/components/app-nav.tsx` — `AppNav` component: top navigation bar with:
  - BrewLog logo/text linking to `/`
  - "New Brew" button linking to `/brews/new`
  - Responsive design (hamburger menu on mobile or simple layout)

**Packages to install** (frontend, via shadcn): `button` component — run `npx shadcn@latest add button`

**Files to modify**:
- `frontend/src/App.tsx` — use `RootLayout` as the root route element

**Dependencies**: P1-T4, P1-T3

**Acceptance criteria**:
- Navigation bar renders at the top of every page
- Logo/text links to the dashboard (`/`)
- "New Brew" button is visible and links to `/brews/new`
- Layout is responsive — usable on mobile and desktop

**Complexity**: S

---

### P2-T5: Build BrewForm component (create/edit)

**Description**: Create a reusable form component for creating and editing brews.

**Files to create**:
- `frontend/src/components/forms/brew-form.tsx` — `BrewForm` component:
  - Props: `defaultValues?: Partial<Brew>`, `onSubmit: (data) => void`, `isLoading: boolean`
  - Fields: `name` (text input), `type` (select: beer/mead/cider), `batch_size_liters` (number input, label "Batch Size (L)"), `target_og` (number input, optional), `target_fg` (number input, optional), `status` (select: planning/fermenting/conditioning/bottled/done), `start_date` (date input), `end_date` (date input, optional), `notes` (textarea, optional)
  - Client-side validation using Zod schema
  - Submit button with loading state
- `frontend/src/lib/validators.ts` — copy/adapt Zod schemas from API for client-side use

**Packages to install** (frontend, via shadcn): `input`, `select`, `textarea`, `label`, `card` components

**Dependencies**: P2-T2, P2-T4

**Acceptance criteria**:
- Form renders all fields with correct input types
- Type selector shows beer, mead, cider options
- Status selector shows all five status options
- Batch size label shows "(L)" for liters
- Form validates required fields before submission
- `onSubmit` is called with validated data
- Form can be pre-filled with `defaultValues` for edit mode

**Complexity**: M

---

### P2-T6: Build BrewCard, BrewStatusBadge, and BrewTypeIcon components

**Description**: Create the brew card component for the dashboard list and supporting display components.

**Files to create**:
- `frontend/src/components/brew-card.tsx` — `BrewCard` component:
  - Displays brew name, type icon, status badge, batch size in L, start date, age
  - Links to `/brews/:brewId` on click
  - Uses shadcn `Card` component
- `frontend/src/components/brew-status-badge.tsx` — `BrewStatusBadge` component:
  - Props: `status: BrewStatus`
  - Colored badge per status (e.g., fermenting=amber, done=green, planning=blue)
  - Uses shadcn `Badge` component
- `frontend/src/components/brew-type-icon.tsx` — `BrewTypeIcon` component:
  - Props: `type: BrewType`
  - Renders an icon for beer, mead, or cider using `lucide-react` icons

**Packages to install** (frontend, via shadcn): `badge` component

**Dependencies**: P2-T2, P1-T3

**Acceptance criteria**:
- `BrewCard` renders brew info in a card layout and links to the detail page
- `BrewStatusBadge` shows different colors for each status
- `BrewTypeIcon` renders distinct icons for beer, mead, and cider
- Components are responsive and look correct on mobile

**Complexity**: S

---

### P2-T7: Build dashboard page with brew list

**Description**: Implement the dashboard page that lists all brews using TanStack Query.

**Files to modify**:
- `frontend/src/routes/dashboard.tsx` — `Dashboard` page:
  - Uses `useBrews()` hook to fetch all brews
  - Renders a grid of `BrewCard` components
  - Shows loading skeleton while fetching
  - Shows empty state message when no brews exist
  - Orders brews by `updated_at` desc (as returned by API)

**Packages to install** (frontend, via shadcn): `skeleton` component

**Dependencies**: P2-T3, P2-T6

**Acceptance criteria**:
- Dashboard fetches and displays all brews as cards
- Loading state shows skeleton placeholders
- Empty state shows a message with a link to create a brew
- Brews are ordered by most recently updated first
- Clicking a brew card navigates to `/brews/:brewId`

**Complexity**: S

---

### P2-T8: Build create brew page

**Description**: Implement the "New Brew" page that uses `BrewForm` and the create mutation.

**Files to modify**:
- `frontend/src/routes/brews/new.tsx` — `NewBrew` page:
  - Renders `BrewForm` with no default values
  - On submit: calls `useCreateBrew().mutate(data)`
  - On success: navigates to `/brews/:newBrewId`
  - Shows error toast on failure

**Packages to install** (frontend, via shadcn): `toast`/`sonner` component

**Dependencies**: P2-T3, P2-T5

**Acceptance criteria**:
- Form submits and creates a brew in the database
- On success, redirects to the new brew's detail page
- On validation error, shows field-level error messages
- On API error, shows a toast notification

**Complexity**: S

---

### P2-T9: Build BrewDetailLayout with tab navigation

**Description**: Create the brew detail layout with tab navigation for sub-pages (Overview, Ingredients, Log, QR).

**Files to create**:
- `frontend/src/layouts/brew-detail-layout.tsx` — `BrewDetailLayout` component:
  - Uses `useBrew(brewId)` to fetch brew data
  - Renders brew name as page title
  - Tab navigation: Overview, Ingredients, Log, QR (using shadcn `Tabs` or custom nav links)
  - Tabs link to: `/brews/:brewId`, `/brews/:brewId/ingredients`, `/brews/:brewId/log`, `/brews/:brewId/qr`
  - Renders `<Outlet />` for child routes
  - Shows 404 if brew not found

**Files to modify**:
- `frontend/src/App.tsx` — add brew detail routes with `BrewDetailLayout` as layout element

**Packages to install** (frontend, via shadcn): `tabs` component

**Dependencies**: P2-T3, P2-T4

**Acceptance criteria**:
- Brew detail layout renders with brew name and tab navigation
- Tabs highlight the active route
- Clicking a tab navigates to the correct sub-page
- Child route content renders in the `<Outlet />`
- Non-existent brew ID shows a 404 message

**Complexity**: M

---

### P2-T10: Build brew overview (summary) page

**Description**: Implement the brew detail overview page showing brew stats.

**Files to create**:
- `frontend/src/components/brew-summary.tsx` — `BrewSummary` component:
  - Displays: name, type (with icon), status (with badge), batch size in L, target OG, target FG, calculated ABV (if OG and FG available), start date, end date, age, notes
  - Uses utility functions from `utils.ts`
- `frontend/src/routes/brews/[brewId]/index.tsx` — brew overview route page:
  - Renders `BrewSummary` with brew data from parent layout context or own query
  - Shows edit button (links to `/brews/:brewId/edit`) — visible only when authenticated
  - Shows delete button with confirmation dialog — visible only when authenticated

**Packages to install** (frontend, via shadcn): `dialog` (for delete confirmation), `alert-dialog`

**Dependencies**: P2-T9, P2-T2

**Acceptance criteria**:
- Brew overview shows all brew metadata in a readable layout
- ABV is calculated and displayed when both OG and FG are available
- Batch size shows "L" unit label
- Edit and delete buttons appear only when API key is in localStorage
- Delete button shows a confirmation dialog before deleting

**Complexity**: M

---

### P2-T11: Build edit brew page

**Description**: Implement the edit brew page that pre-fills the form with current values.

**Files to create**:
- `frontend/src/routes/brews/[brewId]/edit.tsx` — `EditBrew` page:
  - Uses `useBrew(brewId)` to fetch current brew data
  - Renders `BrewForm` with `defaultValues` set to current brew
  - On submit: calls `useUpdateBrew(brewId).mutate(data)`
  - On success: navigates back to `/brews/:brewId`
  - Shows loading state while fetching brew data

**Dependencies**: P2-T5, P2-T9

**Acceptance criteria**:
- Form pre-fills with current brew values
- Submitting updates the brew in the database
- On success, redirects to the brew overview page
- Changed fields are reflected immediately after redirect

**Complexity**: S

---

### P2-T12: Build API key unlock prompt

**Description**: Create a simple authentication prompt for the owner to enter their API key on first visit.

**Files to create**:
- `frontend/src/components/api-key-prompt.tsx` — `ApiKeyPrompt` component:
  - Modal/dialog that appears when no API key is in localStorage
  - Text input for the API key
  - "Unlock" button that saves the key to `localStorage` under `brewlog-api-key`
  - "Continue as viewer" option to dismiss without entering a key
  - Updates Zustand store `apiKey` state

**Files to modify**:
- `frontend/src/layouts/root-layout.tsx` — show `ApiKeyPrompt` on first visit (check localStorage)
- `frontend/src/lib/store.ts` — add `setApiKey` action, initialize `apiKey` from localStorage

**Dependencies**: P2-T4, P1-T5

**Acceptance criteria**:
- On first visit with no stored API key, the prompt appears
- Entering a key and clicking "Unlock" stores it in localStorage
- Subsequent visits do not show the prompt
- "Continue as viewer" dismisses the prompt without storing a key
- Zustand store reflects the current API key state
- Write operations (create/edit/delete buttons) are conditionally shown based on auth state

**Complexity**: S

---

## Phase 3 — Ingredients API & UI

### P3-T1: Implement ingredients API routes

**Description**: Create all ingredient CRUD endpoints.

**Files to create**:
- `api/src/routes/ingredients.ts` — Hono route group with:
  - `GET /api/brews/:brewId/ingredients` — list all ingredients for a brew, ordered by `created_at`
  - `POST /api/brews/:brewId/ingredients` — validate with `createIngredientSchema`, generate `nanoid`, insert
  - `PUT /api/brews/:brewId/ingredients/:id` — validate with `updateIngredientSchema`, update ingredient
  - `DELETE /api/brews/:brewId/ingredients/:id` — delete ingredient
  - All routes verify the parent brew exists (return `404` if not)

**Files to modify**:
- `api/src/index.ts` — mount ingredients routes

**Dependencies**: P2-T1

**Acceptance criteria**:
- `POST` creates an ingredient linked to the brew and returns `201`
- `GET` returns all ingredients for a brew
- `PUT` updates an ingredient and returns the updated object
- `DELETE` removes an ingredient and returns `204`
- Returns `404` if brew ID doesn't exist
- Returns `404` if ingredient ID doesn't exist
- Invalid payloads return `400` with Zod errors
- Mutating requests require valid API key

**Complexity**: M

---

### P3-T2: Create TanStack Query hooks for ingredients

**Description**: Implement query and mutation hooks for ingredient operations.

**Files to modify**:
- `frontend/src/lib/queries.ts` — add:
  - `useIngredients(brewId: string)` — `GET /api/brews/:brewId/ingredients`, query key `['brews', brewId, 'ingredients']`
- `frontend/src/lib/mutations.ts` — add:
  - `useAddIngredient(brewId: string)` — `POST`, invalidates `['brews', brewId, 'ingredients']`
  - `useUpdateIngredient(brewId: string)` — `PUT`, invalidates `['brews', brewId, 'ingredients']`
  - `useDeleteIngredient(brewId: string)` — `DELETE`, invalidates `['brews', brewId, 'ingredients']`

**Dependencies**: P1-T10, P2-T2

**Acceptance criteria**:
- `useIngredients(brewId)` fetches and caches ingredients for a brew
- Mutations correctly call the API and invalidate the ingredient cache
- All hooks handle loading, error, and success states

**Complexity**: S

---

### P3-T3: Build IngredientForm component

**Description**: Create the form for adding and editing ingredients with metric unit selection.

**Files to create**:
- `frontend/src/components/forms/ingredient-form.tsx` — `IngredientForm` component:
  - Props: `brewId: string`, `defaultValues?: Partial<Ingredient>`, `onSubmit`, `onCancel`, `isLoading`
  - Fields: `category` (select: grain/hop/honey/fruit/yeast/adjunct/other), `name` (text), `quantity` (number), `unit` (select: g/kg/ml/L), `date_added` (date, optional), `notes` (textarea, optional)
  - Client-side Zod validation
  - Compact inline layout suitable for table/list context

**Dependencies**: P2-T2, P1-T3

**Acceptance criteria**:
- Form renders all fields with correct input types
- Category selector shows all seven options
- Unit selector shows only metric units: g, kg, ml, L
- Form validates required fields
- Can be pre-filled for edit mode
- Compact layout works inline within a table or list

**Complexity**: S

---

### P3-T4: Build IngredientTable and IngredientRow components

**Description**: Create the ingredient list display with edit and delete actions.

**Files to create**:
- `frontend/src/components/ingredient-table.tsx` — `IngredientTable` component:
  - Props: `ingredients: Ingredient[]`, `brewId: string`, `isAuthenticated: boolean`
  - Renders a table with columns: Category, Name, Quantity, Unit, Date Added, Notes, Actions
  - Actions column shows edit/delete buttons only when authenticated
- `frontend/src/components/ingredient-row.tsx` — `IngredientRow` component:
  - Displays a single ingredient row
  - Edit button toggles inline `IngredientForm` for editing
  - Delete button with confirmation

**Packages to install** (frontend, via shadcn): `table` component

**Dependencies**: P3-T3, P2-T2

**Acceptance criteria**:
- Table displays all ingredients with correct columns
- Edit button shows inline form pre-filled with ingredient data
- Delete button shows confirmation before deleting
- Actions are hidden for unauthenticated users
- Empty state message when no ingredients exist

**Complexity**: M

---

### P3-T5: Build ingredients route page and wire up tab

**Description**: Create the ingredients route page and connect it to the brew detail layout.

**Files to create**:
- `frontend/src/routes/brews/[brewId]/ingredients.tsx` — ingredients page:
  - Uses `useIngredients(brewId)` to fetch ingredients
  - Renders `IngredientTable` with fetched data
  - Shows `IngredientForm` for adding new ingredients (when authenticated)
  - Loading skeleton while fetching

**Files to modify**:
- `frontend/src/App.tsx` — ensure ingredients route is registered under brew detail layout

**Dependencies**: P3-T2, P3-T4, P2-T9

**Acceptance criteria**:
- Navigating to `/brews/:brewId/ingredients` shows the ingredient list
- "Ingredients" tab is active in the brew detail layout
- Adding an ingredient via the form appears in the table immediately
- Editing an ingredient updates the row
- Deleting an ingredient removes it from the table
- Unauthenticated users see the list but not the add/edit/delete controls

**Complexity**: S

---

## Phase 4 — Fermentation Events API & UI

### P4-T1: Implement fermentation events API routes

**Description**: Create all event CRUD endpoints.

**Files to create**:
- `api/src/routes/events.ts` — Hono route group with:
  - `GET /api/brews/:brewId/events` — list all events for a brew, ordered by `event_date` desc
  - `POST /api/brews/:brewId/events` — validate with `createEventSchema`, generate `nanoid`, insert
  - `DELETE /api/brews/:brewId/events/:id` — delete event
  - All routes verify the parent brew exists

**Files to modify**:
- `api/src/index.ts` — mount events routes

**Dependencies**: P2-T1

**Acceptance criteria**:
- `POST` creates an event linked to the brew and returns `201`
- `GET` returns all events ordered by `event_date` desc
- `DELETE` removes an event and returns `204`
- Returns `404` if brew or event ID doesn't exist
- Invalid payloads return `400`
- Mutating requests require valid API key

**Complexity**: S

---

### P4-T2: Create TanStack Query hooks for events

**Description**: Implement query and mutation hooks for event operations.

**Files to modify**:
- `frontend/src/lib/queries.ts` — add:
  - `useEvents(brewId: string)` — `GET /api/brews/:brewId/events`, query key `['brews', brewId, 'events']`
- `frontend/src/lib/mutations.ts` — add:
  - `useAddEvent(brewId: string)` — `POST`, invalidates `['brews', brewId, 'events']` and `['brews', brewId]` (for event count)
  - `useDeleteEvent(brewId: string)` — `DELETE`, invalidates `['brews', brewId, 'events']` and `['brews', brewId]`

**Dependencies**: P1-T10, P2-T2

**Acceptance criteria**:
- `useEvents(brewId)` fetches and caches events for a brew
- Mutations correctly call the API and invalidate event and brew caches
- All hooks handle loading, error, and success states

**Complexity**: S

---

### P4-T3: Build EventForm component

**Description**: Create the event form with dynamic fields based on event type.

**Files to create**:
- `frontend/src/components/forms/event-form.tsx` — `EventForm` component:
  - Props: `brewId: string`, `onSubmit`, `isLoading`
  - Fields:
    - `event_type` (select: gravity_reading, temperature, racking, addition, tasting, note, bottling, other)
    - Dynamic fields based on type:
      - `gravity_reading` → `gravity` (number input)
      - `temperature` → `temperature_celsius` (number input, label "Temperature (°C)")
      - `addition` → `notes` (textarea for what was added)
      - All types → `notes` (textarea), `event_date` (datetime input, defaults to now)
  - Client-side Zod validation

**Dependencies**: P2-T2, P1-T3

**Acceptance criteria**:
- Event type selector shows all eight options
- Selecting "gravity_reading" shows the gravity input field
- Selecting "temperature" shows the temperature input with °C label
- Date/time defaults to current datetime
- Form validates required fields
- Dynamic fields appear/disappear based on selected type

**Complexity**: M

---

### P4-T4: Build EventTimeline, EventCard, and EventTypeIcon components

**Description**: Create the event timeline display components.

**Files to create**:
- `frontend/src/components/event-timeline.tsx` — `EventTimeline` component:
  - Props: `events: FermentationEvent[]`, `brewId: string`, `isAuthenticated: boolean`
  - Renders events in a vertical timeline layout, ordered by `event_date` desc
  - Each event rendered as an `EventCard`
- `frontend/src/components/event-card.tsx` — `EventCard` component:
  - Displays event type icon, event type label, date/time, gravity (if present), temperature in °C (if present), notes
  - Delete button (when authenticated) with confirmation
- `frontend/src/components/event-type-icon.tsx` — `EventTypeIcon` component:
  - Props: `type: EventType`
  - Renders a distinct `lucide-react` icon for each event type

**Dependencies**: P2-T2, P1-T3

**Acceptance criteria**:
- Timeline renders events in chronological order (newest first)
- Each event card shows relevant data based on event type
- Gravity readings show the gravity value
- Temperature readings show value with °C unit
- Delete button appears only for authenticated users
- Event type icons are visually distinct

**Complexity**: M

---

### P4-T5: Build GravityChart component

**Description**: Create a line chart showing gravity readings over time.

**Files to create**:
- `frontend/src/components/gravity-chart.tsx` — `GravityChart` component:
  - Props: `events: FermentationEvent[]`
  - Filters events to only `gravity_reading` type with non-null gravity
  - Renders a Recharts `LineChart` with:
    - X-axis: `event_date` (formatted dates)
    - Y-axis: `gravity` values
    - Tooltip showing date and gravity value
    - Responsive container
  - Shows empty state if no gravity readings exist

**Packages to install** (frontend): `recharts`

**Dependencies**: P2-T2

**Acceptance criteria**:
- Chart renders gravity readings as a line over time
- X-axis shows formatted dates
- Y-axis shows gravity values
- Tooltip displays date and gravity on hover
- Chart is responsive (resizes with container)
- Empty state shown when no gravity readings exist

**Complexity**: S

---

### P4-T6: Build fermentation log route page and update brew summary

**Description**: Create the log route page and update the brew summary to show event-related data.

**Files to create**:
- `frontend/src/routes/brews/[brewId]/log.tsx` — log page:
  - Uses `useEvents(brewId)` to fetch events
  - Renders `EventForm` for adding events (when authenticated)
  - Renders `GravityChart` with events
  - Renders `EventTimeline` with events
  - Loading skeleton while fetching

**Files to modify**:
- `frontend/src/components/brew-summary.tsx` — update to show:
  - Latest gravity reading (from events)
  - Calculated ABV using OG and latest gravity (if available)
  - Total event count
- `frontend/src/App.tsx` — ensure log route is registered under brew detail layout

**Dependencies**: P4-T2, P4-T3, P4-T4, P4-T5, P2-T10

**Acceptance criteria**:
- Navigating to `/brews/:brewId/log` shows the event timeline and gravity chart
- "Log" tab is active in the brew detail layout
- Adding an event via the form appears in the timeline immediately
- Gravity chart updates when a new gravity reading is added
- Deleting an event removes it from the timeline and chart
- Brew summary shows latest gravity, ABV, and event count
- Unauthenticated users see the timeline but not the add/delete controls

**Complexity**: M

---

## Phase 5 — QR Codes & Quick-Log Flow

### P5-T1: Build QR code display components

**Description**: Create QR code rendering, download, and print components.

**Files to create**:
- `frontend/src/components/qr-code-display.tsx` — `QrCodeDisplay` component:
  - Props: `brewId: string`, `brewName: string`, `brewType: BrewType`
  - Generates QR encoding `${VITE_APP_URL}/brews/${brewId}`
  - Renders QR as SVG using `qrcode.react`
  - Displays brew name and type above the QR
- `frontend/src/components/qr-download-button.tsx` — `QrDownloadButton` component:
  - Converts QR SVG to PNG and triggers download
  - File named `brewlog-{brewName}-qr.png`
- `frontend/src/components/qr-print-button.tsx` — `QrPrintButton` component:
  - Opens print dialog with a label layout: brew name, type, start date, QR code

**Packages to install** (frontend): `qrcode.react`

**Dependencies**: P2-T2, P1-T3

**Acceptance criteria**:
- QR code renders as SVG encoding the correct brew URL
- QR URL uses `VITE_APP_URL` environment variable
- Download button saves a PNG file
- Print button opens the browser print dialog with a formatted label
- Components render correctly on mobile and desktop

**Complexity**: M

---

### P5-T2: Build QR code route page and wire up tab

**Description**: Create the QR code route page within the brew detail layout.

**Files to create**:
- `frontend/src/routes/brews/[brewId]/qr.tsx` — QR page:
  - Uses brew data from parent layout or `useBrew(brewId)`
  - Renders `QrCodeDisplay`, `QrDownloadButton`, `QrPrintButton`
  - Shows instructions for printing and attaching to fermenter

**Files to modify**:
- `frontend/src/App.tsx` — ensure QR route is registered under brew detail layout

**Dependencies**: P5-T1, P2-T9

**Acceptance criteria**:
- Navigating to `/brews/:brewId/qr` shows the QR code page
- "QR" tab is active in the brew detail layout
- QR code encodes the correct URL
- Download and print buttons work correctly

**Complexity**: S

---

### P5-T3: Build QuickLogForm component

**Description**: Create the mobile-optimized quick-log form for adding events via QR scan.

**Files to create**:
- `frontend/src/components/forms/quick-log-form.tsx` — `QuickLogForm` component:
  - Props: `brewId: string`, `brewName: string`, `brewStatus: BrewStatus`
  - Large tap targets for event type selection (grid of large buttons)
  - Event types: Gravity, Temperature, Tasting, Racking, Addition, Note
  - Dynamic fields based on selected type:
    - Gravity → numeric input for gravity
    - Temperature → numeric input for °C
    - Tasting/Racking/Note → textarea
    - Addition → name + quantity (g/kg/ml/L) + notes
  - Date/time defaults to now, editable
  - Large submit button
  - Uses `useAddEvent` mutation
  - On success: toast notification, form resets

**Dependencies**: P4-T2, P2-T2, P1-T3

**Acceptance criteria**:
- Event type buttons are large and easy to tap on mobile
- Selecting a type shows the appropriate input fields
- Gravity input uses numeric keyboard on mobile
- Temperature input shows °C label
- Date/time defaults to current time
- Submitting creates an event and shows success toast
- Form resets after successful submission

**Complexity**: M

---

### P5-T4: Build quick-log route page

**Description**: Create the quick-log route page that requires authentication.

**Files to create**:
- `frontend/src/routes/quick-log/[brewId].tsx` — quick-log page:
  - Uses `useBrew(brewId)` to fetch brew data
  - Shows brew name and current status as header
  - Renders `QuickLogForm`
  - Requires authentication — if no API key in localStorage, shows the unlock prompt or redirects
  - Shows 404 if brew doesn't exist
  - Link to full fermentation log at bottom

**Files to modify**:
- `frontend/src/App.tsx` — register quick-log route
- `frontend/src/routes/brews/[brewId]/index.tsx` — add "Quick Log" button visible only when authenticated, linking to `/quick-log/:brewId`

**Dependencies**: P5-T3, P2-T12

**Acceptance criteria**:
- `/quick-log/:brewId` renders the quick-log form
- Page shows brew name and status
- Unauthenticated users are prompted to enter API key
- Submitting the form creates an event via the API
- Success toast appears after submission
- Link to full log navigates to `/brews/:brewId/log`
- 404 shown for non-existent brew ID
- "Quick Log" button appears on brew detail page when authenticated

**Complexity**: S

---

## Phase 6 — Dashboard Polish & Filtering

### P6-T1: Build StatusFilter component

**Description**: Create a filter component for filtering brews by status on the dashboard.

**Files to create**:
- `frontend/src/components/status-filter.tsx` — `StatusFilter` component:
  - Props: `activeStatus: BrewStatus | 'all'`, `onFilterChange: (status) => void`, `brewCounts: Record<BrewStatus, number>`
  - Renders filter buttons/tabs for: All, Planning, Fermenting, Conditioning, Bottled, Done
  - Shows brew count per status
  - Visual indication of active filter (highlighted button)

**Dependencies**: P2-T6, P1-T3

**Acceptance criteria**:
- Filter buttons render for all statuses plus "All"
- Clicking a filter calls `onFilterChange` with the selected status
- Active filter is visually highlighted
- Brew counts are displayed per status
- "All" shows total count

**Complexity**: S

---

### P6-T2: Enhance dashboard with filtering and search

**Description**: Add status filtering, name search, and polished layout to the dashboard.

**Files to modify**:
- `frontend/src/routes/dashboard.tsx` — enhance dashboard:
  - Add `StatusFilter` component above the brew grid
  - Add search input for filtering brews by name (client-side)
  - Filter brew list based on active status and search query
  - Calculate brew counts per status for the filter component
  - Improve grid layout — responsive columns (1 on mobile, 2 on tablet, 3 on desktop)

**Dependencies**: P6-T1, P2-T7

**Acceptance criteria**:
- Status filter buttons filter the displayed brews
- Search input filters brews by name (case-insensitive)
- Filters and search work together (AND logic)
- Grid is responsive — 1/2/3 columns based on screen width
- Empty state shown when filters match no brews

**Complexity**: S

---

### P6-T3: Polish BrewCard with full details

**Description**: Enhance the brew card to show all relevant information.

**Files to modify**:
- `frontend/src/components/brew-card.tsx` — enhance `BrewCard`:
  - Show: brew name, type icon, status badge, batch size in L, brew age (from start date), latest gravity reading (if available), start date
  - Improve card layout with consistent spacing and typography
  - Hover effect for interactivity

**Dependencies**: P2-T6

**Acceptance criteria**:
- Card shows all specified information
- Batch size displays with "L" unit
- Brew age is calculated from start date
- Latest gravity is shown if available
- Card has hover effect
- Layout is clean and consistent

**Complexity**: S

---

### P6-T4: Build 404 page and add error/loading states

**Description**: Implement the 404 page and add consistent error boundaries and loading states across the app.

**Files to modify**:
- `frontend/src/routes/not-found.tsx` — 404 page:
  - Friendly message ("Page not found")
  - Link back to dashboard
  - Consistent with app styling

**Files to create**:
- `frontend/src/components/error-boundary.tsx` — React error boundary component:
  - Catches rendering errors
  - Shows user-friendly error message
  - "Try again" button to reset

**Files to modify**:
- `frontend/src/App.tsx` — wrap routes with error boundary
- `frontend/src/layouts/root-layout.tsx` — add error boundary around `<Outlet />`

**Dependencies**: P2-T4

**Acceptance criteria**:
- Unknown routes show the 404 page with a link to dashboard
- Rendering errors are caught by the error boundary
- Error boundary shows a user-friendly message with retry option
- Loading skeletons are used consistently across data-fetching pages

**Complexity**: S

---

### P6-T5: Add toast notifications for all mutations

**Description**: Ensure all create, update, and delete operations show toast notifications.

**Files to modify**:
- `frontend/src/lib/mutations.ts` — add `onSuccess` and `onError` toast notifications to all mutations:
  - Create brew: "Brew created successfully"
  - Update brew: "Brew updated"
  - Delete brew: "Brew deleted"
  - Change status: "Status updated to {status}"
  - Add ingredient: "Ingredient added"
  - Update ingredient: "Ingredient updated"
  - Delete ingredient: "Ingredient removed"
  - Add event: "Event logged"
  - Delete event: "Event removed"
- `frontend/src/layouts/root-layout.tsx` — ensure toast provider/container is mounted

**Dependencies**: P2-T8 (toast setup), P3-T2, P4-T2

**Acceptance criteria**:
- Every mutation shows a success toast on completion
- Every mutation shows an error toast on failure
- Toasts auto-dismiss after a few seconds
- Toast messages are descriptive and specific to the operation

**Complexity**: S

---

## Phase 7 — Offline Support & PWA

### P7-T1: Configure vite-plugin-pwa for Service Worker

**Description**: Set up the PWA plugin to cache static assets for offline app shell loading.

**Files to modify**:
- `frontend/vite.config.ts` — add `vite-plugin-pwa` configuration:
  - `registerType: 'autoUpdate'`
  - Cache static assets (HTML, JS, CSS, fonts)
  - PWA manifest with app name "BrewLog", icons, theme color
- `frontend/public/manifest.json` — PWA manifest file (or inline in plugin config)

**Files to create**:
- `frontend/public/icons/` — app icons (192x192, 512x512) — can be simple placeholder icons

**Packages to install** (frontend): `vite-plugin-pwa`

**Dependencies**: P1-T2

**Acceptance criteria**:
- Service Worker is registered on app load
- Static assets are cached by the Service Worker
- App shell loads when device is offline (after first visit)
- PWA manifest is served correctly
- No console errors related to Service Worker registration

**Complexity**: M

---

### P7-T2: Implement IndexedDB cache for API responses

**Description**: Create an IndexedDB layer that caches API responses for offline reading.

**Files to create**:
- `frontend/src/lib/offline.ts` — IndexedDB cache using `idb` library:
  - Database name: `brewlog-cache`
  - Stores: `brews`, `ingredients`, `events`
  - Functions:
    - `cacheBrews(brews: Brew[])` — store brew list
    - `getCachedBrews(): Promise<Brew[]>` — retrieve cached brews
    - `cacheBrew(brew: Brew)` — store single brew
    - `getCachedBrew(id: string): Promise<Brew | undefined>`
    - `cacheIngredients(brewId: string, ingredients: Ingredient[])` — store ingredients
    - `getCachedIngredients(brewId: string): Promise<Ingredient[]>`
    - `cacheEvents(brewId: string, events: FermentationEvent[])` — store events
    - `getCachedEvents(brewId: string): Promise<FermentationEvent[]>`
    - `clearCache()` — clear all cached data

**Packages to install** (frontend): `idb`

**Dependencies**: P2-T2

**Acceptance criteria**:
- IndexedDB database is created with correct stores
- API responses can be cached and retrieved
- Cached data persists across page reloads
- `clearCache()` removes all cached data
- Functions handle missing data gracefully (return empty arrays)

**Complexity**: M

---

### P7-T3: Integrate IndexedDB cache with TanStack Query

**Description**: Update TanStack Query hooks to cache responses in IndexedDB and fall back to cached data when offline.

**Files to modify**:
- `frontend/src/lib/queries.ts` — update all query hooks:
  - On successful fetch: cache response in IndexedDB
  - On fetch error (offline): return cached data from IndexedDB
  - Configure `staleTime` and `gcTime` for offline-friendly caching
- `frontend/src/lib/store.ts` — add online/offline detection:
  - Listen to `navigator.onLine` and `online`/`offline` events
  - Update `isOnline` state in Zustand store

**Dependencies**: P7-T2, P2-T3, P3-T2, P4-T2

**Acceptance criteria**:
- API responses are cached in IndexedDB after successful fetches
- When offline, queries return cached data instead of failing
- Online/offline state is tracked in Zustand store
- `staleTime` is set to a reasonable value (e.g., 5 minutes)
- Reconnection triggers refetch of stale queries

**Complexity**: M

---

### P7-T4: Implement offline write queue (outbox)

**Description**: Create an outbox queue in IndexedDB for offline write operations.

**Files to modify**:
- `frontend/src/lib/offline.ts` — add outbox functionality:
  - New IndexedDB store: `outbox`
  - `addToOutbox(request: { method: string, url: string, body?: any, timestamp: number })` — queue a write
  - `getOutboxItems(): Promise<OutboxItem[]>` — retrieve all queued writes
  - `removeFromOutbox(id: number)` — remove a synced item
  - `getOutboxCount(): Promise<number>` — count pending items

**Files to modify**:
- `frontend/src/lib/api-client.ts` — when offline, intercept mutating requests and add to outbox instead of fetching:
  - Check `navigator.onLine` before making mutating requests
  - If offline: save to outbox, return optimistic response
  - If online: make normal fetch request

**Dependencies**: P7-T2, P1-T10

**Acceptance criteria**:
- Mutating requests while offline are saved to the outbox
- Outbox items include method, URL, body, and timestamp
- `getOutboxCount()` returns the correct count
- Online requests bypass the outbox and go directly to the API
- Outbox items persist across page reloads

**Complexity**: M

---

### P7-T5: Implement background sync for outbox

**Description**: Create sync logic that replays queued writes when connectivity is restored.

**Files to create**:
- `frontend/src/lib/sync.ts` — background sync orchestration:
  - `syncOutbox()` — replay all outbox items to the API in order (FIFO):
    - For each item: make the fetch request
    - On success: remove from outbox
    - On failure: stop sync, keep remaining items
  - Listen for `online` event to trigger sync
  - Expose sync status (syncing, idle, error)

**Files to modify**:
- `frontend/src/lib/store.ts` — add sync state:
  - `isSyncing: boolean`
  - `pendingSyncCount: number`
  - `setSyncing`, `setPendingSyncCount` actions
- `frontend/src/App.tsx` — initialize sync listener on app mount

**Dependencies**: P7-T4

**Acceptance criteria**:
- When connectivity is restored, outbox items are synced to the API in order
- Successfully synced items are removed from the outbox
- Failed sync stops processing and retains remaining items
- Zustand store reflects sync status and pending count
- Sync triggers automatically on `online` event

**Complexity**: M

---

### P7-T6: Build SyncStatus UI component

**Description**: Create a component that shows the offline/sync status in the navigation bar.

**Files to create**:
- `frontend/src/components/sync-status.tsx` — `SyncStatus` component:
  - Shows online/offline indicator
  - When offline: shows "Offline" badge
  - When items pending: shows count of pending sync items
  - When syncing: shows spinner/animation
  - On successful sync: shows toast notification

**Files to modify**:
- `frontend/src/components/app-nav.tsx` — add `SyncStatus` component to the navigation bar

**Dependencies**: P7-T5, P2-T4

**Acceptance criteria**:
- "Offline" indicator appears when device is offline
- Pending sync count is displayed when items are queued
- Syncing animation shows during active sync
- Toast notification appears when sync completes
- Component is unobtrusive and doesn't block navigation

**Complexity**: S

---

## Phase 8 — Deployment & Production Readiness

### P8-T1: Configure CORS on the API

**Description**: Add CORS headers to the API to allow requests from the frontend origin.

**Files to modify**:
- `api/src/index.ts` — add Hono CORS middleware:
  - Allow origin from environment variable or `*` for development
  - Allow headers: `Content-Type`, `X-API-Key`
  - Allow methods: `GET`, `POST`, `PUT`, `PATCH`, `DELETE`, `OPTIONS`

**Dependencies**: P1-T6

**Acceptance criteria**:
- Frontend can make API requests without CORS errors
- `OPTIONS` preflight requests return correct CORS headers
- `X-API-Key` header is allowed in CORS config
- CORS origin can be configured via environment variable

**Complexity**: S

---

### P8-T2: Add production build scripts

**Description**: Configure production build commands for both frontend and API.

**Files to modify**:
- `frontend/package.json` — ensure `build` script runs `vite build`, output to `dist/`
- `api/package.json` — ensure `build` script compiles TypeScript or bundles for deployment
- `package.json` (root) — add scripts:
  - `build` — builds both frontend and API
  - `build:frontend` — builds frontend only
  - `build:api` — builds API only

**Dependencies**: P1-T1, P1-T2, P1-T6

**Acceptance criteria**:
- `npm run build` from root builds both packages without errors
- Frontend build produces static files in `frontend/dist/`
- API build produces deployable output
- No TypeScript errors in either package

**Complexity**: S

---

### P8-T3: Create environment files and documentation

**Description**: Create `.env.example` files for both packages and document all environment variables.

**Files to create/modify**:
- `frontend/.env.example`:
  ```
  VITE_APP_URL=https://your-domain.com
  VITE_API_URL=https://your-api-domain.com
  ```
- `api/.env.example`:
  ```
  DATABASE_URL=postgresql://user:password@host:5432/brewlog
  API_KEY=your-secret-api-key
  ```
- `.env.example` (root) — reference both package env files

**Dependencies**: P1-T1

**Acceptance criteria**:
- `.env.example` files exist in both `frontend/` and `api/` directories
- All required environment variables are documented with placeholder values
- Comments explain each variable's purpose

**Complexity**: S

---

### P8-T4: Write README with setup, development, and deployment guide

**Description**: Create a comprehensive README documenting how to set up, develop, and deploy BrewLog.

**Files to create**:
- `README.md` — comprehensive documentation:
  - Project overview and features
  - Tech stack summary
  - Prerequisites (Node.js, PostgreSQL)
  - Setup instructions:
    1. Clone repo
    2. Install dependencies (`npm install` from root)
    3. Set up environment variables (copy `.env.example` files)
    4. Run database migrations (`npx drizzle-kit push`)
    5. Start development servers (`npm run dev`)
  - Development workflow
  - Environment variable reference (table of all variables)
  - Deployment guide:
    - Frontend: Cloudflare Pages / Netlify / Vercel
    - API: Cloudflare Workers / Vercel Functions / Netlify Functions
    - Database: self-hosted PostgreSQL connection string
  - Architecture overview (link to ARCHITECTURE.md)
  - License

**Dependencies**: P8-T2, P8-T3

**Acceptance criteria**:
- README covers setup, development, and deployment
- All environment variables are documented
- Instructions are clear enough for someone unfamiliar with the project
- Links to architecture document are included

**Complexity**: M

---

## Task Summary

| Phase | Tasks | Description |
|-------|-------|-------------|
| **Phase 1** | P1-T1 through P1-T10 | Project scaffold, DB schema, auth, validation, API client |
| **Phase 2** | P2-T1 through P2-T12 | Brews API, CRUD UI, dashboard, forms, auth UX |
| **Phase 3** | P3-T1 through P3-T5 | Ingredients API, query hooks, form, table, route page |
| **Phase 4** | P4-T1 through P4-T6 | Events API, query hooks, form, timeline, chart, log page |
| **Phase 5** | P5-T1 through P5-T4 | QR code components, quick-log form, route pages |
| **Phase 6** | P6-T1 through P6-T5 | Status filter, search, polish, 404, toasts |
| **Phase 7** | P7-T1 through P7-T6 | PWA, IndexedDB cache, offline queue, sync, status UI |
| **Phase 8** | P8-T1 through P8-T4 | CORS, build scripts, env docs, README |

**Total: 47 tasks**

---

## Dependency Graph (Cross-Phase)

```
P1-T1 ──┬── P1-T2 ── P1-T3 ── P1-T4 ── P1-T5 ── P1-T10
         │                                  │
         └── P1-T6 ──┬── P1-T7             │
                      ├── P1-T8             │
                      └── P1-T9             │
                           │                │
                      P2-T1 ────────── P2-T3 ── P2-T7 ── P2-T8
                           │                │
                      P3-T1            P2-T2 ── P2-T5 ── P2-T11
                           │                │
                      P4-T1            P2-T6 ── P2-T7
                                            │
                                       P2-T4 ── P2-T9 ── P2-T10
                                            │         │
                                       P2-T12    P3-T5
                                                      │
                                                 P4-T6
                                                      │
                                                 P5-T2, P5-T4
                                                      │
                                                 P6-T1 ── P6-T2
                                                      │
                                                 P7-T1 ── P7-T2 ── P7-T3
                                                           │
                                                      P7-T4 ── P7-T5 ── P7-T6
                                                                          │
                                                                     P8-T1 ── P8-T4
```
