# Arquitetura e Controles do PostHog — DIYSPUR

## 1. Visão Geral
O PostHog é utilizado exclusivamente para telemetria de jornadas de produto (ativação, exploração do catálogo e engajamento) na aplicação DIYSPUR (`app`), operando estritamente sob consentimento do usuário (Privacy by Default).

## 2. Princípios de Privacidade e LGPD
- **Nenhum evento antes do consentimento**: O PostHog permanece completamente inativo até que o usuário clique em "Aceitar analytics" no banner de consentimento.
- **Opt-out e revogação**: O usuário pode recusar inicialmente ou alterar suas preferências a qualquer momento através do gatilho *Preferências de privacidade* presente no rodapé de todas as páginas.
- **`autocapture: false`**: Captura automática de cliques e formulários desabilitada para evitar gravação acidental de inputs.
- **`capture_pageview: false` / `capture_pageleave: false`**: Controlado explicitamente através de rotas higienizadas.
- **`respect_dnt: true`**: Respeita o cabeçalho Do Not Track do navegador.
- **`person_profiles: 'identified_only'`**: Não cria perfis anônimos desnecessários no banco do PostHog.
- **Sem Session Replay**: Replay de sessão desativado por padrão para proteger a privacidade dos leitores e reduzir consumo de dados.

## 3. Gestão de Identidade e Sessão
- **Identificação Pseudonimizada**: Ao autenticar, o usuário é identificado unicamente pelo UUID do Supabase (`user.id`).
- **Zero-PII**: Email, nome de exibição, senha e tokens nunca são enviados como propriedades de identidade ou eventos.
- **Reset no Logout**: Ao disparar o evento `SIGNED_OUT` no listener do Supabase, `posthog.reset()` é executado imediatamente para limpar tokens de sessão e identidade do dispositivo.

## 4. Componentes de Implementação
- **`src/lib/analytics/consent.ts`**: Armazena e lê a escolha no `localStorage` sob a chave `diyspur.analytics-consent.v1` e despacha eventos no `window`.
- **`src/lib/analytics/client.ts`**: Singleton protegido com inicialização condicional e sanitização de propriedades por allowlist.
- **`src/lib/analytics/events.ts`**: Dicionário de eventos permitidos e propriedades limitadas.
- **`src/components/analytics/posthog-provider.tsx`**: Provider no layout raiz que monitora rotas e sincroniza o estado de consentimento e autenticação.
- **`src/components/analytics/analytics-consent-banner.tsx`**: Interface acessível com botões de aceite, recusa e reabertura de preferências.

## 5. Variáveis de Ambiente
- `NEXT_PUBLIC_POSTHOG_KEY`: Chave pública de ingestão do projeto.
- `NEXT_PUBLIC_POSTHOG_HOST`: Endpoint da região do projeto (ex.: `https://us.i.posthog.com`).
