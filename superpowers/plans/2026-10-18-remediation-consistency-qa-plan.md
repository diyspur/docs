# Plano Abrangente de Remediação, Consistência Visual e Cobertura QA DIYSPUR

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Implementar de ponta a ponta todas as correções e evoluções especificadas no SDD de auditoria (QA-01 a QA-28, blocos B01-B34 e fluxos F01-F41), sanando as incompatibilidades de SQL/frontend, padronizando os componentes visuais e estilos semânticos, integrando o console administrativo no app, e validando todos os fluxos com bateria de testes automatizados e QA geral no navegador.

**Architecture:** Frontend e Console construídos em Next.js 16 (App Router, Server Actions, Server Components e Client Components onde interatividade é requerida) com CSS semântico e design tokens centralizados em `src/styles/tokens.css` e `global.css`. Backend gerenciado no PostgreSQL via Supabase com separação estrita entre schemas (`public` e `private`), migrações SQL idempotentes, Row Level Security (RLS) mandatória, e RPCs transacionais com `SECURITY DEFINER` e `SET search_path = ''`.

**Tech Stack:** Next.js 16, React 19, TypeScript 5.9, Supabase SSR / Supabase JS, PostgreSQL 15+ (com testes locais PGlite / WASM), Vitest, Playwright, Chrome DevTools.

**Spec / Referência:** "DIYSPUR — SDD de remediação, consistência visual e cobertura QA (18/10/2026)", achados QA-01 a QA-28, blocos B01-B34 e fluxos F01-F41.

---

## Global Constraints

- **Segurança Fail-Closed:** Nenhuma rota, action ou RPC pode aceitar autenticação sintética, cookies arbitrários ou variáveis públicas para conceder privilégios. Falhas de sessão ou de verificação de permissão negam o acesso (`42501` / `unauthenticated` / `forbidden`).
- **RLS e Isolamento Privado:** Nenhuma tabela em `private` pode ser acessível diretamente via PostgREST. Toda mutação de dados pessoais ou privados valida o proprietário via `auth.uid()`.
- **Transacionalidade e Auditoria:** Alterações editoriais (livro, temporada, capítulo) devem gravar o log de auditoria na mesma transação. Se a auditoria falhar, a alteração inteira sofre rollback.
- **Resiliência e Erros Honestos:** Falhas de consulta retornam uniões discriminadas tipadas (`QueryResult<T>`) e apresentam feedback visual de tentar novamente; jamais converter erro de banco em "0 leitores", "nenhum livro" ou "sucesso vazio".
- **Acessibilidade e Layout Responsivo:** Todos os componentes devem respeitar WCAG 2.2 AA (foco inicial e armadilha em modais, retorno de foco, labels acessíveis, touch targets ≥ 48px, suporte de 320px a 1440px sem overflow horizontal).
- **Sem Duplicação nem Invenção:** Reutilizar estritamente o código e as convenções já existentes nos repositórios `app`, `db`, `admin` e `doc`. Manter classes CSS canônicas sem frameworks adicionais.
- **Qualidade de Código:** 100% de aprovação nos testes unitários, smoke tests de segurança, `npm run typecheck` e `npm run lint` em todos os repositórios (`admin`, `app`, `db`).

---

## Review Focus (Pontos Críticos de Atenção)

1. **Trigger e Contrato de Metas (QA-01, QA-02, QA-03):** `reading_goals` necessita de `updated_at`, e o trigger `refresh_reading_goal_progress` deve despachar corretamente por tabela sem avaliar `EXTRACT(YEAR FROM pg_catalog.current_date)` de forma inválida.
2. **Consentimento de Spoilers e Paginação de Chat (QA-04, QA-05):** O union `accessReason` do frontend deve casar com o retorno real do SQL (`consent_required`, `chapter_unread`, `not_member`), e a paginação deve chamar `list_club_messages` com `p_channel`, `p_before_created_at`, `p_before_id`, `p_limit`.
3. **Estilos de Encontros e Canais (QA-06, QA-07, QA-08):** O formulário de agendamento não pode ter `<main>` aninhado nem classes Tailwind inexistentes; o canal workspace deve adotar grid em 2 colunas com CSS canônico e links acessíveis; metas e clube devem ter grids semânticos próprios.
4. **Busca Paginada no Catálogo (QA-10):** Ao alterar o termo de busca em qualquer página (ex: página 2 ou 4), o parâmetro `page` deve ser removido para sempre resetar à página 1, evitando erro de range PostgREST.
5. **Console Administrativo Nativo (QA-09):** O app principal precisa conter as rotas `/admin`, `/admin/books`, `/admin/seasons/[id]`, `/admin/moderation` e `/admin/challenges` protegidas pelo guard `requireAdmin` e RPC `is_admin`.

