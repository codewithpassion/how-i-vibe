# Phase 4: Advanced Admin Features - Tasks

**Status**: 🔴 Not Started | 🟡 In Progress | 🟢 Complete
**Progress**: 0/13 tasks complete (0%)

---

## Backend Infrastructure Tasks

### 1. [ ] Create Audit Logging System

**Objective**: Implement audit trail for tracking all admin actions

**Layer**: Backend (Database + Functions)

**Subtasks**:
- [ ] Create AuditLog table schema in Convex
- [ ] Fields: adminId, action, targetUserId, changeType, oldValue, newValue, timestamp
- [ ] Create mutation to log admin actions
- [ ] Log user creation/deletion/role changes
- [ ] Log worklog admin operations
- [ ] Log settings changes by admin
- [ ] Add indexes for efficient querying
- [ ] Create query for audit log retrieval

**Effort**: Medium (4h)
**Complexity**: Medium
**Assigned to**: Backend Developer
**Depends on**: Phase 3 (Admin foundation)
**Blocks**: Task 2, Task 3, Task 6

**Success Criteria**:
- [ ] AuditLog table exists
- [ ] Mutations create audit entries
- [ ] All admin actions logged
- [ ] Query retrieves logs with pagination
- [ ] Indexes created for performance
- [ ] Sensitive data not logged
- [ ] Timestamps accurate

**Technical Notes**:
- Log who made change, what changed, old/new values
- Index on adminId and timestamp
- Don't log passwords or tokens
- Use structured data format
- Consider GDPR for data retention

**Status**: [ ] Not Started [ ] In Progress [ ] Complete

---

### 2. [ ] Create Admin Worklog Query

**Objective**: Implement query for admins to view all user worklogs

**Layer**: Backend (Queries)

**Subtasks**:
- [ ] Create `admin.listAllWorklogs(filters?, limit?, cursor?)` query
- [ ] Add userId filter (search or select user)
- [ ] Add date range filter
- [ ] Add time status filter (all/undertime/overtime)
- [ ] Group results by date and user
- [ ] Include daily totals and status
- [ ] Implement cursor-based pagination
- [ ] Add authorization check (admin only)

**Effort**: Large (8h)
**Complexity**: High
**Assigned to**: Backend Developer
**Depends on**: Phase 3 (Time status), Task 1
**Blocks**: Task 4, Task 7

**Success Criteria**:
- [ ] Query returns all users' worklogs
- [ ] Filters work correctly
- [ ] Pagination handles large datasets
- [ ] Performance acceptable (< 2s p99)
- [ ] Non-admin cannot access
- [ ] Results grouped appropriately
- [ ] Daily totals accurate

**Technical Notes**:
- Use composite indexes (userId, date) for performance
- Return grouped structure for display
- Filter server-side for efficiency
- Include time status for each day
- Handle filtering combinations

**Status**: [ ] Not Started [ ] In Progress [ ] Complete

---

### 3. [ ] Create Admin User Management Mutations

**Objective**: Implement mutations for admin user operations

**Layer**: Backend (Mutations)

**Subtasks**:
- [ ] Create `admin.createUser(email, name, roles)` mutation
- [ ] Create `admin.deleteUser(userId)` mutation
- [ ] Create `admin.assignRole(userId, role)` mutation
- [ ] Add validation for email uniqueness
- [ ] Add validation for role values
- [ ] Handle cascading deletes (clean up user data)
- [ ] Log all operations to audit trail
- [ ] Add authorization checks

**Effort**: Medium (4h)
**Complexity**: Medium
**Assigned to**: Backend Developer
**Depends on**: Task 1 (Audit logging)
**Blocks**: Task 8

**Success Criteria**:
- [ ] Can create new user
- [ ] Email validated as unique
- [ ] Can delete user and clean up
- [ ] Can assign roles
- [ ] All operations logged to audit
- [ ] Non-admin cannot call mutations
- [ ] Error handling clear

**Technical Notes**:
- Email must be unique across system
- Delete should cascade to worklogs
- Log admin who made change
- Validate role against allowed values
- Consider soft deletes for data retention

**Status**: [ ] Not Started [ ] In Progress [ ] Complete

---

### 4. [ ] Create Admin Compliance & Stats Queries

**Objective**: Implement queries for compliance dashboard

**Layer**: Backend (Queries)

**Subtasks**:
- [ ] Create `admin.getComplianceStats()` query
- [ ] Calculate total undertime hours (company-wide)
- [ ] Calculate total overtime hours
- [ ] Get list of users with undertime
- [ ] Get list of users with overtime
- [ ] Calculate trends over time (weekly/monthly)
- [ ] Return stats structure for dashboard
- [ ] Add caching for performance

