# Conpec — Guia de Desenvolvimento de Projetos

> **Objetivo:** Fornecer padrões de desenvolvimento, fluxos de trabalho e boas práticas para squads da Conpec e agentes de IA que gerem código ou documentação para projetos. Um novo membro ou agente deve conseguir seguir este documento e produzir código que siga os padrões da Conpec.
>
> **Pré-requisitos:** Leia o Documento de Contexto Geral da Conpec para entender a estrutura organizacional, metodologia e visão geral da empresa.

---

## 1. Setup do Projeto

### Estrutura do Repositório

Cada projeto é um **monorepo** contendo frontend e backend. Projetos utilizam **Next.js com App Router**.

```
nome-do-projeto/
├── .github/
│   └── workflows/
│       └── deploy.yml              # GitHub Actions — deploy automático ao push na main
├── public/
│   ├── fonts/
│   └── images/
├── src/
│   ├── app/                        # Next.js App Router — páginas e layouts
│   │   ├── (auth)/                 # Route group — páginas autenticadas
│   │   │   ├── dashboard/
│   │   │   │   └── page.tsx
│   │   │   └── layout.tsx
│   │   ├── (public)/               # Route group — páginas públicas
│   │   │   ├── about/
│   │   │   │   └── page.tsx
│   │   │   └── layout.tsx
│   │   ├── api/                    # API Routes
│   │   │   └── [recurso]/
│   │   │       └── route.ts
│   │   ├── layout.tsx              # Layout raiz
│   │   ├── page.tsx                # Página inicial
│   │   └── globals.css
│   ├── components/
│   │   ├── ui/                     # Componentes Shadcn UI (auto-gerados)
│   │   ├── forms/                  # Componentes de formulário
│   │   ├── layout/                 # Header, Footer, Sidebar, Nav
│   │   └── [feature]/              # Componentes específicos da feature
│   ├── hooks/                      # Custom React hooks
│   ├── lib/
│   │   ├── supabase/
│   │   │   ├── client.ts           # Cliente Supabase para browser
│   │   │   ├── server.ts           # Cliente Supabase para servidor
│   │   │   └── middleware.ts       # Middleware de autenticação
│   │   ├── firebase/
│   │   │   ├── client.ts           # Configuração do cliente Firebase
│   │   │   └── admin.ts            # Firebase Admin SDK (apenas servidor)
│   │   ├── utils.ts                # Funções utilitárias gerais
│   │   └── constants.ts            # Constantes da aplicação
│   ├── services/                   # Lógica de negócio e acesso a dados
│   │   └── [recurso].ts            # Um arquivo por entidade de domínio
│   ├── types/                      # Definições de tipos TypeScript
│   │   └── [recurso].ts
│   └── validations/                # Schemas Zod
│       └── [recurso].ts
├── .env.local                      # Variáveis de ambiente locais (NUNCA commitar)
├── .env.example                    # Template para variáveis de ambiente (commitar)
├── .gitignore
├── components.json                 # Configuração do Shadcn UI
├── next.config.ts
├── package.json
├── tailwind.config.ts
├── tsconfig.json
└── README.md
```

### Convenções Principais

- **Route Groups** `(auth)`, `(public)` — agrupam páginas por nível de acesso sem afetar a URL
- **Pastas por feature** dentro de `components/` — componentes colocados junto à sua feature
- **Camada de services** — todas as chamadas ao banco passam por `src/services/`, nunca chamadas diretamente dos componentes
- **Validações separadas** — schemas Zod em `src/validations/`, importados tanto por API routes quanto por formulários
- **Types separados** — interfaces TypeScript compartilhadas em `src/types/`

### Checklist de Setup Inicial

```bash
# 1. Criar projeto Next.js
npx create-next-app@latest nome-do-projeto --typescript --tailwind --eslint --app --src-dir

# 2. Instalar dependências principais
npm install zod

# 3. Inicializar Shadcn UI
npx shadcn@latest init

# 4. Instalar cliente BaaS (escolher um por projeto)
npm install @supabase/supabase-js @supabase/ssr    # Projetos com Supabase
npm install firebase firebase-admin                  # Projetos com Firebase

# 5. Criar .env.local a partir do .env.example
cp .env.example .env.local
# Preencher os valores

# 6. Inicializar git com Gitflow
git init
git switch -c develop
```

