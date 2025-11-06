# Product Requirements Prompt: Worklog Tracking System

## Executive Summary

Build a scalable worklog tracking system for a 10K-user company that enables employees to log work hours with automatic overtime/undertime detection, provides admins with comprehensive workforce visibility, and migrates 10 years of historical worklog data. The system uses React frontend with Clerk authentication, Convex serverless backend, and supports role-based access control with three permission tiers: anonymous users (login page only), registered users (worklog management), and admins (workforce monitoring and user configuration).

---

## Technical Architecture

### System Components Overview

```
┌─────────────────────────────────────────────────────────────┐
│                     Client Layer (React)                     │
│  ┌──────────────┬──────────────┬──────────────────────────┐  │
│  │   Auth       │   User Pages │    Admin Pages           │  │
│  │   (Clerk)    │   (Worklogs) │    (All Users/Config)    │  │
│  └──────────────┴──────────────┴──────────────────────────┘  │
└──────────────────────────┬──────────────────────────────────┘
                           │ HTTP/REST
┌──────────────────────────┴──────────────────────────────────┐
│                    API Layer (Convex)                        │
│  ┌────────────────────────────────────────────────────────┐  │
│  │  Auth Middleware  │  User API  │  Admin API            │  │
│  └────────────────────────────────────────────────────────┘  │
└──────────────────────────┬──────────────────────────────────┘
                           │
┌──────────────────────────┴──────────────────────────────────┐
│              Data Layer (Convex Database)                    │
│  ┌──────────────┬──────────────┬──────────────────────────┐  │
│  │    Users     │   Worklogs   │  User Settings           │  │
│  │              │              │  (Min/Max Hours)         │  │
│  └──────────────┴──────────────┴──────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### Data Model & Relationships

**Users Collection:**
- Primary key: `userId` (string, from Clerk)
- `name` (string): Full user name
- `email` (string): Email address (unique)
- `dailyMinHours` (number): Minimum work hours required per day
- `dailyMaxHours` (number): Maximum work hours per day (standard workday limit)
- `role` (enum): 'user' | 'admin'
- `createdAt` (timestamp): Account creation date
- `updatedAt` (timestamp): Last profile update

**Worklogs Collection:**
- Primary key: `logId` (string)
- `userId` (string): Foreign key to Users
- `date` (ISO date string): YYYY-MM-DD format for day grouping
- `workedHours` (number): Hours worked (positive float, e.g., 8.5)
- `taskId` (string, optional): Reference to task/project
- `description` (string, optional): Work description/notes
- `createdAt` (timestamp): Log creation time
- `updatedAt` (timestamp): Last modification

**Computed Fields (calculated on-read, not stored):**
- `dailyTotalHours`: Sum of all workedHours for a given user/day
- `isUndertime`: boolean (dailyTotalHours < user.dailyMinHours)
- `isOvertime`: boolean (dailyTotalHours > user.dailyMaxHours)
- `timeStatus`: enum ('normal' | 'undertime' | 'overtime')

### API Contract & Endpoints

**Authentication:**
- Clerk integration handles all auth flows (sign-up, sign-in, sign-out)
- Convex queries/mutations validate `auth.getUserIdentity()` on every protected operation
- Role verification occurs at Convex mutation/query level before data access

**User API Endpoints:**

1. **Create Worklog** (POST equivalent)
   - Input: { date: string (ISO), workedHours: number, taskId?: string, description?: string }
   - Returns: { logId, userId, date, workedHours, createdAt }
   - Validation: workedHours > 0, workedHours <= 24, date is valid ISO date
   - Side effect: Timestamp recorded, authenticated user matched to userId

2. **List Worklogs** (GET equivalent - paginated)
   - Parameters: { limit: number (default 20), cursor?: string, filterBy?: 'all' | 'undertime' | 'overtime', sortOrder?: 'desc' (default) | 'asc' }
   - Returns: { worklogs: Array<{ date, dailyTotalHours, logCount, timeStatus, logsForDay: Array<worklog> }>, cursor: string | null, hasMore: boolean }
   - Data: Grouped by date, most recent first (default)
   - Computed: dailyTotalHours, timeStatus per day

3. **Update Worklog** (PATCH equivalent)
   - Input: { logId, workedHours?: number, taskId?: string, description?: string }
   - Returns: Updated worklog object
   - Authorization: User can only update their own logs
   - Validation: Same as create worklog

4. **Delete Worklog** (DELETE equivalent)
   - Input: { logId }
   - Authorization: User can only delete their own logs
   - Returns: { success: boolean }

5. **Get User Profile** (GET equivalent)
   - Returns: { userId, name, email, dailyMinHours, dailyMaxHours, role }

**Admin API Endpoints:**

1. **List All Users** (GET equivalent)
   - Parameters: { limit: number (default 20), cursor?: string, sortBy?: 'name' | 'email' (default: 'name') }
   - Returns: { users: Array<{ userId, name, email, dailyMinHours, dailyMaxHours, role }>, cursor: string | null, hasMore: boolean }

2. **Update User Settings** (PATCH equivalent)
   - Input: { userId, dailyMinHours?: number, dailyMaxHours?: number }
   - Authorization: Admin only
   - Validation: dailyMinHours >= 0, dailyMaxHours > dailyMinHours
   - Returns: Updated user object

3. **List All Worklogs** (GET equivalent - paginated)
   - Parameters: { limit: number (default 20), cursor?: string, filterUserId?: string, filterDate?: string, filterTimeStatus?: 'all' | 'undertime' | 'overtime', sortBy?: 'date' | 'user' (default: both desc), sortOrder?: 'desc' }
   - Returns: { worklogs: Array<{ userId, userName, date, dailyTotalHours, timeStatus, logCount }>, cursor: string | null, hasMore: boolean }
   - Data: Grouped by date, then by user name
   - Sorting: Most recent date first, within date alphabetically by user name

4. **Get User Worklog History** (GET equivalent - same as user endpoint, but admin can view any user)
   - Parameters: { userId, limit: number, cursor?: string, filterTimeStatus?: 'all' | 'undertime' | 'overtime' }
   - Returns: Same as user worklog list structure
   - Authorization: Admin can request any userId

### State Management

**Frontend State Management (Convex + React Query/SWR pattern):**
- User authentication state managed by Clerk context
- Worklog data fetched via Convex hooks (useQuery for reads, useMutation for writes)
- Pagination state: current cursor, limit, total items (if available)
- Filter state: selected filters persist in URL query params
- Loading/error states for each API call
- Optimistic updates for worklog creation (show pending state before confirmation)

**Convex Backend State:**
- User role/permissions cached in Convex auth context for each request
- Data consistency enforced through Convex's ACID transaction model
- No session state needed (Convex handles request isolation)

---

## Implementation Specification

### FEATURE: Role-Based Access Control

**Anonymous Users:**
- Access: Login page (via Clerk sign-in component) and help/documentation page
- Pages: `/` (redirects to `/login` if unauthenticated), `/login` (Clerk sign-in widget), `/help` (static content with lorem ipsum)
- Navigation: Show login link in header, hide all other menu items
- No database access required

**Registered Users:**
- Access: All authenticated user features
- Pages:
  - `/dashboard` (redirect from `/` for logged-in users)
  - `/worklogs` (create new worklog form + list existing worklogs)
  - `/help` (static help content)
- Menu items: Dashboard, Create Worklog, My Worklogs, Help, Profile, Logout
- Permissions: Can create, read, update, delete only their own worklogs; can view their own profile settings

**Admin Users:**
- Access: All user features + admin features
- Additional pages:
  - `/admin/users` (list all users with min/max hour configuration)
  - `/admin/worklogs` (view all worklogs across all users)
  - `/admin/settings` (configure user permissions, if needed)
- Additional menu items: Users Management, All Worklogs, Admin Settings
- Permissions: Can create worklogs for themselves, view/edit all user worklogs, modify user hour settings, view system-wide analytics

**Authentication Flow:**
1. User lands on app (unauthenticated)
2. Clerk detects no session, shows login page or redirect to sign-in
3. User completes Clerk sign-up/sign-in flow
4. Clerk sets user identity and returns to app
5. Convex auth middleware checks role via user.role field
6. Frontend routes protected by role-based guards (redirect if insufficient permissions)
7. All API calls include Clerk JWT in headers; Convex validates token and role on each request

**Error Handling:**
- Invalid auth token: Redirect to login
- Insufficient permissions: Show 403 Forbidden page with explanation
- Session expired: Trigger re-authentication via Clerk
- Role mismatch: Log security event, redirect to appropriate dashboard

### FEATURE: Worklog Creation & Management (User)

**Create Worklog:**
- UI: Form with fields: Date (date picker, default today), Worked Hours (number input, step 0.5), Task ID (optional text), Description (optional textarea)
- Validation rules:
  - Date: Must be today or in the past (within 90 days to prevent data entry errors)
  - Worked Hours: Must be > 0, <= 24, decimal to 1 decimal place (e.g., 8.5, not 8.55)
  - Task ID: If provided, must be non-empty string
  - Description: Optional, max 500 characters
- Submission: User clicks "Save Worklog"
  - UI shows loading state
  - API call to `createWorklog(date, workedHours, taskId, description)`
  - On success: Show toast confirmation, reset form, refresh worklog list
  - On error: Show error toast with message (e.g., "Failed to save worklog. Please try again.")
- Duplicate detection: Allow multiple logs per day (user can split work across multiple tasks)

**Update Worklog:**
- UI: Inline edit or separate edit modal triggered by clicking worklog entry
- Fields: Same as create (date can be changed, but limited to past dates)
- Submission: "Update" button triggers `updateWorklog(logId, { workedHours, taskId, description })`
- Optimistic update: Show updated value immediately, confirm from API
- On error: Revert to previous value, show error toast

**Delete Worklog:**
- UI: Delete button (trash icon) on each worklog entry
- Confirmation: "Are you sure? This cannot be undone."
- API: `deleteWorklog(logId)`
- On success: Remove entry from list, show confirmation toast
- On error: Show error message

### FEATURE: Worklog List & Display (User)

**Worklog List View:**
- Data source: Convex query `listUserWorklogs(userId, { limit, cursor, filterBy })`
- Grouping: By date (YYYY-MM-DD), most recent date first
- Per-day display:
  - Date header (e.g., "Monday, November 4, 2025")
  - Daily total hours (e.g., "Total: 8.5 hours")
  - Time status indicator:
    - Green checkmark or "Normal" badge if dailyTotalHours between min/max
    - Red "UT" (Undertime) badge if below dailyMinHours (e.g., "UT: 3.5 < 7.0 min")
    - Orange "OT" (Overtime) badge if above dailyMaxHours (e.g., "OT: 10.5 > 9.0 max")
  - List of individual worklogs for that day (each showing: hours, task, description, timestamps, edit/delete buttons)

**Filtering:**
- Default: Show all logs
- Filter options:
  - "All logs"
  - "Undertime days only" (days where total < dailyMinHours)
  - "Overtime days only" (days where total > dailyMaxHours)
- Filter state: Persisted in URL query param (e.g., `?filter=overtime`)

**Pagination:**
- Entries per page: 20 (configurable, but default 20)
- Load more: "Next" button at bottom, disabled if no more data
- Cursor-based pagination: Track cursor in state, send with next query
- Display: "Showing 1-20 of ~500" (if total available) or "Showing 1-20 (load more available)"

**Sorting:**
- Default: By date, most recent first
- Sort options:
  - Date descending (default)
  - Date ascending
- Sort state: Persisted in URL query param

**Performance Considerations:**
- Only fetch current page of data (20 entries)
- Lazy load additional pages on demand
- Cache results in frontend state to avoid refetching
- Debounce filter changes before API call (300ms)

### FEATURE: Admin Worklog Monitoring

**All Worklogs View:**
- Data source: Convex query `listAllWorklogs(query object with filters)`
- Display: Table or card list grouped by date, then by user name within each date
- Columns/fields:
  - Date
  - User Name (linked to user profile/detail page)
  - Daily Total Hours
  - Time Status (Normal/UT/OT badge)
  - Number of entries for that day
- Filtering:
  - By date range (from-to date pickers, optional)
  - By user name (text search or dropdown)
  - By time status (All / Undertime / Overtime)
- Sorting:
  - Primary: By date (descending, most recent first)
  - Secondary: By user name (alphabetical, A-Z)
- Pagination: Same as user worklog list (20 per page, cursor-based)
- Actions: Click on user row to view detailed worklog for that user+day

**User Worklog Detail (Admin View):**
- Shows same layout as user's own view, but for selected user
- Additional info: User's current min/max hours displayed
- Ability to modify min/max hours inline or via modal (triggers `updateUserSettings`)
- Back button to return to all worklogs view

**User Management View:**
- Data source: Convex query `listAllUsers({ limit, cursor, sortBy })`
- Display: Table with columns:
  - User Name
  - Email
  - Daily Min Hours (editable field)
  - Daily Max Hours (editable field)
  - Role (display as badge)
  - Actions (edit/view worklogs)
- Sorting:
  - By name (A-Z, default)
  - By email
- Pagination: 20 per page, cursor-based

**Edit User Settings Modal:**
- Trigger: Click edit button on user row
- Fields:
  - Daily Min Hours: Number input, >= 0
  - Daily Max Hours: Number input, > dailyMinHours
- Validation:
  - Min < Max
  - Both non-negative
  - Decimal allowed (e.g., 7.5 hours)
- Submission: `updateUserSettings(userId, { dailyMinHours, dailyMaxHours })`
- On success: Update table row, show confirmation toast
- On error: Show error message in modal

### FEATURE: Data Seeding & Migration

**Development Seeding:**
- Seed script: Runs on first dev environment setup (or manual trigger)
- Seed data:
  - User 1: name="Alice Johnson", dailyMinHours=3, dailyMaxHours=5, role="user"
  - User 2: name="Bob Smith", dailyMinHours=7, dailyMaxHours=9, role="user"
  - Admin: name="Admin User", email="admin@example.com", dailyMinHours=8, dailyMaxHours=10, role="admin"
  - Worklogs: For each user, create 2-3 logs per day for last 30 days
    - Include cases where user has undertime (worked < min)
    - Include cases where user has overtime (worked > max)
    - Include cases where user has normal hours (within range)
    - Vary task IDs and descriptions

**Data Migration (10-year historical data):**
- Source: CSV or JSON file with historical worklogs
- Migration script: Node.js script that reads file, validates records, inserts into Convex
- Validation: Ensure dates are valid, hours > 0, users exist in system
- Error handling: Log invalid records, skip them, continue processing
- Post-migration: Verify record counts match expected total
- Rollback: Keep migration logs, allow re-running from checkpoint if needed

### FEATURE: Responsive UI & Navigation

**Layout:**
- Header: Logo, app title, user name + avatar dropdown (Clerk managed), logout button
- Sidebar/Nav: Menu items based on role
  - Anonymous: Login, Help
  - User: Dashboard, Create Worklog, My Worklogs, Help, Profile
  - Admin: Dashboard, Create Worklog, My Worklogs, Users, All Worklogs, Help, Profile
- Footer: Optional company info or copyright
- Mobile: Hamburger menu instead of sidebar on screens < 768px

**Styling:**
- Use Tailwind CSS for responsive design
- shadcn components: Button, Input, Card, Table, Badge, Dialog, Dropdown, Pagination, Toast/Notification
- Color scheme: Professional (blues, grays), badges with semantic colors (green=normal, red=undertime, orange=overtime)
- Typography: Clear hierarchy, readable fonts

**Accessibility:**
- ARIA labels on interactive elements
- Keyboard navigation supported (Tab, Enter, Escape)
- Color contrast meets WCAG AA standard
- Form labels associated with inputs

### FEATURE: Error Handling & Edge Cases

**Network Errors:**
- Failed API call: Show toast with "Network error. Please try again." with retry button
- Retry mechanism: Exponential backoff (1s, 2s, 4s) up to 3 attempts
- Offline detection: Show banner "You are offline" if connection lost, hide when restored

**Data Validation Errors:**
- Invalid input (e.g., negative hours): Show field-level error message
- Submission fails: Show toast with specific error (e.g., "Hours cannot exceed 24")
- Server rejects data: Show error message from API

**Concurrency Issues:**
- User edits same log from multiple tabs: Last update wins (Convex handles this)
- Show warning: "This entry was modified elsewhere. Refresh to see latest." if detected
- Conflict resolution: Refresh on user demand, show latest state

**Auth & Permission Errors:**
- Unauthorized (401): Redirect to login
- Forbidden (403): Show 403 page, offer redirect to dashboard
- Session expired: Show modal "Your session expired. Please sign in again." with link to re-authenticate

**Data Edge Cases:**
- User with no worklogs: Show empty state "No worklogs yet. Create your first one!"
- User with no hours logged: Show time status "No entries for today"
- Date in future: Prevent creation, show error "Cannot log hours for future dates"
- Zero worklogs for admin view: Show "No worklogs found" message

---

## EXAMPLES: Architecture Patterns & Integration Specifications

### Example 1: Worklog Creation Flow with Validation

**Pattern: Client-side validation + Server-side validation + Optimistic updates**

When user submits worklog form:
1. Client validates inputs (date is valid, hours > 0 && < 24, description < 500 chars)
2. If validation fails, show field errors and prevent submission
3. If valid, show loading spinner, disable form inputs
4. Send to Convex mutation `createWorklog({ date, workedHours, taskId, description })`
5. Convex mutation:
   - Validates auth (user must be authenticated)
   - Validates data (same rules as client)
   - Enforces business rule (date must be today or earlier)
   - Creates record in Worklogs collection with userId and timestamps
   - Returns created record with ID
6. On success:
   - Frontend receives logId and new record
   - Add to local worklog list (optimistic update visible immediately)
   - Show toast "Worklog created"
   - Clear form and date picker
7. On error:
   - Show toast with error message
   - If rate limit (too many requests), show "Please wait X seconds"
   - If auth error, redirect to login

**Anti-patterns to avoid:**
- Don't trust client timestamp; use server timestamp
- Don't allow future dates even if client validation bypassed
- Don't skip server-side validation
- Don't show generic "Something went wrong" errors; be specific

### Example 2: Admin Viewing All Worklogs with Filtering

**Pattern: Parameterized query with cursor-based pagination**

Admin loads "All Worklogs" page:
1. Convex query `listAllWorklogs` receives parameters:
   - `limit`: 20
   - `cursor`: null (first page)
   - `filters`: { userId: null, dateRange: null, timeStatus: 'all' }
   - `sort`: { field: 'date', order: 'desc' }
2. Query execution:
   - Fetch up to 21 worklogs (limit + 1 to detect if more exist)
   - Filter by conditions (if userId provided, only that user; if dateRange, within range; if timeStatus, compute and filter)
   - Sort by date descending, then by user name ascending
   - Return first 20 results + cursor for next page
3. Frontend display:
   - Group results by date
   - Within each date, order by user name
   - Display date header, then user rows
   - Show "Load More" button if hasMore=true
4. User applies filter (e.g., "Undertime only"):
   - Update filter state in URL: `?status=undertime`
   - Trigger new query with updated filter
   - Reset cursor to null (start from page 1)
   - Show loading skeleton while fetching
5. User clicks "Load More":
   - Send cursor from previous result
   - Append new results to list
   - Update cursor for next page

**Anti-patterns to avoid:**
- Don't fetch all records and filter in frontend (scales poorly with 10K users)
- Don't use offset-based pagination; use cursor-based for better performance
- Don't re-sort client-side; let database sort
- Don't cache old results when filter changes; invalidate and refetch

### Example 3: Time Status Calculation (Undertime/Overtime Detection)

**Pattern: Computed field calculated on query with aggregation**

When listing worklogs:
1. Group all Worklogs by (date, userId)
2. For each group:
   - Sum workedHours across all logs for that day → dailyTotalHours
   - Look up user's dailyMinHours and dailyMaxHours
   - Compute timeStatus:
     - If dailyTotalHours < dailyMinHours: "undertime"
     - If dailyTotalHours > dailyMaxHours: "overtime"
     - Otherwise: "normal"
3. Return grouped results with calculated fields

**Data structure returned:**
```
{
  date: "2025-11-06",
  userId: "user123",
  userName: "Alice Johnson",
  dailyTotalHours: 6.5,
  dailyMinHours: 7,
  dailyMaxHours: 9,
  timeStatus: "undertime",
  logCount: 2,
  logsForDay: [
    { logId: "...", workedHours: 4, taskId: "...", description: "..." },
    { logId: "...", workedHours: 2.5, taskId: "...", description: "..." }
  ]
}
```

**Database considerations:**
- Create index on (date, userId) for fast grouping queries
- Consider pre-computing daily summaries if millions of logs exist
- Cache timeStatus calculation result if possible to avoid recomputing on every query

**Anti-patterns to avoid:**
- Don't calculate timeStatus in frontend; compute in database
- Don't store timeStatus as a field; compute on-read to keep data fresh
- Don't fetch all logs then filter; let database do filtering before returning

### Example 4: Auth & Role-Based Access Control

**Pattern: Auth middleware + Role verification at API layer**

Every protected Convex function:
```
1. Entry: mutation/query receives parameters
2. Auth check:
   - Call auth.getUserIdentity()
   - If null/undefined, throw error "Not authenticated"
   - Extract userId from identity
