# Referência de RLS — snapshot read-only

Projeto consultado: xjhehhfhhoomblcggjpk, em 12/10/2026 Brasília. Fonte: pg_class/pg_namespace para relrowsecurity e pg_policies para políticas. A consulta encontrou **82 tabelas public**, todas com RLS habilitada. Isso não certifica predicados, privilégios das views ou comportamento de funções privilegiadas.

Não foram realizadas alterações. Somente policies dos módulos pessoais selecionados são reproduzidas abaixo; não se trata de inventário integral de todas as policies nem de prova de isolamento com JWT de duas contas.

## Predicados efetivamente consultados

| Tabela | Policy | Comando | Roles | USING | WITH CHECK |
|---|---|---|---|---|---|
| comments | comments_delete_own_or_admin | DELETE | {authenticated} | `((user_id = ( SELECT auth.uid() AS uid)) OR ( SELECT is_admin() AS is_admin))` | `` |
| comments | comments_insert_accessible | INSERT | {authenticated} | `` | `((user_id = ( SELECT auth.uid() AS uid)) AND private.can_view_chapter(chapter_id))` |
| comments | comments_read_active | SELECT | {anon,authenticated} | `(deleted_at IS NULL)` | `` |
| comments | comments_update_own | UPDATE | {authenticated} | `(user_id = ( SELECT auth.uid() AS uid))` | `(user_id = ( SELECT auth.uid() AS uid))` |
| notifications | notif_own | ALL | {public} | `(( SELECT auth.uid() AS uid) = user_id)` | `(( SELECT auth.uid() AS uid) = user_id)` |
| profiles | profiles_insert_own | INSERT | {authenticated} | `` | `(id = ( SELECT auth.uid() AS uid))` |
| profiles | profiles_select_public_fields | SELECT | {anon,authenticated} | `true` | `` |
| profiles | profiles_update_own | UPDATE | {authenticated} | `(( SELECT auth.uid() AS uid) = id)` | `(( SELECT auth.uid() AS uid) = id)` |
| reading_goals | goals_own | ALL | {public} | `(user_id = ( SELECT auth.uid() AS uid))` | `(user_id = ( SELECT auth.uid() AS uid))` |
| reading_journal_entries | journal_delete_own | DELETE | {authenticated} | `(user_id = ( SELECT auth.uid() AS uid))` | `` |
| reading_journal_entries | journal_insert_own | INSERT | {authenticated} | `` | `(user_id = ( SELECT auth.uid() AS uid))` |
| reading_journal_entries | journal_read_own | SELECT | {public} | `((user_id = ( SELECT auth.uid() AS uid)) OR ((archived_at IS NULL) AND ((visibility = 'public'::journal_visibility) OR ((visibility = 'friends'::journal_visibility) AND (EXISTS ( SELECT 1    FROM follows f   WHERE ((f.follower_id = ( SELECT auth.uid() AS uid)) AND (f.followed_id = reading_journal_entries.user_id) AND (f.status = 'accepted'::follow_status))))) OR ((visibility = 'club'::journal_visibility) AND (EXISTS ( SELECT 1    FROM (user_club_members m1      JOIN user_club_members m2 ON ((m1.club_id = m2.club_id)))   WHERE ((m1.user_id = ( SELECT auth.uid() AS uid)) AND (m2.user_id = reading_journal_entries.user_id))))))))` | `` |
| reading_journal_entries | journal_update_own | UPDATE | {authenticated} | `(user_id = ( SELECT auth.uid() AS uid))` | `(user_id = ( SELECT auth.uid() AS uid))` |
| reading_list_items | list_items_delete_authorized | DELETE | {authenticated} | `private.can_edit_reading_list(list_id)` | `` |
| reading_list_items | list_items_insert_authorized | INSERT | {authenticated} | `` | `(((added_by IS NULL) OR (added_by = ( SELECT auth.uid() AS uid))) AND private.can_edit_reading_list(list_id))` |
| reading_list_items | list_items_read | SELECT | {public} | `(EXISTS ( SELECT 1    FROM reading_lists l   WHERE ((l.id = reading_list_items.list_id) AND ((l.visibility = 'public'::reading_list_visibility) OR (l.owner_id = ( SELECT auth.uid() AS uid)) OR (EXISTS ( SELECT 1            FROM reading_list_collaborators c           WHERE ((c.list_id = l.id) AND (c.user_id = ( SELECT auth.uid() AS uid)))))))))` | `` |
| reading_list_items | list_items_update_authorized | UPDATE | {authenticated} | `private.can_edit_reading_list(list_id)` | `private.can_edit_reading_list(list_id)` |
| reading_lists | lists_owner_write | ALL | {public} | `(owner_id = ( SELECT auth.uid() AS uid))` | `(owner_id = ( SELECT auth.uid() AS uid))` |
| reading_lists | lists_read | SELECT | {anon,authenticated} | `private.can_view_reading_list(id)` | `` |
| user_club_members | uclub_members_delete_authorized | DELETE | {authenticated} | `((user_id = ( SELECT auth.uid() AS uid)) OR (EXISTS ( SELECT 1    FROM user_clubs c   WHERE ((c.id = user_club_members.club_id) AND (c.owner_id = ( SELECT auth.uid() AS uid))))))` | `` |
| user_club_members | uclub_members_insert_authorized | INSERT | {authenticated} | `` | `(((user_id = ( SELECT auth.uid() AS uid)) AND (role = 'member'::text) AND (EXISTS ( SELECT 1    FROM user_clubs c   WHERE ((c.id = user_club_members.club_id) AND ((NOT c.is_private) OR (c.owner_id = ( SELECT auth.uid() AS uid))))))) OR ((role = ANY (ARRAY['owner'::text, 'moderator'::text, 'member'::text])) AND (EXISTS ( SELECT 1    FROM user_clubs c   WHERE ((c.id = user_club_members.club_id) AND (c.owner_id = ( SELECT auth.uid() AS uid)))))))` |
| user_club_members | uclub_members_read_visible | SELECT | {public} | `((user_id = auth.uid()) OR (EXISTS ( SELECT 1    FROM user_clubs c   WHERE ((c.id = user_club_members.club_id) AND (c.owner_id = auth.uid())))) OR private.can_view_user_club(club_id))` | `` |
| user_club_members | uclub_members_update_authorized | UPDATE | {authenticated} | `((user_id = ( SELECT auth.uid() AS uid)) OR (EXISTS ( SELECT 1    FROM user_clubs c   WHERE ((c.id = user_club_members.club_id) AND (c.owner_id = ( SELECT auth.uid() AS uid))))))` | `(((user_id = ( SELECT auth.uid() AS uid)) AND (role = 'member'::text) AND (EXISTS ( SELECT 1    FROM user_clubs c   WHERE ((c.id = user_club_members.club_id) AND ((NOT c.is_private) OR (c.owner_id = ( SELECT auth.uid() AS uid))))))) OR ((role = ANY (ARRAY['owner'::text, 'moderator'::text, 'member'::text])) AND (EXISTS ( SELECT 1    FROM user_clubs c   WHERE ((c.id = user_club_members.club_id) AND (c.owner_id = ( SELECT auth.uid() AS uid)))))))` |
| user_clubs | uclubs_owner | ALL | {public} | `(( SELECT auth.uid() AS uid) = owner_id)` | `(( SELECT auth.uid() AS uid) = owner_id)` |
| user_clubs | uclubs_read_public | SELECT | {public} | `private.can_view_user_club(id)` | `` |
| user_progress | progress_self_delete | DELETE | {authenticated} | `(user_id = ( SELECT auth.uid() AS uid))` | `` |
| user_progress | progress_self_insert | INSERT | {authenticated} | `` | `((user_id = ( SELECT auth.uid() AS uid)) AND private.can_view_chapter(chapter_id))` |
| user_progress | progress_self_read | SELECT | {public} | `(( SELECT auth.uid() AS uid) = user_id)` | `` |
| user_progress | progress_self_update | UPDATE | {authenticated} | `(user_id = ( SELECT auth.uid() AS uid))` | `((user_id = ( SELECT auth.uid() AS uid)) AND private.can_view_chapter(chapter_id))` |
| user_up_next | up_next_own | ALL | {public} | `(user_id = ( SELECT auth.uid() AS uid))` | `(user_id = ( SELECT auth.uid() AS uid))` |

