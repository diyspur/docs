# Referência técnica e contratos frontend/backend

## Fontes e grau de certeza

Frontend `8a5bcd75ac23d1a7030d804deae146c39fb6d7e6` e backend `3d9bf1a14edea87173a346225d84f55558d64a39` foram clonados e lidos. Catálogos PostgreSQL do projeto xjhehhfhhoomblcggjpk foram consultados de forma read-only. Não houve DDL, migration, atualização de policies ou implantação. Não foi verificado que o commit atual de main é exatamente o deploy live.

As referências relativas abaixo abrem os arquivos neste repositório. Backend: https://github.com/diyspur-cloud/db. Tipos sincronizados: [database.ts](../../src/types/database.ts); migrations e Edge Functions no repositório db são a fonte do contrato do banco.

## Autenticação SSR e autorização

1. Browser Auth usa cliente Supabase do browser; páginas SSR usam createServerSupabase.
2. Proxy renova sessão/cookies e aplica gates previstos em lib/supabase/middleware.ts.
3. Páginas/actions usam getClaims/requireSession para identidade verificada.
4. Server Actions validam Zod e derivam userId no servidor, sem aceitar identidade do formulário como autoridade.
5. RLS filtra dados autorizados e controla operações. Admin consulta is_admin em páginas e novamente nas actions.

[Clientes SSR](../../src/lib/supabase/clients.ts), [cliente browser](../../src/lib/supabase/browser.ts), [middleware](../../src/lib/supabase/middleware.ts), [sessão](../../src/lib/auth/session.ts), [redirect seguro](../../src/lib/safe-redirect.ts).

Policies existentes e funções SECURITY DEFINER exigem revisão independente; chamar uma RPC em uma sessão admin do conector não reproduz o papel anon/authenticated de um leitor. Esta rodada não realizou ataques IDOR/BOLA nem testes de isolamento de duas contas. As consultas administrativas de catálogo não substituem testes com JWT de fixture.

## Leituras e mutações por módulo

| Arquivo de contrato | Tabelas, views, RPCs ou Edge Functions referenciadas |
|---|---|
| [src/app/admin/actions.ts](../../src/app/admin/actions.ts) | `is_admin`, `editorial_audit_log`, `create_editorial_book`, `books`, `seasons`, `quiz_questions`, `chapters`, `meetings`, `book_polls`, `review_content_report`, `hide_reported_content` |
| [src/app/api/account/export/route.ts](../../src/app/api/account/export/route.ts) | `profiles`, `reading_journal_entries`, `reading_lists`, `user_progress`, `meeting_rsvps`, `user_club_members` |
| [src/app/api/affiliate/[slug]/route.ts](../../src/app/api/affiliate/[slug]/route.ts) | `books`, `affiliate_clicks` |
| [src/app/api/ics/[meetingId]/route.ts](../../src/app/api/ics/[meetingId]/route.ts) | `meetings` |
| [src/app/desafios/actions.ts](../../src/app/desafios/actions.ts) | `challenges`, `user_challenges` |
| [src/app/desafios/queries.ts](../../src/app/desafios/queries.ts) | `challenges`, `user_challenges` |
| [src/app/notificacoes/actions.ts](../../src/app/notificacoes/actions.ts) | `notifications` |
| [src/app/notificacoes/queries.ts](../../src/app/notificacoes/queries.ts) | `notifications`, `user_club_invitations` |
| [src/app/sitemap.ts](../../src/app/sitemap.ts) | `books`, `seasons`, `chapters` |
| [src/features/catalog/queries.ts](../../src/features/catalog/queries.ts) | `books`, `authors`, `seasons`, `book_content_warnings`, `reading_lists`, `chapters`, `get_season_chapter_access`, `milestones`, `get_user_milestone_status`, `meetings`, `get_chapter_access`, `v_comments_visible`, `user_progress`, `v_quiz_questions_public`, `host_prompts`, `v_host_prompt_results`, `host_prompt_votes` |
| [src/features/chapters/actions.ts](../../src/features/chapters/actions.ts) | `award-xp`, `chapters`, `seasons`, `comments`, `get_own_chapter_comment_for_edit`, `remove_own_chapter_comment`, `user_progress`, `quiz-validate`, `createChapterComment`, `editChapterComment`, `deleteChapterComment`, `getOwnChapterCommentForEdit` |
| [src/features/chapters/schemas.ts](../../src/features/chapters/schemas.ts) | Validação Zod de comentários em capítulos (`commentSchema`: body min 2 max 2000, chapterId uuid) |
| [src/features/clubs/actions.ts](../../src/features/clubs/actions.ts) | `create_user_club`, `join_user_club`, `leave_user_club`, `update_user_club`, `invite_user_to_club`, `v_profiles_public`, `revoke_club_invitation`, `accept_club_invitation`, `remove_club_member` |
| [src/features/clubs/queries.ts](../../src/features/clubs/queries.ts) | `user_clubs`, `user_club_members` |
| [src/features/goals/actions.ts](../../src/features/goals/actions.ts) | `reading_goals` |
| [src/features/goals/queries.ts](../../src/features/goals/queries.ts) | `reading_goals`, `reading_goal_progress` |
| [src/features/host-prompts/actions.ts](../../src/features/host-prompts/actions.ts) | `host_prompts`, `host_prompt_votes` |
| [src/features/journal/actions.ts](../../src/features/journal/actions.ts) | `reading_journal_entries` |
| [src/features/journal/queries.ts](../../src/features/journal/queries.ts) | `reading_journal_entries` |
| [src/features/lists/actions.ts](../../src/features/lists/actions.ts) | `reading_lists`, `add_book_to_reading_list`, `remove_reading_list_item`, `restore_reading_list_item`, `reorder_reading_list` |
| [src/features/meetings/rsvp-actions.ts](../../src/features/meetings/rsvp-actions.ts) | `meeting_rsvps` |
| [src/features/moderation/actions.ts](../../src/features/moderation/actions.ts) | `report_content` |
| [src/features/newsletter/actions.ts](../../src/features/newsletter/actions.ts) | `newsletter_subscribers` |
| [src/features/polls/actions.ts](../../src/features/polls/actions.ts) | `vote-next-book` |
| [src/features/polls/queries.ts](../../src/features/polls/queries.ts) | `book_polls`, `book_poll_options` |
| [src/features/profile/queries.ts](../../src/features/profile/queries.ts) | `user_achievements`, `achievements`, `user_progress`, `chapters`, `seasons`, `books`, `quiz_attempts`, `profiles`, `v_user_reading_overview`, `user_xp`, `user_streaks`, `user_quiz_averages` |
| [src/features/up-next/actions.ts](../../src/features/up-next/actions.ts) | `add_to_up_next_atomic`, `user_up_next` |
| [src/features/up-next/queries.ts](../../src/features/up-next/queries.ts) | `user_up_next` |
| [src/features/discussions/actions.ts](../../src/features/discussions/actions.ts) | `openOfficialBookGroupAction`, `openChapterDiscussionAction`, `postClubMessageAction`, `setSpoilerConsentAction`, `toggleReactionAction`, `club_channels`, `club_messages` |
| [src/features/rankings/queries.ts](../../src/features/rankings/queries.ts) | `v_season_ranking`, `v_book_reading_stats`, `private.mv_book_reading_stats_v2` |

