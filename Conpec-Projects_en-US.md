# Conpec — Projects Development Guide

> **Purpose:** Provide development standards, workflows, and best practices for Conpec squads and AI agents generating code or documentation for projects. A new member or agent should be able to follow this document and produce code that matches Conpec's patterns.
>
> **Prerequisites:** Read the Conpec General Context Document for organizational structure, methodology, and company overview.

---

## 1. Project Setup

### Repository Structure

Each project is a **monorepo** containing both frontend and backend. Projects use **Next.js with App Router**.

```
project-name/
├── .github/
│   └── workflows/
│       └── deploy.yml              # GitHub Actions — auto-deploy on push to main
├── public/
│   ├── fonts/
│   └── images/
├── src/
│   ├── app/                        # Next.js App Router — pages and layouts
│   │   ├── (auth)/                 # Route group — authenticated pages
│   │   │   ├── dashboard/
│   │   │   │   └── page.tsx
│   │   │   └── layout.tsx
│   │   ├── (public)/               # Route group — public pages
│   │   │   ├── about/
│   │   │   │   └── page.tsx
│   │   │   └── layout.tsx
│   │   ├── api/                    # API Routes
│   │   │   └── [resource]/
│   │   │       └── route.ts
│   │   ├── layout.tsx              # Root layout
│   │   ├── page.tsx                # Home page
│   │   └── globals.css
│   ├── components/
│   │   ├── ui/                     # Shadcn UI components (auto-generated)
│   │   ├── forms/                  # Form components
│   │   ├── layout/                 # Header, Footer, Sidebar, Nav
│   │   └── [feature]/              # Feature-specific components
│   ├── hooks/                      # Custom React hooks
│   ├── lib/
│   │   ├── supabase/
│   │   │   ├── client.ts           # Browser Supabase client
│   │   │   ├── server.ts           # Server Supabase client
│   │   │   └── middleware.ts       # Auth middleware
│   │   ├── firebase/
│   │   │   ├── client.ts           # Firebase client config
│   │   │   └── admin.ts            # Firebase Admin SDK (server-only)
│   │   ├── utils.ts                # General utility functions
│   │   └── constants.ts            # App-wide constants
│   ├── services/                   # Business logic and data access
│   │   └── [resource].ts           # One file per domain entity
│   ├── types/                      # TypeScript type definitions
│   │   └── [resource].ts
│   └── validations/                # Zod schemas
│       └── [resource].ts
├── .env.local                      # Local environment variables (NEVER commit)
├── .env.example                    # Template for env vars (commit this)
├── .gitignore
├── components.json                 # Shadcn UI config
├── next.config.ts
├── package.json
├── tailwind.config.ts
├── tsconfig.json
└── README.md
```

### Key Conventions

- **Route Groups** `(auth)`, `(public)` — group pages by access level without affecting URL
- **Feature folders** inside `components/` — colocate components with their feature
- **Services layer** — all database calls go through `src/services/`, never called directly from components
- **Validations separate** — Zod schemas in `src/validations/`, imported by both API routes and forms
- **Types separate** — shared TypeScript interfaces in `src/types/`

### Initial Setup Checklist

```bash
# 1. Create Next.js project
npx create-next-app@latest project-name --typescript --tailwind --eslint --app --src-dir

# 2. Install core dependencies
npm install zod

# 3. Initialize Shadcn UI
npx shadcn@latest init

# 4. Install BaaS client (pick one per project)
npm install @supabase/supabase-js @supabase/ssr    # Supabase projects
npm install firebase firebase-admin                  # Firebase projects

# 5. Create .env.local from .env.example
cp .env.example .env.local
# Fill in the values

# 6. Initialize git with Gitflow
git init
git switch -c develop
```

---

## 2. Technology Decisions

### When to Use Supabase vs Firebase

