# Baseline de Qualidade e Ambientes — DIYSPUR

**Data do registro:** 11 de outubro de 2026  
**Repositórios:** `diyspur-cloud/app` (branch `fix/audit-regression-2026-10`) e `diyspur-cloud/db` (branch `docs/audit-data-hygiene-2026-10`)  
**Commit base app:** `422061d` (após reconciliação SDD)  
**Commit base db:** `b1b49aa` (reconciliação de produção e integridade B01-B19)

---

## 1. Ambientes e URLs

| Ambiente | URL Base | Finalidade | Política de Mutations |
|---|---|---|---|
| **Local / Dev** | `http://localhost:3000` | Desenvolvimento e testes unitários | Permitidas localmente |
| **CI / Automated** | `http://127.0.0.1:3000` | Playwright E2E e build validation | Fixtures efêmeras em memória |
| **Staging / Preview** | `https://staging.diyspur.app` (controlado) | Testes E2E autenticados, k6 load test | Permitidas apenas com conta QA |
| **Produção** | `https://diyspur.vercel.app` | Ambiente ao vivo para usuários | **STRICT READ-ONLY** (proibido testes de escrita) |

---

## 2. Mapa de Rotas do Sistema

### 2.1 Rotas Públicas (Indexáveis / SEO)
- `/` — Homepage / Proposta de valor
- `/livros` — Catálogo geral e busca de obras
- `/livros/[slug]` — Ficha técnica da obra, capítulos e temporadas
- `/calendario` — Agenda de debates com fuso de Brasília e links Google Agenda/ICS
- `/clubes` — Listagem pública de clubes ativos
- `/comunidade` — Conversas e comentários com moderação e spoiler disclosure
- `/privacidade` — Política de Privacidade e Direitos dos Titulares (LGPD)
- `/termos` — Termos de Uso e regras de convivência

### 2.2 Rotas Privadas (Protegidas por Sessão / Noindex)
- `/diario` — Registros privados de leitura (redireciona para `/entrar?redirect=%2Fdiario&reason=login-required`)
- `/perfil` — Perfil do usuário, conquistas, XP e histórico de leitura
- `/historico` — Histórico detalhado de leituras do leitor
- `/notificacoes` — Central de notificações e avisos
- `/minha-fila` — Fila pessoal de leitura
- `/metas` — Metas anuais e mensais de leitura
- `/clubes/novo` — Formulário de criação de novo clube
- `/admin` — Painel editorial e moderação

---

## 3. Comandos de Verificação do Baseline

```bash
# Frontend
npm run lint          # ESLint 9 sem warnings impeditivos
npm run typecheck     # TypeScript 5.9 com strict mode
npm test              # Vitest suíte unitária completa (12 arquivos, 45 testes)
npm run build         # Next.js 16 com Turbopack (32 rotas estáticas/dinâmicas)
npm run test:e2e      # Playwright Chromium E2E
npm run audit         # Zero vulnerabilidades de produção

# Backend
python3 - <<'PY'      # Validação do parser de 85 migrations SQL com pglast
from pathlib import Path
from pglast import parse_sql
for p in Path('supabase/migrations').rglob('*.sql'): parse_sql(p.read_text())
PY
npm test --prefix scripts/security-smoke  # PGlite security smoke + business rules
```

---

## 4. Métricas de Core Web Vitals (Referência do Relatório de Auditoria)

| Métrica | Meta Recomendada | Baseline Observado (Desktop) | Baseline Observado (Mobile) | Status |
|---|---|---|---|---|
| **LCP (Largest Contentful Paint)** | ≤ 2.5s | 1.8s | 2.4s | PASS |
| **INP (Interaction to Next Paint)** | ≤ 200ms | 45ms | 85ms | PASS |
| **CLS (Cumulative Layout Shift)** | ≤ 0.1 | 0.02 | 0.04 | PASS |
| **FCP (First Contentful Paint)** | ≤ 1.8s | 1.1s | 1.6s | PASS |
| **TTFB (Time to First Byte)** | ≤ 800ms | 320ms (Edge Cache) | 480ms | PASS |

---

## 5. Status dos Achados da Auditoria

| Achado | Prioridade | Status | Evidência de Fechamento |
|---|---|---|---|
| Card residual `QA RETESTE 20261010` em `/clubes` | P0 | PASS | Query em `getVisibleClubs()` filtra `.is('archived_at', null)`; script `inspect-qa-residual.sql` preparado |
| Headers defensivos e CSP Report-Only | P0 | PASS | `X-Frame-Options`, `X-Content-Type-Options`, `Referrer-Policy` e `Content-Security-Policy-Report-Only` ativos |
| Allowlist de imagens remota irrestrita | P0/P1 | PASS | Wildcard `**` removido; permitidos apenas Supabase, Record e Amazon verificados |
| Redirect contextual em `/diario` | P1 | PASS | Redireciona com `?redirect=%2Fdiario&reason=login-required`; mensagem visível com `role="status"` |
| Clareza de encontros online sem link | P1 | PASS | Renderiza aviso da equipe informando publicação de instruções em vez de link quebrado |
| Schema `Book` e `Event` | P2 | PASS | JSON-LD estruturado na ficha de livro e agenda com serialização sanitizada anti-XSS |
| Canonicals absolutos por rota | P2 | PASS | Canonical global retirado do layout raiz; cada rota pública declara seu canonical |
| Sitemap dinâmico higienizado | P2 | PASS | Rota privada `/ranking` excluída; capítulos filtrados por `is_draft = false`; `/clubes` incluído |
| Assets sociais `og-image.png` e `logo.png` | P2 | PASS | Criados assets reais de 1200x630 e 512x512 em `public/` |
| Busca com debounce no catálogo | P2 | PASS | Componente `BookSearch` implementado com debounce de 300ms, transição e fallback HTML nativo |
| Exportação de dados LGPD | P2/P3 | PASS | Endpoint `api/account/export/route.ts` autenticado por claims, no-store e attachment JSON |
| Suíte de regressão Playwright e CI | P2 | PASS | 9 arquivos de testes cobrindo REG-01 a REG-18 e workflow `.github/workflows/ci-regression.yml` |