Essa tabela é um inventário de referências no código, não uma prova de execução de todas as chamadas. Componentes podem depender de conteúdo ausente e endpoints podem existir sem controle navegável.

## Revalidação, concorrência e feedback

Server Actions usam revalidatePath após persistência em rotas relevantes. A fila usa RPC atômica para capacidade máxima de cinco livros. Clube usa requestId na criação e version na edição; listas possuem reorder_reading_list com versão. Quiz usa Idempotency-Key; desafios usam upsert ignoreDuplicates. Diário não fornece chave de idempotência no insert atual: repetição após uma falha de transporte ambígua deve ser revisada antes de reenviar.

Concluir capítulo salva progresso e depois pede XP. A action pode devolver sucesso da leitura com sincronização de XP pendente; não converter isso em fracasso de toda a atividade. Quiz depende da Edge Function validar score: o browser não informa um score confiável. Resultados pessoais e respostas corretas não devem ser expostos em views públicas.

## Exportação pessoal: contrato real

[GET /api/account/export](../../src/app/api/account/export/route.ts) exige claims, responde 401 sem identidade e usa no-store. O payload inclui exportedAt, account/profile, journalEntries, readingLists, readingProgress, meetingRsvps e clubMemberships. Limites: diário 200, listas 50, progresso 200, RSVP 100 e vínculos de clube 50.

Não exporta todos os domínios: por exemplo, itens das listas, tentativas/respostas de quiz, votos e notificações não entram no payload atual. Errors individuais das queries não são checados; uma consulta malsucedida pode virar array vazio em HTTP 200. O texto “cópia integral” do perfil é mais abrangente que a implementação. Recomenda-se documentar limites, paginação, domínios e falhas parciais antes de tratar a exportação como completa.

## Loading, empty, error

| Estado | Implementação | Observação |
|---|---|---|
| Carregamento catálogo | src/app/livros/loading.tsx | Carregando leituras… e skeletons observados; a navegação streaming pode primeiro devolver esse estado |
| Empty de busca | getBooks + página catálogo | Busca inexistente exercitada |
| Empty de fila/grupo/notificação | Componentes de cada módulo | Observados com a conta atual |
| Capítulo bloqueado | get_chapter_access + página | Capítulos 2/3 anônimos bloqueados |
| Error de rota | src/app/error.tsx | Reset/Tentar novamente; Sentry; não forçado no navegador |
| Error global | src/app/global-error.tsx | Tela de recuperação mínima; não forçada |
| Erro confundido com vazio | getBooks/getMeetings e algumas queries | Retorno [] em erro; achado por código, não falha de rede reproduzida |

## Responsividade e acessibilidade

CSS em [global.css](../../src/styles/global.css) e [tokens.css](../../src/styles/tokens.css) inclui media queries, controles com min-height/min-width 48px e prefers-reduced-motion. Há [MobileNav](../../src/components/mobile-nav.tsx) e alternador de tema. A presença dessas regras não certifica comportamento em todos os dispositivos.

