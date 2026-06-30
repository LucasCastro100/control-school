# Control School

Sistema de gerenciamento multi-escolar para controle de turmas, salas, alunos, horários e itens pedagógicos.

## Funcionalidades

- **Escolas** — Cadastro e gerenciamento de múltiplas escolas
- **Turmas** — Organização de turmas por NAPs (Núcleo de Ação Pedagógica) com anos segmentados
- **Salas** — Gerenciamento de salas por turma
- **Alunos** — Cadastro de alunos organizados por sala
- **Horários** — Grade de horários por turma/sala com dias da semana
- **Itens** — Cadastro global de itens (tapetes e tecnologias) com vínculo por NAP e quantidade por escola

## Tecnologias

- [Next.js 16](https://nextjs.org)
- [React 19](https://react.dev)
- [TypeScript](https://www.typescriptlang.org)
- [Tailwind CSS 4](https://tailwindcss.com)
- [shadcn/ui](https://ui.shadcn.com)
- [Lucide React](https://lucide.dev) (ícones)
- Dados armazenados em localStorage via funções CRUD

## Estrutura

```
control-schools/
├── front/          # Aplicação Next.js
│   ├── src/
│   │   ├── app/           # Rotas e páginas
│   │   ├── components/    # Componentes React
│   │   └── lib/           # Tipos, DB (localStorage) e utilitários
│   ├── package.json
│   └── ...
└── back/           # Backend (futuro)
```

## Executar

```bash
cd front
pnpm install
pnpm dev
```

Abra [http://localhost:3000](http://localhost:3000) no navegador.
