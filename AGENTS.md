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

### 6. Progress Tracking with progress.txt

- Create `progress.txt` in project root to prevent context rot
- Record completed work state and lessons learned after each session
- **Write to file on each iteration**, read at start of new context
- Format: timestamp, completed tasks, key decisions, lessons learned, next steps

```markdown
# Progress Record

## Session: [Date/Time]

### Completed

- [Task 1]
- [Task 2]

### Key Decisions

- Decision: [What was decided]
- Rationale: [Why]

### Lessons Learned

- [Lesson 1]
- [Lesson 2]

### Next Steps

- [Next task]
```

### 7. Strategic Planning Reference

- Engineering knowledge is stored in `knowledge/` directory
- Strategic planning document is in `knowledge/planning.md` - answers "what should we do next?" based on project goals and current state
- **todo.md** in project root: tactical task breakdown for current session
- **knowledge/planning.md**: strategic phase planning for project direction

When asked "what should we do next based on current strategic planning?", consult the planning document and provide actionable options with reasoning.

### 8. Docker Sandbox Isolation (Running OpenCode)

Running OpenCode inside a Docker container is a critical security measure. It prevents AI from accidentally deleting host files or executing dangerous commands during automated iterations.

#### Why This Matters

- **Host Protection**: Isolates AI operations from the host filesystem
- **Damage Control**: Even if AI goes rogue, it only affects the container
- **Clean Environment**: Each task starts with a fresh, predictable environment

#### Docker Configuration

Create `docker-compose.yml` in project root:

```yaml
services:
  opencode-sandbox:
    image: node:18-bullseye
    volumes:
      - .:/workspace
    working_dir: /workspace
    user: node
    command: tail -f /dev/null
```

#### Security Best Practices

| Setting   | Recommendation                  | Purpose                     |
| --------- | ------------------------------- | --------------------------- |
| User      | Run as non-root (`user: node`)  | Limit container privileges  |
| Read-only | Use `:ro` for sensitive dirs    | Prevent accidental writes   |
| Network   | `network_mode: none` if offline | Prevent data exfiltration   |
| Resources | `--memory=2g --cpus=1.5`        | Prevent resource exhaustion |

#### Verification

```bash
# Confirm running inside container
ls /.dockerenv

# Check current user
whoami
```

#### Alternative: OrbStack

If using OrbStack (macOS), configure Docker sandbox through its settings for easier management.

---

### 9. Parallel Development with Worktrees

Single-threaded AI coding cannot meet demands for parallel feature development. Use Git worktrees to create isolated "containers within containers" - each worktree runs a dedicated OpenCode instance for independent, parallel development.

#### Core Principles

- **Isolated Execution**: Each worktree operates independently with its own branch, port, and database
- **Parallel Progress**: Multiple features can be developed simultaneously without interference
- **Sequential Integration**: Features merge one-by-one to main after passing tests and rebasing

#### Worktree Setup for Parallel Development

```bash
# Create worktree with dedicated branch
git worktree add -b task/xxx ../fatebook-worktrees/task-xxx

# Create isolated data directory
mkdir -p ../fatebook-worktrees/task-xxx/data

# Symlink shared configuration (NOT progress.txt - each worktree has its own)
ln -s ../../fatebook/data/dev-tasks.json ../fatebook-worktrees/task-xxx/data/
ln -s ../../fatebook/api-key.json ../fatebook-worktrees/task-xxx/data/

# Symlink node_modules for faster startup
ln -s ../fatebook/node_modules ../fatebook-worktrees/task-xxx/

# Dedicated port: base 3005 + task_index
# task-001 → 3005, task-002 → 3006, etc.
```

#### Conflict Resolution Principles

When multiple worktrees modify the same files, conflicts are inevitable. Follow these principles:

##### Rebase Failure Handling

```
If "unstaged changes" error:
  → Commit or stash current changes first

If merge conflicts:
  1. View conflicting files: git status
  2. Read conflict markers to understand both sides' intent
  3. Manually resolve (keep correct code, reject incompatible changes)
  4. git add <resolved-files>
  5. git rebase --continue
  6. Repeat until rebase completes
```

##### Test Failure Handling

```
When tests fail:
  1. Run: npm test
  2. Analyze error messages - identify root cause
  3. Fix the bug in your code
  4. Re-run tests until all pass
  5. Commit fix: git commit -m "fix: ..."
```

**Critical Rule**: Never give up. When rebase or tests fail, you MUST resolve before proceeding. Do not mark task as failed to bypass issues.

##### Conflict Prevention Strategies

- **Assign clear file ownership**: Different worktrees should modify different modules when possible
- **Coordinate before starting**: Check dev-tasks.json to understand what others are working on
- **Communicate intent**: If you must modify shared files (e.g., AGENTS.md, shared types), coordinate with other agents

##### Worktree Cleanup Best Practices

> **Note**: The `worktree-cleanup` skill is now installed globally. Use it for automated cleanup.

- **Clean up immediately** after task completion to avoid stale worktrees
- **Never leave uncommitted changes** in worktrees - always commit before cleanup
- **Use `--force` when needed** - safe if all changes were committed
- **Verify with `git worktree list`** - confirm only main worktree remains
- **If session crashes** - manually clean up in next session using the commands above

---

## Task Lifecycle (Automated)