| Criteria | Supabase | Firebase |
|----------|----------|----------|
| **Database model** | Relational (PostgreSQL) | NoSQL (Firestore) |
| **Best for** | Complex queries, joins, structured data, reporting | Real-time sync, flexible schemas, rapid prototyping |
| **Auth** | Supabase Auth | Firebase Auth |
| **File storage** | Supabase Storage | Firebase Storage |
| **Choose when** | Project needs relational data, complex filtering, SQL | Project needs real-time updates, document-based data |

> **Rule:** pick one per project. Do not mix Supabase and Firebase in same project (Unless strictly needed).

### State Management (Recommended)

Currently not standardized. For complex projects, adopt **Zustand** — lightweight, TypeScript-native, minimal boilerplate:

```typescript
// src/stores/use-example-store.ts
import { create } from "zustand"

interface ExampleState {
  count: number
  increment: () => void
  reset: () => void
}

export const useExampleStore = create<ExampleState>((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
  reset: () => set({ count: 0 }),
}))
```

**When to use:**
- **No store needed:** simple pages, data fetched and displayed (most cases)
- **Zustand:** shared state across multiple components, complex client-side interactions, multi-step forms

### API Patterns

Use **Next.js API Routes** for backend logic. Follow REST conventions:

```typescript
// src/app/api/users/route.ts
import { NextRequest, NextResponse } from "next/server"
import { createUserSchema } from "@/validations/user"
import { UserService } from "@/services/user"

export async function GET() {
  const users = await UserService.getAll()
  return NextResponse.json(users)
}

export async function POST(request: NextRequest) {
  const body = await request.json()
  const parsed = createUserSchema.safeParse(body)

  if (!parsed.success) {
    return NextResponse.json(
      { error: parsed.error.flatten() },
      { status: 400 }
    )
  }

  const user = await UserService.create(parsed.data)
  return NextResponse.json(user, { status: 201 })
}
```

**Also use Server Actions** for form submissions and mutations that don't need a full API endpoint:

```typescript
// src/app/(auth)/settings/actions.ts
"use server"

import { updateProfileSchema } from "@/validations/user"
import { UserService } from "@/services/user"

export async function updateProfile(formData: FormData) {
  const parsed = updateProfileSchema.safeParse(
    Object.fromEntries(formData)
  )

  if (!parsed.success) {
    return { error: parsed.error.flatten() }
  }

  await UserService.update(parsed.data)
  return { success: true }
}
```

**When to use which:**
- **API Routes** — external integrations, webhooks, endpoints consumed by mobile apps or third parties
- **Server Actions** — form submissions, simple mutations triggered from UI

---

## 3. Code Standards

### TypeScript

- **Strict mode** enabled in `tsconfig.json`
- All public interfaces in `src/types/`
- No `any` — use `unknown` and narrow with type guards when type is uncertain
- Prefer `interface` for object shapes, `type` for unions and intersections

```typescript
// src/types/project.ts
export interface Project {
  id: string
  name: string
  status: ProjectStatus
  clientId: string
  createdAt: Date
  updatedAt: Date
}

export type ProjectStatus = "planning" | "in_progress" | "review" | "delivered"
```

### Validation with Zod

Validate all external input (forms, API requests, URL params):

```typescript
// src/validations/project.ts
import { z } from "zod"

export const createProjectSchema = z.object({
  name: z.string().min(1, "Name is required").max(100),
  clientId: z.string().uuid("Invalid client ID"),
  description: z.string().max(500).optional(),
})

export type CreateProjectInput = z.infer<typeof createProjectSchema>
```

### Component Patterns

```typescript
// src/components/projects/project-card.tsx
import { Project } from "@/types/project"
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card"
import { Badge } from "@/components/ui/badge"

interface ProjectCardProps {
  project: Project
}

export function ProjectCard({ project }: ProjectCardProps) {
  return (
    <Card>
      <CardHeader>
        <CardTitle>{project.name}</CardTitle>
      </CardHeader>
      <CardContent>
        <Badge>{project.status}</Badge>
      </CardContent>
    </Card>
  )
}
```

