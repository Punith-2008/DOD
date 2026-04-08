# DoD — Diesel On Delivery

## Overview

Diesel delivery platform where customers order diesel and delivery agents deliver fuel to their location. Includes an admin panel for managing orders, delivery agents, customers, and analytics, plus a customer-facing website for order placement and tracking.

## Stack

- **Monorepo tool**: pnpm workspaces
- **Node.js version**: 24
- **Package manager**: pnpm
- **TypeScript version**: 5.9
- **Frontend**: React + Vite + Tailwind CSS + shadcn/ui
- **API framework**: Express 5
- **Database**: PostgreSQL + Drizzle ORM
- **Authentication**: Clerk (auto-provisioned)
- **AI**: OpenAI via Replit AI Integrations (gpt-5.2)
- **Validation**: Zod (`zod/v4`), `drizzle-zod`
- **API codegen**: Orval (from OpenAPI spec)
- **Build**: esbuild (CJS bundle for server), Vite (frontend)

## Architecture

### Admin Frontend (`artifacts/diesel-app`)
- React + Vite web app at root path `/`
- Clerk authentication with Google + email sign-in
- Pages: Landing, Dashboard, Orders (list/new/detail), Delivery Agents, Customers (list/detail), Analytics, Settings
- Uses generated React Query hooks from `@workspace/api-client-react`
- Custom API client (`src/lib/api.ts`) for delivery agent endpoints not in the generated spec

### Customer Portal (`artifacts/customer-portal`)
- React + Vite web app at `/customer/`
- Clerk authentication for customer self-service
- Pages: Landing (marketing), Place Order, My Orders, Order Detail, Profile
- Uses generated React Query hooks for `/me/*` API endpoints
- Customers auto-created on first login via `clerkUserId` field

### Backend (`artifacts/api-server`)
- Express 5 API at `/api`
- Routes: customers, orders, agents, dashboard, AI, health, portal (/me/*)
- Portal routes: /me/profile, /me/orders, /me/orders/:id (customer self-service)
- Clerk middleware for auth (`requireAuth()` on all non-health endpoints)
- `requireAdmin` middleware on admin routes (customers, orders, agents, dashboard, AI)
- Pino structured logging

### Database (`lib/db`)
- Tables: customers (with clerk_user_id), orders, delivery_agents, customer_notes, activity_log
- `delivery_agents` table: id, name, phone, vehicle_number, vehicle_type, status (available/on_delivery/off_duty), active_deliveries, total_deliveries
- Orders have `assigned_agent_id` FK to delivery_agents
- Drizzle ORM with PostgreSQL

### Delivery Workflow
1. Customer places order (pending)
2. Admin confirms order and assigns delivery agent (confirmed)
3. Agent picks up fuel and starts delivery (in_transit)
4. Agent delivers to customer location (delivered)
- Agent counters (active_deliveries, total_deliveries) are updated automatically on assignment, delivery completion, and cancellation

### AI Features (`artifacts/api-server/src/routes/ai`)
- Customer insights — AI-generated analysis of customer behavior
- Order predictions — predicted next order quantity/date based on history
- Anomaly detection — flags unusual order patterns

### Shared Libraries
- `lib/api-spec` — OpenAPI spec (source of truth)
- `lib/api-client-react` — Generated React Query hooks
- `lib/api-zod` — Generated Zod validation schemas
- `lib/integrations-openai-ai-server` — OpenAI SDK integration

## Authorization

- Admin routes (`/customers`, `/orders`, `/agents`, `/dashboard`, `/ai`) require `role: "admin"` in Clerk `publicMetadata` (or no role set for backward compat)
- Customer portal routes (`/me/*`) are scoped to the authenticated customer's own data
- New customers signing up via the portal are auto-assigned `role: "customer"` in Clerk
- `ADMIN_CLERK_USER_IDS` env var can be set as a comma-separated allowlist for admin access

## Key Commands

- `pnpm run typecheck` — full typecheck across all packages
- `pnpm run build` — typecheck + build all packages
- `pnpm --filter @workspace/api-spec run codegen` — regenerate API hooks and Zod schemas from OpenAPI spec
- `pnpm --filter @workspace/db run push` — push DB schema changes (dev only)
- `pnpm --filter @workspace/api-server run dev` — run API server locally
- `pnpm --filter @workspace/diesel-app run dev` — run frontend locally

## Authorization

- Admin routes (`/customers`, `/orders`, `/dashboard`, `/ai`) require `role: "admin"` in Clerk `publicMetadata`
- Customer portal routes (`/me/*`) are scoped to the authenticated customer's own data
- New customers signing up via the portal are auto-assigned `role: "customer"` in Clerk
- Admin users must have `publicMetadata.role = "admin"` set in Clerk dashboard

## Environment Variables (Auto-Provisioned)

- `DATABASE_URL` — PostgreSQL connection string
- `CLERK_SECRET_KEY`, `CLERK_PUBLISHABLE_KEY`, `VITE_CLERK_PUBLISHABLE_KEY` — Clerk auth
- `AI_INTEGRATIONS_OPENAI_BASE_URL`, `AI_INTEGRATIONS_OPENAI_API_KEY` — OpenAI integration

### Mobile App (`artifacts/fuelflow-mobile`)
- Expo SDK 54 / React Native mobile app at `/mobile/`
- 5-tab layout: Dashboard, Orders, Customers, Analytics, Settings
- Clerk authentication (custom sign-in/sign-up screens)
- Full CRUD: create/view orders, create/view customers, add notes
- AI insights on customer detail screen (behavioral insight + order prediction)
- Dark/amber fuel-industry color scheme with Inter typography
- Uses generated React Query hooks from `@workspace/api-client-react`
- Haptic feedback on key actions (status updates, form submissions)

## Key Commands

### Mobile
- `pnpm --filter @workspace/fuelflow-mobile run dev` — run Expo mobile app

## Seeded Data

- 3 delivery agents: Ravi Patel (available), Suresh Yadav (on_delivery), Amit Joshi (available)
- 3 customers with 6 orders across various statuses

See the `pnpm-workspace` skill for workspace structure, TypeScript setup, and package details.
