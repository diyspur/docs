# Backlog de Melhorias e Roadmap Técnico — DIYSPUR

**Data:** 11 de outubro de 2026  
**Status:** Alinhado com as fases de engenharia P0–P3 do plano técnico.

---

## 1. Matriz de Tarefas por Prioridade

| ID | Prioridade | Descrição da Tarefa | Arquivos Principais | Esforço | Dependências | Critério de Pronto (DoD) |
|---|---|---|---|---|---|---|
| **TSK-01** | P0 | Isolar fixture residual `QA RETESTE 20261010` | `app/src/features/clubs/queries.ts`, `db/scripts/ops/inspect-qa-residual.sql` | Baixo | Acesso admin DB para arquivamento definitivo | Registro filtrado em `/clubes`; teste de regressão E2E verde |
| **TSK-02** | P0 | Content Security Policy Report-Only | `app/next.config.mjs` | Baixo | Mapeamento de origens legítimas | Header presente em respostas HTTP; zero falsos positivos |
| **TSK-03** | P0/P1 | Restringir hosts de imagens em `remotePatterns` | `app/next.config.mjs` | Baixo | Inventário de URLs de capas | Wildcard `**` removido; capas do catálogo carregam perfeitamente |
| **TSK-04** | P1 | Redirecionamento contextual em `/diario` | `app/src/app/diario/page.tsx`, `app/src/components/auth-form.tsx` | Baixo | Nenhuma | Leitor recebe motivo `login-required` e mensagem acessível `role="status"` |
| **TSK-05** | P1 | Clareza de encontros virtuais sem link | `app/src/app/calendario/page.tsx` | Baixo | Definição editorial | Encontro online exibe aviso explicativo em vez de link quebrado |
| **TSK-06** | P2 | Schema.org Book e Event estruturados | `app/src/app/livros/[slug]/page.tsx`, `app/src/app/calendario/page.tsx` | Médio | Tipos de banco | JSON-LD validado em testes e com serialização sanitizada anti-XSS |
| **TSK-07** | P2 | Canonicals absolutos e exclusivos por rota | `app/src/app/layout.tsx`, páginas públicas | Baixo | Auditoria de rotas | Cada página pública declara sua própria URL canônica |
| **TSK-08** | P2 | Higienização de sitemap dinâmico | `app/src/app/sitemap.ts` | Baixo | Filtro de capítulos | `/ranking` removido; capítulos rascunho ocultados; `/clubes` incluído |
| **TSK-09** | P2 | Criação de assets sociais `og-image` e `logo` | `app/public/og-image.png`, `app/public/logo.png` | Baixo | Design e resolução | Arquivos PNG reais de 1200x630 e 512x512 criados e retornando HTTP 200 |
| **TSK-10** | P2 | Componente `BookSearch` com debounce | `app/src/components/catalog/book-search.tsx`, `app/src/app/livros/page.tsx` | Médio | Next.js Suspense | Debounce de 300ms funcional com fallback acessível |
| **TSK-11** | P2 | Suíte Playwright E2E completa | `app/tests/e2e/*.spec.ts` | Médio | Chromium local/CI | 9 arquivos de testes cobrindo REG-01 a REG-18 |
| **TSK-12** | P2 | Workflow de CI no GitHub Actions | `app/.github/workflows/ci-regression.yml` | Baixo | Variáveis de teste | Pipeline executando lint, typecheck, tests, build e E2E |
| **TSK-13** | P2 | Scripts de teste de carga com k6 | `app/tests/load/*.js` | Médio | Staging aprovado | Smoke, spike e stress com guard bloqueando produção |
| **TSK-14** | P2/P3 | Endpoint de exportação LGPD | `app/src/app/api/account/export/route.ts` | Médio | Sessão Supabase | JSON completo do titular com download attachment e no-store |
