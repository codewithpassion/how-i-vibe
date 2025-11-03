# Phase 4: Admin Dashboard & Historical Data at Scale

**Status**: 🔴 Not Started
**Progress**: 0/16 tasks complete (0%)

---

## Infrastructure Tasks

### 1. [ ] Database Indexes for Admin Queries

**Objective**: Add indexes optimized for multi-user admin queries with date and user filtering.

**Layer**: Database

**Subtasks**:
- [ ] Create compound index on (date, userId) for chronological admin queries
- [ ] Create index on users.role for admin privilege checks
- [ ] Verify all indexes with Convex dashboard
- [ ] Measure query performance improvement
- [ ] Document admin query index strategy
- [ ] Test index effectiveness with multi-user queries

**Effort**: Small (2h)
**Complexity**: Low
**Assigned to**: Backend Developer
**Depends on**: None (builds on Phase 3)
**Blocks**: Task 5, Task 7

**Success Criteria**:
- [ ] Compound index (date, userId) created
- [ ] Index on users.role created
- [ ] Convex query planner uses new indexes
- [ ] Admin query performance <1s for 30 days all users
- [ ] No negative impact on write performance
- [ ] Index usage documented

**Technical Notes**:
- (date, userId) supports sorting and filtering
- users.role index optimizes authorization checks
- Test with getAllWorklogs query
- Monitor index size in Convex dashboard
- Compare performance before/after indexing

**Status**:
- [ ] Not Started
- [ ] In Progress
- [ ] Complete

---

### 2. [ ] Historical Data Seed Script

**Objective**: Create comprehensive seed script to generate 10 years of realistic worklog data.

**Layer**: Infrastructure

**Subtasks**:
- [ ] Design realistic data distribution (70% normal, 15% OT, 15% UT)
- [ ] Implement date range generation (2015-2025)
- [ ] Generate 0-5 worklogs per user per day (weighted toward 2-3)
- [ ] Create varied hours, task IDs, and descriptions
- [ ] Implement weekday weighting (more Mon-Fri activity)
- [ ] Add progress logging during execution
- [ ] Make script idempotent and resumable
- [ ] Create npm script: `npm run seed:historical`

**Effort**: Large (8h)
**Complexity**: High
**Assigned to**: Backend Developer
**Depends on**: None (builds on Phase 1 schema)
**Blocks**: Task 12

**Success Criteria**:
- [ ] Generates 15,000-20,000 worklogs across 10 years
- [ ] Realistic distribution: 70% normal, 15% OT, 15% UT
- [ ] Weekday activity higher than weekends
- [ ] Varied workedHours (0.25 to 12 in realistic increments)
- [ ] Some entries have taskId/description, some don't
- [ ] Script completes in <5 minutes
- [ ] Idempotent (can re-run without duplicates)
- [ ] Progress logged to console

**Technical Notes**:
- Batch inserts (1000 records per batch)
- Use Math.random() with weights for distributions
- Weekday probability: 80%, Weekend: 20%
- Random taskId format: "PROJ-1234"
- Random descriptions from template list
- Check existing data to avoid duplicates
- Log: "Generated 5000/20000 worklogs (25%)"
- Handle Convex batch size limits

**Status**:
- [ ] Not Started
- [ ] In Progress
- [ ] Complete

---

## Feature: Role-Based Access Control

### 3. [ ] Role Assignment and Verification

**Objective**: Implement server-side role checking to distinguish admin from regular users.

**Layer**: API

**Subtasks**:
- [ ] Create utility function to get user role from context
- [ ] Implement admin role verification function
- [ ] Store role in Convex users table (already in schema)
- [ ] Sync role from Clerk metadata or configure manually
- [ ] Add role to user profile query response
- [ ] Create authorization error responses

**Effort**: Medium (4h)
**Complexity**: Medium
**Assigned to**: Backend Developer
**Depends on**: None (builds on Phase 1)
**Blocks**: Task 5, Task 6

**Success Criteria**:
- [ ] Function returns user role from database
- [ ] Admin role verified correctly
- [ ] Non-admin users identified correctly
- [ ] Role accessible in Convex context
- [ ] Clear error for unauthorized access
- [ ] Role synced from Clerk or manually set

