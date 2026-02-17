# AGENTS.md - Fatebook Development Guide

This file provides context for AI agents operating in the Fatebook codebase.

## Core Principles

### 1. Clarify Requirements Before Acting

- Ask clarifying questions when uncertain, confirm with user
- **Important**: Don't dive into details immediately; explore from decisive high-level to details
- Ask "what to do" first, then "how to do"

### 2. TDD Approach - Test First

- Design test cases before writing code
- Before starting work, check if corresponding test cases exist
- If no test cases exist, design them and confirm with user
- Run test regression after completing code

### 3. Experience Accumulation

- Summarize and collect effective experiences during work
- Document cases in `knowledge/` directory for accumulation
- Prefer categorization and consolidation over excessive unrelated content
- Important experiences can also sync to AGENTS.md

### 4. Documentation Language Convention

- **Authoritative documents** (like AGENTS.md) keep single language only
- **Explanatory documents** (like README.md) may have bilingual versions
- AI agent working language: follow user preference

### 5. Task Tracking with todo.md

- Before starting any task, create a `todo.md` file in the project root
- Break down the task into atomic subtasks
- Check off each subtask as it is completed with `[x]`
- After all tasks are complete, append git commit message suggestions at the end of `todo.md`
- Delete the `todo.md` file after committing (or keep for reference if needed)

```markdown
# Todo List

## Task: [Task Description]

- [ ] Subtask 1
- [ ] Subtask 2
- [ ] Subtask 3

---

## Git Commit Message Suggestions

// ... commit message suggestions here ...
```

### 6. Strategic Planning Reference

- Engineering knowledge is stored in `knowledge/` directory
- Strategic planning document is in `knowledge/planning.md` - answers "what should we do next?" based on project goals and current state
- **todo.md** in project root: tactical task breakdown for current session
- **knowledge/planning.md**: strategic phase planning for project direction

When asked "what should we do next based on current strategic planning?", consult the planning document and provide actionable options with reasoning.

---

## Knowledge Management

Engineering knowledge documents are stored in `knowledge/` directory for future reference and reuse.

### Directory Structure

```
knowledge/
├── patterns/          # Reusable code patterns and solutions
├── decisions/         # Architecture and design decisions (ADRs)
├── planning.md        # Strategic phase planning document
├── troubleshooting/   # Common issues and resolutions
└── guides/            # How-to guides and tutorials
```

### Document Guidelines

- Each document should focus on a specific topic
- Include context: why this approach was chosen, trade-offs considered
- Reference related code files when possible
- Update documents when patterns change

### Experience Accumulation

During each work session:

- Summarize and collect effective experiences promptly
- Document cases in `knowledge/` directory for accumulation
- Prefer categorization and consolidation over excessive content
- Important experiences can also sync to AGENTS.md

---

## Project Overview

Fatebook (fatebook.io) is a prediction tracking platform with Slack/Discord integrations and a Chrome extension. Built with Next.js, tRPC, Prisma, and Tailwind CSS.

## Tech Stack

- **Framework**: Next.js 14 (Pages Router)
- **API**: tRPC + Zod validation
- **Database**: PostgreSQL + Prisma ORM
- **Styling**: Tailwind CSS + DaisyUI
- **Testing**: Jest + React Testing Library
- **Auth**: NextAuth.js (Google OAuth)
- **Integrations**: Slack API, Discord API

---

## Commands

### Development

```bash
npm run dev          # Start dev server on https://localhost:3005
npm run build        # Production build
npm run start        # Start production server
npm run studio       # Open Prisma Studio
```

### Linting & Formatting

```bash
npm run lint         # ESLint on lib/, pages/, components/
npm run format       # Prettier write all files
```

### Testing

```bash
npm run test                           # Run all tests
npm run test -- --coverage             # Run with coverage report
npm run test -- path/to/test.tsx      # Run single test file
npm run test -- --testPathPattern=foo  # Run tests matching pattern
```

---

## Code Style Guidelines

### TypeScript

- **Strict mode enabled** - No `any` types, no `@ts-ignore`
- **Strict null checks enabled** - Always handle null/undefined
- **Always await async functions** - `require-await` rule enforced

### Naming Conventions

| Element          | Format                          | Example                      |
| ---------------- | ------------------------------- | ---------------------------- |
| Variables        | camelCase                       | `userId`, `isActive`         |
| Constants        | UPPER_CASE                      | `MAX_RETRY_COUNT`            |
| Functions        | camelCase                       | `getUserById()`              |
| React Components | PascalCase                      | `Navbar.tsx`, `QuestionCard` |
| Interfaces       | PascalCase                      | `UserProfile`, `Question`    |
| Types            | PascalCase                      | `ForecastResult`             |
| Enums            | PascalCase (members UPPER_CASE) | `QuestionStatus`             |
| Type Parameters  | PascalCase                      | `T extends string`           |

### Imports

