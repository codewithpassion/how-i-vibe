# Phase 2: Personal Dashboard with OT/UT Detection

**Status**: 🔴 Not Started
**Progress**: 0/12 tasks complete (0%)

---

## Infrastructure Tasks

### 1. [ ] Unit Testing Framework Setup

**Objective**: Configure testing infrastructure for aggregation logic and business rules.

**Layer**: Infrastructure

**Subtasks**:
- [ ] Install Vitest and testing library dependencies
- [ ] Configure Vitest for TypeScript and React
- [ ] Set up test utilities for Convex functions
- [ ] Create test helpers for date/time mocking
- [ ] Configure test coverage reporting
- [ ] Add npm scripts for running tests

**Effort**: Medium (4h)
**Complexity**: Medium
**Assigned to**: Developer
**Depends on**: None (builds on Phase 1)
**Blocks**: Task 5, Task 9

**Success Criteria**:
- [ ] Vitest runs successfully with `npm test`
- [ ] Test coverage reports generate correctly
- [ ] Can mock Convex context for testing
- [ ] Date/time utilities available for tests
- [ ] Tests run in CI environment
- [ ] All test files discoverable by Vitest

**Technical Notes**:
- Use Vitest (fast, Vite-native test runner)
- Install @testing-library/react for component tests
- Create mock utilities for ctx.auth and ctx.db
- Use vi.setSystemTime for date mocking
- Configure code coverage threshold (70% minimum)

**Status**:
- [ ] Not Started
- [ ] In Progress
- [ ] Complete

---

## Feature: Daily Hours Aggregation

### 2. [ ] Daily Aggregation Logic - Database

**Objective**: Implement server-side daily aggregation to sum worklogs by date efficiently.

**Layer**: API

**Subtasks**:
- [ ] Enhance api.worklogs.getMyWorklogs query
- [ ] Fetch user's dailyMinHours and dailyMaxHours thresholds
- [ ] Group worklogs by date on server-side
- [ ] Calculate total hours per day (sum of workedHours)
- [ ] Format aggregation result structure
- [ ] Optimize query performance with proper indexes

**Effort**: Large (8h)
**Complexity**: High
**Assigned to**: Backend Developer
**Depends on**: None (builds on Phase 1)
**Blocks**: Task 3, Task 6

**Success Criteria**:
- [ ] Query groups worklogs by date correctly
- [ ] Daily totals calculated accurately
- [ ] User thresholds fetched efficiently
- [ ] Handles multiple worklogs per day
- [ ] Handles days with single worklog
- [ ] Returns structured data: { date, totalHours, worklogs[] }
- [ ] Query executes in <200ms for 30 days

**Technical Notes**:
- Use Convex aggregation patterns
- Group by normalized date (strip time component)
- Round total hours to 2 decimal places
- Return days in descending order
- Pre-fetch user profile for thresholds
- Use existing worklog.userId index

**Status**:
- [ ] Not Started
- [ ] In Progress
- [ ] Complete

---

### 3. [ ] OT/UT Status Calculation Logic

**Objective**: Implement business logic to determine OT/UT status for each day based on user thresholds.

**Layer**: API

**Subtasks**:
- [ ] Create status calculation function
- [ ] Implement OT detection (totalHours > dailyMaxHours)
- [ ] Implement UT detection (totalHours < dailyMinHours)
- [ ] Implement normal status (within min/max range)
- [ ] Handle edge cases (exactly at threshold)
- [ ] Add status field to daily aggregation result
- [ ] Return status enum: "OT" | "UT" | "normal"

**Effort**: Medium (4h)
**Complexity**: Medium
**Assigned to**: Backend Developer
**Depends on**: Task 2
**Blocks**: Task 6

**Success Criteria**:
- [ ] Status "OT" when totalHours > dailyMaxHours
- [ ] Status "UT" when totalHours < dailyMinHours
- [ ] Status "normal" when dailyMinHours <= totalHours <= dailyMaxHours
- [ ] Edge case: totalHours === dailyMaxHours returns "normal"
- [ ] Edge case: totalHours === dailyMinHours returns "normal"
- [ ] Floating-point precision handled correctly
- [ ] All edge cases covered with unit tests

**Technical Notes**:
- Use strict comparison operators (<, >, <=, >=)
- Round hours to 2 decimals before comparison
- Document threshold boundary behavior
- Return as enum string for type safety
- Consider creating reusable utility function

**Status**:
- [ ] Not Started
- [ ] In Progress
- [ ] Complete

---