**Technical Notes**:
- Query users table by clerkUserId for role
- Cache role lookup in request context if possible
- Return ConvexError for authorization failures
- Default role should be "user"
- Admin role set manually for now
- Consider Clerk metadata for role sync later

**Status**:
- [ ] Not Started
- [ ] In Progress
- [ ] Complete

---

### 4. [ ] Admin-Only Query Authorization

**Objective**: Create authorization middleware for admin-only Convex queries.

**Layer**: API

**Subtasks**:
- [ ] Create admin authorization wrapper function
- [ ] Verify user is authenticated
- [ ] Verify user has admin role
- [ ] Reject unauthorized requests with 403 error
- [ ] Log authorization failures for security monitoring
- [ ] Return clear error messages (no data leakage)

**Effort**: Medium (4h)
**Complexity**: Medium
**Assigned to**: Backend Developer
**Depends on**: Task 3
**Blocks**: Task 5

**Success Criteria**:
- [ ] Admin queries protected by authorization check
- [ ] Non-admin users receive 403 error
- [ ] Unauthenticated users receive 401 error
- [ ] Authorization failures logged
- [ ] Error messages don't leak sensitive data
- [ ] Reusable across multiple admin functions

**Technical Notes**:
- Create wrapper: requireAdmin(ctx)
- Throw ConvexError with appropriate status
- Log: userId, timestamp, attempted action
- Error message: "Unauthorized: Admin access required"
- Use at start of every admin query/mutation
- Document usage pattern for future admin features

**Status**:
- [ ] Not Started
- [ ] In Progress
- [ ] Complete

---

## Feature: Admin Dashboard

### 5. [ ] Admin Worklogs Query - Multi-User Support

**Objective**: Create getAllWorklogs query for admin users to view all users' worklogs with enhanced filtering.

**Layer**: API

**Subtasks**:
- [ ] Create Convex query: api.worklogs.getAllWorklogs
- [ ] Implement admin authorization check
- [ ] Support filters: status (OT/UT/normal/all), userId, dateRange
- [ ] Fetch and include user names in results
- [ ] Sort by date DESC, then userName ASC
- [ ] Implement cursor-based pagination
- [ ] Calculate OT/UT status per user's thresholds
- [ ] Return structured grouped data with user info

**Effort**: Large (8h)
**Complexity**: High
**Assigned to**: Backend Developer
**Depends on**: Task 1, Task 3, Task 4
**Blocks**: Task 9

**Success Criteria**:
- [ ] Query returns all users' worklogs for admins
- [ ] Non-admin access blocked with 403 error
- [ ] Status filter works (OT/UT/normal/all)
- [ ] User filter returns specific user's data
- [ ] Date range filter works correctly
- [ ] User names included in response
- [ ] Sorted by date DESC, userName ASC
- [ ] Cursor pagination functional
- [ ] OT/UT calculated per user's thresholds
- [ ] Query performance <1s for 30 days all users

**Technical Notes**:
- Use requireAdmin(ctx) for authorization
- Join/denormalize user name into results
- Apply filters in efficient order
- Calculate status using each user's min/max hours
- Support combined filters (user + status + dateRange)
- Use compound index (date, userId)
- Return structure: { days: [...], nextCursor, hasMore }
- Each day includes: date, userId, userName, totalHours, status, worklogs[]

**Status**:
- [ ] Not Started
- [ ] In Progress
- [ ] Complete

---

### 6. [ ] User Filter Implementation

**Objective**: Implement user-specific filtering in admin query.

**Layer**: API

**Subtasks**:
- [ ] Add userId parameter to getAllWorklogs
- [ ] Filter worklogs by specific userId when provided
- [ ] Combine with status and date range filters
- [ ] Validate userId exists in database
- [ ] Handle null/undefined userId (show all users)
- [ ] Return user name in filtered results

**Effort**: Small (2h)
**Complexity**: Low
**Assigned to**: Backend Developer
**Depends on**: Task 5
**Blocks**: Task 9

