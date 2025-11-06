# Technology Stack

## Architecture Overview

This is a modern, full-stack web application built with server-side rendering (SSR) on Cloudflare Workers. The architecture follows a distributed approach where:

- **Frontend**: React 19 with TypeScript, served as SSR from Cloudflare Workers
- **Backend**: Hono framework running on Cloudflare Workers with Node.js compatibility
- **Database**: Convex (real-time database as a service)
- **Authentication**: Clerk (external auth provider with JWT integration)
- **Hosting**: Cloudflare Workers (edge computing platform)

The application uses file-system based routing with React Router 7, enabling a file-first development experience similar to Next.js.

```
┌─────────────────────────────────────────┐
│      Browser / Client Application       │
└──────────────┬──────────────────────────┘
               │
    ┌──────────▼──────────┐
    │ React Router 7 SSR  │
    │  (Edge Runtime)     │
    └──────────┬──────────┘
               │
    ┌──────────▼──────────────┐      ┌─────────────┐
    │  Cloudflare Workers     │──────│   Convex    │
    │  (Hono Framework)       │      │  Database   │
    └──────────┬──────────────┘      └─────────────┘
               │
    ┌──────────▼──────────┐
    │ Clerk Auth Service  │
    └─────────────────────┘
```

## Frontend Stack

### Framework & Runtime
- **React**: 19.0.0 (latest, with concurrent rendering)
- **React Router**: 7.4.1 (file-system based routing with SSR)
- **TypeScript**: 5.7.2 (strict mode enforced)
- **Module System**: ES Modules

### Build Tools
- **Vite**: 6.3.5 (dev server and build tool)
- **Vite TsConfig Paths**: 5.1.4 (TypeScript path alias support)
- **Cloudflare Vite Plugin**: 1.3.1 (Cloudflare Workers integration)

### UI Components & Styling
- **ShadCN/UI**: Component library built on Radix UI
- **Radix UI**: 1.x (headless component primitives)
  - Dialog, Dropdown Menu, Select, Tooltip, Checkbox, Avatar, Scroll Area, Separator, Slot
- **Tailwind CSS**: 4.0.0 (utility-first CSS framework)
- **Tailwind CSS Vite Plugin**: 4.0.0
- **Tailwind CSS Typography**: 0.5.16 (prose styling)
- **Tailwind Merge**: 3.1.0 (class merging utility)
- **tw-animate-css**: 1.2.5 (animation utilities)

### State Management & Data
- **TanStack React Query**: 5.80.7 (server state management)
- **TanStack React Query DevTools**: 5.80.7 (debugging)
- **React Hook Form**: 7.58.0 (form state management)
- **Zod**: 3.25.62 (schema validation)

### Theme Management
- **Next Themes**: 0.4.6 (dark/light mode support)

### Animations & Effects
- **Framer Motion**: 12.6.3 (motion and animation library)
- **Sonner**: 2.0.5 (toast notifications)

### Utilities & Icons
- **Lucide React**: 0.486.0 (icon library)
- **Clsx**: 2.1.1 (conditional classname utility)
- **Class Variance Authority**: 0.7.1 (component variant management)

## Backend Stack

### Runtime & Framework
- **Cloudflare Workers**: Edge computing platform (no cold starts)
- **Hono**: 4.7.5 (lightweight web framework for Workers)
- **React Router Cloudflare**: 7.4.1 (SSR adapter for Cloudflare)
- **Wrangler**: 4.34.0 (Cloudflare CLI and development tool)

### Database & Persistence
- **Convex**: 1.17.4 (real-time database as a service)
  - Replaces traditional database setup (PostgreSQL, MongoDB, etc.)
  - Built-in WebSocket subscriptions for real-time features
  - Schema defined in TypeScript
  - Current schema includes: `users` table with email, name, clerkId, imageUrl, roles, timestamps

### Authentication & Authorization
- **Clerk**: 2.1.6 (external authentication provider)
- **@clerk/react-router**: Latest (React Router integration)
- **JWT-based Convex Integration**: Custom auth.ts helpers for Convex auth
- **Role-Based Access Control**: Three-tier system (user, admin, superadmin)

### Email Service
- **Resend**: 4.5.1 (transactional email delivery)

## Infrastructure

### Hosting & Deployment
- **Cloudflare Workers**: Primary hosting platform
  - Compatibility: nodejs_compat for Node.js compatibility
  - Compatibility Date: 2025-09-01
  - Main entry point: `./workers/app.ts`

