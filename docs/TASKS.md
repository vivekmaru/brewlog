# BrewLog — Granular Implementation Tasks

> Derived from [ARCHITECTURE.md](../ARCHITECTURE.md) and [IMPLEMENTATION_PHASES.md](./IMPLEMENTATION_PHASES.md).
> Each task is self-contained, ordered, and designed for an AI coding agent to execute in a single session.

---

## Phase 1 — Project Scaffold & Database Schema

### P1-T1: Initialize monorepo root and workspace configuration

**Description**: Create the monorepo root with npm workspaces, shared TypeScript config, and project-level configuration files.

**Files to create/modify**:
- `package.json` — root package with `workspaces: ["frontend", "api"]`
- `tsconfig.base.json` — shared TS config with strict mode, ES2022 target
- `.gitignore` — Node, Vite, Wrangler, `.env`, `node_modules`, `dist`
- `.env.example` — document all env vars (placeholder)

**Dependencies**: None

**Acceptance Criteria**:
- `npm install` at root succeeds (even with empty workspace dirs)
- `tsconfig.base.json` exists with `strict: true`, `esModuleInterop: true`
- `.gitignore` covers `node_modules/`, `dist/`, `.env`, `.wrangler/`

**Complexity**: S

---

### P1-T2: Scaffold frontend with Vite + React 19 + TypeScript

**Description**: Initialize the `frontend/` workspace with Vite, React 19, TypeScript, and basic entry files.

**Files to create/modify**:
- `frontend/package.json` — dependencies: `react@19`, `react-dom@19`, `typescript`, `vite`, `@vitejs/plugin-react`
- `frontend/tsconfig.json` — extends `../tsconfig.base.json`, includes `src`
- `frontend/vite.config.ts` — React plugin, dev server on port 5173
- `frontend/index.html` — HTML entry with `<div id="root">`
- `frontend/src/main.tsx` — React root render
- `frontend/src/App.tsx` — placeholder component with "BrewLog" heading

**Dependencies**: P1-T1

**Acceptance Criteria**:
- `cd frontend && npm run dev` starts dev server at `localhost:5173`
- Browser shows a placeholder "BrewLog" page
- TypeScript compilation succeeds with no errors

**Complexity**: S

---

### P1-T3: Configure Tailwind CSS and shadcn/ui in frontend

**Description**: Add Tailwind CSS v4 and initialize shadcn/ui with base components.

**Files to create/modify**:
- `frontend/package.json` — add `tailwindcss`, `postcss`, `autoprefixer`, `@tailwindcss/vite` (or postcss plugin), `class-variance-authority`, `clsx`, `tailwind-merge`, `lucide-react`
- `frontend/src/index.css` — Tailwind directives (`@tailwind base; @tailwind components; @tailwind utilities;`)
- `frontend/tailwind.config.ts` — content paths, theme extensions
- `frontend/postcss.config.js` — Tailwind + autoprefixer plugins
- `frontend/src/lib/utils.ts` — `cn()` utility using `clsx` + `tailwind-merge`
- `frontend/components.json` — shadcn/ui config file

**Dependencies**: P1-T2

**Acceptance Criteria**:
- Tailwind utility classes render correctly in the browser
- `cn()` utility merges class names properly
- shadcn/ui is initialized and ready for component additions (e.g., `npx shadcn@latest add button` works)

**Complexity**: S

---

### P1-T4: Set up React Router v7 with placeholder routes

**Description**: Install React Router v7 and define the route structure with placeholder pages.

**Files to create/modify**:
- `frontend/package.json` — add `react-router`
- `frontend/src/App.tsx` — configure `BrowserRouter` with route definitions
- `frontend/src/routes/root.tsx` — root layout with `<Outlet />`
- `frontend/src/routes/dashboard.tsx` — placeholder "Dashboard" page
- `frontend/src/routes/not-found.tsx` — placeholder 404 page

**Dependencies**: P1-T3

**Acceptance Criteria**:
- `/` renders the dashboard placeholder
- Unknown routes render the 404 page
- Root layout wraps all routes with an `<Outlet />`

**Complexity**: S

---

### P1-T5: Wire up TanStack Query provider and Zustand store skeleton

**Description**: Install TanStack Query and Zustand, create the query client provider and an initial Zustand store.

**Files to create/modify**:
- `frontend/package.json` — add `@tanstack/react-query`, `zustand`
- `frontend/src/App.tsx` — wrap routes with `<QueryClientProvider>`
- `frontend/src/lib/store.ts` — Zustand store with `apiKey: string | null`, `isAuthenticated: boolean`, `setApiKey()`, `clearApiKey()`
- `frontend/src/lib/types.ts` — TypeScript types: `Brew`, `BrewStatus`, `BrewType`, `Ingredient`, `IngredientCategory`, `IngredientUnit`, `FermentationEvent`, `EventType`

**Dependencies**: P1-T4

**Acceptance Criteria**:
- `QueryClientProvider` wraps the app (visible in React DevTools or no errors in console)
- Zustand store can be imported and used; `setApiKey('test')` updates state
- All TypeScript types are defined and exported from `types.ts`

**Complexity**: S

---

### P1-T6: Scaffold API with Hono and health-check endpoint

**Description**: Initialize the `api/` workspace with Hono (lightweight serverless router), TypeScript, and a health-check endpoint.

**Files to create/modify**:
- `api/package.json` — dependencies: `hono`, `typescript`, `wrangler` (dev), scripts for `dev` and `build`
- `api/tsconfig.json` — extends `../tsconfig.base.json`
- `api/src/index.ts` — Hono app with `GET /api/health` returning `{ status: "ok" }`
- `api/wrangler.toml` — Cloudflare Workers config with `[vars]` section

**Dependencies**: P1-T1

**Acceptance Criteria**:
- `cd api && npm run dev` starts the dev server (wrangler or node)
- `GET /api/health` returns `200` with `{ "status": "ok" }`
- TypeScript compiles without errors

**Complexity**: S

---

### P1-T7: Define Drizzle ORM schema for all tables

**Description**: Create the Drizzle schema file defining `brews`, `ingredients`, and `fermentation_events` tables with all columns, indexes, and relations per the architecture.

**Files to create/modify**:
- `api/package.json` — add `drizzle-orm`, `postgres` (postgres-js driver), `nanoid`, `drizzle-kit` (dev)
- `api/src/db/schema.ts` — Drizzle table definitions for `brews`, `ingredients`, `fermentation_events` with:
  - All columns per architecture (text IDs with nanoid, user_id defaults, timestamptz, etc.)
  - Foreign keys: `ingredients.brew_id` → `brews.id`, `fermentation_events.brew_id` → `brews.id`
  - Indexes on `ingredients.brew_id` and `fermentation_events.brew_id`
  - Drizzle relations defined
- `api/src/db/index.ts` — Drizzle client creation using `postgres` driver and `DATABASE_URL` env var
- `api/drizzle.config.ts` — Drizzle Kit config pointing to schema and connection string

**Dependencies**: P1-T6

**Acceptance Criteria**:
- Schema file exports `brews`, `ingredients`, `fermentationEvents` tables
- All columns match the architecture document exactly (types, defaults, constraints)
- Foreign keys and indexes are defined
- `drizzle-kit generate` produces a migration SQL file without errors

**Complexity**: M

---

### P1-T8: Generate and apply first database migration

**Description**: Generate the initial Drizzle migration and verify it creates all tables in PostgreSQL.

