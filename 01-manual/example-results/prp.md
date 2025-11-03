# Product Requirements Prompt: Worklog Tracking System

## Executive Summary

Build a production-ready worklog tracking system for a 10,000-user organization with 10 years of historical data. The system enables employees to log daily work hours with automatic overtime/undertime detection based on configurable per-user thresholds. The application provides role-based access (Anonymous, Registered User, Admin) with comprehensive worklog management, filtering, and reporting capabilities. Built on React with Convex backend and Clerk authentication, the system must handle high-volume historical data migration while maintaining sub-second query performance for current operations.

## Technical Architecture

### System Overview

The worklog tracking system follows a modern serverless architecture with real-time database synchronization:

**Frontend Layer:**
- React SPA with React-Router for navigation
- Tailwind CSS + shadcn/ui component library for consistent UI
- Client-side state management via Convex reactive queries
- Clerk authentication integration for session management

**Backend Layer:**
- Convex serverless database and functions
- Real-time query subscriptions for live data updates
- Server-side business logic for worklog calculations
- Bun runtime for local development and tooling

**Authentication & Authorization:**
- Clerk for user authentication and session management
- Role-based access control (RBAC) enforced at database query level
- Secure user context propagation from frontend to Convex functions

**Data Architecture:**
- Convex document database with indexed queries
- Two primary collections: users, worklogs
- Computed fields for daily totals and OT/UT status
- Efficient pagination for historical data browsing

### Component Interaction Flow

```
User Browser
    ↓ (HTTPS)
Clerk Authentication
    ↓ (Auth Token)
React Frontend (shadcn UI)
    ↓ (WebSocket + HTTPS)
Convex Real-time Queries/Mutations
    ↓ (Server Functions)
Business Logic Layer (OT/UT Calculation)
    ↓ (Database Queries)
Convex Document Store (users, worklogs)
```

## Implementation Specification

### FEATURE SECTION

#### F1: User Authentication & Authorization

**Anonymous User Access:**
- Public routes: `/login` and `/help` pages only
- All other routes redirect to login page
- Help page displays informational content about the system
- No data access or API calls permitted for anonymous sessions

**Registered User Capabilities:**
- Authenticated access via Clerk
- Personal worklog creation with validation
- Worklog viewing with date grouping and aggregation
- Personal dashboard showing own work history only
- Automatic OT/UT badge calculation based on personal thresholds

**Admin User Capabilities:**
- All registered user capabilities
- System-wide worklog viewing across all users
- User management interface (stub pages as specified)
- Admin-specific navigation menu items
- Access to user configuration pages (stub implementation)

**Authorization Requirements:**
- Convex functions must verify user identity from Clerk session
- Database queries must filter by authenticated user ID (except admin)
- Admin role detection via Clerk metadata or database flag
- Automatic session refresh and token validation
- Graceful handling of expired or invalid sessions

#### F2: Worklog Creation

**Input Requirements:**
- Date field (date picker, defaults to today, cannot be future date)
- Worked hours (numeric input, 0.01-24.00 range, 0.25 hour increments)
- Task ID (optional text field, max 50 characters)
- Description (optional textarea, max 500 characters)
- All fields must have client-side and server-side validation

**Validation Rules:**
- Date cannot be in the future
- Worked hours must be positive and ≤ 24 hours
- Multiple worklogs allowed per day (no duplicate prevention needed)
- User can only create worklogs for themselves (enforced server-side)
- Required fields: date, workedHours
- Optional fields: taskId, description

**Business Logic:**
- Store worklog with authenticated user ID
- Timestamp creation with server time
- Automatic calculation of daily totals upon insertion
- Real-time UI update via Convex subscription
- Optimistic UI updates with rollback on error

**Error Handling:**
- Validation errors display inline with specific field messages
- Network errors show retry option
- Duplicate submission prevention during processing
- Server validation errors override client-side validation
- Clear error messages for business rule violations

#### F3: Worklog Viewing - Personal Dashboard

**Data Display Requirements:**
- Group worklogs by date (day granularity)
- Display date as section header (e.g., "Monday, January 15, 2025")
- Within each day, show individual worklog entries
- Each entry displays: time worked, task ID (if present), description (if present)
- Calculate and display total hours per day

**Daily Aggregation Logic:**
- Sum all worklogs for a given date
- Compare daily total against user's dailyMinHours and dailyMaxHours
- Display OT badge if total > dailyMaxHours
- Display UT badge if total < dailyMinHours
- No badge if within min/max range
- Badges should be visually distinct (colors, icons)

**Filtering Capabilities:**
- Filter by OT days only (days exceeding max hours)
- Filter by UT days only (days below min hours)
- Filter by normal days (within range)
- Clear filter button to show all entries
- Filters apply to entire date range
- Filter state persists during session

**Pagination Specification:**
- Page size: configurable (default 10 days)
- Load more button or infinite scroll pattern
- Total count display (e.g., "Showing 1-10 of 247 days")
- Pagination maintains filter state
- Efficient cursor-based pagination (not offset-based)
- Performance target: <500ms per page load

**Sorting Requirements:**
- Default sort: date descending (most recent first)
- No user-configurable sorting in v1
- Consistent ordering across pagination boundaries
- Secondary sort by creation timestamp if multiple logs same day

