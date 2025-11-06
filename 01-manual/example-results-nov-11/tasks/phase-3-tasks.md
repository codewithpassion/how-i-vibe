# Phase 3: Time Status & Analytics - Tasks

**Status**: 🔴 Not Started | 🟡 In Progress | 🟢 Complete
**Progress**: 10/10 tasks complete (100%)

---

## Infrastructure & Backend Tasks

### 1. [x] Extend User Schema with Time Settings ✅

**Objective**: Add daily hour requirements to user profile

**Layer**: Backend (Database)

**Subtasks**:
- [ ] Add `dailyMinHours` field to user schema (e.g., 8.0)
- [ ] Add `dailyMaxHours` field to user schema (e.g., 10.0)
- [ ] Add optional `timezone` field for future use
- [ ] Set default values (8 min, 10 max) for new users
- [ ] Create migration to add fields to existing users
- [ ] Update user sync to set defaults for Clerk users
- [ ] Test schema with sample data

**Effort**: Small (2h)
**Complexity**: Low
**Assigned to**: Backend Developer
**Depends on**: Phase 1 (Auth), Phase 2 (Worklogs)
**Blocks**: Task 2, Task 3

**Success Criteria**:
- [ ] User schema includes dailyMinHours and dailyMaxHours
- [ ] Default values applied to new and existing users
- [ ] Type generation works
- [ ] Migration runs without errors
- [ ] Sample user has correct defaults

**Technical Notes**:
- Default: dailyMinHours = 8, dailyMaxHours = 10
- Store as decimal numbers
- Update existing users with defaults
- Validate min < max

**Status**: [ ] Not Started [ ] In Progress [ ] Complete

---

### 2. [x] Implement Time Status Calculation Logic ✅

**Objective**: Create utility functions for calculating time status

**Layer**: Backend (Utilities)

**Subtasks**:
- [ ] Create `/convex/time-status.ts` utility file
- [ ] Implement `calculateTimeStatus()` function
- [ ] Function takes dailyTotal, dailyMin, dailyMax
- [ ] Returns 'normal' | 'undertime' | 'overtime'
- [ ] Handle edge cases (0 hours, negative values)
- [ ] Add validation for min < max
- [ ] Create unit tests for function
- [ ] Export for use in queries

**Effort**: Small (2h)
**Complexity**: Low
**Assigned to**: Backend Developer
**Depends on**: Task 1
**Blocks**: Task 3, Task 4

**Success Criteria**:
- [ ] Function returns correct status for all cases
- [ ] Normal: dailyTotal >= min AND <= max
- [ ] Undertime: dailyTotal < min
- [ ] Overtime: dailyTotal > max
- [ ] Edge cases handled correctly
- [ ] No TypeScript errors

**Technical Notes**:
- Pure function (no side effects)
- Use simple number comparisons
- Handle decimal hours (8.5, etc.)
- Clear, understandable logic
- Testable in isolation

**Status**: [ ] Not Started [ ] In Progress [ ] Complete

---

### 3. [x] Create Daily Aggregation Query Function ✅

**Objective**: Implement query to aggregate worklogs by day and calculate status

**Layer**: Backend (Queries)

**Subtasks**:
- [x] Create function to group worklogs by date
- [x] Sum workedHours for each day
- [x] Calculate time status for each day
- [x] Return daily totals with status
- [x] Implement optional date range filtering
- [x] Add sorting (most recent first)
- [x] Optimize with database indexes
- [x] Test with multiple entries per day

**Effort**: Medium (4h)
**Complexity**: Medium
**Assigned to**: Backend Developer
**Depends on**: Task 2
**Blocks**: Task 4

**Success Criteria**:
- [x] Query groups worklogs correctly by date
- [x] Daily totals calculated accurately
- [x] Time status correct for each day
- [x] Date range filtering works
- [x] Sorted by date descending
- [x] Query performance acceptable (< 1s)

**Technical Notes**:
- Use Convex query to fetch and aggregate
- Can aggregate on backend or frontend
- Use index on (userId, date) for efficiency
- Return format includes date, dailyTotal, timeStatus, count
- Handle days with no entries

**Status**: [x] Complete

---

### 4. [x] Extend Worklog List Query with Time Status ✅

**Objective**: Enhance list query from Phase 2 to include time status

**Layer**: Backend (Queries)

**Subtasks**:
- [x] Modify existing `listUserWorklogs()` query
- [x] Call daily aggregation for each date group
- [x] Include timeStatus in response
- [x] Include dailyTotal in response
- [x] Preserve pagination from Phase 2
- [x] Add optional filter by time status
- [x] Test with various data scenarios
- [x] Verify performance