**Files to create/modify**:
- `api/src/db/migrations/` — generated migration files (via `drizzle-kit generate`)
- `api/package.json` — add scripts: `db:generate`, `db:migrate`, `db:studio`

**Dependencies**: P1-T7

**Acceptance Criteria**:
- `npm run db:generate` creates migration files in `api/src/db/migrations/`
- `npm run db:migrate` applies the migration to a running PostgreSQL instance
- All three tables (`brews`, `ingredients`, `fermentation_events`) exist with correct columns
- `drizzle-kit studio` shows the schema visually

**Complexity**: S

---

### P1-T9: Implement API key auth middleware

**Description**: Create middleware that validates the `X-API-Key` header on all mutating requests (POST, PUT, PATCH, DELETE) and passes through GET requests.

**Files to create/modify**:
- `api/src/middleware/auth.ts` — Hono middleware that:
  - Reads `API_KEY` from environment variables
  - On `GET` requests: calls `next()` (no auth required)
  - On mutating requests: checks `X-API-Key` header against `API_KEY`
  - Returns `401 Unauthorized` with JSON error if missing/invalid
- `api/src/index.ts` — apply auth middleware to all `/api/*` routes

**Dependencies**: P1-T6

**Acceptance Criteria**:
- `GET /api/health` returns `200` without any API key
- `POST /api/health-auth-test` without `X-API-Key` returns `401`
- `POST /api/health-auth-test` with correct `X-API-Key` returns `200`
- `POST` with incorrect key returns `401`

**Complexity**: S

---

### P1-T10: Create Zod validation schemas

**Description**: Define Zod schemas for all API request payloads (brew, ingredient, event) that will be used for both server-side and client-side validation.

**Files to create/modify**:
- `api/src/validators.ts` — Zod schemas:
  - `createBrewSchema` — name (required), type (enum: beer/mead/cider), batch_size_liters (positive number), target_og (optional), target_fg (optional), status (enum, default fermenting), start_date (ISO string), end_date (optional), notes (optional)
  - `updateBrewSchema` — partial version of create schema
  - `updateBrewStatusSchema` — status enum only
  - `createIngredientSchema` — category (enum), name, quantity (positive), unit (enum: g/kg/ml/L), date_added (optional), notes (optional)
  - `updateIngredientSchema` — partial version
  - `createEventSchema` — event_type (enum), gravity (optional), temperature_celsius (optional), notes (optional), event_date (ISO string)

**Dependencies**: P1-T6

**Acceptance Criteria**:
- All schemas export correctly and can be imported
- Valid payloads pass validation
- Invalid payloads (missing required fields, wrong types, invalid enums) are rejected with descriptive errors
- Schemas match the database column definitions exactly

**Complexity**: S

---

### P1-T11: Build frontend API client with auth header

**Description**: Create a fetch wrapper that automatically attaches the API key from localStorage and targets the configured API URL.

**Files to create/modify**:
- `frontend/src/lib/api-client.ts` — fetch wrapper:
  - Reads `VITE_API_URL` from `import.meta.env`
  - Reads API key from `localStorage` (key: `brewlog-api-key`)
  - Attaches `X-API-Key` header on all requests when key is present
  - Sets `Content-Type: application/json` for POST/PUT/PATCH
  - Exports typed helper functions: `apiGet<T>(path)`, `apiPost<T>(path, body)`, `apiPut<T>(path, body)`, `apiPatch<T>(path, body)`, `apiDelete(path)`
  - Throws on non-2xx responses with parsed error message
- `frontend/.env` — `VITE_API_URL=http://localhost:8787` (or appropriate dev port)
- `frontend/.env.example` — document `VITE_API_URL` and `VITE_APP_URL`

**Dependencies**: P1-T5

**Acceptance Criteria**:
- `apiGet('/brews')` makes a GET request to `${VITE_API_URL}/api/brews`
- Requests include `X-API-Key` header when key exists in localStorage
- Requests omit `X-API-Key` header when no key is stored
- Non-2xx responses throw an error with the response message
- POST/PUT/PATCH requests include `Content-Type: application/json`

**Complexity**: S

---

## Phase 2 — Brews API & Core Brew UI

### P2-T1: Implement Brews API routes (CRUD)

**Description**: Create all brew CRUD endpoints using Hono, Drizzle ORM, and Zod validation.

**Files to create/modify**:
- `api/src/routes/brews.ts` — route handlers:
  - `GET /api/brews` — list all brews ordered by `updated_at` desc
  - `GET /api/brews/:brewId` — get single brew with ingredient count and event count
  - `POST /api/brews` — create brew (validate with `createBrewSchema`, generate nanoid)
  - `PUT /api/brews/:brewId` — update brew (validate with `updateBrewSchema`, update `updated_at`)
  - `PATCH /api/brews/:brewId/status` — update status only (validate with `updateBrewStatusSchema`)
  - `DELETE /api/brews/:brewId` — delete brew and cascade (delete related ingredients and events)
- `api/src/index.ts` — register brew routes

**Dependencies**: P1-T7, P1-T8, P1-T9, P1-T10

**Acceptance Criteria**:
- `POST /api/brews` with valid body creates a brew and returns it with `201`
- `GET /api/brews` returns an array of brews ordered by `updated_at` desc
- `GET /api/brews/:id` returns the brew with `ingredientCount` and `eventCount`
- `PUT /api/brews/:id` updates fields and returns the updated brew
- `PATCH /api/brews/:id/status` changes only the status field
- `DELETE /api/brews/:id` removes the brew and returns `204`
- Invalid payloads return `400` with Zod error details
- Non-existent brew IDs return `404`

**Complexity**: M

---

### P2-T2: Create frontend TypeScript types and utility functions

**Description**: Define all shared types and utility functions needed for the brew UI.

**Files to create/modify**:
- `frontend/src/lib/types.ts` — ensure all types are complete:
  - `Brew` (all fields from schema including `ingredientCount`, `eventCount`)
  - `BrewStatus` union type: `'planning' | 'fermenting' | 'conditioning' | 'bottled' | 'done'`
  - `BrewType` union type: `'beer' | 'mead' | 'cider'`
  - `ApiError` type for error responses
- `frontend/src/lib/utils.ts` — utility functions:
  - `calculateAbv(og: number, fg: number): number` — ABV formula: `(og - fg) * 131.25`
  - `formatDate(dateString: string): string` — human-readable date using `date-fns`
  - `getBrewAge(startDate: string): string` — "X days" since start
- `frontend/package.json` — add `date-fns`

**Dependencies**: P1-T5

**Acceptance Criteria**:
- All types are exported and usable in components
- `calculateAbv(1.050, 1.010)` returns approximately `5.25`
- `formatDate` returns a readable date string
- `getBrewAge` returns a human-readable duration

**Complexity**: S

---

### P2-T3: Create TanStack Query hooks for brews

**Description**: Implement query and mutation hooks for all brew operations.

**Files to create/modify**:
- `frontend/src/lib/queries.ts` — query hooks:
  - `useBrews()` — fetches `GET /api/brews`, returns `Brew[]`
  - `useBrew(brewId: string)` — fetches `GET /api/brews/:brewId`, returns `Brew`
- `frontend/src/lib/mutations.ts` — mutation hooks:
  - `useCreateBrew()` — `POST /api/brews`, invalidates `['brews']` query on success
  - `useUpdateBrew()` — `PUT /api/brews/:brewId`, invalidates `['brews']` and `['brew', brewId]`
  - `useUpdateBrewStatus()` — `PATCH /api/brews/:brewId/status`, invalidates same
  - `useDeleteBrew()` — `DELETE /api/brews/:brewId`, invalidates `['brews']`