---

## Tarefas de Implementação

### Tarefa 1: P01 - Correção Contratual de Metas de Leitura e Triggers no Banco (QA-01, QA-02, QA-03, QA-23)

**Repositório:** `db`

**Arquivos:**
- Criar: `db/supabase/migrations/20261018001500_repair_personal_goals_contract.sql`
- Modificar: `db/scripts/security-smoke/goals-multiple.mjs`

**Interfaces:**
- Consumes: `public.reading_goals`, `private.recalculate_reading_goals_for_user_year`, triggers em `reading_journal_entries`, `user_book_readings`, `user_progress`, `reading_goals`.
- Produces: `ALTER TABLE public.reading_goals ADD COLUMN IF NOT EXISTS updated_at timestamptz NOT NULL DEFAULT now()`, `private.refresh_reading_goal_progress()`.

- [ ] **Passo 1.1: Escrever teste no PGlite reproduzindo os erros 42703 e 42P01**
  Em `db/scripts/security-smoke/goals-multiple.mjs`, remover a adição artificial de `updated_at` do bootstrap inicial e adicionar teste de inserção e atualização em `reading_goals` com o trigger `trg_goal_progress_refresh_goal` ativo.
- [ ] **Passo 1.2: Executar teste e verificar falha esperada**
  Executar: `(cd db/scripts/security-smoke && node goals-multiple.mjs)`
  Esperado: FAIL com `42703: column "updated_at" of relation "reading_goals" does not exist` ou `42P01: missing FROM-clause entry for table "pg_catalog"`.
- [ ] **Passo 1.3: Criar migração SQL `20261018001500_repair_personal_goals_contract.sql`**
  - Adicionar coluna `updated_at` em `public.reading_goals`.
  - Recriar `private.refresh_reading_goal_progress()` com dispatch estrito por `TG_TABLE_NAME`:
    - Para `reading_goals`: `v_new_year := NEW.year`, `v_old_year := OLD.year`.
    - Para `reading_journal_entries`: extrair de `entry_date`.
    - Para `user_book_readings`: extrair de `completed_at` / `updated_at`.
    - Para `user_progress`: extrair de `finished_at` / `updated_at`.
    - Recalcular apenas para anos válidos não nulos.
  - Alinhar validação de limites na RPC `set_personal_reading_goal` (máximo 365 livros, 50.000 páginas, 500.000 minutos).
- [ ] **Passo 1.4: Executar suíte de testes de fumo do banco**
  Executar: `(cd db/scripts/security-smoke && npm test)`
  Esperado: PASS em todos os 9 scripts.
- [ ] **Passo 1.5: Commit das alterações no repo `db`**
  Executar: `git -C db commit -am "fix(db): repair reading_goals contract, add updated_at column, and fix trigger dispatch (QA-01, QA-02, QA-03)"`

---

### Tarefa 2: P02 - Alinhamento de Contrato de Acesso de Canais e Spoiler Consent (QA-04)

**Repositório:** `app`

**Arquivos:**
- Modificar: `app/src/features/discussions/types.ts`
- Criar: `app/src/features/discussions/access.ts`
- Modificar: `app/src/features/discussions/queries.ts`
- Modificar: `app/src/components/discussions/channel-chat.tsx`
- Modificar: `app/src/components/discussions/channel-directory.tsx`
- Modificar: `app/src/app/comunidade/page.tsx`
- Criar: `app/tests/unit/channel-access.test.ts`

**Interfaces:**
- Consumes: RPC `get_club_channel_directory`.
- Produces: `ChannelAccessReason`, helper `resolveChannelAccessState(entry)`, componentes de gate.

- [ ] **Passo 2.1: Escrever teste unitário para `resolveChannelAccessState`**
  Em `app/tests/unit/channel-access.test.ts`, testar:
  - `consent_required` resulta em estado de exigência de consentimento de spoilers.
  - `chapter_unread` resulta em aviso para leitura prévia do capítulo.
  - `not_member` resulta em orientação para ingressar no clube.
  - `banned` e `chapter_locked` resultam em bloqueios estritos.
  - `ok` permite leitura/interação.