This section describes the automated task lifecycle for AI agents. When properly configured, the agent can automatically pick up tasks, implement them in isolated workspaces, commit changes, and merge to main.

### Task Data Structure (`data/dev-tasks.json`)

Create `data/dev-tasks.json` in project root to manage tasks:

```json
[
  {
    "id": "task-001",
    "title": "Add user profile feature",
    "description": "Implement user profile page with avatar upload",
    "status": "pending",
    "priority": "high",
    "assignee": "agent",
    "createdAt": "2026-02-01T00:00:00Z"
  }
]
```

### Workflow Steps

#### Step 1: Pick Up Task

Atomic operation - read from `data/dev-tasks.json` and select a task with `status: "pending"`. Mark it as `in_progress` before starting.

#### Step 2: Create Isolated Workspace

Follow the worktree setup instructions in **Section 8: Parallel Development with Worktrees** above. Each task gets its own isolated Git worktree with dedicated branch and port.

#### Step 3: Implement Feature

In the isolated worktree:

- Create `todo.md` for task breakdown
- Implement the feature following TDD approach
- Run tests locally before proceeding

#### Step 4: Commit Changes

```bash
git add .
git commit -m "<type>(<scope>): <subject>"
```

Follow [Conventional Commits](#git-commit-guidelines) format.

#### Step 5: Merge + Test

```bash
# Fetch and merge latest main
git fetch origin
git merge origin/main

# Run tests
npm test

# If any step fails, rollback to Step 3 and fix
```

#### Step 6: Auto-Merge to Main

```bash
# Rebase onto main
git fetch origin main
git rebase origin/main

# If rebase fails: resolve conflicts, then continue
# If rebase succeeds: merge to main
git merge main task-xxx
git push origin main

# If any step fails: rollback to Step 5
```

#### Step 7: Mark Complete

**CRITICAL**: Update `data/dev-tasks.json` to mark task as `completed` BEFORE cleanup. This prevents task status loss if process is killed.

```json
{
  "id": "task-xxx",
  "status": "completed",
  "completedAt": "2026-02-19T00:00:00Z"
}
```

#### Step 8: Cleanup

```bash
# Check for uncommitted changes first
git status

# If there are uncommitted changes, either commit or force remove
git worktree remove --force ../fatebook-worktrees/task-xxx

# Remove local branch
git branch -D task/xxx

# Delete remote task branch
git push origin --delete task/xxx
```

> **Tip**: Use the `worktree-cleanup` skill for automated cleanup (installed globally).

**Manual Cleanup Commands (backup):**

```bash
# Check for uncommitted changes first
git status

# If there are uncommitted changes, either commit or force remove
git worktree remove --force ../fatebook-worktrees/task-xxx

# Remove local branch
git branch -D task/xxx

# Delete remote task branch
git push origin --delete task/xxx
```

**Important Cleanup Notes:**

- **Always cleanup after task completion** - Stale worktrees cause confusion and potential conflicts
- **Use `--force` if needed** - If worktree has untracked files, force remove is safe since changes were committed
- **Verify cleanup** - Run `git worktree list` to confirm only main worktree remains
- **Session interrupted?** - If OpenCode session ends unexpectedly, manually cleanup in next session

#### Step 9: Experience Accumulation (Optional)

Record lessons learned in `progress.txt`:

```markdown
## Session: [Date/Time]

### Completed

- [Task description]

### Key Decisions

- Decision: [What was decided]
- Rationale: [Why]

### Lessons Learned

- [Lesson 1]
```

This step is optional - if the process is killed, task status in `dev-tasks.json` is preserved.

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

### Atomic Commits Principle

**Encourage small, atomic commits after each successful fix or implementation step.** This principle helps with:

- **Rollback safety**: Easy to revert specific changes without affecting others
- **Quality tracking**: Each commit represents a verified, working state
- **Code review**: Reviewers can focus on one logical change at a time
- **Debugging**: `git bisect` becomes much more effective

#### What Makes a Commit Atomic

An atomic commit should be:

1. **Focused** - One logical change per commit (e.g., "fix login bug" NOT "fix login and update styles")
2. **Complete** - The change is fully implemented, not half-done
3. **Buildable** - All tests pass after the commit
4. **Independent** - Can be understood without other commits

#### When to Commit

Commit after each successful:

- Bug fix (even a small one)
- Small feature implementation
- Refactoring that improves code
- Adding a test case
- Documentation update

#### Examples

```bash
# Good - Atomic commits
git commit -m "fix(auth): resolve null pointer in token validation"
git commit -m "feat(ui): add loading spinner to submit button"
git commit -m "test(api): add unit test for question router"
git commit -m "docs(readme): update installation instructions"

# Bad - Non-atomic commits
git commit -m "fix various bugs"           # Too vague, multiple fixes
git commit -m "wip"                         # Work in progress
git commit -m "update"                     # What was updated?
git commit -m "fix and refactor"           # Mixed concerns
```

#### Workflow Integration

During task implementation:

1. Complete a small, working change
2. Run tests to verify
3. Commit immediately with descriptive message
4. Continue to next step

If you make a mistake later, you can always use `git revert` or `git reset` to fix it.

---

## Notes

- This is a **Next.js Pages Router** project (not App Router)
- The dev server runs on **port 3005** with SSL proxy to port 3000
- ESLint is configured to require explicit return types on complex functions
- Prettier uses **no semicolons**