**Dependencies**: P1-T11, P2-T2

**Acceptance Criteria**:
- `useBrews()` returns `{ data, isLoading, error }` with brew list
- `useBrew(id)` returns a single brew
- `useCreateBrew().mutate(data)` creates a brew and refetches the list
- All mutations invalidate the appropriate query keys on success
- Loading and error states are properly exposed

**Complexity**: S

---

### P2-T4: Build RootLayout and AppNav components

**Description**: Create the root layout with navigation bar that wraps all pages.

**Files to create/modify**:
- `frontend/src/layouts/root-layout.tsx` — root layout with `<AppNav />` and `<Outlet />`
- `frontend/src/components/app-nav.tsx` — top navigation bar:
  - Logo/title "BrewLog" linking to `/`
  - Navigation link to dashboard
  - Responsive (hamburger on mobile or simple layout)
- `frontend/src/routes/root.tsx` — update to use `RootLayout`
- Add shadcn/ui components as needed: `npx shadcn@latest add button` (or relevant primitives)

**Dependencies**: P1-T3, P1-T4

**Acceptance Criteria**:
- All pages render inside the root layout with the nav bar visible
- "BrewLog" logo/title links to `/`
- Navigation is visible on both mobile and desktop
- Layout uses Tailwind for styling

**Complexity**: S

---

### P2-T5: Build BrewForm component (create/edit)

**Description**: Create a reusable form component for creating and editing brews with Zod validation.

**Files to create/modify**:
- `frontend/src/components/forms/brew-form.tsx` — form component:
  - Props: `defaultValues?: Partial<Brew>`, `onSubmit: (data) => void`, `isLoading: boolean`
  - Fields: name (text), type (select: beer/mead/cider), batch_size_liters (number), target_og (number, optional), target_fg (number, optional), status (select), start_date (date), end_date (date, optional), notes (textarea)
  - Client-side validation using Zod schema
  - All units labeled in metric (L for batch size)
- `frontend/src/lib/validators.ts` — copy/adapt Zod schemas from API for client-side use
- Add shadcn/ui components: `input`, `select`, `textarea`, `label`, `card`

**Dependencies**: P2-T4

**Acceptance Criteria**:
- Form renders all fields with correct input types
- Validation errors display inline on invalid submission
- Form accepts `defaultValues` for edit mode and pre-fills fields
- `onSubmit` is called with validated data
- Batch size is labeled "Batch Size (L)"

**Complexity**: M

---

### P2-T6: Build BrewCard, BrewStatusBadge, and BrewTypeIcon components

**Description**: Create the brew card for the dashboard list and supporting display components.

**Files to create/modify**:
- `frontend/src/components/brew-card.tsx` — card displaying: brew name, type icon, status badge, batch size (L), start date, age
  - Clickable — navigates to `/brews/:brewId`
- `frontend/src/components/brew-status-badge.tsx` — colored badge per status:
  - `planning` → gray, `fermenting` → amber, `conditioning` → blue, `bottled` → purple, `done` → green
- `frontend/src/components/brew-type-icon.tsx` — icon per brew type using `lucide-react`:
  - `beer` → Beer icon, `mead` → Wine/Droplet icon, `cider` → Apple icon
- Add shadcn/ui components: `badge`, `card`

**Dependencies**: P2-T2, P2-T4

**Acceptance Criteria**:
- `BrewCard` renders brew info and navigates on click
- `BrewStatusBadge` shows correct color per status
- `BrewTypeIcon` shows correct icon per type
- Components use Tailwind and shadcn/ui primitives

**Complexity**: S

---

### P2-T7: Build Dashboard page with brew list

**Description**: Implement the dashboard route that fetches and displays all brews.

**Files to create/modify**:
- `frontend/src/routes/dashboard.tsx` — dashboard page:
  - Uses `useBrews()` hook to fetch brews
  - Renders a grid of `BrewCard` components
  - Shows loading state while fetching
  - Shows empty state with "Create your first brew" CTA
  - "New Brew" button linking to `/brews/new`
- `frontend/src/components/brew-dashboard.tsx` — extracted dashboard content component (optional, can be inline in route)

**Dependencies**: P2-T3, P2-T6

**Acceptance Criteria**:
- Dashboard fetches and displays all brews as cards
- Brews are ordered by `updated_at` desc
- Loading state shows while data is being fetched
- Empty state shows when no brews exist
- "New Brew" button navigates to `/brews/new`

**Complexity**: S

---

### P2-T8: Build Create Brew page

**Description**: Implement the new brew route with the brew form and create mutation.

**Files to create/modify**:
- `frontend/src/routes/brews/new.tsx` — create brew page:
  - Renders `BrewForm` with no default values
  - On submit: calls `useCreateBrew().mutate(data)`
  - On success: navigates to `/brews/:newBrewId`
  - Shows error toast on failure
- `frontend/src/App.tsx` — add route for `/brews/new`
- Add shadcn/ui `toast` / `sonner` for notifications

**Dependencies**: P2-T5, P2-T3

**Acceptance Criteria**:
- Form submits and creates a brew in the database
- On success, redirects to the new brew's detail page
- On error, shows a toast notification with the error message
- Form validation prevents submission of invalid data

**Complexity**: S

---

### P2-T9: Build Brew Detail Layout with tab navigation

**Description**: Create the brew detail layout with tabs for Overview, Ingredients, Log, and QR.

**Files to create/modify**:
- `frontend/src/layouts/brew-detail-layout.tsx` — layout component:
  - Fetches brew via `useBrew(brewId)` from URL params
  - Renders brew name and type as header
  - Tab navigation: Overview, Ingredients, Log, QR
  - Tabs link to sub-routes: `/brews/:brewId`, `/brews/:brewId/ingredients`, `/brews/:brewId/log`, `/brews/:brewId/qr`
  - `<Outlet />` for tab content
- `frontend/src/App.tsx` — add nested routes under `/brews/:brewId`
- Add shadcn/ui `tabs` component

**Dependencies**: P2-T4

**Acceptance Criteria**:
- Navigating to `/brews/:brewId` shows the detail layout with tabs
- Clicking tabs navigates to the correct sub-route
- Active tab is visually highlighted
- Brew name displays in the header
- `<Outlet />` renders the active tab's content

**Complexity**: M

---

### P2-T10: Build Brew Overview (summary) page

**Description**: Implement the brew detail index page showing brew summary information.

**Files to create/modify**:
- `frontend/src/routes/brews/$brewId/index.tsx` — brew overview page:
  - Displays: name, type, batch size (L), OG/FG targets, current status, start/end dates, notes, age
  - Shows calculated ABV if both OG and FG are available
  - Edit button linking to `/brews/:brewId/edit` (visible when authenticated)
  - Delete button with confirmation dialog (visible when authenticated)
- `frontend/src/components/brew-summary.tsx` — reusable summary display component
- Add shadcn/ui `dialog` (for delete confirmation)

**Dependencies**: P2-T9, P2-T3

**Acceptance Criteria**:
- Brew overview displays all brew metadata
- ABV is calculated and shown when OG and FG are present
- Edit button navigates to edit page
- Delete button shows confirmation dialog, then deletes and redirects to dashboard
- Unauthenticated users see read-only view (no edit/delete buttons)

**Complexity**: M

---

### P2-T11: Build Edit Brew page

**Description**: Implement the brew edit route with pre-filled form and update mutation.

