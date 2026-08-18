# Alterações - 17/08/2026

## Resumo

Migração do armazenamento de dados de **localStorage** para **Supabase (PostgreSQL)**.

## O que mudou

### 1. Correção do bug de turmas
- **Arquivo:** `front/src/app/(dashboard)/schools/[id]/classes/page.tsx`
- **Problema:** Ao criar turma, o campo `year` recebia o nome do segmento ("1 ano") em vez do ano letivo ("2026")
- **Causa:** O estado `classYear` era sobrescrito pelo seletor de ano do segmento
- **Fix:** `const yearValue = editingClass ? editingClass.year : filterYear`

### 2. Migração para Supabase
- **Novo pacote:** `@supabase/supabase-js`
- **Novo arquivo:** `front/src/lib/supabase.ts` — cliente Supabase
- **Novo arquivo:** `front/supabase-schema.sql` — SQL para criar as 13 tabelas
- **Modificado:** `front/src/lib/db.ts` — todas as 67 funções CRUD agora usam Supabase em vez de localStorage

### 3. Configuração do pnpm 11
- **Modificado:** `front/pnpm-workspace.yaml` — configuração `allowBuilds` para sharp e unrs-resolver
- **Modificado:** `front/.npmrc` — limpo (pnpm 11 não lê mais settings aqui)

### 4. Remoção do seed.json
- **Removido:** `front/src/data/seed.json` — dados agora ficam no Supabase
- **Removido:** `front/src/app/api/seed-data/route.ts` — API não era necessária
- **Modificado:** `front/src/components/app-sidebar.tsx` — removido botão "Salvar no seed.json"

### 5. Variáveis de ambiente
- **Novo arquivo:** `front/.env.local` — credenciais do Supabase

## Arquivos criados

```
front/.env.local
front/src/lib/supabase.ts
front/supabase-schema.sql
```

## Arquivos modificados

```
front/package.json
front/pnpm-workspace.yaml
front/.npmrc
front/src/lib/db.ts
front/src/components/app-sidebar.tsx
front/src/app/(dashboard)/schools/[id]/classes/page.tsx
```

## Arquivos removidos

```
front/src/data/seed.json
front/src/app/api/seed-data/route.ts
```

## Como testar

1. Executar o SQL do `supabase-schema.sql` no Supabase SQL Editor
2. Configurar `.env.local` com URL e key do Supabase
3. `pnpm run dev` na pasta `front`
4. Acessar o app e criar dados (escolas, turmas, etc.)
5. Verificar no painel do Supabase que os dados aparecem nas tabelas
