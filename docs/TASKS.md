# BrewLog — Implementation Tasks

> Granular, ordered task breakdown derived from [IMPLEMENTATION_PHASES.md](./IMPLEMENTATION_PHASES.md) and [ARCHITECTURE.md](../ARCHITECTURE.md).
> Each task is sized for an AI agent to execute in a single focused session.

---

## Task Format

| Field | Description |
|-------|-------------|
| **Task ID** | `P{phase}-T{task}` format (e.g., `P1-T1`) |
| **Title** | Short, action-oriented title |
| **Description** | What to build, including file paths and libraries |
| **Dependencies** | Other task IDs that must be completed first |
| **Acceptance Criteria** | Concrete, testable conditions for "done" |
| **Complexity** | `S` (small, < 30 min), `M` (medium, 30–90 min), `L` (large, 90+ min) |

---

## Phase 1 — Project Scaffold & Database Schema

### P1-T1: Initialize monorepo root with workspaces

**Description**: Create the root `package.json` with npm workspaces pointing to `frontend/` and `api/`. Add a shared `tsconfig.base.json` with strict TypeScript settings. Create `.gitignore` (node_modules, dist, .env, .wrangler) and `.env.example` files documenting `DATABASE_URL`, `API_KEY`, `VITE_APP_URL`, `VITE_API_URL`.

**Dependencies**: None

**Acceptance Criteria**:
- Root `package.json` exists with `workspaces` field listing `frontend` and `api`
- `tsconfig.base.json` exists with `strict: true` and shared compiler options
- `.gitignore` covers node_modules, dist, .env, .wrangler, and IDE files
- `.env.example` documents all required environment variables

**Complexity**: S

---

### P1-T2: Scaffold frontend with Vite + React 19 + TypeScript

**Description**: Initialize `frontend/` with Vite, React 19, and TypeScript. Create `frontend/package.json`, `frontend/tsconfig.json` (extending root base), `frontend/vite.config.ts`, and `frontend/src/main.tsx` as the app entry point. Add `frontend/src/App.tsx` with a placeholder "BrewLog" heading. Install dependencies: `react`, `react-dom`, `typescript`, `vite`, `@vitejs/plugin-react`.

**Dependencies**: P1-T1

**Acceptance Criteria**:
- `cd frontend && npm run dev` starts the Vite dev server at `localhost:5173`
- Browser shows a placeholder page with "BrewLog" heading
- TypeScript compilation succeeds with no errors

**Complexity**: M

---

### P1-T3: Configure Tailwind CSS and shadcn/ui

**Description**: Install and configure Tailwind CSS in `frontend/`: `tailwindcss`, `postcss`, `autoprefixer`. Create `frontend/tailwind.config.ts` and `frontend/postcss.config.js`. Add Tailwind directives to `frontend/src/index.css`. Initialize shadcn/ui and install base primitives (Button, Card, Input, Badge) into `frontend/src/components/ui/`. Install `clsx`, `tailwind-merge`, `class-variance-authority`, `lucide-react`.

**Dependencies**: P1-T2

**Acceptance Criteria**:
- Tailwind utility classes render correctly in the browser
- At least Button, Card, Input, and Badge components exist in `frontend/src/components/ui/`
- `lucide-react` icons render without errors

**Complexity**: M

---

### P1-T4: Set up React Router v7 with placeholder root route

**Description**: Install `react-router` (v7) in `frontend/`. Configure client-side routing in `frontend/src/App.tsx` with a `BrowserRouter`. Create placeholder route files: `frontend/src/routes/root.tsx` (root layout), `frontend/src/routes/dashboard.tsx` (placeholder dashboard). Wire up the root route at `/` rendering the dashboard placeholder.

**Dependencies**: P1-T2

**Acceptance Criteria**:
- Navigating to `/` renders the placeholder dashboard page
- Route structure matches the architecture: `src/routes/root.tsx`, `src/routes/dashboard.tsx`
- No console errors related to routing

**Complexity**: S

---

### P1-T5: Wire up TanStack Query provider and Zustand store skeleton

**Description**: Install `@tanstack/react-query` and `zustand` in `frontend/`. Create `frontend/src/lib/store.ts` with a Zustand store skeleton containing `apiKey: string | null`, `isOnline: boolean`, and `pendingSyncCount: number`. Wrap the app in `QueryClientProvider` in `frontend/src/main.tsx`. Create `frontend/src/lib/types.ts` with TypeScript types for `Brew`, `Ingredient`, `FermentationEvent`, `BrewStatus`, `BrewType` matching the database schema.

**Dependencies**: P1-T2

**Acceptance Criteria**:
- `QueryClientProvider` wraps the app in the component tree
- `frontend/src/lib/store.ts` exports a Zustand store with `apiKey`, `isOnline`, `pendingSyncCount`
- `frontend/src/lib/types.ts` exports types for `Brew`, `Ingredient`, `FermentationEvent`, `BrewStatus` (`planning | fermenting | conditioning | bottled | done`), `BrewType` (`beer | mead | cider`)
- No runtime errors on app load

**Complexity**: S

---

### P1-T6: Scaffold API with Hono and health-check endpoint

**Description**: Initialize `api/` with TypeScript and Hono as the lightweight router. Create `api/package.json`, `api/tsconfig.json` (extending root base), and `api/src/index.ts` as the entry point. Implement `GET /api/health` returning `{ status: "ok" }`. Install dependencies: `hono`, `typescript`. Add a dev script using `wrangler dev` or a local Node adapter.

**Dependencies**: P1-T1

**Acceptance Criteria**:
- `cd api && npm run dev` starts the API dev server
- `GET /api/health` returns `200` with `{ "status": "ok" }`
- TypeScript compilation succeeds

**Complexity**: M

---

### P1-T7: Define Drizzle ORM schema for all tables

**Description**: Install `drizzle-orm`, `postgres` (postgres-js driver), `nanoid`, and `drizzle-kit` in `api/`. Create `api/src/db/schema.ts` defining three tables per the architecture:
- `brews`: id (text PK, nanoid), user_id (text, default `'default'`), name, type, batch_size_liters (real), target_og (real, nullable), target_fg (real, nullable), status (text, default `'fermenting'`), start_date, end_date (nullable), notes (nullable), created_at (timestamptz), updated_at (timestamptz)
- `ingredients`: id (text PK, nanoid), brew_id (text FK → brews.id), user_id, category, name, quantity (real), unit, date_added (nullable), notes (nullable), created_at (timestamptz)
- `fermentation_events`: id (text PK, nanoid), brew_id (text FK → brews.id), user_id, event_type, gravity (real, nullable), temperature_celsius (real, nullable), notes (nullable), event_date, created_at (timestamptz)

Add indexes on `ingredients.brew_id` and `fermentation_events.brew_id`. Define Drizzle relations between tables. Create `api/src/db/index.ts` to initialize the Drizzle client with the `postgres-js` adapter using `DATABASE_URL`.

**Dependencies**: P1-T6

**Acceptance Criteria**:
- `api/src/db/schema.ts` defines all three tables with correct column types and constraints
- Foreign keys from `ingredients` and `fermentation_events` to `brews` are defined
- Indexes exist on `ingredients.brew_id` and `fermentation_events.brew_id`
- `api/src/db/index.ts` exports a configured Drizzle client
- TypeScript compiles without errors

**Complexity**: M

---

### P1-T8: Generate and apply first Drizzle migration

**Description**: Configure `drizzle-kit` in `api/drizzle.config.ts` pointing to the schema file and PostgreSQL connection. Generate the initial migration with `drizzle-kit generate`. Apply the migration with `drizzle-kit migrate` (or `drizzle-kit push` for development). Verify tables exist in the database.

**Dependencies**: P1-T7

**Acceptance Criteria**:
- `api/drizzle.config.ts` exists with correct schema path and DB connection
- Migration files are generated in `api/src/db/migrations/`
- Running the migration creates `brews`, `ingredients`, and `fermentation_events` tables in PostgreSQL
- `drizzle-kit studio` shows the schema correctly

**Complexity**: S

---

### P1-T9: Implement API key auth middleware