**Files to create/modify**:
- `frontend/src/routes/brews/$brewId/edit.tsx` — edit brew page:
  - Fetches current brew data via `useBrew(brewId)`
  - Renders `BrewForm` with `defaultValues` from current brew
  - On submit: calls `useUpdateBrew().mutate(data)`
  - On success: navigates back to brew overview
  - Shows error toast on failure

**Dependencies**: P2-T5, P2-T3, P2-T9

**Acceptance Criteria**:
- Form pre-fills with current brew values
- Submitting updates the brew in the database
- On success, redirects to brew overview with updated data
- Status can be changed via the form or a separate status selector

**Complexity**: S

---

### P2-T12: Implement API key unlock prompt

**Description**: Build a simple unlock UI that prompts the user to enter their API key on first visit, storing it in localStorage.

**Files to create/modify**:
- `frontend/src/components/api-key-prompt.tsx` — dialog/modal component:
  - Text input for API key
  - "Unlock" button that saves to localStorage (`brewlog-api-key`) and updates Zustand store
  - "Continue as viewer" option to dismiss without key
- `frontend/src/layouts/root-layout.tsx` — check for API key on mount; show prompt if missing
- `frontend/src/lib/store.ts` — add `checkAuth()` that reads from localStorage on init

**Dependencies**: P1-T5, P2-T4

**Acceptance Criteria**:
- On first visit (no key in localStorage), the unlock prompt appears
- Entering a key stores it in localStorage and Zustand
- Subsequent visits auto-load the key from localStorage
- "Continue as viewer" dismisses the prompt
- Authenticated state is reflected in the UI (edit/delete buttons visible)

**Complexity**: S

---

## Phase 3 — Ingredients API & UI

### P3-T1: Implement Ingredients API routes

**Description**: Create all ingredient CRUD endpoints nested under brews.

**Files to create/modify**:
- `api/src/routes/ingredients.ts` — route handlers:
  - `GET /api/brews/:brewId/ingredients` — list ingredients for a brew, ordered by `created_at`
  - `POST /api/brews/:brewId/ingredients` — add ingredient (validate with `createIngredientSchema`, generate nanoid)
  - `PUT /api/brews/:brewId/ingredients/:id` — update ingredient (validate with `updateIngredientSchema`)
  - `DELETE /api/brews/:brewId/ingredients/:id` — delete ingredient
- `api/src/index.ts` — register ingredient routes

**Dependencies**: P1-T7, P1-T8, P1-T9, P1-T10

**Acceptance Criteria**:
- `POST` creates an ingredient linked to the brew and returns `201`
- `GET` returns all ingredients for the specified brew
- `PUT` updates the ingredient and returns the updated record
- `DELETE` removes the ingredient and returns `204`
- Returns `404` if brew or ingredient doesn't exist
- Validates that unit is one of `g`, `kg`, `ml`, `L`

**Complexity**: M

---

### P3-T2: Create TanStack Query hooks for ingredients

**Description**: Implement query and mutation hooks for ingredient operations.

**Files to create/modify**:
- `frontend/src/lib/queries.ts` — add:
  - `useIngredients(brewId: string)` — fetches `GET /api/brews/:brewId/ingredients`
- `frontend/src/lib/mutations.ts` — add:
  - `useAddIngredient()` — `POST /api/brews/:brewId/ingredients`, invalidates `['ingredients', brewId]`
  - `useUpdateIngredient()` — `PUT /api/brews/:brewId/ingredients/:id`, invalidates same
  - `useDeleteIngredient()` — `DELETE /api/brews/:brewId/ingredients/:id`, invalidates same

**Dependencies**: P1-T11, P2-T2

**Acceptance Criteria**:
- `useIngredients(brewId)` returns ingredient list for a brew
- All mutations invalidate the ingredients query on success
- Hooks handle loading and error states

**Complexity**: S

---

### P3-T3: Build IngredientForm component

**Description**: Create the ingredient add/edit form with category selector and metric unit picker.

**Files to create/modify**:
- `frontend/src/components/forms/ingredient-form.tsx` — form component:
  - Props: `brewId: string`, `defaultValues?: Partial<Ingredient>`, `onSubmit`, `onCancel`, `isLoading`
  - Fields: category (select: grain/hop/honey/fruit/yeast/adjunct/other), name (text), quantity (number), unit (select: g/kg/ml/L), date_added (date, optional), notes (textarea, optional)
  - Zod validation
- `frontend/src/lib/validators.ts` — add ingredient Zod schema for client-side

**Dependencies**: P2-T5 (pattern reference)

**Acceptance Criteria**:
- Form renders all fields with correct input types
- Category selector shows all 7 categories
- Unit selector shows only metric units: g, kg, ml, L
- Validation errors display inline
- Form supports both create (empty) and edit (pre-filled) modes

**Complexity**: S

---

### P3-T4: Build IngredientTable and IngredientRow components

**Description**: Create the ingredient list display with edit and delete actions.

**Files to create/modify**:
- `frontend/src/components/ingredient-table.tsx` — table component:
  - Columns: Category, Name, Quantity, Unit, Date Added, Notes, Actions
  - Renders `IngredientRow` for each ingredient
  - Empty state message when no ingredients
- `frontend/src/components/ingredient-row.tsx` — row component:
  - Displays ingredient data
  - Edit button (toggles inline edit form or opens modal)
  - Delete button with confirmation
  - Uses `useUpdateIngredient` and `useDeleteIngredient` mutations
- Add shadcn/ui `table` component

**Dependencies**: P3-T2, P3-T3

**Acceptance Criteria**:
- Table displays all ingredients for a brew
- Edit action allows modifying an ingredient
- Delete action shows confirmation and removes the ingredient
- Empty state shows when no ingredients exist

**Complexity**: M

---

### P3-T5: Build Ingredients route page and wire up tab

**Description**: Create the ingredients route page and connect it to the brew detail tab navigation.

**Files to create/modify**:
- `frontend/src/routes/brews/$brewId/ingredients.tsx` — ingredients page:
  - Uses `useIngredients(brewId)` to fetch ingredients
  - Renders `IngredientTable` with the data
  - Renders `IngredientForm` for adding new ingredients (collapsible or always visible)
  - Loading and error states
- `frontend/src/layouts/brew-detail-layout.tsx` — ensure Ingredients tab route is active
- `frontend/src/App.tsx` — ensure `/brews/:brewId/ingredients` route is registered

**Dependencies**: P3-T4, P2-T9

**Acceptance Criteria**:
- Navigating to `/brews/:brewId/ingredients` shows the ingredient list
- Adding an ingredient via the form adds it to the table
- Ingredients tab is active in the brew detail layout
- Page handles loading and error states gracefully

**Complexity**: S

---

## Phase 4 — Fermentation Events API & UI

### P4-T1: Implement Fermentation Events API routes

**Description**: Create event CRUD endpoints nested under brews.

**Files to create/modify**:
- `api/src/routes/events.ts` — route handlers:
  - `GET /api/brews/:brewId/events` — list events ordered by `event_date` desc
  - `POST /api/brews/:brewId/events` — create event (validate with `createEventSchema`, generate nanoid)
  - `DELETE /api/brews/:brewId/events/:id` — delete event
- `api/src/index.ts` — register event routes

**Dependencies**: P1-T7, P1-T8, P1-T9, P1-T10

**Acceptance Criteria**:
- `POST` creates an event linked to the brew and returns `201`
- `GET` returns all events for the brew ordered by `event_date` desc
- `DELETE` removes the event and returns `204`
- Returns `404` if brew or event doesn't exist
- Validates `event_type` is one of the allowed enum values

