# Phase 2: Personal Dashboard with OT/UT Detection

## Objective
Transform the basic worklog list into an intelligent personal dashboard that automatically calculates daily hours, detects overtime/undertime based on user-specific thresholds, and displays visual indicators. This phase delivers the core value proposition of the worklog tracking system.

## Scope

### Features
- Daily hours aggregation (sum all worklogs per day)
- OT/UT badge calculation using user's dailyMinHours and dailyMaxHours
- Date-grouped worklog display (worklogs organized by day)
- Visual OT/UT badges with distinct styling
- Basic pagination (initial implementation, not optimized yet)
- Sorting by date descending (most recent first)

### Frontend
- Redesigned personal dashboard with date grouping
  - Each day displayed as section header (e.g., "Monday, January 15, 2025")
  - Individual worklogs nested under each day
  - Daily total hours displayed per day
- OT/UT badge components
  - OT badge: displayed when total > dailyMaxHours
  - UT badge: displayed when total < dailyMinHours
  - No badge: when within min/max range
  - Color-coded badges (e.g., red for OT, yellow for UT)
- Enhanced worklog display
  - Time worked, task ID (if present), description (if present)
  - Clear visual hierarchy
- Basic pagination UI
  - "Load more" button or simple page navigation
  - Page size: 10 days per page
  - Total count display (e.g., "Showing 1-10 of 47 days")

### Backend
- Enhanced query: `api.worklogs.getMyWorklogs`
  - Server-side grouping by date
  - Daily totals calculation
  - OT/UT status computation per day
  - Fetch user's dailyMinHours and dailyMaxHours for comparison
  - Pagination support (initial implementation with limit/offset)
  - Sort by date descending
- Optimized data structure for frontend
  - Returns grouped days with pre-calculated totals and status
  - Format: `{ days: [{ date, totalHours, status, worklogs: [...] }], totalCount }`
- Database query optimization
  - Efficient aggregation queries
  - Minimize data transfer (only necessary fields)

### Infrastructure
- Unit tests for aggregation logic
  - Test daily totals calculation with various hour combinations
  - Test OT/UT badge logic with boundary conditions (exactly at min/max)
  - Test date grouping across month/year boundaries
- Performance baseline measurement
  - Measure query time for 30 days of data
  - Target: <200ms for personal dashboard query
- Documentation updates
  - Document OT/UT calculation logic
  - API documentation for enhanced query

## Success Criteria
- [ ] Personal dashboard groups worklogs by date (descending)
- [ ] Daily total hours calculated correctly for each day
- [ ] OT badge appears when daily total exceeds user's dailyMaxHours
- [ ] UT badge appears when daily total is below user's dailyMinHours
- [ ] No badge shown when daily total is within min/max range
- [ ] Badges are visually distinct and easily recognizable
- [ ] Edge case: Day with exactly dailyMaxHours shows no OT badge
- [ ] Edge case: Day with exactly dailyMinHours shows no UT badge
- [ ] Multiple worklogs on same day aggregate correctly
- [ ] Empty days (no worklogs) not displayed
- [ ] Pagination works with 10 days per page
- [ ] Total count accurate across pagination
- [ ] Query performance <200ms for 30 days of data
- [ ] Mobile responsive dashboard layout
- [ ] Real-time updates: new worklog triggers recalculation and UI update

## Effort Breakdown
- Frontend: 40%
  - Dashboard redesign with grouping: 20%
  - Badge components and styling: 10%
  - Pagination UI: 10%
- Backend: 45%
  - Aggregation logic implementation: 20%
  - OT/UT calculation: 15%
  - Pagination backend: 10%
- Infrastructure/Testing: 15%
  - Unit tests for business logic: 10%
  - Performance testing: 5%

## Dependencies
- **Requires Phase 1**: Authentication, basic worklog creation, and database schema must be complete
- User profiles must have dailyMinHours and dailyMaxHours configured

## Risks & Mitigation

### Risk: Aggregation Performance Degradation
- **Impact**: High - Slow queries would make dashboard unusable
- **Probability**: Medium - Aggregation can be expensive for large datasets
- **Mitigation**: Server-side aggregation, test with larger datasets (100+ days), monitor query performance
- **Contingency**: Implement caching layer for aggregated results if needed

### Risk: Floating-Point Precision Issues
- **Impact**: Medium - Incorrect totals could show wrong OT/UT status
- **Probability**: Medium - JavaScript floating-point arithmetic can be imprecise
- **Mitigation**: Round to 2 decimal places consistently, use 0.01 precision, test with edge cases like 0.25 + 0.25 + 0.25
- **Contingency**: Use integer representation (cents of hours) if precision issues arise

### Risk: Complex Business Logic Bugs
- **Impact**: High - Incorrect OT/UT detection undermines core value
- **Probability**: Medium - Multiple edge cases and boundary conditions
- **Mitigation**: Comprehensive unit tests covering all scenarios, manual testing with known datasets
- **Contingency**: Extensive logging of calculations for debugging in development

### Risk: Pagination Implementation Issues
- **Impact**: Medium - Could show duplicate or missing data
- **Probability**: Medium - Offset-based pagination has pitfalls
- **Mitigation**: Test pagination thoroughly, verify no data duplication across pages, consistent ordering
- **Contingency**: Note: Will be replaced with cursor-based pagination in Phase 3 for better performance

### Risk: Real-time Update Race Conditions
- **Impact**: Medium - Stale data could show incorrect totals briefly
- **Probability**: Low - Convex handles reactivity well
- **Mitigation**: Test concurrent worklog creation and dashboard viewing, verify Convex subscription behavior
- **Contingency**: Add manual refresh button if automatic updates inconsistent

## Notes
- This phase uses basic offset-based pagination for simplicity - will upgrade to cursor-based in Phase 3
- Filters (OT/UT/normal days) intentionally deferred to Phase 3 to focus on core aggregation
- Date grouping should handle edge cases like worklogs spanning multiple years
- Badge design should be accessible (not relying on color alone - use icons/text too)
- Consider adding totals summary at top of dashboard (total OT days, UT days, normal days)
- Performance testing should use realistic data (create 90 days of worklogs for testing)