**Effort**: Medium (4h)
**Complexity**: Medium
**Assigned to**: Backend Developer
**Depends on**: Phase 2 Task 4, Task 3
**Blocks**: Task 6, Task 7

**Success Criteria**:
- [x] Query returns timeStatus with results
- [x] dailyTotal included in response
- [x] Pagination still works
- [x] Filter by status works (all/undertime/overtime)
- [x] Performance acceptable with aggregation
- [x] Results correctly grouped by date

**Technical Notes**:
- Extend existing query rather than replace
- Maintain cursor-based pagination
- Calculate status per day efficiently
- Return grouped by date structure
- Preserve backward compatibility if possible

**Status**: [x] Complete

---

### 5. [x] Create User Settings Mutation ✅

**Objective**: Implement mutation to update user's daily hour settings

**Layer**: Backend (Mutations)

**Subtasks**:
- [x] Create `updateUserSettings(dailyMinHours, dailyMaxHours)` mutation
- [x] Add validation (min > 0, min < max, reasonable values)
- [x] Verify user can only update own settings
- [x] Create admin version for admins to update any user
- [x] Add audit logging for settings changes
- [x] Return updated user object
- [x] Handle error cases

**Effort**: Small (2h)
**Complexity**: Low
**Assigned to**: Backend Developer
**Depends on**: Task 1
**Blocks**: Task 7

**Success Criteria**:
- [x] User can update own settings
- [x] Values validated correctly
- [x] User cannot update others' settings
- [x] Admin can update any user's settings
- [x] Changes applied immediately
- [x] Error messages clear

**Technical Notes**:
- Validate min < max
- Validate reasonable ranges (0.5 to 24)
- Check user authorization
- Log changes for audit trail
- Return full user object

**Status**: [x] Not Started [ ] In Progress [x] Complete

---

## Frontend Core Tasks

### 6. [x] Build Time Status Badge Component

**Objective**: Create reusable component for displaying time status

**Layer**: Frontend (Components)

**Subtasks**:
- [x] Create `/app/components/worklog/time-status-badge.tsx`
- [x] Accept status ('normal' | 'undertime' | 'overtime') as prop
- [x] Accept dailyTotal as optional prop for display
- [x] Color code: Green (normal), Red (undertime), Orange (overtime)
- [x] Display text label and icon
- [x] Make responsive and accessible
- [x] Style with ShadCN Badge component
- [x] Write component tests

**Effort**: Small (2h)
**Complexity**: Low
**Assigned to**: Frontend Developer
**Depends on**: Task 2
**Blocks**: Task 7, Task 8

**Success Criteria**:
- [x] Component renders correctly
- [x] Colors match design (green/red/orange)
- [x] Text labels clear
- [x] Icons display correctly
- [x] Accessible (ARIA labels)
- [x] Responsive on mobile
- [x] Exports prop types