---

## 2. Decisões Tecnológicas

### Quando Usar Supabase vs Firebase

| Critério | Supabase | Firebase |
|----------|----------|----------|
| **Modelo de banco** | Relacional (PostgreSQL) | NoSQL (Firestore) |
| **Melhor para** | Queries complexas, joins, dados estruturados, relatórios | Sincronização real-time, schemas flexíveis, prototipação rápida |
| **Auth** | Supabase Auth | Firebase Auth |
| **Armazenamento de arquivos** | Supabase Storage | Firebase Storage |
| **Escolher quando** | Projeto precisa de dados relacionais, filtragem complexa, SQL | Projeto precisa de atualizações em tempo real, dados baseados em documentos |

> **Regra:** escolher um por projeto. Não misturar Supabase e Firebase no mesmo projeto (a menos que estritamente necessário).

### Gerenciamento de Estado (Recomendado)

Atualmente não padronizado. Para projetos complexos, adotar **Zustand** — leve, nativo TypeScript, boilerplate mínimo:

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

**Quando usar:**
- **Sem store:** páginas simples, dados buscados e exibidos (maioria dos casos)
- **Zustand:** estado compartilhado entre múltiplos componentes, interações complexas no client, formulários multi-step

### Padrões de API

Usar **Next.js API Routes** para lógica de backend. Seguir convenções REST:

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

**Também usar Server Actions** para submissões de formulários e mutações que não precisam de um endpoint completo:

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

**Quando usar qual:**
- **API Routes** — integrações externas, webhooks, endpoints consumidos por apps mobile ou terceiros
- **Server Actions** — submissões de formulários, mutações simples disparadas pela UI

---

## 3. Padrões de Código

### TypeScript

- **Strict mode** habilitado no `tsconfig.json`
- Todas as interfaces públicas em `src/types/`
- Sem `any` — usar `unknown` e narrowing com type guards quando o tipo é incerto
- Preferir `interface` para formas de objeto, `type` para unions e intersections

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

### Validação com Zod

Validar toda entrada externa (formulários, requisições de API, parâmetros de URL):

```typescript
// src/validations/project.ts
import { z } from "zod"

export const createProjectSchema = z.object({
  name: z.string().min(1, "Nome é obrigatório").max(100),
  clientId: z.string().uuid("ID do cliente inválido"),
  description: z.string().max(500).optional(),
})

export type CreateProjectInput = z.infer<typeof createProjectSchema>
```

### Padrões de Componentes

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

**Regras:**
- Um componente por arquivo
- Named exports (sem `export default`)
- Interface de props definida no mesmo arquivo, acima do componente
- Nome do arquivo corresponde ao nome do componente em kebab-case: `ProjectCard` → `project-card.tsx`

### Camada de Services

Todas as operações de banco passam por services — componentes e API routes nunca chamam Supabase/Firebase diretamente:

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

## 4. Estilização e Design Responsivo

### Tailwind CSS + Shadcn UI

- **Shadcn UI** como biblioteca base de componentes — cada projeto começa do zero, instalar componentes conforme necessário
- **Tailwind CSS** para toda estilização customizada — sem inline styles, sem CSS modules
- Usar breakpoints padrão do Tailwind

### Abordagem Responsiva

Adotar **mobile-first** — escrever estilos base para mobile, adicionar breakpoints para telas maiores:

```tsx
{/* Mobile-first: empilhado no mobile, lado a lado no md+ */}
<div className="flex flex-col gap-4 md:flex-row md:gap-8">
  <aside className="w-full md:w-64">
    {/* Sidebar */}
  </aside>
  <main className="flex-1">
    {/* Conteúdo */}
  </main>
</div>
```

