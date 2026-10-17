# Auditoria de Marketing, CRO e Taxonomia de Eventos — DIYSPUR

**Data da análise:** 11 de outubro de 2026  
**Status:** Camada tipada de telemetria alinhada com governança Zero-PII e consentimento explícito.

---

## 1. Proposta de Valor e Comunicação

O DIYSPUR se posiciona como um clube de leitura comunitário, coletivo e acolhedor, onde os livros ganham vida ao serem lidos juntos. O design e copy enfatizam:
- **Leitura em Temporadas**: Sem pressa, capítulo por capítulo;
- **Debates ao Vivo**: Encontros marcados com hora e fuso explícitos;
- **Comunidade sem Spoilers**: Proteção ao leitor contra revelações prematuras;
- **Diário Privado**: Espaço seguro para anotações reflexivas.

---

## 2. CTAs Principais e Otimização de Conversão (CRO)

| Localização | Texto do CTA | Destino | Objetivo de Conversão |
|---|---|---|---|
| **Hero da Home** | "Explorar livros" | `/livros` | Engajar visitante no catálogo |
| **Card em Destaque** | "Ver temporada do livro" | `/temporadas/[slug]` | Apresentar o cronograma |
| **Ficha do Livro** | "Comprar exemplar na Amazon" | `/api/affiliate/[slug]` | Conversão de leitor com exemplar físico/digital |
| **Ficha do Livro** | "Compartilhar leitura" | Modal / Web Share | Viralização e convite de novos membros |
| **Formulário de Entrada** | "Entrar na comunidade" | Callback / Perfil | Retenção e ativação de leitor |

---

## 3. Taxonomia de Eventos (Governança Zero-PII)

Todos os eventos rastreados utilizam identificadores pseudonimizados (UUID) e propriedades estritamente agregadas. **É terminantemente proibido enviar nomes, e-mails, senhas, termos de busca brutos, conteúdo do diário ou comentários para o PostHog ou qualquer ferramenta de analytics.**

| Nome do Evento | Disparo | Propriedades Permitidas |
|---|---|---|
| `catalog_viewed` | Página `/livros` carregada | `source_path` |
| `book_search_submitted` | Busca realizada | `has_query: boolean`, `query_length: number` |
| `book_detail_viewed` | Ficha do livro acessada | `book_slug`, `has_cover: boolean` |
| `calendar_viewed` | Agenda de encontros aberta | `meetings_count: number` |
| `meeting_rsvp_intent` | Clique em confirmação de presença | `meeting_id` (UUID público) |
| `auth_started` | Início do fluxo de login/cadastro | `mode: 'signin' | 'signup'`, `reason` |
| `sign_up_completed` | Cadastro finalizado com sucesso | `auth_method: 'password' | 'magic_link'` |
| `spoiler_revealed` | Leitor clica para ver spoiler | `component: 'comment' | 'chapter'` |

---

## 4. Consentimento e Conformidade com a LGPD

- **Privacy by Default**: Nenhum evento ou cookie de rastreamento comportamental é disparado antes da aceitação explícita no banner de cookies (`AnalyticsConsentBanner`).
- **Opt-in / Opt-out**: Usuário pode aceitar ou recusar através do banner, e alterar sua preferência a qualquer momento no rodapé.
