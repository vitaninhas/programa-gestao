# Technical Project Summary — Pedra Papel Tesoura · Gestão

## 1. Stack

| Layer | Technology |
|---|---|
| Frontend | React 18 + TypeScript + Vite 5 |
| Styling | Tailwind CSS 3 + custom CSS variables (dark/light theme) |
| Icons | lucide-react 0.446 |
| Backend / Database | Supabase (Postgres + Auth + Storage) |
| Build output | Static SPA (`dist/`) — deployable to any static host |

---

## 2. Folder Structure

```
project/
├── public/                   # Static assets copied verbatim to dist/
│   ├── favicon.svg
│   ├── _redirects            # Netlify/Cloudflare SPA fallback rule
│   └── robots.txt
├── src/
│   ├── App.tsx               # Root component: auth guard + hash-based router
│   ├── main.tsx              # React entry point
│   ├── index.css             # Global styles, CSS variables, Tailwind layers
│   ├── vite-env.d.ts
│   ├── components/
│   │   ├── EditarEncomendaModal.tsx   # Full order-edit modal (items, payments, timeline)
│   │   ├── EditorVisual.tsx           # Drag-and-drop template visual editor (canvas)
│   │   ├── GerarDocumentoModal.tsx    # Generate PDF document from a template
│   │   ├── Sidebar.tsx                # Left navigation (desktop + mobile drawer)
│   │   ├── Topbar.tsx                 # Top bar: search, theme toggle, user menu
│   │   └── ui/
│   │       ├── ConfirmDialog.tsx      # Reusable yes/no confirmation dialog
│   │       ├── Modal.tsx              # Base modal wrapper
│   │       ├── PageHeader.tsx         # Page title + subtitle block
│   │       ├── States.tsx             # Loading / empty / error state components
│   │       └── Typeahead.tsx          # Searchable dropdown input
│   ├── lib/
│   │   ├── documentos.ts     # Document generation logic (build HTML from template elements)
│   │   ├── elementos.ts      # Helpers for visual editor elements (positions, defaults)
│   │   ├── financeiro.ts     # Financial calculation helpers (totals, margins, etc.)
│   │   ├── supabase.ts       # Supabase singleton client (reads VITE_ env vars)
│   │   ├── theme.tsx         # ThemeProvider + useTheme hook (dark/light, persisted in localStorage)
│   │   ├── types.ts          # All TypeScript interfaces for every DB entity
│   │   ├── useConfig.ts      # Singleton hook: loads + caches app_config row; exposes logo helpers
│   │   └── utils.ts          # Formatting helpers (currency, dates, order numbers, etc.)
│   └── pages/
│       ├── Dashboard.tsx           # KPI overview: revenue, orders, production, stock
│       ├── Clientes.tsx            # Customer list with search, filters, stats
│       ├── Fornecedores.tsx        # Supplier list
│       ├── Produtos.tsx            # Product catalogue with variants, tiered pricing
│       ├── Compras.tsx             # Purchase orders list (from suppliers)
│       ├── Encomendas.tsx          # Customer order list with kanban-style status
│       ├── EncomendaDetalhe.tsx    # Full order detail view (items, payments, documents, timeline)
│       ├── EncomendasFornecedor.tsx# Supplier order notes list
│       ├── Producao.tsx            # Production board (per item, per order)
│       ├── Financeiro.tsx          # Financial module (income, expenses, movements, summary)
│       ├── Relatorios.tsx          # Reports (revenue, orders, products, clients, expenses)
│       ├── DocumentosModelos.tsx   # Document templates CRUD + visual editor launcher
│       ├── Configuracoes.tsx       # App configuration (company info, logos, signup code)
│       ├── Login.tsx               # Email/password login + signup with access code
│       └── forms/
│           ├── NovaEncomenda.tsx           # New customer order form
│           ├── NovaEncomendaFornecedor.tsx # New supplier order note form
│           ├── NovoCliente.tsx             # New/edit customer form
│           ├── NovoFornecedor.tsx          # New/edit supplier form
│           ├── NovoProduto.tsx             # New/edit product form (variants, tiered pricing, images)
│           └── NovaCompra.tsx              # New/edit purchase form (items, payments)
├── supabase/
│   └── migrations/           # SQL migration files (applied via Supabase MCP tool)
├── index.html
├── vite.config.ts
├── tailwind.config.js
├── tsconfig.app.json
├── package.json
├── README.md
└── TECHNICAL_SUMMARY.md
```

---