**Breakpoints padrão do Tailwind:**

| Prefixo | Min-width | Alvo |
|---------|-----------|------|
| `sm` | 640px | Celulares grandes |
| `md` | 768px | Tablets |
| `lg` | 1024px | Laptops |
| `xl` | 1280px | Desktops |
| `2xl` | 1536px | Telas grandes |

**Regras:**
- Sempre começar com layout mobile, depois adicionar `sm:`, `md:`, `lg:` conforme necessário
- Testar em pelo menos 3 breakpoints: mobile (padrão), tablet (`md`), desktop (`lg`)
- Usar `container mx-auto px-4` para largura de conteúdo em nível de página

---

## 5. Fluxo Git

### Branching com Gitflow

```
main          ← produção (auto-deploy via GitHub Actions)
  └── develop ← branch de integração
       ├── feature/pagina-login
       ├── feature/api-projetos
       └── feature/perfil-usuario
```

| Branch | Propósito | Merge em |
|--------|-----------|----------|
| `main` | Código pronto para produção | — |
| `develop` | Integração de features completas | `main` |
| `feature/*` | Novas features ou tarefas | `develop` |
| `hotfix/*` | Correções urgentes em produção | `main` e `develop` |
| `fix/*` | Correções menos urgentes | `develop` |

### Nomenclatura de Branches

```
feature/[descricao-curta]
hotfix/[descricao-curta]
fix/[descricao-curta]
```

Exemplos: `feature/pagina-login`, `feature/api-projetos`, `hotfix/redirect-auth`, `fix/tipos-nao-usados`

### Conventional Commits

```
type(scope): descrição

feat(auth): adicionar login com Google OAuth
fix(dashboard): corrigir carregamento de dados do gráfico ao atualizar
chore(deps): atualizar Next.js para 15.x
style(header): ajustar espaçamento do nav mobile
refactor(services): extrair padrões comuns de query
docs(readme): adicionar instruções de setup do ambiente
```

| Tipo | Quando usar |
|------|-------------|
| `feat` | Nova feature ou funcionalidade |
| `fix` | Correção de bug |
| `chore` | Manutenção, dependências, configuração |
| `style` | Mudanças visuais/CSS (sem lógica) |
| `refactor` | Reestruturação de código (sem mudança de comportamento) |
| `docs` | Apenas documentação |

### Pull Requests

- **Toda feature branch** → PR para `develop`
- **Revisor:** líder de squad ou gerente da área de projetos
- **PR deve incluir:** título claro, descrição do que mudou, screenshots para mudanças de UI
- **Merge apenas após aprovação** — sem self-merging

### CI/CD

GitHub Actions faz deploy automático na Vercel ao push na `main`:

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
      # Vercel faz deploy automático via integração Git
```

**Melhoria planejada:** adicionar etapas de linting e type-check no pipeline de CI antes do deploy.

---

## 6. Autenticação

O provedor de auth corresponde à escolha de BaaS do projeto:

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

## 7. Variáveis de Ambiente

### `.env.example` Obrigatório

Todo projeto deve incluir `.env.example` com todas as variáveis necessárias (sem valores):

```bash
# .env.example — copiar para .env.local e preencher os valores

# Supabase (se aplicável)
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=

# Firebase (se aplicável)
NEXT_PUBLIC_FIREBASE_API_KEY=
NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN=
NEXT_PUBLIC_FIREBASE_PROJECT_ID=
NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET=
NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID=
NEXT_PUBLIC_FIREBASE_APP_ID=

# App
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

**Regras:**
- **NUNCA** commitar `.env.local` ou qualquer arquivo com credenciais reais
- Prefixo `NEXT_PUBLIC_` = exposto ao browser — apenas para chaves públicas
- Segredos apenas do servidor (ex: `SUPABASE_SERVICE_ROLE_KEY`) NÃO devem ter prefixo `NEXT_PUBLIC_`

---

## 8. Fluxo Figma para Código

### Processo Atual