**Success Criteria**:
- [ ] userId parameter filters to specific user
- [ ] null/undefined userId shows all users
- [ ] Invalid userId returns clear error
- [ ] Combines with other filters correctly
- [ ] User name included in results
- [ ] Performance acceptable with user filter

**Technical Notes**:
- Optional parameter: userId?: string
- Apply filter early in query chain
- Validate userId exists before querying
- Combine with .filter() for other criteria
- Test with all filter combinations

**Status**:
- [ ] Not Started
- [ ] In Progress
- [ ] Complete

---

### 7. [ ] Date Range Filter Implementation

**Objective**: Implement date range filtering (start and end dates) in admin query.

**Layer**: API

**Subtasks**:
- [ ] Add startDate and endDate parameters to getAllWorklogs
- [ ] Filter worklogs within date range (inclusive)
- [ ] Handle partial ranges (only start or only end)
- [ ] Validate date format and ranges
- [ ] Combine with status and user filters
- [ ] Handle timezone considerations (use UTC)

**Effort**: Medium (4h)
**Complexity**: Medium
**Assigned to**: Backend Developer
**Depends on**: Task 5
**Blocks**: Task 9

**Success Criteria**:
- [ ] startDate filters worklogs >= startDate
- [ ] endDate filters worklogs <= endDate
- [ ] Both parameters work together correctly
- [ ] Null/undefined dates ignored (no filter)
- [ ] Invalid dates return clear error
- [ ] startDate > endDate returns clear error
- [ ] Timezone handled consistently (UTC)
- [ ] Performance acceptable with date filter

**Technical Notes**:
- Parameters: startDate?: number (Unix timestamp), endDate?: number
- Filter: date >= startDate && date <= endDate
- Validate: startDate <= endDate
- Store dates as Unix timestamps for easy comparison
- Document timezone expectations (UTC)
- Test with edge cases: same day, year boundaries

**Status**:
- [ ] Not Started
- [ ] In Progress
- [ ] Complete

---

### 8. [ ] Admin Navigation and Layout

**Objective**: Create admin-specific navigation menu with role-based visibility.

**Layer**: Frontend

**Subtasks**:
- [ ] Add admin menu items to navigation
- [ ] Show/hide admin menu based on user role
- [ ] Add admin badge/indicator in header
- [ ] Create separate admin dashboard route
- [ ] Style admin sections distinctly
- [ ] Ensure mobile responsive admin menu

**Effort**: Medium (4h)
**Complexity**: Low
**Assigned to**: Frontend Developer
**Depends on**: None (builds on Phase 1 navigation)
**Blocks**: Task 9, Task 11

**Success Criteria**:
- [ ] Admin menu items visible only to admins
- [ ] Admin badge shown in header for admins
- [ ] Admin dashboard accessible at /admin/dashboard
- [ ] Navigation highlights active admin page
- [ ] Consistent styling with main app
- [ ] Mobile menu includes admin items