### 4. [ ] Basic Pagination Backend

**Objective**: Implement offset-based pagination for worklog days (initial simple implementation).

**Layer**: API

**Subtasks**:
- [ ] Add pagination parameters to getMyWorklogs query (limit, offset)
- [ ] Implement limit/offset logic in aggregation
- [ ] Calculate total count of days
- [ ] Return pagination metadata (totalCount, hasMore)
- [ ] Set default page size to 10 days
- [ ] Validate pagination parameters

**Effort**: Medium (4h)
**Complexity**: Medium
**Assigned to**: Backend Developer
**Depends on**: Task 2
**Blocks**: Task 7

**Success Criteria**:
- [ ] Pagination limits results to specified page size
- [ ] Offset skips correct number of days
- [ ] Total count accurate across all pages
- [ ] hasMore flag indicates more data available
- [ ] Default limit is 10 days
- [ ] Maximum limit enforced (e.g., 100 days)
- [ ] No duplicate days across pages

**Technical Notes**:
- Use offset-based pagination (simple but not optimal)
- Will be replaced with cursor-based in Phase 3
- Return structure: { days: [...], totalCount: number, hasMore: boolean }
- Validate: limit > 0 and offset >= 0
- Consider performance with large offsets

**Status**:
- [ ] Not Started
- [ ] In Progress
- [ ] Complete

---

### 5. [ ] Aggregation Logic Unit Tests

**Objective**: Comprehensive unit tests for daily aggregation and OT/UT calculation logic.

**Layer**: Testing

**Subtasks**:
- [ ] Test daily totals with multiple worklogs per day
- [ ] Test daily totals with single worklog per day
- [ ] Test OT detection at various thresholds
- [ ] Test UT detection at various thresholds
- [ ] Test boundary conditions (exactly at min/max)
- [ ] Test floating-point precision (e.g., 0.25 + 0.25 + 0.25)
- [ ] Test date grouping across month/year boundaries
- [ ] Test with empty worklog sets

**Effort**: Large (8h)
**Complexity**: Medium
**Assigned to**: Developer
**Depends on**: Task 1, Task 2, Task 3
**Blocks**: None

**Success Criteria**:
- [ ] 100% coverage of aggregation functions
- [ ] All edge cases tested and passing
- [ ] Boundary conditions validated
- [ ] Floating-point precision verified
- [ ] Test suite runs in <5 seconds
- [ ] Tests are deterministic (no flakiness)
- [ ] Clear test descriptions and assertions

**Technical Notes**:
- Mock user thresholds for controlled testing
- Test with various hour combinations
- Use exact decimal values (avoid 0.1 + 0.2 issues)
- Test leap years and month boundaries
- Create test data factories for consistency
- Document expected behavior in test names

**Status**:
- [ ] Not Started
- [ ] In Progress
- [ ] Complete

---

## Feature: Dashboard UI Redesign

### 6. [ ] Date-Grouped Dashboard Backend Integration

**Objective**: Update getMyWorklogs query to return fully structured data for grouped dashboard display.

**Layer**: API

**Subtasks**:
- [ ] Finalize return structure with all required fields
- [ ] Include date, totalHours, status, worklogs[] per day
- [ ] Sort days by date descending
- [ ] Sort worklogs within each day by creation time
- [ ] Format dates consistently (ISO or Unix timestamp)
- [ ] Add query response type definitions

**Effort**: Small (2h)
**Complexity**: Low
**Assigned to**: Backend Developer
**Depends on**: Task 2, Task 3
**Blocks**: Task 8

**Success Criteria**:
- [ ] Query returns structured grouped data
- [ ] Each day object has: date, totalHours, status, worklogs[]
- [ ] Days sorted descending (most recent first)
- [ ] Worklogs within day sorted by time
- [ ] TypeScript types exported for frontend
- [ ] Response format documented

**Technical Notes**:
- Export TypeScript type for query response
- Use consistent date format throughout
- Ensure null safety for optional fields
- Document response structure in API docs
- Consider adding example response in comments

**Status**:
- [ ] Not Started
- [ ] In Progress
- [ ] Complete

---

### 7. [ ] Pagination UI Component

**Objective**: Build pagination controls for loading more days or navigating pages.

**Layer**: Frontend

**Subtasks**:
- [ ] Create Pagination component
- [ ] Implement "Load More" button with loading state
- [ ] Display current count (e.g., "Showing 1-10 of 47 days")
- [ ] Add loading spinner during data fetch
- [ ] Handle end-of-list state (no more data)
- [ ] Style pagination controls with shadcn

