# Phase 4: Advanced Admin Features - Workforce Monitoring

## Objective

Provide comprehensive admin interface for monitoring all employee worklogs, enforcing hour compliance, managing user settings, and generating workforce insights.

## Scope

### Features

- View all worklogs across all users
- Advanced filtering (by user, date range, time status)
- Detailed user worklog history (admin view)
- Bulk user settings management
- Worklog reporting and analytics
- User activity timeline
- Hour compliance dashboard
- Admin audit trails (who changed what)
- User creation and role assignment
- User deletion and data cleanup

### Frontend

- Admin worklogs view page
- Advanced filters and search
- User worklog detail view (from admin)
- User management with inline editing
- Bulk actions toolbar
- Compliance dashboard
- Activity/audit logs view
- User detail view with history
- Export/report generation UI
- Mobile-friendly admin tables

### Backend

- Query for all worklogs with admin-only access
- Aggregated user time status across company
- User activity logs and audit trail
- Bulk user settings updates
- User creation endpoint (admin)
- User deletion endpoint (admin)
- Worklog reporting queries
- Compliance checking logic
- Activity logging for all admin actions

### Infrastructure

- Database indexes for efficient admin queries
- Audit logging system
- Activity event tracking
- Performance optimization for large datasets
- API rate limiting for protection
- Background jobs for batch operations

## Success Criteria

- [ ] Admin can view all worklogs across all users
- [ ] Admin can filter by user name (search or dropdown)
- [ ] Admin can filter by date range
- [ ] Admin can filter by time status (All/Undertime/Overtime)
- [ ] Admin can click on user to view detailed worklog history
- [ ] Admin can edit user's min/max hours inline
- [ ] Admin can view list of all users with sortable columns
- [ ] Admin can create new user account
- [ ] Admin can delete user and clean up data
- [ ] Admin can assign roles to users
- [ ] Compliance dashboard shows undertime/overtime trends
- [ ] Activity logs show all admin actions
- [ ] Audit trail shows who changed what and when
- [ ] Performance acceptable for 10K users (p99 < 2s)
- [ ] All admin-only pages require admin role
- [ ] Bulk operations complete without timeout

## Effort Breakdown

- Frontend: 35% (admin UI, filters, tables, dashboards)
- Backend: 45% (complex queries, aggregations, audit logging)
- Infrastructure/DevOps: 20% (indexes, performance tuning, logging)

## Dependencies

- Phase 1 (Authentication & User Management)
- Phase 2 (Worklog Core Features)
- Phase 3 (Time Status & Analytics)

## Risks & Mitigation

| Risk | Impact | Mitigation |
|------|--------|-----------|
| Admin queries too slow | Timeout errors | Implement caching, optimize indexes, use pagination |
| Accidental data deletion | Data loss | Require confirmation, log deletions, consider soft deletes |
| Audit log growth | Storage issues | Archive old logs, implement retention policy |
| Unauthorized admin access | Security breach | Strict role verification, audit suspicious activity |

## Team Assignment

- **Frontend Developer**: Admin UI, filters, tables, dashboards
- **Backend Developer**: Complex queries, aggregations, audit logging
- **Database Admin**: Index optimization, query performance
- **DevOps**: Rate limiting, background jobs, logging infrastructure

## Technical Notes

### Admin Worklog Query Response

```typescript
// Admin list all worklogs (grouped by date, then by user)
{
  date: string // ISO 8601
  entries: Array<{
    userId: string
    userName: string
    dailyTotalHours: number
    timeStatus: 'normal' | 'undertime' | 'overtime'
    logCount: number
    logsForDay: Worklog[]
  }>
}
```

### Audit Log Schema

```typescript
// AuditLogs table
{
  adminId: string // who made the change
  action: 'user_created' | 'user_deleted' | 'user_updated' | 'hours_changed' | 'role_assigned'
  targetUserId?: string // who was affected
  changeType: string // what was changed
  oldValue?: unknown
  newValue?: unknown
  timestamp: number
  ipAddress?: string // for security
}
```

