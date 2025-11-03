# Phase 1: MVP - Authentication & Core Worklog Creation

**Status**: 🔴 Not Started
**Progress**: 0/18 tasks complete (0%)

---

## Infrastructure Tasks

### 1. [ ] Project Setup and Configuration

**Objective**: Initialize the full-stack project with Vite, React, TypeScript, Convex, and Clerk.

**Layer**: Infrastructure

**Subtasks**:
- [ ] Initialize Vite + React + TypeScript project
- [ ] Install and configure Convex CLI and dependencies
- [ ] Set up Clerk account and development instance
- [ ] Configure environment variables (.env.local for Clerk keys)
- [ ] Set up TypeScript strict mode configuration
- [ ] Configure ESLint + Prettier with recommended rules

**Effort**: Large (8h)
**Complexity**: Medium
**Assigned to**: Full-stack Developer
**Depends on**: None
**Blocks**: All other tasks

**Success Criteria**:
- [ ] `npm run dev` starts Vite dev server successfully
- [ ] `npx convex dev` runs without errors
- [ ] TypeScript compilation passes with strict mode
- [ ] ESLint runs without errors
- [ ] All environment variables properly loaded
- [ ] Git repository initialized with .gitignore

**Technical Notes**:
- Use latest Vite template for React + TypeScript
- Follow official Convex + Clerk integration guide
- Add .env.local to .gitignore
- Document all environment variables in .env.example
- Set up npm scripts for concurrent dev (frontend + Convex)

**Status**:
- [ ] Not Started
- [ ] In Progress
- [ ] Complete

---

### 2. [ ] UI Component Library Setup

**Objective**: Configure Tailwind CSS and shadcn/ui component library for consistent UI design.

**Layer**: Frontend

**Subtasks**:
- [ ] Install and configure Tailwind CSS with Vite
- [ ] Initialize shadcn/ui configuration
- [ ] Install core shadcn/ui components (Button, Input, Form, Card)
- [ ] Set up custom theme configuration (colors, typography)
- [ ] Create base layout components (Header, Container, Page wrapper)

**Effort**: Medium (4h)
**Complexity**: Low
**Assigned to**: Frontend Developer
**Depends on**: Task 1
**Blocks**: All frontend tasks

**Success Criteria**:
- [ ] Tailwind CSS compiles and styles render correctly
- [ ] shadcn/ui components imported and functional
- [ ] Custom theme applied across application
- [ ] Base layout components render correctly
- [ ] Responsive design works on mobile and desktop

**Technical Notes**:
- Use shadcn/ui CLI for component installation
- Configure Tailwind to use CSS variables for theming
- Set up mobile-first responsive breakpoints
- Document component usage patterns

**Status**:
- [ ] Not Started
- [ ] In Progress
- [ ] Complete

---

### 3. [ ] Convex Database Schema Definition

**Objective**: Define and deploy database schemas for users and worklogs collections with proper indexes.

**Layer**: Database

**Subtasks**:
- [ ] Create Convex schema file with users table definition
- [ ] Create Convex schema file with worklogs table definition
- [ ] Define field types and validation rules in schema
- [ ] Create index on users.clerkUserId (unique)
- [ ] Create index on worklogs.userId
- [ ] Deploy schema to Convex development environment

**Effort**: Medium (4h)
**Complexity**: Medium
**Assigned to**: Backend Developer
**Depends on**: Task 1
**Blocks**: Task 4, Task 7, Task 10, Task 13

**Success Criteria**:
- [ ] Users table created with fields: clerkUserId, name, dailyMinHours, dailyMaxHours, role
- [ ] Worklogs table created with fields: userId, date, workedHours, taskId, description
- [ ] Unique index on users.clerkUserId enforced
- [ ] Index on worklogs.userId verified in Convex dashboard
- [ ] Schema validation passes for all fields
- [ ] No TypeScript errors in schema files

**Technical Notes**:
- Use Convex v.object() for schema definition
- clerkUserId should be string, unique
- dailyMinHours and dailyMaxHours should be float64
- workedHours should be float64 (0.01 precision)
- date should be stored as Unix timestamp or ISO string
- taskId and description are optional (v.optional())

**Status**:
- [ ] Not Started
- [ ] In Progress
- [ ] Complete