- [ ] **Passo 2.2: Executar teste e verificar falha**
  Executar: `(cd app && npm test -- tests/unit/channel-access.test.ts --run)`
  Esperado: FAIL.
- [ ] **Passo 2.3: Implementar `access.ts` e atualizar `types.ts` e `channel-chat.tsx`**
  - Atualizar `types.ts` com o enum canônico retornado pelo SQL: `'ok' | 'unauthenticated' | 'not_member' | 'banned' | 'chapter_locked' | 'chapter_unread' | 'consent_required'`.
  - Criar `access.ts` exportando `resolveChannelAccessState` e schema Zod.
  - Em `channel-chat.tsx`, apresentar `SpoilerConsentGate` quando o motivo for `consent_required`.
  - Em `channel-directory.tsx` e `comunidade/page.tsx`, exibir badges e descrições apropriadas para cada motivo de acesso.
- [ ] **Passo 2.4: Executar testes unitários**
  Executar: `(cd app && npm test -- tests/unit/channel-access.test.ts tests/unit/channel-chat.test.ts --run)`
  Esperado: PASS.
- [ ] **Passo 2.5: Commit das alterações**
  Executar: `git -C app commit -am "fix(discussions): align channel access reason with sql contract and support spoiler consent gate (QA-04)"`

---

### Tarefa 3: P03 - Paginação Histórica de Mensagens via RPC Real (QA-05)

**Repositório:** `app`

**Arquivos:**
- Criar: `app/src/features/discussions/message-page.ts`
- Modificar: `app/src/components/discussions/channel-chat.tsx`
- Criar: `app/tests/unit/message-page.test.ts`

**Interfaces:**
- Consumes: RPC `list_club_messages(p_channel, p_before_created_at, p_before_id, p_limit)`.
- Produces: `paginateEarlierMessages(currentMessages, rawRows, pageSize)`.

- [ ] **Passo 3.1: Escrever teste unitário para `paginateEarlierMessages`**
  Em `app/tests/unit/message-page.test.ts`, validar:
  - Deduplicação de mensagens contra o estado existente.
  - Ordenação correta por `(created_at, id)` ascendente.
  - Identificação de `hasMore` quando retorno for maior que `pageSize`.
- [ ] **Passo 3.2: Executar teste e verificar falha**
  Executar: `(cd app && npm test -- tests/unit/message-page.test.ts --run)`
  Esperado: FAIL.
- [ ] **Passo 3.3: Implementar `message-page.ts` e atualizar `channel-chat.tsx`**
  - Em `message-page.ts`, implementar a lógica de merge de páginas e detecção de excedente.
  - Em `channel-chat.tsx`, corrigir `handleLoadEarlier` para enviar `p_channel: activeChannel.id`, `p_before_created_at: oldest.createdAt`, `p_before_id: oldest.id`, `p_limit: 30`. Tratar erros de rede exibindo alerta recuperável com botão de tentar novamente e manter a posição do scroll.
- [ ] **Passo 3.4: Executar testes unitários**
  Executar: `(cd app && npm test -- tests/unit/message-page.test.ts --run)`
  Esperado: PASS.
- [ ] **Passo 3.5: Commit das alterações**
  Executar: `git -C app commit -am "fix(discussions): correct list_club_messages rpc parameters and cursor pagination (QA-05)"`

---

### Tarefa 4: P04 - Layout e CSS do Workspace e Diretório de Canais (QA-07)

**Repositório:** `app`

**Arquivos:**
- Modificar: `app/src/styles/global.css`
- Modificar: `app/src/components/discussions/channel-directory.tsx`
- Modificar: `app/src/app/clubes/[slug]/canais/[channelSlug]/page.tsx`

**Interfaces:**
- Consumes: CSS tokens de `tokens.css`.
- Produces: Classes `.channel-workspace-grid`, `.channel-workspace-sidebar`, `.channel-workspace-main`, `.channel-directory-link`, `.channel-directory-link[aria-current="page"]`.