Desktop desta rodada: home com clientWidth=scrollWidth=1348 e um main. Viewports 375, 640, 768, 860, 1024 e 1440 precisam de execução visual adicional. A API de navegador disponível nesta sessão não expôs configuração de viewport; não foi usada simulação por CSS/zoom como substituto de teste mobile real.

## Atualizar o manual

Quando mudar uma página, revise o guia correspondente, campos, policy/RPC, mensagens, screenshots e cobertura. Use uma fixture em staging para gravar novos cenários. Não copie senhas, JWTs, keys secretas, e-mails de login ou payloads de exportação pessoal para os arquivos publicados. Mantenha evidências limitadas aos estados necessários.

## Contratos backend lidos no repositório db

| Fonte backend | Regra confirmada no código |
|---|---|
| supabase/migrations/20261011204219_reconcile_prod_missing_contracts.sql | can_view_chapter exige não rascunho, temporada ativa/finalizada, data publicada e, após etapa 1, tentativa do quiz da etapa anterior pelo auth.uid; não exige nota mínima nesse predicado |
| supabase/migrations/20261012160000_add_to_up_next_atomic.sql | SECURITY INVOKER; auth.uid obrigatório; advisory lock por usuário; retorna posição existente para duplicata; primeira posição livre 1–5; queue_full se cheia |
| supabase/migrations/20261009025157_restore_admin_predicate.sql | is_admin usa wrapper invoker e helper private com search_path vazio; papel deriva de profiles.role para auth.uid, não de metadados editáveis do cliente |
| supabase/functions/quiz-validate/index.ts | Valida identidade, formato, perguntas/respostas completas, gate do quiz anterior, idempotência e rate limit; usa service client internamente e repete autorização de acesso |

Essas fontes fixam o comportamento do código, não a comprovação de deploy de cada função. O acesso live observado coincide com o gate para os capítulos 2/3 anônimos. A fila atômica e o quiz não foram invocados para novas escritas nesta rodada.

### Exemplo de contrato de acesso, extraído da migration

```sql
c.number = 1 OR EXISTS (
  SELECT 1 FROM public.chapters previous
  JOIN public.quiz_attempts attempt ON attempt.chapter_id = previous.id
    AND attempt.user_id = (SELECT auth.uid())
  WHERE previous.season_id = c.season_id
    AND previous.number = c.number - 1
)
```

Trata-se de trecho explicativo do predicado, não de SQL para aplicar isoladamente. O restante da função também verifica rascunho, estado da temporada e publicação. “Tentativa anterior” não significa aprovação com 100% nem que todas as etapas ficam abertas apenas por login.

### Evidência de exportação iCalendar

O download no navegador retornou arquivo de 468 bytes com VCALENDAR/VEVENT, DTSTART 20261014T230000Z, DTEND 20261015T000000Z e 16 CRLF reais. Isso confirma exportação do evento auditado, sem confirmar importação por Google/Apple/Outlook. Nenhum compromisso externo foi criado.

## Camada de Design System e Componentes Base UI (`src/components/ui/`)

A interface foi padronizada sob uma arquitetura de componentes reutilizáveis e desacoplados, consumindo o Design System baseado em variáveis CSS atômicas declaradas em `src/styles/tokens.css` e classes utilitárias no Tailwind:

| Componente | Arquivo | Responsabilidade Técnica | Contratos de Props |
|---|---|---|---|
| `GlassCard` | `src/components/ui/glass-card.tsx` | Contêiner polimórfico (`as`, default `div`) com efeito de vidro translúcido (`backdrop-blur-md`), bordas reativas e brilho especular. | `as?: ElementType`, `variant?: 'default' \| 'interactive' \| 'ghost'`, `glow?: boolean`, `children`, `className` |
| `CyberButton` | `src/components/ui/cyber-button.tsx` | Botão tátil futurista com micro-interação GPU (`active:scale-[0.98]`), foco com anel neon e alto contraste WCAG AAA. | `variant?: 'primary' \| 'secondary' \| 'ghost' \| 'danger'`, `size?: 'sm' \| 'md' \| 'lg' \| 'icon'`, `glow?: boolean`, `asChild?: boolean`, `disabled`, `className` |
| `CyberSkeleton` | `src/components/ui/cyber-skeleton.tsx` | Skeleton loader sem salto de layout (CLS = 0.000) com animação `shimmer` em gradiente angular de 60fps. | `variant?: 'text' \| 'circular' \| 'rectangular' \| 'card'`, `width?: string \| number`, `height?: string \| number`, `glow?: boolean`, `className` |
| `BookCard` | `src/components/books/book-card.tsx` | Card de catálogo/estante com hover especular, badges de estado de leitura e imagem otimizada via Next/Image. | `book: BookCatalogItem`, `priority?: boolean`, `showReadingStatus?: boolean` |
| `SiteHeader` | `src/components/layout/site-header.tsx` | Header global fixo com frosted glass (`backdrop-blur-md`), navegação semântica ARIA e menu mobile responsivo. | `currentUser?: SessionUser \| null`, `notificationsCount?: number` |

