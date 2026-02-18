# Architecture Overview

This document provides a comprehensive overview of the Fatebook system architecture.

---

## System Overview

Fatebook is a prediction tracking platform with the following integrations:

- **Web Application**: Primary interface
- **Slack Integration**: Create/resolve predictions from Slack
- **Discord Integration**: Create/resolve predictions from Discord
- **Chrome Extension**: Quick prediction creation
- **REST API**: Programmatic access

---

## Technology Stack

| Layer     | Technology                   |
| --------- | ---------------------------- |
| Framework | Next.js 14 (Pages Router)    |
| API       | tRPC + Zod validation        |
| Database  | PostgreSQL + Prisma ORM      |
| Auth      | NextAuth.js (Google OAuth)   |
| Styling   | Tailwind CSS + DaisyUI       |
| Testing   | Jest + React Testing Library |

### Key Dependencies

```json
{
  "next": "^14.2.35",
  "@trpc/server": "^10.45.2",
  "@prisma/client": "^5.15.0",
  "next-auth": "^4.24.7",
  "zod": "^3.23.8",
  "daisyui": "^4.12.2"
}
```

---

## Project Structure

```
fatebook/
├── pages/                    # Next.js pages (API routes + frontend)
│   ├── api/                  # API endpoints
│   │   ├── auth/            # NextAuth.js endpoints
│   │   ├── discord/        # Discord bot endpoints
│   │   ├── trpc/          # tRPC API
│   │   └── *.ts           # Various webhook handlers
│   ├── embed/              # Embedded prediction widgets
│   └── *.tsx              # Page routes
├── components/              # React components
│   ├── ui/                 # Reusable UI components
│   ├── questions/          # Question-related components
│   ├── predict-form/       # Prediction form components
│   └── *.tsx
├── lib/
│   ├── web/                 # tRPC routers and web utilities
│   │   ├── app_router.ts  # Main API router (aggregates all routers)
│   │   ├── trpc_base.ts   # tRPC context and procedures
│   │   ├── question_router.ts
│   │   ├── userList_router.ts
│   │   ├── tournament_router.ts
│   │   ├── tags_router.ts
│   │   ├── feedback_router.ts
│   │   └── import_router.ts
│   ├── blocks-designs/      # Slack Block Kit designs
│   ├── interactive_handlers/ # Slack interaction handlers
│   ├── slash_handlers/      # Slack slash command handlers
│   └── discord/             # Discord utilities
├── prisma/
│   └── schema.prisma        # Database schema
├── chrome-extension/        # Chrome extension source
└── knowledge/               # Engineering documentation
```

---

## Data Model

### Core Entities

```
User
├── id (cuid)
├── email (unique)
├── name, image
├── apiKey (for REST API)
├── discordUserId
└── relations: forecasts, questions, comments, tournaments

Question
├── id (cuid)
├── title (text)
├── type (BINARY | MULTIPLE_CHOICE | QUANTITY)
├── resolveBy (date)
├── resolved (boolean)
├── resolvedAt (date)
├── userId (author)
├── sharedPublicly, unlisted
└── relations: forecasts, options, comments, tags, tournaments

Forecast
├── id (autoincrement)
├── questionId
├── userId
├── forecast (Decimal - probability)
├── comment (optional)
├── optionId (for MULTIPLE_CHOICE)
└── relations: user, question, option

QuestionOption
├── id (cuid)
├── questionId
├── text
├── userId (author)
└── relations: forecasts, resolution

QuestionScore
├── questionId
├── userId
├── relativeScore (Decimal)
├── absoluteScore (Decimal)
├── rank
└── questionOptionId (optional - for MULTIPLE_CHOICE)

Tournament
├── id (cuid)
├── name, description
├── authorId
├── userListId (optional - team-based)
├── predictYourYear (optional - yearly prediction mode)
└── relations: questions, author

UserList (Team)
├── id (cuid)
├── name
├── inviteId (unique)
├── emailDomains (for auto-join)
├── syncToSlackTeamId, syncToSlackChannelId
├── authorId
└── relations: users, questions, tournaments

Profile
├── id (autoincrement)
├── userId
├── slackId, slackTeamId
└── relations: user, forecasts, questions

Notification
├── id (cuid)
├── userId
├── title, content, url
├── questionId (optional)
├── read (boolean)
└── emailSentAt (optional)
```

### Enums