**Effort**: Medium (4h)
**Complexity**: Medium
**Assigned to**: Backend Developer
**Depends on**: Task 2, Task 1
**Blocks**: Task 9

**Success Criteria**:
- [ ] Query returns company-wide stats
- [ ] Undertime/overtime totals accurate
- [ ] User lists correct
- [ ] Trends calculated correctly
- [ ] Performance acceptable with caching
- [ ] Returns data suitable for charts
- [ ] Admin only access

**Technical Notes**:
- Aggregate across all users
- Calculate weekly and monthly trends
- Consider caching results
- Return structure suitable for charts
- Include time periods in results

**Status**: [ ] Not Started [ ] In Progress [ ] Complete

---

## Frontend Admin Dashboard Tasks

### 5. [ ] Build Admin Worklogs View with Filters

**Objective**: Create page for admins to view all worklogs with filtering

**Layer**: Frontend (Pages)

**Subtasks**:
- [ ] Create `/app/routes/_auth.admin.worklogs._index.tsx`
- [ ] Build advanced filter controls (user, date range, status)
- [ ] Implement user search/autocomplete
- [ ] Add date range picker
- [ ] Add time status checkboxes
- [ ] Display filtered worklogs in table
- [ ] Show daily totals and status badges
- [ ] Implement pagination

**Effort**: Large (8h)
**Complexity**: High
**Assigned to**: Frontend Developer
**Depends on**: Task 2, Phase 3 (Time status UI)
**Blocks**: Task 6

**Success Criteria**:
- [ ] Filter controls render correctly
- [ ] Filters apply and update list
- [ ] Table displays worklog data
- [ ] Pagination works with filters
- [ ] Search finds users
- [ ] Date range picker works
- [ ] Time status badges display
- [ ] Non-admin denied access

**Technical Notes**:
- Use React Query for data fetching
- Build reusable filter components
- Use ShadCN table component
- Add search debouncing
- Store filters in URL params

**Status**: [ ] Not Started [ ] In Progress [ ] Complete

---

### 6. [ ] Build User Detail & Worklog History

**Objective**: Create page to view individual user's complete worklog history

**Layer**: Frontend (Pages)

**Subtasks**:
- [ ] Create `/app/routes/_auth.admin.worklogs.user.$userId.tsx`
- [ ] Display user info and settings
- [ ] Show all user's worklogs with pagination
- [ ] Show time status badges
- [ ] Show daily totals
- [ ] Allow inline editing of user's hours
- [ ] Show last updated information
- [ ] Add breadcrumbs for navigation

**Effort**: Medium (4h)
**Complexity**: Medium
**Assigned to**: Frontend Developer
**Depends on**: Task 5
**Blocks**: None

**Success Criteria**:
- [ ] Page loads user details
- [ ] Worklogs display in list
- [ ] Pagination works
- [ ] Can edit hours inline
- [ ] Changes persist
- [ ] Time status badges show
- [ ] Navigation works

**Technical Notes**:
- Fetch user data and worklogs
- Allow editing daily min/max hours
- Show audit trail of changes
- Use modal or inline edit
- Confirm before saving changes

**Status**: [ ] Not Started [ ] In Progress [ ] Complete

---

### 7. [ ] Build Admin User Management Interface

**Objective**: Create interface for managing users (create, delete, role assignment)

**Layer**: Frontend (Pages)

**Subtasks**:
- [ ] Create `/app/routes/_auth.admin.users._index.tsx` (enhanced from Phase 1)
- [ ] Add "Create User" button
- [ ] Build create user modal/form
- [ ] Add role assignment interface
- [ ] Add delete button with confirmation
- [ ] Show creation date and last activity
- [ ] Add bulk actions (if time permits)
- [ ] Show audit info (created by whom, when)

**Effort**: Medium (4h)
**Complexity**: Medium
**Assigned to**: Frontend Developer
**Depends on**: Phase 1 Task 10, Task 3
**Blocks**: Task 8

**Success Criteria**:
- [ ] Can create new user
- [ ] Can assign/change roles
- [ ] Can delete user
- [ ] Confirmation dialog on delete
- [ ] User list shows created/updated dates
- [ ] Non-admin cannot access
- [ ] Form validation works

**Technical Notes**:
- Use modal for create/edit
- Show role checkboxes
- Require confirmation for delete
- Display audit info
- Show success/error toasts

**Status**: [ ] Not Started [ ] In Progress [ ] Complete

---

