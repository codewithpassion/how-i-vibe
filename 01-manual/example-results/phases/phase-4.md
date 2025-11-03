# Phase 4: Admin Dashboard & Historical Data at Scale

## Objective
Complete the system for organizational use by implementing admin-level features for viewing and managing all users' worklogs. Generate 10 years of realistic historical data to validate performance at scale and prepare for production deployment with 10,000-user capacity.

## Scope

### Features
- Admin-only dashboard with all-user worklog viewing
- User identification in worklog entries (display user names)
- Enhanced sorting: date DESC, then user name ASC
- User-specific filtering for admins
- Date range filtering (start date, end date)
- Combined filter support (e.g., specific user + OT days)
- Historical data seeding (10 years, realistic patterns)
- Admin stub pages (User Management, Edit User Hours)
- Role-based access enforcement
- Performance validation with large datasets (15K-20K records)

### Frontend
- Admin dashboard page
  - All features from personal dashboard PLUS user identification
  - User name prominently displayed in each worklog entry/group
  - OT/UT badges apply per-user thresholds
- Enhanced admin filters
  - User filter: dropdown or search/autocomplete for user selection
  - Date range filter: start date and end date pickers
  - All personal dashboard filters (OT/UT/normal)
  - Clear all filters button
  - Active filter tags/badges showing current filter state
- Admin navigation menu items
  - Admin Dashboard (implemented)
  - User Management → stub page
  - Edit User Hours → stub page
- Stub page components
  - User Management: "Feature Coming Soon: Manage user accounts, roles, and permissions."
  - Edit User Hours: "Feature Coming Soon: Configure minimum and maximum daily hours for each employee."
  - Consistent layout and navigation
- Role-based UI elements
  - Show/hide admin menu items based on user role
  - Admin badge or indicator in header
  - Different dashboard routes for user vs admin

### Backend
- New query: `api.worklogs.getAllWorklogs` (admin only)
  - All features from `getMyWorklogs` plus multi-user support
  - Input parameters:
    - filter: OT/UT/normal/all
    - userId: optional user filter
    - startDate: optional date range start
    - endDate: optional date range end
    - cursor: pagination cursor
    - pageSize: results per page (default 10, max 100)
  - Authorization: verify admin role server-side
  - Reject non-admin requests with clear error
  - Sorting: date DESC, then userName ASC
  - Include user name in response (denormalize or join)
- Enhanced authorization middleware
  - Role verification for admin-only functions
  - Extract role from Clerk metadata or database
  - Reject unauthorized access attempts
  - Log authorization failures for security monitoring
- Database indexes for admin queries
  - Compound index on (date, userId) for admin chronological queries
  - Index on users.role for admin privilege checks
  - Verify index usage with Convex dashboard
- Historical data seed script
  - Generate 10 years of worklogs (2015-2025)
  - 0-5 worklogs per user per day (weighted toward 2-3)
  - Realistic distribution: 70% normal, 15% OT, 15% UT
  - Varied worked hours (0.25 to 12 hour increments)
  - Random task IDs and descriptions
  - Some null taskId/description for variety
  - Weekday weighting (more activity Mon-Fri)
  - Total: ~15,000-20,000 worklogs
  - Progress logging during execution
  - Idempotent (can re-run safely)
  - Command: `npm run seed:historical`

### Infrastructure
- Performance testing with historical data
  - Admin query performance with full dataset
  - Target: <1 second p95 latency
  - Test with various filter combinations
  - Measure pagination performance across deep pages
- Integration tests for admin features
  - Admin can view all users' worklogs
  - Non-admin cannot access admin endpoints
  - User filtering returns correct subset
  - Date range filtering works correctly
  - Combined filters apply properly
- Authorization testing
  - Verify role checks in all admin functions
  - Test API manipulation attempts (non-admin trying admin endpoints)
  - Verify error messages don't leak sensitive info
- Data volume testing
  - Verify aggregation accuracy with 20K records
  - Test pagination through 100+ pages
  - Memory usage monitoring
  - Query performance degradation analysis
- Production readiness checklist
  - All critical paths tested
  - Performance benchmarks met
  - Security review completed
  - Documentation updated
  - Deployment runbook created