### Environments
1. **Development** (`worktracking-app`)
   - DEV_MODE: true
   - Full debugging capabilities

2. **Staging** (`worktracking-app-staging`)
   - DEV_MODE: false
   - Pre-production testing

3. **Production** (`worktracking-app-prod`)
   - DEV_MODE: false
   - Stability and performance optimized

### Configuration
- **Wrangler Configuration**: `wrangler.jsonc`
- **Environment Variables**: Managed via Wrangler config and `.env` files
- **Secrets**: Managed through Wrangler secrets manager

### Environment Variables
- `ENVIRONMENT`: Development/staging/production flag
- `CLERK_JWT_ISSUER_DOMAIN`: Clerk domain URL
- `CLERK_SECRET_KEY`: Server-side Clerk secret
- `VITE_CLERK_PUBLISHABLE_KEY`: Client-side Clerk key
- `VITE_CLERK_TEST_EMAIL`: Test user email (development)
- `VITE_CLERK_TEST_PASSWORD`: Test user password (development)
- `FROM_EMAIL`: Email sender address (Resend)
- `FROM_NAME`: Email sender name
- `SUPPORT_EMAIL`: Support contact email

## Development Tools

### Code Quality & Linting
- **Biome**: 1.9.4 (unified linter and formatter)
  - Linting with recommended rules
  - Auto-formatting enforced
  - Tab-based indentation (2 tabs)
  - Double quotes for strings
  - Import organization enabled
  - Pre-commit integration via Husky

### Type Checking
- **TypeScript**: 5.7.2 (strict mode)
  - Strict type checking enabled
  - Path aliases for clean imports
  - Type generation from Convex and Cloudflare

### Git Hooks
- **Husky**: 9.1.7 (git hook management)
- Pre-commit hooks for code quality checks

### Package Management
- **Bun**: Runtime and package manager
  - Uses bun.lock (lockfile v1)
  - Faster than npm/pnpm
  - Built-in test runner and bundler

### Build Scripts
```json
{
  "dev": "react-router dev",                         // Local development
  "build": "react-router build",                     // Build for production
  "build:staging": "CLOUDFLARE_ENV=staging ...",     // Staging build
  "build:prod": "CLOUDFLARE_ENV=production ...",     // Production build
  "deploy": "bun run build && wrangler deploy",      // Deploy to Cloudflare
  "deploy:staging": "bun run build:staging && ...",  // Deploy to staging
  "deploy:prod": "bun run build:prod && ...",        // Deploy to production
  "start": "wrangler dev",                           // Local Worker dev server
  "check": "Runs all quality checks",                // Code quality validation
  "biome:check": "Lint and format code",             // Format and lint
  "convex:dev": "Convex development mode",           // Backend dev server
  "convex:deploy": "Deploy Convex functions"         // Deploy backend
}
```

## Project Structure

```
/template
├── /app                          # Frontend React application
│   ├── /routes                   # File-system based routes (React Router)
│   ├── /components
│   │   ├── /ui                  # ShadCN/UI components
│   │   ├── /shared              # Shared components
│   │   ├── /layouts             # Layout components
│   │   └── /features            # Feature-specific components
│   ├── /hooks                   # Custom React hooks
│   ├── /contexts                # React contexts (if any)
│   ├── /lib                     # Utility functions and helpers
│   │   └── permissions.ts       # RBAC permission checking
│   ├── /types                   # TypeScript type definitions
│   ├── entry.server.tsx         # Server-side entry point (SSR)
│   ├── root.tsx                 # Root layout component
│   ├── tailwind.css             # Global styles
│   └── tailwind.config.ts       # Tailwind customization
├── /convex                      # Backend functions and database schema
│   ├── schema.ts                # Convex database schema definition
│   ├── auth.ts                  # Authentication helpers
│   ├── users.ts                 # User-related functions
│   ├── http.ts                  # HTTP configuration
│   └── _generated/              # Auto-generated Convex types
├── /workers                     # Cloudflare Workers code
│   ├── app.ts                   # Main Worker entry point
│   └── types.ts                 # Worker types and RBAC permissions
├── /public                      # Static assets (images, favicon, etc.)
├── Configuration Files
│   ├── package.json             # Project metadata and dependencies
│   ├── bun.lock                 # Bun lockfile
│   ├── vite.config.ts           # Vite configuration
│   ├── react-router.config.ts   # React Router SSR configuration
│   ├── tsconfig.json            # Base TypeScript configuration
│   ├── tsconfig.cloudflare.json # Cloudflare-specific TypeScript config
│   ├── biome.json               # Biome linter/formatter config
│   ├── wrangler.jsonc           # Cloudflare Workers configuration
│   ├── components.json          # ShadCN components configuration
│   ├── tailwind.config.ts       # Tailwind CSS customization
│   └── .env.example             # Environment variable template
├── Git Hooks
│   └── .husky/                  # Pre-commit hooks
└── Generated Files (ignore)
    ├── .react-router/           # React Router generated types
    ├── .wrangler/               # Wrangler cache
    └── worker-configuration.d.ts # Auto-generated Worker types
```

