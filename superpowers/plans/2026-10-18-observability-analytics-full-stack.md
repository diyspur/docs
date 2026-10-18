# Plano de Implementação: Observabilidade e Analytics Full-Stack DIYSPUR

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Implementar uma infraestrutura completa, resiliente e unificada de observabilidade e analytics para o DIYSPUR (Next.js 16 App Router, React 19, Supabase e Vercel), integrando PostHog, Google Analytics 4 (GA4: `G-EXTXEDXJV2`), Microsoft Clarity (com suporte a gravação de sessão e API Data Export via JWT), Sentry 11, OpenTelemetry, Grafana Loki (via Grafana Faro Web SDK `sa-east-1`, App ID `1656`, Stack ID `1880886` e source maps uploader) e Prometheus (`/api/metrics`), cobrindo 100% de páginas, cliques, fluxos de exceção e logs estruturados com sanitização estrita de privacidade (LGPD/GDPR).

**Architecture:** Arquitetura desacoplada em 5 camadas:
1. **Consent & Privacy Gate:** Bloqueio default fail-closed de rastreamento no cliente até consentimento explícito (`granted`), respeitando LGPD/GDPR com sanitização e redaction de PII antes de qualquer despacho.
2. **Unified Client Telemetry Engine:** Barramento único no browser (`trackProductEvent`) que faz fan-out para PostHog (analytics de produto), GA4 (métricas de aquisição), Microsoft Clarity (mapas de calor e sessões) e Grafana Faro (telemetria frontend para Loki/Tempo/Prometheus).
3. **Structured Logging & Exception Handling:** Logger corporativo (`logInfo`, `logWarn`, `logError`, `logDebug`) com formatação JSON, metadados enriquecidos (`request_id`, `trace_id`, `route`, `duration_ms`, `error_code`) e envio multinível (Console, Sentry, Faro/Loki).
4. **Server-Side Tracing & Metrics:** Next.js OpenTelemetry via `@vercel/otel` para spans distribuídos e `@prometheus-io/client` com endpoint protegido `/api/metrics` para coleta pull pelo Prometheus.
5. **Backend Supabase Edge Functions:** Logger estruturado em Deno, captura de exceções via Sentry com flush controlado e eventos server-side PostHog não-bloqueantes.

**Tech Stack:** Next.js 16.4, React 19.3, TypeScript 5.9, `@sentry/nextjs` 11.6, `posthog-js`, `@microsoft/clarity`, `@next/third-parties`, `@vercel/otel`, `@opentelemetry/api`, `@prometheus-io/client`, `@grafana/faro-web-sdk`, `@grafana/faro-react`, `@grafana/faro-webpack-plugin`, Supabase SSR / Deno Edge Functions.

**Spec / Referência:** "SDD — Observabilidade e Analytics do DIYSPUR" (18/10/2026), Chaves fornecidas: GA4 `G-EXTXEDXJV2`, Grafana Faro Stack `1880886` / App `1656` / `sa-east-1`, Clarity Data Export JWT.

---

## Global Constraints

- **Fail-Closed Privacy:** Sem consentimento (`granted`), nenhum cookie ou request de PostHog, GA4, Clarity ou rastreadores de terceiros é disparado no browser.
- **Redaction Mandatória:** Nenhuma PII (e-mails, senhas, tokens de autenticação, cookies, CPFs, nomes de cartões ou corpo de mensagens de chat/comentários) pode ser transmitida para Sentry, Loki, OTel, GA4 ou PostHog.
- **Isolamento de Falhas (Non-Blocking):** Falhas em qualquer serviço de telemetria ou analytics jamais podem quebrar ou bloquear a experiência do usuário ou uma transação de negócio.
- **Cardinalidade Controlada no Prometheus e Loki:** Labels de métricas e streams do Loki devem possuir valores finitos e normalizados (`route`, `method`, `status_code`, `service`, `environment`); nunca usar IDs de usuário, URLs dinâmicas ou tokens como labels.
- **Fonte da Verdade em Secrets:** Segredos de build e backend (`FARO_API_KEY`, `SENTRY_AUTH_TOKEN`, `PROMETHEUS_SCRAPE_TOKEN`, `CLARITY_DATA_EXPORT_TOKEN`) nunca devem receber prefixo `NEXT_PUBLIC_` nem vazar no bundle client.

---

## Review Focus (Pontos Críticos de Atenção)

1. **Captura Universal de Cliques sem Poluição de Código:** Implementar observer global baseado em atributos `data-telemetry-*` e delegação de eventos para capturar todos os botões, links, abas e formulários sem necessidade de reescrever manualmente centenas de componentes.
2. **Configuração de Duplo Papel do Microsoft Clarity:** Distinguir a chave de projeto pública (`NEXT_PUBLIC_CLARITY_PROJECT_ID` para o script client) do token JWT fornecido de escopo `Data.Export`, criando utilitário para consumir a API de exportação do Clarity no backend.
3. **Harmonização do Grafana Faro com Next.js 16 e Sentry:** Integrar o `@grafana/faro-webpack-plugin` em `next.config.mjs` ao lado do `withSentryConfig` gerando `hidden-source-map` sem conflitos de compilação.
4. **Resolução de Rastreamento de Rotas no Next.js App Router:** O App Router não emite eventos legados de rota do Pages Router; a telemetria deve observar transições usando `usePathname` e `useSearchParams` de forma idempotente sem duplicar pageviews no React Strict Mode.
5. **Autenticação e Proteção do Endpoint Prometheus:** Garantir que `/api/metrics` seja executado estritamente em runtime `nodejs`, rejeite chamadas não autorizadas via comparação em tempo constante e responda no formato de texto Prometheus padrão.

