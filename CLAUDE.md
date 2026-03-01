# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**UIGen** is an AI-powered React component generator with live preview. Users describe UI components in natural language, and Claude generates the code, which is displayed in a virtual file system with a Monaco editor and live preview.

## Commands

```bash
npm run setup        # First-time setup: installs deps, generates Prisma client, runs migrations
npm run dev          # Start dev server with Turbopack at localhost:3000
npm run build        # Production build
npm run lint         # ESLint
npm run test         # Run all Vitest tests
npm run db:reset     # Reset SQLite database
```

To run a single test file:
```bash
npx vitest run src/components/chat/__tests__/ChatInterface.test.tsx
```

## Architecture

### Request Flow

1. User submits a prompt in `ChatInterface` → dispatched via `chat-context`
2. `POST /api/chat` (route.ts) calls Claude using the Vercel AI SDK with streaming
3. Claude responds using tool calls defined in `src/lib/tools/` (file creation/modification)
4. Tool results update the virtual file system in `file-system-context`
5. `PreviewFrame` renders the updated files live in an iframe via JSX transformation

### Key Architectural Patterns

- **Virtual File System:** All generated files live in memory (`src/lib/file-system.ts`) and in the `Project.data` JSON column in SQLite — there is no actual filesystem for project files.
- **AI Tools:** Claude uses structured tools (`file-manager.ts`, `str-replace.ts`) to create/edit files. Tool definitions live in `src/lib/tools/`.
- **JSX Transformation:** `src/lib/transform/jsx-transformer.ts` converts JSX to plain JS for the browser preview iframe.
- **Server Actions:** All DB operations go through `src/actions/` (Next.js server actions), not direct API calls from the client.
- **Auth:** JWT-based, managed in `src/lib/auth.ts` and `src/middleware.ts`. Unauthenticated users can generate components but work is not persisted. `src/lib/anon-work-tracker.ts` tracks anonymous session work.

### Data Model

Projects store everything in two JSON columns:
- `messages`: the full chat history
- `data`: the virtual file system (file paths → content)

`userId` is optional — anonymous users can create projects, but they are not linked to an account.

### Path Alias

`@/*` resolves to `./src/*` (configured in `tsconfig.json`).

## Environment

Copy `.env` and set `ANTHROPIC_API_KEY` for AI generation to work. Without a key the app still runs but generation will fail.

## Testing

Tests use Vitest with jsdom and `@testing-library/react`. Test files are colocated with source in `__tests__/` subdirectories. The vitest config is at `vitest.config.mts`.
