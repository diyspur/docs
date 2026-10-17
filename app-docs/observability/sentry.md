# Arquitetura e Configuração do Sentry — DIYSPUR

## 1. Visão Geral
O Sentry é o pilar de observabilidade técnica e tratamento de exceções da aplicação DIYSPUR (`app`), capturando falhas não tratadas, exceções capturadas por error boundaries e transições de rotas de forma estruturada e sem vazamento de dados pessoais (PII).

## 2. Componentes e Arquivos
- **`src/instrumentation-client.ts`**: Inicializa o SDK no navegador, configura `beforeSend` para higienização e captura spans de transição de rota.
- **`src/sentry.server.config.ts`**: Inicializa o SDK no runtime Node.js do Next.js (SSR, Server Components, Route Handlers).
- **`src/sentry.edge.config.ts`**: Inicializa o SDK para runtimes Edge.
- **`src/instrumentation.ts`**: Hook padrão do Next.js (`register()`) e captura global de erros de requisição (`onRequestError = Sentry.captureRequestError`).
- **`next.config.mjs`**: Envolve a configuração com `withSentryConfig` da subrota `@sentry/nextjs/config`.
- **`src/app/error.tsx`**: Error boundary de rotas que captura a exceção verdadeira com tag `boundary: 'app-route'` e `nextDigest`.
- **`src/app/global-error.tsx`**: Error boundary de nível raiz para capturar falhas no layout global.

## 3. Política de Privacidade e Higienização (Zero-PII)
- **`beforeSend` sanitization**: Remove cabeçalhos sensíveis, cookies, dados de corpo (`event.request.data`) e remove query strings e fragmentos de URLs.
- **Identificação Pseudonimizada**: Usuários são identificados estritamente pelo UUID interno do Supabase (`user.id`). Nomes, emails e metadados de autenticação nunca são enviados ao Sentry.
- **Logout**: No evento `SIGNED_OUT`, `Sentry.setUser(null)` é chamado imediatamente para desvincular a sessão anterior.

## 4. Variáveis de Ambiente
- `NEXT_PUBLIC_SENTRY_DSN`: DSN público para o bundle do navegador.
- `SENTRY_DSN`: DSN do servidor/edge (pode ser o mesmo ou específico).
- `SENTRY_ENVIRONMENT`: `development`, `preview` ou `production`.
- `NEXT_PUBLIC_SENTRY_TRACES_SAMPLE_RATE`: Taxa de amostragem no cliente (padrão 0.1 em produção, 1.0 em dev).
- `SENTRY_TRACES_SAMPLE_RATE`: Taxa de amostragem no servidor.
- `SENTRY_AUTH_TOKEN`, `SENTRY_ORG`, `SENTRY_PROJECT`: Utilizados exclusivamente durante o build de CI para upload seguro de source maps.

## 5. Source Maps
Configurado com `deleteSourcemapsAfterUpload: true`, garantindo que os mapas de código sejam enviados ao servidor do Sentry e imediatamente excluídos dos artefatos estáticos públicos expostos pelo Vercel.
