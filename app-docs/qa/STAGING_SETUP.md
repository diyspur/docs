# Guia de Configuração de Staging e Testes — DIYSPUR

**Data:** 11 de outubro de 2026  
**Repositórios:** `diyspur-cloud/app` e `diyspur-cloud/db`

---

## 1. Objetivo do Ambiente de Staging

O ambiente de staging serve para validação prévia de releases com comportamento espelhado de produção, mas com dados descartáveis ou anonimizados. Staging permite:
- Execução de testes Playwright autenticados multiusuário;
- Testes de carga controlados com k6 durante janelas aprovadas;
- Validação de fluxos de login com provedores OAuth e Magic Link;
- Replay e teste de novas migrações de banco antes de aplicar em produção.

---

## 2. Variáveis de Ambiente de Staging

### Frontend (`app/.env.staging`)
```env
NEXT_PUBLIC_SUPABASE_URL=https://<staging-ref>.supabase.co
NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY=sb_publishable_staging_key
NEXT_PUBLIC_SITE_URL=https://staging.diyspur.app

# Telemetria com ambiente isolado
NEXT_PUBLIC_SENTRY_DSN=https://<sentry-key>@o<org>.ingest.sentry.io/<proj>
SENTRY_ENVIRONMENT=staging
NEXT_PUBLIC_POSTHOG_KEY=phc_staging_placeholder
NEXT_PUBLIC_POSTHOG_HOST=https://us.i.posthog.com
```

> **Atenção:** A chave `service_role` NUNCA deve ser incluída no frontend, nem em arquivos de variáveis do Next.js.

---

## 3. Matriz de Usuários de Teste em Staging

Para testes E2E autenticados, criar no Supabase Auth de staging as seguintes contas controladas:
1. `qa-reader-a@example.com` — Leitor comum com histórico e clube ativo;
2. `qa-reader-b@example.com` — Leitor secundário para testes de privacidade e isolamento;
3. `qa-admin@example.com` — Usuário com flag `role: admin` na tabela `public.profiles` para testes editoriais.

As credenciais dessas contas devem ser armazenadas exclusivamente no cofre de segredos do GitHub Actions (`Secrets`) e nunca commitadas no repositório.

---

## 4. Limpeza e Isolamento de Fixtures

Ao criar clubes, leituras compartilhadas ou notas de diário durante baterias de testes em staging:
- Identificar as fixtures com prefixo estruturado (ex.: `[TEST-E2E-TIMESTAMP]`);
- Executar rotina de cleanup ao final da suíte (`teardown` do Playwright);
- Nunca misturar banco de staging com banco compartilhado ou réplica direta de produção.