**Description**: Create `api/src/middleware/auth.ts` exporting a Hono middleware function. The middleware reads the `API_KEY` environment variable and the `X-API-Key` request header. For `GET` and `OPTIONS` requests, pass through without checking. For all other methods (`POST`, `PUT`, `PATCH`, `DELETE`), return `401 Unauthorized` with `{ error: "Unauthorized" }` if the header is missing or doesn't match. Apply the middleware globally in `api/src/index.ts`.

**Dependencies**: P1-T6

**Acceptance Criteria**:
- `GET /api/health` works without any API key header
- `POST` requests without `X-API-Key` return `401`
- `POST` requests with an incorrect `X-API-Key` return `401`
- `POST` requests with the correct `X-API-Key` pass through to the handler
- Middleware is applied in `api/src/index.ts`

**Complexity**: S

---

### P1-T10: Create Zod validation schemas

**Description**: Install `zod` in `api/`. Create `api/src/validators.ts` with Zod schemas for all API request bodies:
- `createBrewSchema`: name (string, min 1), type (enum: beer/mead/cider), batch_size_liters (number, positive), target_og (number, optional), target_fg (number, optional), status (enum: planning/fermenting/conditioning/bottled/done, optional, default fermenting), start_date (string, ISO date), end_date (string, optional), notes (string, optional)
- `updateBrewSchema`: same as create but all fields optional (partial)
- `updateBrewStatusSchema`: status (enum: planning/fermenting/conditioning/bottled/done)
- `createIngredientSchema`: category (enum: grain/hop/honey/fruit/yeast/adjunct/other), name (string, min 1), quantity (number, positive), unit (enum: g/kg/ml/L), date_added (string, optional), notes (string, optional)
- `updateIngredientSchema`: partial of create
- `createEventSchema`: event_type (enum: gravity_reading/temperature/racking/addition/tasting/note/bottling/other), gravity (number, optional), temperature_celsius (number, optional), notes (string, optional), event_date (string, ISO datetime)

**Dependencies**: P1-T6

**Acceptance Criteria**:
- All schemas are exported from `api/src/validators.ts`
- Valid payloads pass validation
- Invalid payloads (missing required fields, wrong types, invalid enums) are rejected with descriptive errors
- Schemas match the database column constraints from the architecture

**Complexity**: M

---

### P1-T11: Build frontend API client with auth header

**Description**: Create `frontend/src/lib/api-client.ts` exporting a `fetchApi` function that wraps `fetch`. It reads `VITE_API_URL` from `import.meta.env` as the base URL. For all requests, if an API key exists in `localStorage` (key: `brewlog-api-key`), attach it as the `X-API-Key` header. Include `Content-Type: application/json` for requests with bodies. Handle non-OK responses by throwing an error with the response body. Export typed helper functions: `get<T>(path)`, `post<T>(path, body)`, `put<T>(path, body)`, `patch<T>(path, body)`, `del(path)`.

**Dependencies**: P1-T2

**Acceptance Criteria**:
- `frontend/src/lib/api-client.ts` exports `get`, `post`, `put`, `patch`, `del` functions
- API key from `localStorage` is attached as `X-API-Key` header when present
- Base URL is read from `VITE_API_URL` environment variable
- Non-OK responses throw errors with response body details
- TypeScript types are correct (generic return types)

**Complexity**: S

---

## Phase 2 — Brews API & Core Brew UI

### P2-T1: Implement brews API routes (CRUD)

**Description**: Create `api/src/routes/brews.ts` with Hono route handlers:
- `GET /api/brews` — query all brews ordered by `updated_at` desc, return as JSON array
- `GET /api/brews/:brewId` — query single brew by ID, include ingredient count and event count via subqueries, return 404 if not found
- `POST /api/brews` — validate body with `createBrewSchema`, generate `nanoid` for ID, insert into DB, return 201 with created brew
- `PUT /api/brews/:brewId` — validate body with `updateBrewSchema`, update brew, set `updated_at` to now, return updated brew or 404
- `PATCH /api/brews/:brewId/status` — validate body with `updateBrewStatusSchema`, update status and `updated_at`, return updated brew or 404
- `DELETE /api/brews/:brewId` — delete brew (cascade: delete ingredients and events first), return 204

Register routes in `api/src/index.ts`.

**Dependencies**: P1-T7, P1-T8, P1-T9, P1-T10

**Acceptance Criteria**:
- All six endpoints work correctly via curl/Postman
- `POST` creates a brew with a nanoid and returns 201
- `GET /api/brews` returns brews ordered by `updated_at` desc
- `GET /api/brews/:brewId` includes `ingredientCount` and `eventCount`
- `PUT` updates only provided fields and bumps `updated_at`
- `PATCH /status` changes only the status field
- `DELETE` removes the brew and its related data, returns 204
- Invalid payloads return 400 with Zod error details
- Non-existent IDs return 404

**Complexity**: L

---

### P2-T2: Create TypeScript types and utility functions

**Description**: Populate `frontend/src/lib/types.ts` with full TypeScript types matching the API responses:
- `Brew` (all fields from schema + `ingredientCount?: number`, `eventCount?: number`)
- `BrewStatus` union type: `'planning' | 'fermenting' | 'conditioning' | 'bottled' | 'done'`
- `BrewType` union type: `'beer' | 'mead' | 'cider'`
- `Ingredient`, `FermentationEvent` types

Create `frontend/src/lib/utils.ts` with:
- `calculateAbv(og: number, fg: number): number` — standard ABV formula: `(og - fg) * 131.25`
- `formatDate(dateString: string): string` — human-readable date using `date-fns`
- `brewAge(startDate: string): string` — e.g., "12 days", "3 weeks" using `date-fns`
- `cn(...inputs)` — class name merge utility using `clsx` + `tailwind-merge`

Install `date-fns` in `frontend/`.

**Dependencies**: P1-T5

**Acceptance Criteria**:
- All types are exported and match the API response shapes
- `calculateAbv(1.050, 1.010)` returns approximately `5.25`
- `formatDate` returns a readable date string
- `brewAge` returns a human-readable duration
- `cn` merges class names correctly

**Complexity**: S

---

### P2-T3: Create TanStack Query hooks for brews

**Description**: Create `frontend/src/lib/queries.ts` with TanStack Query hooks:
- `useBrews()` — fetches `GET /api/brews`, query key `['brews']`, returns `UseQueryResult<Brew[]>`
- `useBrew(brewId: string)` — fetches `GET /api/brews/:brewId`, query key `['brew', brewId]`, returns `UseQueryResult<Brew>`

Create `frontend/src/lib/mutations.ts` with TanStack Query mutations:
- `useCreateBrew()` — `POST /api/brews`, invalidates `['brews']` query on success
- `useUpdateBrew(brewId: string)` — `PUT /api/brews/:brewId`, invalidates `['brews']` and `['brew', brewId]`
- `useUpdateBrewStatus(brewId: string)` — `PATCH /api/brews/:brewId/status`, invalidates same
- `useDeleteBrew()` — `DELETE /api/brews/:brewId`, invalidates `['brews']`

Use the `api-client.ts` functions for all requests.

**Dependencies**: P1-T11, P2-T2

**Acceptance Criteria**:
- `useBrews()` fetches and caches the brew list
- `useBrew(id)` fetches and caches a single brew
- `useCreateBrew` mutation creates a brew and invalidates the list cache
- `useUpdateBrew` mutation updates and invalidates relevant caches
- `useDeleteBrew` mutation deletes and invalidates the list cache
- All hooks use correct query keys for cache management

**Complexity**: M

---

### P2-T4: Build RootLayout and AppNav components

**Description**: Create `frontend/src/layouts/root-layout.tsx` as the root layout component wrapping all pages. It should include the `AppNav` component and an `<Outlet />` for nested routes. Create `frontend/src/components/app-nav.tsx` with a top navigation bar containing the BrewLog logo/text and a link to the dashboard (`/`). Style with Tailwind CSS. Update `frontend/src/App.tsx` to use `RootLayout` as the root route element.

**Dependencies**: P1-T3, P1-T4

