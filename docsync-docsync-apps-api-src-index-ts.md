# docsync/docsync/apps/api/src/index.ts

# index.ts

## Overview

Main entry point for the DocSync API server. Configures Express application with security middleware, registers all API route handlers, and starts the HTTP server. Handles webhook routes separately with raw body parsing to support signature verification.

## Exports

### `supabase`

Re-exported Supabase client instance for database operations.

**Type:** `SupabaseClient`

**Usage:**
```typescript
import { supabase } from './index'

const { data, error } = await supabase
  .from('users')
  .select('*')
```

### `app` (default export)

Configured Express application instance.

**Type:** `Express.Application`

**Usage:**
```typescript
import app from './index'

// Use in tests or for server composition
const server = app.listen(4000)
```

## Application Configuration

### Middleware Stack

Applied in specific order:

1. **Webhook Route** (`/webhook`) - Registered FIRST before other middleware
   - Uses `express.raw()` to preserve raw request body
   - Required for payment provider signature verification

2. **Security & Parsing Middleware** - Applied to all other routes
   - `helmet()` - Sets secure HTTP headers
   - `cors()` - Enables cross-origin requests
   - `express.json()` - Parses JSON request bodies

### Routes

| Path | Handler | Description |
|------|---------|-------------|
| `/health` | GET | Health check endpoint |
| `/privacy` | GET | Privacy policy page |
| `/terms` | GET | Terms of service page |
| `/auth` | authRoutes | Authentication endpoints |
| `/docs` | docsRoutes | Document management |
| `/auth/google` | googleRoutes | Google OAuth flow |
| `/billing` | billingRoutes | Billing operations |
| `/webhook` | webhookRoutes | Payment webhooks (raw body) |
| `/projects` | projectRoutes | Project CRUD operations |
| `/auth/confluence` | confluenceRoutes | Confluence OAuth |
| `/auth/notion` | notionRoutes | Notion OAuth |
| `/auth/github` | githubRoutes | GitHub OAuth |
| `/email` | emailRoutes | Email operations |
| `/auth/linear` | linearRoutes | Linear OAuth |
| `/readme` | readmeRoutes | ReadMe integration |

## Environment Variables

- `PORT` - Server port (default: 3000)
- Additional variables loaded via `dotenv.config()` from `.env` file

## Server Initialization

Server starts on configured port and logs startup message to console.

```typescript
// Server starts automatically when module is imported
// Listens on process.env.PORT or 3000
```

## Notes

**Critical:** Webhook route must be registered before `express.json()` middleware. Webhook signature verification requires raw request body. Moving webhook registration after JSON parsing will break payment webhooks.

The queue service is initialized via side-effect import (`import './services/queue'`). Ensure queue configuration is complete before application starts.

Health check endpoint returns:
```json
{
  "status": "ok",
  "message": "DocSync API running"
}
```