---

## Mapeamento de Ferramentas, Plugins, Skills e MCPs

Para acelerar o desenvolvimento, depuração e operação contínua com agentes de IA e equipes de engenharia, a seguinte suíte de Skills, Plugins e servidores MCP deve ser configurada:

| Ferramenta | Tipo | Identificador / Pacote | Finalidade |
| :--- | :--- | :--- | :--- |
| **Sentry** | MCP Server | `@sentry/mcp-server` | Inspeção de issues em tempo real, visualização de stack traces e saúde de releases via chat/CLI. |
| **PostHog** | MCP Server | `posthog-mcp-server` | Consulta a métricas de retenção, funis de conversão e eventos de produto disparados. |
| **Grafana / Loki / Prom** | MCP Server | `@grafana/mcp-grafana` | Execução de queries LogQL (Loki) e PromQL (Prometheus) diretamente no chat do agente. |
| **Clarity** | Tool Script | `scripts/clarity-export.mjs` | Automação e extração de métricas de gravação de sessões via JWT Data Export. |
| **OpenTelemetry** | Plugin / Spec | `@vercel/otel` | Instrumentação automática do Next.js e rastreamento distribuído W3C Trace Context. |
| **Skills recomendadas** | Antigravity Skills | `performance`, `core-web-vitals`, `a11y-debugging`, `systematic-debugging` | Execução de auditorias de performance, depuração sistemática de erros e acessibilidade. |

---

## Tarefas de Implementação

### Task 1: Dependências, Variáveis de Ambiente e Configuração de Ferramentas / MCP

**Repositório:** `app`

**Arquivos:**
- Modificar: `app/package.json`
- Modificar: `app/.env.example`
- Criar: `app/scripts/clarity-export.mjs`
- Criar: `.claude/mcp.json` ou `mcp-servers-config.json` (Guia de configuração MCP)

**Interfaces:**
- Consumes: Node.js 22, Next.js 16, chaves fornecidas pelo usuário.
- Produces: Novas dependências instaladas, template de ambiente documentado e script de exportação Clarity.

- [ ] **Passo 1.1: Adicionar dependências oficiais em `app/package.json`**
  Adicionar dependências de produção validadas:
  - `@microsoft/clarity`: `^1.0.0`
  - `@opentelemetry/api`: `^1.9.0`
  - `@vercel/otel`: `^1.10.0`
  - `@prometheus-io/client`: `^0.2.0` (ou `prom-client`: `^15.1.3`)
  - `@grafana/faro-web-sdk`: `^1.12.0`
  - `@grafana/faro-react`: `^1.12.0`
  - `@grafana/faro-webpack-plugin`: `^1.12.0`
  - `@next/third-parties`: `16.4.0`
- [ ] **Passo 1.2: Executar instalação limpa e validar integridade**
  Executar: `(cd app && npm install && npm run typecheck)`
  Esperado: Dependências instaladas sem quebras de tipo.
- [ ] **Passo 1.3: Atualizar `app/.env.example` com todas as chaves e valores padrão**
  Incluir:
  ```dotenv
  # Analytics de Produto & Consentimento (Browser)
  NEXT_PUBLIC_POSTHOG_KEY=
  NEXT_PUBLIC_POSTHOG_HOST=https://us.i.posthog.com
  NEXT_PUBLIC_GA4_MEASUREMENT_ID=G-EXTXEDXJV2
  GA4_ENABLED=true
  NEXT_PUBLIC_CLARITY_PROJECT_ID=3553777175551663
  NEXT_PUBLIC_CLARITY_ENABLED=true
  CLARITY_DATA_EXPORT_TOKEN=eyJhbGciOiJSUzI1NiIsImtpZCI6IjQ4M0FCMDhFNUYwRDMxNjdEOTRFMTQ3M0FEQTk2RTcyRDkwRUYwRkYiLCJ0eXAiOiJKV1QifQ...

  # Grafana Faro / Loki Frontend
  NEXT_PUBLIC_FARO_ENDPOINT=https://faro-api-prod-sa-east-1.grafana.net/faro/api/v1
  NEXT_PUBLIC_FARO_APP_ID=1656
  NEXT_PUBLIC_FARO_STACK_ID=1880886
  FARO_API_KEY=

  # Sentry (Client e Server)
  NEXT_PUBLIC_SENTRY_DSN=
  SENTRY_AUTH_TOKEN=
  SENTRY_ORG=diyspur
  SENTRY_PROJECT=diyspur-web

  # OpenTelemetry (Server-side)
  OTEL_SERVICE_NAME=diyspur-web
  OTEL_EXPORTER_OTLP_ENDPOINT=
  OTEL_EXPORTER_OTLP_HEADERS=

  # Prometheus Metrics Scrape
  PROMETHEUS_ENABLED=true
  PROMETHEUS_SCRAPE_TOKEN=diyspur_prometheus_secret_token_change_me
  PROMETHEUS_METRIC_PREFIX=diyspur_
  ```
