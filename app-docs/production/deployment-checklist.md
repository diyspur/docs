# Checklist de Deploy e Preparação para Produção — DIYSPUR

## 1. Verificações Pré-Deploy (Quality Gates)

| Verificação | Comando | Critério de Aceite | Status |
|---|---|---|---|
| Instalação de dependências | `npm ci` | Instalação limpa a partir do lockfile | CONCLUÍDO |
| Lint | `npm run lint` | 0 erros e 0 warnings impeditivos | CONCLUÍDO |
| Tipagem | `npm run typecheck` | `tsc --noEmit` executado sem erros | CONCLUÍDO |
| Testes unitários | `npm test` | Todos os 12 arquivos de teste (45 testes) passando | CONCLUÍDO |
| Build de produção | `npm run build` | Next.js Turbopack compila 32 rotas com sucesso | CONCLUÍDO |
| Auditoria de dependências | `npm run audit` | 0 vulnerabilidades conhecidas | CONCLUÍDO |

## 2. Variáveis e Segredos de Ambiente (Vercel)

- [ ] `NEXT_PUBLIC_SUPABASE_URL`: URL do projeto Supabase de produção.
- [ ] `NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY`: Chave anônima/publicável do Supabase (NUNCA `service_role`).
- [ ] `NEXT_PUBLIC_SITE_URL`: Domínio canônico de produção (`https://diyspur.vercel.app`).
- [ ] `NEXT_PUBLIC_SENTRY_DSN`: DSN do Sentry para o cliente web.
- [ ] `SENTRY_DSN`: DSN para servidor e edge.
- [ ] `SENTRY_ENVIRONMENT`: `production`.
- [ ] `SENTRY_AUTH_TOKEN`: Token de autenticação Sentry para upload de source maps (segredo exclusivo de build).
- [ ] `SENTRY_ORG`: Slug da organização no Sentry.
- [ ] `SENTRY_PROJECT`: Slug do projeto no Sentry.
- [ ] `NEXT_PUBLIC_POSTHOG_KEY`: Chave de ingestão de projeto do PostHog.
- [ ] `NEXT_PUBLIC_POSTHOG_HOST`: Endpoint da região da conta PostHog.

## 3. Verificações no Ambiente de Teste / Preview (Smoke Tests)
1. **Consentimento**:
   - Acessar a aplicação em modo anônimo e confirmar que o PostHog não dispara eventos antes do clique em "Aceitar analytics".
   - Clicar em "Recusar" e verificar persistência do opt-out.
   - Clicar em "Preferências de privacidade" e verificar reabertura do modal de controle.
2. **Sentry**:
   - Provocar erro de teste em página não crítica e verificar chegada ao painel com `boundary: 'app-route'`, commit SHA e sem PII (cookies, senhas ou emails).
3. **Sitemap e Robots**:
   - Abrir `/sitemap.xml` e verificar ausência de `/ranking` e capítulos em rascunho/futuros.
   - Abrir `/robots.txt` e verificar regras de bloqueio para `/admin/`, `/api/`, `/auth/`, `/perfil`, etc.
4. **Health Check**:
   - Fazer requisição GET em `/api/health` e validar resposta `{"status":"ok"}` com status 200.

## 4. Plano de Rollback
- **Frontend (Vercel)**:
  - Em caso de spike de erros 5xx ou regressão de autenticação, executar "Instant Rollback" no painel da Vercel para o deployment estável anterior.
- **Observabilidade**:
  - Se houver saturação de cota do Sentry, diminuir `NEXT_PUBLIC_SENTRY_TRACES_SAMPLE_RATE` e `SENTRY_TRACES_SAMPLE_RATE` ou remover a DSN temporariamente sem necessidade de novo deploy de código.