3. Fetch user document to get role
4. Role check (based on endpoint):
   - For user endpoints: No additional check (user can access their own data)
   - For admin endpoints: Check if role === 'admin', throw error if not
5. Data access:
   - If role=admin, access all data
   - If role=user, filter to only own data (userId match)
6. Return response with data + metadata
```

**Error responses:**
- `{ error: "Not authenticated", status: 401 }`
- `{ error: "Forbidden: Admin access required", status: 403 }`
- Frontend handles these with appropriate UI (redirect to login, show 403 page)

**Token handling:**
- Clerk provides JWT in Authorization header (handled by Clerk SDK)
- Convex validates token automatically
- No manual token management needed on frontend

**Anti-patterns to avoid:**
- Don't skip auth check for "safe" read operations
- Don't rely on client-side role; always verify server-side
- Don't return sensitive data then filter in frontend
- Don't store tokens in localStorage (Clerk manages this securely)

### Example 5: Handling Concurrent Updates & Race Conditions

**Pattern: Last-write-wins with conflict detection**

Scenario: User updates same worklog from two browser tabs
1. Tab A submits update: `updateWorklog(logId, { workedHours: 8 })`
2. Tab B (almost simultaneously) submits: `updateWorklog(logId, { description: "Fixed" })`
3. Both reach Convex mutation:
   - Convex ACID transactions ensure one completes first
   - First update: Finds document, updates workedHours, updates timestamp
   - Second update: Finds document (now has updated timestamp), updates description, updates timestamp
   - Both changes are preserved (merged)
4. Tab A receives response with new data (includes B's description update)
5. Tab B receives response with new data (includes A's hours update)
6. If frontend caches old value:
   - Show warning: "This record was updated elsewhere"
   - Offer to refresh and reload from server

**Conflict detection (if implementing):**
- Add `version` field to worklog (incremented on each update)
- If updating with old version, throw conflict error
- Show UI: "This entry was modified. Reload to see latest?"

**Anti-patterns to avoid:**
- Don't use optimistic locking that's too aggressive (degrades UX)
- Don't silently lose updates; alert user to conflict
- Don't assume updates from different fields won't conflict
- Convex transactions handle this, so trust the database

### Example 6: Mobile Responsiveness & Progressive Enhancement

**Pattern: Mobile-first Tailwind + shadcn responsive components**

Layout structure:
```
Mobile (< 768px):
- Full-width header with hamburger menu
- Collapsed sidebar (hamburger opens drawer)
- Single-column main content
- Touch-friendly button sizes (min 44px tap target)

