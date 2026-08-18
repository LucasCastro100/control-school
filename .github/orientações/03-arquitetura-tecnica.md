# Arquitetura Técnica

## Stack

| Camada | Tecnologia |
|---|---|
| Framework | Next.js 16 (App Router) |
| UI | React 19 + shadcn/ui + Tailwind CSS v4 |
| Linguagem | TypeScript 5 |
| Banco de Dados | Supabase (PostgreSQL) |
| ORM/Client | @supabase/supabase-js |
| Deploy | Vercel |
| Package Manager | pnpm 11 |

## Camada de Dados (db.ts)

O arquivo `src/lib/db.ts` é a **única interface** entre o app e o banco de dados.
Todas as 67 funções CRUD passam por ele.

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
School → Classes → Rooms → Students
       → Schedules
       → SegmentConfigs
       → NapItems
       → TbrTeams
       → OrientadorSchedules

Item → NapItems
TbrCategory → TbrTeams
Orientador → Agenda, desvincula de Schools
```

## Segurança

- **RLS (Row Level Security):** Habilitado em todas as tabelas com políticas permissivas
- **Autenticação:** Hardcoded no frontend (admin, orientador, escola)
- **Chaves:** `NEXT_PUBLIC_SUPABASE_URL` e `NEXT_PUBLIC_SUPABASE_ANON_KEY` no `.env.local`
- **NÃO commitar:** `.env.local` está no `.gitignore`

## Deploy

1. Push para o GitHub
2. Vercel faz deploy automático
3. Configurar variáveis de ambiente no painel do Vercel (Settings → Environment Variables)
   - `NEXT_PUBLIC_SUPABASE_URL`
   - `NEXT_PUBLIC_SUPABASE_ANON_KEY`
