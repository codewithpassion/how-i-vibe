# Phase 1: Authentication & User Management - Tasks

**Status**: 🔴 Not Started | 🟡 In Progress | 🟢 Complete
**Progress**: 2/12 tasks complete (17%)

---

## Infrastructure & Foundation Tasks

### 1. [x] Setup Clerk Webhook Configuration

**Objective**: Configure Clerk webhooks to sync user creation/updates to Convex database

**Layer**: Infrastructure

**Subtasks**:
- [x] Create Clerk webhook handler in Convex (`/convex/http.ts`)
- [x] Validate webhook signatures from Clerk
- [x] Handle user.created event
- [x] Handle user.updated event
- [ ] Configure webhook endpoint URL in Clerk dashboard
- [ ] Test webhook delivery with Clerk test events
- [x] Add error logging for webhook failures

**Effort**: Medium (4h)
**Complexity**: Medium
**Assigned to**: Backend Developer
**Depends on**: None
**Blocks**: Task 2, Task 3

**Success Criteria**:
- [x] Webhook endpoint receives and validates Clerk events
- [x] User creation events trigger Convex sync
- [x] User updates from Clerk sync to database
- [x] Error handling logs all webhook failures
- [ ] Webhook can be tested successfully via Clerk dashboard

**Technical Notes**:
- Use Clerk webhook secret from environment variables
- Webhook signature verification required for security
- Handle idempotency (same event may arrive multiple times)
- Log all events for debugging

**Status**: [ ] Not Started [ ] In Progress [x] Complete

---

### 2. [ ] Extend Convex User Schema

**Objective**: Create complete user schema with proper fields and indexes

**Layer**: Backend (Database)

**Subtasks**:
- [ ] Define User table schema (clerkId, email, name, imageUrl, roles, timestamps)
- [ ] Create index on clerkId for webhook lookups
- [ ] Create index on email for duplicate prevention
- [ ] Add timestamps (createdAt, updatedAt)
- [ ] Define role type as union ('user' | 'admin' | 'superadmin')
- [ ] Test schema with Convex development environment
- [ ] Add schema validation for required fields

**Effort**: Small (2h)
**Complexity**: Low
**Assigned to**: Backend Developer
**Depends on**: None
**Blocks**: Task 3, Task 4

**Success Criteria**:
- [ ] User schema exists in Convex
- [ ] Indexes created for clerkId and email
- [ ] Schema compiles without errors
- [ ] Type generation works (`convex/_generated/` types)
- [ ] Sample user can be inserted without errors

**Technical Notes**:
- Index on clerkId critical for webhook sync performance
- Email index prevents duplicate users
- Roles stored as array: `['user']` or `['user', 'admin']`
- Use TypeScript for schema definition

**Status**: [ ] Not Started [ ] In Progress [ ] Complete

---

### 3. [ ] Create Convex User Sync Mutations

**Objective**: Implement mutations to sync users from Clerk to Convex

**Layer**: Backend (Queries/Mutations)

**Subtasks**:
- [ ] Create `syncUser` mutation (called from webhook)
- [ ] Implement user lookup by clerkId
- [ ] Handle user creation (new clerkId)
- [ ] Handle user update (existing clerkId)
- [ ] Add validation for required fields
- [ ] Set default role as 'user' for new users
- [ ] Create mutation for manual sync (admin)
- [ ] Add comprehensive error handling

**Effort**: Medium (4h)
**Complexity**: Medium
**Assigned to**: Backend Developer
**Depends on**: Task 2
**Blocks**: Task 5, Task 6

**Success Criteria**:
- [ ] New Clerk user creates Convex record
- [ ] Existing user updates propagate correctly
- [ ] Default role assigned to new users
- [ ] No duplicate users created
- [ ] Mutation returns complete user object
- [ ] Error cases handled gracefully

**Technical Notes**:
- Use unique constraint on clerkId
- Handle race conditions on concurrent syncs
- Preserve createdAt timestamp on update
- Update updatedAt on every sync

**Status**: [ ] Not Started [ ] In Progress [ ] Complete

---

## Backend Core Tasks

### 4. [ ] Create Convex User Queries

**Objective**: Implement queries for user data retrieval with proper authorization

**Layer**: Backend (Queries)

