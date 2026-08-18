# Guia de Setup - Supabase

## 1. Criar conta no Supabase

1. Acesse [https://supabase.com](https://supabase.com)
2. Crie uma conta gratuita
3. Crie um novo projeto
4. Anote a **URL** e a **anon key** (Settings → API)

## 2. Criar as tabelas

1. No painel do Supabase, vá em **SQL Editor**
2. Cole todo o conteúdo do arquivo `front/supabase-schema.sql`
3. Clique em **Run** para executar

Isso cria as 14 tabelas:

| Tabela | Descrição |
|---|---|
| `users` | Usuários unificados (admin/orientador/professor) |
| `user_schools` | Pivot users ↔ schools |
| `schools` | Escolas cadastradas |
| `classes` | Turmas (vinculadas a escolas) |
| `rooms` | Salas (vinculadas a turmas) |
| `students` | Alunos (vinculados a salas) |
| `schedules` | Horários das turmas/salas |
| `orientador_schedules` | Horários dos orientadores por escola |
| `segment_configs` | Configuração por segmento NAP |
| `items` | Catálogo de itens (tapetes/tecnologia) |
| `nap_items` | Itens vinculados a NAPs por escola |
| `agenda` | Agenda dos orientadores |
| `tbr_categories` | Categorias TBR |
| `tbr_teams` | Equipes TBR por escola |

## 3. Configurar variáveis de ambiente

Crie o arquivo `front/.env.local`:

```
NEXT_PUBLIC_SUPABASE_URL=sua_url_aqui
NEXT_PUBLIC_SUPABASE_ANON_KEY=sua_chave_aqui
```

## 4. Rodar o projeto

```bash
cd front
pnpm install
pnpm dev
```

## 5. Comandos Úteis

```bash
pnpm dev          # Rodar em desenvolvimento
pnpm run build    # Build de produção
pnpm start        # Rodar build de produção
```