## 3. Routing

The app uses **client-side hash routing** (`window.location.hash`), not a router library. Navigation is handled by `navigate(pageKey)` in `App.tsx`, which sets both React state and the URL hash.

| Hash / PageKey | Component | Description |
|---|---|---|
| `#dashboard` | `Dashboard` | Home — KPIs and summaries |
| `#clientes` | `Clientes` | Customer list |
| `#novo-cliente` | `NovoCliente` | Create customer |
| `#fornecedores` | `Fornecedores` | Supplier list |
| `#novo-fornecedor` | `NovoFornecedor` | Create supplier |
| `#produtos` | `Produtos` | Product catalogue |
| `#novo-produto` | `NovoProduto` | Create / edit product |
| `#compras` | `Compras` | Purchase orders |
| `#nova-compra` | `NovaCompra` | Create purchase order |
| `#editar-compra` | `NovaCompra` (edit mode) | Edit purchase order |
| `#encomendas` | `Encomendas` | Customer orders |
| `#nova-encomenda` | `NovaEncomenda` | Create customer order |
| `#encomenda-detalhe` | `EncomendaDetalhe` | Order detail (id passed via React state) |
| `#encomendas-fornecedor` | `EncomendasFornecedor` | Supplier order notes |
| `#nova-encomenda-fornecedor` | `NovaEncomendaFornecedor` | Create supplier order note |
| `#producao` | `Producao` | Production board |
| `#financeiro` | `Financeiro` | Financial module |
| `#relatorios` | `Relatorios` | Reports |
| `#configuracoes` | `Configuracoes` | Settings |

> **Note:** Order detail and edit-purchase pages receive their record ID via React state (`detalheEncomendaId`, `editarCompraId`), not via the URL. Deep-linking to these pages after a hard refresh will land on the list view instead.

---

## 4. Authentication

- Supabase email + password auth (`signInWithPassword` / `signUp`).
- Sign-up is gated behind a configurable access code stored in `app_config.signup_code`.
- `App.tsx` guards the entire application: unauthenticated users see only `Login.tsx`.
- Session is read on mount via `supabase.auth.getSession()` and kept in sync with `onAuthStateChange`.

---

## 5. Database Tables (Supabase / Postgres)

All 26 tables have **RLS enabled**. All belong to the `public` schema.

| Table | Description |
|---|---|
| `app_config` | Single-row configuration: company info, logos, signup code, default document templates |
| `clientes` | Customers — contact details, aggregated stats (orders count, total spent) |
| `fornecedores` | Suppliers — contact, payment conditions |
| `produtos` | Products — categories, pricing, stock control, type (`simples`/`metro`/`servico`) |
| `variantes` | Product variants (colour, size, capacity) — own stock + reserved stock |
| `precos_escalao` | Tiered pricing per product (quantity breakpoints, supplier + sale price) |
| `encomendas` | Customer orders — status, priority, origin, delivery date, payment summary |
| `itens_encomenda` | Line items on an order (product/variant ref, personalisation, production notes) |
| `pagamentos` | Payments received for a customer order |
| `estados_encomenda` | Configurable order status list (name, colour, display order, production flag) |
| `cronologia` | Audit trail / timeline events per order |
| `producao` | Production records — one row per order item, tracks production status |
| `compras` | Purchase orders from suppliers — total, payment state |
| `itens_compra` | Line items on a purchase order |
| `pagamentos_compra` | Payments made for a purchase order |
| `notas_encomenda` | Supplier order notes (sent to supplier) |
| `notas_encomenda_itens` | Line items on a supplier order note |
| `catalogos_fornecedor` | Supplier catalogues / price lists (URL or uploaded file) |
| `despesas` | Expenses — category, amount, date, optional receipt attachment |
| `categorias_despesa` | Expense categories with colours (pre-seeded with 14 defaults) |
| `movimentos_financeiros` | Financial movement log (income + expense entries, auto-generated) |
| `vendas_avulso` | Standalone sales not linked to a customer order |
| `movimentos_stock` | Stock movement log (per variant — in/out/adjustment) |
| `documentos_modelos` | Document templates — full visual editor config + element layout as JSONB |
| `variaveis_customizadas` | User-defined text variables usable in document templates |
| `anexos` | File attachments — linked to any entity by (`entidade`, `entidade_id`) |

### Storage Bucket
- `anexos-encomendas` — stores document PDFs and order attachments. Public URLs are stored in the `anexos` table.

---

## 6. Supabase Edge Functions

