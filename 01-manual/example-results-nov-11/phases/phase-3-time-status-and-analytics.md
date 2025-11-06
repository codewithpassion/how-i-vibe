# Phase 3: Time Status & Analytics - Automatic Tracking

## Objective

Implement automatic detection of undertime and overtime hours with visual indicators, enable users to configure their daily hour requirements, and provide analytics for work patterns.

## Scope

### Features

- Automatic daily total hours calculation
- Undertime/overtime detection based on user settings
- Daily min/max hours configuration per user
- Time status filtering (All, Undertime, Overtime)
- Time status badges with visual indicators
- Daily totals display on worklog list
- Summary statistics and trends
- Hour configuration for admin users
- Responsive filtering UI

### Frontend

- User settings page for daily min/max hours
- Time status badges (Normal/UT/OT) with colors
- Filter controls for time status
- Daily totals display in worklog list
- Summary statistics card
- Min/max hours editing interface
- Loading and error states for settings
- Visual indicators for at-risk days

### Backend

- Extend user schema with dailyMinHours and dailyMaxHours
- Query to get user settings
- Mutation to update user settings
- Query to list worklogs with computed daily totals
- Time status calculation logic (normal/undertime/overtime)
- Daily totals aggregation
- Filtering by time status
- Admin query to view all users' time statuses

### Infrastructure

- Database indexes for efficient time status queries
- Caching strategy for computed fields
- Performance optimization for large datasets
- Real-time updates when settings change

## Success Criteria

- [ ] User can set daily min/max hours in settings
- [ ] Daily total hours calculated correctly (sum of logs for day)
- [ ] Time status badge shows "Normal" when within min/max
- [ ] Time status badge shows "UT" (Undertime) when below minimum
- [ ] Time status badge shows "OT" (Overtime) when above maximum
- [ ] Time status updates immediately when worklog added/edited/deleted
- [ ] User can filter worklogs by time status
- [ ] Filter state persists in URL query param
- [ ] Admin can see time status for all users
- [ ] Settings changes propagate to all displayed worklogs
- [ ] Performance acceptable with 10K users (p99 < 2s)
- [ ] Default values (8 min, 10 max) set for new users

## Effort Breakdown

- Frontend: 40% (filter UI, badges, settings page, styling)
- Backend: 45% (aggregation logic, indexes, queries)
- Infrastructure/DevOps: 15% (performance optimization, monitoring)

## Dependencies

- Phase 1 (Authentication & User Management)
- Phase 2 (Worklog Core Features)

## Risks & Mitigation

| Risk | Impact | Mitigation |
|------|--------|-----------|
| Slow aggregation queries | Performance degradation | Add database indexes, consider pre-computed summaries |
| Invalid min/max values | Incorrect time status | Validate min < max, set defaults, handle edge cases |
| Stale cached values | Inconsistent display | Invalidate cache on updates, use real-time subscriptions |
| Time zone issues | Wrong date calculations | Use ISO 8601 dates, handle user timezone separately |

## Team Assignment

- **Frontend Developer**: Filter UI, time status badges, settings page
- **Backend Developer**: Aggregation logic, queries, calculation functions
- **Database Admin**: Index optimization, query performance tuning

## Technical Notes

### Extended User Schema

```typescript
// Users table additions
{
  // ... existing fields
  dailyMinHours: number // e.g., 8
  dailyMaxHours: number // e.g., 10
  timezone?: string // optional, for future use
}
```

### Time Status Calculation

```typescript
// Computed on query, not stored
function calculateTimeStatus(
  dailyTotalHours: number,
  dailyMinHours: number,
  dailyMaxHours: number
): 'undertime' | 'normal' | 'overtime' {
  if (dailyTotalHours < dailyMinHours) return 'undertime';
  if (dailyTotalHours > dailyMaxHours) return 'overtime';
  return 'normal';
}
```

### Query Response Structure

```typescript
// When listing worklogs with time status
{
  date: string // ISO 8601
  userId: string
  dailyTotalHours: number // sum of workedHours for day
  timeStatus: 'normal' | 'undertime' | 'overtime'
  logCount: number // number of entries for that day
  logsForDay: Worklog[]
}
```

### Key Files to Create/Modify

**Frontend:**
- `/app/routes/_auth.settings.tsx` - User settings page
- `/app/routes/_auth.settings.hours.tsx` - Hour configuration
- `/app/components/worklog/time-status-badge.tsx` - Enhanced status badge
- `/app/components/worklog/time-filter.tsx` - Filter control
- `/app/components/worklog/worklog-list.tsx` - Enhanced with time status
- `/app/hooks/use-user-settings.ts` - Settings hook
- `/app/lib/time-status.ts` - Time status calculation helpers

**Backend:**
- Extend `/convex/schema.ts` - Add dailyMinHours, dailyMaxHours to users
- `/convex/worklogs.ts` - Add time status calculation to list query
- Extend `/convex/users.ts` - Add updateUserSettings mutation

### API Contracts

**Mutations:**
- `users.updateUserSettings(dailyMinHours, dailyMaxHours)` → User
- `admin.updateUserSettings(userId, dailyMinHours, dailyMaxHours)` → User (admin only)

**Queries:**
- `worklogs.listUserWorklogs(filterBy?, limit?, offset?)` → { worklogs: [], hasMore: boolean }
  - filterBy: 'all' | 'undertime' | 'overtime'
  - Returns worklogs grouped by date with computed time status
- `users.getUserSettings()` → { dailyMinHours, dailyMaxHours, timezone? }

### Time Status Badge Component

```typescript
interface TimeStatusBadgeProps {
  status: 'normal' | 'undertime' | 'overtime';
  dailyTotal: number;
  minHours: number;
  maxHours: number;
}

// Renders with semantic colors:
// Normal: Green checkmark or badge
// Undertime: Red "UT" badge with value
// Overtime: Orange "OT" badge with value
```

### Filtering Implementation

- Client stores filter state in URL query params: `?filter=undertime`
- Server-side filtering: compute time status, filter results
- Filter options: All logs, Undertime days only, Overtime days only
- Reset button clears filters

### Testing Strategy

**Functional Tests:**
- [ ] User can set daily min/max hours
- [ ] Time status badge displays correct indicator
- [ ] Undertime detected when total < min
- [ ] Overtime detected when total > max
- [ ] Normal status when within range
- [ ] Filter "Undertime only" shows only undertime days
- [ ] Filter "Overtime only" shows only overtime days
- [ ] Time status updates after worklog change
- [ ] Settings changes apply to all displayed data

**Integration Tests:**
- [ ] Daily totals calculated correctly with multiple entries per day
- [ ] Aggregation query performs well with large datasets
- [ ] Filter query returns correct subset
- [ ] Admin can view all users' time statuses

**Performance Tests:**
- [ ] List query with time status calc < 1s (p99)
- [ ] Filtering doesn't cause N+1 queries
- [ ] Settings update completes < 500ms

### Security Considerations

- Validate min/max values (min >= 0, max > min)
- Users can only update their own settings
- Admin can update any user's settings
- Enforce authorization in all update mutations

## Deliverables

1. Time status calculation and display system
2. User settings interface for hour configuration
3. Filtering UI with multiple filter options
4. Efficient aggregation queries with proper indexes
5. Visual indicators for under/overtime hours
6. Dashboard summary statistics

## Notes

- Time status is computed on-read, not stored (keeps data fresh)
- Default values (8-10 hours) established for new users
- Aggregation happens at database level for efficiency
- Consider real-time updates for instant feedback
- Foundation for future reporting and analytics features
