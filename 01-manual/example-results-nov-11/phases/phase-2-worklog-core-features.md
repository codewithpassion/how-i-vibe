# Phase 2: Worklog Core Features - CRUD Operations

## Objective

Enable users to create, read, update, and delete their work logs with proper validation, error handling, and responsive UI components.

## Scope

### Features

- Create new worklog entries with validation
- List user's worklogs with pagination
- Update existing worklog entries
- Delete worklog entries with confirmation
- Worklog grouping by date
- Form validation (client and server)
- Optimistic updates for better UX
- Loading and error states

### Frontend

- Create worklog form page (`/worklogs/new`)
- Worklog list page (`/worklogs`)
- Worklog edit modal or separate edit page
- Worklog delete confirmation dialog
- WorklogCard component for individual entries
- WorklogForm component (reusable for create/edit)
- TimeStatusBadge component (basic, without status calculation)
- Loading spinner component
- Empty state component
- Toast notifications for user feedback
- Form validation using Zod

### Backend

- Worklog schema in Convex (userId, date, workedHours, taskId, description, timestamps)
- Create worklog mutation with validation
- Update worklog mutation
- Delete worklog mutation
- List user worklogs query with pagination
- Cursor-based pagination implementation
- Server-side validation
- Authorization checks (users can only access their own logs)

### Infrastructure

- Database indexes for userId and date
- Error logging for mutations
- Input validation schemas
- Rate limiting consideration

## Success Criteria

- [ ] User can create worklog with date, hours, optional task/description
- [ ] Form validates required fields and shows errors
- [ ] Form rejects invalid input (hours > 24, future dates, non-numeric)
- [ ] Created worklog appears in user's list immediately
- [ ] User can edit their own worklog
- [ ] Edit updates reflected without page refresh (optimistic update)
- [ ] User can delete worklog with confirmation dialog
- [ ] Deleted worklog removed from list
- [ ] User cannot edit/delete other users' worklogs
- [ ] Pagination works with "Load More" button
- [ ] Worklogs grouped by date, most recent first
- [ ] All form fields validate on client and server
- [ ] Loading states display during API calls
- [ ] Error messages are helpful and specific

## Effort Breakdown

- Frontend: 50% (forms, list display, components, validation UI)
- Backend: 35% (Convex schema, mutations, queries, pagination)
- Infrastructure/DevOps: 15% (database indexes, error handling)

## Dependencies

- Phase 1 (Authentication & User Management) - requires authenticated user

## Risks & Mitigation

| Risk | Impact | Mitigation |
|------|--------|-----------|
| Concurrent edit conflicts | Data inconsistency | Implement optimistic locking or show conflict warning |
| Large datasets slow down | Performance degradation | Use cursor-based pagination with proper indexes |
| Form validation mismatch | User confusion | Implement identical client/server validation logic |
| Date parsing errors | Invalid data stored | Use ISO 8601 format, validate dates strictly |

## Team Assignment

- **Frontend Developer**: Form components, list display, pagination UI, validation
- **Backend Developer**: Convex schema, mutations, queries, pagination logic
- **QA**: Test validation, edge cases, concurrent scenarios

## Technical Notes

### Database Schema

```typescript
// Worklogs table
{
  userId: string (indexed, foreign key to Users)
  date: string // ISO 8601 format YYYY-MM-DD (indexed)
  workedHours: number // positive decimal, e.g., 8.5
  taskId?: string // optional reference to task/project
  description?: string // optional, max 500 chars
  createdAt: number
  updatedAt: number
}

// Indexes needed
- on userId for listing user's logs
- on (userId, date) for efficient grouping
```

### Form Validation Schema (Zod)

```typescript
const worklogFormSchema = z.object({
  date: z.string().date().refine(d => new Date(d) <= new Date()),
  workedHours: z.number().positive().max(24),
  taskId: z.string().optional(),
  description: z.string().max(500).optional(),
});
```

### Key Files to Create/Modify

**Frontend:**
- `/app/routes/_auth.worklogs._index.tsx` - List worklogs
- `/app/routes/_auth.worklogs.new.tsx` - Create worklog
- `/app/routes/_auth.worklogs.$id.edit.tsx` - Edit worklog (optional)
- `/app/components/worklog/worklog-form.tsx` - Reusable form
- `/app/components/worklog/worklog-list.tsx` - List component
- `/app/components/worklog/worklog-card.tsx` - Individual entry
- `/app/components/worklog/time-status-badge.tsx` - Status indicator (basic)
- `/app/hooks/use-worklogs.ts` - Convex mutations/queries hook
- `/app/lib/validation.ts` - Zod schemas

**Backend:**
- Extend `/convex/schema.ts` - Add Worklogs table
- `/convex/worklogs.ts` - Create, update, delete, list mutations/queries
- Extend `/convex/auth.ts` - Add worklog authorization checks

### API Contracts

**Mutations:**
- `worklogs.createWorklog(date, workedHours, taskId?, description?)` → Worklog
- `worklogs.updateWorklog(logId, workedHours?, taskId?, description?)` → Worklog
- `worklogs.deleteWorklog(logId)` → { success: boolean }

**Queries:**
- `worklogs.listUserWorklogs(limit?, offset?, sortOrder?)` → { worklogs: [], hasMore: boolean, nextOffset?: number }

### Pagination Implementation

Use cursor-based pagination:
- Fetch limit+1 records to determine if more exist
- Return cursor (encoded timestamp or ID) with results
- Client sends cursor with next request

### Component Patterns

**WorklogForm:**
- Props: `{ onSubmit, defaultValues?, isLoading? }`
- Handles both create and edit modes
- Client-side validation with Zod
- Shows field errors
- Disables submit while loading

**WorklogList:**
- Props: None (fetches own data via hook)
- State: worklogs[], loading, error, pagination
- Renders grouped by date
- Shows loading skeleton while fetching

### Testing Strategy

**Functional Tests:**
- [ ] Create worklog with valid data
- [ ] Form validation prevents invalid submission
- [ ] Created worklog appears in list
- [ ] Edit updates the entry
- [ ] Delete removes from list with confirmation
- [ ] Cannot edit/delete other users' logs
- [ ] Pagination loads more entries
- [ ] Optimistic update shows immediately
- [ ] Error handling shows helpful message

**Integration Tests:**
- [ ] Convex mutations enforce user authorization
- [ ] Server-side validation matches client
- [ ] Pagination cursor works correctly
- [ ] Worklogs grouped by date on frontend

### Security Considerations

- Verify userId matches authenticated user in mutations
- Validate all inputs server-side
- Enforce data ownership before returning or modifying
- Log mutation errors for debugging

## Deliverables

1. Complete CRUD interface for worklogs
2. Form with client and server validation
3. Pagination with cursor-based implementation
4. Error handling and loading states
5. Type-safe Convex functions
6. Responsive layout with shadcn components

## Notes

- This phase establishes the core worklog functionality
- Time status calculation (undertime/overtime) deferred to Phase 3
- Focus on stable CRUD operations and validation
- Ensure pagination is cursor-based for scalability
- Optimistic updates improve perceived performance