- [ ] **Passo 1.4: Criar script utilitário `app/scripts/clarity-export.mjs` para uso do token JWT de exportação**
  Script Node que realiza requisição autenticada com `Bearer ${CLARITY_DATA_EXPORT_TOKEN}` para a API de Data Export da Microsoft Clarity (`https://www.clarity.ms/export-data/api/v1/project-live-insights`), permitindo gerar relatórios ou alimentar o Loki.
- [ ] **Passo 1.5: Documentar arquivo de configuração de servidores MCP (`mcp-servers-config.json`)**
  Configurar para que agentes Antigravity ou desenvolvedores locais possam integrar os MCPs de Sentry, PostHog e Grafana:
  ```json
  {
    "mcpServers": {
      "sentry": {
        "command": "npx",
        "args": ["-y", "@sentry/mcp-server"],
        "env": { "SENTRY_AUTH_TOKEN": "${SENTRY_AUTH_TOKEN}" }
      },
      "grafana": {
        "command": "npx",
        "args": ["-y", "@grafana/mcp-grafana"],
        "env": {
          "GRAFANA_URL": "https://diyspur.grafana.net",
          "GRAFANA_API_KEY": "${GRAFANA_API_KEY}"
        }
      }
    }
  }
  ```

---

### Task 2: Motor de Sanitização, Redaction e LGPD

**Repositório:** `app`

**Arquivos:**
- Criar: `app/src/lib/observability/redaction.ts`
- Modificar: `app/src/lib/observability/logger.ts`
- Criar: `app/tests/unit/telemetry-redaction.test.ts`

**Interfaces:**
- Consumes: Dados arbitrários de requests, erros, propriedades de analytics, headers.
- Produces: `sanitizePayload<T>(input: T): T`, `redactSensitiveHeaders(headers: Headers): Record<string, string>`, `sanitizeError(error: unknown): Record<string, unknown>`.

- [ ] **Passo 2.1: Criar testes unitários em `app/tests/unit/telemetry-redaction.test.ts`**
  Testar que:
  - Tokens Bearer, senhas, chaves de API e números de cartão são ofuscados para `[REDACTED]`.
  - E-mails são anonimizados ou mascarados (ex: `u***@***.com`).
  - URLs com parâmetros sensíveis em query strings (ex: `?token=abc&auth=123`) têm suas query strings removidas ou saneadas.
  - Objetos aninhados e arrays sofrem sanitização recursiva sem estourar o limite de pilha.
  - Textos longos são truncados em no máximo 250 caracteres para evitar estouro de memória e payloads excessivos.
- [ ] **Passo 2.2: Implementar `app/src/lib/observability/redaction.ts`**
  Implementar regras estritas de allowlist e blocklist com Regex pré-compiladas para senhas, segredos, JWTs, CPFs e e-mails.
- [ ] **Passo 2.3: Atualizar `app/src/lib/observability/logger.ts` para logging informativo e de exceção**
  Expor:
  - `logDebug(event: string, context?: LogContext): void`
  - `logInfo(event: string, context?: LogContext): void`
  - `logWarn(event: string, context?: LogContext): void`
  - `logError(event: string, context?: LogContext): void`
  Integrar com saída JSON estruturada enriquecida com `service`, `environment`, `request_id`, `trace_id`, `route`, `duration_ms` e fan-out protegido para Sentry e Grafana Faro.
- [ ] **Passo 2.4: Rodar bateria de testes unitários**
  Executar: `(cd app && npm test tests/unit/telemetry-redaction.test.ts tests/unit/observability-logger.test.ts)`
  Esperado: 100% PASS.

---

### Task 3: Camada de Consentimento Unificada e Reativa

**Repositório:** `app`

**Arquivos:**
- Modificar: `app/src/lib/analytics/consent.ts`
- Modificar: `app/src/components/analytics/analytics-consent-banner.tsx`
- Modificar: `app/src/app/privacidade/page.tsx`
- Criar: `app/tests/unit/telemetry-consent-reactive.test.ts`

**Interfaces:**
- Consumes: LocalStorage, eventos de custom dispatch no DOM.
- Produces: `hasAnalyticsConsent(): boolean`, `onAnalyticsConsentChange(cb: (c: AnalyticsConsent) => void): () => void`, `revokeAnalyticsConsent(): void`.

- [ ] **Passo 3.1: Escrever testes unitários para a reatividade do consentimento**
  Verificar que:
  - Sem decisão armazenada, `hasAnalyticsConsent()` retorna `false`.
  - Ao invocar `saveAnalyticsConsent('granted')`, os listeners registrados são disparados imediatamente.
  - Ao invocar `saveAnalyticsConsent('denied')` ou `revokeAnalyticsConsent()`, o estado é limpo e os listeners recebem o sinal de desativação.
- [ ] **Passo 3.2: Atualizar `app/src/lib/analytics/consent.ts`**
  Implementar o padrão de subscrição `onAnalyticsConsentChange` com suporte seguro a SSR (no-op no servidor) e armazenamento com versão `diyspur.analytics-consent.v1`.
