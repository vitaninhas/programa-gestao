# Pedra Papel Tesoura · Gestão

Aplicação web de gestão (clientes, encomendas, fornecedores, produtos, produção, financeiro, relatórios e documentos).

## Stack

- **Frontend:** React 18 + TypeScript + Vite + Tailwind CSS
- **Backend/Base de dados:** Supabase (Postgres + Auth + Storage)
- **Ícones:** lucide-react

## Pré-requisitos

- Node.js 18 ou superior
- npm

## Instalação local

```bash
npm install
```

## Variáveis de ambiente

O ficheiro `.env` já vem com as credenciais do Supabase pré-preenchidas:

```
VITE_SUPABASE_URL=...
VITE_SUPABASE_ANON_KEY=...
```

Se precisares de apontar para outro projeto Supabase, substitui estes valores.

## Comandos

- `npm run dev` — inicia o servidor de desenvolvimento
- `npm run build` — compila para produção (pasta `dist/`)
- `npm run preview` — pré-visualiza a build de produção localmente
- `npm run typecheck` — verifica tipos TypeScript
- `npm run lint` — corre o ESLint

## Deploy

A pasta `dist/` gerada por `npm run build` é estática e pode ser publicada em qualquer hosting estático (Netlify, Vercel, Cloudflare Pages, GitHub Pages, etc.).

O ficheiro `dist/_redirects` já está incluído para garantir que as rotas (em hash) funcionam em qualquer plataforma. Em Netlify/Cloudflare Pages isto é automático; noutros hostings pode ser necessário configurar o fallback para `index.html`.

## Base de dados

As migrações SQL estão em `supabase/migrations/`. O projeto Supabase associado já tem todas as migrações aplicadas. Para um projeto novo, corre as migrações por ordem.