**Complexity**: M

---

### P4-T2: Create TanStack Query hooks for events

**Description**: Implement query and mutation hooks for fermentation event operations.

**Files to create/modify**:
- `frontend/src/lib/queries.ts` — add:
  - `useEvents(brewId: string)` — fetches `GET /api/brews/:brewId/events`
- `frontend/src/lib/mutations.ts` — add:
  - `useAddEvent()` — `POST /api/brews/:brewId/events`, invalidates `['events', brewId]` and `['brew', brewId]` (for event count)
  - `useDeleteEvent()` — `DELETE /api/brews/:brewId/events/:id`, invalidates same

**Dependencies**: P1-T11, P2-T2

**Acceptance Criteria**:
- `useEvents(brewId)` returns event list for a brew
- Mutations invalidate both events and brew queries (for count updates)
- Hooks handle loading and error states

**Complexity**: S

---

### P4-T3: Build EventForm component with dynamic fields

**Description**: Create the event form with event type selector that shows different fields based on the selected type.

**Files to create/modify**:
- `frontend/src/components/forms/event-form.tsx` — form component:
  - Event type selector: `gravity_reading`, `temperature`, `racking`, `addition`, `tasting`, `note`, `bottling`, `other`
  - Dynamic fields based on type:
    - `gravity_reading` → gravity input (number), temperature (optional), notes
    - `temperature` → temperature input (°C), notes
    - `racking` → notes textarea
    - `addition` → notes textarea
    - `tasting` → notes textarea
    - `note` → notes textarea
    - `bottling` → notes textarea
    - `other` → notes textarea
  - `event_date` field defaulting to current datetime
  - Zod validation
- `frontend/src/lib/validators.ts` — add event Zod schema for client-side

**Dependencies**: P2-T5 (pattern reference)

**Acceptance Criteria**:
- Selecting an event type shows the appropriate fields
- Gravity reading type shows gravity and optional temperature inputs
- Temperature type shows temperature input labeled "°C"
- Date defaults to now but is editable
- Validation prevents submission without required fields

**Complexity**: M

---

### P4-T4: Build EventTimeline, EventCard, and EventTypeIcon components

**Description**: Create the event timeline display components.

**Files to create/modify**:
- `frontend/src/components/event-timeline.tsx` — timeline component:
  - Renders events in chronological order (newest first)
  - Visual timeline line connecting events
  - Empty state message
- `frontend/src/components/event-card.tsx` — card for a single event:
  - Shows event type icon, date, and relevant data (gravity, temperature, notes)
  - Delete button (visible when authenticated)
  - Uses `useDeleteEvent` mutation
- `frontend/src/components/event-type-icon.tsx` — icon per event type using `lucide-react`

**Dependencies**: P4-T2

**Acceptance Criteria**:
- Timeline displays events in reverse chronological order
- Each event card shows type icon, date, and relevant data
- Gravity readings show the gravity value
- Temperature readings show the value with °C
- Delete button removes the event with confirmation
- Empty state shows when no events exist

**Complexity**: M

---

### P4-T5: Build GravityChart component

**Description**: Create a line chart showing gravity readings over time using Recharts.

**Files to create/modify**:
- `frontend/package.json` — add `recharts`
- `frontend/src/components/gravity-chart.tsx` — chart component:
  - Props: `events: FermentationEvent[]`
  - Filters events to only `gravity_reading` type
  - X-axis: `event_date` (formatted dates)
  - Y-axis: `gravity` values
  - Responsive container
  - Shows "No gravity readings yet" if no data
  - Optional: target OG/FG reference lines if provided

**Dependencies**: P4-T2

**Acceptance Criteria**:
- Chart renders gravity readings as a line chart
- X-axis shows dates, Y-axis shows gravity values
- Chart is responsive and resizes with container
- Empty state shown when no gravity readings exist
- Chart only includes events of type `gravity_reading`

**Complexity**: M

---

### P4-T6: Build Fermentation Log route page and wire up tab

**Description**: Create the log route page combining the timeline, chart, and event form.

**Files to create/modify**:
- `frontend/src/routes/brews/$brewId/log.tsx` — log page:
  - Uses `useEvents(brewId)` to fetch events
  - Renders `GravityChart` at the top
  - Renders `EventForm` for adding new events (collapsible)
  - Renders `EventTimeline` below
  - Loading and error states
- `frontend/src/layouts/brew-detail-layout.tsx` — ensure Log tab route is active
- `frontend/src/App.tsx` — ensure `/brews/:brewId/log` route is registered

**Dependencies**: P4-T3, P4-T4, P4-T5, P2-T9

**Acceptance Criteria**:
- Navigating to `/brews/:brewId/log` shows the gravity chart, event form, and timeline
- Adding an event via the form adds it to the timeline
- Gravity chart updates when new gravity readings are added
- Log tab is active in the brew detail layout

**Complexity**: S

---

### P4-T7: Update BrewSummary with event data

**Description**: Enhance the brew summary component to show latest gravity, calculated ABV, and event count.

**Files to create/modify**:
- `frontend/src/components/brew-summary.tsx` — update to:
  - Show latest gravity reading (from events)
  - Calculate and display ABV using OG and latest gravity
  - Show total event count
  - Show total ingredient count
- `frontend/src/routes/brews/$brewId/index.tsx` — pass events data to summary (or fetch within summary)

**Dependencies**: P4-T2, P2-T10

**Acceptance Criteria**:
- Brew summary shows latest gravity reading
- ABV is calculated from target OG and latest gravity reading
- Event count and ingredient count are displayed
- Summary gracefully handles missing data (no gravity readings, no OG set)

**Complexity**: S

---

## Phase 5 — QR Codes & Quick-Log Flow

### P5-T1: Build QR Code display page and components

**Description**: Create the QR code page with display, download, and print functionality.

**Files to create/modify**:
- `frontend/package.json` — add `qrcode.react`
- `frontend/src/components/qr-code-display.tsx` — renders QR code via `qrcode.react`:
  - Encodes URL: `${VITE_APP_URL}/brews/${brewId}`
  - Renders as SVG
  - Large size for easy scanning
- `frontend/src/components/qr-download-button.tsx` — converts QR SVG to PNG and triggers download
- `frontend/src/components/qr-print-button.tsx` — opens print dialog with label layout:
  - Brew name, type, start date, and QR code
- `frontend/src/routes/brews/$brewId/qr.tsx` — QR page:
  - Displays brew name and type above QR
  - Renders `QrCodeDisplay`, `QrDownloadButton`, `QrPrintButton`
- `frontend/src/App.tsx` — ensure `/brews/:brewId/qr` route is registered

**Dependencies**: P2-T9

**Acceptance Criteria**:
- QR code page renders a scannable QR code encoding the correct URL
- Download button saves the QR as a PNG file
- Print button opens print dialog with a label layout
- QR tab is active in the brew detail layout
- `VITE_APP_URL` env var is used for the URL base

**Complexity**: M

---

### P5-T2: Build QuickLogForm component

**Description**: Create the mobile-optimized quick-log form with large tap targets.

**Files to create/modify**:
- `frontend/src/components/forms/quick-log-form.tsx` — mobile-first form:
  - Large event type selector buttons (tap targets)
  - Event types: Gravity, Temperature, Tasting, Racking, Addition, Note
  - Dynamic fields based on selected type (same logic as `EventForm` but mobile-optimized)
  - Date/time defaults to now, editable
  - Large submit button
  - Uses `useAddEvent` mutation
  - Success: toast notification, form resets, link to full log