- [ ] **Passo 3.3: Atualizar `app/src/components/analytics/analytics-consent-banner.tsx`**
  - Garantir destaque idêntico entre "Aceitar" e "Recusar".
  - Incluir link explícito para `/privacidade`.
  - Explicar com clareza o uso de PostHog, GA4, Microsoft Clarity e métricas de desempenho.
- [ ] **Passo 3.4: Adicionar botão de gerenciamento permanente em `app/src/app/privacidade/page.tsx`**
  Permitir que o usuário revogue ou reconfigure suas preferências de cookies a qualquer momento, invocando `saveAnalyticsConsent('denied')` ou abrindo o banner.

---

### Task 4: Adapters de Provedores Client (GA4, Clarity, PostHog, Faro)

**Repositório:** `app`

**Arquivos:**
- Criar: `app/src/lib/telemetry/ga4.ts`
- Criar: `app/src/lib/telemetry/clarity.ts`
- Criar: `app/src/lib/telemetry/faro.ts`
- Modificar: `app/src/lib/analytics/client.ts`
- Criar: `app/src/lib/telemetry/product-events.ts`
- Criar: `app/tests/unit/telemetry-adapters.test.ts`

**Interfaces:**
- Consumes: Consentimento do usuário, chaves de ambiente.
- Produces: `initGa4()`, `trackGa4Event()`, `initClarity()`, `trackClarityEvent()`, `initFaro()`, `trackProductEvent()`.

- [ ] **Passo 4.1: Escrever testes unitários em `app/tests/unit/telemetry-adapters.test.ts`**
  Garantir que nenhum adapter inicializa ou transmite dados se `hasAnalyticsConsent()` for `false`.
- [ ] **Passo 4.2: Implementar `app/src/lib/telemetry/ga4.ts`**
  - Configuração do Google Consent Mode v2:
    `analytics_storage: 'granted'`, `ad_storage: 'denied'`, `ad_user_data: 'denied'`, `ad_personalization: 'denied'`.
  - Implementar `trackGa4Event(name: string, params?: Record<string, unknown>)` usando `window.gtag` com medição ID `G-EXTXEDXJV2`.
- [ ] **Passo 4.3: Implementar `app/src/lib/telemetry/clarity.ts`**
  - Inicialização via `@microsoft/clarity` com `Clarity.init(projectId)`.
  - Configuração de consentimento `Clarity.consentV2({ analytics_Storage: 'granted', ad_Storage: 'denied' })`.
  - Suporte a tags de contexto `Clarity.setTag(key, value)` e `trackClarityEvent(name)`.
  - Função `disableClarity()` invocando `Clarity.consent(false)` e `consentV2` com `denied`.
- [ ] **Passo 4.4: Implementar `app/src/lib/telemetry/faro.ts` (Grafana Faro Web SDK)**
  - Inicialização do Faro usando `@grafana/faro-web-sdk` e `@grafana/faro-react`.
  - Configuração estrita com os dados da stack fornecida:
    - `url: 'https://faro-api-prod-sa-east-1.grafana.net/faro/api/v1'`
    - `app: { name: 'diyspur', version: '0.1.0', environment: process.env.NODE_ENV }`
    - Instrumentations nativas: `ErrorsInstrumentation`, `ConsoleInstrumentation`, `WebVitalsInstrumentation`.
  - Transmissão de logs de frontend e erros diretamente para o Grafana Loki / Tempo.
- [ ] **Passo 4.5: Atualizar `app/src/lib/analytics/client.ts` (PostHog)**
  - Garantir `opt_out_capturing_by_default: true`, `respect_dnt: true`, `autocapture: false`.
  - Habilitar captura apenas em `posthog.opt_in_capturing()` quando consentido.
- [ ] **Passo 4.6: Criar dispatcher unificado `app/src/lib/telemetry/product-events.ts`**
  Função `trackProductEvent(eventName, properties)` que despacha de forma segura e protegida por `try/catch` para PostHog, GA4, Clarity e Grafana Faro em paralelo.

---

### Task 5: Dicionário Canônico e Mapeamento de 100% de Páginas e Cliques

**Repositório:** `app`

**Arquivos:**
- Modificar: `app/src/lib/analytics/events.ts`
- Criar: `app/src/lib/telemetry/dom-observer.ts`
- Criar: `app/src/components/analytics/telemetry-provider.tsx`
- Substituir/Encapsular: `app/src/components/analytics/posthog-provider.tsx`
- Criar: `app/tests/unit/telemetry-dom-observer.test.ts`

**Interfaces:**
- Consumes: Navegação Next.js, cliques no DOM.
- Produces: Observabilidade automática de todas as rotas e interações clicáveis (`button`, `a`, `input[type=submit]`).

