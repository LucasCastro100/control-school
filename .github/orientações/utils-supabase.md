# Utils e Middlewares - Supabase

## Estrutura de Diretórios

```
src/lib/
├── supabase.ts              # Cliente Supabase (middleware central)
├── types.ts                 # Interfaces TypeScript
├── db.ts                    # Barrel file (re-exporta tudo de db/)
└── db/
    ├── index.ts             # Re-exports centralizados + exportStorageData/importStorageData
    ├── helpers.ts           # Funções utilitárias (toCamel, toSnake, generateId)
    ├── auth.ts              # Login, logout, getAuthUser
    ├── users.ts             # CRUD users + pivot user_schools
    ├── schools.ts           # CRUD schools + queries auxiliares
    ├── classes.ts           # CRUD classes
    ├── rooms.ts             # CRUD rooms
    ├── students.ts          # CRUD students
    ├── schedules.ts         # CRUD schedules
    ├── orientador-schedules.ts  # CRUD orientador_schedules
    ├── segment-configs.ts   # CRUD segment_configs
    ├── items.ts             # CRUD items
    ├── nap-items.ts         # CRUD nap_items
    ├── agenda.ts            # CRUD agenda
    └── tbr.ts               # CRUD tbr_categories + tbr_teams
```

---

## `supabase.ts` - Cliente Central

O cliente Supabase é a camada middleware que conecta o app ao banco de dados.

```typescript
import { createClient } from "@supabase/supabase-js"

const supabaseUrl = process.env.NEXT_PUBLIC_SUPABASE_URL!
const supabaseAnonKey = process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!

export const supabase = createClient(supabaseUrl, supabaseAnonKey)
```

**Variáveis de ambiente** (em `.env.local`):
- `NEXT_PUBLIC_SUPABASE_URL` - URL do projeto Supabase
- `NEXT_PUBLIC_SUPABASE_ANON_KEY` - Chave pública (anon key)

**Uso:** Todos os arquivos em `db/` importam `supabase` deste módulo para fazer queries.

---

## `db/helpers.ts` - Funções Utilitárias

| Função | Descrição |
|--------|-----------|
| `generateId()` | Gera UUID v4 via `crypto.randomUUID()` |
| `toCamel<T>(row)` | Converte snake_case (banco) → camelCase (JS) |
| `toSnake(obj)` | Converte camelCase (JS) → snake_case (banco) |

### Exemplo de conversão:
```typescript
// Banco retorna: { first_name: "João", created_at: "2026-01-01" }
// toCamel converte: { firstName: "João", createdAt: "2026-01-01" }
```

**Padrão em todas as queries:**
```typescript
const { data } = await supabase.from("tabela").select("*")
return (data ?? []).map(toCamel<MinhaType>)
```

---

## `db/auth.ts` - Autenticação

| Função | Tipo | Descrição |
|--------|------|-----------|
| `login(email, password)` | async | Busca user na tabela `users` + fallback escola na tabela `schools` |
| `logout()` | sync | Remove sessão do localStorage |
| `getAuthUser()` | sync | Lê sessão do localStorage |

### Fluxo de Login:

```
1. Admin → hardcodado (admin@gmail.com / mudar123)
2. Orientador/Professor → busca tabela "users" WHERE email + password
3. Escola → busca tabela "schools" WHERE email + password
```

**Sessão:** Mantida em `localStorage` com chave `control-schools:auth`.

**Tipo AuthUser armazenado:**
```typescript
{
  email: string
  name: string
  role: "admin" | "orientador" | "professor" | "escola"
  userId?: string    // ID na tabela users (para orientador/professor)
  schoolId?: string  // ID da escola (para role "escola")
}
```

---

## `db/users.ts` - CRUD Usuários + Pivot

### Tabela `users`

| Função | Descrição |
|--------|-----------|
| `getUsers()` | Lista todos os users |
| `getUsersByRole(role)` | Filtra por role (admin/orientador/professor) |
| `getUser(id)` | Busca user por ID |
| `createUser(data)` | Cria user (password padrão: "mudar123") |
| `updateUser(id, data)` | Atualiza user |
| `deleteUser(id)` | Remove user |

### Tabela pivot `user_schools`

| Função | Descrição |
|--------|-----------|
| `getSchoolsByUser(userId)` | Retorna IDs das escolas vinculadas ao user |
| `getUsersBySchool(schoolId)` | Retorna IDs dos users vinculados à escola |
| `addUserSchool(userId, schoolId)` | Vincula user a escola |
| `removeUserSchool(userId, schoolId)` | Desvincula user da escola |
| `replaceUserSchools(userId, schoolIds)` | Substitui todas as vinculações de um user |

