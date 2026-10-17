# Critérios e Checklist de Release Readiness — DIYSPUR

**Data:** 11 de outubro de 2026  
**Status Atual:** PRONTO PARA HOMOLOGAÇÃO / PREVIEW (STAGING APROVADO)

---

## 1. Quality Gates de Segurança e Integridade de Dados

- [x] **Higiene de Fixtures Remotas**: O registro `QA RETESTE 20261010` foi isolado via query segura `.is('archived_at', null)` e o script read-only `db/scripts/ops/inspect-qa-residual.sql` está disponível para auditoria do operador.
- [x] **Privacidade de Clubes e RLS**: `/clubes` exclui clubes arquivados e `getClubPage(slug)` bloqueia clubes arquivados para não proprietários.
- [x] **Content Security Policy (CSP)**: Implementada em modo `Content-Security-Policy-Report-Only`, permitindo monitorar violações sem quebrar login, vídeos do YouTube, fontes e telemetria PostHog/Sentry.
- [x] **Headers Defensivos**: `X-Frame-Options: DENY`, `X-Content-Type-Options: nosniff`, `Referrer-Policy: strict-origin-when-cross-origin` e `Permissions-Policy` ativos em todas as rotas.
- [x] **Restrição de Imagens Remotas**: Wildcard `**` removido de `remotePatterns` em `next.config.mjs`; apenas domínios autorizados (Supabase, Record e Amazon) permanecem.
- [x] **Isolamento de Segredos**: Nenhuma chave `service_role` ou segredo de infraestrutura é exposta no cliente ou em variáveis públicas.

---

## 2. Quality Gates de Experiência e Acessibilidade (UX / SXO)

- [x] **Feedback Contextual no Login**: Ao tentar acessar `/diario` sem sessão, o redirecionamento carrega `?redirect=%2Fdiario&reason=login-required` e o formulário exibe aviso claro e acessível com `role="status"` e `aria-live="polite"`.
- [x] **Encontros Virtuais Claros**: Encontros marcados como online sem link publicado informam que as instruções de acesso serão liberadas pela equipe, evitando links quebrados ou especulação.
- [x] **Busca Dinâmica com Debounce**: Componente `BookSearch` implementado com 300ms de debounce, atualização de URL via transição sem recarregar e formulário HTML nativo como fallback.
- [x] **Acessibilidade de Spoilers**: Spoilers utilizam componentes acessíveis com navegação por teclado e foco visível.
- [x] **Responsividade Multi-Viewport**: Testado nos viewports 375px, 640px, 768px, 860px, 1024px e 1440px sem transbordamento ou rolagem horizontal.

---

## 3. Quality Gates de SEO Técnico e Descoberta (GEO / AIO)

- [x] **Canonicals Exclusivos**: Canonical global retirado do layout raiz; cada rota pública (`/`, `/livros`, `/livros/[slug]`, `/calendario`, `/comunidade`, `/clubes`) declara seu canonical exclusivo.
- [x] **Sitemap Dinâmico Higienizado**: Rota privada `/ranking` excluída do sitemap; capítulos em rascunho (`is_draft = true`) filtrados; rotas públicas essenciais incluídas.
- [x] **Dados Estruturados Schema.org**:
  - `Book` na ficha do livro (`/livros/[slug]`)
  - `Event` na agenda do clube (`/calendario`)
  - `Organization` e `WebSite` no layout raiz
  - Serialização protegida contra injeção de script via `replace(/</g, '\\u003c')`.
- [x] **Assets de Open Graph e Logo**: Arquivos `public/og-image.png` (1200x630) e `public/logo.png` (512x512) gerados e presentes no build.

---

## 4. Quality Gates de Performance e Infraestrutura

- [x] **Otimização de Imagens Next.js**: Atributo `unoptimized` removido da capa do livro na página de detalhes, habilitando o pipeline de conversão e compressão da Vercel.
- [x] **Pipeline de CI Frontend**: Criado `.github/workflows/ci-regression.yml` para execução automatizada de lint, typecheck, testes unitários, build e E2E.
- [x] **Scripts de Carga com Guard de Segurança**: Criados testes de carga k6 (`smoke.js`, `spike.js`, `stress.js`) com `guard.js` impedindo execução acidental em ambientes de produção.
- [x] **Conformidade LGPD**: Criado endpoint `GET /api/account/export` com exportação em anexo JSON dos dados do titular e proteção contra acesso cruzado.

---

## 5. Procedimento de Rollback

Caso seja detectada qualquer anomalia crítica após o deploy de frontend na Vercel:
1. No painel da Vercel, acionar **Instant Rollback** para o deployment anterior imediatamente.
2. Nenhuma alteração DDL destrutiva foi feita no banco Supabase remoto, garantindo compatibilidade reversa total com o código anterior.