- [ ] **Passo 4.1: Adicionar regras no CSS e padronizar JSX**
  - Em `global.css`, definir grid com 2 colunas no desktop (`minmax(0, 18rem) minmax(0, 1fr)`) e 1 coluna no mobile.
  - Estilizar `.channel-directory-link` com min-height 48px, padding adequado, foco acessível e estilo de canal ativo quando `aria-current="page"`.
  - Garantir wrapping de mensagens longas e quebra correta de links.
- [ ] **Passo 4.2: Verificar typecheck e lint**
  Executar: `(cd app && npm run typecheck && npm run lint)`
  Esperado: PASS.
- [ ] **Passo 4.3: Commit das alterações**
  Executar: `git -C app commit -am "style(discussions): reconcile channel workspace grid and directory link classes (QA-07)"`

---

### Tarefa 5: P05 - Refatoração do Formulário de Agendamento e Eliminação de Classes Órfãs (QA-06)

**Repositório:** `app`

**Arquivos:**
- Modificar: `app/src/app/clubes/[slug]/eventos/novo/page.tsx`
- Modificar: `app/src/components/calendar/meeting-form.tsx`
- Modificar: `app/src/styles/global.css`

**Interfaces:**
- Consumes: Classes de formulário semântico (`.field`, `.field-group`, `.form-card`, `.form-actions`).
- Produces: Interface de agendamento de encontros acessível, responsiva e sem `<main>` duplicado.

- [ ] **Passo 5.1: Refatorar página de agendamento e formulário**
  - Em `page.tsx`, substituir `<main>` interno por `<section className="container page-stack">`.
  - Remover classes Tailwind inexistentes (`shell-container`, `text-sm text-muted-foreground`, `card p-6 max-w-2xl`).
  - Em `meeting-form.tsx`, substituir `input-field` por `field`.
  - Adicionar botão "Cancelar" com retorno seguro para a página do clube.
  - Conectar labels, hints e mensagens de erro via `aria-describedby` e `aria-invalid`.
- [ ] **Passo 5.2: Adicionar estilos para `.form-card` e `.page-stack` em `global.css`**
- [ ] **Passo 5.3: Executar testes de formulário e lint**
  Executar: `(cd app && npm test -- tests/unit/datetime-local.test.ts --run && npm run lint)`
  Esperado: PASS.
- [ ] **Passo 5.4: Commit das alterações**
  Executar: `git -C app commit -am "fix(meetings): use semantic fields and eliminate nested main in meeting schedule form (QA-06)"`

---

### Tarefa 6: P06 - Separação de Grids de Metas e Painel do Clube (QA-08)

**Repositório:** `app`

**Arquivos:**
- Modificar: `app/src/app/metas/page.tsx`
- Modificar: `app/src/app/clubes/[slug]/page.tsx`
- Modificar: `app/src/styles/global.css`

- [ ] **Passo 6.1: Criar layouts semânticos no CSS e aplicar nas páginas**
  - Em `global.css`, criar `.goal-layout` com grid responsivo `1.4fr : 0.8fr` no desktop e `.club-overview-layout` com `1fr : 0.6fr`.
  - Em `metas/page.tsx`, substituir `member-grid` por `goal-layout`.
  - Em `clubes/[slug]/page.tsx`, substituir `member-grid` por `club-overview-layout`.
- [ ] **Passo 6.2: Verificar ausência de quebras de layout**
  Executar: `(cd app && npm run typecheck && npm run lint)`
  Esperado: PASS.
- [ ] **Passo 6.3: Commit das alterações**
  Executar: `git -C app commit -am "style(layout): replace rigid member-grid in goals and club overview with semantic layouts (QA-08)"`

---

### Tarefa 7: P07 - Componente Field Reutilizável e Padronização de Componentes UI (QA-19, QA-27)

**Repositório:** `app`

**Arquivos:**
- Criar: `app/src/components/forms/field.tsx`
- Criar: `app/src/components/forms/form-actions.tsx`
- Modificar: `app/src/components/ui/cyber-button.tsx`
- Criar: `app/tests/unit/field.test.tsx`

- [ ] **Passo 7.1: Escrever teste unitário para `Field`**
  Em `app/tests/unit/field.test.tsx`, validar geração de `id`, associação de label, exibição de hint e marcação de `aria-invalid` com mensagem de erro vinculada.
- [ ] **Passo 7.2: Implementar `Field` e `FormActions`**
  - Implementar suporte para `input`, `select` e `textarea`.
  - Limpar classes órfãs em `CyberButton` e outros componentes UI para usarem tokens do sistema.