**None deployed.** All Supabase interactions are direct client-side queries using the anon key. There are no server-side functions.

---

## 7. Applied Migrations (19 total)

| # | Filename | Summary |
|---|---|---|
| 1 | `create_initial_schema` | Core tables: clients, suppliers, products, variants, orders, production, payments, stock movements |
| 2 | `add_encomenda_advanced_fields` | Extended order fields: personalisation, priority, origin, delivery, internal notes |
| 3 | `create_config_table` | `app_config` singleton row |
| 4 | `lockdown_rls_authenticated` | Set all policies to `TO authenticated` |
| 5 | `allow_anon_read_app_config` | Allow unauthenticated users to read logo/name for the login page |
| 6 | `add_signup_code` | `signup_code` + `signup_enabled` columns to `app_config` |
| 7 | `add_signup_code_verify_function` | `verify_signup_code` PL/pgSQL RPC function |
| 8 | `restrict_anon_app_config_columns` | Column-level SELECT grants for `anon` (logo, name, contact only) |
| 9 | `evolve_encomenda_personalization` | Extended item personalisation fields |
| 10 | `enhance_products_production_timeline` | Product type, tiered pricing, production status improvements |
| 11 | `align_estados_and_new_features` | `estados_encomenda` table, timeline/cronologia, `fechada` state |
| 12 | `create_catalogos_fornecedor` | Supplier catalogue table |
| 13 | `add_compras_payments_encomendas_notas_internas` | Purchases, purchase payments, supplier order notes, internal order notes |
| 14 | `add_variantes_cor_hex` | `cor_hex` column on `variantes` |
| 15 | `remove_iva_columns` | Removed VAT columns from orders (business decision) |
| 16 | `add_supplier_and_sale_prices_to_metro_tiers` | `preco_compra` + `preco_venda` on `precos_escalao` |
| 17 | `create_missing_financial_movements_table` | `movimentos_financeiros` table |
| 18 | `complete_app_config_fields_and_public_read` | All company/social/payment fields on `app_config`; corrected anon grants |
| 19 | `create_missing_document_and_financial_tables` | `documentos_modelos`, `variaveis_customizadas`, `categorias_despesa`, `vendas_avulso`; `categoria_id` + `anexo_url` on `despesas` |

---

## 8. Known Issues

### Bugs

| Severity | Location | Description |
|---|---|---|
| Low | `App.tsx` | Deep-linking to `#encomenda-detalhe` or `#editar-compra` after a hard page refresh renders nothing (ID is in React state, not URL). User lands on blank content until they navigate back to the list. |
| Low | `Typeahead.tsx` | Previously used `var(--surface)` — an undefined CSS variable — for the dropdown background, causing a transparent background. **Fixed in last session.** |

### ESLint Errors (build passes, lint fails)

The project builds and typechecks cleanly (`tsc --noEmit` exits 0), but `npm run lint` exits with errors:

| File | Issue |
|---|---|
| `EditarEncomendaModal.tsx` (lines 114, 145) | `@typescript-eslint/no-explicit-any` — two event handler parameters typed as `any` |
| `EditorVisual.tsx` (line 529+) | `@typescript-eslint/no-explicit-any` — drag event payload typed as `any` |
| `EditorVisual.tsx` (line 27) | `react-hooks/exhaustive-deps` — `elementos` initialised outside `useMemo`, causes `useCallback` dependency to be unstable (5 warnings) |
| `EditorVisual.tsx` (line 208) | `react-hooks/exhaustive-deps` — `calcGuides` missing from `useEffect` dependency array |

The warnings in `EditorVisual.tsx` related to `elementos` / `calcGuides` could cause subtle stale-closure bugs in the visual editor if elements change rapidly.

### Security Notes

- `app_config` has **both** `anon` and `authenticated` policies for INSERT/UPDATE/DELETE (duplicated from an early migration and a later migration). Functionally harmless because the table has only one row and the app never writes from the login screen, but the duplicate policies are redundant.
- All RLS policies on business tables use `USING (true)` / `WITH CHECK (true)` — correct for a **single-tenant** app (one organisation, all authenticated users share the same data). This would need to change if multi-tenancy were required.

---

## 9. Environment Variables

| Variable | Purpose |
|---|---|
| `VITE_SUPABASE_URL` | Supabase project URL |
| `VITE_SUPABASE_ANON_KEY` | Supabase anon (public) key |

Both are pre-populated in `.env` and baked into the frontend build at compile time.