### 8. [ ] Build Compliance Dashboard

**Objective**: Create dashboard showing company-wide time compliance metrics

**Layer**: Frontend (Pages)

**Subtasks**:
- [ ] Create `/app/routes/_auth.admin.compliance._index.tsx`
- [ ] Display company-wide stats (total undertime/overtime)
- [ ] Build chart showing time status distribution
- [ ] List top undertime users
- [ ] List top overtime users
- [ ] Show trends over time (weekly chart)
- [ ] Add date range selector
- [ ] Style with Tailwind and chart library

**Effort**: Medium (4h)
**Complexity**: Medium
**Assigned to**: Frontend Developer
**Depends on**: Task 4
**Blocks**: None

**Success Criteria**:
- [ ] Dashboard loads stats
- [ ] Charts display correctly
- [ ] Stats are accurate
- [ ] User lists show
- [ ] Date range filters work
- [ ] Responsive on mobile
- [ ] Colors match design

**Technical Notes**:
- Use chart library (Recharts, Chart.js, etc.)
- Show pie chart for status distribution
- Line chart for trends
- Bar chart for top users
- Color code: green/orange/red
- Format large numbers nicely

**Status**: [ ] Not Started [ ] In Progress [ ] Complete

---

### 9. [ ] Build Audit Log Viewer

**Objective**: Create page for viewing admin activity and audit trail

**Layer**: Frontend (Pages)

**Subtasks**:
- [ ] Create `/app/routes/_auth.admin.audit-logs.tsx`
- [ ] Fetch audit logs from backend
- [ ] Display logs in table format
- [ ] Show admin, action, target user, timestamp
- [ ] Show before/after values for changes
- [ ] Add filters (admin, action type, date range)
- [ ] Implement pagination
- [ ] Make responsive

**Effort**: Small (2h)
**Complexity**: Low
**Assigned to**: Frontend Developer
**Depends on**: Task 1
**Blocks**: None

**Success Criteria**:
- [ ] Audit logs display
- [ ] All columns show correct data
- [ ] Pagination works
- [ ] Filters narrow results
- [ ] Timestamps readable
- [ ] Changes show before/after
- [ ] Table responsive

**Technical Notes**:
- Format timestamps nicely
- Show change details expanded
- Use table with search
- Sortable columns
- Good performance with pagination

**Status**: [ ] Not Started [ ] In Progress [ ] Complete

---

## Integration & Testing Tasks

### 10. [ ] Create Admin Navigation & Route Guards

**Objective**: Add admin menu items to navigation and implement route protection

**Layer**: Frontend (Integration)

**Subtasks**:
- [ ] Add "Admin" section to navigation menu (admin only)
- [ ] Add links to all admin pages
- [ ] Create admin layout with sub-navigation
- [ ] Protect all admin routes with role check
- [ ] Redirect non-admin away from admin pages
- [ ] Add breadcrumbs for admin pages
- [ ] Test permission enforcement
- [ ] Ensure mobile nav includes admin links

**Effort**: Small (2h)
**Complexity**: Low
**Assigned to**: Frontend Developer
**Depends on**: Task 5, Task 7, Task 8, Task 9
**Blocks**: Task 11

**Success Criteria**:
- [ ] Admin menu visible only to admins
- [ ] Admin links work
- [ ] Non-admin cannot access admin pages
- [ ] Breadcrumbs correct
- [ ] Mobile nav includes admin
- [ ] TypeScript passes

**Technical Notes**:
- Conditional rendering based on isAdmin()
- Route guards check role
- Sub-navigation for admin area
- Mobile hamburger includes admin
- Clear visual indication of admin section

**Status**: [ ] Not Started [ ] In Progress [ ] Complete

---

### 11. [ ] End-to-End Admin Testing

**Objective**: Test all admin features end-to-end

**Layer**: Frontend + Backend (Testing/Integration)

**Subtasks**:
- [ ] Test create user flow (form → mutation → list update)
- [ ] Test delete user (confirmation → delete → audit log)
- [ ] Test role assignment (change role → verify in list)
- [ ] Test worklog filtering (apply filters → results update)
- [ ] Test compliance dashboard (load → verify stats)
- [ ] Test audit log viewing (actions appear in log)
- [ ] Test error scenarios (invalid data, permission denied)
- [ ] Verify non-admin cannot access admin features

**Effort**: Medium (4h)
**Complexity**: Medium
**Assigned to**: QA/Frontend Developer
**Depends on**: Task 5, Task 7, Task 8, Task 9, Task 10
**Blocks**: None

