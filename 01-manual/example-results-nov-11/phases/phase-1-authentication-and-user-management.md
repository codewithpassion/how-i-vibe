# Phase 1: Authentication & User Management - MVP Foundation

## Objective

Establish secure authentication, user management foundation, and basic admin dashboard to enable users to sign in and access the system with role-based permissions.

## Scope

### Features

- Complete Clerk authentication integration
- User sync from Clerk to Convex
- Role-based access control (user, admin, superadmin)
- User listing and management (admin only)
- Protected routes and permission enforcement
- User profile management
- Sign-in/sign-up pages
- Basic navigation and layout

### Frontend

- Login page with Clerk sign-in component
- Sign-up page with Clerk sign-up component
- Protected routes with role-based guards
- Main navigation header with user menu
- Dashboard landing page (authenticated users)
- Admin dashboard overview
- User list page (admin)
- User profile page
- Error pages (403 Forbidden, 404 Not Found)

### Backend

- User schema in Convex (clerkId, email, name, roles, timestamps)
- User sync mutation from Clerk webhooks
- User queries (getMe, listUsers, getUserById)
- Auth middleware for role verification
- Admin stats query (user counts by role)
- Permission helper functions

### Infrastructure

- Clerk webhook configuration for user sync
- Environment variables for Clerk integration
- Convex schema and indexes for users table
- Role permission mapping system
- Error handling and logging

## Success Criteria

- [ ] User can sign up via Clerk
- [ ] User can sign in via Clerk
- [ ] User is automatically synced to Convex database
- [ ] User profile displays correctly after login
- [ ] Admin can view list of all users
- [ ] Non-admin users cannot access admin routes (shows 403)
- [ ] Admin users can access admin dashboard
- [ ] Users can log out and session is cleared
- [ ] Session persists after page refresh
- [ ] All pages are mobile responsive
- [ ] No type errors when running `bun check`

## Effort Breakdown

- Frontend: 45% (auth UI, routes, navigation, layout)
- Backend: 35% (Convex schema, user sync, queries)
- Infrastructure/DevOps: 20% (Clerk webhook, environment setup)

## Dependencies

- None (Phase 1 is foundational)

## Risks & Mitigation

| Risk | Impact | Mitigation |
|------|--------|-----------|
| Clerk webhook timing issues | User sync delays | Implement sync polling and manual sync button |
| JWT token expiration | Session timeouts | Set appropriate Clerk session timeout, handle token refresh |
| Race conditions on user creation | Duplicate users | Use Convex indexes on clerkId, handle unique constraint errors |
| RBAC complexity | Authorization bugs | Define clear permission matrix, test edge cases |

## Team Assignment

- **Frontend Developer**: Login/signup pages, protected routes, navigation
- **Backend Developer**: User schema, Convex mutations/queries, auth middleware
- **DevOps**: Clerk webhook setup, environment configuration

## Technical Notes

### Database Schema

```typescript
// Users table
{
  clerkId: string (indexed, from Clerk)
  email: string (indexed)
  name?: string
  imageUrl?: string
  roles: string[] // ['user', 'admin', 'superadmin']
  createdAt: number
  updatedAt: number
}
```

### Key Files to Create/Modify

**Frontend:**
- `/app/routes/login._index.tsx` - Login page
- `/app/routes/sign-up._index.tsx` - Sign-up page
- `/app/routes/_auth.tsx` - Protected auth layout
- `/app/routes/_auth.dashboard.tsx` - User dashboard
- `/app/routes/_auth.admin._index.tsx` - Admin dashboard
- `/app/routes/_auth.admin.users._index.tsx` - Admin user list
- `/app/routes/_auth.profile.tsx` - User profile page
- `/app/components/layouts/header.tsx` - Navigation header
- `/app/hooks/use-auth.ts` - Auth hook with role checking
- `/app/lib/permissions.ts` - Permission helper functions

**Backend:**
- `/convex/auth.ts` - Auth configuration and helpers
- Extend `/convex/users.ts` - Add admin stats query
- `/convex/schema.ts` - Already exists, ensure correct structure

**Configuration:**
- Update Clerk webhooks for user sync events
- Configure environment variables in `wrangler.jsonc`

### API Contracts

**Mutations:**
- `users.syncUser(clerkId, email, name, imageUrl, roles)` → userId
- `users.syncUserInternal(...)` → userId (internal)

**Queries:**
- `users.getMe()` → User | null
- `users.getAdminStats()` → { users: { total, admins, superAdmins } }
- `users.listUsers(search?, role?, limit?, offset?)` → { users[], totalCount, hasMore }
- `users.getUserById(userId)` → User | null

### Type Safety

All TypeScript interfaces must be explicit (no `any` types):
- User interface matching Convex schema
- Role type union: `type UserRole = 'user' | 'admin' | 'superadmin'`
- Permission interface for RBAC system

### Testing Strategy

**Functional Tests:**
- [ ] Unauthenticated user redirected to login
- [ ] User can complete sign-up flow
- [ ] User can complete sign-in flow
- [ ] User profile synced to Convex
- [ ] Admin sees all users in list
- [ ] Non-admin cannot access /admin routes
- [ ] Session persists across page refresh
- [ ] Logout clears session

**Integration Tests:**
- [ ] Clerk → Convex sync works end-to-end
- [ ] Role permissions enforced at API level
- [ ] Admin queries return correct data

### Security Considerations

- JWT token validation in all protected functions
- Role check before returning admin-only data
- Clerk webhook signature verification
- Environment variables not exposed to client
- HTTPS enforced in production

## Deliverables

1. Fully functional authentication system
2. User management admin interface
3. Role-based access control foundation
4. Protected routes and navigation
5. Documentation for auth flow and permission system
6. All code passes type checking and linting

## Notes

- Phase 1 establishes the foundation for all subsequent phases
- Most of the Clerk integration and user schema already exist in the codebase
- Focus on completing the sync flow and admin dashboard
- Ensure permission system is extensible for future features
