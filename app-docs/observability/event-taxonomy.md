# Taxonomia Central de Eventos de Produto — DIYSPUR

## 1. Diretrizes de Coleta
- Todos os eventos e suas propriedades passam pela allowlist em `src/lib/analytics/events.ts`.
- Nomes em formato `snake_case`.
- Nenhuma chave fora da lista permitida é transmitida.
- Strings são limitadas a 100 caracteres para evitar injeção acidental de dados grandes ou query strings.

## 2. Dicionário de Eventos

| Evento | Momento do Disparo | Propriedades Permitidas | Propriedades Proibidas |
|---|---|---|---|
| `page_viewed` | Rota client-side exibida após consentimento | `page_path`, `page_type` | Query string completa, email, IDs de conteúdo privado |
| `landing_page_viewed` | Landing page pública visualizada | `source`, `referrer_category` | URL completa do referer com parâmetros de campanha |
| `search_performed` | Busca no catálogo submetida | `query_length`, `result_count`, `search_scope` | Texto integral digitado pelo usuário por padrão |
| `content_viewed` | Detalhes de livro ou temporada pública abertos | `content_category`, `content_type` | Título ou trecho privado, respostas de quiz |
| `content_shared` | Compartilhamento nativo/clipboard acionado | `content_type`, `share_channel` | Token ou URL privada |
| `favorite_added` / `favorite_removed` | Adição ou remoção de favorito concluída | `content_type` | Payload completo, UUIDs externos |
| `cta_clicked` | Clique em CTA promocional/navegação | `cta_id`, `page_type`, `placement` | Texto arbitrário de formulário |
| `sign_up_started` | Formulário de cadastro validado e submetido | `auth_method` | Email, senha, nome |
| `sign_up_completed` | Cadastro concluído com sucesso | `auth_method` | Dados cadastrais completos |
| `login_completed` | Autenticação realizada com sucesso | `auth_method` | Email, senha, tokens JWT |
| `onboarding_started` / `onboarding_completed` | Etapas do onboarding do clube | `step_id` | Respostas pessoais de perfil |
| `activation_completed` | Leitura do primeiro capítulo concluída | `activation_version` | Notas e dados privados de leitura |
| `return_session_started` | Retorno do leitor em nova sessão | `session_type` | IP bruto ou fingerprint |

## 3. Sanitização em Código
A função `sanitizeAnalyticsProperties(event, input)` aplica a filtragem em tempo de execução:
```ts
export function sanitizeAnalyticsProperties(
  event: AnalyticsEvent,
  input: Record<string, string | number | boolean>,
): Record<string, string | number | boolean>
```
Qualquer propriedade não listada explicitamente para o evento correspondente é descartada antes do envio ao PostHog.