## Success Criteria
- [ ] Admin user can log in and access admin dashboard
- [ ] Admin dashboard displays all users' worklogs with user names
- [ ] Admin dashboard sorted by date DESC, then userName ASC
- [ ] Non-admin user cannot access admin dashboard (403 error)
- [ ] Admin can filter by specific user
- [ ] Admin can filter by date range (start and end dates)
- [ ] Admin can combine filters (e.g., user + OT days + date range)
- [ ] OT/UT badges use each user's specific thresholds in admin view
- [ ] Clear filters button resets all filter criteria
- [ ] Pagination works efficiently with full historical dataset
- [ ] Admin stub pages accessible via admin menu
- [ ] Seed script generates 10 years of data in <5 minutes
- [ ] Seed script is idempotent (can re-run without duplicates)
- [ ] Admin query performance <1 second for 30 days all users
- [ ] System handles 15K-20K total worklogs without degradation
- [ ] Authorization properly enforced server-side (not just UI)
- [ ] All Phase 1-3 features still work correctly
- [ ] Mobile responsive admin dashboard

## Effort Breakdown
- Frontend: 35%
  - Admin dashboard UI: 20%
  - Enhanced filters: 10%
  - Stub pages: 5%
- Backend: 45%
  - Admin query implementation: 20%
  - Authorization enforcement: 10%
  - Historical seed script: 15%
- Infrastructure/Testing: 20%
  - Performance testing: 8%
  - Authorization testing: 7%
  - Production readiness: 5%

## Dependencies
- **Requires Phase 1**: Authentication and authorization foundation
- **Requires Phase 2**: Aggregation logic for multi-user display
- **Requires Phase 3**: Cursor-based pagination for performance
- User role assignment must be configured in Clerk or database

## Risks & Mitigation

### Risk: Admin Query Performance at Scale
- **Impact**: Critical - Slow admin queries would make system unusable for managers
- **Probability**: High - Multi-user aggregation is expensive
- **Mitigation**: Compound database indexes, server-side filtering, cursor-based pagination, performance testing with full dataset
- **Contingency**: Implement query result caching, consider pre-aggregated summary tables

### Risk: Authorization Bypass Vulnerabilities
- **Impact**: Critical - Would expose all user data to unauthorized access
- **Probability**: Low - But high severity requires careful attention
- **Mitigation**: Server-side role verification in every admin function, security testing, code review for auth logic
- **Contingency**: Add request logging and monitoring for unauthorized access attempts

### Risk: Historical Data Seed Script Timeout
- **Impact**: Medium - Would block development and testing with realistic data
- **Probability**: Medium - 20K records could exceed execution limits
- **Mitigation**: Batch processing (1,000 records per batch), progress logging, checkpoint/resume capability
- **Contingency**: Split seed script into multiple smaller scripts (by year)

### Risk: Complex Filter Logic Bugs
- **Impact**: Medium - Incorrect filtering would show wrong data
- **Probability**: Medium - Multiple filter combinations to test
- **Mitigation**: Comprehensive integration tests for filter combinations, manual testing with known datasets
- **Contingency**: Disable problematic filter combinations, offer filters separately

### Risk: User Name Display Performance
- **Impact**: Medium - Could slow queries if implemented inefficiently
- **Probability**: Medium - Database joins or N+1 queries are common pitfalls
- **Mitigation**: Denormalize user name into worklog query result, or fetch users in single batch query
- **Contingency**: Cache user profiles client-side to reduce lookups

### Risk: Role Assignment Configuration Issues
- **Impact**: High - Users with wrong roles would have incorrect access
- **Probability**: Low - But manual role assignment is error-prone
- **Mitigation**: Document role assignment procedure clearly, provide admin UI for role management (stub for now), verify role in development
- **Contingency**: Emergency role override procedure in Clerk dashboard

## Notes
- Admin features should gracefully handle edge cases: users with no worklogs, date ranges with no data
- Consider adding export functionality in future (not in Phase 4 scope)
- Seed script should generate realistic task IDs (e.g., "PROJ-1234") and descriptions
- Historical data should span full date range evenly (not clustered in recent dates)
- Performance testing should measure p50, p95, and p99 latencies, not just averages
- Admin authorization should be fail-secure: deny by default if role unclear
- User dropdown for filtering should support search/autocomplete for 10K users
- Consider adding summary statistics for admins (total users, total hours logged, etc.)
- Stub pages should indicate these are placeholder features planned for future releases
- Date range filter should handle timezone considerations (use UTC consistently)
- Seed script should log warnings for any data generation anomalies
- Performance benchmarks should be documented for future regression testing
- Consider rate limiting for admin queries if abuse potential exists
- Admin dashboard should support export to CSV for reporting (future enhancement)