**Acceptance Criteria**:
- `RootLayout` renders `AppNav` and an `Outlet` for child routes
- `AppNav` shows "BrewLog" branding and a link to `/`
- Navigation bar is responsive (works on mobile and desktop)
- Root route in `App.tsx` uses `RootLayout` as the layout element

**Complexity**: S

---

### P2-T5: Build BrewForm component (create/edit)

**Description**: Create `frontend/src/components/forms/brew-form.tsx`. The form includes fields for: name (text input), type (select: beer/mead/cider), batch_size_liters (number input, label "Batch Size (L)"), target_og (number input, optional), target_fg (number input, optional), status (select: planning/fermenting/conditioning/bottled/done), start_date (date input), end_date (date input, optional), notes (textarea, optional). Use shadcn/ui form components (Input, Select, Button, Card). The form accepts an optional `initialData` prop for edit mode. On submit, call the provided `onSubmit` callback with the form data. Create `frontend/src/lib/validators.ts` with Zod schemas mirroring the API validators for client-side validation.

**Dependencies**: P1-T3, P2-T2

**Acceptance Criteria**:
- Form renders all fields with correct input types and labels
- Metric units are displayed (L for batch size)
- In create mode, form starts empty with sensible defaults (status: fermenting, start_date: today)
- In edit mode, form pre-fills with `initialData`
- Validation prevents submission of invalid data (empty name, missing required fields)
- `onSubmit` is called with validated form data

**Complexity**: M

---

### P2-T6: Build BrewCard and BrewStatusBadge components

**Description**: Create `frontend/src/components/brew-card.tsx` — a card component displaying a brew summary: name, type icon, status badge, batch size in L, start date, and brew age. Clicking the card navigates to `/brews/:brewId`. Create `frontend/src/components/brew-status-badge.tsx` — a colored badge component for brew status (e.g., green for fermenting, blue for conditioning). Create `frontend/src/components/brew-type-icon.tsx` — renders a `lucide-react` icon based on brew type (beer/mead/cider). Use shadcn/ui Card and Badge components.

**Dependencies**: P1-T3, P2-T2

**Acceptance Criteria**:
- `BrewCard` displays brew name, type icon, status badge, batch size (L), and age
- Clicking a `BrewCard` navigates to the brew detail page
- `BrewStatusBadge` renders different colors per status
- `BrewTypeIcon` renders distinct icons for beer, mead, and cider
- Components are styled with Tailwind and responsive

**Complexity**: S

---

### P2-T7: Build dashboard page with brew list

**Description**: Implement `frontend/src/routes/dashboard.tsx` as the main dashboard page. Use the `useBrews()` hook to fetch all brews. Render a grid of `BrewCard` components. Add a "New Brew" button linking to `/brews/new`. Create `frontend/src/components/brew-dashboard.tsx` as the container component that handles the query state (loading, error, empty). Show a loading skeleton while fetching and an empty state message when no brews exist.

**Dependencies**: P2-T3, P2-T4, P2-T6

**Acceptance Criteria**:
- Dashboard page renders at `/`
- Brews are displayed as a grid of `BrewCard` components
- "New Brew" button links to `/brews/new`
- Loading state shows skeleton placeholders
- Empty state shows a message encouraging the user to create their first brew
- Error state shows a user-friendly error message

**Complexity**: M

---

### P2-T8: Build create brew page

