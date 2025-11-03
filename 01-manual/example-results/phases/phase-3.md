# Phase 3: Enhanced Filtering & Performance Optimization

## Objective
Enhance the personal dashboard with powerful filtering capabilities and optimize pagination for large historical datasets. This phase improves usability for users who need to analyze specific work patterns (OT/UT days) and prepares the system to handle 10 years of historical data efficiently.

## Scope

### Features
- Filter worklogs by day status:
  - Filter by OT days only (days exceeding dailyMaxHours)
  - Filter by UT days only (days below dailyMinHours)
  - Filter by normal days (within min/max range)
  - Show all days (default)
- Clear filter button to reset to "all"
- Filter state persistence during session
- Cursor-based pagination (replacing offset-based from Phase 2)
- Improved pagination UX (infinite scroll or "Load more")
- Stub pages for navigation structure
- Performance optimization for large datasets

### Frontend
- Filter controls UI
  - Button group or dropdown for filter selection (All, OT, UT, Normal)
  - Active filter visual indicator
  - Filter badge/tag showing current filter
  - Clear filter button (when filter active)
- Enhanced pagination UI
  - "Load more" button with loading state
  - Infinite scroll option (smooth UX)
  - Display: "Showing X days of Y total"
  - End-of-list indicator ("No more entries")
- Filter state management
  - React state for current filter
  - Filter changes trigger new query with parameters
  - Pagination reset when filter changes
  - Session persistence (survives component remount)
- Stub pages implementation
  - "My Worklogs" menu item → stub page with "Feature Coming Soon" message
  - Consistent navigation structure
  - Layout maintained across stub and implemented pages

### Backend
- Enhanced query: `api.worklogs.getMyWorklogs`
  - Accept filter parameter (enum: "all" | "OT" | "UT" | "normal")
  - Server-side filtering based on daily totals vs thresholds
  - Cursor-based pagination with opaque cursor
  - Return structure: `{ days: [...], nextCursor: string | null, hasMore: boolean }`
- Database indexes optimization
  - Compound index on (userId, date) for efficient querying
  - Verify index usage with Convex dashboard
- Pagination cursor implementation
  - Cursor encodes: last seen document ID + date
  - Stateless cursor (no server-side session)
  - Stable sorting for consistent pagination
- Performance optimization
  - Query only necessary fields (no over-fetching)
  - Efficient aggregation strategy
  - Pre-calculate filter eligibility server-side

### Infrastructure
- Integration tests for filtering
  - Test each filter type returns correct subset
  - Test filter combinations (though only single filter in v1)
  - Verify pagination works with filters applied
- Performance testing with large datasets
  - Create test dataset: 365 days of worklogs per user
  - Measure query time with filters applied
  - Target: <300ms for filtered query
  - Benchmark cursor-based vs offset-based pagination
- Load testing preparation
  - Document performance baselines
  - Identify query optimization opportunities
- Error handling improvements
  - Network failure recovery
  - Graceful handling of invalid cursors
  - Filter validation

## Success Criteria
- [ ] User can filter dashboard to show only OT days
- [ ] User can filter dashboard to show only UT days
- [ ] User can filter dashboard to show only normal days
- [ ] "Clear filter" button resets to show all days
- [ ] Active filter visually indicated in UI
- [ ] Pagination maintains filter state across pages
- [ ] Filter change resets pagination to first page
- [ ] Cursor-based pagination implemented correctly
- [ ] No duplicate days when navigating pages
- [ ] No missing days when navigating pages
- [ ] Pagination performance <200ms per page load
- [ ] Filtered query performance <300ms
- [ ] "Load more" button shows loading state during fetch
- [ ] End-of-list indicator when no more data
- [ ] Stub pages accessible via navigation menu
- [ ] Filter state persists during browser session
- [ ] Mobile-responsive filter controls
- [ ] 365 days of data loads and filters efficiently

## Effort Breakdown
- Frontend: 40%
  - Filter UI components: 15%
  - Pagination UX improvements: 15%
  - Stub pages: 5%
  - State management: 5%
- Backend: 45%
  - Filter logic implementation: 15%
  - Cursor-based pagination: 20%
  - Database index optimization: 10%
- Infrastructure/Testing: 15%
  - Integration tests: 8%
  - Performance testing: 7%

## Dependencies
- **Requires Phase 2**: Daily aggregation and OT/UT calculation must be working
- Pagination infrastructure from Phase 2 (will be refactored)

## Risks & Mitigation

### Risk: Cursor-Based Pagination Complexity
- **Impact**: High - Broken pagination would severely degrade UX
- **Probability**: Medium - Cursor implementation can be tricky
- **Mitigation**: Use Convex's built-in pagination API (well-tested), extensive testing with edge cases
- **Contingency**: Keep Phase 2's offset pagination as fallback if cursor approach fails

### Risk: Filter + Pagination Interaction Bugs
- **Impact**: Medium - Could show wrong data or duplicate entries
- **Probability**: Medium - State management complexity increases
- **Mitigation**: Comprehensive integration tests for filter+pagination combinations, reset cursor when filter changes
- **Contingency**: Disable filter persistence if state bugs arise

### Risk: Performance Degradation with Large Datasets
- **Impact**: High - Slow queries undermine scalability goal
- **Probability**: Medium - Aggregation + filtering can be expensive
- **Mitigation**: Database indexes on query paths, server-side filtering, performance benchmarking with 1-year dataset
- **Contingency**: Implement query result caching if performance targets not met

### Risk: Invalid Cursor Handling
- **Impact**: Medium - Could cause errors or missing data
- **Probability**: Low - But cursors can be tampered with by users
- **Mitigation**: Validate cursor format server-side, graceful error handling for invalid cursors
- **Contingency**: Fall back to first page if cursor invalid

### Risk: Filter Logic Edge Cases
- **Impact**: Medium - Incorrect filtering undermines feature value
- **Probability**: Medium - Boundary conditions easy to get wrong
- **Mitigation**: Unit tests for filter logic, test with days exactly at min/max thresholds
- **Contingency**: Extensive logging to debug filter issues in development

## Notes
- Cursor implementation should be opaque to frontend (no cursor parsing in UI)
- Consider adding keyboard shortcuts for filters (e.g., "O" for OT, "U" for UT)
- Stub pages should maintain navigation structure for future features
- Filter state persistence can use sessionStorage (not localStorage - don't persist across sessions yet)
- Performance testing should use realistic distribution: 70% normal, 15% OT, 15% UT days
- Infinite scroll vs "Load more" decision can be made during implementation based on UX testing
- Consider adding filter count indicators (e.g., "OT Days (12)")
- Ensure filters work correctly with empty results (e.g., user has no OT days)