**Success Criteria**:
- [ ] All admin workflows work end-to-end
- [ ] Create/delete/edit operations work
- [ ] Filters and search work
- [ ] Audit trail accurate
- [ ] Errors handled gracefully
- [ ] Non-admin properly denied access
- [ ] Performance acceptable
- [ ] TypeScript strict mode passes

**Technical Notes**:
- Test with admin and non-admin users
- Test with various data volumes
- Test edge cases (empty results, etc.)
- Check mobile responsiveness
- Verify error messages helpful

**Status**: [ ] Not Started [ ] In Progress [ ] Complete

---

### 12. [ ] Implement Bulk Actions (Optional)

**Objective**: Add bulk operations for efficiency (if time permits)

**Layer**: Frontend + Backend

**Subtasks**:
- [ ] Add checkboxes to user list
- [ ] Implement "bulk delete users" action
- [ ] Implement "bulk role assignment"
- [ ] Add confirmation dialog for bulk ops
- [ ] Log bulk operations to audit trail
- [ ] Show progress indicator
- [ ] Handle partial failures gracefully
- [ ] Show summary of completed actions

**Effort**: Medium (4h)
**Complexity**: Medium
**Assigned to**: Frontend Developer
**Depends on**: Task 7
**Blocks**: None

**Success Criteria**:
- [ ] Can select multiple users
- [ ] Bulk actions execute
- [ ] Operations logged
- [ ] Errors handled
- [ ] Success message shown

**Technical Notes**:
- Batch operations in backend
- Log each action individually
- Handle network errors
- Show rollback option if needed
- Avoid timeout on large batches

**Status**: [ ] Not Started [ ] In Progress [ ] Complete

---

### 13. [ ] Admin Feature Documentation

**Objective**: Create admin documentation and help content

**Layer**: Documentation

**Subtasks**:
- [ ] Document admin dashboard features
- [ ] Create user management guide
- [ ] Document worklog monitoring interface
- [ ] Create compliance dashboard explanation
- [ ] Document audit log system
- [ ] Create troubleshooting guide
- [ ] Add screenshots to documentation
- [ ] Create quick-start guide for new admins

**Effort**: Small (2h)
**Complexity**: Low
**Assigned to**: Technical Writer/Developer
**Depends on**: All admin tasks
**Blocks**: None

**Success Criteria**:
- [ ] All admin features documented
- [ ] Screenshots included
- [ ] Steps are clear and complete
- [ ] Troubleshooting covers common issues
- [ ] Documentation is accessible

**Technical Notes**:
- Create in docs/ directory
- Include screenshots
- Step-by-step guides
- FAQ section
- Contact support info

**Status**: [ ] Not Started [ ] In Progress [ ] Complete

---

## Task Dependency Graph

**Critical Path (must complete in order):**
```
Task 1 (Audit Logging)
    ↓
Task 2 (Admin Worklog Query)
    ↓
Task 5 (Admin Worklogs View)
    ↓
Task 3 (Admin User Mutations)
    ↓
Task 7 (User Management Interface)
    ↓
Task 10 (Admin Navigation)

Task 4 (Compliance Stats)
    ↓
Task 8 (Compliance Dashboard)

Task 9 (Audit Log Viewer)

Task 11 (E2E Testing)
    ↓
Task 13 (Documentation)
```

**Parallel Work:**
```
- Task 4 (Stats) and Task 2 (Queries) can develop in parallel
- Task 6 (User Detail) and Task 5 can develop in parallel
- Task 8 (Compliance) and Task 5 (Worklogs) can develop in parallel
- Task 9 (Audit Viewer) and Task 8 can develop in parallel
```

## Effort Summary

| Category | Tasks | Percentage |
|----------|-------|-----------|
| Backend | 1, 2, 3, 4 | 40% |
| Frontend | 5, 6, 7, 8, 9, 10, 11, 12 | 55% |
| Other | 13 | 5% |
| **Total** | **13** | **100%** |

**Estimated Total Effort**: ~40 hours (Large phase)

---

## Tracking Progress

Mark tasks complete as you finish by changing `[ ]` to `[x]`:

```markdown
### 1. [x] Create Audit Logging System ✅ Completed
```

Update progress at top: `**Progress**: 1/13 tasks complete (8%)`

---

## Notes

- Phase 4 builds significantly on Phase 3 (time status foundation)
- Audit logging is critical for compliance
- Admin worklog query is most complex backend task
- Performance optimization important for large datasets
- Consider caching compliance stats (they don't change frequently)
- Bulk operations are nice-to-have, can be deferred
- Admin features have high security requirements
- Extensive testing important for admin features
