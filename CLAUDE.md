# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Next.js 15 application using the App Router, built with TypeScript, Tailwind CSS, and Supabase for authentication and data storage. It's based on the official Next.js + Supabase starter template with shadcn/ui components.

## Development Commands

- **Development server**: `npm run dev` (uses Turbopack)
- **Build**: `npm run build`
- **Start production**: `npm start`
- **Lint**: `npm run lint`

## Environment Setup

Required environment variables (copy from `.env.example` to `.env.local`):
```
NEXT_PUBLIC_SUPABASE_URL=https://ekrtdfjiayzrstxdblni.supabase.co
NEXT_PUBLIC_SUPABASE_PUBLISHABLE_OR_ANON_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImVrcnRkZmppYXl6cnN0eGRibG5pIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NTM5NjkzOTIsImV4cCI6MjA2OTU0NTM5Mn0.QVSIphabiqaCG_v-U9pLC9F4zYyxd-e0HLM1TrIx-m8
```

Both values can be found in your Supabase project's API settings.

## Architecture

### Authentication Flow

**Supabase SSR Integration**: Uses cookie-based authentication via `@supabase/ssr` package, making user sessions available across:
- Client Components
- Server Components
- Route Handlers
- Server Actions
- Middleware

**Client Creation Patterns**:
- **Server Components/Actions**: Use `createClient()` from `lib/supabase/server.ts` - creates new client per request (required for Fluid compute)
- **Client Components**: Use `createClient()` from `lib/supabase/client.ts` - creates browser client
- **Middleware**: Use `updateSession()` from `lib/supabase/middleware.ts`

**IMPORTANT**: Never store Supabase clients in global variables - always create a new client within each function.

### Protected Routes

The middleware (`middleware.ts`) automatically:
- Refreshes user sessions via `updateSession()`
- Redirects unauthenticated users to `/auth/login` (except for public routes like `/`, `/auth/*`)
- Uses `supabase.auth.getClaims()` to check authentication status

### Route Structure

```
app/
├── page.tsx                    # Public homepage
├── layout.tsx                  # Root layout with ThemeProvider
├── auth/
│   ├── login/                  # Login page
│   ├── sign-up/                # Sign up page
│   ├── forgot-password/        # Password reset
│   ├── update-password/        # Update password
│   ├── sign-up-success/        # Post-signup confirmation
│   ├── error/                  # Auth error handling
│   └── confirm/route.ts        # Email confirmation handler
└── protected/
    ├── layout.tsx              # Protected layout
    └── page.tsx                # Protected page (requires auth)
```

### Component Structure

- **UI Components**: Located in `components/ui/` - shadcn/ui components (Button, Card, Input, Label, Checkbox, DropdownMenu, Badge)
- **Auth Components**: Login/signup forms, auth buttons, logout functionality
- **Tutorial Components**: Step-by-step guides in `components/tutorial/`
- **Theme**: Dark/light mode support via `next-themes` with ThemeSwitcher component

### Path Aliases

TypeScript paths configured with `@/*` pointing to root directory:
```typescript
import { createClient } from "@/lib/supabase/server";
```

## Key Implementation Notes

1. **Session Management**: The middleware must call `supabase.auth.getClaims()` to prevent random logouts with SSR
2. **Response Handling**: When creating custom responses in middleware, always copy cookies from `supabaseResponse` to avoid session termination
3. **Theme Support**: Application supports system/light/dark themes with `suppressHydrationWarning` on html tag
4. **Styling**: Uses Tailwind CSS with custom configuration in `tailwind.config.ts` and shadcn/ui components via `components.json`