- **Order**: External libs → Internal libs → Components/UI → Styles
- **Use path aliases**: Avoid relative paths crossing directories
- **React imports**: Use `import { useState } from "react"` (not default)

```typescript
// Good
import { useState } from "react"
import { api } from "../lib/web/trpc"
import clsx from "clsx"
import Navbar from "../components/Navbar"

// Avoid
import React, { useState } from "react" // React not needed in React 18+
import { useState } from "../../../lib/utils" // Deep relative paths
```

### Error Handling

- **Never suppress errors** - No `as any`, `@ts-ignore`, `@ts-expect-error`
- **Handle promises properly** - Use `await` or `.catch()`, never float promises
- **Use Zod for validation** - All API inputs validated with Zod schemas

```typescript
// Good
const user = await getUser(id)
if (!user) throw new Error("User not found")

// Bad
const user = getUser(id) // Floating promise
const result = data as any // Type suppression
```

### Component Patterns

- Use **functional components** with hooks
- **Export named components**: `export function Navbar() {...}`
- **Co-locate tests**: `QuestionCard.test.tsx` next to `QuestionCard.tsx`
- **Co-locate types**: Define types in same file or `lib/types.ts`

### Tailwind CSS

- Use **DaisyUI components** when available (buttons, modals, dropdowns)
- Use semantic class names with DaisyUI tokens:
  - `btn btn-primary`, `btn btn-sm`
  - `modal`, `dropdown`, `card`
- Use `clsx` or `cva` for conditional classes

```tsx
<button className={clsx("btn btn-primary", isLoading && "loading")}>
  Submit
</button>
```

---

## Project Structure

```
fatebook/
├── components/          # React components
│   └── ui/             # Reusable UI components
├── lib/
│   ├── web/            # tRPC routers and web utilities
│   ├── blocks-designs/ # Slack block kit designs
│   └── interactive_handlers/  # Slack interaction handlers
├── pages/              # Next.js pages (API routes + pages)
│   ├── api/            # API endpoints
│   └── *.tsx          # Page routes
├── chrome-extension/   # Chrome extension source
└── prisma/            # Database schema
```

---

## Database

- **Prisma** for ORM - schema in `prisma/schema.prisma`
- Run `npx prisma generate` after schema changes
- Use `npx prisma studio` to explore data

---

## Testing Guidelines

- Tests go in `*.test.tsx` or `*.test.ts` files
- Import `@testing-library/jest-dom` for custom matchers
- Use `jest.useFakeTimers()` for time-dependent tests

```typescript
import "@testing-library/jest-dom"
import { render, screen } from "@testing-library/react"
import { MyComponent } from "../MyComponent"

describe("MyComponent", () => {
  it("renders correctly", () => {
    render(<MyComponent />)
    expect(screen.getByText("Hello")).toBeInTheDocument()
  })
})
```

---

## VS Code / Debugging

- Use the **"Next.js: debug fatebook full stack"** launch config for debugging
- Set breakpoints in both frontend and API routes
- Works in Cursor and VS Code

---

## Common Patterns

### tRPC Router

```typescript
// lib/web/question_router.ts
import { z } from "zod"
import { router, protectedProcedure } from "./trpc"

export const questionRouter = router({
  create: protectedProcedure
    .input(z.object({ question: z.string(), resolutionDate: z.date() }))
    .mutation(async ({ input, ctx }) => {
      return ctx.prisma.question.create({ ... })
    }),
})
```

### React Component with tRPC

```typescript
import { api } from "../lib/web/trpc"

export function MyComponent() {
  const questions = api.question.list.useQuery()

  if (questions.isLoading) return <span>Loading...</span>

  return (
    <ul>
      {questions.data?.map(q => <li key={q.id}>{q.text}</li>)}
    </ul>
  )
}
```

---

## Git Commit Guidelines

### Commit Message Format

Use [Conventional Commits](https://www.conventionalcommits.org/) standard format:

```
<type>(<scope>): <subject>

[optional body]

[optional footer]
```

### Type

| Type       | Description                               |
| ---------- | ----------------------------------------- |
| `feat`     | New feature                               |
| `fix`      | Bug fix                                   |
| `docs`     | Documentation only                        |
| `style`    | Code style changes (no functionality)     |
| `refactor` | Code refactoring (not new feature or fix) |
| `test`     | Adding or updating tests                  |
| `chore`    | Build process, tooling changes            |

### Examples

```
feat(auth): add Google OAuth login support
fix(api): resolve null pointer in question router
docs(readme): update installation instructions
refactor(components): extract shared Button component
```

### Important

**At the end of each conversation, provide git commit message suggestions for the current work.**

Suggestions should include:

- Commit message in the above format
- If multiple types of changes, provide multiple commit message suggestions

---

## Notes

- This is a **Next.js Pages Router** project (not App Router)
- The dev server runs on **port 3005** with SSL proxy to port 3000
- ESLint is configured to require explicit return types on complex functions
- Prettier uses **no semicolons**