---

## Feature: User Authentication

### 4. [ ] Clerk Authentication Integration

**Objective**: Integrate Clerk authentication provider with React application and Convex backend.

**Layer**: API + Frontend

**Subtasks**:
- [ ] Wrap React app with ClerkProvider
- [ ] Configure Convex authentication with Clerk
- [ ] Set up authentication middleware in Convex
- [ ] Implement user session management
- [ ] Create authentication utility functions
- [ ] Test authentication token flow

**Effort**: Large (8h)
**Complexity**: High
**Assigned to**: Full-stack Developer
**Depends on**: Task 1, Task 3
**Blocks**: Task 5, Task 6, Task 7

**Success Criteria**:
- [ ] ClerkProvider properly configured with publishable key
- [ ] Convex authenticates requests using Clerk tokens
- [ ] User identity accessible in Convex functions via ctx.auth
- [ ] Authentication state reactive in React components
- [ ] Session persists across page refreshes
- [ ] Token refresh handled automatically

**Technical Notes**:
- Use @clerk/clerk-react package
- Configure ConvexProviderWithClerk
- Follow Convex + Clerk integration guide exactly
- Handle edge cases: token expiry, logout, network failures
- Add error boundaries for auth failures

**Status**:
- [ ] Not Started
- [ ] In Progress
- [ ] Complete

---

### 5. [ ] Login Page with Clerk UI

**Objective**: Build login and signup pages using Clerk prebuilt UI components.

**Layer**: Frontend

**Subtasks**:
- [ ] Create login page component
- [ ] Integrate Clerk SignIn component
- [ ] Create signup page component
- [ ] Integrate Clerk SignUp component
- [ ] Configure routing for /login and /signup
- [ ] Style Clerk components to match app theme

**Effort**: Small (2h)
**Complexity**: Low
**Assigned to**: Frontend Developer
**Depends on**: Task 4
**Blocks**: Task 6

**Success Criteria**:
- [ ] Login page renders Clerk SignIn component
- [ ] Signup page renders Clerk SignUp component
- [ ] Email/password authentication works
- [ ] Successful login redirects to dashboard
- [ ] Clerk UI matches application theme
- [ ] Mobile responsive design

**Technical Notes**:
- Use Clerk's <SignIn /> and <SignUp /> components
- Configure appearance prop for theming
- Set redirectUrl to dashboard after authentication
- Handle authentication errors gracefully

**Status**:
- [ ] Not Started
- [ ] In Progress
- [ ] Complete

---

### 6. [ ] Protected Route Guards

**Objective**: Implement route protection to redirect unauthenticated users to login page.

**Layer**: Frontend

**Subtasks**:
- [ ] Create ProtectedRoute wrapper component
- [ ] Implement authentication check using Clerk hooks
- [ ] Configure React Router with protected routes
- [ ] Redirect anonymous users to /login
- [ ] Create public route configuration (help page)
- [ ] Test route protection edge cases

**Effort**: Medium (4h)
**Complexity**: Medium
**Assigned to**: Frontend Developer
**Depends on**: Task 4, Task 5
**Blocks**: Task 8, Task 11, Task 14

**Success Criteria**:
- [ ] Authenticated users can access protected routes
- [ ] Unauthenticated users redirected to /login
- [ ] Public routes (help page) accessible without auth
- [ ] Redirect preserves intended destination URL
- [ ] No flash of protected content before redirect
- [ ] Loading state during authentication check

**Technical Notes**:
- Use Clerk's useAuth() hook for auth state
- Implement route guard as HOC or wrapper component
- Use React Router's Navigate for redirects
- Handle loading state while auth initializes
- Store intended route for post-login redirect

**Status**:
- [ ] Not Started
- [ ] In Progress
- [ ] Complete

---

### 7. [ ] User Profile Query and Creation

**Objective**: Implement Convex query to fetch current user profile and create user on first login.

**Layer**: API

**Subtasks**:
- [ ] Create Convex query: api.users.getMe
- [ ] Implement user lookup by clerkUserId
- [ ] Create mutation: api.users.createUser for first-time users
- [ ] Set default dailyMinHours and dailyMaxHours values
- [ ] Handle user creation on first authentication
- [ ] Return user profile with all fields