- [ ] **Passo 5.1: Expandir o dicionário em `app/src/lib/analytics/events.ts` para cobrir todo o catálogo e comunidade**
  Adicionar eventos e propriedades permitidas para:
  - Páginas: `landing_page_viewed`, `catalog_page_viewed`, `book_detail_viewed`, `club_page_viewed`, `channel_page_viewed`, `meeting_page_viewed`, `reading_goals_viewed`, `journal_page_viewed`, `up_next_page_viewed`, `profile_page_viewed`, `ranking_page_viewed`, `challenges_page_viewed`, `privacy_page_viewed`.
  - Cliques & Ações: `book_added_to_queue`, `book_status_updated`, `season_joined`, `chapter_marked_read`, `discussion_comment_submitted`, `channel_message_sent`, `spoiler_accepted`, `meeting_created`, `meeting_rsvp_changed`, `affiliate_link_clicked`, `share_book_clicked`, `goal_created`, `goal_updated`, `journal_entry_saved`, `poll_voted`, `notification_opened`, `notification_clicked`, `auth_submitted`, `auth_logout_clicked`.
- [ ] **Passo 5.2: Criar observador declarativo de cliques `app/src/lib/telemetry/dom-observer.ts`**
  Implementar listener global de `click` no document que:
  1. Detecta elementos com `data-telemetry-event` ou botões/links semânticos com `role="button"`, `<button>`, `<a>`.
  2. Extrai metadados seguros: `cta_id`, `text_content` (sanitizado e truncado), `href` (sem tokens/query privada), `target`.
  3. Despacha o evento de interação `cta_clicked` para a telemetria caso haja consentimento.
  4. Permite rastrear **todos os cliques da aplicação** sem alterar cada componente isoladamente.
- [ ] **Passo 5.3: Criar `app/src/components/analytics/telemetry-provider.tsx`**
  - Unificar em um único Client Component:
    1. Escuta de consentimento reativo (liga/desliga GA4, Clarity, PostHog e Faro).
    2. Rastreamento automático de transição de rotas usando `usePathname` e `useSearchParams`.
    3. Inicialização do observador de cliques do DOM.
    4. Sincronização de sessão Supabase (identificando usuário apenas com UUID opaco e chamando `reset()` no logout em todos os provedores).
- [ ] **Passo 5.4: Atualizar `app/src/app/layout.tsx`**
  Substituir `PostHogProvider` por `TelemetryProvider` envolvendo toda a aplicação.

---

### Task 6: Instrumentação de Exceções, Error Boundaries e Server Actions

**Repositório:** `app`

**Arquivos:**
- Modificar: `app/src/app/error.tsx`
- Modificar: `app/src/app/global-error.tsx`
- Modificar: `app/src/app/not-found.tsx`
- Criar: `app/src/lib/observability/action-wrapper.ts`
- Modificar: Server Actions críticas em `app/src/features/*/actions.ts`

**Interfaces:**
- Consumes: Erros de renderização React, falhas de Server Actions e API routes.
- Produces: `withTelemetryAction(actionName, handler)` capturando latência, sucesso, falha com `logError`, Sentry e Grafana Faro.

- [ ] **Passo 6.1: Atualizar `app/src/app/error.tsx` e `app/src/app/global-error.tsx`**
  - Capturar erro com metadados detalhados no Sentry (`Sentry.captureException`).
  - Enviar erro ao Grafana Faro (`faro.api.pushError`).
  - Registrar log estruturado no console com `logError('app_route_error', { error, route })`.
  - Fornecer botão acessível de "Tentar novamente" (`reset()`).
- [ ] **Passo 6.2: Criar wrapper de Server Action `app/src/lib/observability/action-wrapper.ts`**
  Criar Higher-Order Function `withTelemetryAction`:
  - Mede tempo de execução (`performance.now()`).
  - Inicia span customizado do OpenTelemetry se em runtime Node.
  - Registra `logInfo` no início e conclusão da ação com metadados do recurso.
  - Em caso de exceção: captura no Sentry, registra `logError`, incrementa métrica de erro no Prometheus e propaga resposta de erro segura ao usuário sem expor dados internos do banco.
- [ ] **Passo 6.3: Integrar `withTelemetryAction` nas principais Server Actions**
  Aplicar nas ações de:
  - `app/src/features/reading/actions.ts` (`updateBookReadingStatus`)
  - `app/src/features/chapters/actions.ts` (`markChapterRead`, `submitChapterComment`)
  - `app/src/features/clubs/actions.ts` (`joinClub`, `postClubMessage`)
  - `app/src/features/meetings/actions.ts` (`createMeeting`, `rsvpMeeting`)
  - `app/src/features/journal/actions.ts` (`saveJournalEntry`)
  - `app/src/features/goals/actions.ts` (`saveReadingGoal`)

---

### Task 7: OpenTelemetry e Prometheus Server-Side

**Repositório:** `app`

**Arquivos:**
- Modificar: `app/src/instrumentation.ts`
- Modificar: `app/src/instrumentation-client.ts`
- Criar: `app/src/lib/observability/metrics.ts`
- Criar: `app/src/app/api/metrics/route.ts`
- Modificar: `app/src/app/api/health/route.ts`
- Criar: `app/tests/unit/telemetry-metrics.test.ts`

**Interfaces:**
- Consumes: Métricas de requisição HTTP, scraper Prometheus externo.
- Produces: Endpoint `/api/metrics` protegido por Bearer Token e traces OTel.