### Exemplo de uso:
```typescript
// Orientador vê só suas escolas
const schoolIds = await getSchoolsByUser(orientadorUserId)
const mySchools = allSchools.filter((s) => schoolIds.includes(s.id))

// Vincular orientador a escola ao criar
await createUser({ name: "João", email: "joao@email.com", role: "orientador", ... })
await addUserSchool(newUser.id, schoolId)
```

---

## `db/schools.ts` - CRUD Escolas

| Função | Descrição |
|--------|-----------|
| `getSchools()` | Lista todas as escolas |
| `getSchoolsByYear(year)` | Filtra por ano de criação |
| `getSchoolYears()` | Retorna anos únicos (criação + turmas) |
| `getAcademicYears()` | Retorna anos letivos (turmas + nap_items) |
| `getSchool(id)` | Busca escola por ID |
| `createSchool(data)` | Cria escola |
| `updateSchool(id, data)` | Atualiza escola |
| `deleteSchool(id)` | Remove escola (cascade: classes, rooms, etc.) |

**Nota:** A tabela `schools` NÃO tem mais `orientador_id`. O vínculo é feito via tabela pivot `user_schools`.

---

## `db/classes.ts` - CRUD Turmas

| Função | Descrição |
|--------|-----------|
| `getClasses()` | Lista todas as turmas |
| `getClassesBySchool(schoolId)` | Turmas de uma escola |
| `getClass(id)` | Busca turma por ID |
| `getClassesBySchoolAndYear(schoolId, year)` | Turmas de uma escola em um ano |
| `createClass(data)` | Cria turma |
| `updateClass(id, data)` | Atualiza turma |
| `deleteClass(id)` | Remove turma |

---

## `db/rooms.ts` - CRUD Salas

| Função | Descrição |
|--------|-----------|
| `getRooms()` | Lista todas as salas |
| `getRoomsByClass(classId)` | Salas de uma turma |
| `getRoom(id)` | Busca sala por ID |
| `createRoom(data)` | Cria sala |
| `updateRoom(id, data)` | Atualiza sala |
| `deleteRoom(id)` | Remove sala |

---

## `db/students.ts` - CRUD Alunos

| Função | Descrição |
|--------|-----------|
| `getStudents()` | Lista todos os alunos |
| `getStudentsByRoom(roomId)` | Alunos de uma sala |
| `createStudent(data)` | Cria aluno |
| `updateStudent(id, data)` | Atualiza aluno |
| `deleteStudent(id)` | Remove aluno |

---

## `db/schedules.ts` - CRUD Horários

| Função | Descrição |
|--------|-----------|
| `getSchedules()` | Lista todos os horários |
| `getSchedulesByClass(classId)` | Horários de uma turma |
| `getSchedulesByRoom(roomId)` | Horários de uma sala |
| `createSchedule(data)` | Cria horário |
| `updateSchedule(id, data)` | Atualiza horário |
| `deleteSchedule(id)` | Remove horário |

---

## `db/orientador-schedules.ts` - Agenda do Orientador por Escola

| Função | Descrição |
|--------|-----------|
| `getOrientadorSchedules()` | Lista todos |
| `getOrientadorSchedulesBySchool(schoolId, year)` | Filtra por escola + ano |
| `createOrientadorSchedule(data)` | Cria |
| `updateOrientadorSchedule(id, data)` | Atualiza |
| `deleteOrientadorSchedule(id)` | Remove |
| `deleteOrientadorSchedulesBySchool(schoolId)` | Remove todos de uma escola |

**Nota:** `orientador_id` referencia a tabela `users` (não mais `orientadores`).

---

## `db/segment-configs.ts` - Configurações de Segmento

| Função | Descrição |
|--------|-----------|
| `getSegmentConfigs(schoolId)` | Configs de uma escola |
| `getSegmentConfigsAll()` | Todas as configs |
| `getSegmentConfig(schoolId, segmentName)` | Busca específica |
| `upsertSegmentConfig(...)` | Cria ou atualiza (upsert) |
| `deleteSegmentConfig(id)` | Remove |
| `deleteSegmentConfigsBySchool(schoolId)` | Remove todas de uma escola |

---

## `db/items.ts` - CRUD Itens (Tapete/Tecnologia)

| Função | Descrição |
|--------|-----------|
| `getAllItems()` | Lista todos os itens |
| `getItem(id)` | Busca item por ID |
| `createItem(data)` | Cria item (naps é JSON stringificado) |
| `updateItem(id, data)` | Atualiza item |
| `deleteItem(id)` | Remove item |

**Nota:** Campo `naps` é armazenado como JSON string no banco.