Tablet (768px - 1024px):
- Header with visible nav items
- Sidebar partially visible or collapsible
- Two-column layouts possible

Desktop (> 1024px):
- Full sidebar + main content area
- Tables with all columns visible
- Modals and popovers
```

**Component patterns:**
- Use Tailwind responsive classes: `hidden md:block`, `w-full md:w-1/2`
- shadcn Table component: automatically scrollable on mobile
- shadcn Dialog: full-screen on mobile, centered modal on desktop
- Pagination: "Prev/Next" buttons on mobile, numeric pagination on desktop
- Filter controls: Drawer on mobile, sidebar on desktop

**Touch-friendly interactions:**
- Button size: At least 44x44px for touch targets
- Spacing: Adequate padding around interactive elements
- Forms: Large input fields, easy-to-tap checkboxes
- Modals: Full-screen on mobile, easy to close (swipe or X button)

### Example 7: Error Scenarios & Recovery Strategies

**Scenario: Convex database outage**
- Symptom: All Convex queries timeout or 500 error
- Detection: Catch error in mutation/query
- Recovery: Show banner "Service temporarily unavailable. Please try again in a moment."
- User experience: Disable form inputs, show loading state, auto-retry after 5 seconds
- Fallback: If outage persists >1min, show "Contact support" link

**Scenario: Clerk auth service unavailable**
- Symptom: Can't sign in, session validation fails
- Detection: Clerk SDK throws error
- Recovery: Show "Authentication service temporarily unavailable"
- User experience: If already logged in, continue (auth token valid); prevent new logins

**Scenario: Invalid user data (e.g., dailyMaxHours < dailyMinHours after admin edit)**
- Symptom: Query returns user with invalid configuration
- Detection: Frontend validation or explicit error flag from API
- Recovery: Show warning "Invalid configuration. Please contact admin."
- Temporary fix: Use safe defaults (e.g., assume 8-10 hours) until admin corrects

**Scenario: Date parsing errors (malformed dates in imported data)**
- Symptom: Migration script fails on certain records
- Detection: Catch JSON.parse or date validation error
- Recovery: Log error with record ID, skip record, continue processing
- Post-migration: Report invalid records to user with IDs for manual review

---

## TECHNICAL ARCHITECTURE: Detailed Design Specifications

### Frontend Architecture (React + TypeScript)

**Project Structure:**
```
src/
├── components/
│   ├── Auth/
│   │   ├── ProtectedRoute.tsx (role-based route guard)
│   │   ├── SignInPage.tsx (Clerk sign-in widget wrapper)
│   │   └── SignUpPage.tsx (Clerk sign-up widget wrapper)
│   ├── Worklog/
│   │   ├── WorklogForm.tsx (create/edit form)
│   │   ├── WorklogList.tsx (user worklog display)
│   │   ├── WorklogCard.tsx (individual worklog item)
│   │   └── TimeStatusBadge.tsx (Normal/UT/OT indicator)
│   ├── Admin/
│   │   ├── UsersList.tsx (admin user management)
│   │   ├── AllWorklogsList.tsx (admin worklog monitoring)
│   │   ├── UserSettingsModal.tsx (edit min/max hours)
│   │   └── UserWorklogDetail.tsx (view user's logs)
│   ├── Layout/
│   │   ├── Header.tsx (navigation header with user menu)
│   │   ├── Sidebar.tsx (navigation sidebar)
│   │   └── MainLayout.tsx (wrapper for all protected pages)
│   └── Common/
│       ├── LoadingSpinner.tsx
│       ├── ErrorBoundary.tsx
│       ├── EmptyState.tsx
│       └── Toast.tsx (notifications)
├── pages/
│   ├── Login.tsx
│   ├── Help.tsx
│   ├── Dashboard.tsx
│   ├── CreateWorklog.tsx
│   ├── MyWorklogs.tsx
│   ├── AdminUsers.tsx
│   ├── AdminWorklogs.tsx
│   └── Profile.tsx
├── hooks/
│   ├── useAuth.ts (Clerk context)
│   ├── useWorklogs.ts (Convex queries/mutations for user)
│   ├── useAdminAPI.ts (Convex queries/mutations for admin)
│   └── useLocalStorage.ts (persist filters/preferences)
├── lib/
│   ├── convex.ts (Convex client setup)
│   ├── clerk.ts (Clerk client setup)
│   ├── api.ts (API helper functions)
│   └── validation.ts (form validation schemas using Zod)
├── types/
│   ├── worklog.ts
│   ├── user.ts
│   └── api.ts
├── styles/
│   ├── globals.css (Tailwind imports)
│   └── theme.css (custom theme variables)
├── App.tsx (router and main layout)
└── main.tsx (entry point)
```

**Key Technologies:**
- React 18 with TypeScript (strict mode)
- React Router v6 for routing
- Convex React hooks (useQuery, useMutation) for data fetching
- Clerk React SDK for authentication
- Tailwind CSS for styling
- shadcn/ui for component library
- Zod for runtime validation (frontend)
- React Query or Convex built-in caching for state management

**Component Props & State Patterns:**

WorklogForm component:
- Props: { onSubmit: (data: CreateWorklogInput) => Promise<void>, defaultValues?: Worklog, isLoading?: boolean }
- State: Form fields (date, workedHours, taskId, description), errors, isSubmitting
- Behavior: Validates on blur, disables submit button while loading, shows field errors

WorklogList component:
- Props: None (fetches own data via Convex hook)
- State: worklogs[], filterBy, sortOrder, currentPage, loading, error
- Behavior: Fetches data on mount, groups by date, shows time status, handles pagination

UsersList (admin):
- Props: None
- State: users[], sortBy, currentPage, editingUserId, loading, error
- Behavior: Fetches all users, displays in table, allows inline edit of min/max hours

**Error Boundary:**
- Wrap main app and route pages with ErrorBoundary component
- Catches unhandled exceptions, displays fallback UI, logs to monitoring service

### Backend Architecture (Convex + TypeScript)

**Project Structure:**
```
convex/
├── schema.ts (database schema definition)
├── auth.ts (auth utilities, role checks)
├── seed.ts (development data seeding)
├── functions/
│   ├── auth/
│   │   ├── getCurrentUser.ts (query)
│   │   └── getCurrentUserRole.ts (query)
│   ├── worklogs/
│   │   ├── create.ts (mutation)
│   │   ├── update.ts (mutation)
│   │   ├── delete.ts (mutation)
│   │   ├── listUserWorklogs.ts (query)
│   │   └── listAllWorklogs.ts (query, admin only)
│   ├── users/
│   │   ├── listAllUsers.ts (query, admin only)
│   │   ├── updateUserSettings.ts (mutation, admin only)
│   │   ├── getUserProfile.ts (query)
│   │   └── seed.ts (internal function for seeding)
│   └── migrations/
│       └── importHistoricalWorklogs.ts (import function)
├── lib/
│   ├── validation.ts (shared validation utilities)
│   ├── types.ts (TypeScript interfaces mirrored from frontend)
│   └── pagination.ts (cursor pagination utilities)
└── package.json (Convex project config)
```

**Database Schema (Convex):**

Users table:
- `userId` (v.string()): Clerk user ID, indexed
- `email` (v.string()): Email, unique index
- `name` (v.string()): User name
- `dailyMinHours` (v.number()): Minimum hours requirement
- `dailyMaxHours` (v.number()): Maximum hours limit
- `role` (v.enum('user', 'admin')): User role
- `createdAt` (v.number()): Timestamp
- `updatedAt` (v.number()): Timestamp
- Indexes: userId (primary), email (unique), role

Worklogs table:
- `userId` (v.string()): Foreign key to Users, indexed
- `date` (v.string()): ISO date string (YYYY-MM-DD), indexed
- `workedHours` (v.number()): Hours worked
- `taskId` (v.string(), optional)
- `description` (v.string(), optional)
- `createdAt` (v.number()): Timestamp
- `updatedAt` (v.number()): Timestamp
- Indexes: userId, date, userId+date (composite for efficient grouping)

**Query Patterns:**

listUserWorklogs query:
- Input: { userId, limit, cursor, filterBy, sortOrder }
- Validation: Current user must match userId (unless admin)
- Execution:
  1. Fetch user settings (to get dailyMinHours, dailyMaxHours)
  2. Query worklogs where userId matches and within date bounds
  3. Paginate using cursor
  4. Group by date, compute dailyTotalHours, calculate timeStatus
  5. Filter by timeStatus if specified
  6. Return grouped results with cursor

listAllWorklogs query (admin):
- Input: { limit, cursor, filters: { userId, dateRange, timeStatus }, sort }
- Validation: User must have role='admin'
- Execution:
  1. Build query with filters applied
  2. Join with Users table to get min/max hours and names
  3. Group by date and userId
  4. Compute dailyTotalHours and timeStatus
  5. Filter by timeStatus if specified
  6. Sort by date desc, then userName asc
  7. Paginate and return

**Mutation Patterns:**

createWorklog mutation:
- Input: { date, workedHours, taskId, description }
- Validation:
  1. Auth check (user must be authenticated)
  2. Data validation (hours > 0, <= 24; date is valid ISO; description <= 500 chars)
  3. Business rule check (date not in future)
- Execution:
  1. Insert into Worklogs with userId from auth, createdAt/updatedAt timestamps
  2. Return created record with logId
- Error handling: Throw typed errors (ValidationError, AuthError, etc.)

updateUserSettings mutation:
- Input: { userId, dailyMinHours, dailyMaxHours }
- Validation:
  1. Auth check: user must be admin
  2. Data validation: min >= 0, max > min, both numbers
- Execution:
  1. Update Users record
  2. Return updated user settings
- Error handling: Throw error if user not found

**Error Handling Strategy:**
- Define typed errors: ValidationError, AuthError, NotFoundError
- Throw errors from mutations/queries with clear messages
- Frontend catches and handles with appropriate UI
- Log all errors (auth failures, validation failures, DB errors) for monitoring

### Deployment & Infrastructure

**Development Environment:**
- Convex local development (`npx convex dev` runs local backend)
- Frontend runs on localhost:5173 (Vite)
- Convex dashboard at localhost:3210
- Clerk test keys for local development

**Production Deployment:**
- Frontend: Deployed to Vercel or similar (static site)
- Backend: Convex cloud (serverless)
- Database: Convex cloud (managed D1/SQLite)
- Auth: Clerk cloud
- No separate API server needed (Convex handles all backend)

**Environment Variables:**
```
.env.local (frontend):
VITE_CONVEX_URL=http://localhost:5173 (dev) or https://... (prod)
VITE_CLERK_PUBLISHABLE_KEY=...
VITE_API_URL=http://localhost:5173/api (dev) or https://... (prod)

.env (Convex):
CLERK_JWT_ISSUER_DOMAIN=https://your-tenant.clerk.accounts.com
```

**Database Migrations:**
- Convex schema.ts defines all tables and indexes
- Changes to schema.ts trigger migration on next deployment
- Convex handles schema migrations automatically (no manual SQL)

---

## Validation Framework: Success Criteria & Testing Strategy

### Functional Testing Checklist

**Authentication & Authorization:**
- [ ] Anonymous user cannot access protected routes (redirects to login)
- [ ] User can sign up via Clerk sign-up form
- [ ] User can sign in via Clerk sign-in form
- [ ] User is logged out after clicking logout button
- [ ] Admin can access admin pages; non-admin cannot (shows 403)
- [ ] User can only see their own worklogs in list
- [ ] Admin can see all worklogs across all users
- [ ] Session persists after page refresh

**Worklog Creation & Management:**
- [ ] User can create worklog with date, hours, optional task/description
- [ ] Form validates required fields (shows error if missing)
- [ ] Form rejects invalid data (hours > 24, future date, non-numeric hours)
- [ ] Created worklog appears in user's worklog list
- [ ] User can edit their own worklog (hours, task, description)
- [ ] Edit updates are reflected in list without refresh
- [ ] User can delete their own worklog (shows confirmation)
- [ ] Deleted worklog is removed from list
- [ ] User cannot edit/delete other users' worklogs

**Time Status Calculation:**
- [ ] Daily total hours calculated correctly (sum of all logs for the day)
- [ ] Time status badge shows "Normal" when between min/max
- [ ] Time status badge shows "UT" when below minimum
- [ ] Time status badge shows "OT" when above maximum
- [ ] Time status updates when worklog is added/edited/deleted

**Worklog List & Filtering:**
- [ ] Worklogs displayed grouped by date, most recent first
- [ ] Filter "All logs" shows all worklogs
- [ ] Filter "Undertime" shows only days with undertime
- [ ] Filter "Overtime" shows only days with overtime
- [ ] Pagination shows 20 entries per page
- [ ] "Load More" button appears when more data available
- [ ] Clicking load more appends next page to list
- [ ] Filter state persists in URL query param
- [ ] Reset filters button clears all filters and resets to page 1

**Admin Features:**
- [ ] Admin can view list of all users
- [ ] Admin can edit user min/max hours
- [ ] Admin edits update immediately in user list
- [ ] Admin can view all worklogs across all users
- [ ] Admin can filter by user, date range, or time status
- [ ] Admin can click on user to view detailed worklog history
- [ ] Admin can edit user hours from worklog detail view

**Data Seeding:**
- [ ] Dev database has 3 users (2 regular, 1 admin)
- [ ] Seed data includes users with different min/max hours
- [ ] Seed data includes worklogs with normal, undertime, and overtime entries
- [ ] Migration script successfully imports 10-year historical data
- [ ] Migrated data is queryable and displays correctly

**Error Handling:**
- [ ] Network error shows toast with retry option
- [ ] Invalid input shows field-level error messages
- [ ] Auth error (401) redirects to login
- [ ] Permission error (403) shows 403 page
- [ ] Not found error (404) shows not found page
- [ ] Concurrent edit shows conflict warning
- [ ] Form validation prevents invalid submission

**Performance:**
- [ ] Worklog list loads in < 2 seconds (20 entries)
- [ ] Filter/pagination changes load in < 1 second
- [ ] Create worklog completes in < 1 second
- [ ] Admin all-worklogs list with 10K users loads pages in < 2 seconds
- [ ] No full-page reloads when switching between routes

**Mobile Responsiveness:**
- [ ] Layout is single-column on mobile (< 768px)
- [ ] Navigation menu collapses to hamburger on mobile
- [ ] Form inputs and buttons are touch-friendly (44px+ tap targets)
- [ ] Tables scroll horizontally on mobile without breaking layout
- [ ] Pagination controls are mobile-friendly
- [ ] All features accessible on mobile (no desktop-only features)

### Integration Testing Checklist

**Frontend-Backend Integration:**
- [ ] Convex mutations successfully create/update/delete worklogs
- [ ] Convex queries fetch correct data with filters applied
- [ ] Pagination cursor correctly fetches next page
- [ ] User authentication state syncs between Clerk and Convex
- [ ] User role verified by Convex on protected mutations

**API Data Contracts:**
- [ ] API responses match expected schema (type checking passes)
- [ ] API responses include all required fields
- [ ] Timestamps are correctly formatted and timezone-aware
- [ ] Computed fields (dailyTotalHours, timeStatus) calculated correctly
- [ ] Pagination metadata (hasMore, cursor) accurate

**Database Integrity:**
- [ ] Foreign key relationships enforced (userId references exist)
- [ ] Unique constraints enforced (email unique per user)
- [ ] Indexes created and improving query performance
- [ ] Data is correctly isolated per user (no data leakage)
- [ ] Concurrent updates don't cause data corruption

### Performance Testing Checklist

**Load Testing:**
- [ ] System handles 100 concurrent users (simulate with load test)
- [ ] Single worklog creation takes < 500ms at p99
- [ ] Worklog list query with pagination takes < 1000ms at p99
- [ ] Admin all-worklogs query (1M records) takes < 2000ms at p99

**Database Performance:**
- [ ] Query for user's worklogs (indexed on userId, date) < 500ms
- [ ] Query for all worklogs grouped by date/user (composite index) < 1000ms
- [ ] Update user settings completes < 300ms
- [ ] Concurrent mutations to same user's data complete without contention

**Frontend Performance:**
- [ ] Initial load (bundle size) < 500KB gzipped
- [ ] Time to interactive < 3 seconds on 4G
- [ ] Switching between pages < 500ms
- [ ] Rendering 20-entry worklog list < 100ms

### Security Testing Checklist

**Authentication:**
- [ ] Invalid JWT token rejected
- [ ] Expired session triggers re-authentication
- [ ] User cannot forge JWT token
- [ ] Session tokens not exposed in URLs (use headers)

**Authorization:**
- [ ] User cannot access other user's worklogs (API returns 403)
- [ ] Admin endpoints reject non-admin users (API returns 403)
- [ ] User cannot modify user role or min/max hours
- [ ] API calls validate permission before data access

**Input Validation:**
- [ ] SQL injection attempts rejected (Convex prevents via type safety)
- [ ] XSS in worklog description sanitized or escaped
- [ ] File upload (if added) validates file type and size
- [ ] Form inputs validated on client and server

**Data Protection:**
- [ ] Sensitive data (passwords) never logged or stored
- [ ] HTTPS enforced for all API calls
- [ ] Convex API endpoints protected by auth
- [ ] Historical worklog data migrated securely (no data loss or corruption)

---

## Operational Guidelines: Deployment, Monitoring, & Maintenance

### Pre-Launch Checklist

**Code Quality:**
- [ ] All TypeScript strict mode checks pass (no `any` types)
- [ ] ESLint rules configured and no warnings
- [ ] Tests pass (unit + integration)
- [ ] Code reviewed by team member
- [ ] No console.log or debug code in production

**Security:**
- [ ] Clerk production keys configured
- [ ] Environment variables set (no secrets in code)
- [ ] HTTPS enabled for all endpoints
- [ ] Rate limiting configured on API endpoints
- [ ] Input validation in place for all user inputs

**Performance:**
- [ ] Bundle size acceptable (< 500KB gzipped)
- [ ] Database indexes created
- [ ] Pagination implemented for large datasets
- [ ] Images optimized, lazy-loaded where applicable
- [ ] Load testing completed, results acceptable

**Data:**
- [ ] Convex production database initialized
- [ ] Seeding script tested and runs successfully
- [ ] Historical data migration script tested on sample data
- [ ] Backup strategy defined (Convex handles)
- [ ] Data retention policy documented

**Operations:**
- [ ] Monitoring/alerting configured (error tracking, performance metrics)
- [ ] Logging configured (structured logs, searchable)
- [ ] Deployment procedure documented
- [ ] Rollback procedure documented
- [ ] Runbook created for common operational tasks

### Deployment Procedure

**Frontend Deployment (Vercel):**
1. Ensure all tests pass locally
2. Create pull request with changes
3. PR review and approval
4. Merge to main branch
5. Vercel automatically builds and deploys to production
6. Verify deployment: Visit app URL, test core flows
7. Monitor error tracking dashboard for 1 hour

**Backend Deployment (Convex):**
1. Push code changes to Convex directory
2. Run `npx convex deploy` from project root
3. Confirm schema changes (if any)
4. Convex automatically deploys to cloud
5. Verify API endpoints responding
6. Monitor Convex dashboard for errors

**Data Migration (if needed):**
1. Backup production database (Convex automatic backups)
2. Run migration script on staging environment first
3. Verify data integrity (record counts, sample checks)
4. Schedule migration during off-hours (if possible)
5. Run migration script on production
6. Verify results (spot-check records, total count)
7. Monitor application for data-related errors

**Rollback Procedure:**
- Frontend: Revert commit, push to main, Vercel redeploys previous version (< 5 min)
- Backend: Convex maintains version history; revert schema/functions via CLI
- Database: Convex backups available; contact support for restore if needed

### Monitoring & Alerting

**Metrics to Track:**
- Error rate (% of requests resulting in 5xx errors)
- API response times (p50, p95, p99 latencies)
- Database query times (especially group/aggregate queries)
- Authentication failures (401/403 errors)
- Create worklog success rate
- List worklog pagination cursor validity

**Alerts (trigger when):**
- Error rate > 1% for 5 minutes
- API response time p99 > 5 seconds
- Database unavailable (all queries timeout)
- Authentication service unavailable
- Convex function execution timeout rate > 5%

**Monitoring Tools:**
- Sentry or Rollbar: Error tracking
- Vercel Analytics: Frontend performance
- Convex dashboard: Backend performance, function logs
- Clerk dashboard: Authentication metrics

**Logging Strategy:**
- Structured JSON logs with timestamp, level (debug/info/warn/error), context, message
- Log authentication attempts (especially failures)
- Log data mutations (create/update/delete with userId, timestamp)
- Log API errors with request/response details
- Don't log sensitive data (passwords, full emails in some contexts)

### Maintenance Tasks

**Weekly:**
- Review error logs, investigate and fix new errors
- Monitor alert dashboard for threshold violations
- Check database size/performance metrics

**Monthly:**
- Review performance metrics for trends
- Validate authentication/authorization still functioning correctly
- Backup data (Convex handles automatically, but verify)
- Update dependencies if security patches available

**Quarterly:**
- Load testing with updated data volume
- Disaster recovery drill (restore from backup)
- Security audit (review access logs, permissions)
- Performance optimization review (slow queries, unused indexes)

**Yearly:**
- Archive old worklog data (if retention policy requires)
- Upgrade major dependencies
- Security penetration testing (hire external firm)
- Capacity planning review (growth projections, scaling strategy)

### Troubleshooting Guide

**Issue: Users cannot log in**
- Check Clerk dashboard for service status
- Verify Clerk API keys are correct
- Check browser console for auth errors
- Try incognito mode (rules out cookie issues)
- Clear browser cache and try again

**Issue: Worklogs not saving**
- Check browser network tab for failed requests
- Verify Convex backend is running and responding
- Check Convex dashboard for mutation errors
- Verify user has permission to create worklog
- Check form validation (no validation errors?)

**Issue: Slow performance on worklog list**
- Check database indexes are created (Convex dashboard)
- Review query for N+1 problems (fetching user data per worklog?)
- Check if filtering is happening in frontend (should be in database)
- Verify pagination is working (fetching too many records?)
- Monitor database CPU/memory usage

**Issue: Concurrent edit conflict**
- Expected behavior if same worklog edited from multiple tabs
- Show user warning to refresh
- If happening frequently, implement pessimistic locking or conflict resolution UI

**Issue: Data migration import fails**
- Check CSV/JSON file format matches expected schema
- Verify data types (hours are numbers, dates are valid ISO format)
- Check for duplicate user IDs (should match existing users)
- Log which record caused failure, skip and continue
- After import, manually review failed records and retry

**Issue: Admin cannot see all worklogs (403 error)**
- Verify user role is 'admin' in database
- Check Convex function checks for role='admin' before returning data
- Ensure auth token is valid and recent
- Try logging out and back in to refresh token

---

## Data Dictionary

**User Document:**
```
{
  _id: ObjectId (internal Convex ID)
  userId: string (Clerk user ID)
  email: string (unique)
  name: string
  dailyMinHours: number (e.g., 7)
  dailyMaxHours: number (e.g., 9)
  role: 'user' | 'admin'
  createdAt: number (milliseconds)
  updatedAt: number (milliseconds)
}
```

**Worklog Document:**
```
{
  _id: ObjectId (internal Convex ID)
  userId: string (foreign key to User.userId)
  date: string (ISO 8601, e.g., "2025-11-06")
  workedHours: number (e.g., 8.5)
  taskId: string | null (optional reference)
  description: string | null (optional, max 500 chars)
  createdAt: number (milliseconds)
  updatedAt: number (milliseconds)
}
```

**API Response Types:**

CreateWorklogResponse:
```
{
  logId: string
  userId: string
  date: string
  workedHours: number
  taskId?: string
  description?: string
  createdAt: number
  updatedAt: number
}
```

WorklogListResponse:
```
{
  worklogs: Array<{
    date: string
    userId: string
    userName: string (for admin list)
    dailyTotalHours: number
    timeStatus: 'normal' | 'undertime' | 'overtime'
    logCount: number
    logsForDay: Array<{
      logId: string
      workedHours: number
      taskId?: string
      description?: string
      createdAt: number
      updatedAt: number
    }>
  }>
  cursor: string | null
  hasMore: boolean
}
```

---

## Constraints & Limitations

### Technical Constraints

**Convex Limitations:**
- Request execution time: 5-minute maximum (should not be exceeded for normal worklog operations)
- Request size: 10MB payload limit
- Database storage: Depends on plan (development plan sufficient for pilot)
- Database query complexity: Avoid complex joins; use simple indexed queries

**Clerk Limitations:**
- Session timeout: Configurable (default 24 hours)
- Rate limiting: Clerk handles; users can't auth-spam

**Frontend Constraints:**
- Browser storage: localStorage/sessionStorage ~5MB limit (not needed; use Convex)
- Browser execution time: No hard limit, but UI should be responsive (< 100ms for interactions)

### Scalability Considerations

**Current Scale (10K users):**
- Estimated worklog records: ~3.65M (10K users × 365 days × ~1 log per day on average)
- Database queries should complete in < 1 second with proper indexes
- Convex pricing scales with reads/writes (fine for current load)

**Future Scaling (100K users):**
- Worklog records: ~36.5M
- Need to optimize queries (aggregation may become slow)
- Consider archiving old data (> 2 years) to separate cold storage
- Implement caching layer if read load increases

**Cost Optimization:**
- Use Convex query caching when available
- Batch mutations where possible (reduce request count)
- Archive historical data to reduce live database size
- Monitor Convex usage dashboard; adjust plan if needed

---

## Key Technical Decisions & Justifications

**Why Convex (serverless database) instead of traditional backend?**
- Zero infrastructure management
- Built-in real-time subscriptions (if needed for live updates)
- Integrated auth support (works directly with Clerk)
- Automatic scaling without DevOps
- Lower operational overhead for small team

**Why Clerk for auth instead of custom implementation?**
- Battle-tested security (no risk of auth bugs)
- Multi-factor auth built-in (future-proofing)
- Social login ready (if needed)
- OAuth/SAML support for enterprise customers
- Managed session handling, no server-side session storage needed

**Why Tailwind + shadcn instead of Material UI or Bootstrap?**
- Utility-first approach matches Convex serverless mentality (composable, lightweight)
- shadcn components are unstyled/customizable
- Tailwind has excellent mobile-responsive utilities
- Smaller bundle size than Bootstrap
- Active community and good documentation

**Why cursor-based pagination instead of offset?**
- Offset-based pagination is O(n) database cost (scan through offset records)
- Cursor-based is O(1) (jump directly to cursor position)
- Cursor-based handles concurrent data updates better
- Cursor-based naturally prevents issues with duplicate/missing records when data changes mid-pagination

---

## Success Metrics & Business KPIs

**User Adoption:**
- Target: 50% of employees log work within first week
- Tracking: DAU (daily active users) / total users

**Data Quality:**
- Target: 95%+ of work days have at least one worklog entry (for active users)
- Tracking: Days with entries / total active user-days

**System Reliability:**
- Target: 99.5% uptime (Convex SLA)
- Tracking: API error rate < 0.5%, page load time p99 < 3 seconds

**User Satisfaction:**
- Target: 4+ stars in user feedback (if surveyed)
- Tracking: User support tickets related to missing features

---

## References & Documentation

**Official Documentation:**
- Convex: https://docs.convex.dev (queries, mutations, schema, auth)
- Clerk: https://clerk.com/docs (authentication, session management)
- React Router: https://reactrouter.com/docs (routing, navigation)
- Tailwind CSS: https://tailwindcss.com/docs (styling utilities)
- shadcn/ui: https://ui.shadcn.com (component library)
- Zod: https://zod.dev (schema validation)

**Key Sections to Review:**
- Convex: Schema, queries/mutations, database schema, indexes
- Clerk: OAuth flows, JWT verification, user sessions
- React Router: Protected routes, navigation guards
- Tailwind: Responsive design, component patterns
- shadcn: Theming, component customization

**Related Standards:**
- WCAG 2.1 AA: Accessibility guidelines for web content
- ISO 8601: Date/time format (YYYY-MM-DD)
- REST API best practices: Pagination, filtering, error codes

---

## Appendix: Checklist for Implementation Hand-Off

**Before Implementation Starts:**
- [ ] Team has read and understood entire PRP
- [ ] Technical stack (Node.js, Bun, React, Convex, Clerk) approved
- [ ] Database schema finalized and documented
- [ ] API endpoints and data contracts agreed upon
- [ ] UI mockups/wireframes created for key pages
- [ ] Testing strategy reviewed and test cases identified
- [ ] Deployment and monitoring setup plan finalized

**During Implementation:**
- [ ] Daily standups to track progress and blockers
- [ ] Code reviews for all PRs (focusing on requirements compliance)
- [ ] Functional testing performed on feature completion
- [ ] Performance testing (response times, database queries)
- [ ] Security testing (auth, permissions, input validation)

**Before Launch:**
- [ ] All tests passing (unit, integration, e2e)
- [ ] Manual QA sign-off on all requirements
- [ ] Load testing completed, results acceptable
- [ ] Security audit completed, vulnerabilities resolved
- [ ] Production environment configured and verified
- [ ] Data migration dry-run successful
- [ ] Monitoring and alerting configured
- [ ] Team trained on operational procedures
- [ ] Documentation complete and accessible

**Post-Launch:**
- [ ] Monitor error rates and performance metrics
- [ ] Respond quickly to critical issues
- [ ] Gather user feedback and plan enhancements
- [ ] Schedule post-launch retrospective (week after launch)

---

**Document Version:** 1.0
**Last Updated:** November 6, 2025
**Status:** Ready for Implementation