**Effort**: Medium (4h)
**Complexity**: Low
**Assigned to**: Frontend Developer
**Depends on**: Task 4
**Blocks**: Task 8

**Success Criteria**:
- [ ] "Load More" button functional and styled
- [ ] Loading state shows spinner, disables button
- [ ] Count display accurate and updates correctly
- [ ] End-of-list shows "No more entries" message
- [ ] Mobile responsive design
- [ ] Smooth UX with no layout shift

**Technical Notes**:
- Use shadcn Button component
- Show loading spinner from shadcn
- Track current page offset in component state
- Append new results to existing list
- Disable button when loading or at end
- Consider infinite scroll as alternative

**Status**:
- [ ] Not Started
- [ ] In Progress
- [ ] Complete

---

### 8. [ ] Dashboard Redesign with Date Grouping

**Objective**: Redesign personal dashboard to group worklogs by day with daily totals and status.

**Layer**: Frontend

**Subtasks**:
- [ ] Update Dashboard component to use new query structure
- [ ] Create DayGroup component for each date section
- [ ] Display day header (e.g., "Monday, January 15, 2025")
- [ ] Show daily total hours in day header
- [ ] Render individual worklogs nested under each day
- [ ] Integrate pagination component
- [ ] Update loading and empty states

**Effort**: Large (8h)
**Complexity**: High
**Assigned to**: Frontend Developer
**Depends on**: Task 6, Task 7
**Blocks**: Task 10

**Success Criteria**:
- [ ] Dashboard displays days in descending order
- [ ] Each day shows formatted date header
- [ ] Daily total hours displayed prominently
- [ ] Individual worklogs nested and styled
- [ ] Pagination works correctly
- [ ] Loading state during query
- [ ] Empty state for new users
- [ ] Real-time updates when new worklog added

**Technical Notes**:
- Format dates with date-fns or Intl.DateTimeFormat
- Use shadcn Card for day groups
- Implement visual hierarchy (day > worklogs)
- Maintain mobile responsiveness
- Use Convex useQuery for reactive updates
- Handle timezone considerations

**Status**:
- [ ] Not Started
- [ ] In Progress
- [ ] Complete

---

## Feature: OT/UT Badge System

### 9. [ ] Badge Component Library

**Objective**: Create reusable badge components for OT, UT, and normal status display.

**Layer**: Frontend

**Subtasks**:
- [ ] Create StatusBadge component
- [ ] Design OT badge (color, icon, text)
- [ ] Design UT badge (color, icon, text)
- [ ] Implement conditional rendering based on status
- [ ] Ensure accessibility (not color-only)
- [ ] Add hover tooltips with explanation

**Effort**: Medium (4h)
**Complexity**: Low
**Assigned to**: Frontend Developer
**Depends on**: Task 1
**Blocks**: Task 10

**Success Criteria**:
- [ ] OT badge visually distinct (e.g., red background)
- [ ] UT badge visually distinct (e.g., yellow background)
- [ ] No badge shown for "normal" status
- [ ] Badges include icon + text (not color-only)
- [ ] Accessible (ARIA labels, sufficient contrast)
- [ ] Tooltips explain OT/UT meaning
- [ ] Mobile-friendly badge sizing

**Technical Notes**:
- Use shadcn Badge component as base
- OT color: red/destructive variant
- UT color: yellow/warning variant
- Include icons (e.g., arrow-up for OT, arrow-down for UT)
- Add aria-label for screen readers
- Keep badge text concise ("OT", "UT")

**Status**:
- [ ] Not Started
- [ ] In Progress
- [ ] Complete

---

### 10. [ ] Badge Integration in Dashboard

**Objective**: Integrate OT/UT badges into dashboard day headers based on calculated status.

**Layer**: Frontend

**Subtasks**:
- [ ] Add StatusBadge to DayGroup component
- [ ] Pass day status from query to badge
- [ ] Position badge in day header
- [ ] Show badge only for OT/UT days (not normal)
- [ ] Test badge rendering with all status types
- [ ] Verify badge updates in real-time

**Effort**: Small (2h)
**Complexity**: Low
**Assigned to**: Frontend Developer
**Depends on**: Task 8, Task 9
**Blocks**: None

**Success Criteria**:
- [ ] Badge appears for OT days
- [ ] Badge appears for UT days
- [ ] No badge for normal days
- [ ] Badge positioned clearly in header
- [ ] Badge updates when status changes
- [ ] Responsive layout maintains badge visibility