**Subtasks**:
- [ ] Create `getMe()` query (current authenticated user)
- [ ] Create `getUserById(userId)` query with auth check
- [ ] Create `listUsers(search?, role?, limit?, offset?)` for admin
- [ ] Create `getAdminStats()` query (user counts by role)
- [ ] Implement authorization checks (verify JWT token)
- [ ] Add pagination logic with offset/limit
- [ ] Test queries with different user roles

**Effort**: Medium (4h)
**Complexity**: Medium
**Assigned to**: Backend Developer
**Depends on**: Task 3
**Blocks**: Task 8, Task 9

**Success Criteria**:
- [ ] `getMe()` returns authenticated user's data
- [ ] `getUserById()` returns user or null if not found
- [ ] `listUsers()` returns paginated results (admin only)
- [ ] `getAdminStats()` shows correct user counts
- [ ] Non-admin cannot call admin queries
- [ ] Pagination works correctly with offset

**Technical Notes**:
- Use Convex auth context to get current user ID
- Add role checks with clear error messages
- Implement pagination with cursor-based approach
- Return only needed fields (don't expose secrets)

**Status**: [ ] Not Started [ ] In Progress [ ] Complete

---

### 5. [ ] Create Auth Helper Functions

**Objective**: Implement permission checking and authorization utilities

**Layer**: Backend (Utilities)

**Subtasks**:
- [ ] Create permission helper file (`/convex/auth.ts` or extend)
- [ ] Implement `isAdmin()` check function
- [ ] Implement `isSuperAdmin()` check function
- [ ] Implement `canAccessUser(userId)` check
- [ ] Create middleware for role verification
- [ ] Add audit logging for permission checks
- [ ] Test permission helpers with different roles

**Effort**: Small (2h)
**Complexity**: Low
**Assigned to**: Backend Developer
**Depends on**: Task 4
**Blocks**: Task 10

**Success Criteria**:
- [ ] Admin role verified correctly
- [ ] Non-admin denied access to admin functions
- [ ] User can access own data
- [ ] User cannot access other users' data
- [ ] SuperAdmin can access all data
- [ ] Permission checks logged for audit trail

**Technical Notes**:
- Helper functions should be reusable across all mutations
- Use TypeScript for type safety
- Return clear error messages on permission denial
- Log all permission checks

**Status**: [ ] Not Started [ ] In Progress [ ] Complete

---

## Frontend Core Tasks

### 6. [ ] Build Login & Sign-Up Pages

**Objective**: Create authentication entry pages with Clerk components

**Layer**: Frontend (UI/Pages)

**Subtasks**:
- [ ] Create `/app/routes/login._index.tsx` with Clerk SignIn
- [ ] Create `/app/routes/sign-up._index.tsx` with Clerk SignUp
- [ ] Implement redirect after successful login (to dashboard)
- [ ] Implement redirect for already-authenticated users
- [ ] Add responsive styling with Tailwind
- [ ] Add loading states during auth
- [ ] Test with Clerk test user credentials

**Effort**: Small (2h)
**Complexity**: Low
**Assigned to**: Frontend Developer
**Depends on**: Task 1
**Blocks**: Task 9

**Success Criteria**:
- [ ] Login page loads and displays Clerk component
- [ ] Sign-up page loads and displays Clerk component
- [ ] Successful login redirects to dashboard
- [ ] Already-authenticated users redirected from login page
- [ ] Pages responsive on mobile (< 768px)
- [ ] No TypeScript errors

**Technical Notes**:
- Use Clerk React Router integration
- Handle loading and error states
- Preserve redirect URL on login
- Responsive design with Tailwind breakpoints

**Status**: [ ] Not Started [ ] In Progress [ ] Complete

---

### 7. [ ] Create Protected Routes & Auth Guard

**Objective**: Implement route protection and role-based route guards

**Layer**: Frontend (Routing)

**Subtasks**:
- [ ] Create `_auth.tsx` layout (authenticated routes wrapper)
- [ ] Implement route guards that check authentication
- [ ] Create route guards for admin-only pages
- [ ] Implement redirect to login for unauthenticated users
- [ ] Add loading skeleton while checking auth
- [ ] Create 403 Forbidden error page
- [ ] Create 404 Not Found error page
- [ ] Test routes with and without authentication

**Effort**: Small (2h)
**Complexity**: Low
**Assigned to**: Frontend Developer
**Depends on**: Task 6
**Blocks**: Task 9, Task 10

**Success Criteria**:
- [ ] Unauthenticated users redirected to login
- [ ] Authenticated users can access protected routes
- [ ] Admin routes reject non-admin users (403)
- [ ] Error pages display correctly
- [ ] Loading state shows while checking auth
- [ ] No infinite redirects or loops

**Technical Notes**:
- Use React Router layout routes
- Check user role from Clerk token
- Handle token refresh gracefully
- Clear error messages for permission denied

**Status**: [ ] Not Started [ ] In Progress [ ] Complete

---

### 8. [ ] Build Navigation & Layout Header

**Objective**: Create main navigation header with user menu

**Layer**: Frontend (Components)

**Subtasks**:
- [ ] Create header component with logo/branding
- [ ] Add navigation links (Dashboard, Worklogs, Admin)
- [ ] Create user menu dropdown (Profile, Settings, Logout)
- [ ] Implement responsive mobile menu
- [ ] Add admin badge for admin users
- [ ] Add loading state during user fetch
- [ ] Style with Tailwind CSS and ShadCN components
- [ ] Test responsive behavior on mobile

**Effort**: Small (2h)
**Complexity**: Low
**Assigned to**: Frontend Developer
**Depends on**: Task 7
**Blocks**: Task 9

**Success Criteria**:
- [ ] Header displays on all authenticated pages
- [ ] Navigation links route correctly
- [ ] User menu shows logged-in user name/avatar
- [ ] Logout clears session
- [ ] Admin menu items only show for admin users
- [ ] Responsive on mobile (hamburger menu)

**Technical Notes**:
- Use ShadCN Dropdown Menu component
- Display user avatar from Clerk
- Include logout functionality via Clerk
- Mobile menu collapses at 768px

**Status**: [ ] Not Started [ ] In Progress [ ] Complete

---

### 9. [x] Build User Dashboard & Profile Pages

**Objective**: Create authenticated user dashboard and profile management page

**Layer**: Frontend (Pages)

**Subtasks**:
- [x] Create `/app/routes/_auth.dashboard.tsx` (landing page)
- [x] Create `/app/routes/_auth.profile.tsx` (user profile)
- [x] Display user info (name, email, avatar)
- [x] Allow user to update profile (name, avatar)
- [x] Show role information
- [x] Add "View Worklogs" call-to-action
- [x] Add loading and error states
- [x] Style with Tailwind CSS

**Effort**: Small (2h)
**Complexity**: Low
**Assigned to**: Frontend Developer
**Depends on**: Task 8
**Blocks**: Task 11

**Success Criteria**:
- [x] Dashboard loads and displays user welcome
- [x] Profile page shows current user data
- [x] User can update profile name
- [x] Changes persist to Convex
- [x] Error messages show if update fails
- [x] Profile responsive on mobile

**Technical Notes**:
- Use React Query for data fetching
- Handle loading and error states
- Use ShadCN form components
- Display user image from Clerk/Convex

**Status**: [x] Complete

---

### 10. [ ] Build Admin Dashboard & User List

**Objective**: Create admin interface for user management and overview

**Layer**: Frontend (Pages)

**Subtasks**:
- [ ] Create `/app/routes/_auth.admin._index.tsx` (admin dashboard)
- [ ] Create `/app/routes/_auth.admin.users._index.tsx` (user list)
- [ ] Display admin statistics (total users, admins, superadmins)
- [ ] Show list of all users in table format
- [ ] Add search/filter by name or email
- [ ] Add sortable columns (name, email, role, created date)
- [ ] Add pagination controls
- [ ] Style table with ShadCN components

**Effort**: Medium (4h)
**Complexity**: Medium
**Assigned to**: Frontend Developer
**Depends on**: Task 7
**Blocks**: Task 11, Task 12

**Success Criteria**:
- [ ] Admin dashboard shows correct stats
- [ ] User list displays all users
- [ ] Search filters users by name/email
- [ ] Pagination loads more users
- [ ] Columns sortable (client-side)
- [ ] Non-admin users cannot access page

**Technical Notes**:
- Use React Query with pagination
- Implement client-side filtering/sorting first
- Use ShadCN Table component
- Loading skeleton while fetching

**Status**: [ ] Not Started [ ] In Progress [ ] Complete

---

## Integration & Testing Tasks

### 11. [ ] Implement Auth Hook for Role Checking

**Objective**: Create reusable React hook for checking user roles in components

**Layer**: Frontend (Hooks)

**Subtasks**:
- [ ] Create `/app/hooks/use-auth.ts` custom hook
- [ ] Expose current user data
- [ ] Expose `isAdmin()` helper
- [ ] Expose `isSuperAdmin()` helper
- [ ] Expose `canAccess(resource)` helper
- [ ] Add loading and error states
- [ ] Test hook with different user roles

**Effort**: Small (2h)
**Complexity**: Low
**Assigned to**: Frontend Developer
**Depends on**: Task 4
**Blocks**: Task 12

**Success Criteria**:
- [ ] Hook returns current user data
- [ ] `isAdmin()` returns true/false correctly
- [ ] `canAccess()` works for authorization
- [ ] Hook handles loading state
- [ ] Hook handles errors gracefully
- [ ] No TypeScript errors

**Technical Notes**:
- Use React context or Convex query
- Cache user data to prevent refetches
- Handle token refresh
- Return clear boolean values for checks

**Status**: [ ] Not Started [ ] In Progress [ ] Complete

---

### 12. [ ] Setup Permission System & Middleware

**Objective**: Create centralized permission configuration and middleware for enforcement

**Layer**: Frontend + Backend

**Subtasks**:
- [ ] Create permission matrix (user roles → accessible features)
- [ ] Create `/app/lib/permissions.ts` helper functions
- [ ] Implement `hasPermission(user, action)` function
- [ ] Add permission checks to protected routes
- [ ] Add permission checks to admin components
- [ ] Create unit tests for permission functions
- [ ] Document permission system
- [ ] Test with different user roles

**Effort**: Medium (4h)
**Complexity**: Medium
**Assigned to**: Frontend + Backend Developer
**Depends on**: Task 5, Task 11
**Blocks**: None (foundational for future work)

**Success Criteria**:
- [ ] Permission matrix clearly defined
- [ ] Permission helpers work correctly
- [ ] Routes protected by role
- [ ] Components hidden based on permissions
- [ ] Non-admin cannot access admin features
- [ ] SuperAdmin can access everything
- [ ] Permission system extensible for future roles

**Technical Notes**:
- Keep permission config DRY (single source of truth)
- Test edge cases (no role, invalid role)
- Log permission denials for security audit
- Make system extensible for new roles/permissions

**Status**: [ ] Not Started [ ] In Progress [ ] Complete

---

## Task Dependency Graph

**Critical Path (must complete in order):**
```
Task 1 (Webhook Config)
    ↓
Task 2 (Schema) → Task 3 (Sync Mutations) → Task 4 (Queries)
    ↓                                           ↓
    └─────────────────────────────────── Task 5 (Auth Helpers)
    ↓
Task 6 (Login/Sign-Up Pages)
    ↓
Task 7 (Protected Routes)
    ↓
Task 8 (Navigation Header)
    ↓
Task 9 (Dashboard & Profile)
    ↓
Task 10 (Admin Dashboard)
    ↓
Task 12 (Permission System - combines Tasks 5 & 11)
```

**Parallel Work:**
```
- Tasks 6, 8, 9 (Frontend pages) can start once Task 7 is done
- Task 11 (Hook) can develop in parallel with Task 10
- Task 12 (Permission system) depends on Tasks 5 & 11
```

## Effort Summary

| Category | Tasks | Percentage |
|----------|-------|-----------|
| Backend | 1, 2, 3, 4, 5 | 40% |
| Frontend | 6, 7, 8, 9, 10, 11 | 45% |
| Full Stack | 12 | 15% |
| **Total** | **12** | **100%** |

**Estimated Total Effort**: ~30 hours (Medium phase)

---

## Tracking Progress

Mark tasks complete as you finish by changing `[ ]` to `[x]`:

```markdown
### 1. [x] Setup Clerk Webhook Configuration ✅ Completed
```

Update progress at top: `**Progress**: 1/12 tasks complete (8%)`

---

## Notes

- Phase 1 is foundational - all subsequent phases depend on complete authentication
- Clerk webhook setup is critical path - test early and thoroughly
- Use Convex development environment for local testing
- Test permission system extensively - security-critical
- Consider implementing webhook retry logic for reliability