SELECT filtra linhas visíveis; WITH CHECK controla o novo estado de escrita. Funções private.can_view_* e private.can_edit_* devem ser lidas no backend para compreender privacidade/colaboração. Policies permissivas se combinam; ler uma única policy não comprova o resultado combinado. Policies não substituem grants de acesso ao schema/tabela.

## Pontos de atenção

- Diário: titular pode escrever; leitura de registros não arquivados também considera visibility public/friends/club. O formulário auditado sempre grava private.
- Listas: escrita da lista é do owner; leitura usa can_view_reading_list; itens usam can_edit_reading_list.
- Progresso: escrita pessoal também exige can_view_chapter. O gate precisa ser validado com a sessão correta.
- Clubes: acesso privado passa por can_view_user_club; ingressos e papéis têm predicados distintos. RPCs precisam preservar a mesma regra.
- Perfil: a exposição pública deve ser analisada junto da view pública e campos selecionados; não concluir que a tabela inteira é pública.
- Tabelas agregadas como ranking/XP podem ter leitura ampla por desenho do produto. Revisar granularidade e consentimento com a operação; não confundir público agregado com dado pessoal privado.

## Reproduzir a consulta

```sql
SELECT c.relname, c.relrowsecurity
FROM pg_class AS c
JOIN pg_namespace AS n ON n.oid = c.relnamespace
WHERE n.nspname = 'public' AND c.relkind = 'r'
ORDER BY c.relname;

SELECT tablename, policyname, roles, cmd, qual, with_check
FROM pg_policies
WHERE schemaname = 'public'
ORDER BY tablename, policyname;
```

Essas consultas requerem acesso operacional autorizado aos catálogos. Para certificação funcional, use staging e duas fixtures com dados pessoais distintos; consulte como cada role e compare leitura/escrita permitidas e negadas. Não use conexão privilegiada como prova de que a RLS funciona para um leitor.