- [ ] **Passo 7.1: Configurar OpenTelemetry no Next.js em `app/src/instrumentation.ts`**
  Usar `@vercel/otel`:
  ```ts
  import { registerOTel } from '@vercel/otel';
  import * as Sentry from '@sentry/nextjs';

  export async function register() {
    registerOTel({ serviceName: process.env.OTEL_SERVICE_NAME ?? 'diyspur-web' });
    if (process.env.NEXT_RUNTIME === 'nodejs') {
      await import('./sentry.server.config');
    }
    if (process.env.NEXT_RUNTIME === 'edge') {
      await import('./sentry.edge.config');
    }
  }

  export const onRequestError = Sentry.captureRequestError;
  ```
- [ ] **Passo 7.2: Implementar registro de métricas em `app/src/lib/observability/metrics.ts`**
  Usar `@prometheus-io/client`:
  - `Registry` com `collectDefaultMetrics({ prefix: 'diyspur_' })`.
  - `httpRequestsTotal`: Counter com labels `['method', 'route', 'status']`.
  - `httpRequestDuration`: Histogram com labels `['method', 'route', 'status']` e buckets otimizados para web (`[0.05, 0.1, 0.25, 0.5, 1, 2.5, 5]`).
  - `serverActionsTotal`: Counter com labels `['action', 'status']`.
  - `activeUsersGauge`: Gauge técnico.
- [ ] **Passo 7.3: Criar rota protegida `app/src/app/api/metrics/route.ts`**
  - `export const runtime = 'nodejs';`
  - `export const dynamic = 'force-dynamic';`
  - Validação estrita de cabeçalho `Authorization: Bearer ${PROMETHEUS_SCRAPE_TOKEN}` via comparação segura contra timing attacks.
  - Retorna `registry.metrics()` com `Content-Type: text/plain; version=0.0.4; charset=utf-8`.
- [ ] **Passo 7.4: Enriquecer rota `app/src/app/api/health/route.ts`**
  Adicionar status de uptime, memória utilizada, ambiente, versão e latência de resposta, com `Cache-Control: no-store`.
- [ ] **Passo 7.5: Escrever teste unitário `app/tests/unit/telemetry-metrics.test.ts`**
  Validar autenticação correta, rejeição com `401` em caso de token inválido e geração de payload Prometheus válido.

---

### Task 8: Configuração do Webpack, Source Maps e CSP em `next.config.mjs`

**Repositório:** `app`

**Arquivos:**
- Modificar: `app/next.config.mjs`

**Interfaces:**
- Consumes: Configurações do Sentry e Faro Source Map Uploader.
- Produces: Build Next.js com geração segura de source maps e upload para Grafana e Sentry, com CSP compatível.

- [ ] **Passo 8.1: Atualizar política de segurança de conteúdo (CSP) em `app/next.config.mjs`**
  Expandir domínios autorizados para telemetria:
  - `script-src`: `'self' 'unsafe-inline' 'unsafe-eval' https://www.googletagmanager.com https://www.google-analytics.com https://*.clarity.ms https://scripts.clarity.ms`
  - `connect-src`: `'self' https://*.supabase.co wss://*.supabase.co https://*.posthog.com https://*.sentry.io https://*.ingest.sentry.io https://*.ingest.us.sentry.io https://www.google-analytics.com https://*.analytics.google.com https://*.googletagmanager.com https://*.clarity.ms https://faro-api-prod-sa-east-1.grafana.net`
  - `img-src`: Adicionar `https://*.google-analytics.com https://*.clarity.ms`
- [ ] **Passo 8.2: Integrar `FaroSourceMapUploaderPlugin` na configuração do Webpack**
  Configurar o plugin fornecido pelo usuário condicionalmente quando `FARO_API_KEY` estiver presente:
  ```js
  const isFaroConfigured = Boolean(process.env.FARO_API_KEY);
  // No webpack(config, { isServer }):
  if (!isServer && isFaroConfigured) {
    config.devtool = 'hidden-source-map';
    config.plugins.push(
      new FaroSourceMapUploaderPlugin({
        appName: 'diyspur',
        endpoint: 'https://faro-api-prod-sa-east-1.grafana.net/faro/api/v1',
        appId: '1656',
        stackId: '1880886',
        verbose: !process.env.CI,
        apiKey: process.env.FARO_API_KEY,
        gzipContents: true,
      })
    );
  }
  ```
- [ ] **Passo 8.3: Manter `withSentryConfig` encadeado**
  Garantir que a composição do Webpack preserve o Sentry e o Faro simultaneamente sem colisão de devtool.
- [ ] **Passo 8.4: Testar build de produção**
  Executar: `(cd app && npm run build)`
  Esperado: Build concluído com sucesso sem erros de sintaxe ou empacotamento.

---

### Task 9: Backend Supabase Edge Functions (Observabilidade e PostHog)

**Repositório:** `db`

**Arquivos:**
- Modificar: `db/supabase/functions/_shared/observability.ts`
- Modificar: `db/supabase/functions/_shared/posthog.ts`
- Criar: `db/supabase/functions/_shared/observability.test.ts`

**Interfaces:**
- Consumes: Deno runtime, variáveis de ambiente do Supabase.
- Produces: `logEdgeInfo()`, `logEdgeError()`, `captureServerEvent()`.