## Key Architectural Decisions

### Why Cloudflare Workers?
- **Zero Cold Starts**: Requests execute immediately on edge servers
- **Global Distribution**: Automatic geographic distribution
- **Cost Effective**: Pay only for what you use
- **Integrated Ecosystem**: Seamless integration with Cloudflare services
- **Developer Experience**: Wrangler CLI makes local development straightforward

### Why Convex Instead of Traditional Database?
- **Real-time Subscriptions**: Built-in WebSocket support for reactive data
- **Serverless**: No database server to manage
- **Type-Safe Queries**: TypeScript types generated from schema
- **Zero Configuration**: Automatic backups and scaling
- **Reduced Complexity**: Fewer moving parts than traditional stacks

### Why React Router 7?
- **File-System Based Routing**: Familiar, intuitive routing similar to Next.js
- **Server-Side Rendering**: Can run on Cloudflare Workers
- **Type Safety**: Full TypeScript support with automatic type generation
- **Modern React**: Leverages React 19 features and patterns
- **Smaller Bundle**: Lighter than Next.js

### Why ShadCN/UI + Tailwind?
- **Copy-Paste Components**: Full control over component code
- **Design System**: Pre-built accessible components
- **Utility-First CSS**: Fast styling with minimal custom CSS
- **Easy Customization**: Easy to modify components and theme
- **Tailwind Ecosystem**: Vast plugin and component ecosystem

### Why Biome?
- **Single Tool**: Linting and formatting in one tool
- **Performance**: Written in Rust, incredibly fast
- **Opinionated**: Reduces configuration decisions
- **Active Development**: Modern tooling matching current best practices

## Technology Trade-offs

### Cloudflare Workers vs Traditional Server Hosting
| Aspect | Workers | Traditional (Node.js) |
|--------|---------|----------------------|
| **Startup Time** | ~10ms | 1-30s+ (cold start) |
| **Cost** | Pay per request | Fixed server cost |
| **Scalability** | Automatic | Manual/autoscaling |
| **Limitations** | CPU timeout, memory limits | Full system access |
| **Debugging** | Limited local parity | Full local parity |

**Decision**: Workers chosen for global distribution and cost efficiency, accepting CPU/memory trade-offs.

### Convex vs PostgreSQL + Traditional ORM
| Aspect | Convex | PostgreSQL |
|--------|--------|-----------|
| **Real-time** | Built-in | Requires polling/WebSocket setup |
| **Serverless** | Yes | No (server management required) |
| **Scaling** | Automatic | Manual |
| **Flexibility** | Schema-based | Unlimited queries |
| **Cost** | Usage-based | Fixed infrastructure |

**Decision**: Convex chosen for real-time capabilities and serverless simplicity, accepting less query flexibility.

### React Router vs Next.js
| Aspect | React Router | Next.js |
|--------|--------------|---------|
| **File Routing** | Yes | Yes |
| **Bundle Size** | Lighter | Larger |
| **Complexity** | Simpler | More features |
| **Ecosystem** | Growing | Massive |
| **Edge Runtime** | Works everywhere | Vercel optimized |

**Decision**: React Router chosen for simplicity and lighter bundle, accepting smaller ecosystem.

### Clerk vs Self-Hosted Auth
| Aspect | Clerk | Self-Hosted |
|--------|-------|------------|
| **Setup Time** | Minutes | Weeks |
| **Maintenance** | None | High |
| **Cost** | Per user | Fixed infrastructure |
| **Control** | Limited | Full |
| **Security** | Enterprise grade | Depends on implementation |

**Decision**: Clerk chosen for rapid development and reduced security responsibility.

## Alternatives Considered

### Frontend Frameworks
- **Next.js**: Rejected for being overly complex for this use case; React Router provides sufficient features with less overhead
- **Svelte/SvelteKit**: Not chosen due to smaller ecosystem and less team familiarity
- **Astro**: Good alternative but less suitable for interactive app requirements