- [ ] **Passo 7.3: Executar testes unitários**
  Executar: `(cd app && npm test -- tests/unit/field.test.tsx --run)`
  Esperado: PASS.
- [ ] **Passo 7.4: Commit das alterações**
  Executar: `git -C app commit -am "feat(ui): add accessible Field component and clean up orphan ui tokens (QA-19, QA-27)"`

---

### Tarefa 8: P08 - Correção da Busca Paginada no Catálogo (QA-10)

**Repositório:** `app`

**Arquivos:**
- Modificar: `app/src/components/catalog/book-search.tsx`
- Modificar: `app/src/app/livros/page.tsx`
- Modificar: `app/src/features/catalog/queries.ts`
- Criar: `app/tests/unit/catalog-search-pagination.test.ts`

- [ ] **Passo 8.1: Escrever teste para reset de página na busca**
  Em `app/tests/unit/catalog-search-pagination.test.ts`, verificar que a alteração da busca remove o parâmetro `page`.
- [ ] **Passo 8.2: Implementar remoção de `page` no `BookSearch` e resiliência na query do catálogo**
  - Em `book-search.tsx`, garantir `params.delete('page')` no debounce e no submit.
  - Em `livros/page.tsx` e `queries.ts`, quando a página solicitada for superior ao total de páginas existentes, redirecionar de volta para a página 1 em vez de exibir erro de serviço.
- [ ] **Passo 8.3: Executar testes do catálogo**
  Executar: `(cd app && npm test -- tests/unit/catalog-*.test.ts --run)`
  Esperado: PASS.
- [ ] **Passo 8.4: Commit das alterações**
  Executar: `git -C app commit -am "fix(catalog): reset page parameter on search query change and handle out-of-range pages gracefully (QA-10)"`

---

### Tarefa 9: P09 - Consistência de Consulta de Clube e Gestão Unificada (QA-11, QA-12)

**Repositório:** `app`

**Arquivos:**
- Modificar: `app/src/features/clubs/queries.ts`
- Modificar: `app/src/app/clubes/[slug]/page.tsx`
- Modificar: `app/src/components/clubs/club-management.tsx`

- [ ] **Passo 9.1: Incluir `kind` e `official_book_id` na seleção do clube**
  - Em `features/clubs/queries.ts`, adicionar `kind, official_book_id` no `.select()` de `getClubPage`.
  - Em `clubes/[slug]/page.tsx`, repassar `kind` obrigatório para `ClubManagement`.
  - No hub do clube, vincular a temporada ativa ou apresentar CTA contextual claro caso o clube seja oficial de um livro.
- [ ] **Passo 9.2: Verificar typecheck e lint**
  Executar: `(cd app && npm run typecheck && npm run lint)`
  Esperado: PASS.
- [ ] **Passo 9.3: Commit das alterações**
  Executar: `git -C app commit -am "fix(clubs): select kind and official_book_id to maintain consistent club management rules (QA-11, QA-12)"`

---

### Tarefa 10: P10 - Gestão de Visibilidade e Colaboradores em Listas de Leitura (QA-13)

**Repositório:** `app`

**Arquivos:**
- Modificar: `app/src/features/lists/schemas.ts`
- Modificar: `app/src/features/lists/actions.ts`
- Modificar: `app/src/components/lists/create-list-form.tsx`
- Criar: `app/src/components/lists/list-sharing.tsx`
- Modificar: `app/src/app/listas/page.tsx`
- Modificar: `app/src/app/listas/[slug]/page.tsx`
- Criar: `app/tests/unit/list-sharing.test.ts`

- [ ] **Passo 10.1: Escrever teste unitário para schemas e actions de compartilhamento de listas**
- [ ] **Passo 10.2: Implementar opções de visibilidade pública/privada e colaboradores**
  - Em `schemas.ts`, permitir `visibility: 'private' | 'public'` e `isCollaborative: boolean`.
  - Em `actions.ts`, implementar `addListCollaboratorAction` e `removeListCollaboratorAction` utilizando a tabela existente `reading_list_collaborators`.
  - Em `create-list-form.tsx`, permitir selecionar visibilidade ao criar.
  - Criar `list-sharing.tsx` para gerenciar colaboradores com confirmação de visibilidade pública.