**Rules:**
- One component per file
- Named exports (no `export default`)
- Props interface defined in same file, above component
- File name matches component name in kebab-case: `ProjectCard` → `project-card.tsx`

### Services Layer

All database operations go through services — components and API routes never call Supabase/Firebase directly:

```typescript
// src/services/project.ts
import { supabase } from "@/lib/supabase/client"
import { Project } from "@/types/project"
import { CreateProjectInput } from "@/validations/project"

export const ProjectService = {
  async getAll(): Promise<Project[]> {
    const { data, error } = await supabase
      .from("projects")
      .select("*")
      .order("created_at", { ascending: false })

    if (error) throw error
    return data
  },

  async create(input: CreateProjectInput): Promise<Project> {
    const { data, error } = await supabase
      .from("projects")
      .insert(input)
      .select()
      .single()

    if (error) throw error
    return data
  },
}
```

---

## 4. Styling & Responsive Design

### Tailwind CSS + Shadcn UI

- **Shadcn UI** as base component library — each project starts fresh, install components as needed
- **Tailwind CSS** for all custom styling — no inline styles, no CSS modules
- Use Tailwind's default breakpoints

### Responsive Approach

Adopt **mobile-first** — write base styles for mobile, add breakpoints for larger screens:

```tsx
{/* Mobile-first: stack on mobile, side-by-side on md+ */}
<div className="flex flex-col gap-4 md:flex-row md:gap-8">
  <aside className="w-full md:w-64">
    {/* Sidebar */}
  </aside>
  <main className="flex-1">
    {/* Content */}
  </main>
</div>
```

**Tailwind default breakpoints:**

| Prefix | Min-width | Target |
|--------|-----------|--------|
| `sm` | 640px | Large phones |
| `md` | 768px | Tablets |
| `lg` | 1024px | Laptops |
| `xl` | 1280px | Desktops |
| `2xl` | 1536px | Large screens |

**Rules:**
- Always start with mobile layout, then add `sm:`, `md:`, `lg:` as needed
- Test on at least 3 breakpoints: mobile (default), tablet (`md`), desktop (`lg`)
- Use `container mx-auto px-4` for page-level content width

---

## 5. Git Workflow

### Gitflow Branching

```
main          ← production (auto-deploys via GitHub Actions)
  └── develop ← integration branch
       ├── feature/login-page
       ├── feature/dashboard-api
       └── feature/user-profile
```

| Branch | Purpose | Merges into |
|--------|---------|-------------|
| `main` | Production-ready code | — |
| `develop` | Integration of completed features | `main` |
| `feature/*` | New features or tasks | `develop` |
| `hotfix/*` | Urgent production fixes | `main` and `develop` |
| `fix/*` | Less urgent fixes | `develop` |

### Branch Naming

```
feature/[short-description]
hotfix/[short-description]
fix/[short-description]
```

Examples: `feature/login-page`, `feature/api-projects`, `hotfix/auth-redirect`, `fix/unused-types`

### Conventional Commits

```
type(scope): description

feat(auth): add Google OAuth login
fix(dashboard): correct chart data loading on refresh
chore(deps): update Next.js to 15.x
style(header): adjust mobile nav spacing
refactor(services): extract common query patterns
docs(readme): add environment setup instructions
```

| Type | When |
|------|------|
| `feat` | New feature or functionality |
| `fix` | Bug fix |
| `chore` | Maintenance, dependencies, config |
| `style` | Visual/CSS changes (no logic) |
| `refactor` | Code restructuring (no behavior change) |
| `docs` | Documentation only |

### Pull Requests

- **Every feature branch** → PR into `develop`
- **Reviewer:** squad leader or projects area manager
- **PR must include:** clear title, description of what changed, screenshots for UI changes
- **Merge only after approval** — no self-merging

### CI/CD

GitHub Actions auto-deploys to Vercel on push to `main`:

```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm ci
      - run: npm run build
      # Vercel auto-deploys via Git integration
```

**Planned improvement:** add linting and type-check steps to CI pipeline before deploy.

---

## 6. Authentication

Auth provider matches BaaS choice for project:

### Supabase Auth

```typescript
// src/lib/supabase/client.ts
import { createBrowserClient } from "@supabase/ssr"

export function createClient() {
  return createBrowserClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
  )
}
```

```typescript
// src/lib/supabase/server.ts
import { createServerClient } from "@supabase/ssr"
import { cookies } from "next/headers"

export async function createServerSupabase() {
  const cookieStore = await cookies()

  return createServerClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    {
      cookies: {
        getAll() {
          return cookieStore.getAll()
        },
        setAll(cookiesToSet) {
          cookiesToSet.forEach(({ name, value, options }) =>
            cookieStore.set(name, value, options)
          )
        },
      },
    }
  )
}
```

### Firebase Auth

```typescript
// src/lib/firebase/client.ts
import { initializeApp, getApps } from "firebase/app"
import { getAuth } from "firebase/auth"
import { getFirestore } from "firebase/firestore"

const firebaseConfig = {
  apiKey: process.env.NEXT_PUBLIC_FIREBASE_API_KEY,
  authDomain: process.env.NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN,
  projectId: process.env.NEXT_PUBLIC_FIREBASE_PROJECT_ID,
  storageBucket: process.env.NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET,
  messagingSenderId: process.env.NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID,
  appId: process.env.NEXT_PUBLIC_FIREBASE_APP_ID,
}

const app = getApps().length === 0 ? initializeApp(firebaseConfig) : getApps()[0]
export const auth = getAuth(app)
export const db = getFirestore(app)
```

---

## 7. Environment Variables

### Required `.env.example`

Every project must include `.env.example` with all required variables (no values):

```bash
# .env.example — copy to .env.local and fill in values

# Supabase (if applicable)
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=

# Firebase (if applicable)
NEXT_PUBLIC_FIREBASE_API_KEY=
NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN=
NEXT_PUBLIC_FIREBASE_PROJECT_ID=
NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET=
NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID=
NEXT_PUBLIC_FIREBASE_APP_ID=

# App
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

**Rules:**
- **NEVER** commit `.env.local` or any file with real credentials
- `NEXT_PUBLIC_` prefix = exposed to browser — only for public keys
- Server-only secrets (e.g., `SUPABASE_SERVICE_ROLE_KEY`) must NOT have `NEXT_PUBLIC_` prefix

---

## 8. Figma to Code Workflow

### Current Process

1. Designer/dev creates screens in Figma
2. Dev opens Figma, inspects design, implements manually in code
3. Uses Shadcn UI components as building blocks
4. Matches colors, spacing, typography from Figma

### Recommended Practices

**Before coding:**
- Extract design tokens from Figma: colors, fonts, spacing scale
- Map Figma components to Shadcn UI equivalents
- Identify reusable patterns across screens

**Figma → Tailwind mapping:**

| Figma property | Tailwind approach |
|----------------|-------------------|
| Colors | Define in `tailwind.config.ts` under `theme.extend.colors` |
| Typography | Map to Tailwind font sizes or extend in config |
| Spacing | Use Tailwind's spacing scale (4px base) |
| Border radius | Map to Tailwind's `rounded-*` classes |
| Shadows | Map to Tailwind's `shadow-*` or custom in config |

**Custom theme setup:**

```typescript
// tailwind.config.ts
import type { Config } from "tailwindcss"

const config: Config = {
  // ...
  theme: {
    extend: {
      colors: {
        brand: {
          50: "#f0f7ff",
          500: "#3b82f6",
          900: "#1e3a5f",
        },
      },
      fontFamily: {
        heading: ["var(--font-heading)"],
        body: ["var(--font-body)"],
      },
    },
  },
}