**Dependencies**: P4-T3 (pattern reference), P4-T2

**Acceptance Criteria**:
- Event type buttons are large and easy to tap on mobile
- Selecting a type shows appropriate input fields
- Submitting creates an event and shows success toast
- Form resets after successful submission
- Link to full log is shown after submission

**Complexity**: M

---

### P5-T3: Build Quick-Log route page

**Description**: Create the quick-log route page that requires authentication.

**Files to create/modify**:
- `frontend/src/routes/quick-log/$brewId.tsx` — quick-log page:
  - Fetches brew via `useBrew(brewId)` for header display
  - Shows brew name and current status in header
  - Renders `QuickLogForm`
  - Requires authentication — shows unlock prompt or redirects if no API key
  - 404 if brew doesn't exist
- `frontend/src/App.tsx` — add route for `/quick-log/:brewId`

**Dependencies**: P5-T2, P2-T12

**Acceptance Criteria**:
- `/quick-log/:brewId` renders the quick-log form
- Brew name and status shown in header
- Unauthenticated users see the API key prompt
- Non-existent brew IDs show 404
- Form submission creates an event successfully

**Complexity**: S

---

### P5-T4: Add Quick Log button to brew detail page

**Description**: Add a "Quick Log" button on the brew detail page visible only to authenticated users.

**Files to create/modify**:
- `frontend/src/routes/brews/$brewId/index.tsx` — add "Quick Log" button:
  - Only visible when `isAuthenticated` is true (from Zustand store)
  - Links to `/quick-log/:brewId`
  - Styled prominently (primary button)
- `frontend/src/layouts/brew-detail-layout.tsx` — optionally add quick-log link in header area

**Dependencies**: P5-T3, P2-T10, P2-T12

**Acceptance Criteria**:
- Authenticated users see a "Quick Log" button on the brew detail page
- Button navigates to `/quick-log/:brewId`
- Unauthenticated users do not see the button
- Button is visually prominent

**Complexity**: S

---

## Phase 6 — Dashboard Polish & Filtering

### P6-T1: Build StatusFilter component

**Description**: Create a filter component for the dashboard that filters brews by status.

**Files to create/modify**:
- `frontend/src/components/status-filter.tsx` — filter component:
  - Shows all status options: All, Planning, Fermenting, Conditioning, Bottled, Done
  - Toggle/button group style
  - Active filter visually highlighted
  - Props: `activeStatus: string | null`, `onFilterChange: (status: string | null) => void`

**Dependencies**: P2-T6

**Acceptance Criteria**:
- All status options are displayed
- Clicking a status filters the list
- "All" shows all brews
- Active filter is visually distinct
- Component is responsive

**Complexity**: S

---

### P6-T2: Add search and filtering to Dashboard

**Description**: Enhance the dashboard with client-side search by name and status filtering.

**Files to create/modify**:
- `frontend/src/routes/dashboard.tsx` — update dashboard:
  - Add search input for filtering by brew name (client-side)
  - Integrate `StatusFilter` component
  - Filter brews based on both search text and selected status
  - Show result count
- `frontend/src/components/brew-dashboard.tsx` — if extracted, update here

**Dependencies**: P6-T1, P2-T7

**Acceptance Criteria**:
- Typing in search filters brews by name (case-insensitive)
- Status filter narrows results to selected status
- Both filters work together (AND logic)
- Result count updates as filters change
- Clearing filters shows all brews

**Complexity**: S

---

### P6-T3: Polish BrewCard with full data display

**Description**: Enhance the brew card to show all relevant data including latest gravity and age.

**Files to create/modify**:
- `frontend/src/components/brew-card.tsx` — update card:
  - Show: brew name, type icon, status badge, batch size (L), start date, brew age, latest gravity (if available)
  - Improved layout with consistent spacing
  - Hover state for interactivity

**Dependencies**: P2-T6, P4-T7

**Acceptance Criteria**:
- Card shows all specified data points
- Batch size displayed with "L" unit
- Age calculated from start date
- Latest gravity shown if available
- Card has hover effect and consistent styling

**Complexity**: S

---

### P6-T4: Build 404 page

**Description**: Create a proper 404 page with navigation back to the dashboard.

**Files to create/modify**:
- `frontend/src/routes/not-found.tsx` — 404 page:
  - Clear "Page Not Found" message
  - Link back to dashboard
  - Consistent styling with the rest of the app

**Dependencies**: P2-T4

**Acceptance Criteria**:
- Unknown routes render the 404 page
- Page has a clear message and link to dashboard
- Styling is consistent with the app theme

**Complexity**: S

---

### P6-T5: Add loading skeletons and error states

**Description**: Add loading skeleton placeholders and error boundary/states across all pages.

**Files to create/modify**:
- `frontend/src/components/ui/skeleton.tsx` — skeleton component (shadcn/ui)
- `frontend/src/components/brew-card-skeleton.tsx` — skeleton version of brew card
- `frontend/src/routes/dashboard.tsx` — add skeleton loading state
- `frontend/src/routes/brews/$brewId/index.tsx` — add loading/error states
- `frontend/src/routes/brews/$brewId/ingredients.tsx` — add loading/error states
- `frontend/src/routes/brews/$brewId/log.tsx` — add loading/error states

**Dependencies**: P2-T7, P3-T5, P4-T6

**Acceptance Criteria**:
- Dashboard shows skeleton cards while loading
- Brew detail pages show loading indicators
- Error states show user-friendly messages with retry option
- No raw error messages shown to users

**Complexity**: M

---

### P6-T6: Add toast notifications for all mutations

**Description**: Ensure all create, update, and delete operations show toast notifications.

**Files to create/modify**:
- `frontend/src/lib/mutations.ts` — add `onSuccess` and `onError` toast notifications to all mutations:
  - Create brew: "Brew created successfully"
  - Update brew: "Brew updated"
  - Delete brew: "Brew deleted"
  - Add ingredient: "Ingredient added"
  - Update ingredient: "Ingredient updated"
  - Delete ingredient: "Ingredient removed"
  - Add event: "Event logged"
  - Delete event: "Event removed"
- Ensure toast/sonner is properly configured in the root layout

**Dependencies**: P2-T8 (toast setup)

**Acceptance Criteria**:
- All mutations show success toasts on completion
- All mutations show error toasts on failure
- Toasts auto-dismiss after a few seconds
- Toast messages are descriptive and user-friendly

**Complexity**: S

---

### P6-T7: Responsive layout polish

**Description**: Ensure all pages are fully responsive and usable on mobile screens.

**Files to create/modify**:
- `frontend/src/layouts/root-layout.tsx` — responsive nav (mobile hamburger or simplified)
- `frontend/src/routes/dashboard.tsx` — responsive grid (1 col mobile, 2-3 cols desktop)
- `frontend/src/layouts/brew-detail-layout.tsx` — responsive tabs (scrollable on mobile)
- `frontend/src/components/ingredient-table.tsx` — responsive table (card layout on mobile or horizontal scroll)
- `frontend/src/components/event-timeline.tsx` — responsive timeline

**Dependencies**: P6-T5

**Acceptance Criteria**:
- Dashboard grid adapts from 1 column (mobile) to 3 columns (desktop)
- Navigation is usable on mobile screens
- Brew detail tabs are scrollable/accessible on mobile
- Ingredient table is readable on mobile
- Event timeline is readable on mobile
- No horizontal overflow on any page at 320px width

**Complexity**: M

---

## Phase 7 — Offline Support & PWA

### P7-T1: Configure vite-plugin-pwa for Service Worker