- [ ] **Passo 10.3: Executar testes unitários**
  Executar: `(cd app && npm test -- tests/unit/list-*.test.ts --run)`
  Esperado: PASS.
- [ ] **Passo 10.4: Commit das alterações**
  Executar: `git -C app commit -am "feat(lists): support public/private list visibility and collaborator management (QA-13)"`

---

### Tarefa 11: P11 - Reordenação Atômica da Fila de Leitura "Minha Fila" (QA-22)

**Repositórios:** `db`, `app`

**Arquivos:**
- Criar: `db/supabase/migrations/20261018001600_up_next_reorder_rpc.sql`
- Modificar: `app/src/features/up-next/actions.ts`
- Criar: `app/src/components/up-next/queue-management.tsx`
- Modificar: `app/src/app/minha-fila/page.tsx`
- Criar: `app/tests/unit/queue-reorder.test.ts`

- [ ] **Passo 11.1: Criar migração SQL com RPC transacional `reorder_up_next`**
  - Implementar RPC `public.reorder_up_next(p_book_ids uuid[])` com lock por usuário (`pg_advisory_xact_lock`), verificando que todos os IDs pertencem à fila do usuário e atualizando posições de forma atômica.
- [ ] **Passo 11.2: Implementar Server Action e componente de reordenação no app**
  - Em `features/up-next/actions.ts`, implementar `moveQueueItemAction(bookId: string, direction: 'up' | 'down')`.
  - Criar `queue-management.tsx` com botões "Subir" e "Descer" acessíveis por teclado com `aria-label` incluindo o título do livro.
  - Integrar no `minha-fila/page.tsx`.
- [ ] **Passo 11.3: Executar testes de reordenação**
  Executar: `(cd app && npm test -- tests/unit/queue-*.test.ts --run)`
  Esperado: PASS.
- [ ] **Passo 11.4: Commit das alterações em `db` e `app`**
  Executar: `git -C db commit -am "feat(db): add atomic up_next reordering rpc"`
  Executar: `git -C app commit -am "feat(queue): implement accessible queue reordering controls in minha-fila (QA-22)"`

---

### Tarefa 12: P12 - Exportação Integral de Dados com Manifest e Alertas de Truncamento (QA-14)

**Repositório:** `app`

**Arquivos:**
- Modificar: `app/src/app/api/account/export/route.ts`
- Criar: `app/src/features/account/export.ts`
- Criar: `app/tests/unit/account-export.test.ts`

- [ ] **Passo 12.1: Escrever teste de contrato para a exportação de dados**
  Em `app/tests/unit/account-export.test.ts`, validar presença de todas as coleções, manifest estruturado e flags de integridade.
- [ ] **Passo 12.2: Implementar extração completa e manifest**
  - Incluir `reading_list_items`, `reading_list_collaborators`, `user_up_next`, `quiz_attempts`, `user_achievements`.
  - Gerar manifest com contagens exatas por seção e marcar `status: 'complete'` somente quando nenhuma coleção atingir limite sem continuação.
- [ ] **Passo 12.3: Executar testes unitários**
  Executar: `(cd app && npm test -- tests/unit/account-export.test.ts --run)`
  Esperado: PASS.
- [ ] **Passo 12.4: Commit das alterações**
  Executar: `git -C app commit -am "fix(account): provide complete account export with dataset manifest and integrity flags (QA-14)"`

---

### Tarefa 13: P13 - Paginação Completa de Notificações e Reducer Reativo (QA-15, QA-16)

**Repositório:** `app`

**Arquivos:**
- Modificar: `app/src/app/notificacoes/page.tsx`
- Modificar: `app/src/app/notificacoes/queries.ts`
- Modificar: `app/src/app/notificacoes/notification-list.tsx`
- Modificar: `app/src/features/notifications/notification-provider.tsx`

- [ ] **Passo 13.1: Implementar paginação por cursor na página de notificações**
  - Adicionar suporte a carregar mais itens e filtro de não lidas na página `/notificacoes`.
- [ ] **Passo 13.2: Consolidar reducer puro no provider de notificações**
  - Tratar eventos de realtime com deduplicação idempotente por ID, evitando incremento duplo do badge unread.
- [ ] **Passo 13.3: Executar testes de notificações**
  Executar: `(cd app && npm test -- tests/unit/notification-*.test.ts --run)`
  Esperado: PASS.