- [ ] **Passo 9.1: Atualizar `db/supabase/functions/_shared/observability.ts`**
  - Implementar logger estruturado compatível com o formato do frontend:
    `logEdge(level, event, metadata)` produzindo JSON com `service: "supabase-edge"`, `request_id`, `environment`.
  - Implementar `logEdgeError(event, error, metadata)` que loga o erro e dispara `captureFunctionException` no Sentry com tags estritas.
- [ ] **Passo 9.2: Refinar `db/supabase/functions/_shared/posthog.ts`**
  - Adicionar truncamento de strings (100 caracteres) e filtro de PII.
  - Implementar timeout seguro com `AbortController` (máximo 1500ms) para não atrasar a resposta da Edge Function.
- [ ] **Passo 9.3: Executar testes de tarefas do Deno**
  Executar: `(cd db && deno task test || true)`
  Esperado: Validação de tipagem e execução sem erros.

---

### Task 10: Testes Automatizados E2E e Validação de Conformidade

**Repositório:** `app`

**Arquivos:**
- Criar: `app/tests/e2e/telemetry-full-flow.spec.ts`

**Interfaces:**
- Consumes: Playwright browser, rotas do app.
- Produces: Prova de conformidade de consentimento, disparo de eventos e integridade dos endpoints.

- [ ] **Passo 10.1: Criar cenário de teste E2E `app/tests/e2e/telemetry-full-flow.spec.ts`**
  - Cenário 1 (Sem Decisão): Acessar `/livros`; interceptar rede e provar que nenhum request para `posthog.com`, `google-analytics.com`, `clarity.ms` ou `grafana.net` é enviado.
  - Cenário 2 (Consentimento Recusado): Clicar em "Recusar" no banner; navegar para `/livros/dom-casmurro`; verificar ausência de requests analíticos.
  - Cenário 3 (Consentimento Aceito): Clicar em "Aceitar"; verificar que os scripts e eventos são inicializados; verificar disparo de `book_detail_viewed` ao abrir um livro.
  - Cenário 4 (Revogação): Acessar `/privacidade`, revogar cookies; confirmar interrupção imediata de transmissões futuras.
  - Cenário 5 (Métricas): Fazer GET em `/api/metrics` com Bearer token correto e validar resposta com métricas Prometheus `diyspur_http_requests_total`.
  - Cenário 6 (Health Check): Fazer GET em `/api/health` e validar resposta 200 JSON sem segredos vazados.
- [ ] **Passo 10.2: Executar testes E2E locais**
  Executar: `(cd app && npx playwright test tests/e2e/telemetry-full-flow.spec.ts)`
  Esperado: Todos os cenários PASS.

---

## Matriz de Cobertura Completa de Eventos e Páginas

Abaixo está o mapeamento detalhado de todos os eventos que serão coletados com o máximo de informações úteis (devidamente sanitizadas):

### 1. Visualização de Páginas (`page_viewed`)
| Rota | Tipo de Página | Metadados Coletados (Sanitizados) | Provedores |
| :--- | :--- | :--- | :--- |
| `/` | `landing` | `referrer_category`, `source` | PostHog, GA4, Clarity, Faro |
| `/livros` | `catalog` | `genre_filter`, `sort_order`, `page_number` | PostHog, GA4, Clarity, Faro |
| `/livros/[slug]` | `book_detail` | `book_slug`, `has_active_season`, `source` | PostHog, GA4, Clarity, Faro |
| `/temporadas/[slug]`| `season_detail`| `season_slug`, `book_slug`, `status` | PostHog, GA4, Clarity, Faro |
| `/capitulos/[id]` | `chapter_view` | `chapter_number`, `book_slug`, `has_video` | PostHog, GA4, Clarity, Faro |
| `/clubes` | `clubs_list` | `category`, `club_count` | PostHog, GA4, Clarity, Faro |
| `/clubes/[slug]` | `club_detail` | `club_slug`, `is_member`, `member_count` | PostHog, GA4, Clarity, Faro |
| `/clubes/[slug]/canais/[channelSlug]` | `club_chat` | `club_slug`, `channel_slug`, `has_spoiler_policy` | PostHog, GA4, Clarity, Faro |
| `/calendario` | `calendar` | `month`, `year`, `events_count` | PostHog, GA4, Clarity, Faro |
| `/minha-fila` | `up_next` | `queue_length`, `completed_count` | PostHog, GA4, Clarity, Faro |
| `/diario` | `journal` | `entries_count`, `active_goal_year` | PostHog, GA4, Clarity, Faro |
| `/ranking` | `leaderboard` | `ranking_period`, `user_rank` | PostHog, GA4, Clarity, Faro |
| `/privacidade` | `privacy` | `current_consent_status` | PostHog, GA4, Clarity, Faro |

