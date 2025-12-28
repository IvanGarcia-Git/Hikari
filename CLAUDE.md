# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

```bash
# Development
pnpm dev                    # Start Next.js dev server
pnpm build                  # Production build
pnpm lint                   # Run ESLint
pnpm prettier-fix           # Format code

# Supabase (requires Docker)
pnpm supabase:start         # Start local Supabase
pnpm supabase:stop          # Stop local Supabase
pnpm supabase:reset         # Reset database to migrations
pnpm supabase:generate-types # Generate TypeScript types → types_db.ts

# Stripe (requires Stripe CLI)
pnpm stripe:login           # Authenticate Stripe CLI
pnpm stripe:listen          # Forward webhooks to localhost:3000/api/webhooks
pnpm stripe:fixtures        # Load test products/prices
```

## Architecture Overview

### Next.js App Router Structure
- `app/(auth_forms)/` - Auth pages (signin, signup, magic_link, forgot_password)
- `app/(dashboard)/dashboard/` - Protected dashboard with account/settings
- `app/(marketing)/` - Public pages (landing, pricing)
- `app/api/` - API routes including tRPC endpoint and Stripe webhooks
- `app/docs/[[...slug]]/` - Fumadocs documentation
- `app/blog/` - MDX blog

### Key Directories
- `server/api/` - tRPC routers and context (backend logic lives here)
- `trpc/` - tRPC client setup (react.tsx for hooks, server.ts for Server Components)
- `utils/supabase/` - Supabase clients (client.ts for browser, server.ts for SSR)
- `utils/stripe/` - Stripe server actions (checkout, portal)
- `supabase/migrations/` - Database schema with RLS policies
- `content/docs/` and `content/blog/` - MDX content

### Data Flow Patterns

**Supabase Clients:**
- Browser: `import { createClient } from '@/utils/supabase/client'`
- Server (SSR/RSC): `import { createClient } from '@/utils/supabase/server'`
- Types auto-generated to `types_db.ts`

**tRPC:**
- Define routers in `server/api/routers/`
- Register in `server/api/root.ts`
- Client hooks: `import { api } from '@/trpc/react'`
- Server Components: `import { api } from '@/trpc/server'`

**Auth Pattern:**
```typescript
// In Server Components
const supabase = createClient()
const { data: { user } } = await supabase.auth.getUser()
if (!user) redirect('/signin')
```

**Stripe Webhooks → Supabase:**
Webhook at `app/api/webhooks/stripe/route.ts` syncs products, prices, and subscriptions to Supabase via `utils/supabase/admin.ts`.

### RLS (Row Level Security)
All tables use RLS. Common pattern:
```sql
CREATE POLICY "Users can only access own data" ON table_name
  FOR ALL USING (auth.uid() = user_id)
```

### Multi-Tenant Ready
Current setup is user-scoped. For org-based multi-tenancy, add `organization_id` column and update RLS policies.

## Tech Stack Constraints

**Use:**
- Next.js App Router with Server Components by default
- Supabase client directly (no ORMs)
- shadcn/ui for components
- React Hook Form + Zod for forms
- tRPC for type-safe API

**Avoid:**
- Redux/Zustand (use Server Components)
- Prisma/Drizzle (use Supabase client)
- Pages Router