**Effort**: Medium (4h)
**Complexity**: Medium
**Assigned to**: Backend Developer
**Depends on**: Task 3, Task 4
**Blocks**: Task 13

**Success Criteria**:
- [ ] getMe query returns current user profile
- [ ] New users automatically created on first login
- [ ] Default thresholds set correctly (dailyMinHours: 8, dailyMaxHours: 8)
- [ ] Query returns null for unauthenticated requests
- [ ] User profile includes all required fields
- [ ] No duplicate user creation on concurrent requests

**Technical Notes**:
- Use ctx.auth.getUserIdentity() for Clerk user ID
- Implement upsert pattern for user creation
- Default role should be "user" (not "admin")
- Store user name from Clerk metadata
- Handle race conditions in user creation

**Status**:
- [ ] Not Started
- [ ] In Progress
- [ ] Complete

---

### 8. [ ] Public Help Page

**Objective**: Create a public help page with system information accessible without authentication.

**Layer**: Frontend

**Subtasks**:
- [ ] Create Help page component
- [ ] Add informational content about the worklog system
- [ ] Style page with consistent layout
- [ ] Configure public route for /help
- [ ] Add navigation link to help page
- [ ] Test accessibility without login

**Effort**: Small (2h)
**Complexity**: Low
**Assigned to**: Frontend Developer
**Depends on**: Task 2, Task 6
**Blocks**: None

**Success Criteria**:
- [ ] Help page accessible at /help without authentication
- [ ] Content explains worklog system purpose
- [ ] Consistent styling with rest of application
- [ ] Navigation link visible in header
- [ ] Mobile responsive layout
- [ ] No authentication errors when accessing

**Technical Notes**:
- Mark /help route as public in router config
- Include information about authentication requirement
- Add links to login/signup if not authenticated
- Keep content concise and user-friendly

**Status**:
- [ ] Not Started
- [ ] In Progress
- [ ] Complete

---

## Feature: Worklog Creation

### 9. [ ] Navigation Structure

**Objective**: Implement application header with navigation menu and user information.

**Layer**: Frontend

**Subtasks**:
- [ ] Create Header component with logo/title
- [ ] Add navigation menu (Dashboard, Create Worklog, Help)
- [ ] Display logged-in user name
- [ ] Add logout button with Clerk SignOutButton
- [ ] Implement mobile responsive menu (hamburger)
- [ ] Style navigation with Tailwind/shadcn

**Effort**: Medium (4h)
**Complexity**: Medium
**Assigned to**: Frontend Developer
**Depends on**: Task 2, Task 4
**Blocks**: Task 11, Task 14

**Success Criteria**:
- [ ] Header visible on all authenticated pages
- [ ] Navigation links functional and highlight active page
- [ ] User name displayed correctly
- [ ] Logout button signs user out successfully
- [ ] Mobile menu toggles correctly
- [ ] Responsive design works on all screen sizes

**Technical Notes**:
- Use shadcn NavigationMenu component
- Get user name from Clerk useUser() hook
- Use Clerk's <SignOutButton /> component
- Implement active route highlighting
- Mobile breakpoint at 768px

**Status**:
- [ ] Not Started
- [ ] In Progress
- [ ] Complete

---

### 10. [ ] Worklog Creation Form Component

**Objective**: Build the worklog creation form with all fields and client-side validation.

**Layer**: Frontend

**Subtasks**:
- [ ] Create WorklogForm component with React Hook Form
- [ ] Add date picker field (defaults to today)
- [ ] Add hours input field with number validation
- [ ] Add optional task ID text input (max 50 chars)
- [ ] Add optional description textarea (max 500 chars)
- [ ] Implement client-side validation rules
- [ ] Add form submit button with loading state

**Effort**: Large (8h)
**Complexity**: High
**Assigned to**: Frontend Developer
**Depends on**: Task 2
**Blocks**: Task 11

**Success Criteria**:
- [ ] Date picker prevents future date selection
- [ ] Hours input validates range (0.01-24.00)
- [ ] Hours input suggests 0.25 increments
- [ ] Task ID limited to 50 characters
- [ ] Description limited to 500 characters
- [ ] Validation errors display clearly
- [ ] Form disabled during submission
- [ ] Mobile-friendly form layout

