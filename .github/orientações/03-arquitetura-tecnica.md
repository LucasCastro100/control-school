# Arquitetura Técnica

## Stack

| Camada | Tecnologia |
|---|---|
| Framework | Next.js 16 (App Router) |
| UI | React 19 + shadcn/ui + Tailwind CSS v4 |
| Linguagem | TypeScript 5 |
| Banco de Dados | Supabase (PostgreSQL) |
| Auth | Supabase Auth (@supabase/ssr) |
| Deploy | Vercel |
| Package Manager | pnpm 11 |

## Camada de Supabase

### utils/supabase/ (Padrão Oficial)

```
utils/supabase/
├── client.ts       ← Browser client (createBrowserClient)
├── server.ts       ← Server client (createServerClient + cookies)
└── middleware.ts    ← Session refresh + proteção de rotas
```

### src/middleware.ts

Executa em CADA request do Next.js:
1. Refresh da sessão Supabase Auth
2. Redireciona para `/login` se rota protegida sem sessão
3. Redireciona para `/schools` se logado e acessando `/login`

### Matcher do Middleware

```typescript
// Executa em todas as rotas EXCETO:
// - _next/static, _next/image
// - favicon.ico
// - Arquivos públicos (svg, png, jpg, etc)
```

## Camada de Dados (db/)

O diretório `src/lib/db/` contém módulos CRUD organizados por domínio:

```
db/
├── helpers.ts     ← toCamel, toSnake, generateId
├── auth.ts        ← login, logout, getSession
├── users.ts       ← CRUD users + pivot user_schools
├── schools.ts     ← CRUD schools
├── classes.ts     ← CRUD classes
├── rooms.ts       ← CRUD rooms
├── students.ts    ← CRUD students
├── schedules.ts   ← CRUD schedules
├── orientador-schedules.ts
├── segment-configs.ts
├── items.ts
├── nap-items.ts
├── agenda.ts
├── tbr.ts
└── index.ts       ← Re-exports + exportStorageData/importStorageData
```

O `db.ts` na raiz de `lib/` é barrel file que re-exporta tudo:
```typescript
export * from "./db/index"
```

### Padrão de cada função:

```typescript
// CREATE
export async function createClass(data: Omit<Class, "id" | "createdAt">): Promise<Class>

// READ
export async function getClasses(): Promise<Class[]>
export async function getClassesBySchool(schoolId: string): Promise<Class[]>
export async function getClassesBySchoolAndYear(schoolId: string, year: string): Promise<Class[]>
export async function getClass(id: string): Promise<Class | undefined>

// UPDATE
export async function updateClass(id: string, data: Partial<Omit<Class, "id" | "createdAt">>): Promise<Class | undefined>

// DELETE (com cascade)
export async function deleteClass(id: string): Promise<void>
```

### Mapeamento JS ↔ PostgreSQL

| JavaScript (camelCase) | PostgreSQL (snake_case) |
|---|---|
| `schoolId` | `school_id` |
| `classId` | `class_id` |
| `roomId` | `room_id` |
| `dayOfWeek` | `day_of_week` |
| `startTime` | `start_time` |
| `endTime` | `end_time` |
| `orientadorId` | `orientador_id` |
| `createdAt` | `created_at` |
| `registrationNumber` | `registration_number` |
| `segmentName` | `segment_name` |
| `scheduleType` | `schedule_type` |

### Cascade Delete

Quando uma entidade pai é deletada, as filhas são removidas em cascata via `ON DELETE CASCADE` no Supabase:

```
User → UserSchools, OrientadorSchedules, Agenda
School → Classes, Schedules, SegmentConfigs, NapItems, TbrTeams, OrientadorSchedules
Class → Rooms, Schedules
Room → Students, Schedules
Item → NapItems
TbrCategory → TbrTeams
```

## Tabelas do Banco

| Tabela | Descrição |
|---|---|
| `users` | Usuários unificados (admin/orientador/professor) |
| `user_schools` | Pivot users ↔ schools (N:N) |
| `schools` | Escolas |
| `classes` | Turmas |
| `rooms` | Salas |
| `students` | Alunos |
| `schedules` | Horários |
| `orientador_schedules` | Agenda do orientador por escola |
| `segment_configs` | Config de segmento |
| `items` | Itens (tapete/tecnologia) |
| `nap_items` | Itens por NAP/escola/ano |
| `agenda` | Agenda do orientador |
| `tbr_categories` | Categorias TBR |
| `tbr_teams` | Equipes TBR |

## Segurança

- **RLS (Row Level Security):** Habilitado em todas as tabelas com políticas permissivas
- **Auth:** Supabase Auth com sessão via cookies (middleware refresha a cada request)
- **Chaves:** `NEXT_PUBLIC_SUPABASE_URL` e `NEXT_PUBLIC_SUPABASE_ANON_KEY` no `.env.local`
- **NÃO commitar:** `.env.local` está no `.gitignore`

## Deploy

1. Push para o GitHub
2. Vercel faz deploy automático
3. Configurar variáveis de ambiente no painel do Vercel (Settings → Environment Variables)
   - `NEXT_PUBLIC_SUPABASE_URL`
   - `NEXT_PUBLIC_SUPABASE_ANON_KEY`

## Comandos

```bash
pnpm install      # Instalar dependências
pnpm dev          # Rodar em desenvolvimento
pnpm run build    # Build de produção
pnpm start        # Rodar build de produção
```