**Description**: Set up the PWA plugin to cache static assets for offline app shell loading.

**Files to create/modify**:
- `frontend/package.json` — add `vite-plugin-pwa`
- `frontend/vite.config.ts` — configure `VitePWA` plugin:
  - `registerType: 'autoUpdate'`
  - Cache static assets (HTML, JS, CSS, fonts)
  - Manifest with app name, icons, theme color
- `frontend/public/icons/` — add app icons (192x192, 512x512)
- `frontend/src/sw.ts` — Service Worker registration

**Dependencies**: P1-T2

**Acceptance Criteria**:
- Service Worker is registered on app load
- Static assets are cached by the Service Worker
- App shell loads when device is offline (after first visit)
- PWA manifest is generated with correct metadata

**Complexity**: M

---

### P7-T2: Implement IndexedDB cache for API responses

**Description**: Create an IndexedDB layer that caches API responses for offline access.

**Files to create/modify**:
- `frontend/package.json` — add `idb`
- `frontend/src/lib/offline.ts` — IndexedDB cache:
  - Database name: `brewlog-cache`
  - Stores: `brews`, `ingredients`, `events` (keyed by ID or composite key)
  - Functions: `cacheBrews(brews)`, `getCachedBrews()`, `cacheBrew(brew)`, `getCachedBrew(id)`, `cacheIngredients(brewId, ingredients)`, `getCachedIngredients(brewId)`, `cacheEvents(brewId, events)`, `getCachedEvents(brewId)`
  - Clear cache functions for invalidation

**Dependencies**: P1-T2

**Acceptance Criteria**:
- API responses can be stored in IndexedDB
- Cached data can be retrieved by key
- Cache can be cleared/invalidated
- Database schema is versioned for future migrations

**Complexity**: M

---

### P7-T3: Integrate IndexedDB cache with TanStack Query

**Description**: Wire up the IndexedDB cache as a fallback data source in TanStack Query hooks.

**Files to create/modify**:
- `frontend/src/lib/queries.ts` — update all query hooks:
  - On successful fetch: cache response in IndexedDB
  - On fetch error (offline): return cached data from IndexedDB
  - Configure `staleTime` and `gcTime` for offline-friendly caching
- `frontend/src/lib/api-client.ts` — detect offline state and throw specific error

**Dependencies**: P7-T2, P2-T3, P3-T2, P4-T2

**Acceptance Criteria**:
- Successful API responses are cached in IndexedDB
- When offline, cached data is returned instead of an error
- `staleTime` is set to a reasonable value (e.g., 5 minutes)
- `gcTime` is set to keep data available for offline use (e.g., 24 hours)
- On reconnection, stale queries are refetched

**Complexity**: M

---

### P7-T4: Implement offline write outbox queue

**Description**: Create an IndexedDB outbox that queues write operations when offline.

**Files to create/modify**:
- `frontend/src/lib/offline.ts` — add outbox functions:
  - `addToOutbox(request: { method, url, body, timestamp })` — queue a write operation
  - `getOutboxItems()` — retrieve all pending items ordered by timestamp
  - `removeFromOutbox(id)` — remove a synced item
  - `getOutboxCount()` — count of pending items
- `frontend/src/lib/api-client.ts` — update to detect offline and queue writes:
  - On POST/PUT/PATCH/DELETE when offline: save to outbox instead of throwing
  - Return a "pending" response so the UI can show optimistic state

**Dependencies**: P7-T2

**Acceptance Criteria**:
- Write operations are queued in IndexedDB when offline
- Outbox items include method, URL, body, and timestamp
- Items can be retrieved in order and removed after sync
- API client detects offline state and queues instead of failing

**Complexity**: M

---

### P7-T5: Implement background sync logic

**Description**: Create the sync orchestration that replays queued writes when connectivity is restored.

**Files to create/modify**:
- `frontend/src/lib/sync.ts` — sync orchestration:
  - `syncOutbox()` — replays all outbox items to the API in order
  - Handles failures (retry logic, skip permanently failed items)
  - Clears synced items from outbox
  - Emits events/updates Zustand store on progress
- `frontend/src/lib/store.ts` — add sync state:
  - `pendingSyncCount: number`
  - `isSyncing: boolean`
  - `updateSyncStatus()`
- Wire up `online` event listener to trigger sync

**Dependencies**: P7-T4

**Acceptance Criteria**:
- When connectivity is restored, queued writes are replayed in order
- Successfully synced items are removed from the outbox
- Failed items are retried or flagged
- Zustand store reflects sync status (pending count, syncing flag)
- `online` event triggers sync automatically

**Complexity**: M

---

### P7-T6: Build SyncStatus UI component

**Description**: Create a component that shows the offline/sync status in the navigation bar.

**Files to create/modify**:
- `frontend/src/components/sync-status.tsx` — sync status component:
  - Shows online/offline indicator
  - Shows pending sync count when items are queued
  - Shows "Syncing..." during active sync
  - Toast notification on successful sync completion
- `frontend/src/layouts/root-layout.tsx` — add `SyncStatus` to the nav bar
- `frontend/src/lib/store.ts` — add `isOnline: boolean` state, listen to `online`/`offline` events

**Dependencies**: P7-T5, P2-T4

**Acceptance Criteria**:
- Online/offline status is visible in the nav bar
- Pending sync count shows when items are queued
- "Syncing..." indicator shows during active sync
- Toast notification appears when sync completes
- Status updates in real-time as connectivity changes

**Complexity**: S

---

## Phase 8 — Deployment & Production Readiness

### P8-T1: Configure API for production deployment

**Description**: Set up the API for deployment to Cloudflare Workers (or alternative serverless provider).

**Files to create/modify**:
- `api/wrangler.toml` — production configuration:
  - Name, compatibility date, routes
  - Secrets: `DATABASE_URL`, `API_KEY`
- `api/package.json` — add `build` and `deploy` scripts
- `api/src/index.ts` — ensure CORS middleware is configured:
  - Allow requests from frontend origin (`FRONTEND_URL` env var or `*` for development)
  - Handle preflight `OPTIONS` requests

**Dependencies**: P1-T6

**Acceptance Criteria**:
- `npm run build` produces a deployable bundle
- `npm run deploy` deploys to the serverless provider
- CORS headers are set correctly for the frontend origin
- `OPTIONS` preflight requests return correct headers
- Environment secrets are configured (not hardcoded)

**Complexity**: M

---

### P8-T2: Configure frontend for production deployment

**Description**: Set up the frontend for static deployment with production environment variables.

**Files to create/modify**:
- `frontend/vite.config.ts` — ensure production build settings are correct
- `frontend/package.json` — add `build` and `preview` scripts
- `frontend/.env.example` — document `VITE_APP_URL` and `VITE_API_URL`
- `frontend/public/_redirects` (Netlify) or equivalent — SPA redirect rule for client-side routing

**Dependencies**: P1-T2

**Acceptance Criteria**:
- `npm run build` produces optimized static files in `dist/`
- `npm run preview` serves the production build locally
- SPA routing works (all paths serve `index.html`)
- Environment variables are correctly embedded in the build

**Complexity**: S

---

### P8-T3: Write project README

**Description**: Create a comprehensive README with setup instructions, environment variable reference, and deployment guide.

**Files to create/modify**:
- `README.md` — documentation:
  - Project overview and features
  - Tech stack summary
  - Prerequisites (Node.js, PostgreSQL)
  - Local development setup (clone, install, env vars, DB setup, run dev)
  - Environment variable reference (all vars for frontend and API)
  - Database setup (migration commands)
  - Deployment guide (frontend static host + API serverless)
  - Project structure overview