**Technical Notes**:
- Use React Hook Form with zod validation
- Use shadcn DatePicker component
- Use shadcn Input component with type="number"
- Set step="0.25" for hours input
- Disable date picker for dates > today
- Show character count for description field

**Status**:
- [ ] Not Started
- [ ] In Progress
- [ ] Complete

---

### 11. [ ] Worklog Creation Page

**Objective**: Integrate worklog form into a dedicated page with success/error handling.

**Layer**: Frontend

**Subtasks**:
- [ ] Create Create Worklog page component
- [ ] Integrate WorklogForm component
- [ ] Handle form submission with Convex mutation
- [ ] Display success message on successful creation
- [ ] Display error messages on validation failure
- [ ] Redirect to dashboard after successful creation
- [ ] Add "Create Another" option

**Effort**: Medium (4h)
**Complexity**: Medium
**Assigned to**: Frontend Developer
**Depends on**: Task 6, Task 9, Task 10
**Blocks**: Task 12

**Success Criteria**:
- [ ] Page accessible at /worklogs/create
- [ ] Form submission triggers Convex mutation
- [ ] Success toast/message shown after creation
- [ ] Validation errors displayed from server
- [ ] User redirected to dashboard on success
- [ ] Loading state during mutation
- [ ] "Create Another" option clears form

**Technical Notes**:
- Use Convex useMutation hook
- Handle mutation errors gracefully
- Use toast notification for success message
- Clear form after successful submission
- Preserve form data if validation fails

**Status**:
- [ ] Not Started
- [ ] In Progress
- [ ] Complete

---

### 12. [ ] Worklog Creation Mutation with Validation

**Objective**: Implement server-side worklog creation with comprehensive validation and authorization.

**Layer**: API

**Subtasks**:
- [ ] Create Convex mutation: api.worklogs.create
- [ ] Validate date (no future dates)
- [ ] Validate workedHours range (0.01-24.00)
- [ ] Validate taskId length (max 50 chars)
- [ ] Validate description length (max 500 chars)
- [ ] Verify user is authenticated (ctx.auth)
- [ ] Insert worklog into database
- [ ] Return created worklog or validation errors

**Effort**: Large (8h)
**Complexity**: High
**Assigned to**: Backend Developer
**Depends on**: Task 3, Task 7, Task 11
**Blocks**: None

**Success Criteria**:
- [ ] Mutation creates worklog successfully with valid data
- [ ] Mutation rejects future dates with clear error
- [ ] Mutation rejects hours outside 0.01-24.00 range
- [ ] Mutation rejects taskId longer than 50 chars
- [ ] Mutation rejects description longer than 500 chars
- [ ] Mutation rejects unauthenticated requests
- [ ] Mutation returns created worklog with ID
- [ ] Error messages are clear and actionable

**Technical Notes**:
- Use Convex v.object() for input validation
- Store date as Unix timestamp (ms) for easier querying
- Round hours to 2 decimal places
- Use ctx.auth.getUserIdentity() to get userId
- Return ConvexError for validation failures
- Log validation failures for debugging

**Status**:
- [ ] Not Started
- [ ] In Progress
- [ ] Complete

---

## Feature: Worklog Viewing

### 13. [ ] Personal Worklogs Query

**Objective**: Implement Convex query to fetch authenticated user's worklogs (basic list, no aggregation).

**Layer**: API

**Subtasks**:
- [ ] Create Convex query: api.worklogs.getMyWorklogs
- [ ] Filter worklogs by current user's ID
- [ ] Sort worklogs by date descending
- [ ] Return worklog fields (date, hours, taskId, description)
- [ ] Verify user authentication before query
- [ ] Handle empty results gracefully

**Effort**: Medium (4h)
**Complexity**: Medium
**Assigned to**: Backend Developer
**Depends on**: Task 3, Task 7
**Blocks**: Task 14

**Success Criteria**:
- [ ] Query returns only current user's worklogs
- [ ] Worklogs sorted by date (most recent first)
- [ ] Query returns empty array for users with no worklogs
- [ ] Unauthenticated requests return error
- [ ] All worklog fields included in results
- [ ] Query executes efficiently with proper indexes