---

## `db/nap-items.ts` - Itens por Segmento/NAP

| Função | Descrição |
|--------|-----------|
| `getNapItems(schoolId)` | Itens NAP de uma escola |
| `getAllNapItems()` | Todos os itens NAP |
| `getNapItemsBySchoolAndYear(schoolId, year)` | Filtra por escola + ano |
| `getNapItemsBySegment(schoolId, segmentName, year?)` | Filtra por segmento |
| `upsertNapItem(...)` | Cria ou atualiza |
| `deleteNapItem(id)` | Remove |
| `deleteNapItemsBySchool(schoolId)` | Remove todos de uma escola |
| `deleteNapItemsBySchoolAndYear(schoolId, year)` | Remove por escola + ano |

---

## `db/agenda.ts` - CRUD Agenda do Orientador

| Função | Descrição |
|--------|-----------|
| `getAgendaItems()` | Lista todos |
| `getAgendaItemsByOrientador(orientadorId)` | Filtra por orientador |
| `getAgendaItem(id)` | Busca por ID |
| `createAgendaItem(data)` | Cria |
| `updateAgendaItem(id, data)` | Atualiza |
| `deleteAgendaItem(id)` | Remove |

**Nota:** `orientador_id` referencia a tabela `users`.

---

## `db/tbr.ts` - Categorias + Equipes TBR

### Categorias
| Função | Descrição |
|--------|-----------|
| `getTbrCategories()` | Lista categorias |
| `getTbrCategory(id)` | Busca por ID |
| `createTbrCategory(data)` | Cria |
| `updateTbrCategory(id, data)` | Atualiza |
| `deleteTbrCategory(id)` | Remove |

### Equipes
| Função | Descrição |
|--------|-----------|
| `getAllTbrTeams()` | Lista todas |
| `getTbrTeamsBySchool(schoolId)` | Equipes de uma escola |
| `getTbrTeamsByCategory(categoryId)` | Equipes de uma categoria |
| `createTbrTeam(data)` | Cria |
| `deleteTbrTeam(id)` | Remove |
| `deleteTbrTeamsBySchool(schoolId)` | Remove todas de uma escola |
| `replaceTbrTeamsForSchool(schoolId, teams)` | Substitui todas as equipes de uma escola |

---

## `db/index.ts` - Barrel + Export/Import

### Re-exports
Todas as funções dos domínios são re-exportadas aqui para manter o import `@/lib/db` funcionando.

### Export/Import de Dados

| Função | Descrição |
|--------|-----------|
| `exportStorageData()` | Exporta todas as tabelas como JSON |
| `importStorageData(data)` | Importa JSON substituindo dados |
| `initStorage()` | No-op (compatibilidade) |

**Formato de export:**
```json
{
  "control-schools:schools": [...],
  "control-schools:classes": [...],
  "control-schools:users": [...],
  ...
}
```

**Tabelas mapeadas no import:**
```typescript
{
  "control-schools:schools": "schools",
  "control-schools:users": "users",
  "control-schools:orientador-schedules": "orientador_schedules",
  ...
}
```

---

## Padrões de Uso

### Padrão 1: useEffect com dados async
```typescript
useEffect(() => {
  async function load() {
    const data = await getSchools()
    setSchools(data)
  }
  load()
}, [])
```

### Padrão 2: CRUD com toast
```typescript
async function handleSave() {
  setSaving(true)
  if (editingItem) {
    await updateItem(editingItem.id, { name })
    toast.success("Item atualizado!")
  } else {
    await createItem({ name, category: "tapete", naps: [] })
    toast.success("Item criado!")
  }
  setSaving(false)
  await refresh()
}
```

### Padrão 3: Refresh
```typescript
async function refresh() {
  const data = await getSchools()
  setSchools(data)
}
```

### Padrão 4: Pivot user ↔ school
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

---

## Schema SQL

O schema completo está em `supabase-schema.sql` na raiz do projeto frontend.

**Tabelas:**
- `users` - Usuários unificados (admin/orientador/professor)
- `user_schools` - Pivot users ↔ schools
- `schools` - Escolas
- `classes` - Turmas
- `rooms` - Salas
- `students` - Alunos
- `schedules` - Horários
- `orientador_schedules` - Agenda do orientador por escola
- `segment_configs` - Configurações de segmento
- `items` - Itens (tapete/tecnologia)
- `nap_items` - Itens por NAP/segmento
- `agenda` - Agenda do orientador
- `tbr_categories` - Categorias TBR
- `tbr_teams` - Equipes TBR

**RLS (Row Level Security):** Habilitado em todas as tabelas com políticas permissivas (app controla acesso).
