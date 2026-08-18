# Utils e Middlewares - Supabase

## Estrutura

```
src/utils/supabase/          ← Padrão oficial Supabase
├── client.ts                ← Browser client (createBrowserClient)
├── server.ts                ← Server client (createServerClient + cookies)
└── middleware.ts             ← Session refresh + proteção de rotas

src/middleware.ts             ← Next.js middleware (chama utils/supabase/middleware)

src/lib/supabase.ts          ← Cliente legado (ainda usado por db/)
src/lib/db/                  ← CRUD por domínio
```

---

## utils/supabase/client.ts

Browser client para Componentes React (client components).

```typescript
import { createBrowserClient } from "@supabase/ssr"

export function createClient() {
  return createBrowserClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
  )
}
```

**Uso:**
```typescript
import { createClient } from "@/utils/supabase/client"
const supabase = createClient()
const { data } = await supabase.from("users").select("*")
```

---

## utils/supabase/server.ts

Server client para Server Components e Server Actions.

```typescript
import { createServerClient } from "@supabase/ssr"
import { cookies } from "next/headers"

export async function createClient() {
  const cookieStore = await cookies()
  return createServerClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    {
      cookies: {
        getAll() { return cookieStore.getAll() },
        setAll(cookiesToSet) {
          try {
            cookiesToSet.forEach(({ name, value, options }) =>
              cookieStore.set(name, value, options)
            )
          } catch { /* Server Component - ignorar */ }
        },
      },
    }
  )
}
```

**Uso em Server Component:**
```typescript
import { createClient } from "@/utils/supabase/server"

export default async function Page() {
  const supabase = await createClient()
  const { data: { user } } = await supabase.auth.getUser()
  // ...
}
```

---

## utils/supabase/middleware.ts

Middleware que roda em CADA request para:
1. Refresh da sessão Supabase Auth
2. Proteger rotas (redireciona para /login se não autenticado)
3. Redirecionar logado para /schools se acessar /login

```typescript
import { createServerClient } from "@supabase/ssr"
import { NextResponse, type NextRequest } from "next/server"

export async function updateSession(request: NextRequest) {
  let supabaseResponse = NextResponse.next({ request })

  const supabase = createServerClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    {
      cookies: {
        getAll() { return request.cookies.getAll() },
        setAll(cookiesToSet) {
          cookiesToSet.forEach(({ name, value }) =>
            request.cookies.set(name, value)
          )
          supabaseResponse = NextResponse.next({ request })
          cookiesToSet.forEach(({ name, value, options }) =>
            supabaseResponse.cookies.set(name, value, options)
          )
        },
      },
    }
  )

  await supabase.auth.getUser()

  const { data: { user } } = await supabase.auth.getUser()
  const isAuthPage = request.nextUrl.pathname === "/login"
  const isProtectedRoute = !isAuthPage && request.nextUrl.pathname !== "/"

  if (!user && isProtectedRoute) {
    return NextResponse.redirect(new URL("/login", request.url))
  }
  if (user && isAuthPage) {
    return NextResponse.redirect(new URL("/schools", request.url))
  }

  return supabaseResponse
}
```

---

## src/middleware.ts

Ponto de entrada do Next.js middleware:

```typescript
import { type NextRequest } from "next/server"
import { updateSession } from "@/utils/supabase/middleware"

export async function middleware(request: NextRequest) {
  return await updateSession(request)
}

export const config = {
  matcher: [
    "/((?!_next/static|_next/image|favicon.ico|.*\\.(?:svg|png|jpg|jpeg|gif|webp)$).*)",
  ],
}
```

---

## Fluxo de Auth

```
1. Login → Supabase Auth cria sessão (cookies HttpOnly)
2. Request → Middleware refresha sessão automaticamente
3. Rota protegida sem sessão → /login
4. /login com sessão → /schools
5. Logout → supabase.auth.signOut() limpa cookies
```

---

## Tipos de Client

| Tipo | Onde | Para quê |
|------|------|----------|
| `createBrowserClient` | Client Components | Queries do lado do cliente |
| `createServerClient` | Server Components/Actions | Queries do lado do servidor |
| Middleware client | middleware.ts | Refresh de sessão + proteção |

---

## db/ (CRUD por Domínio)

Todos os arquivos em `db/` importam o browser client:

```typescript
import { createClient } from "@/utils/supabase/client"
const supabase = createClient()
```

### helpers.ts
| Função | Descrição |
|--------|-----------|
| `generateId()` | UUID v4 |
| `toCamel<T>(row)` | snake_case → camelCase |
| `toSnake(obj)` | camelCase → snake_case |

### auth.ts
| Função | Descrição |
|--------|-----------|
| `login(email, password)` | Autentica + cria sessão Supabase Auth |
| `logout()` | Encerra sessão |
| `getSession()` | Retorna AuthUser da sessão atual |

### users.ts
| Função | Descrição |
|--------|-----------|
| `getUsers()` | Lista todos |
| `getUsersByRole(role)` | Filtra por role |
| `createUser(data)` | Cria user |
| `getSchoolsByUser(userId)` | Escolas vinculadas ao user |
| `getUsersBySchool(schoolId)` | Users vinculados à escola |
| `addUserSchool(userId, schoolId)` | Vincula |
| `removeUserSchool(userId, schoolId)` | Desvincula |
| `replaceUserSchools(userId, schoolIds)` | Substitui vinculações |

### schools.ts
| Função | Descrição |
|--------|-----------|
| `getSchools()` | Lista todas |
| `getSchoolsByYear(year)` | Filtra por ano |
| `getAcademicYears()` | Anos letivos |
| `createSchool(data)` | Cria |
| `updateSchool(id, data)` | Atualiza |
| `deleteSchool(id)` | Remove (cascade) |

### Outros módulos
- `classes.ts` - CRUD turmas
- `rooms.ts` - CRUD salas
- `students.ts` - CRUD alunos
- `schedules.ts` - CRUD horários
- `orientador-schedules.ts` - CRUD agenda por escola
- `segment-configs.ts` - CRUD config segmento
- `items.ts` - CRUD itens
- `nap-items.ts` - CRUD itens NAP
- `agenda.ts` - CRUD agenda orientador
- `tbr.ts` - CRUD categorias + equipes TBR

---

## Padrões de Uso

### useEffect com dados async
```typescript
useEffect(() => {
  async function load() {
    const data = await getSchools()
    setSchools(data)
  }
  load()
}, [])
```

### CRUD com toast
```typescript
async function handleSave() {
  setSaving(true)
  if (editingItem) {
    await updateItem(editingItem.id, { name })
    toast.success("Atualizado!")
  } else {
    await createItem({ name, category: "tapete", naps: [] })
    toast.success("Criado!")
  }
  setSaving(false)
  await refresh()
}
```

### Pivot user ↔ school
```typescript
// Orientador vê suas escolas
const mySchoolIds = await getSchoolsByUser(userId)
const mySchools = allSchools.filter((s) => mySchoolIds.includes(s.id))

// Vincular ao criar
const user = await createUser({ ...role: "orientador" })
await addUserSchool(user.id, schoolId)

// Atualizar vinculações
await replaceUserSchools(userId, [schoolId1, schoolId2])
```
