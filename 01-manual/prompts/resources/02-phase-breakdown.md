# Phases - Break Into Development Phases

Breaks your PRD into full-stack development phases. Each phase delivers a working product with core functionality, using a full-stack approach (frontend + backend + infrastructure together).

## Process

The Phase Breakdown process will:

1. **Analyze PRD & Tech Stack**
   - Review all features and requirements
   - Understand technical dependencies
   - Identify critical path items
   - Map feature interactions

2. **Identify Phase Boundaries**
   - Core MVP features in Phase 1
   - Progressive enhancement in subsequent phases
   - Each phase delivers working product
   - Balance between scope and delivery speed

3. **Create Full-Stack Phases**
   - Frontend components for the phase
   - Backend APIs and logic
   - Database schema updates
   - Deployment & infrastructure
   - Testing & quality assurance

4. **Generate Phase Documents**
   - `phases/phase-1.md` through `phases/phase-N.md`
   - Clear scope and deliverables
   - Success criteria for each phase
   - Dependencies and ordering
   - Effort estimation

## Output

- `project-docs/phases/` directory with markdown for each phase
- `project-docs/phases-overview.md` - Phase summary and dependencies

! MAKE SURE YOU WRITE THE PHASE FILES !
! DO NOT USE THE 'task-decomposition agent'!
! DO NOT OUTPUT 'Duration' or 'Timeline' or similar estimations in the phase documents !

## Example Phase Structure

```
# Phase 1: MVP - Core Product Launch

## Objective
Deliver minimum viable product with core functionality that provides immediate value to primary user persona.

## Scope
### Features
- User authentication (email/password)
- Create and manage profiles
- Core CRUD operations
- Basic dashboard

### Frontend
- Login/signup pages
- User dashboard
- Profile management
- Navigation structure

### Backend
- User authentication API
- Profile management endpoints
- Database schema v1
- Basic error handling

### Infrastructure
- Deployment pipeline
- Database setup
- Environment configuration
- Logging & monitoring basics

## Success Criteria
- [ ] All features are functional
- [ ] Mobile responsive design
- [ ] API performance < 200ms
- [ ] 95% test coverage for critical paths
- [ ] Zero critical security issues
- [ ] Documentation complete

## Effort Breakdown
- Frontend: 40%
- Backend: 40%
- Infrastructure/DevOps: 20%

## Dependencies
- None (Phase 1 is independent)

## Risks & Mitigation
[Risk analysis]

## Team Assignment
[Who works on what]

---

# Phase 2: Enhanced Core Features

[Similar structure for Phase 2, building on Phase 1]
```

## Phase Design Principles

1. **Each Phase is Self-Contained Full-Stack**
   - Frontend, backend, and infrastructure work together
   - Not: "Backend phase, then frontend phase"
   - But: "Authentication phase (includes login UI + auth API)"

2. **Progressive Value Delivery**
   - Phase 1: Minimum viable product
   - Phase 2: Critical enhancements
   - Phase 3+: Advanced features and optimizations

3. **Minimal Dependencies**
   - Phases ordered by logical progression
   - Avoid circular dependencies
   - Can run phases in parallel if independent

4. **Clear Scope**
   - Know what's IN and OUT of each phase
   - Time-boxed delivery
   - Defined success metrics


## Typical Phase Structure

- **Phase 1**: MVP with core features
- **Phase 2**: Critical missing features
- **Phase 3**: Advanced features
- **Phase 4+**: Polish, optimization, analytics