### Key Files to Create/Modify

**Frontend:**
- `/app/routes/_auth.admin.worklogs._index.tsx` - All worklogs view
- `/app/routes/_auth.admin.worklogs.user.$userId.tsx` - User detail view
- `/app/routes/_auth.admin.users._index.tsx` - Enhanced user list
- `/app/routes/_auth.admin.users.$id.edit.tsx` - User edit dialog
- `/app/routes/_auth.admin.users.new.tsx` - Create user form
- `/app/routes/_auth.admin.compliance._index.tsx` - Compliance dashboard
- `/app/routes/_auth.admin.audit-logs.tsx` - Activity logs view
- `/app/components/admin/worklog-filters.tsx` - Advanced filter controls
- `/app/components/admin/compliance-chart.tsx` - Visualization
- `/app/components/admin/audit-log-table.tsx` - Activity logs
- `/app/components/admin/bulk-actions.tsx` - Bulk operations

**Backend:**
- Extend `/convex/worklogs.ts` - Add admin list query
- Extend `/convex/users.ts` - Add admin functions
- `/convex/audit.ts` - Create audit logging functions
- `/convex/admin.ts` - Admin-specific mutations and queries

### API Contracts

**Queries:**
- `admin.listAllWorklogs(limit?, offset?, filters?)` → { worklogs: [], hasMore: boolean }
  - filters: { userId?, dateRange?, timeStatus? }
- `admin.getUserWorklogHistory(userId, limit?, offset?)` → { worklogs: [], hasMore: boolean }
- `admin.getComplianceStats()` → { undertime: [], overtime: [], total: number }
- `admin.listAuditLogs(limit?, offset?, filter?)` → { logs: [], hasMore: boolean }
- `admin.getUserStats()` → { users: { total, undertime, overtime } }

**Mutations:**
- `admin.createUser(email, name, roles)` → User
- `admin.deleteUser(userId)` → { success: boolean }
- `admin.assignRole(userId, role)` → User
- `admin.updateMultipleUserSettings(updates)` → { success: boolean }

### Filtering Implementation

- Date range picker for worklog dates
- User search/autocomplete
- Time status checkboxes
- Sorting by name, date, status
- Filter state in URL query params
- Reset all filters button

### Compliance Dashboard

- Widget showing company undertime hours
- Widget showing company overtime hours
- Chart of time status distribution
- Top undertime users
- Trend over time (weekly/monthly)
- Alert for compliance issues

### Testing Strategy

**Functional Tests:**
- [ ] Admin views all worklogs correctly
- [ ] Admin can filter by user, date, status
- [ ] Admin can drill down to single user history
- [ ] Admin can edit user hours
- [ ] Admin can create new user
- [ ] Admin can delete user
- [ ] Admin can assign roles
- [ ] Audit log records all actions
- [ ] Compliance dashboard shows accurate stats

**Integration Tests:**
- [ ] Complex queries return correct data
- [ ] Pagination works with filters applied
- [ ] Audit log entries created for all actions
- [ ] User deletion cleans up related data

**Performance Tests:**
- [ ] All admin queries < 2s (p99)
- [ ] Admin list loads 10K users in < 5s with indexes
- [ ] Filters don't cause N+1 queries
- [ ] Bulk operations complete within timeout

### Security Considerations

- Verify admin role before returning any admin-only data
- Log all admin actions in audit trail
- Validate user IDs match existing users
- Implement deletion confirmation with role verification
- Rate limit admin endpoints to prevent abuse
- Don't expose sensitive data in audit logs

## Deliverables

1. Complete admin worklog monitoring interface
2. Advanced filtering and search capabilities
3. User management interface
4. Compliance dashboard
5. Audit trail and activity logs
6. Bulk operations support
7. Performance-optimized queries with proper indexes

## Notes

- Phase 4 builds on Phases 1-3 to provide admin visibility
- Focus on performance with proper indexes
- Implement audit logging for compliance and security
- Consider future expansion to reports/exports
- Compliance dashboard provides actionable insights
