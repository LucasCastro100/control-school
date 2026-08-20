# Alterações - 19/08/2026

## Resumo

Correções de schema/auth, simplificação da página de horários e implementação completa de páginas de autenticação (esqueci a senha, redefinir senha, perfil, mudar senha).

## O que mudou

### 1. Correção do schema Supabase
- **Arquivo:** `front/supabase-schema.sql`
- **Fix:** CHECK constraint da tabela `users` atualizado para incluir role `'escola'` (antes só permitia `admin`, `orientador`, `professor`)
- **Novo arquivo:** `front/seed.sql` — script de reset do banco + seed do admin (excluído do git via `.gitignore`)

### 2. Página de horários da escola simplificada
- **Arquivo:** `front/src/app/(dashboard)/schools/[id]/schedules/page.tsx`
- **Antes:** Tinha dialog de criação/edição de agenda do orientador com selector de orientador
- **Agora:** Página somente visualização dos horários de aula das turmas
- Botão "Novo Horário" removido
- Seção "Agenda do Orientador" removida
- Orientador auto-detectado via `user_schools` (quando necessário)

### 3. Formulário de horários da turma simplificado
- **Arquivo:** `front/src/app/(dashboard)/schools/[id]/classes/[classId]/schedules/page.tsx`
- Campo "Disciplina" removido (todas as aulas são Educação Tecnológica)
- Subject salvo automaticamente como `"Educação Tecnológica"`
- Dropdown de turma no lugar do campo de texto livre

### 4. Correções gerais
- **Arquivo:** `front/src/lib/types.ts` — tipo `User.role` agora inclui `'escola'`
- **Arquivo:** `front/src/app/login/page.tsx` — labels corrigidos ("E-mail", placeholder genérico)
- **Arquivo:** `front/src/app/(dashboard)/schools/[id]/classes/page.tsx` — label "General Schedules" traduzido para "Horário geral"

### 5. Página "Esqueci a senha" (`/forgot-password`)
- **Novo arquivo:** `front/src/app/forgot-password/page.tsx`
- Formulário com campo de email
- Chama `supabase.auth.resetPasswordForEmail()` para enviar link de redefinição
- Tela de confirmação com mensagem "Email enviado"

### 6. Página "Redefinir senha" (`/reset-password`)
- **Novo arquivo:** `front/src/app/reset-password/page.tsx`
- Acessada via link do email (Supabase redireciona com token de recovery)
- Detecta evento `PASSWORD_RECOVERY` via `onAuthStateChange`
- Formulário com nova senha + confirmação (validação Zod)
- Chama `supabase.auth.updateUser({ password })`
- Redireciona para `/login` após sucesso

### 7. Página "Meu Perfil" (`/profile`)
- **Novo arquivo:** `front/src/app/(dashboard)/profile/page.tsx`
- Exibe email (somente leitura), nome editável e perfil (role)
- Chama `supabase.auth.updateUser()` para atualizar `user_metadata`
- Sincroniza também a tabela `users` via `updateUser()`
- Botão "Mudar senha" linka para `/profile/change-password`

### 8. Página "Mudar senha" (`/profile/change-password`)
- **Novo arquivo:** `front/src/app/(dashboard)/profile/change-password/page.tsx`
- Formulário com nova senha + confirmação (validação Zod)
- Chama `supabase.auth.updateUser({ password })`
- Toast de sucesso e redirecionamento para `/profile`

### 9. Funções de auth adicionadas
- **Arquivo:** `front/src/lib/db/auth.ts`
- `resetPassword(email)` — envia email de redefinição
- `updatePassword(password)` — atualiza senha do usuário
- `updateProfile({ name })` — atualiza nome no Auth e na tabela `users`
- **Arquivo:** `front/src/lib/db/index.ts` — novas funções exportadas

### 10. Atualizações no sidebar e login
- **Arquivo:** `front/src/components/app-sidebar.tsx` — link "Meu Perfil" adicionado no footer
- **Arquivo:** `front/src/app/login/page.tsx` — link "Esqueci a senha" abaixo do campo de senha

### 11. Middleware atualizado
- **Arquivo:** `front/src/utils/supabase/middleware.ts`
- `/forgot-password` e `/reset-password` adicionados como páginas públicas (não redirecionam para login)

### 12. Gitignore atualizado
- **Arquivo:** `.gitignore` — `seed.sql` adicionado (excluído do repositório)

## Arquivos criados

```
front/seed.sql
front/src/app/forgot-password/page.tsx
front/src/app/reset-password/page.tsx
front/src/app/(dashboard)/profile/page.tsx
front/src/app/(dashboard)/profile/change-password/page.tsx
```

## Arquivos modificados

```
front/supabase-schema.sql
front/src/lib/types.ts
front/src/lib/db/auth.ts
front/src/lib/db/index.ts
front/src/utils/supabase/middleware.ts
front/src/app/login/page.tsx
front/src/components/app-sidebar.tsx
front/src/app/(dashboard)/schools/[id]/schedules/page.tsx
front/src/app/(dashboard)/schools/[id]/classes/[classId]/schedules/page.tsx
front/src/app/(dashboard)/schools/[id]/classes/page.tsx
.gitignore
```

## Como testar

### Autenticação
1. Acessar `/login` e clicar em "Esqueci a senha"
2. Inserir email e verificar caixa de entrada (email do Supabase Auth)
3. Clicar no link do email e definir nova senha
4. Login com a nova senha

### Perfil
1. Fazer login
2. Clicar em "Meu Perfil" no sidebar
3. Editar nome e salvar
4. Clicar em "Mudar senha" e alterar a senha

### Schema
1. Rodar o `seed.sql` no Supabase SQL Editor para atualizar o CHECK constraint
2. Verificar que a role `'escola'` agora é aceita na tabela `users`