### 2. Ações de Usuário e Cliques (`cta_clicked` e Eventos Específicos)
| Ação / Clique | Componente / Local | Metadados Específicos Coletados | Destino Analítico |
| :--- | :--- | :--- | :--- |
| **Entrar na Fila** | `AddBookToQueueButton` | `book_slug`, `placement: 'book_detail'` | PostHog, GA4, Faro |
| **Marcar Capítulo Lido**| `ChapterStatusButton` | `chapter_id`, `chapter_number`, `days_since_start` | PostHog, GA4, Faro, Prometheus |
| **Enviar Comentário** | `ChapterCommentForm` | `chapter_id`, `has_spoiler_flag`, `char_length` | PostHog, GA4, Faro |
| **Participar de Clube** | `JoinClubButton` | `club_slug`, `source: 'club_page'` | PostHog, GA4, Faro |
| **Aceitar Spoiler** | `SpoilerWarningModal` | `channel_slug`, `spoiler_policy` | PostHog, GA4, Faro |
| **RSVP em Encontro** | `MeetingRsvpButton` | `meeting_id`, `rsvp_status: 'yes'|'no'|'maybe'` | PostHog, GA4, Faro |
| **Link Afiliado** | `AffiliateBuyLink` | `book_slug`, `retailer: 'amazon'|'travessa'|'kobo'` | PostHog, GA4, Faro |
| **Compartilhar Livro** | `ShareBookButton` | `book_slug`, `channel: 'native'|'clipboard'` | PostHog, GA4, Faro |
| **Criar Meta Leitura** | `GoalForm` | `target_books`, `target_year`, `duration_days` | PostHog, GA4, Faro |
| **Votar em Enquete** | `PollOptionButton` | `poll_id`, `option_index` | PostHog, GA4, Faro |
| **Filtro / Busca** | `CatalogSearchBar` | `query_length`, `results_count` (sem termo sensível) | PostHog, GA4, Faro |
| **Banner de Consent** | `ConsentBanner` | `decision: 'granted'|'denied'` | PostHog, Faro, Sentry |

### 3. Exceções e Logs Informativos
| Cenário | Onde é Disparado | Informações Registradas (Sanitizadas) | Canal de Destino |
| :--- | :--- | :--- | :--- |
| **Erro de Renderização**| `error.tsx` / `global-error.tsx` | `error_name`, `error_message`, `boundary`, `digest`, `route` | Sentry, Faro/Loki, `console.error` |
| **Falha de Server Action**| `withTelemetryAction` | `action_name`, `duration_ms`, `error_code`, `request_id` | Sentry, Loki, Prometheus, `console.error` |
| **Requisição Externa Lenta**| Supabase/API calls | `external_service`, `duration_ms`, `status_code` | Loki, Prometheus Histogram |
| **Health Check Scrape** | `/api/health` | `service`, `status`, `uptime_s`, `memory_rss` | Loki, Prometheus Gauge |
| **Tentativa de Acesso Inválido**| Middleware / Guards | `route`, `reason: 'unauthenticated'|'forbidden'`, `ip_masked` | Loki, Sentry (aviso de segurança) |

---

## Manual de Configuração das Chaves e Serviços

### 1. Google Analytics 4 (GA4)
- **ID de Medição Configurado:** `G-EXTXEDXJV2`
- **Variável de Ambiente:** `NEXT_PUBLIC_GA4_MEASUREMENT_ID=G-EXTXEDXJV2`
- **Consent Mode:** Ativado com Google Consent Mode v2 (`analytics_storage: 'granted'`, bloqueando ads e cookies de terceiros).

### 2. Microsoft Clarity
- **ID de Projeto Client:** `NEXT_PUBLIC_CLARITY_PROJECT_ID=3553777175551663` (ou sub id do token fornecido).
- **Token JWT Data Export Fornecido:**
  `eyJhbGciOiJSUzI1NiIsImtpZCI6IjQ4M0FCMDhFNUYwRDMxNjdEOTRFMTQ3M0FEQTk2RTcyRDkwRUYwRkYiLCJ0eXAiOiJKV1QifQ...`
  - Utilizado no script `scripts/clarity-export.mjs` para extrair métricas de sessão agregadas diretamente para a esteira analítica.
  - Válido até o ano 2126 com escopo `Data.Export`.

### 3. Grafana Loki & Grafana Faro
- **Endpoint:** `https://faro-api-prod-sa-east-1.grafana.net/faro/api/v1`
- **App ID:** `1656`
- **Stack ID:** `1880886`
- **App Name:** `diyspur`
- **Variável de API Key:** `FARO_API_KEY` (configurada no ambiente de build da Vercel para upload de source maps via `FaroSourceMapUploaderPlugin`).

### 4. Prometheus Scraper
- **Endpoint:** `https://diyspur.vercel.app/api/metrics`
- **Autenticação:** `Bearer ${PROMETHEUS_SCRAPE_TOKEN}`
- **Formato:** Texto oficial Prometheus v0.0.4.

---

## Checklist de Aprovação Pré-Deploy

- [ ] Nenhum segredo ou chave privada exposta via `NEXT_PUBLIC_*` ou no bundle JavaScript.
- [ ] O banner de consentimento bloqueia 100% de conexões analíticas externas até o clique em "Aceitar".
- [ ] A revogação em `/privacidade` interrompe o envio de telemetria imediatamente.
- [ ] `npm run typecheck` e `npm run lint` executam sem warnings ou erros em `app`.
- [ ] Bateria de testes unitários e de integração passa com 100% de sucesso.
- [ ] Build de produção (`npm run build`) gera source maps e conclui sem falhas.
- [ ] O endpoint `/api/metrics` responde 401 sem token e 200 com payload Prometheus no cabeçalho autorizado.
- [ ] O endpoint `/api/health` retorna 200 com informações de diagnóstico limpas.