- `QuestionType`: BINARY, MULTIPLE_CHOICE, QUANTITY
- `Resolution`: YES, NO, AMBIGUOUS
- `DayOfTheWeek`: MONDAY - SUNDAY
- `TargetType`: FORECAST, QUESTION
- `GroupType`: WEB, SLACK

---

## API Architecture

### tRPC Router Hierarchy

```typescript
appRouter
├── question    // Question CRUD, forecasting, resolution
├── userList   // Team/user list management
├── tags       // Tag management
├── import     // Import from external sources
├── tournament // Tournament management
├── feedback   // User feedback collection
└── (public procedures)
    ├── sendEmail
    ├── getSlackPermalink
    ├── unsubscribe
    ├── editName
    ├── getApiKey
    ├── regenerateApiKey
    ├── getUserInfo
    ├── getUserNotifications
    ├── markNotificationRead
    └── countForecasts
```

### Authentication

- **Web**: NextAuth.js with Google OAuth
- **API**: Bearer token via `apiKey` field on User model
- **Slack/Discord**: Bot token verification

### Procedure Types

- `publicProcedure`: Accessible without authentication (most read operations)
- `protectedProcedure`: Requires session (mutations, user-specific data)

---

## Integration Points

### Slack Integration

| Endpoint                       | Purpose                                 |
| ------------------------------ | --------------------------------------- |
| `/api/slack/install`           | OAuth flow for adding app to workspace  |
| `/api/slack/install_approved`  | Handle successful installation          |
| `/api/slash_forecast`          | Handle `/forecast` slash command        |
| `/api/slash_forecast_multiple` | Handle multiple predictions             |
| `/api/interactive_endpoint`    | Handle button clicks, modal submissions |

**Block Kit Designs**: `lib/blocks-designs/`

- `question_modal.ts` - Create question modal
- `resolve_question.ts` - Resolution interface
- `question_resolved.ts` - Resolution notification

### Discord Integration

| Endpoint                    | Purpose                         |
| --------------------------- | ------------------------------- |
| `/api/discord/install`      | OAuth flow                      |
| `/api/discord/interactions` | Handle slash commands & buttons |
| `/api/discord/commands`     | Register slash commands         |

### REST API (OpenAPI)

tRPC exposes OpenAPI document at `/api/openapi.json`:

- `GET /api/v0/countForecasts` - Count user forecasts

---

## Frontend Architecture

### Pages (Routes)

| Route                          | Description                  |
| ------------------------------ | ---------------------------- |
| `/`                            | Home - user predictions list |
| `/q/[id]`                      | Question detail page         |
| `/user/[id]`                   | User profile & predictions   |
| `/list/[...anything]`          | Team list view               |
| `/team/[list_id]`              | Team management              |
| `/team/join/[invite_id]`       | Join team via invite         |
| `/tournament/[id]`             | Tournament view              |
| `/tournament/[id]/leaderboard` | Tournament rankings          |
| `/predict-your-year`           | Yearly prediction mode       |
| `/embed/q/[id]`                | Embedded question widget     |
| `/embed/toast`                 | Toast notification embed     |
| `/stats`                       | User statistics              |
| `/global-stats`                | Global platform stats        |

### Key Components

- **Questions**: `Question.tsx`, `QuestionDetails.tsx`
- **Forecast Form**: `Predict.tsx`, `PredictModal.tsx`
- **Resolution**: `ResolveButton.tsx`, `BinaryResolutionOptions.tsx`
- **Tournament**: `TournamentView.tsx`, `TournamentLeaderboard.tsx`
- **Team**: `TeamView.tsx`, `UserLists.tsx`

---

## Development Commands

```bash
npm run dev          # Start dev server (port 3005, proxied to 3000)
npm run build       # Production build
npm run test        # Run Jest tests
npm run lint        # ESLint
npm run format      # Prettier
npm run studio      # Prisma Studio
```

---

## Database

- **Prisma** ORM with PostgreSQL
- Schema: `prisma/schema.prisma`
- Run `npx prisma generate` after schema changes
- Use `npx prisma studio` for data exploration

---

## Testing

- Jest + React Testing Library
- Tests colocated: `Component.test.tsx` next to `Component.tsx`
- Run: `npm run test`
- Coverage: `npm run test -- --coverage`

---

## Notes

- This is a **Next.js Pages Router** project (not App Router)
- Dev server runs on port **3005** with SSL proxy to port 3000
- ESLint enforces strict TypeScript with explicit return types
- Prettier configured with **no semicolons**