export default config
```

---

## 9. Common Patterns

### Loading States

```tsx
import { Skeleton } from "@/components/ui/skeleton"

export function ProjectListSkeleton() {
  return (
    <div className="space-y-4">
      {Array.from({ length: 3 }).map((_, i) => (
        <Skeleton key={i} className="h-24 w-full rounded-lg" />
      ))}
    </div>
  )
}
```

### Error Handling in Services

```typescript
// src/lib/utils.ts
export class AppError extends Error {
  constructor(
    message: string,
    public statusCode: number = 500
  ) {
    super(message)
  }
}

// Usage in API routes
import { AppError } from "@/lib/utils"

export async function GET() {
  try {
    const data = await ProjectService.getAll()
    return NextResponse.json(data)
  } catch (error) {
    if (error instanceof AppError) {
      return NextResponse.json(
        { error: error.message },
        { status: error.statusCode }
      )
    }
    return NextResponse.json(
      { error: "Internal server error" },
      { status: 500 }
    )
  }
}
```

### Protected Routes (Supabase Example)

```typescript
// src/middleware.ts
import { createServerClient } from "@supabase/ssr"
import { NextResponse, type NextRequest } from "next/server"

export async function middleware(request: NextRequest) {
  const response = NextResponse.next()

  const supabase = createServerClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    {
      cookies: {
        getAll() {
          return request.cookies.getAll()
        },
        setAll(cookiesToSet) {
          cookiesToSet.forEach(({ name, value, options }) => {
            response.cookies.set(name, value, options)
          })
        },
      },
    }
  )

  const { data: { user } } = await supabase.auth.getUser()

  if (!user && request.nextUrl.pathname.startsWith("/dashboard")) {
    return NextResponse.redirect(new URL("/login", request.url))
  }

  return response
}

export const config = {
  matcher: ["/dashboard/:path*"],
}
```

---

## 10. Definition of Done

A task is considered **done** when:

- [ ] Feature works as described in Kanban card
- [ ] Code follows patterns in this document
- [ ] No TypeScript errors (`npm run build` succeeds)
- [ ] No ESLint warnings
- [ ] Responsive on mobile, tablet, and desktop
- [ ] PR created and approved by squad leader or projects manager
- [ ] Demonstrated or tested during daily meeting

---

## 11. Known Challenges & Mitigations

| Challenge | Impact | Mitigation |
|-----------|--------|------------|
| **Prototyping bottleneck** | Members avoid design work → delays | Use Shadcn UI prebuilt patterns, reference similar sites, leverage AI tools for initial layouts |
| **Onboarding speed** | New members take time to be productive | Follow this guide, pair with experienced member first sprint |
| **No automated tests** | Bugs caught late, manual QA only | *Planned:* introduce basic testing (Vitest + React Testing Library) |
| **No standard documentation** | Knowledge lost between project generations | *Planned:* standardize README template and architecture decision records |
| **Variable skill levels** | Code quality inconsistent across squads | Code review on every PR, this guide as reference |

---

## 12. Quick Reference for AI Agents

When generating code for Conpec projects:

- **Framework:** Next.js (App Router) + TypeScript
- **Styling:** Tailwind CSS + Shadcn UI (mobile-first)
- **Validation:** Zod for all external input
- **Database:** Supabase client or Firebase client (never both in same project)
- **Auth:** matches BaaS choice (Supabase Auth or Firebase Auth)
- **Components:** named exports, one per file, kebab-case filenames
- **API:** Next.js API Routes for endpoints, Server Actions for form mutations
- **Services:** all DB calls in `src/services/`, never in components
- **State:** Zustand for complex client state (only when needed)
- **Git:** Gitflow branches, Conventional Commits
- **Language:** code and technical docs in English, user-facing content in Portuguese (unless client specifies otherwise)
- **No `any`**, no inline styles, no direct DB calls in components
- **Always include** `.env.example` with all required vars (no values)

---

*Last updated: 2026-04-15*
*Document version: 1.0*