**Technical Notes**:
- Conditionally render based on status prop
- Position badge inline with date header
- Ensure visual hierarchy clear
- Test with mixed day statuses
- Verify real-time badge updates after worklog creation

**Status**:
- [ ] Not Started
- [ ] In Progress
- [ ] Complete

---

## Testing & Performance

### 11. [ ] Performance Baseline Measurement

**Objective**: Measure and document query performance with realistic dataset.

**Layer**: Infrastructure

**Subtasks**:
- [ ] Create 90 days of test worklogs
- [ ] Measure getMyWorklogs query latency
- [ ] Test with various pagination sizes
- [ ] Document p50, p95, p99 latency
- [ ] Verify <200ms target for 30 days
- [ ] Create performance regression tests

**Effort**: Medium (4h)
**Complexity**: Medium
**Assigned to**: Developer
**Depends on**: Task 2, Task 4
**Blocks**: None

**Success Criteria**:
- [ ] 90-day dataset created for testing
- [ ] Query latency measured with Convex dashboard
- [ ] p95 latency < 200ms for 30 days
- [ ] Pagination performance acceptable
- [ ] Performance baselines documented
- [ ] Regression test suite in place

**Technical Notes**:
- Use Convex dashboard for latency metrics
- Test with realistic worklog distribution
- Measure both cold and warm query performance
- Document baseline for Phase 3 optimization
- Create seed script for perf testing data
- Consider browser performance metrics too

**Status**:
- [ ] Not Started
- [ ] In Progress
- [ ] Complete

---

### 12. [ ] End-to-End Feature Testing

**Objective**: Manual testing of complete Phase 2 features including aggregation and badges.

**Layer**: Testing

**Subtasks**:
- [ ] Test dashboard displays grouped by date
- [ ] Test daily totals calculated correctly
- [ ] Test OT badge appears when threshold exceeded
- [ ] Test UT badge appears when below threshold
- [ ] Test no badge for normal days
- [ ] Test pagination "Load More" functionality
- [ ] Test real-time updates after creating worklogs
- [ ] Test edge cases (exactly at threshold)
- [ ] Test mobile responsive design
- [ ] Test with multiple worklogs per day

**Effort**: Medium (4h)
**Complexity**: Low
**Assigned to**: QA / Developer
**Depends on**: All feature tasks
**Blocks**: None

**Success Criteria**:
- [ ] All aggregation scenarios tested
- [ ] Badge display verified for all statuses
- [ ] Pagination works across multiple pages
- [ ] Real-time updates confirmed
- [ ] Edge cases handled correctly
- [ ] Mobile UX validated
- [ ] No console errors or warnings
- [ ] Performance acceptable on slower devices

**Technical Notes**:
- Test with varied user thresholds
- Create worklogs crossing threshold boundaries
- Test with empty days (no worklogs)
- Verify floating-point calculations
- Test on multiple browsers
- Document any issues or edge cases found

**Status**:
- [ ] Not Started
- [ ] In Progress
- [ ] Complete

---

## Task Dependency Graph

**Critical Path (must complete in order)**:
```
Task 2 (Aggregation Logic)
  → Task 3 (OT/UT Calculation)
    → Task 6 (Backend Integration)
      → Task 8 (Dashboard Redesign)
        → Task 10 (Badge Integration)
```

**Parallel Work Opportunities**:
```
After Phase 1 complete:
- Task 1 (Testing Setup) || Task 2 (Aggregation Logic)

Parallel UI Work:
- Task 7 (Pagination UI) || Task 9 (Badge Components)

After Task 2:
- Task 4 (Pagination Backend) || Task 3 (OT/UT Logic)

Testing in Parallel:
- Task 5 (Unit Tests) can proceed alongside feature development
- Task 11 (Performance) can run after Task 2 complete
```

---

## Effort Summary

| Category | Tasks | Hours | Percentage |
|----------|-------|-------|------------|
| Infrastructure | 2 | 8h | 15% |
| Backend | 4 | 18h | 33% |
| Frontend | 4 | 18h | 33% |
| Testing | 2 | 12h | 22% |
| **Total** | **12** | **54h** | **100%** |

---

## Tracking Progress

Update task status as you work:
1. Mark task as `In Progress` when you start
2. Check off subtasks as you complete them
3. Mark task as `Complete` when all success criteria met
4. Update progress percentage at top of document

**Note**: This phase builds directly on Phase 1 infrastructure. Ensure Phase 1 is fully complete and tested before starting Phase 2.