- [ ] **Passo 13.4: Commit das alterações**
  Executar: `git -C app commit -am "fix(notifications): add cursor pagination to full page and idempotent realtime reducer (QA-15, QA-16)"`

---

### Tarefa 14: P14 - Harmonização Semântica de Perfil e Dicionário de Status (QA-20)

**Repositório:** `app`

**Arquivos:**
- Criar: `app/src/features/reading/presentation.ts`
- Modificar: `app/src/app/perfil/page.tsx`
- Modificar: `app/src/app/livros/[slug]/page.tsx`
- Modificar: `app/src/app/historico/page.tsx`

- [ ] **Passo 14.1: Centralizar mapeamento de status de leitura**
  - Em `presentation.ts`, padronizar: `want_to_read: 'Quero ler'`, `reading: 'Estou lendo'`, `read: 'Concluído'`, `dnf: 'Abandonei'`.
  - Em `perfil/page.tsx`, se `hasActivity === true`, não exibir a mensagem de "Quando sua leitura começar...", mantendo o foco nas conquistas e métricas reais do leitor.
- [ ] **Passo 14.2: Executar testes de perfil**
  Executar: `(cd app && npm run typecheck && npm run lint)`
  Esperado: PASS.
- [ ] **Passo 14.3: Commit das alterações**
  Executar: `git -C app commit -am "fix(profile): harmonize reading status dictionary and fix onboarding bio fallback (QA-20)"`

---

### Tarefa 15: P15 - Tipagem Forte do Supabase e Alinhamento de Limites (QA-23, QA-24)

**Repositório:** `app`

**Arquivos:**
- Modificar: `app/src/features/goals/schemas.ts`
- Modificar: `app/src/components/goals/goal-form.tsx`
- Modificar: `app/src/types/database.ts`
- Modificar: `app/src/lib/supabase/clients.ts`
- Modificar: `app/src/lib/supabase/browser.ts`

- [ ] **Passo 15.1: Alinhar limites numéricos entre Form, Zod e Banco**
  - Form e Zod: Máximo 365 livros, 50.000 páginas, 500.000 minutos.
- [ ] **Passo 15.2: Atualizar tipos do banco e plugar genérico `<Database>`**
  - Atualizar tipos de `reading_goals` em `database.ts` com `updated_at`, `title`, `version`, `archived_at`.
  - Passar `<Database>` em `createServerClient` e `createBrowserClient`.
- [ ] **Passo 15.3: Executar typecheck completo**
  Executar: `(cd app && npm run typecheck)`
  Esperado: PASS sem erros de tipagem.
- [ ] **Passo 15.4: Commit das alterações**
  Executar: `git -C app commit -am "chore(types): bind supabase clients with Database generic and synchronize goal validation limits (QA-23, QA-24)"`

---

### Tarefa 16: P16 - Integração do Console Administrativo no App Principal (QA-09, B31..B34, F01..F03)

**Repositório:** `app`

**Arquivos:**
- Criar: `app/src/lib/auth/require-admin.ts`
- Criar: `app/src/app/admin/layout.tsx`
- Criar: `app/src/app/admin/page.tsx`
- Criar: `app/src/app/admin/books/page.tsx`
- Criar: `app/src/app/admin/seasons/[id]/page.tsx`
- Criar: `app/src/app/admin/moderation/page.tsx`
- Criar: `app/src/app/admin/challenges/page.tsx`
- Criar: `app/src/components/admin/admin-nav.tsx`
- Criar: `app/src/components/admin/book-editor.tsx`
- Criar: `app/src/components/admin/season-editor.tsx`
- Criar: `app/src/components/admin/report-review.tsx`
- Criar: `app/src/features/admin/actions.ts`
- Criar: `app/tests/integration/admin-gate.test.ts`

- [ ] **Passo 16.1: Escrever teste de integração para o guard `requireAdmin`**
  Em `app/tests/integration/admin-gate.test.ts`, validar:
  - Anônimo é redirecionado para `/entrar?redirect=/admin`.
  - Usuário autenticado comum recebe `notFound()` (404) sem revelar existência de rotas administrativas.
  - Administrador autenticado com `rpc('is_admin') = true` obtém acesso autorizado.
- [ ] **Passo 16.2: Implementar `requireAdmin` e layout administrativo**
  - Em `lib/auth/require-admin.ts`, validar sessão e autorização via RPC `is_admin`.
  - Em `app/admin/layout.tsx`, aplicar o guard e renderizar navegação para Catálogo de Livros, Temporadas, Moderação e Desafios.
