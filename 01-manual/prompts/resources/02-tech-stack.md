# Tech Stack - Technology Stack Documentation

Recommends and documents the technology stack based on your PRD. This command invokes the Tech Stack Analyzer Agent to create detailed technology decisions.

## Process

The Tech Stack Analyzer will:

1. **Analyze Requirements**
   - Review PRD for technical needs
   - Identify scalability requirements
   - Check for integration needs
   - Note performance requirements

2. **Analyze Existing Stack**
   - Map current technologies
   - Identify what can be reused
   - Flag breaking changes needed

3. **Recommend Stack**
   - Frontend framework & libraries
   - Backend framework & runtime
   - Database & cache solutions
   - Infrastructure & deployment
   - Development tools & testing

4. **Document Decisions**
   - Rationale for each choice
   - Trade-offs vs alternatives
   - Compatibility matrix
   - Migration path (if updating existing)

## Output

- `tech-stack.md` - Complete technology stack documentation

## Example Tech Stack Document

```
# Technology Stack

## Architecture Overview
[High-level architecture diagram/description]

## Frontend Stack
- Framework: React 19 with TypeScript
- Build Tool: Vite
- UI Component Library: ShadCN/UI
- Styling: TailwindCSS
- State Management: TanStack Query + Zustand

## Backend Stack
- Runtime: Node.js/Bun
- Framework: Hono
- Database: PostgreSQL
- Cache: Redis
- Authentication: Clerk

## Infrastructure
- Hosting: Cloudflare Workers
- Database Hosting: Convex / Supabase
- CDN: Cloudflare
- Email: Resend / SendGrid

## Development Tools
- Package Manager: Bun
- Linter: Biome
- Type Checking: TypeScript Strict Mode
- Testing: Vitest + Testing Library
- CI/CD: GitHub Actions

## Decision Rationale
[Detailed rationale for each choice]

## Trade-offs & Alternatives
[Document what else was considered and why it was rejected]
```