**Technical Notes**:
- Use ctx.auth.getUserIdentity() for userId lookup
- Use Convex .filter() for userId matching
- Use .order("desc") for date sorting
- Leverage index on worklogs.userId
- No pagination in this phase (Phase 2 will add it)
- Return worklog data as array of objects

**Status**:
- [ ] Not Started
- [ ] In Progress
- [ ] Complete

---

### 14. [ ] Personal Dashboard with Worklog List

**Objective**: Create dashboard page displaying user's worklogs in a simple list format.

**Layer**: Frontend

**Subtasks**:
- [ ] Create Dashboard page component
- [ ] Fetch worklogs using api.worklogs.getMyWorklogs query
- [ ] Display each worklog in a card/list item
- [ ] Show date, hours, task ID (if present), description (if present)
- [ ] Handle loading state during query
- [ ] Handle empty state (no worklogs yet)
- [ ] Display real-time updates when new worklog created

**Effort**: Large (8h)
**Complexity**: Medium
**Assigned to**: Frontend Developer
**Depends on**: Task 2, Task 6, Task 9, Task 13
**Blocks**: None

**Success Criteria**:
- [ ] Dashboard accessible at / or /dashboard
- [ ] Worklogs displayed in descending date order
- [ ] Each worklog shows all fields clearly
- [ ] Loading spinner shown during query
- [ ] Empty state message shown for new users
- [ ] New worklogs appear automatically (real-time)
- [ ] Mobile responsive list layout

**Technical Notes**:
- Use Convex useQuery hook for reactive data
- Use shadcn Card component for worklog items
- Format date as human-readable (e.g., "Jan 15, 2025")
- Show task ID and description only if present
- Implement skeleton loading state
- Add "Create your first worklog" CTA for empty state

**Status**:
- [ ] Not Started
- [ ] In Progress
- [ ] Complete

---

## Testing & Documentation

### 15. [ ] Database Seed Script for Development

**Objective**: Create seed script to populate database with test users and worklogs for development.

**Layer**: Infrastructure

**Subtasks**:
- [ ] Create seed script file
- [ ] Generate 3 test users (user1, user2, admin)
- [ ] Set appropriate roles and thresholds for test users
- [ ] Generate 10 worklogs per user with varied data
- [ ] Make script idempotent (safe to re-run)
- [ ] Add npm script command: npm run seed

**Effort**: Medium (4h)
**Complexity**: Medium
**Assigned to**: Backend Developer
**Depends on**: Task 3, Task 7, Task 12
**Blocks**: Task 16

**Success Criteria**:
- [ ] Seed script creates 3 users successfully
- [ ] Admin user has role "admin"
- [ ] Each user has 10 varied worklogs
- [ ] Script can be re-run without errors
- [ ] Worklogs span multiple days
- [ ] Some worklogs have optional fields, some don't
- [ ] Script completes in under 1 minute

**Technical Notes**:
- Use Convex mutation calls from Node.js script
- Check if users exist before creating (idempotency)
- Generate random dates within past 30 days
- Vary hours between 4-10 for realistic distribution
- Some worklogs with taskId/description, some without
- Log progress to console

**Status**:
- [ ] Not Started
- [ ] In Progress
- [ ] Complete

---

### 16. [ ] End-to-End User Flow Testing

**Objective**: Manually test complete user journey from signup to worklog creation and viewing.

**Layer**: Testing

**Subtasks**:
- [ ] Test new user registration flow
- [ ] Test login with existing user
- [ ] Test logout and re-login
- [ ] Test worklog creation with all fields
- [ ] Test worklog creation with only required fields
- [ ] Test client-side validation error handling
- [ ] Test server-side validation error handling
- [ ] Test dashboard displays worklogs correctly
- [ ] Test real-time updates after creating worklog
- [ ] Test anonymous user redirect to login

**Effort**: Medium (4h)
**Complexity**: Low
**Assigned to**: QA / Developer
**Depends on**: Task 15
**Blocks**: None

**Success Criteria**:
- [ ] All user flows complete without errors
- [ ] Validation catches invalid inputs
- [ ] Error messages clear and helpful
- [ ] Real-time updates work consistently
- [ ] Protected routes properly guarded
- [ ] Mobile responsive on multiple devices
- [ ] No console errors during testing
- [ ] Performance acceptable (no lag)