- [ ] **Passo 16.3: Implementar páginas editoriais e moderação com RPCs transacionais auditadas**
  - Integrar `create_editorial_book`, `update_editorial_book`, `update_editorial_season`, `update_editorial_chapter` e `moderate_content_report`.
  - Em `app/admin/challenges/page.tsx`, fornecer interface para criação e publicação de desafios anuais.
- [ ] **Passo 16.4: Executar testes de integração do admin**
  Executar: `(cd app && npm test -- tests/integration/admin-gate.test.ts --run)`
  Esperado: PASS.
- [ ] **Passo 16.5: Commit das alterações**
  Executar: `git -C app commit -am "feat(admin): port authenticated admin management console to app with transactional audit (QA-09)"`

---

### Tarefa 17: P17 - Atualização da Documentação Técnica e Evidências (QA-26)

**Repositório:** `doc`

**Arquivos:**
- Modificar: `doc/02 - Funcionalidades/02 - Mapeamento de Funcionalidades.md`
- Modificar: `doc/03 - Fluxos e Telas/03 - Jornada do Usuario e Fluxo de Telas.md`
- Modificar: `doc/05 - Qualidade, QA e Operacao/05.1 - Relatorio de QA, Casos de Teste e Quality Gates.md`
- Criar: `doc/05 - Qualidade, QA e Operacao/05.4 - Registro de Reteste e Homologacao 2026-10-18.md`

- [ ] **Passo 17.1: Registrar o reteste, status de homologação e resolução dos achados QA-01 a QA-28**
- [ ] **Passo 17.2: Commit das alterações na documentação**
  Executar: `git -C doc commit -am "docs(qa): document remediation resolution, updated contracts and retest baseline (QA-26)"`

---

### Tarefa 18: P18 - Homologação e Teste Geral de QA no Navegador (B01..B34, F01..F41)

**Repositórios:** Todos (`app`, `db`, `admin`, `doc`)

**Arquivos:**
- Execução no navegador via Chrome DevTools / Playwright.
- Criar script de teste E2E consolidado: `app/tests/e2e/qa-browser-full-flow.test.ts`.

- [ ] **Passo 18.1: Executar todas as suítes de validação automatizada**
  - `(cd db/scripts/security-smoke && npm test)`
  - `(cd app && npm test)`
  - `(cd app && npm run typecheck)`
  - `(cd app && npm run lint)`
  - `(cd admin && npm test && npm run typecheck)`
- [ ] **Passo 18.2: Iniciar servidor de desenvolvimento local**
  - Iniciar Next.js em `http://localhost:3000`.
- [ ] **Passo 18.3: Executar bateria de testes de QA no navegador via Chrome DevTools / Playwright**
  - B01 (Home): Carregamento, Hero, Temporada, Catálogo, agenda, votação e alternância de temas claro/escuro.
  - B02/B03: Menu/Header acessível e Footer institucional.
  - B04 (Catálogo): Busca paginada (página 2 -> busca "Verity" -> reseta para página 1 com 1 resultado).
  - B05 (Ficha do Livro): Adicionar à fila, salvar em lista, ver desafios.
  - B06/B07 (Temporada e Capítulos): Progresso, quiz de 5 questões, desbloqueio de capítulos e canal de debate.
  - B08/B09/B10/B11 (Autenticação): Validação de erros por campo, redirect seguro contra open redirect.
  - B12 (Perfil): Métricas de leitura, XP, streak, histórico recente e bio honesta.
  - B16 (Metas): Criação de meta com objetivos válidos, atualização e arquivamento com bloqueio otimista.
  - B19 (Minha Fila): Adição, remoção e reordenação (subir/descer).
  - B20 (Listas): Criação com visibilidade privada/pública e gestão de colaboradores.
  - B23/B24/B26 (Comunidade e Clubes): Diretório, grupo oficial, canais com e sem spoilers, gate de consentimento.
  - B27 (Calendário e Agendamento): Formulário `/clubes/[slug]/eventos/novo` com campos estilizados e sem `<main>` aninhado.
  - B31..B34 (Admin): Acesso restrito a administradores, edição editorial e moderação.
- [ ] **Passo 18.4: Coletar evidências visuais e gerar relatório de homologação final**