**Dependencies**: None (can be done anytime, but best after all features)

**Acceptance Criteria**:
- README covers all setup steps from clone to running dev
- All environment variables are documented with descriptions
- Deployment steps are clear for at least one provider
- Project structure is explained

**Complexity**: M

---

### P8-T4: Create .env.example files for both packages

**Description**: Document all required environment variables with example values.

**Files to create/modify**:
- `frontend/.env.example`:
  ```
  VITE_APP_URL=http://localhost:5173
  VITE_API_URL=http://localhost:8787
  ```
- `api/.env.example`:
  ```
  DATABASE_URL=postgresql://user:password@localhost:5432/brewlog
  API_KEY=your-secret-api-key-here
  ```
- `.env.example` (root) — reference to package-level env files

**Dependencies**: None

**Acceptance Criteria**:
- All required environment variables are documented
- Example values are provided (not real secrets)
- Comments explain each variable's purpose

**Complexity**: S

---

### P8-T5: End-to-end production verification

**Description**: Verify the full application flow works in a production-like environment.

**Files to create/modify**:
- No new files — this is a verification task
- May need to fix issues discovered during testing in any file

**Dependencies**: P8-T1, P8-T2, P8-T3, P8-T4

**Acceptance Criteria**:
- Frontend loads at the production URL
- API responds to requests from the frontend (no CORS errors)
- Full brew CRUD works end-to-end
- Ingredient and event CRUD works
- QR codes encode the production URL
- Quick-log flow works on mobile
- Offline mode works (Service Worker caches, IndexedDB stores data)
- Auth flow works (API key prompt, stored key used for mutations)

**Complexity**: M

---

## Task Dependency Graph

```mermaid
flowchart TD
    P1T1[P1-T1: Monorepo root] --> P1T2[P1-T2: Frontend scaffold]
    P1T1 --> P1T6[P1-T6: API scaffold]
    P1T2 --> P1T3[P1-T3: Tailwind + shadcn]
    P1T3 --> P1T4[P1-T4: React Router]
    P1T4 --> P1T5[P1-T5: TanStack Query + Zustand]
    P1T5 --> P1T11[P1-T11: API client]
    P1T6 --> P1T7[P1-T7: Drizzle schema]
    P1T6 --> P1T9[P1-T9: Auth middleware]
    P1T6 --> P1T10[P1-T10: Zod schemas]
    P1T7 --> P1T8[P1-T8: DB migration]

    P1T3 --> P2T4[P2-T4: RootLayout + AppNav]
    P1T5 --> P2T2[P2-T2: Types + utils]
    P1T11 --> P2T3[P2-T3: Brew query hooks]
    P2T2 --> P2T3
    P1T8 --> P2T1[P2-T1: Brews API]
    P1T9 --> P2T1
    P1T10 --> P2T1
    P2T4 --> P2T5[P2-T5: BrewForm]
    P2T2 --> P2T6[P2-T6: BrewCard + badges]
    P2T4 --> P2T6
    P2T3 --> P2T7[P2-T7: Dashboard page]
    P2T6 --> P2T7
    P2T5 --> P2T8[P2-T8: Create Brew page]
    P2T3 --> P2T8
    P2T4 --> P2T9[P2-T9: Brew Detail Layout]
    P2T9 --> P2T10[P2-T10: Brew Overview page]
    P2T3 --> P2T10
    P2T5 --> P2T11[P2-T11: Edit Brew page]
    P2T9 --> P2T11
    P1T5 --> P2T12[P2-T12: API key prompt]
    P2T4 --> P2T12

    P1T8 --> P3T1[P3-T1: Ingredients API]
    P1T11 --> P3T2[P3-T2: Ingredient hooks]
    P3T2 --> P3T4[P3-T4: IngredientTable]
    P2T5 --> P3T3[P3-T3: IngredientForm]
    P3T3 --> P3T4
    P3T4 --> P3T5[P3-T5: Ingredients page]
    P2T9 --> P3T5

    P1T8 --> P4T1[P4-T1: Events API]
    P1T11 --> P4T2[P4-T2: Event hooks]
    P4T2 --> P4T4[P4-T4: EventTimeline]
    P4T2 --> P4T5[P4-T5: GravityChart]
    P2T5 --> P4T3[P4-T3: EventForm]
    P4T3 --> P4T6[P4-T6: Log page]
    P4T4 --> P4T6
    P4T5 --> P4T6
    P2T9 --> P4T6
    P4T2 --> P4T7[P4-T7: Update BrewSummary]
    P2T10 --> P4T7

    P2T9 --> P5T1[P5-T1: QR Code page]
    P4T2 --> P5T2[P5-T2: QuickLogForm]
    P5T2 --> P5T3[P5-T3: Quick-Log page]
    P2T12 --> P5T3
    P5T3 --> P5T4[P5-T4: Quick Log button]
    P2T10 --> P5T4

    P2T6 --> P6T1[P6-T1: StatusFilter]
    P6T1 --> P6T2[P6-T2: Dashboard search + filter]
    P2T7 --> P6T2
    P2T6 --> P6T3[P6-T3: Polish BrewCard]
    P2T4 --> P6T4[P6-T4: 404 page]
    P2T7 --> P6T5[P6-T5: Loading skeletons]
    P3T5 --> P6T5
    P4T6 --> P6T5
    P2T8 --> P6T6[P6-T6: Toast notifications]
    P6T5 --> P6T7[P6-T7: Responsive polish]

    P1T2 --> P7T1[P7-T1: PWA plugin]
    P1T2 --> P7T2[P7-T2: IndexedDB cache]
    P7T2 --> P7T3[P7-T3: Cache + TanStack Query]
    P2T3 --> P7T3
    P7T2 --> P7T4[P7-T4: Outbox queue]
    P7T4 --> P7T5[P7-T5: Background sync]
    P7T5 --> P7T6[P7-T6: SyncStatus UI]
    P2T4 --> P7T6

    P1T6 --> P8T1[P8-T1: API deployment]
    P1T2 --> P8T2[P8-T2: Frontend deployment]
    P8T1 --> P8T5[P8-T5: E2E verification]
    P8T2 --> P8T5
    P8T3[P8-T3: README]
    P8T4[P8-T4: .env.example files]
    P8T4 --> P8T5
```

---

## Summary

| Phase | Tasks | Complexity Breakdown |
|-------|-------|---------------------|
| Phase 1 — Project Scaffold & DB Schema | P1-T1 through P1-T11 (11 tasks) | 9S, 2M |
| Phase 2 — Brews API & Core Brew UI | P2-T1 through P2-T12 (12 tasks) | 7S, 5M |
| Phase 3 — Ingredients API & UI | P3-T1 through P3-T5 (5 tasks) | 3S, 2M |
| Phase 4 — Fermentation Events API & UI | P4-T1 through P4-T7 (7 tasks) | 2S, 5M |
| Phase 5 — QR Codes & Quick-Log | P5-T1 through P5-T4 (4 tasks) | 2S, 2M |
| Phase 6 — Dashboard Polish & Filtering | P6-T1 through P6-T7 (7 tasks) | 4S, 3M |
| Phase 7 — Offline Support & PWA | P7-T1 through P7-T6 (6 tasks) | 1S, 5M |
| Phase 8 — Deployment & Production | P8-T1 through P8-T5 (5 tasks) | 2S, 3M |
| **Total** | **57 tasks** | **30S, 27M** |