**Stub Page Implementation:**
- Menu item labeled "My Worklogs" navigates to stub page
- Stub page displays: "Feature Coming Soon: View and analyze your complete work history with advanced filtering and reporting capabilities."
- Stub maintains navigation structure and layout

#### F4: Worklog Viewing - Admin Dashboard

**Enhanced Display Requirements:**
- All personal dashboard features PLUS user identification
- Each worklog entry shows user's full name
- User name displayed prominently in each group/entry
- Same OT/UT badge logic applies per-user thresholds
- Admin can view any user's complete history

**Sorting Specification:**
- Primary sort: date descending (most recent first)
- Secondary sort: user name ascending (alphabetical)
- Within same user+date: creation timestamp ascending
- Sorting must be efficient for 10K users × 10 years data
- Database indexes required for sort performance

**Filtering Enhancements:**
- All personal dashboard filters
- Additional filter: by specific user (dropdown or search)
- Filter by date range (start date, end date)
- Combine multiple filters (e.g., user + OT days)
- Clear filters button resets all filter criteria

**Pagination for Scale:**
- Same pagination mechanism as personal dashboard
- Must handle larger result sets efficiently
- Performance target: <1 second per page load
- Cursor-based pagination essential for performance
- Preload next page for smooth UX

**Stub Pages for Admin:**
- "User Management" menu item → stub page: "Feature Coming Soon: Manage user accounts, roles, and permissions."
- "Edit User Hours" menu item → stub page: "Feature Coming Soon: Configure minimum and maximum daily hours for each employee."

#### F5: Database Seeding

**User Seed Data:**
- Create exactly 3 users: user1, user2, admin
- user1: name="User One", dailyMinHours=3, dailyMaxHours=5, role="user"
- user2: name="User Two", dailyMinHours=7, dailyMaxHours=9, role="user"
- admin: name="Admin User", dailyMinHours=6, dailyMaxHours=8, role="admin"
- All users have valid Clerk authentication setup
- Seed data must be idempotent (re-runnable without duplicates)

**Worklog Seed Data:**
- Generate 10 years of historical data (2015-2025)
- 0-5 worklogs per user per day (weighted toward 2-3)
- 70% of days have normal hours (within min/max)
- 15% of days have OT (above max)
- 15% of days have UT (below min)
- Random distribution of task IDs and descriptions
- Some entries with null taskId/description for variety
- Total estimated records: ~15,000-20,000 worklogs

**Data Generation Requirements:**
- Realistic work patterns (more activity on weekdays)
- Varied worked hours (0.25 to 12 hour increments)
- Mix of short/long descriptions
- Ensure edge cases present (exactly at min/max, 0.25 hours, etc.)
- Seed script must complete in <5 minutes
- Progress logging during long-running seed operations

**Seed Script Execution:**
- Command: `npm run seed` or `bun run seed`
- Checks for existing data before seeding
- Option to clear and reseed for development
- Success confirmation with record counts
- Error handling and rollback capability

#### F6: Data Migration Support

**Migration Requirements:**
- System must handle import of 10 years historical data
- Estimated 10,000 users × 220 work days/year × 10 years = 22M records
- Batch import capability (not single-transaction)
- Validation during import (reject invalid records)
- Import progress tracking and resumability
- Duplicate detection and handling

**Performance Targets:**
- Import rate: ≥1,000 records/second
- No impact on live system during migration
- Validation errors logged but don't stop import
- Transaction batching (1,000 records per batch)
- Memory-efficient streaming processing

**Data Validation During Import:**
- User ID must exist
- Date must be valid and not future
- Worked hours must be 0.01-24.00
- Optional field format validation
- Log validation failures to error file
- Continue processing after validation errors

### EXAMPLES SECTION

#### E1: Reference Architecture Patterns

**Convex Real-time Query Pattern:**
- Frontend components subscribe to Convex queries
- Queries automatically re-run when underlying data changes
- UI updates reactively without manual refresh
- Pattern: `const worklogs = useQuery(api.worklogs.getMyWorklogs, { filter: "OT" })`
- Loading states handled by checking query result for undefined
- Error states propagated through query subscription

**Role-Based Access Control Pattern:**
- Convex functions receive authenticated user context
- Extract user ID and role from Clerk session token
- Every query/mutation checks authorization before proceeding
- Pattern: Reject unauthorized access with clear error messages
- Admin functions verify role before executing privileged operations
- User-scoped queries automatically filter by authenticated user ID

**Pagination Cursor Pattern:**
- Use Convex pagination API with cursor-based navigation
- Frontend maintains current cursor state
- Load more button triggers query with next cursor
- Pattern returns: `{ items: [...], nextCursor: "...", isDone: boolean }`
- Cursor encodes last seen document ID + sort key
- No offset arithmetic (avoids performance degradation)

**Daily Aggregation Pattern:**
- Client-side aggregation for small result sets (current month)
- Server-side aggregation for large queries (historical data)
- Computed field approach: calculate OT/UT status per day
- Pattern: Group by date, sum hours, compare to user thresholds
- Cache aggregation results for repeated queries
- Invalidate cache when worklogs added/updated

#### E2: Anti-Patterns to Avoid

