# Phase 1: MVP - Authentication & Core Worklog Creation

## Objective
Deliver a minimum viable product that enables authenticated users to create and view their own worklog entries. This phase establishes the foundational architecture (Convex + Clerk + React) and delivers immediate value: users can start logging their work hours.

## Scope

### Features
- User authentication via Clerk (email/password)
- Anonymous user access control (redirect to login)
- Basic help page for system information
- Worklog creation with validation
- Simple personal worklog viewing (list format, no aggregation)
- Protected routing for authenticated users only

### Frontend
- Clerk authentication integration
- Protected route guards with React Router
- Login page with Clerk UI
- Help page (public, informational content)
- Worklog creation form with validation
  - Date picker (defaults to today, no future dates)
  - Hours input (0.01-24.00 range, 0.25 increments)
  - Optional task ID field (max 50 chars)
  - Optional description field (max 500 chars)
- Basic worklog list view (personal worklogs only)
- Navigation structure (header/menu)
- Tailwind CSS + shadcn/ui component setup

### Backend
- Convex project initialization
- Database schema definition:
  - `users` collection with fields: clerkUserId, name, dailyMinHours, dailyMaxHours, role
  - `worklogs` collection with fields: userId, date, workedHours, taskId, description
- Database indexes:
  - users: by clerkUserId (unique)
  - worklogs: by userId
- Convex mutations:
  - `api.worklogs.create` - Create new worklog with validation
- Convex queries:
  - `api.users.getMe` - Get current user profile
  - `api.worklogs.getMyWorklogs` - Fetch user's worklogs (basic, no pagination yet)
- Server-side validation:
  - Date validation (no future dates)
  - Hours range validation (0.01-24.00)
  - Field length limits
  - User authorization (can only create for self)

### Infrastructure
- Local development environment setup
  - Vite + React + TypeScript
  - Convex development mode
  - Clerk development instance
- Environment configuration
  - `.env.local` for Clerk API keys
  - Convex environment variables
- Development scripts
  - `npm run dev` for frontend
  - `npx convex dev` for backend
- TypeScript configuration (strict mode)
- ESLint + Prettier setup
- Git repository initialization
- Basic seed script (3 test users: user1, user2, admin)

## Success Criteria
- [ ] User can register/login via Clerk authentication
- [ ] Anonymous users redirected to login page from protected routes
- [ ] Public help page accessible without authentication
- [ ] Authenticated user can create worklog with all fields
- [ ] Authenticated user can create worklog with only required fields (date, hours)
- [ ] Client-side validation prevents invalid inputs (future dates, invalid hours)
- [ ] Server-side validation rejects invalid data with clear error messages
- [ ] User can view list of their own worklogs (simple list, no grouping yet)
- [ ] User cannot view other users' worklogs
- [ ] Form displays success message after worklog creation
- [ ] Real-time UI update shows new worklog after creation
- [ ] Mobile responsive design for all pages
- [ ] Zero TypeScript compilation errors
- [ ] All Convex functions properly typed

## Effort Breakdown
- Frontend: 35%
  - Clerk integration: 10%
  - Form components & validation: 15%
  - List view & navigation: 10%
- Backend: 40%
  - Schema design & indexes: 10%
  - Authentication integration: 15%
  - Mutations & queries: 15%
- Infrastructure/DevOps: 25%
  - Project setup (Vite, Convex, Clerk): 15%
  - TypeScript/linting configuration: 5%
  - Seed script: 5%

## Dependencies
- None (Phase 1 is the foundation)

## Risks & Mitigation

### Risk: Clerk + Convex Integration Complexity
- **Impact**: Medium - Authentication issues could block all development
- **Probability**: Low - Well-documented integration pattern
- **Mitigation**: Follow official Convex + Clerk integration guide closely, test authentication flow early in phase
- **Contingency**: Allocate buffer time for debugging authentication issues

### Risk: TypeScript Learning Curve
- **Impact**: Medium - Could slow initial development
- **Probability**: Medium - Team may be new to TypeScript strict mode
- **Mitigation**: Start with simple components, gradually add complexity, leverage TypeScript error messages for learning
- **Contingency**: Pair programming for complex type definitions

### Risk: Form Validation Gaps
- **Impact**: High - Invalid data could corrupt database
- **Probability**: Medium - Easy to miss edge cases
- **Mitigation**: Comprehensive validation testing with boundary values, server-side validation as final authority
- **Contingency**: Add validation layer-by-layer, test each rule independently

### Risk: Real-time Updates Not Working
- **Impact**: Medium - Degrades UX but worklog creation still functional
- **Probability**: Low - Convex handles this automatically
- **Mitigation**: Test query subscriptions early, verify re-fetching behavior after mutations
- **Contingency**: Manual refresh button as fallback if auto-update fails

## Notes
- This phase intentionally omits daily aggregation and OT/UT badges - those come in Phase 2
- Focus on getting the full stack working correctly with simple features
- Seed script creates minimal test data (3 users, ~10 worklogs each) for development
- All stub pages deferred to later phases
- No pagination in this phase - assume small dataset for MVP testing