**Description**: Implement `frontend/src/routes/brews/new.tsx`. Render the `BrewForm` in create mode. On submit, call `useCreateBrew` mutation. On success, navigate to `/brews/:brewId` (the newly created brew's detail page). Show a toast notification on success. Handle errors with a toast or inline message. Install and configure shadcn/ui Toast component if not already present.

**Dependencies**: P2-T3, P2-T5

**Acceptance Criteria**:
- `/brews/new` renders the `BrewForm` in create mode
- Submitting the form creates a brew via the API
- On success, user is redirected to the new brew's detail page
- Success toast notification is shown
- API errors are displayed to the user

**Complexity**: S

---

### P2-T9: Build BrewDetailLayout with tab navigation

**Description**: Create `frontend/src/layouts/brew-detail-layout.tsx` — a layout component for brew detail pages with tab navigation. Tabs: Overview, Ingredients, Log, QR Code. Each tab links to the corresponding sub-route (`/brews/:brewId`, `/brews/:brewId/ingredients`, `/brews/:brewId/log`, `/brews/:brewId/qr`). Use shadcn/ui Tabs component. The layout fetches the brew via `useBrew(brewId)` and displays the brew name as a heading. Implement `frontend/src/routes/brews/$brewId/layout.tsx` using this layout with an `<Outlet />` for child routes.

**Dependencies**: P1-T3, P1-T4, P2-T3

**Acceptance Criteria**:
- Tab navigation renders with Overview, Ingredients, Log, and QR Code tabs
- Active tab is highlighted based on current route
- Clicking a tab navigates to the correct sub-route
- Brew name is displayed as a heading above the tabs
- Layout handles loading and error states for the brew query

**Complexity**: M

---

### P2-T10: Build brew overview/summary page

**Description**: Create `frontend/src/components/brew-summary.tsx` — displays brew details: name, type, batch size (L), target OG/FG, current status, start/end dates, notes, and brew age. Show calculated ABV if both OG and FG are available using `calculateAbv()` from `utils.ts`. Implement `frontend/src/routes/brews/$brewId/index.tsx` using `BrewSummary`. Add an "Edit" button (visible when authenticated) linking to `/brews/:brewId/edit`. Add a "Delete" button with confirmation dialog using shadcn/ui Dialog.

**Dependencies**: P2-T2, P2-T3, P2-T9

**Acceptance Criteria**:
- Brew overview page renders at `/brews/:brewId`
- All brew fields are displayed with correct formatting and metric units
- ABV is calculated and shown when OG and FG are available
- "Edit" button links to the edit page (shown only when API key is in localStorage)
- "Delete" button shows a confirmation dialog and deletes the brew on confirm
- After deletion, user is redirected to the dashboard

**Complexity**: M

---

### P2-T11: Build edit brew page

**Description**: Implement `frontend/src/routes/brews/$brewId/edit.tsx`. Fetch the current brew data with `useBrew(brewId)`. Render `BrewForm` in edit mode with `initialData` set to the current brew. On submit, call `useUpdateBrew` mutation. On success, navigate back to the brew overview page with a success toast. Include a status change dropdown that uses `useUpdateBrewStatus`.

**Dependencies**: P2-T3, P2-T5, P2-T9

**Acceptance Criteria**:
- `/brews/:brewId/edit` renders the `BrewForm` pre-filled with current brew data
- Submitting saves changes via the API
- On success, user is redirected to the brew overview page
- Status can be changed independently via the status dropdown
- Only accessible when authenticated (redirect or prompt if no API key)

**Complexity**: S

---

### P2-T12: Implement API key unlock prompt

**Description**: Create `frontend/src/components/auth-prompt.tsx` — a modal/dialog prompting the user to enter their API key. On first visit (no `brewlog-api-key` in `localStorage`), show the prompt. Store the key in `localStorage` on submit. Update the Zustand store (`frontend/src/lib/store.ts`) to track `apiKey` state and sync with `localStorage`. Add a "Lock" button in `AppNav` (`frontend/src/components/app-nav.tsx`) to clear the stored key. Ensure the API client (`frontend/src/lib/api-client.ts`) reads from `localStorage`.

**Dependencies**: P1-T5, P1-T11, P2-T4

**Acceptance Criteria**:
- On first visit with no stored API key, a prompt/dialog appears
- Entering a key stores it in `localStorage` and Zustand store
- Subsequent visits use the stored key without prompting
- "Lock" button in nav clears the key and shows the prompt again
- API client includes the key in requests when available
- Write operations fail gracefully when no key is stored

**Complexity**: M

---

### P2-T13: Register all Phase 2 routes in App.tsx

**Description**: Update `frontend/src/App.tsx` to register all brew-related routes with React Router v7:
- `/` → `dashboard.tsx`
- `/brews/new` → `brews/new.tsx`
- `/brews/:brewId` → `brews/$brewId/layout.tsx` (layout) with children:
  - index → `brews/$brewId/index.tsx`
  - `edit` → `brews/$brewId/edit.tsx`
  - `ingredients` → placeholder
  - `log` → placeholder
  - `qr` → placeholder

**Dependencies**: P2-T7, P2-T8, P2-T9, P2-T10, P2-T11

**Acceptance Criteria**:
- All routes resolve correctly in the browser
- Navigation between pages works without full page reloads
- Nested routes under `/brews/:brewId` render within the `BrewDetailLayout`
- Placeholder tabs for ingredients, log, and QR don't error

**Complexity**: S

---

## Phase 3 — Ingredients API & UI

### P3-T1: Implement ingredients API routes

**Description**: Create `api/src/routes/ingredients.ts` with Hono route handlers:
- `GET /api/brews/:brewId/ingredients` — query all ingredients for a brew ordered by `created_at`, return 404 if brew doesn't exist
- `POST /api/brews/:brewId/ingredients` — validate with `createIngredientSchema`, generate nanoid, insert, return 201
- `PUT /api/brews/:brewId/ingredients/:id` — validate with `updateIngredientSchema`, update, return updated or 404
- `DELETE /api/brews/:brewId/ingredients/:id` — delete, return 204 or 404

Register routes in `api/src/index.ts`.

**Dependencies**: P1-T7, P1-T8, P1-T9, P1-T10

**Acceptance Criteria**:
- All four endpoints work correctly
- `POST` creates an ingredient linked to the correct brew
- `GET` returns only ingredients for the specified brew
- `PUT` updates only provided fields
- `DELETE` removes the ingredient and returns 204
- Invalid payloads return 400
- Non-existent brew or ingredient IDs return 404

**Complexity**: M

---

### P3-T2: Create TanStack Query hooks for ingredients

**Description**: Add to `frontend/src/lib/queries.ts`:
- `useIngredients(brewId: string)` — fetches `GET /api/brews/:brewId/ingredients`, query key `['ingredients', brewId]`

Add to `frontend/src/lib/mutations.ts`:
- `useAddIngredient(brewId: string)` — `POST`, invalidates `['ingredients', brewId]`
- `useUpdateIngredient(brewId: string)` — `PUT`, invalidates `['ingredients', brewId]`
- `useDeleteIngredient(brewId: string)` — `DELETE`, invalidates `['ingredients', brewId]`

**Dependencies**: P1-T11, P2-T2

**Acceptance Criteria**:
- `useIngredients` fetches and caches ingredients for a brew
- All mutations invalidate the ingredients cache on success
- Hooks use the API client for requests

**Complexity**: S

---

### P3-T3: Build IngredientForm component

**Description**: Create `frontend/src/components/forms/ingredient-form.tsx`. Fields: category (select: grain/hop/honey/fruit/yeast/adjunct/other), name (text input), quantity (number input), unit (select: g/kg/ml/L — metric only), date_added (date input, optional), notes (textarea, optional). Accept optional `initialData` for edit mode. Validate with Zod schemas from `frontend/src/lib/validators.ts`. Use shadcn/ui components (Input, Select, Button).

**Dependencies**: P1-T3, P2-T2

**Acceptance Criteria**:
- Form renders all fields with correct input types
- Unit selector only shows metric units: g, kg, ml, L
- Category selector shows all seven categories
- Validation prevents empty name or non-positive quantity
- Edit mode pre-fills with `initialData`

**Complexity**: S

---

### P3-T4: Build IngredientTable and IngredientRow components

**Description**: Create `frontend/src/components/ingredient-table.tsx` — renders a table of ingredients with columns: Category, Name, Quantity, Unit, Date Added, Notes, Actions. Create `frontend/src/components/ingredient-row.tsx` — a single table row with edit and delete action buttons. Edit opens the `IngredientForm` inline or in a shadcn/ui Dialog. Delete shows a confirmation before removing. Use shadcn/ui Table component.

**Dependencies**: P1-T3, P3-T3

**Acceptance Criteria**:
- Table displays all ingredients with correct columns
- Edit button opens the ingredient form pre-filled with current data
- Delete button shows confirmation and removes the ingredient on confirm
- Empty state shows a message when no ingredients exist
- Table is responsive (horizontal scroll on mobile or card layout)

**Complexity**: M

---

### P3-T5: Build ingredients route page and wire up tab

**Description**: Implement `frontend/src/routes/brews/$brewId/ingredients.tsx`. Use `useIngredients(brewId)` to fetch ingredients. Render `IngredientTable` with the data. Include `IngredientForm` for adding new ingredients (above or below the table). Wire up the Ingredients tab in `BrewDetailLayout` to navigate to this route. Update route registration in `frontend/src/App.tsx` to replace the placeholder.

**Dependencies**: P2-T9, P2-T13, P3-T2, P3-T3, P3-T4

**Acceptance Criteria**:
- `/brews/:brewId/ingredients` renders the ingredient list and add form
- Adding an ingredient via the form persists it and updates the table
- Editing an ingredient updates the row in place
- Deleting an ingredient removes it from the table
- Ingredients tab in the brew detail layout navigates to this page
- Loading and error states are handled

**Complexity**: M

---

## Phase 4 — Fermentation Events API & UI

### P4-T1: Implement fermentation events API routes

**Description**: Create `api/src/routes/events.ts` with Hono route handlers:
- `GET /api/brews/:brewId/events` — query all events for a brew ordered by `event_date` desc, return 404 if brew doesn't exist
- `POST /api/brews/:brewId/events` — validate with `createEventSchema`, generate nanoid, insert, return 201
- `DELETE /api/brews/:brewId/events/:id` — delete, return 204 or 404

Register routes in `api/src/index.ts`.

**Dependencies**: P1-T7, P1-T8, P1-T9, P1-T10

**Acceptance Criteria**:
- All three endpoints work correctly
- `POST` creates an event linked to the correct brew
- `GET` returns events ordered by `event_date` descending
- `DELETE` removes the event and returns 204
- Invalid payloads return 400 with Zod error details
- Non-existent brew or event IDs return 404

**Complexity**: M

---

### P4-T2: Create TanStack Query hooks for events

**Description**: Add to `frontend/src/lib/queries.ts`:
- `useEvents(brewId: string)` — fetches `GET /api/brews/:brewId/events`, query key `['events', brewId]`

Add to `frontend/src/lib/mutations.ts`:
- `useAddEvent(brewId: string)` — `POST /api/brews/:brewId/events`, invalidates `['events', brewId]` and `['brew', brewId]` (to update event count)
- `useDeleteEvent(brewId: string)` — `DELETE /api/brews/:brewId/events/:id`, invalidates same

**Dependencies**: P1-T11, P2-T2

**Acceptance Criteria**:
- `useEvents` fetches and caches events for a brew
- `useAddEvent` mutation creates an event and invalidates relevant caches
- `useDeleteEvent` mutation deletes and invalidates relevant caches
- Event count on brew detail is updated after adding/deleting events

**Complexity**: S

---

### P4-T3: Build EventForm component with dynamic fields

**Description**: Create `frontend/src/components/forms/event-form.tsx`. The form has an event type selector (large buttons or select: gravity_reading, temperature, racking, addition, tasting, note, bottling, other) and dynamic fields based on selected type:
- `gravity_reading` → gravity (number input), temperature_celsius (number, optional), notes (optional)
- `temperature` → temperature_celsius (number input, label "Temperature (°C)"), notes (optional)
- `racking` → notes (textarea)
- `addition` → notes (textarea describing what was added)
- `tasting` → notes (textarea)
- `note` → notes (textarea)
- `bottling` → notes (textarea)
- `other` → notes (textarea)

All types include `event_date` (datetime input, defaults to now). Validate with Zod. Use shadcn/ui components.

**Dependencies**: P1-T3, P2-T2

**Acceptance Criteria**:
- Event type selector shows all eight event types
- Selecting a type shows the correct dynamic fields
- Gravity reading type shows gravity and optional temperature inputs
- Temperature type shows temperature input with °C label
- Event date defaults to current date/time
- Validation ensures required fields per type are filled
- Form calls `onSubmit` with validated data

**Complexity**: M

---

### P4-T4: Build EventTimeline, EventCard, and EventTypeIcon components

**Description**: Create `frontend/src/components/event-timeline.tsx` — a chronological list of fermentation events, styled as a vertical timeline. Create `frontend/src/components/event-card.tsx` — displays a single event with: type icon, event type label, date, gravity (if present), temperature in °C (if present), notes, and a delete button (when authenticated). Create `frontend/src/components/event-type-icon.tsx` — renders a `lucide-react` icon per event type (e.g., thermometer for temperature, droplet for gravity). Use shadcn/ui Card component.

**Dependencies**: P1-T3, P2-T2

**Acceptance Criteria**:
- Timeline renders events in chronological order (newest first)
- Each event card shows type icon, label, date, and relevant data
- Gravity readings display the gravity value
- Temperature readings display the value with °C unit
- Delete button is visible only when authenticated
- Empty state shows a message when no events exist

**Complexity**: M

---

### P4-T5: Build GravityChart component

**Description**: Install `recharts` in `frontend/`. Create `frontend/src/components/gravity-chart.tsx` — a line chart plotting gravity readings over time. X-axis: event_date (formatted dates). Y-axis: gravity values. Filter events to only `gravity_reading` type. Use Recharts `LineChart`, `Line`, `XAxis`, `YAxis`, `Tooltip`, `ResponsiveContainer`. Show target OG and target FG as horizontal reference lines if available. Lazy-load the component with `React.lazy()` to reduce initial bundle size.

**Dependencies**: P1-T3, P2-T2

**Acceptance Criteria**:
- Chart renders gravity readings as a line over time
- X-axis shows formatted dates, Y-axis shows gravity values
- Target OG and FG are shown as dashed horizontal reference lines when available
- Chart is responsive (fills container width)
- Component is lazy-loaded
- Empty state shown when no gravity readings exist

**Complexity**: M

---

### P4-T6: Build fermentation log route page and wire up tab

**Description**: Implement `frontend/src/routes/brews/$brewId/log.tsx`. Use `useEvents(brewId)` to fetch events. Render `EventTimeline` with the events data. Include `EventForm` for adding new events. Render `GravityChart` above the timeline (if gravity readings exist). Wire up the Log tab in `BrewDetailLayout`. Update route registration in `frontend/src/App.tsx`.

**Dependencies**: P2-T9, P2-T13, P4-T2, P4-T3, P4-T4, P4-T5

**Acceptance Criteria**:
- `/brews/:brewId/log` renders the gravity chart, event timeline, and add-event form
- Adding an event via the form persists it and updates the timeline and chart
- Deleting an event removes it from the timeline
- Gravity chart updates when new gravity readings are added
- Log tab in the brew detail layout navigates to this page
- Loading and error states are handled

**Complexity**: M

---

### P4-T7: Update BrewSummary with event data

**Description**: Update `frontend/src/components/brew-summary.tsx` to display: latest gravity reading (from events), calculated ABV (using OG from brew and latest gravity reading), total event count, and latest event date. Use `useEvents(brewId)` to fetch events and extract the latest gravity reading. Update the ABV calculation to use the latest gravity reading as FG if the brew's `target_fg` is not set.

**Dependencies**: P2-T10, P4-T2

**Acceptance Criteria**:
- Brew summary shows latest gravity reading value
- ABV is calculated using OG and latest gravity (or target FG)
- Total event count is displayed
- Latest event date is shown
- Summary gracefully handles brews with no events

**Complexity**: S

---

## Phase 5 — QR Codes & Quick-Log Flow

### P5-T1: Build QrCodeDisplay component

**Description**: Install `qrcode.react` in `frontend/`. Create `frontend/src/components/qr-code-display.tsx` — renders a QR code using `qrcode.react` (SVG mode) encoding the URL `{VITE_APP_URL}/brews/{brewId}`. Read `VITE_APP_URL` from `import.meta.env`. Display the brew name and type above the QR code. Size the QR code large enough for easy scanning (at least 256x256).

**Dependencies**: P1-T3

**Acceptance Criteria**:
- QR code renders as an SVG encoding the correct brew URL
- `VITE_APP_URL` is used as the base URL (not hardcoded)
- Brew name and type are displayed above the QR
- QR code is at least 256x256 pixels
- Component accepts `brewId`, `brewName`, and `brewType` as props

**Complexity**: S

---

### P5-T2: Build QrDownloadButton and QrPrintButton components

**Description**: Create `frontend/src/components/qr-download-button.tsx` — converts the QR SVG to a PNG using canvas and triggers a file download named `{brewName}-qr.png`. Create `frontend/src/components/qr-print-button.tsx` — opens the browser print dialog with a print-friendly layout containing the brew name, type, start date, and QR code. Use shadcn/ui Button component.

**Dependencies**: P5-T1

**Acceptance Criteria**:
- Download button saves a PNG file of the QR code
- Downloaded file is named with the brew name
- Print button opens the print dialog
- Print layout includes brew name, type, start date, and QR code
- Print layout is clean (no navigation or other UI elements)

**Complexity**: M

---

### P5-T3: Build QR code route page and wire up tab

**Description**: Implement `frontend/src/routes/brews/$brewId/qr.tsx`. Render `QrCodeDisplay`, `QrDownloadButton`, and `QrPrintButton`. Fetch brew data via `useBrew(brewId)` for the brew name, type, and start date. Wire up the QR Code tab in `BrewDetailLayout`. Update route registration in `frontend/src/App.tsx`.

**Dependencies**: P2-T9, P2-T13, P5-T1, P5-T2

**Acceptance Criteria**:
- `/brews/:brewId/qr` renders the QR code with download and print buttons
- QR code encodes the correct URL for the brew
- QR Code tab in the brew detail layout navigates to this page
- Page handles loading state while brew data is fetched

**Complexity**: S

---

### P5-T4: Build QuickLogForm component

**Description**: Create `frontend/src/components/forms/quick-log-form.tsx` — a mobile-optimized event form with large tap targets. Header shows brew name and current status. Event type selector uses large buttons (not a small dropdown). Dynamic fields based on selected type (same logic as `EventForm` but with larger inputs):
- Gravity → numeric input with large font
- Temperature → numeric input with °C label, large font
- Tasting/Racking/Note/Bottling/Other → large textarea
- Addition → name + quantity (g/kg/ml/L) + notes

Date/time defaults to now, editable. Large prominent submit button. On success, show a toast and reset the form with a link to the full log.

**Dependencies**: P1-T3, P2-T2, P4-T3

**Acceptance Criteria**:
- Form is mobile-optimized with large tap targets (min 44px touch targets)
- Event type selector uses large, tappable buttons
- Dynamic fields match the event type
- Date/time defaults to current time
- Submit button is large and prominent
- Success toast is shown after submission
- Form resets after successful submission
- Link to full log is shown after submission

**Complexity**: M

---

### P5-T5: Build quick-log route page

**Description**: Implement `frontend/src/routes/quick-log/$brewId.tsx`. Fetch brew data with `useBrew(brewId)`. Render `QuickLogForm`. On submit, call `useAddEvent` mutation. Require authentication — if no API key in localStorage, show the auth prompt or redirect. Show 404 if brew doesn't exist. Add a "Quick Log" button to the brew detail page (`frontend/src/routes/brews/$brewId/index.tsx`) visible only when authenticated, linking to `/quick-log/:brewId`.

**Dependencies**: P2-T3, P2-T10, P2-T12, P4-T2, P5-T4

**Acceptance Criteria**:
- `/quick-log/:brewId` renders the quick-log form
- Form submits events via the events API
- Authentication is required (prompt shown if no API key)
- 404 page shown if brew ID doesn't exist
- "Quick Log" button appears on brew detail page when authenticated
- Success toast shown after logging an event
- Route is registered in `frontend/src/App.tsx`

**Complexity**: M

---

## Phase 6 — Dashboard Polish & Filtering

### P6-T1: Build StatusFilter component

**Description**: Create `frontend/src/components/status-filter.tsx` — a row of filter buttons for brew statuses: All, Planning, Fermenting, Conditioning, Bottled, Done. Clicking a filter updates the active filter state. Active filter is visually highlighted. Use shadcn/ui Button or ToggleGroup component. The component accepts `activeFilter` and `onFilterChange` props.

**Dependencies**: P1-T3, P2-T2

**Acceptance Criteria**:
- Filter buttons render for all statuses plus "All"
- Active filter is visually distinct (highlighted/selected state)
- Clicking a filter calls `onFilterChange` with the selected status
- "All" filter shows all brews (no filtering)

**Complexity**: S

---

### P6-T2: Add search and filtering to dashboard

**Description**: Update `frontend/src/routes/dashboard.tsx` and `frontend/src/components/brew-dashboard.tsx` to include:
- `StatusFilter` component for filtering by status
- A search input for filtering brews by name (client-side, case-insensitive)
- Client-side filtering logic: filter the `useBrews()` results by active status and search term
- Brew cards sorted by status groups or by `updated_at` desc

**Dependencies**: P2-T7, P6-T1

**Acceptance Criteria**:
- Status filter buttons filter the displayed brews
- Search input filters brews by name (case-insensitive)
- Filters can be combined (status + search)
- Clearing filters shows all brews
- Empty state shown when no brews match the filters

**Complexity**: M

---

### P6-T3: Polish BrewCard with full details

**Description**: Update `frontend/src/components/brew-card.tsx` to display: status badge (via `BrewStatusBadge`), type icon (via `BrewTypeIcon`), brew name, batch size in L, brew age (via `brewAge()` utility), and latest gravity reading (from `ingredientCount`/`eventCount` on the brew object). Ensure the card layout is visually polished with consistent spacing and typography.

**Dependencies**: P2-T6, P2-T2

**Acceptance Criteria**:
- Card shows status badge, type icon, name, batch size (L), age, and latest gravity
- Card layout is visually consistent and polished
- Cards are responsive (stack on mobile, grid on desktop)

**Complexity**: S

---

### P6-T4: Build 404 not-found page

**Description**: Create `frontend/src/routes/not-found.tsx` — a 404 page with a friendly message and a link back to the dashboard. Register as the catch-all route in `frontend/src/App.tsx`. Style with Tailwind and shadcn/ui Button for the link.

**Dependencies**: P1-T3, P1-T4

**Acceptance Criteria**:
- Unknown routes render the 404 page
- Page shows a clear "Page Not Found" message
- Link to dashboard is prominent and functional
- Page is styled consistently with the rest of the app

**Complexity**: S

---

### P6-T5: Add loading skeletons for all data-fetching pages

**Description**: Create skeleton loading components for: brew card grid (dashboard), brew summary, ingredient table, event timeline, and QR code page. Use shadcn/ui Skeleton component or Tailwind `animate-pulse` classes. Apply skeletons in all route pages where TanStack Query `isLoading` is true.

**Dependencies**: P2-T7, P2-T10, P3-T5, P4-T6, P5-T3

**Acceptance Criteria**:
- Dashboard shows skeleton cards while loading
- Brew summary shows skeleton layout while loading
- Ingredient table shows skeleton rows while loading
- Event timeline shows skeleton cards while loading
- Skeletons match the approximate shape of the real content

**Complexity**: M

---

### P6-T6: Add error boundaries and error states

**Description**: Create a reusable error boundary component at `frontend/src/components/error-boundary.tsx` using React's `ErrorBoundary` pattern. Add error state UI to all data-fetching pages (dashboard, brew detail, ingredients, events) showing a user-friendly message with a retry button when TanStack Query `isError` is true. Wrap route sections with the error boundary.

**Dependencies**: P2-T7, P2-T10, P3-T5, P4-T6

**Acceptance Criteria**:
- Error boundary catches rendering errors and shows a fallback UI
- All data-fetching pages show error messages when API requests fail
- Retry button re-fetches the failed query
- Error messages are user-friendly (not raw error objects)

**Complexity**: M

---

### P6-T7: Add toast notifications for all mutations

**Description**: Ensure all create, update, and delete mutations across the app show toast notifications using shadcn/ui Toast (or Sonner). Add success toasts for: brew created, brew updated, brew deleted, status changed, ingredient added/updated/deleted, event logged/deleted. Add error toasts for failed mutations. Configure the toast provider in `frontend/src/layouts/root-layout.tsx` if not already present.

**Dependencies**: P2-T8, P2-T10, P2-T11, P3-T5, P4-T6, P5-T5

**Acceptance Criteria**:
- All successful mutations show a success toast
- All failed mutations show an error toast
- Toasts auto-dismiss after a few seconds
- Toast provider is configured at the root layout level

**Complexity**: S

---

### P6-T8: Responsive layout polish for all pages

**Description**: Review and polish the responsive layout of all pages for mobile (< 640px) and desktop (> 1024px) viewports. Ensure: navigation collapses or adapts on mobile, brew card grid switches from multi-column to single-column, forms are full-width on mobile, tables scroll horizontally or switch to card layout on mobile, tab navigation is scrollable on small screens. Test all pages at 375px and 1280px widths.

**Dependencies**: P2-T4, P2-T7, P2-T10, P3-T5, P4-T6, P5-T3, P5-T5

**Acceptance Criteria**:
- All pages are usable at 375px width (mobile)
- All pages are well-laid-out at 1280px width (desktop)
- No horizontal overflow or broken layouts
- Touch targets are at least 44px on mobile
- Navigation is accessible on all screen sizes

**Complexity**: M

---

## Phase 7 — Offline Support & PWA

### P7-T1: Configure vite-plugin-pwa for Service Worker

**Description**: Install `vite-plugin-pwa` in `frontend/`. Configure it in `frontend/vite.config.ts` with: `registerType: 'autoUpdate'`, a manifest with app name "BrewLog", theme color, and icons. Configure the Service Worker to cache static assets (HTML, JS, CSS, images) using a cache-first strategy. Create `frontend/src/sw.ts` for Service Worker registration on app load.

**Dependencies**: P1-T2

**Acceptance Criteria**:
- Service Worker is generated during build
- Static assets are cached by the Service Worker
- App shell loads when device is offline (after first visit)
- PWA manifest is generated with correct app name and icons
- Service Worker auto-updates when new versions are deployed

**Complexity**: M

---

### P7-T2: Implement IndexedDB cache for API responses

**Description**: Install `idb` in `frontend/`. Create `frontend/src/lib/offline.ts` with IndexedDB operations using the `idb` library:
- Database name: `brewlog-cache`, stores: `brews`, `ingredients`, `events`
- `cacheBrews(brews: Brew[])` — store brew list
- `getCachedBrews(): Promise<Brew[]>` — retrieve cached brew list
- `cacheBrew(brew: Brew)` — store single brew
- `getCachedBrew(id: string): Promise<Brew | undefined>`
- `cacheIngredients(brewId: string, ingredients: Ingredient[])` — store ingredients
- `getCachedIngredients(brewId: string): Promise<Ingredient[]>`
- `cacheEvents(brewId: string, events: FermentationEvent[])` — store events
- `getCachedEvents(brewId: string): Promise<FermentationEvent[]>`

**Dependencies**: P1-T5

**Acceptance Criteria**:
- IndexedDB database is created with correct stores
- All cache/get functions work correctly
- Cached data persists across page reloads
- Functions handle missing data gracefully (return empty arrays)

**Complexity**: M

---

### P7-T3: Integrate IndexedDB cache with TanStack Query

**Description**: Update `frontend/src/lib/queries.ts` to integrate with the IndexedDB cache:
- On successful API fetch, cache the response in IndexedDB (via `offline.ts` functions)
- Configure `staleTime` (e.g., 5 minutes) and `gcTime` (e.g., 24 hours) for offline-friendly caching
- Add `initialData` or `placeholderData` to queries that reads from IndexedDB cache
- On reconnection (online event), trigger refetch of stale queries

Update the Zustand store (`frontend/src/lib/store.ts`) to track `isOnline` state using `navigator.onLine` and `online`/`offline` events.

**Dependencies**: P2-T3, P3-T2, P4-T2, P7-T2

**Acceptance Criteria**:
- API responses are cached in IndexedDB after successful fetches
- Previously viewed data is available from IndexedDB when offline
- `staleTime` and `gcTime` are configured for offline use
- Zustand store tracks online/offline state
- Stale queries are refetched when connectivity is restored

**Complexity**: L

---

### P7-T4: Implement offline write outbox queue

**Description**: Extend `frontend/src/lib/offline.ts` with an outbox queue:
- IndexedDB store: `outbox` with fields: `id` (auto-increment), `method`, `url`, `body`, `createdAt`
- `addToOutbox(method: string, url: string, body?: object): Promise<void>`
- `getOutboxItems(): Promise<OutboxItem[]>`
- `removeFromOutbox(id: number): Promise<void>`
- `getOutboxCount(): Promise<number>`

Update the API client (`frontend/src/lib/api-client.ts`) to detect offline state: when offline and a write operation is attempted, save to the outbox instead of making the network request. Show a toast indicating the operation is queued.

**Dependencies**: P1-T11, P7-T2

**Acceptance Criteria**:
- Write operations while offline are saved to the IndexedDB outbox
- Outbox items include method, URL, body, and timestamp
- User sees a toast notification that the operation is queued
- Outbox count is retrievable
- API client detects offline state via `navigator.onLine`

**Complexity**: M

---

### P7-T5: Implement background sync for outbox

**Description**: Create `frontend/src/lib/sync.ts` with sync orchestration:
- `syncOutbox(): Promise<void>` — reads all outbox items, replays them to the API in order (FIFO), removes successfully synced items
- Handle failures: if a sync request fails, stop and retry later (don't skip items)
- Register for the `online` event to trigger sync automatically
- Use the Service Worker Background Sync API if available, fall back to manual sync on `online` event

Update the Zustand store (`frontend/src/lib/store.ts`) with `pendingSyncCount` and `isSyncing` state.

**Dependencies**: P7-T1, P7-T4

**Acceptance Criteria**:
- When connectivity is restored, queued writes are replayed to the API in order
- Successfully synced items are removed from the outbox
- Failed sync stops and retries on next connectivity event
- Zustand store reflects pending sync count and syncing state
- Background Sync API is used when available

**Complexity**: L

---

### P7-T6: Build SyncStatus UI component

**Description**: Create `frontend/src/components/sync-status.tsx` — displays the pending sync count in the navigation bar. Shows a small badge or indicator when there are pending operations. Shows a toast notification when sync completes successfully. Reads from the Zustand store (`pendingSyncCount`, `isSyncing`). Add the component to `frontend/src/components/app-nav.tsx`.

**Dependencies**: P2-T4, P7-T5

**Acceptance Criteria**:
- Sync status indicator appears in the nav bar when there are pending operations
- Badge shows the count of pending sync items
- Spinning/loading indicator shown while syncing
- Toast notification shown when all items are synced
- Indicator disappears when outbox is empty

**Complexity**: S

---

### P7-T7: Handle offline brew creation with client-side nanoid

**Description**: Update the create brew flow to work offline: when offline, generate a `nanoid` client-side for the new brew ID, save the brew to IndexedDB cache immediately (so it appears in the UI), and queue the API request in the outbox. Install `nanoid` in `frontend/`. Apply the same pattern to ingredient and event creation. Show a visual indicator on items that are pending sync.

**Dependencies**: P7-T3, P7-T4

**Acceptance Criteria**:
- Creating a brew offline generates a client-side nanoid
- The brew appears in the dashboard immediately from IndexedDB
- The create request is queued in the outbox
- When synced, the brew persists to the server
- Pending-sync items show a visual indicator (e.g., a sync icon)
- Same pattern works for ingredients and events

**Complexity**: L

---

### P7-T8: Add offline delete warning

**Description**: When a user attempts a delete operation while offline, show a warning dialog explaining that the delete will be queued and may conflict if the item is modified elsewhere before sync. Use shadcn/ui AlertDialog. If the user confirms, queue the delete in the outbox. Remove the item from the local IndexedDB cache immediately.

**Dependencies**: P7-T4

**Acceptance Criteria**:
- Delete operations while offline show a warning dialog
- Warning explains the operation will be queued
- On confirm, delete is queued in outbox and item removed from local cache
- UI updates immediately to reflect the deletion
- Queued delete syncs when connectivity is restored

**Complexity**: S

---

## Phase 8 — Deployment & Production Readiness

### P8-T1: Configure frontend production build

**Description**: Update `frontend/vite.config.ts` for production: set `base` path if needed, configure build output to `frontend/dist/`. Add `build` and `preview` scripts to `frontend/package.json`. Ensure environment variables `VITE_APP_URL` and `VITE_API_URL` are read correctly in production builds. Test the production build locally with `npm run build && npm run preview`.

**Dependencies**: P1-T2

**Acceptance Criteria**:
- `cd frontend && npm run build` produces optimized static files in `dist/`
- `npm run preview` serves the production build locally
- Environment variables are correctly embedded in the build
- No build errors or warnings

**Complexity**: S

---

### P8-T2: Configure API for serverless deployment

**Description**: Prepare the API for deployment to Cloudflare Workers (primary target). Ensure `api/wrangler.toml` is configured with: name, compatibility date, environment variables (`DATABASE_URL`, `API_KEY` as secrets). Add `deploy` script to `api/package.json` using `wrangler deploy`. Alternatively, configure for Vercel Functions or Netlify Functions with the appropriate config files. Add production build script.

**Dependencies**: P1-T6

**Acceptance Criteria**:
- `api/wrangler.toml` (or equivalent config) is properly configured
- `DATABASE_URL` and `API_KEY` are configured as secrets (not in code)
- `npm run deploy` deploys the API to the serverless provider
- API responds to requests after deployment

**Complexity**: M

---

### P8-T3: Configure CORS on the API

**Description**: Add CORS middleware to `api/src/index.ts` allowing requests from the frontend origin. Configure: `Access-Control-Allow-Origin` set to the frontend URL (from environment variable or hardcoded for production), `Access-Control-Allow-Methods` for GET, POST, PUT, PATCH, DELETE, OPTIONS, `Access-Control-Allow-Headers` for Content-Type and X-API-Key. Handle preflight `OPTIONS` requests. Use Hono's built-in CORS middleware if available.

**Dependencies**: P1-T6

**Acceptance Criteria**:
- API responds with correct CORS headers
- Preflight OPTIONS requests return 204 with correct headers
- Frontend can make cross-origin requests without CORS errors
- CORS origin is configurable (not hardcoded to localhost)

**Complexity**: S

---

### P8-T4: Deploy frontend to static host

**Description**: Deploy the frontend to Cloudflare Pages (or Netlify/Vercel). Configure the build command (`npm run build`), output directory (`dist/`), and environment variables (`VITE_APP_URL`, `VITE_API_URL`). Set up the deployment pipeline (manual or CI). Verify the app loads at the public URL.

**Dependencies**: P8-T1

**Acceptance Criteria**:
- Frontend is live at a public URL
- App loads and renders correctly
- Environment variables point to the production API
- Static assets are served with appropriate caching headers

**Complexity**: M

---

### P8-T5: Deploy API to serverless provider

**Description**: Deploy the API to Cloudflare Workers (or Vercel/Netlify Functions). Set `DATABASE_URL` and `API_KEY` as secrets/environment variables on the provider. Verify the health-check endpoint responds. Test CRUD operations from the deployed frontend.

**Dependencies**: P8-T2, P8-T3

**Acceptance Criteria**:
- API is live and responds to requests
- `GET /api/health` returns 200
- CRUD operations work end-to-end from the deployed frontend
- Secrets are properly configured (not exposed in logs or responses)

**Complexity**: M

---

### P8-T6: Create .env.example files for frontend and API

**Description**: Create `frontend/.env.example` documenting:
```
VITE_APP_URL=http://localhost:5173
VITE_API_URL=http://localhost:8787
```

Create `api/.env.example` documenting:
```
DATABASE_URL=postgresql://user:password@localhost:5432/brewlog
API_KEY=your-secret-api-key-here
```

Ensure `.env` files are in `.gitignore`.

**Dependencies**: P1-T1

**Acceptance Criteria**:
- Both `.env.example` files exist with all required variables documented
- `.env` files are gitignored
- Variable names match what the code expects

**Complexity**: S

---

### P8-T7: Write README.md with setup and deployment guide

**Description**: Create `README.md` at the project root with:
- Project overview and features
- Tech stack summary
- Prerequisites (Node.js, PostgreSQL)
- Local development setup instructions (clone, install, configure .env, run migrations, start dev servers)
- Environment variable reference table
- Deployment guide for frontend (Cloudflare Pages/Netlify/Vercel)
- Deployment guide for API (Cloudflare Workers/Vercel Functions/Netlify Functions)
- Database setup instructions
- QR code workflow explanation

**Dependencies**: P8-T4, P8-T5, P8-T6

**Acceptance Criteria**:
- README covers all sections listed above
- Instructions are clear enough for a developer to set up the project from scratch
- Environment variables are fully documented
- Deployment steps are specific and actionable

**Complexity**: M

---

### P8-T8: End-to-end verification in production

**Description**: Perform a full end-to-end test of the deployed application:
1. Create a brew via the frontend
2. Add ingredients to the brew
3. Log fermentation events (gravity, temperature)
4. View the gravity chart
5. Generate and view the QR code
6. Open the QR URL on a mobile device (or emulator)
7. Use the quick-log form to add an event
8. Verify offline mode (disable network, view cached data, queue a write, re-enable and verify sync)
9. Delete a brew and verify cascade

Document any issues found and fix them.

**Dependencies**: P8-T4, P8-T5

**Acceptance Criteria**:
- All CRUD operations work in production
- QR codes encode the production URL correctly
- Quick-log flow works on mobile
- Offline caching serves previously viewed data
- Offline writes sync when connectivity is restored
- No console errors in production

**Complexity**: L

---

## Task Dependency Graph

```mermaid
flowchart TD
    P1T1[P1-T1: Monorepo root] --> P1T2[P1-T2: Frontend scaffold]
    P1T1 --> P1T6[P1-T6: API scaffold]
    P1T2 --> P1T3[P1-T3: Tailwind + shadcn]
    P1T2 --> P1T4[P1-T4: React Router]
    P1T2 --> P1T5[P1-T5: TanStack Query + Zustand]
    P1T2 --> P1T11[P1-T11: API client]
    P1T6 --> P1T7[P1-T7: Drizzle schema]
    P1T6 --> P1T9[P1-T9: Auth middleware]
    P1T6 --> P1T10[P1-T10: Zod validators]
    P1T7 --> P1T8[P1-T8: First migration]

    P1T7 & P1T8 & P1T9 & P1T10 --> P2T1[P2-T1: Brews API]
    P1T5 --> P2T2[P2-T2: Types + utils]
    P1T11 & P2T2 --> P2T3[P2-T3: Brew query hooks]
    P1T3 & P1T4 --> P2T4[P2-T4: RootLayout + AppNav]
    P1T3 & P2T2 --> P2T5[P2-T5: BrewForm]
    P1T3 & P2T2 --> P2T6[P2-T6: BrewCard + badges]
    P2T3 & P2T4 & P2T6 --> P2T7[P2-T7: Dashboard page]
    P2T3 & P2T5 --> P2T8[P2-T8: Create brew page]
    P1T3 & P1T4 & P2T3 --> P2T9[P2-T9: BrewDetailLayout]
    P2T2 & P2T3 & P2T9 --> P2T10[P2-T10: Brew overview page]
    P2T3 & P2T5 & P2T9 --> P2T11[P2-T11: Edit brew page]
    P1T5 & P1T11 & P2T4 --> P2T12[P2-T12: Auth prompt]
    P2T7 & P2T8 & P2T9 & P2T10 & P2T11 --> P2T13[P2-T13: Route registration]

    P1T7 & P1T8 & P1T9 & P1T10 --> P3T1[P3-T1: Ingredients API]
    P1T11 & P2T2 --> P3T2[P3-T2: Ingredient hooks]
    P1T3 & P2T2 --> P3T3[P3-T3: IngredientForm]
    P1T3 & P3T3 --> P3T4[P3-T4: IngredientTable]
    P2T9 & P2T13 & P3T2 & P3T3 & P3T4 --> P3T5[P3-T5: Ingredients page]

    P1T7 & P1T8 & P1T9 & P1T10 --> P4T1[P4-T1: Events API]
    P1T11 & P2T2 --> P4T2[P4-T2: Event hooks]
    P1T3 & P2T2 --> P4T3[P4-T3: EventForm]
    P1T3 & P2T2 --> P4T4[P4-T4: EventTimeline]
    P1T3 & P2T2 --> P4T5[P4-T5: GravityChart]
    P2T9 & P2T13 & P4T2 & P4T3 & P4T4 & P4T5 --> P4T6[P4-T6: Log page]
    P2T10 & P4T2 --> P4T7[P4-T7: BrewSummary update]

    P1T3 --> P5T1[P5-T1: QrCodeDisplay]
    P5T1 --> P5T2[P5-T2: QR download + print]
    P2T9 & P2T13 & P5T1 & P5T2 --> P5T3[P5-T3: QR page]
    P1T3 & P2T2 & P4T3 --> P5T4[P5-T4: QuickLogForm]
    P2T3 & P2T10 & P2T12 & P4T2 & P5T4 --> P5T5[P5-T5: Quick-log page]

    P1T3 & P2T2 --> P6T1[P6-T1: StatusFilter]
    P2T7 & P6T1 --> P6T2[P6-T2: Dashboard filtering]
    P2T6 & P2T2 --> P6T3[P6-T3: BrewCard polish]
    P1T3 & P1T4 --> P6T4[P6-T4: 404 page]
    P2T7 & P2T10 & P3T5 & P4T6 & P5T3 --> P6T5[P6-T5: Loading skeletons]
    P2T7 & P2T10 & P3T5 & P4T6 --> P6T6[P6-T6: Error boundaries]
    P2T8 & P2T10 & P2T11 & P3T5 & P4T6 & P5T5 --> P6T7[P6-T7: Toast notifications]
    P2T4 & P2T7 & P2T10 & P3T5 & P4T6 & P5T3 & P5T5 --> P6T8[P6-T8: Responsive polish]

    P1T2 --> P7T1[P7-T1: PWA config]
    P1T5 --> P7T2[P7-T2: IndexedDB cache]
    P2T3 & P3T2 & P4T2 & P7T2 --> P7T3[P7-T3: Query + cache integration]
    P1T11 & P7T2 --> P7T4[P7-T4: Outbox queue]
    P7T1 & P7T4 --> P7T5[P7-T5: Background sync]
    P2T4 & P7T5 --> P7T6[P7-T6: SyncStatus UI]
    P7T3 & P7T4 --> P7T7[P7-T7: Offline creation]
    P7T4 --> P7T8[P7-T8: Offline delete warning]

    P1T2 --> P8T1[P8-T1: Frontend build]
    P1T6 --> P8T2[P8-T2: API deployment config]
    P1T6 --> P8T3[P8-T3: CORS config]
    P8T1 --> P8T4[P8-T4: Deploy frontend]
    P8T2 & P8T3 --> P8T5[P8-T5: Deploy API]
    P1T1 --> P8T6[P8-T6: .env.example files]
    P8T4 & P8T5 & P8T6 --> P8T7[P8-T7: README]
    P8T4 & P8T5 --> P8T8[P8-T8: E2E verification]
```

---

## Summary

| Phase | Tasks | S | M | L |
|-------|-------|---|---|---|
| Phase 1 — Project Scaffold & DB Schema | 11 | 5 | 6 | 0 |
| Phase 2 — Brews API & Core Brew UI | 13 | 5 | 6 | 1 (P2-T1) |
| Phase 3 — Ingredients API & UI | 5 | 2 | 3 | 0 |
| Phase 4 — Fermentation Events API & UI | 7 | 2 | 5 | 0 |
| Phase 5 — QR Codes & Quick-Log | 5 | 2 | 3 | 0 |
| Phase 6 — Dashboard Polish & Filtering | 8 | 3 | 4 | 0 (P6-T7 is S) |
| Phase 7 — Offline Support & PWA | 8 | 2 | 3 | 3 |
| Phase 8 — Deployment & Production | 8 | 3 | 4 | 1 (P8-T8) |
| **Total** | **65** | **24** | **34** | **5** |