**Technical Notes**:
- Test on both desktop and mobile viewports
- Use browser dev tools to verify API calls
- Test with various data combinations
- Document any bugs found
- Create test user accounts for regression testing

**Status**:
- [ ] Not Started
- [ ] In Progress
- [ ] Complete

---

### 17. [ ] TypeScript Compilation and Linting

**Objective**: Ensure zero TypeScript errors and ESLint warnings across entire codebase.

**Layer**: Infrastructure

**Subtasks**:
- [ ] Run TypeScript compiler check (tsc --noEmit)
- [ ] Fix all TypeScript errors
- [ ] Run ESLint across all source files
- [ ] Fix all ESLint errors and warnings
- [ ] Configure pre-commit hooks for type checking
- [ ] Document any intentional type assertions

**Effort**: Small (2h)
**Complexity**: Low
**Assigned to**: Developer
**Depends on**: All code tasks
**Blocks**: None

**Success Criteria**:
- [ ] `tsc --noEmit` passes with zero errors
- [ ] ESLint passes with zero errors
- [ ] No "any" types used without justification
- [ ] All Convex functions properly typed
- [ ] Pre-commit hooks prevent committing bad code
- [ ] CI pipeline validates types and linting

**Technical Notes**:
- Use TypeScript strict mode
- Configure ESLint with recommended React rules
- Add husky for pre-commit hooks
- Document any necessary type assertions
- Fix type errors, don't suppress them

**Status**:
- [ ] Not Started
- [ ] In Progress
- [ ] Complete

---

### 18. [ ] Phase 1 Documentation

**Objective**: Document Phase 1 setup, architecture decisions, and API contracts.

**Layer**: Documentation

**Subtasks**:
- [ ] Document local development setup steps
- [ ] Document environment variable configuration
- [ ] Document database schema and indexes
- [ ] Document Convex API endpoints (queries/mutations)
- [ ] Document authentication flow
- [ ] Create README with quick start guide

**Effort**: Small (2h)
**Complexity**: Low
**Assigned to**: Developer
**Depends on**: All other tasks
**Blocks**: None

**Success Criteria**:
- [ ] README has clear setup instructions
- [ ] All environment variables documented
- [ ] Database schema documented with field descriptions
- [ ] API contracts documented (inputs/outputs)
- [ ] Authentication flow diagram or description
- [ ] New developer can set up project from docs

**Technical Notes**:
- Use markdown for all documentation
- Include code examples where helpful
- Document known limitations
- Link to official Convex/Clerk docs
- Keep documentation up to date with code changes

**Status**:
- [ ] Not Started
- [ ] In Progress
- [ ] Complete

---

## Task Dependency Graph

**Critical Path (must complete in order)**:
```
Task 1 (Project Setup)
  → Task 3 (DB Schema)
    → Task 4 (Auth Integration)
      → Task 7 (User Profile Query)
        → Task 12 (Worklog Creation Mutation)
          → Task 13 (Personal Worklogs Query)
            → Task 14 (Dashboard)
```

**Parallel Work Opportunities**:
```
After Task 1:
- Task 2 (UI Setup) || Task 3 (DB Schema)

After Task 4:
- Task 5 (Login Page) || Task 7 (User Profile)

After Task 2:
- Task 10 (Form Component) can start in parallel with backend work

After Task 3:
- Task 15 (Seed Script) can be developed alongside features
```

---

## Effort Summary

| Category | Tasks | Hours | Percentage |
|----------|-------|-------|------------|
| Infrastructure | 4 | 20h | 22% |
| Frontend | 8 | 32h | 36% |
| Backend | 4 | 24h | 27% |
| Testing | 1 | 4h | 4% |
| Documentation | 1 | 2h | 2% |
| **Total** | **18** | **90h** | **100%** |

---

## Tracking Progress

Update task status as you work:
1. Mark task as `In Progress` when you start
2. Check off subtasks as you complete them
3. Mark task as `Complete` when all success criteria met
4. Update progress percentage at top of document

**Example completed task**:
```markdown
### 1. [x] Project Setup and Configuration ✅ Completed
```
