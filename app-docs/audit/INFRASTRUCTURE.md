# Arquitetura de Infraestrutura e Operações — DIYSPUR

**Data:** 11 de outubro de 2026  
**Ambiente Cloud:** Vercel (Frontend & Edge) + Supabase (PostgreSQL 17, Auth, Storage)

---

## 1. Topologia de Ambientes

```
[Desenvolvedor Local] ──▶ Localhost :3000 (SQLite/PGlite ou Supabase local)
[Pull Request GitHub] ──▶ GitHub Actions CI (Lint, Typecheck, Tests, Build, Playwright)
[Vercel Preview]      ──▶ Preview Deployment (Conectado a Staging Supabase)
[Vercel Production]   ──▶ Produção https://diyspur.vercel.app (Supabase Prod)
```

---

## 2. Headers de Segurança em Produção

Configurados de forma declarativa em `app/next.config.mjs`:
- `X-Content-Type-Options: nosniff` — Impede ataques de MIME-sniffing;
- `X-Frame-Options: DENY` — Bloqueia clickjacking ao proibir embedding em iframes de outros sites;
- `Referrer-Policy: strict-origin-when-cross-origin` — Protege privacidade em requisições de saída;
- `Permissions-Policy: camera=(), microphone=(), geolocation=()` — Desabilita APIs de sensores de hardware não utilizadas pela aplicação;
- `Content-Security-Policy-Report-Only` — Monitora violações de recursos sem indisponibilizar a plataforma antes do rollout estrito;
- `poweredByHeader: false` — Remove o cabeçalho `X-Powered-By: Next.js` para reduzir superfície de enumeração de versão.

---

## 3. Gestão de Variáveis e Segredos

- **Variáveis Públicas (`NEXT_PUBLIC_*`)**:
  - `NEXT_PUBLIC_SUPABASE_URL`: Endpoint da API Supabase (não confidencial);
  - `NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY`: Chave anônima / pública (sujeita a regras de RLS);
  - `NEXT_PUBLIC_SITE_URL`: URL canônica da aplicação (`https://diyspur.vercel.app`).
- **Segredos Restritos**:
  - `SUPABASE_SERVICE_ROLE_KEY`: Armazenada apenas nas Edge Functions do Supabase e no pipeline de administração; **proibida no frontend**.
  - `SENTRY_AUTH_TOKEN`: Utilizada exclusivamente em tempo de build para upload de sourcemaps.

---

## 4. Política de Backup e Continuidade de Negócio

1. **Snapshots de Banco de Dados**: O Supabase executa backups automáticos diários com retenção Point-in-Time Recovery (PITR) em planos corporativos.
2. **Deploy Reversível**: Deploy do frontend no Vercel é atômico. Em caso de regressão, o rollback é instantâneo no dashboard sem interrupção de serviço.
3. **Desacoplamento de Migrations**: O build do frontend nunca executa migrations no banco de dados. Migrations são versionadas no repositório `db` e aplicadas através de um processo operacional com revisão e restore point prévio.