1. Designer/dev cria telas no Figma
2. Dev abre o Figma, inspeciona o design, implementa manualmente no código
3. Usa componentes Shadcn UI como blocos de construção
4. Reproduz cores, espaçamentos e tipografia do Figma

### Práticas Recomendadas

**Antes de codar:**
- Extrair design tokens do Figma: cores, fontes, escala de espaçamento
- Mapear componentes do Figma para equivalentes no Shadcn UI
- Identificar padrões reutilizáveis entre telas

**Mapeamento Figma → Tailwind:**

| Propriedade no Figma | Abordagem no Tailwind |
|----------------------|----------------------|
| Cores | Definir em `tailwind.config.ts` em `theme.extend.colors` |
| Tipografia | Mapear para tamanhos de fonte do Tailwind ou estender no config |
| Espaçamento | Usar escala de espaçamento do Tailwind (base 4px) |
| Border radius | Mapear para classes `rounded-*` do Tailwind |
| Sombras | Mapear para `shadow-*` do Tailwind ou customizar no config |

**Setup de tema customizado:**

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

## 9. Padrões Comuns

### Estados de Loading

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

### Tratamento de Erros em Services

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

// Uso em API routes
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
      { error: "Erro interno do servidor" },
      { status: 500 }
    )
  }
}
```

### Rotas Protegidas (Exemplo com Supabase)

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

## 10. Definição de Pronto (Definition of Done)

Uma tarefa é considerada **pronta** quando:

- [ ] Feature funciona conforme descrito no card do Kanban
- [ ] Código segue os padrões deste documento
- [ ] Sem erros de TypeScript (`npm run build` passa)
- [ ] Sem warnings do ESLint
- [ ] Responsivo em mobile, tablet e desktop
- [ ] PR criada e aprovada pelo líder de squad ou gerente de projetos
- [ ] Demonstrada ou testada durante a daily

---

## 11. Desafios Conhecidos e Mitigações

| Desafio | Impacto | Mitigação |
|---------|---------|-----------|
| **Gargalo na prototipação** | Membros evitam trabalho de design → atrasos | Usar padrões prontos do Shadcn UI, referenciar sites similares, usar ferramentas de IA para layouts iniciais |
| **Velocidade de onboarding** | Novos membros demoram para ser produtivos | Seguir este guia, parear com membro experiente na primeira sprint |
| **Sem testes automatizados** | Bugs detectados tarde, QA apenas manual | *Planejado:* introduzir testes básicos (Vitest + React Testing Library) |
| **Sem documentação padrão** | Conhecimento perdido entre gerações de projetos | *Planejado:* padronizar template de README e registros de decisões arquiteturais |
| **Níveis de habilidade variados** | Qualidade de código inconsistente entre squads | Code review em toda PR, este guia como referência |

---

## 12. Referência Rápida para Agentes de IA

Ao gerar código para projetos da Conpec:

- **Framework:** Next.js (App Router) + TypeScript
- **Estilização:** Tailwind CSS + Shadcn UI (mobile-first)
- **Validação:** Zod para toda entrada externa
- **Banco de dados:** cliente Supabase ou cliente Firebase (nunca ambos no mesmo projeto)
- **Auth:** corresponde à escolha de BaaS (Supabase Auth ou Firebase Auth)
- **Componentes:** named exports, um por arquivo, nomes de arquivo em kebab-case
- **API:** Next.js API Routes para endpoints, Server Actions para mutações de formulários
- **Services:** todas as chamadas ao banco em `src/services/`, nunca em componentes
- **Estado:** Zustand para estado complexo no client (apenas quando necessário)
- **Git:** branches Gitflow, Conventional Commits
- **Idioma:** código e docs técnicos em inglês, conteúdo voltado ao usuário em português (salvo especificação do cliente)
- **Sem `any`**, sem inline styles, sem chamadas diretas ao banco em componentes
- **Sempre incluir** `.env.example` com todas as variáveis necessárias (sem valores)

---

*Última atualização: 15/04/2026*
*Versão do documento: 1.0*