### Backend/Database
- **Supabase**: Good alternative to Convex, but Convex's real-time by default was preferred
- **Firebase**: More expensive and less suitable for server-side rendering on Workers
- **Traditional PostgreSQL + Prisma**: Would add infrastructure management overhead

### Styling
- **Styled Components / Emotion**: Rejected for heavier bundle size compared to Tailwind
- **CSS Modules**: Rejected for less maintainability than utility-first approach
- **Material UI**: More opinionated and heavier than ShadCN/UI

### Linting/Formatting
- **ESLint + Prettier**: Would work but Biome offers better performance as a unified tool
- **Deno Lint**: Not chosen due to less TypeScript integration

## Deployment Architecture

### Development Environment
```
npm run dev
    ↓
React Router dev server (localhost:5173)
    ↓
Local Convex deployment
    ↓
Clerk development instance
```

### Staging Environment
```
bun run build:staging && wrangler deploy
    ↓
Vite builds React Router app for Cloudflare
    ↓
Deployed to worktracking-app-staging.workers.dev
    ↓
Staging Convex deployment
    ↓
Staging Clerk application
```

### Production Environment
```
bun run build:prod && wrangler deploy --env production
    ↓
Vite builds optimized production bundle
    ↓
Deployed to production domain via Cloudflare
    ↓
Production Convex deployment
    ↓
Production Clerk application
```

## Type Safety & Code Quality

### TypeScript Configuration
- **Strict Mode**: Enabled (all strict options active)
- **No Implicit Any**: Enforced at linter level
- **No Unused Locals**: Enforced by Biome
- **Module Resolution**: Node
- **Target**: ES2022

### Code Standards Enforced
- **Tabs**: Must use tabs (2-width) for indentation
- **Quotes**: Double quotes for all strings
- **Imports**: Auto-organized and sorted
- **Formatting**: Automatic via pre-commit hooks
- **Unused Code**: Automatically detected and flagged

### Auto-Generated Types
- **Convex**: Types auto-generated from schema in `convex/_generated/`
- **Cloudflare Workers**: Types generated from bindings in `worker-configuration.d.ts`
- **React Router**: Types generated in `.react-router/types/`

## Performance Characteristics

### Bundle Size Targets
- React Router framework intentionally kept lightweight
- Tailwind CSS compiled with PurgeCSS
- ShadCN components tree-shakeable
- Biome for aggressive dead code removal

### Runtime Performance
- **Edge Locations**: Cloudflare's 300+ global edge servers
- **Caching**: Cloudflare's built-in cache for static assets
- **Database**: Convex optimizes queries with indexing
- **Type Generation**: Compile-time, zero runtime overhead

## Security Considerations

### Authentication Flow
```
User → Clerk UI → Clerk API → JWT Token → Browser
        ↓
        Convex Auth Helpers → Verify JWT → Grant Access
```

### Authorization
- **RBAC**: Three-level role system (user, admin, superadmin)
- **Worker Level**: Permissions checked at Hono middleware
- **Client Level**: Permissions checked in React components
- **Database Level**: Convex auth enforces data access

### Environment Isolation
- Development secrets isolated from staging/production
- Clerk test mode for development
- Wrangler manages secret rotation

## Monitoring & Debugging

### Development
- **Vite HMR**: Hot module reloading for instant feedback
- **React Query DevTools**: Browser extension for state inspection
- **Biome Diagnostics**: Real-time linting feedback
- **Source Maps**: Full TypeScript source mapping

### Production
- **Wrangler Tail**: Real-time log streaming from Workers
- **Convex Dashboard**: Query and function monitoring
- **Clerk Analytics**: Authentication metrics
- **Error Tracking**: Integration ready (e.g., Sentry)

## Maintenance & Updates

### Dependency Management
- Bun for fast package resolution
- Regular security updates via npm audit
- Gradual upgrade path for major versions

### Code Formatting
- Pre-commit hooks ensure consistent formatting
- Biome auto-fixes most issues
- CI checks prevent unformatted code

### Version Pinning
- Package.json uses semantic versioning ranges
- bun.lock provides reproducible builds
- Regular dependency audits

## Future Scalability

### Horizontal Scaling
- Cloudflare Workers: Automatic (managed by Cloudflare)
- Convex: Automatic (managed by Convex)
- No infrastructure scaling concerns

### Vertical Optimization
- Worker CPU limits may require optimization of heavy computations
- Convex query optimization for large result sets
- Consider caching layer if needed

### Feature Expansion
- Easy to add new components via ShadCN CLI
- Convex schema easily extensible
- TypeScript ensures type safety during expansion
