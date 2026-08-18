# Como Funciona o Sistema - Control School

## Visão Geral

O **Control School** é um sistema de gerenciamento escolar construído com **Next.js 16 + React 19 + TypeScript + Tailwind CSS + Supabase**.

## Hierarquia de Dados

```
Escola (School)
  └── Turma (Class) — vinculada a um NAP (1-4) e ano letivo
        └── Sala (Room)
              └── Aluno (Student)
        └── Horário (Schedule)

Orientador
  └── Horário do Orientador (OrientadorSchedule)
  └── Agenda (AgendaItem)

Item (catálogo global de tapetes/tecnologia)
  └── NapItem (vincula item a NAP/escola/ano)
```

## Fluxo de Cadastro

### 1. Criar Escola
- Preencha nome, endereço, estado, cidade, cor
- Opcionalmente vincule um orientador
- Para a escola ter login próprio, preencha email e senha

### 2. Criar Turmas
- Acesse Escolas → clique no ícone de livros na escola
- Selecione o **Ano Letivo** no filtro
- Clique em **Nova Turma**
- Escolha o **NAP** (1: Infantil, 2: Fundamental I, 3: Fundamental II, 4: Ensino Médio)
- Escolha o **Ano** dentro do segmento
- Digite o identificador (ex: "1 Ano A")

### 3. Criar Salas
- Na listagem de turmas, clique no ícone de porta 🚪
- Digite o nome da sala (ex: "1A")
- Informe a quantidade de alunos (cria automaticamente)

### 4. Gerenciar Itens
- Acesse **Items** no menu lateral
- Cadastre tapetes e tecnologia, vinculando a NAPs
- Na página de turmas, clique nos contadores de itens para ajustar quantidades por NAP

## Autenticação

| Usuário | Email | Senha | Acesso |
|---|---|---|---|
| Admin | admin@gmail.com | mudar123 | Tudo |
| Orientador | (cada orientador) | mudar123 (padrão) | Agenda |
| Escola | (email da escola) | (senha da escola) | Sua escola |

## Armazenamento de Dados

- **Produção:** Supabase (PostgreSQL)
- **Auth:** localStorage (hardcoded no código)
- **Backup local:** Exportar/Importar JSON (menu Dados na sidebar)

## Estrutura de Pastas

```
front/src/
├── app/
│   ├── (dashboard)/        ← Rotas protegidas com sidebar
│   │   ├── schools/        ← CRUD de escolas e turmas
│   │   ├── advisors/       ← CRUD de orientadores
│   │   ├── items/          ← Catálogo de itens
│   │   ├── tbr/            ← Categorias e equipes TBR
│   │   ├── all-schedules/  ← Horário geral
│   │   └── agenda/         ← Agenda do orientador
│   ├── login/              ← Página de login
│   └── api/                ← API routes (Next.js)
├── components/             ← Componentes reutilizáveis (shadcn/ui)
├── lib/
│   ├── db.ts              ← Todas as funções CRUD (Supabase)
│   ├── supabase.ts        ← Cliente Supabase
│   └── types.ts           ← Interfaces TypeScript
└── data/
    └── seed.json          ← Dados iniciais (seed)
```

## Hierarquia de Rotas

```
/schools                          → Lista de escolas
/schools/[id]/classes             → Turmas da escola
/schools/[id]/classes/[classId]/rooms        → Salas da turma
/schools/[id]/classes/[classId]/rooms/[roomId]/students  → Alunos da sala
/schools/[id]/classes/[classId]/schedules    → Horários da turma
/schools/[id]/schedules           → Horários gerais da escola
/advisors                         → Orientadores
/items                            → Itens (tapetes/tecnologia)
/tbr                              → Categorias e equipes TBR
/all-schedules                    → Horário geral do sistema
/agenda                           → Agenda do orientador logado
```