**Technical Notes**:
- Green (#22c55e) for normal
- Red (#ef4444) for undertime
- Orange (#f97316) for overtime
- Icons from lucide-react
- Use ShadCN Badge as base

**Status**: [x] Complete

---

### 7. [x] Build User Settings Page ✅

**Objective**: Create page for users to configure daily hour requirements

**Layer**: Frontend (Pages)

**Subtasks**:
- [x] Create `/app/routes/_auth.settings.hours.tsx` page
- [x] Fetch current user settings
- [x] Build form with min/max hour inputs
- [x] Add form validation
- [x] Show current values
- [x] Add reset to defaults button
- [x] Show loading and error states
- [x] Style with Tailwind and ShadCN

**Effort**: Small (2h)
**Complexity**: Low
**Assigned to**: Frontend Developer
**Depends on**: Task 5, Task 6
**Blocks**: None

**Success Criteria**:
- [x] Page loads with current settings
- [x] Form fields editable
- [x] Validation prevents invalid values
- [x] Submit updates settings
- [x] Success message shown
- [x] Error handling works
- [x] Page responsive on mobile

**Technical Notes**:
- Use React Hook Form
- Add number input fields
- Validate min < max
- Show helpful error messages
- Toast notification on save
- Display current/default values

**Status**: [x] Complete

---

### 8. [x] Build Time Status Filter Component

**Objective**: Create filter control for filtering worklogs by time status

**Layer**: Frontend (Components)

**Subtasks**:
- [x] Create `/app/components/worklog/time-filter.tsx` component
- [x] Add radio buttons or toggles: All, Undertime, Overtime
- [x] Accept onChange callback prop
- [x] Store selected filter in state
- [x] Add clear/reset button
- [x] Style with ShadCN components
- [x] Make accessible with ARIA labels
- [x] Responsive design

**Effort**: Small (2h)
**Complexity**: Low
**Assigned to**: Frontend Developer
**Depends on**: Task 6
**Blocks**: Task 9

**Success Criteria**:
- [x] Filter options render
- [x] Selection toggles correctly
- [x] onChange callback fires
- [x] Clear button resets to "All"
- [x] Accessible with keyboard
- [x] Responsive on mobile
- [x] ARIA labels present

**Technical Notes**:
- Use ShadCN Radio or Tabs component
- Store filter in parent state or URL param
- Call API with filter value
- Button states clear when on default

**Status**: [x] Complete

---

### 9. [x] Build Enhanced Worklog List with Filters ✅

**Objective**: Enhance worklog list page with time status display and filtering

**Layer**: Frontend (Pages)

**Subtasks**:
- [x] Modify `/app/routes/_auth.worklogs._index.tsx`
- [x] Add TimeFilter component above list
- [x] Pass selected filter to query
- [x] Display TimeStatusBadge on each day group
- [x] Show daily total hours
- [x] Update WorklogList to handle filter
- [x] Store filter in URL query param
- [x] Show filtered results

**Effort**: Medium (4h)
**Complexity**: Medium
**Assigned to**: Frontend Developer
**Depends on**: Task 4, Task 8, Phase 2 Task 9
**Blocks**: Task 10

**Success Criteria**:
- [x] Filter UI renders
- [x] Selecting filter updates list
- [x] Only matching days shown when filtered
- [x] Daily total hours displayed
- [x] Time status badge shows for each day
- [x] Filter state persists in URL
- [x] Empty state when filtered results empty
- [x] Performance acceptable

**Technical Notes**:
- Update query with filterBy parameter
- Store filter in URL: ?filter=undertime
- Refetch list when filter changes
- Show badge and total per day group
- Preserve pagination with filters

**Status**: [x] Complete

---

### 10. [x] Build Settings Navigation & Integration ✅

**Objective**: Add settings link to navigation and integrate into app

**Layer**: Frontend (Integration)

**Subtasks**:
- [x] Add Settings link to main navigation
- [x] Create settings layout page (`/app/routes/_auth.settings._index.tsx`)
- [x] Link to hours settings page
- [x] Test settings page load and save
- [x] Test filter functionality with real data
- [x] Verify time status calculations
- [x] Test permission enforcement (own settings only)
- [x] Verify TypeScript and linting pass

**Effort**: Small (2h)
**Complexity**: Low
**Assigned to**: Frontend Developer
**Depends on**: Task 7, Task 9
**Blocks**: None

**Success Criteria**:
- [x] Settings in navigation
- [x] Settings page accessible
- [x] Can navigate to hour settings
- [x] Save and load settings works
- [x] Filters work end-to-end
- [x] Time status displays correctly
- [x] TypeScript strict mode passes
- [x] Biome linting passes

**Technical Notes**:
- Create _auth.settings layout
- Test with multiple filter states
- Verify time status calculations
- Check mobile responsiveness
- Test error scenarios

**Status**: [x] Complete

---

## Task Dependency Graph

**Critical Path (must complete in order):**
```
Task 1 (Extend User Schema)
    ↓
Task 2 (Time Status Calculation)
    ↓
Task 3 (Daily Aggregation)
    ↓
Task 4 (Extend List Query)
    ↓
Task 6 (Time Status Badge)
    ↓
Task 8 (Time Filter)
    ↓
Task 9 (Enhanced List Page)
    ↓
Task 10 (Integration & Nav)
```

**Parallel Work:**
```
- Task 5 (Settings Mutation) can develop in parallel with Task 3
- Task 7 (Settings Page) depends on Task 5 but can develop after Task 6
- Task 6 (Badge) and Task 8 (Filter) can start once Task 2 is done
```

## Effort Summary

| Category | Tasks | Percentage |
|----------|-------|-----------|
| Backend | 1, 2, 3, 4, 5 | 45% |
| Frontend | 6, 7, 8, 9, 10 | 55% |
| **Total** | **10** | **100%** |

**Estimated Total Effort**: ~22 hours (Medium phase)

---

## Tracking Progress

Mark tasks complete as you finish by changing `[ ]` to `[x]`:

```markdown
### 1. [x] Extend User Schema with Time Settings ✅ Completed
```

Update progress at top: `**Progress**: 1/10 tasks complete (10%)`

---

## Notes

- Phase 3 builds on Phase 2 worklogs (complete CRUD first)
- Time status is computed at read-time, not stored
- Efficient aggregation critical for large worklog volumes
- Consider caching daily totals if performance issues arise
- Index strategy: (userId, date) composite index is key
- Frontend filtering can be optimized later if needed
- Real-time updates will come in future phases