**Technical Notes**:
- Get user role from Convex user profile query
- Conditionally render admin menu items
- Add "Admin" badge to header
- Use different route prefix: /admin/*
- Maintain header/navigation layout
- Test with admin and non-admin users

**Status**:
- [ ] Not Started
- [ ] In Progress
- [ ] Complete

---

### 9. [ ] Admin Dashboard Page

**Objective**: Build admin dashboard with all-user worklog viewing and enhanced filtering UI.

**Layer**: Frontend

**Subtasks**:
- [ ] Create AdminDashboard component
- [ ] Fetch data with api.worklogs.getAllWorklogs
- [ ] Display user names prominently in worklog entries
- [ ] Show OT/UT badges per user thresholds
- [ ] Integrate all filters (status, user, date range)
- [ ] Implement cursor pagination
- [ ] Sort display by date DESC, userName ASC
- [ ] Handle loading and error states
- [ ] Add empty state for no results
- [ ] Implement mobile responsive layout

**Effort**: Large (8h)
**Complexity**: High
**Assigned to**: Frontend Developer
**Depends on**: Task 5, Task 6, Task 7, Task 8
**Blocks**: None

**Success Criteria**:
- [ ] Admin dashboard shows all users' worklogs
- [ ] User names displayed clearly
- [ ] OT/UT badges use correct user thresholds
- [ ] All filters functional and responsive
- [ ] Pagination works smoothly
- [ ] Data sorted correctly
- [ ] Loading states during queries
- [ ] Error handling for unauthorized access
- [ ] Empty state for zero results
- [ ] Mobile responsive design

**Technical Notes**:
- Use Convex useQuery with getAllWorklogs
- Display user name with each day group
- Pass all filter parameters to query
- Reset cursor when filters change
- Show loading spinner during fetch
- Redirect non-admin users with error message
- Format dates consistently
- Use shadcn components for consistency

**Status**:
- [ ] Not Started
- [ ] In Progress
- [ ] Complete

---

### 10. [ ] Admin Filter Controls

**Objective**: Build comprehensive filter UI for admin dashboard (status, user, date range).

**Layer**: Frontend

**Subtasks**:
- [ ] Create AdminFilterControls component
- [ ] Add status filter buttons (All, OT, UT, Normal)
- [ ] Add user filter dropdown/autocomplete
- [ ] Add date range picker (start and end dates)
- [ ] Add "Clear All Filters" button
- [ ] Display active filter tags/badges
- [ ] Implement filter state management
- [ ] Style filters consistently

**Effort**: Large (8h)
**Complexity**: High
**Assigned to**: Frontend Developer
**Depends on**: None (builds on Phase 3 filters)
**Blocks**: Task 9

**Success Criteria**:
- [ ] Status filters match Phase 3 design
- [ ] User dropdown supports search/autocomplete
- [ ] Date range picker selects start and end dates
- [ ] Clear filters button resets all criteria
- [ ] Active filters shown as badges/tags
- [ ] Filters work together (combined filtering)
- [ ] Mobile responsive filter layout
- [ ] Accessible (keyboard navigation, ARIA)

**Technical Notes**:
- Reuse StatusFilter from Phase 3
- Use shadcn Select or Combobox for user filter
- Use shadcn DatePicker for date range
- Fetch user list for dropdown (separate query)
- Support search in user dropdown (for 10K users)
- Show count of active filters
- Position filters above dashboard
- Manage filter state in parent component

**Status**:
- [ ] Not Started
- [ ] In Progress
- [ ] Complete

---

### 11. [ ] Admin Stub Pages

**Objective**: Create stub pages for future admin features (User Management, Edit User Hours).

**Layer**: Frontend

**Subtasks**:
- [ ] Create "User Management" stub page
- [ ] Create "Edit User Hours" stub page
- [ ] Add routes for stub pages
- [ ] Add navigation menu items for stub pages
- [ ] Use StubPage component from Phase 3
- [ ] Add feature descriptions to stub pages

**Effort**: Small (2h)
**Complexity**: Low
**Assigned to**: Frontend Developer
**Depends on**: Task 8
**Blocks**: None

**Success Criteria**:
- [ ] User Management stub page accessible
- [ ] Edit User Hours stub page accessible
- [ ] Both pages in admin navigation menu
- [ ] Consistent layout with admin dashboard
- [ ] Clear "Coming Soon" messaging
- [ ] Feature descriptions informative

**Technical Notes**:
- Routes: /admin/users, /admin/hours
- Reuse StubPage component
- User Management description: "Manage user accounts, roles, and permissions."
- Edit Hours description: "Configure minimum and maximum daily hours for each employee."
- Maintain admin layout
- Protected by admin authorization

**Status**:
- [ ] Not Started
- [ ] In Progress
- [ ] Complete

---

## Testing & Performance

### 12. [ ] Historical Data Generation and Validation

**Objective**: Execute historical seed script and validate data quality and distribution.

**Layer**: Infrastructure

**Subtasks**:
- [ ] Run seed:historical script
- [ ] Verify 15K-20K worklogs generated
- [ ] Validate data distribution (70/15/15)
- [ ] Check weekday vs weekend distribution
- [ ] Verify date range spans 10 years
- [ ] Validate data integrity (no duplicates, valid fields)
- [ ] Document seed data characteristics

**Effort**: Small (2h)
**Complexity**: Low
**Assigned to**: Developer
**Depends on**: Task 2
**Blocks**: Task 13

**Success Criteria**:
- [ ] Script completes successfully in <5 minutes
- [ ] 15,000-20,000 worklogs created
- [ ] Distribution matches targets (±5%)
- [ ] Weekday activity higher than weekends
- [ ] Data spans 2015-2025
- [ ] No duplicate entries
- [ ] All fields valid (hours, dates, IDs)
- [ ] Seed characteristics documented

**Technical Notes**:
- Query Convex to count records
- Aggregate to verify distributions
- Check for duplicate (userId, date, timestamp)
- Sample random records for manual validation
- Document in seed script README
- Save distribution report

**Status**:
- [ ] Not Started
- [ ] In Progress
- [ ] Complete

---

### 13. [ ] Admin Query Performance Testing

**Objective**: Measure and validate admin query performance with full historical dataset.

**Layer**: Infrastructure

**Subtasks**:
- [ ] Test getAllWorklogs with no filters (all data)
- [ ] Test each filter type (status, user, date range)
- [ ] Test combined filters
- [ ] Measure pagination performance through deep pages
- [ ] Document p50, p95, p99 latencies
- [ ] Verify <1s target for 30 days all users
- [ ] Identify performance bottlenecks if any

**Effort**: Medium (4h)
**Complexity**: Medium
**Assigned to**: Developer
**Depends on**: Task 5, Task 12
**Blocks**: None

**Success Criteria**:
- [ ] All queries under 1s p95 latency
- [ ] Combined filters perform acceptably
- [ ] Pagination through 100+ pages functional
- [ ] No performance degradation with full dataset
- [ ] Bottlenecks identified and documented
- [ ] Performance baselines documented

**Technical Notes**:
- Use Convex dashboard for metrics
- Test with realistic filter combinations
- Measure client-side and server-side performance
- Test with 30-day, 90-day, 1-year ranges
- Monitor memory usage
- Compare to performance targets
- Document results for future optimization

**Status**:
- [ ] Not Started
- [ ] In Progress
- [ ] Complete

---

### 14. [ ] Authorization Testing

**Objective**: Comprehensive security testing of role-based access controls.

**Layer**: Testing

**Subtasks**:
- [ ] Test admin user can access admin dashboard
- [ ] Test non-admin user blocked from admin dashboard
- [ ] Test non-admin cannot call getAllWorklogs
- [ ] Test unauthorized API access attempts
- [ ] Test role verification in all admin functions
- [ ] Test error messages don't leak data
- [ ] Verify authorization logging works

**Effort**: Medium (4h)
**Complexity**: Medium
**Assigned to**: QA / Developer
**Depends on**: Task 4, Task 5
**Blocks**: None

**Success Criteria**:
- [ ] Admin users access all admin features
- [ ] Non-admin users blocked with 403 errors
- [ ] Direct API calls blocked for non-admins
- [ ] Authorization checked on every admin query
- [ ] Error messages secure (no data leakage)
- [ ] Authorization failures logged correctly
- [ ] All tests passing consistently

**Technical Notes**:
- Test with both admin and non-admin users
- Attempt direct Convex API calls
- Verify 403 vs 401 error codes
- Check logs for authorization failures
- Test with tampered auth tokens
- Ensure client-side hiding + server-side enforcement

**Status**:
- [ ] Not Started
- [ ] In Progress
- [ ] Complete

---

### 15. [ ] Data Volume and Integration Testing

**Objective**: Validate system behavior with 15K-20K records across all features.

**Layer**: Testing

**Subtasks**:
- [ ] Test personal dashboard with full dataset
- [ ] Test admin dashboard with full dataset
- [ ] Test all filter combinations work correctly
- [ ] Test pagination through entire dataset
- [ ] Verify aggregation accuracy with large data
- [ ] Test real-time updates still work
- [ ] Test memory usage acceptable
- [ ] Test UI responsiveness with large data

**Effort**: Medium (4h)
**Complexity**: Medium
**Assigned to**: QA / Developer
**Depends on**: Task 12, Task 13
**Blocks**: None

**Success Criteria**:
- [ ] All features functional with full dataset
- [ ] Aggregations accurate at scale
- [ ] Pagination works through all data
- [ ] Filters return correct results
- [ ] Real-time updates still functional
- [ ] Memory usage within acceptable limits
- [ ] UI remains responsive
- [ ] No data corruption or loss

**Technical Notes**:
- Test on both personal and admin dashboards
- Verify daily totals with manual calculations
- Sample random days for accuracy checks
- Monitor browser memory in dev tools
- Test on slower devices/connections
- Document any scalability issues
- Verify no N+1 query problems

**Status**:
- [ ] Not Started
- [ ] In Progress
- [ ] Complete

---

### 16. [ ] Production Readiness Checklist

**Objective**: Comprehensive review and validation that system is ready for production deployment.

**Layer**: Infrastructure

**Subtasks**:
- [ ] All Phase 1-4 features tested and working
- [ ] Performance benchmarks met
- [ ] Security review completed
- [ ] Authorization properly enforced
- [ ] Documentation complete and accurate
- [ ] Environment variables documented
- [ ] Database schema finalized
- [ ] Error handling comprehensive
- [ ] Deployment runbook created
- [ ] Rollback plan documented

**Effort**: Medium (4h)
**Complexity**: Low
**Assigned to**: Tech Lead / Senior Developer
**Depends on**: All other tasks
**Blocks**: None

**Success Criteria**:
- [ ] All critical features tested end-to-end
- [ ] All performance targets met
- [ ] Security vulnerabilities addressed
- [ ] Documentation complete
- [ ] Deployment process documented
- [ ] Rollback procedure tested
- [ ] Team aligned on release plan
- [ ] Production environment configured

**Technical Notes**:
- Review all success criteria from Phases 1-4
- Run full regression test suite
- Security audit checklist completed
- Performance baseline documented
- Create deployment runbook with steps
- Document environment setup for production
- Plan gradual rollout if possible
- Prepare monitoring and alerting

**Status**:
- [ ] Not Started
- [ ] In Progress
- [ ] Complete

---

## Task Dependency Graph

**Critical Path (must complete in order)**:
```
Task 1 (Admin Indexes)
  → Task 3 (Role Verification)
    → Task 4 (Authorization Middleware)
      → Task 5 (Admin Query)
        → Task 9 (Admin Dashboard)
```

**Parallel Work Opportunities**:
```
After Phase 3 complete:
- Task 1 (Indexes) || Task 2 (Seed Script) || Task 3 (Role System)

Admin Query Features (parallel):
- Task 6 (User Filter) || Task 7 (Date Range Filter) after Task 5

Frontend Work (parallel):
- Task 8 (Admin Nav) || Task 10 (Filter Controls) || Task 11 (Stub Pages)

Testing (parallel after features):
- Task 13 (Performance) || Task 14 (Authorization) || Task 15 (Data Volume)
```

---

## Effort Summary

| Category | Tasks | Hours | Percentage |
|----------|-------|-------|------------|
| Infrastructure | 3 | 14h | 20% |
| Backend | 5 | 22h | 31% |
| Frontend | 4 | 22h | 31% |
| Testing | 4 | 14h | 20% |
| **Total** | **16** | **70h** | **100%** |

---

## Tracking Progress

Update task status as you work:
1. Mark task as `In Progress` when you start
2. Check off subtasks as you complete them
3. Mark task as `Complete` when all success criteria met
4. Update progress percentage at top of document

**Note**: This is the final phase. Ensure Phases 1-3 are complete, tested, and performing well before starting Phase 4. This phase prepares the system for production deployment with enterprise-scale data.

---

## Production Deployment Considerations

After completing Phase 4:
- Run full regression test suite
- Perform security audit
- Verify all performance targets met
- Complete documentation review
- Plan staged rollout
- Prepare monitoring and alerting
- Create incident response plan
- Schedule training for admin users