**Offset-Based Pagination (DON'T DO THIS):**
- Problem: Performance degrades with large offsets (OFFSET 10000)
- Database must scan and skip many records
- As users navigate to later pages, queries become slower
- Solution: Always use cursor-based pagination with indexed fields

**Client-Side Aggregation of Large Datasets (AVOID):**
- Problem: Fetching 10 years of worklogs to calculate totals in browser
- Massive data transfer overhead
- Browser memory issues with large arrays
- Solution: Server-side aggregation, return only computed results

**Synchronous Data Migration (WRONG APPROACH):**
- Problem: Single transaction for 22M records
- Timeout issues, memory exhaustion
- Locks database during entire operation
- Solution: Batch processing with checkpoints and resumability

**Hard-Coded User Roles in UI (INSECURE):**
- Problem: Frontend checks like `if (user.role === 'admin')` for access control
- Easily bypassed by modifying client code
- Solution: All authorization enforced in Convex server functions

#### E3: Component Integration Specifications

**Authentication Flow:**
1. User visits protected route
2. React-Router guard checks Clerk authentication status
3. If unauthenticated, redirect to `/login`
4. Clerk handles login flow (email/password, social, etc.)
5. On success, Clerk sets session token in cookies
6. Frontend extracts token and passes to Convex
7. Convex validates token with Clerk backend
8. Convex returns user context (ID, role, metadata)
9. React Router allows access to protected route

**Worklog Creation Flow:**
1. User fills form with validation feedback
2. Client-side validation before submission
3. On submit, call Convex mutation `api.worklogs.create`
4. Mutation extracts authenticated user from context
5. Server-side validation of all inputs
6. Insert worklog document with userId, timestamp
7. Convex automatically triggers query subscriptions
8. UI updates reactively with new worklog
9. Show success message, reset form

**Dashboard Data Loading Flow:**
1. Component mounts, initiates Convex query subscription
2. Query runs server-side with user authentication
3. Apply filters (OT/UT) and pagination parameters
4. Fetch worklogs with indexes on userId + date
5. Server-side aggregation groups by date, calculates totals
6. Compare totals to user's min/max thresholds
7. Attach OT/UT flags to each day group
8. Return paginated results with cursor
9. Component renders grouped data reactively

**Filter State Management:**
1. Filter selections stored in React component state
2. Filter changes trigger new query subscription
3. Convex query receives filter parameters
4. Server applies filters at database level (not in-memory)
5. Use indexed queries for efficient filtering
6. Return filtered results with pagination intact
7. UI updates to show filtered data
8. Display active filter badges/tags

#### E4: Error Handling Requirements

**Network Failure Scenarios:**
- Convex connection lost during query: Show "Connection lost" message with retry button
- Mutation submission timeout: Enable retry with idempotency check
- Clerk authentication service down: Show maintenance message, prevent login attempts
- Graceful degradation: Show cached data with "stale data" indicator

**Validation Error Handling:**
- Client-side validation: Immediate inline feedback on form fields
- Server-side validation failure: Display server error message, override client state
- Date validation: "Future dates not allowed" with date picker constraint
- Hours validation: "Must be between 0.01 and 24 hours" with numeric input constraints

**Authorization Errors:**
- Unauthorized access attempt: Redirect to login with "Session expired" message
- Role mismatch (user accessing admin route): Show 403 forbidden page
- Invalid Clerk token: Clear session, force re-authentication
- Insufficient permissions: Display clear message explaining required access level

**Data Integrity Errors:**
- User not found during worklog creation: Log error, show "User account error, contact support"
- Invalid user ID reference: Reject mutation with clear error
- Database constraint violations: Log to monitoring, show generic error to user
- Corrupted data during migration: Log to error file, skip record, continue processing

#### E5: Testing Requirements

**Unit Testing Scope:**
- Business logic: Daily hours aggregation function (test with various hour combinations)
- OT/UT calculation: Verify correct badge assignment for edge cases
- Validation functions: Test all validation rules with valid/invalid inputs
- Date grouping logic: Verify correct grouping across month/year boundaries
- Filter logic: Test combinations of filters produce correct results

**Integration Testing Scope:**
- Convex mutation: Create worklog end-to-end with authentication
- Query with filters: Verify server-side filtering returns correct subset
- Pagination: Test cursor-based navigation across multiple pages
- Role-based access: Verify admin can access all data, users only their own
- Real-time updates: Create worklog in one session, verify another session updates

**Success Criteria:**
- All validation rules enforced (tested with boundary values)
- Authentication required for all protected operations
- Aggregation calculations accurate (within 0.01 hour precision)
- Pagination returns consistent results across page boundaries
- Filters apply correctly in combination (e.g., user + OT + date range)

### DOCUMENTATION SECTION

#### D1: Official API Documentation

**Convex Documentation:**
- Official Guide: https://docs.convex.dev/home
- Database Queries: https://docs.convex.dev/database/reading-data
- Mutations: https://docs.convex.dev/database/writing-data
- Pagination: https://docs.convex.dev/database/pagination
- Authentication: https://docs.convex.dev/auth
- React Integration: https://docs.convex.dev/client/react

**Clerk Authentication:**
- Official Docs: https://clerk.com/docs
- React Integration: https://clerk.com/docs/references/react/overview
- Convex + Clerk: https://docs.convex.dev/auth/clerk
- Session Management: https://clerk.com/docs/references/react/use-session
- User Metadata: https://clerk.com/docs/users/metadata

**React Router:**
- Official Docs: https://reactrouter.com/
- Protected Routes: https://reactrouter.com/en/main/start/overview#authentication
- Navigation Guards: https://reactrouter.com/en/main/components/navigate

**shadcn/ui Components:**
- Official Docs: https://ui.shadcn.com/
- Form Components: https://ui.shadcn.com/docs/components/form
- Date Picker: https://ui.shadcn.com/docs/components/date-picker
- Data Table: https://ui.shadcn.com/docs/components/data-table
- Badge: https://ui.shadcn.com/docs/components/badge

**Tailwind CSS:**
- Official Docs: https://tailwindcss.com/docs
- Utility Classes: https://tailwindcss.com/docs/utility-first
- Responsive Design: https://tailwindcss.com/docs/responsive-design

#### D2: Architecture Decision Records

**Why Convex for Backend:**
- Real-time data synchronization without manual WebSocket setup
- Serverless scaling handles variable load (10K users)
- Built-in TypeScript support for type-safe queries
- Simplified deployment and infrastructure management
- Strong consistency guarantees for worklog data integrity

**Why Clerk for Authentication:**
- Production-ready authentication with minimal setup
- Seamless integration with Convex
- Built-in user management UI
- Supports multiple authentication methods
- Session management and token refresh handled automatically

**Why Cursor-Based Pagination:**
- Consistent performance regardless of page depth
- Avoids OFFSET performance degradation with large datasets
- Stable results even when data changes during pagination
- Essential for 10 years of historical data (22M potential records)

**Why shadcn/ui Component Library:**
- Built on Radix UI primitives (accessible by default)
- Tailwind-based styling for consistency
- Copy-paste component model (no dependency lock-in)
- Customizable and production-ready components
- Strong TypeScript support

#### D3: Performance Benchmarks

**Query Performance Targets:**
- Personal worklog query (30 days): <200ms
- Admin worklog query (30 days, all users): <500ms
- Worklog creation mutation: <100ms
- Filter application: <300ms
- Pagination navigation: <200ms per page

**Indexing Requirements:**
- Index on `worklogs.userId` for user-scoped queries
- Compound index on `(userId, date)` for sorted personal queries
- Compound index on `(date, userId)` for admin sorted queries
- Index on `users.role` for admin privilege checks

**Scalability Metrics:**
- Support 10,000 concurrent authenticated users
- Handle 1,000 worklog creations per minute
- Serve 10,000 dashboard page loads per minute
- Maintain <1 second p95 latency for all queries
- Data migration: 1,000 records/second minimum

**Resource Limits:**
- Convex free tier: 1M function calls/month, 1GB storage
- Consider scaling to paid tier for production (unlimited functions, 10GB storage)
- Clerk free tier: 5,000 monthly active users
- Production deployment requires Clerk Pro plan

#### D4: Security Guidelines

**Authentication Requirements:**
- All Convex functions must verify authenticated user context
- Reject anonymous requests to protected endpoints
- Use Clerk session tokens for user verification
- Token validation on every request (Convex handles automatically)
- Session timeout: follow Clerk defaults (configurable)

**Authorization Rules:**
- Users can only create worklogs for themselves
- Users can only view their own worklogs (except admins)
- Admin role verified server-side before privileged operations
- No client-side authorization checks for security decisions
- All role checks in Convex functions, not frontend

**Data Protection:**
- User passwords handled by Clerk (never stored in application)
- Worklog data scoped to authenticated user
- No PII in worklog descriptions (user responsibility)
- HTTPS required for all client-server communication
- Clerk session tokens transmitted securely (httpOnly cookies)

**Input Validation:**
- All user inputs validated client-side and server-side
- Server-side validation is source of truth
- Sanitize text inputs to prevent XSS (React handles automatically)
- Numeric inputs validated for range and type
- Date inputs validated for format and logical constraints (no future dates)

### TECHNICAL ARCHITECTURE SECTION

#### A1: Data Models

**User Collection (users):**

Required properties:
- `_id`: Convex-generated unique identifier (string)
- `clerkUserId`: Clerk user ID for authentication mapping (string, unique)
- `name`: User's full name for display (string, 1-100 characters)
- `dailyMinHours`: Minimum expected hours per day (number, 0-24, precision 0.01)
- `dailyMaxHours`: Maximum expected hours per day (number, 0-24, precision 0.01)
- `role`: User role for authorization (enum: "user" | "admin")
- `_creationTime`: Automatic timestamp (Convex system field)

Constraints:
- `dailyMaxHours` must be greater than `dailyMinHours`
- `clerkUserId` must be unique across all users
- `role` defaults to "user" if not specified

Indexes:
- Primary: `_id`
- Secondary: `clerkUserId` (unique)
- Secondary: `role` (for admin queries)

**Worklog Collection (worklogs):**

Required properties:
- `_id`: Convex-generated unique identifier (string)
- `userId`: Reference to user document (string, foreign key to users._id)
- `date`: Date of work (string, ISO 8601 format "YYYY-MM-DD")
- `workedHours`: Hours worked (number, 0.01-24.00, precision 0.01)
- `_creationTime`: Automatic timestamp (Convex system field)

Optional properties:
- `taskId`: Optional task identifier (string, max 50 characters, nullable)
- `description`: Optional work description (string, max 500 characters, nullable)

Constraints:
- `userId` must reference existing user
- `date` cannot be future date (validated server-side)
- `workedHours` must be positive and ≤ 24
- Multiple worklogs per user per date allowed

Indexes:
- Primary: `_id`
- Compound: `(userId, date)` for user worklog queries
- Compound: `(date, userId)` for admin chronological queries
- Single: `userId` for user-scoped operations

#### A2: API Specifications

**Convex Query: `api.worklogs.getMyWorklogs`**

Purpose: Retrieve authenticated user's worklogs with filtering and pagination

Input parameters:
- `filter`: Optional OT/UT filter (enum: "all" | "OT" | "UT" | "normal")
- `cursor`: Optional pagination cursor (string, opaque)
- `pageSize`: Optional results per page (number, default 10, max 100)

Output structure:
- `days`: Array of day groupings
  - Each day contains: `date`, `totalHours`, `status` (OT/UT/normal), `worklogs` array
  - Each worklog contains: `id`, `workedHours`, `taskId`, `description`, `createdAt`
- `nextCursor`: Pagination cursor for next page (string or null)
- `hasMore`: Boolean indicating more results available

Authorization: Requires authenticated user, returns only their worklogs

**Convex Query: `api.worklogs.getAllWorklogs` (Admin only)**

Purpose: Retrieve all users' worklogs with filtering and pagination

Input parameters:
- `filter`: Optional OT/UT filter (enum: "all" | "OT" | "UT" | "normal")
- `userId`: Optional user filter (string, user ID)
- `startDate`: Optional date range start (string, ISO 8601)
- `endDate`: Optional date range end (string, ISO 8601)
- `cursor`: Optional pagination cursor (string, opaque)
- `pageSize`: Optional results per page (number, default 10, max 100)

Output structure:
- Same as `getMyWorklogs` but includes `userName` in each day grouping
- Sorted by date DESC, then userName ASC

Authorization: Requires admin role, verified server-side

**Convex Mutation: `api.worklogs.create`**

Purpose: Create new worklog entry for authenticated user

Input parameters:
- `date`: Work date (string, ISO 8601 format "YYYY-MM-DD")
- `workedHours`: Hours worked (number, 0.01-24.00)
- `taskId`: Optional task identifier (string, max 50 chars, optional)
- `description`: Optional description (string, max 500 chars, optional)

Validation:
- Date must be valid ISO 8601 format
- Date cannot be in future
- workedHours must be 0.01-24.00 range
- taskId/description length limits enforced

Output: Created worklog document ID (string)

Side effects:
- Triggers reactive query updates for user's dashboard
- Logs creation event for audit trail

Authorization: Requires authenticated user, worklog created for that user only

**Convex Query: `api.users.getMe`**

Purpose: Retrieve current authenticated user's profile

Input parameters: None (user ID from authentication context)

Output structure:
- `id`: User ID
- `name`: Display name
- `dailyMinHours`: Minimum daily hours
- `dailyMaxHours`: Maximum daily hours
- `role`: User role

Authorization: Requires authenticated user

#### A3: State Management

**Frontend State Categories:**

Authentication State (Clerk):
- Managed by Clerk React hooks
- `useUser()` provides current user object
- `useSession()` provides session status
- Automatic token refresh and session management

Query State (Convex):
- Managed by Convex React hooks
- `useQuery()` creates reactive subscription
- Automatic re-fetching on data changes
- Loading/error states handled by hook return values

Local UI State (React):
- Form inputs managed by React state or form library
- Filter selections in component state
- Pagination cursor in component state
- Modal open/close states
- Transient UI feedback (toasts, loading spinners)

**State Synchronization:**
- Convex handles real-time data sync via WebSocket
- No manual refresh or polling required
- Optimistic updates for mutations (optional)
- Automatic rollback on mutation failure

**State Persistence:**
- Authentication session persisted in Clerk cookies
- No local storage for worklog data (always fetch from server)
- UI preferences (theme, filter defaults) can use localStorage
- No offline mode required in v1

#### A4: Deployment Strategy

**Local Development:**
- Frontend: `npm run dev` (Vite dev server on localhost:5173)
- Backend: `npx convex dev` (Convex local development mode)
- Database: Convex cloud (dev environment)
- Authentication: Clerk development instance

**Environment Configuration:**
- `.env.local` for local development secrets
- Convex dashboard for backend environment variables
- Clerk dashboard for authentication settings
- No deployed staging environment (localhost only as specified)

**Deployment Targets:**
- Frontend: Vercel, Netlify, or static hosting (build output)
- Backend: Convex cloud (automatic deployment)
- Database: Convex managed database
- No manual infrastructure setup required

**Build Process:**
- Frontend build: `npm run build` (Vite production build)
- Backend deployment: `npx convex deploy` (pushes functions to Convex cloud)
- Type checking: `npm run type-check` before deployment
- Linting: `npm run lint` for code quality

**Rollback Procedure:**
- Frontend: Redeploy previous build from version control
- Backend: Convex maintains function version history, rollback via dashboard
- Database: No schema migrations in v1, data preserved across deployments
- Emergency: Disable authentication to prevent new writes during incident

### OTHER CONSIDERATIONS SECTION

#### C1: Convex Platform Limitations

**Function Execution Limits:**
- Query execution timeout: 30 seconds maximum
- Mutation execution timeout: 30 seconds maximum
- Actions (external API calls): 5 minutes maximum
- Memory limit: 512MB per function invocation
- No cold start delay (Convex keeps functions warm)

**Database Constraints:**
- Document size limit: 1MB per document
- Query results: 8MB maximum per query response
- No server-side joins (must fetch related data in separate queries)
- Indexes required for efficient filtering and sorting
- Schema validation optional but recommended

**Concurrency Characteristics:**
- Optimistic concurrency control (OCC) for mutations
- Automatic retry on transaction conflicts
- No explicit locking mechanisms
- High write throughput supported
- Read scaling via caching and indexes

**Free Tier Limits (Development):**
- 1M function calls per month
- 1GB database storage
- 5GB bandwidth per month
- Sufficient for development and testing
- Production requires paid plan

#### C2: Error Scenarios & Mitigation

**Network Failures:**
- Scenario: User's internet connection lost during worklog submission
- Detection: Convex client detects WebSocket disconnect
- Mitigation: Show "Connection lost" banner, queue mutation for retry
- Recovery: Auto-retry when connection restored, show success confirmation

**Service Outages:**
- Scenario: Convex or Clerk service degradation
- Detection: Multiple consecutive request failures
- Mitigation: Display service status message, prevent new writes
- Recovery: Automatic reconnection when service restored

**Data Corruption:**
- Scenario: Invalid data in worklog (e.g., negative hours due to bug)
- Detection: Server-side validation catches invalid data
- Mitigation: Reject mutation with detailed error message
- Recovery: User corrects input and resubmits

**Authentication Failures:**
- Scenario: Clerk session expires during active use
- Detection: Convex returns authentication error
- Mitigation: Clerk automatically attempts token refresh
- Recovery: If refresh fails, redirect to login page

**Migration Failures:**
- Scenario: Data import encounters malformed records
- Detection: Validation error during batch processing
- Mitigation: Log error to file, skip record, continue processing
- Recovery: Review error log, fix source data, re-import failed records

#### C3: Performance Bottlenecks

**Large Historical Data Queries:**
- Issue: Fetching 10 years of worklogs without pagination
- Symptom: Slow page loads, high memory usage
- Prevention: Always use pagination with reasonable page sizes
- Mitigation: Implement cursor-based pagination, server-side aggregation

**N+1 Query Pattern:**
- Issue: Fetching user details for each worklog separately
- Symptom: Multiple database queries for single page render
- Prevention: Denormalize user name into worklog, or batch fetch users
- Mitigation: Use Convex batching or include user data in worklog query

**Unindexed Queries:**
- Issue: Filtering or sorting without database index
- Symptom: Query slowness increasing with data volume
- Prevention: Create compound indexes on filter/sort columns
- Mitigation: Add indexes for `(userId, date)` and `(date, userId)`

**Client-Side Aggregation:**
- Issue: Summing hours in browser for large date ranges
- Symptom: Slow UI rendering, high CPU usage
- Prevention: Server-side aggregation for all queries
- Mitigation: Compute daily totals in Convex query, return aggregated data

**Concurrent Write Conflicts:**
- Issue: Multiple admins modifying same user's data simultaneously
- Symptom: Transaction conflicts, automatic retries
- Prevention: Design for append-only worklog model (rare conflicts)
- Mitigation: Convex handles retries automatically, exponential backoff

#### C4: Security Vulnerabilities

**Cross-Site Scripting (XSS):**
- Attack Vector: Malicious script in worklog description
- Defense: React automatically escapes output (safe by default)
- Validation: No additional sanitization needed for display
- Testing: Verify special characters rendered as text, not executed

**Injection Attacks:**
- Attack Vector: SQL injection via user inputs
- Defense: Convex uses parameterized queries (safe by default)
- Validation: Input validation prevents malformed data
- Testing: Test with SQL injection payloads, verify rejection

**Broken Authentication:**
- Attack Vector: Session hijacking, token theft
- Defense: Clerk handles secure session management (httpOnly cookies)
- Validation: HTTPS required for all communication
- Testing: Verify session expiration, token refresh behavior

**Broken Access Control:**
- Attack Vector: User modifying URL to access other users' worklogs
- Defense: Server-side authorization in every Convex function
- Validation: User ID from authenticated context, not client input
- Testing: Verify user cannot access other users' data via API manipulation

**Sensitive Data Exposure:**
- Attack Vector: PII in worklog descriptions logged to monitoring
- Defense: Avoid logging user-generated content
- Validation: User education about appropriate worklog descriptions
- Testing: Review logs for accidental PII exposure

#### C5: Operational Requirements

**Monitoring & Observability:**
- Metrics to track: Query latency (p50, p95, p99), mutation success rate, error rate
- Dashboard: Convex built-in dashboard for function metrics
- Alerting: Set up alerts for error rate >5%, latency >1s
- Logging: Convex automatic logging of function executions and errors
- Custom metrics: Log business events (worklog created, user login, etc.)

**Error Reporting:**
- Client errors: Log to browser console, send to error tracking service (e.g., Sentry)
- Server errors: Convex logs all function errors automatically
- Validation errors: Log to Convex with context (user ID, input data)
- Critical alerts: Email/Slack notification for high error rates

**Debugging Capabilities:**
- Convex dashboard: View function execution history, logs, performance
- Browser dev tools: React DevTools, Network tab for query inspection
- Clerk dashboard: User session history, authentication events
- Local debugging: Console logs, React strict mode for development

**Backup & Recovery:**
- Database backups: Convex handles automatic backups
- Point-in-time recovery: Available via Convex support
- Data export: Convex CLI tools for exporting collections
- Restore testing: Verify backup restore procedure in development

**Maintenance Procedures:**
- Schema changes: Deploy new Convex functions, maintain backward compatibility
- Data migrations: Write migration scripts as Convex actions
- User management: Admin UI for basic operations, Clerk dashboard for advanced
- Performance tuning: Monitor slow queries, add indexes as needed

**Incident Response:**
- Severity levels: P0 (system down), P1 (degraded), P2 (minor issue)
- On-call rotation: Define for production deployment
- Runbooks: Document common issues and resolution steps
- Post-mortem: Required for P0/P1 incidents

## Validation Framework

### Implementation Checkpoints

**Phase 1: Foundation (Authentication & Data Models)**
- [ ] Clerk authentication integrated with React app
- [ ] Protected routes redirect unauthenticated users
- [ ] Convex project initialized with TypeScript
- [ ] User and worklog schemas defined in Convex
- [ ] Database indexes created for required query patterns
- [ ] Seed script creates 3 users (user1, user2, admin) with correct thresholds
- [ ] Validation: Can log in with each seeded user, see correct role

**Phase 2: Core Functionality (Worklog Creation)**
- [ ] Worklog creation form with all required fields
- [ ] Client-side validation for date, hours, optional fields
- [ ] Convex mutation creates worklog with authenticated user ID
- [ ] Server-side validation enforces business rules
- [ ] Success/error feedback displayed to user
- [ ] Validation: Create worklog, verify stored in database with correct user ID
- [ ] Validation: Submit invalid data, verify error messages displayed

**Phase 3: Personal Dashboard**
- [ ] Query fetches authenticated user's worklogs
- [ ] Worklogs grouped by date (descending order)
- [ ] Daily hours aggregated per day
- [ ] OT/UT badges displayed based on user's thresholds
- [ ] Pagination implemented with cursor-based navigation
- [ ] Validation: Create worklogs spanning multiple days, verify grouping and totals
- [ ] Validation: Create OT day (>max hours), verify OT badge appears

**Phase 4: Filtering & Pagination**
- [ ] Filter by OT, UT, normal days
- [ ] Filter state updates query parameters
- [ ] Pagination maintains filter state
- [ ] Clear filter button resets to "all"
- [ ] Validation: Apply OT filter, verify only OT days shown
- [ ] Validation: Navigate to page 2, verify filter still applied

**Phase 5: Admin Dashboard**
- [ ] Admin-only query fetches all users' worklogs
- [ ] User name displayed in each worklog entry
- [ ] Sorting by date DESC, then user name ASC
- [ ] Admin can filter by specific user
- [ ] Authorization check prevents non-admins from accessing
- [ ] Validation: Log in as admin, verify see all users' data
- [ ] Validation: Log in as user, verify cannot access admin endpoints

**Phase 6: Stub Pages & Navigation**
- [ ] All menu items present for respective roles
- [ ] Stub pages display descriptive text
- [ ] Navigation between implemented and stub pages works
- [ ] Validation: Navigate to each menu item, verify correct page displays

**Phase 7: Data Seeding**
- [ ] Seed script generates 10 years of worklog data
- [ ] Distribution includes OT, UT, and normal days
- [ ] Script runs in <5 minutes
- [ ] Idempotent execution (can re-run safely)
- [ ] Validation: Run seed script, verify data count matches expected range
- [ ] Validation: Query admin dashboard, verify see historical data

### Automated Testing Requirements

**Unit Tests (Required):**
- Daily hours aggregation logic (various input combinations)
- OT/UT badge calculation (boundary conditions: exactly at min/max)
- Date grouping function (across month/year boundaries)
- Validation functions (valid/invalid inputs for each field)
- Filter logic (OT, UT, normal, combined filters)

**Integration Tests (Required):**
- End-to-end worklog creation flow (authenticated user)
- Personal dashboard query with filters
- Admin dashboard query with user filter
- Authorization checks (user accessing admin endpoint)
- Pagination navigation (cursor-based)

**Manual Testing Checklist:**
- [ ] Anonymous user sees only login and help pages
- [ ] User can create worklog with all fields populated
- [ ] User can create worklog with only required fields
- [ ] Future date rejected with appropriate error message
- [ ] Hours outside 0.01-24 range rejected
- [ ] Personal dashboard shows only authenticated user's worklogs
- [ ] OT badge appears when daily total exceeds user's max
- [ ] UT badge appears when daily total below user's min
- [ ] Filter by OT shows only days exceeding max hours
- [ ] Pagination loads additional pages correctly
- [ ] Admin sees all users' worklogs with names
- [ ] Admin dashboard sorts by date then user name
- [ ] Non-admin user cannot access admin endpoints
- [ ] Stub pages display for unimplemented features

### Performance Validation

**Query Performance Tests:**
- Personal worklog query (30 days): Measure latency, target <200ms
- Admin worklog query (30 days, all users): Measure latency, target <500ms
- Worklog creation: Measure latency, target <100ms
- Pagination navigation: Measure latency, target <200ms per page

**Load Testing (Optional for v1, Required for Production):**
- Simulate 100 concurrent users creating worklogs
- Simulate 500 concurrent dashboard page loads
- Measure error rate under load (target <1%)
- Verify no degradation beyond 2x latency increase

**Data Volume Testing:**
- Seed database with 20,000 worklogs (10 years for 3 users)
- Verify query performance remains within targets
- Test pagination through 100+ pages
- Verify aggregation calculations remain accurate

## Operational Guidelines

### Development Workflow

**Local Setup:**
1. Clone repository
2. Install dependencies: `npm install`
3. Set up Clerk development instance, add API keys to `.env.local`
4. Initialize Convex: `npx convex dev`
5. Run seed script: `npm run seed`
6. Start frontend: `npm run dev`
7. Verify: Access http://localhost:5173, log in with seeded user

**Code Quality Standards:**
- TypeScript strict mode enabled
- All Convex functions must be typed
- React components use TypeScript
- ESLint + Prettier for code formatting
- Pre-commit hooks run linter and type checks
- No console.logs in production code (use proper logging)

**Version Control:**
- Feature branches for all changes
- Pull requests required for main branch
- PR description includes testing performed
- At least one reviewer approval required
- Squash merge to maintain clean history

### Deployment Process

**Pre-Deployment Checklist:**
- [ ] All unit tests passing
- [ ] All integration tests passing
- [ ] TypeScript compilation successful
- [ ] ESLint warnings addressed
- [ ] Manual testing completed per checklist
- [ ] Performance benchmarks met

**Deployment Steps:**
1. Run production build: `npm run build`
2. Verify build output contains no errors
3. Deploy Convex functions: `npx convex deploy --prod`
4. Deploy frontend to hosting platform (Vercel/Netlify)
5. Smoke test: Log in, create worklog, verify dashboard loads

**Post-Deployment Verification:**
- [ ] Authentication works (login/logout)
- [ ] Worklog creation succeeds
- [ ] Dashboard loads with correct data
- [ ] Filters and pagination functional
- [ ] No console errors in browser
- [ ] Convex dashboard shows no error spikes

**Rollback Procedure:**
1. Identify issue (authentication, data, performance)
2. Notify team of rollback decision
3. Revert frontend deployment to previous version
4. Rollback Convex functions via dashboard
5. Verify system restored to working state
6. Post-mortem to analyze root cause

### Monitoring & Maintenance

**Daily Monitoring:**
- Check Convex dashboard for error rate trends
- Review Clerk dashboard for authentication issues
- Monitor application performance metrics
- Check for unusual usage patterns

**Weekly Maintenance:**
- Review slow query reports from Convex
- Analyze user feedback and bug reports
- Update dependencies (security patches)
- Review database growth trends

**Monthly Tasks:**
- Performance benchmark comparison (month-over-month)
- Security review of authentication and authorization
- Capacity planning based on user growth
- Update documentation for any changes

### Troubleshooting Guide

**Issue: User cannot log in**
- Check Clerk dashboard for service status
- Verify Clerk API keys in environment variables
- Check browser console for errors
- Test with different browser/incognito mode
- Verify user account exists in Clerk

**Issue: Worklog not appearing after creation**
- Check Convex function logs for mutation errors
- Verify user authentication context passed correctly
- Check database for worklog document (Convex dashboard)
- Verify query subscription re-fetching after mutation
- Test with browser refresh to rule out cache issue

**Issue: Slow dashboard loading**
- Check Convex query execution time in dashboard
- Verify indexes exist on query filter/sort columns
- Review query result size (may need pagination adjustment)
- Check user's network speed (DevTools Network tab)
- Verify no unnecessary data fetching (N+1 queries)

**Issue: OT/UT badges incorrect**
- Verify user's dailyMinHours and dailyMaxHours values
- Check daily aggregation calculation logic
- Test with manual calculation of daily totals
- Verify comparison logic (>, <, ===) in badge assignment
- Check for floating-point precision issues (0.01 rounding)

**Issue: Admin cannot see all users' worklogs**
- Verify admin user has correct role assignment
- Check server-side authorization logic in Convex function
- Review Clerk user metadata for role field
- Test with database query directly (Convex dashboard)
- Verify no client-side filtering hiding results

---

## Summary

This Product Requirements Prompt provides comprehensive specification for building a production-ready worklog tracking system. The system handles 10,000 users with 10 years of historical data using a modern serverless architecture with Convex backend, React frontend, and Clerk authentication.

**Key Implementation Priorities:**
1. Authentication and role-based access control (foundation)
2. Core worklog creation with validation
3. Personal dashboard with aggregation and OT/UT detection
4. Admin dashboard with multi-user view
5. Filtering and pagination for performance at scale
6. Data seeding for realistic testing

**Critical Success Factors:**
- Server-side authorization enforced in all Convex functions
- Cursor-based pagination for large datasets
- Database indexes on all query filter/sort columns
- Comprehensive validation (client-side and server-side)
- Real-time UI updates via Convex subscriptions

**Non-Negotiable Requirements:**
- No future dates in worklogs
- Users can only access their own data (except admins)
- OT/UT badges calculated server-side based on user-specific thresholds
- All sensitive operations require authentication
- Performance targets met for query latency

The AI implementer should follow this specification to build a complete, production-ready worklog tracking system that meets all functional requirements, performance targets, and security standards.
