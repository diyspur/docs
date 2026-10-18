# Plano Abrangente de Implementação: UI, Navegação, Comunidades, Responsividade e Segurança RLS

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Implementar as melhorias de UI no rodapé (ícones animados para redes sociais, ícones para links institucionais e remoção do botão de privacidade flutuante), substituir a opção de magic link na tela de login por "Esqueci minha senha", reorganizar a navegação retirando "Clubes" do header e integrando os clubes com seus chats dentro da página `/comunidade`, corrigir a quebra de layout responsivo em `/livros/[slug]` (ex: `/livros/verity`) e habilitar Row Level Security (RLS) nas 9 tabelas do schema `private` com políticas apropriadas sem quebrar operações backend ou RPCs.

**Architecture:** A aplicação utiliza Next.js 16 (App Router) no frontend com CSS vanilla estruturado em design tokens (`tokens.css`, `global.css`), e PostgreSQL gerenciado via Supabase no backend com separação estrita de schemas (`public` e `private`), migrações idempotentes e RPCs seguras (`SECURITY DEFINER` e `SECURITY INVOKER`). As correções respeitam a arquitetura orientada a grupos canônicos por livro e garantem conformidade com o Supabase Advisor e práticas de defesa em profundidade.

**Tech Stack:** Next.js 16, React 19, TypeScript 5.9, Supabase SSR, PostgreSQL 15+ (pglite nos testes de smoke), Vitest, Playwright.

**Spec / Referência Técnica:** `doc/01 - Arquitetura/01.1 - Modelo de Dados e Seguranca RLS.md`, `doc/02 - Funcionalidades/02.4 - Comunidade de Leitura, Clubes e Canais de Discussao.md` e migrações M01–M16 em `db/supabase/migrations/`.

---

## Global Constraints

- **Segurança de Dados:** Nenhuma tabela em `private` deve ser acessível diretamente via PostgREST por `anon` ou `authenticated`. Acesso estrito por `service_role` e funções `SECURITY DEFINER`.
- **Zero Quebras no Backend:** O habilitamento de RLS nas 9 tabelas não pode bloquear edge functions, cron jobs (`pg_cron`) nem RPCs transacionais já homologadas.
- **Mobile First e Acessibilidade:** Todas as novas interfaces devem ser responsivas a partir de 320px de largura de tela, respeitar `prefers-reduced-motion` para animações e ter tags ARIA adequadas.
- **Consistência de Tipos:** Manter sincronizados os tipos de banco de dados entre `app/src/types/database.ts` e `db/src/lib/supabase/database.types.ts`.
- **Padrão de Código:** Preservar comentários existentes, lint limpo (`npm run lint`), tipagem TypeScript estrita sem erros (`npm run typecheck`) e testes automatizados passando.

---

## Review Focus (Pontos Críticos de Atenção)

1. **Tentativa de leitura de canais por visitantes anônimos em `/comunidade`:** A RPC `get_club_channel_directory` não possuía concessão `GRANT EXECUTE TO anon`, o que causaria erro 42501 e retorno de lista vazia de chats na página de comunidades para quem não estiver logado. A migração deve conceder execute seguro para `anon`.
2. **Políticas de RLS para `service_role` em schema `private`:** Embora `service_role` costume ter bypass de RLS no Supabase, ambientes locais, PGlite ou queries parametrizadas podem falhar sem política explícita. Cada uma das 9 tabelas terá uma política `FOR ALL TO service_role USING (true) WITH CHECK (true)`.
3. **Quebra de grid no mobile em `/livros/[slug]`:** `.book-hero-grid` não tinha estilos CSS definidos no `global.css`. Em telas móveis (< 768px), o layout deve mudar de 2 colunas para 1 coluna centralizada, evitando que capas de 320px causem overflow horizontal.
4. **Remoção de Magic Link no Login:** Garantir que o botão "Esqueci minha senha" direcione para `/recuperar-senha` preservando o parâmetro `redirect` para que o usuário não perca seu destino original.
5. **Acessibilidade do Rodapé:** Os ícones SVG animados de redes sociais devem possuir `aria-hidden="true"` enquanto a tag `<a>` correspondente deve conter texto legível e/ou `aria-label` descritivo.

---

## Estrutura de Arquivos a Serem Modificados / Criados

```
db/
├── supabase/migrations/
│   └── 20261018001000_enable_rls_on_private_tables.sql       # [Criar] Migração para RLS e políticas
└── scripts/security-smoke/
    └── community-discussions.mjs                             # [Modificar] Adicionar assertions de RLS das 9 tabelas

app/
├── src/
│   ├── components/
│   │   ├── analytics/
│   │   │   └── analytics-consent-banner.tsx                 # [Modificar] Remover botão flutuante sob o footer
│   │   ├── site-shell.tsx                                   # [Modificar] Footer com ícones animados e SVG
│   │   ├── auth-form.tsx                                    # [Modificar] Substituir magic link por esqueci senha
│   │   ├── desktop-nav.tsx                                  # [Modificar] Remover Clubes do header
│   │   └── mobile-nav.tsx                                   # [Modificar] Remover Clubes da gaveta mobile
│   ├── app/
│   │   ├── comunidade/
│   │   │   └── page.tsx                                     # [Modificar] Listar clubes e chats/canais
│   │   ├── privacidade/
│   │   │   └── page.tsx                                     # [Modificar] Botão contextual para preferências
│   │   └── livros/[slug]/
│   │       └── page.tsx                                     # [Modificar] Ajustes de classes e responsividade
│   └── styles/
│       └── global.css                                       # [Modificar] Estilos do book-hero, footer icons, etc.
└── tests/
    └── e2e/
        └── auth.spec.ts                                     # [Modificar] Validar botão Esqueci Minha Senha
```

---

## Tarefas de Implementação

### Tarefa 1: Habilitar RLS e Políticas de Segurança nas 9 Tabelas do Schema `private`

**Arquivos:**
- Criar: `db/supabase/migrations/20261018001000_enable_rls_on_private_tables.sql`
- Modificar: `db/scripts/security-smoke/community-discussions.mjs`

**Interfaces:**
- Consome: Tabelas existentes em `private` criadas nas migrações anteriores.
- Produz: RLS ativo nas 9 tabelas, políticas para `service_role`, e concessão de `EXECUTE ON FUNCTION public.get_club_channel_directory(uuid) TO anon`.

- [ ] **Passo 1: Criar a migração `20261018001000_enable_rls_on_private_tables.sql`**

Criar o arquivo com o código SQL para habilitar RLS nas 9 tabelas, criar políticas explícitas permissivas para `service_role` e liberar a execução da RPC de diretório para leitura anônima dos chats de clubes:

```sql
-- Migration: 20261018001000_enable_rls_on_private_tables.sql
-- Description: Enable RLS on 9 private schema tables with service_role policies and grant get_club_channel_directory to anon

-- 1. Habilitar RLS em todas as tabelas do schema private
ALTER TABLE private.quiz_rate_limits ENABLE ROW LEVEL SECURITY;
ALTER TABLE private.sent_reminders ENABLE ROW LEVEL SECURITY;
ALTER TABLE private.newsletter_dispatch_runs ENABLE ROW LEVEL SECURITY;
ALTER TABLE private.stripe_customers ENABLE ROW LEVEL SECURITY;
ALTER TABLE private.rate_limits ENABLE ROW LEVEL SECURITY;
ALTER TABLE private.book_reading_requests ENABLE ROW LEVEL SECURITY;
ALTER TABLE private.meeting_operation_requests ENABLE ROW LEVEL SECURITY;
ALTER TABLE private.community_notification_outbox ENABLE ROW LEVEL SECURITY;
ALTER TABLE private.book_share_invitations ENABLE ROW LEVEL SECURITY;

-- 2. Criar políticas de acesso irrestrito para service_role (tarefas de background, workers e edge functions)
DO $$
BEGIN
  IF NOT EXISTS (SELECT 1 FROM pg_policies WHERE schemaname = 'private' AND tablename = 'quiz_rate_limits' AND policyname = 'service_role_all') THEN
    CREATE POLICY service_role_all ON private.quiz_rate_limits FOR ALL TO service_role USING (true) WITH CHECK (true);
  END IF;

  IF NOT EXISTS (SELECT 1 FROM pg_policies WHERE schemaname = 'private' AND tablename = 'sent_reminders' AND policyname = 'service_role_all') THEN
    CREATE POLICY service_role_all ON private.sent_reminders FOR ALL TO service_role USING (true) WITH CHECK (true);
  END IF;

  IF NOT EXISTS (SELECT 1 FROM pg_policies WHERE schemaname = 'private' AND tablename = 'newsletter_dispatch_runs' AND policyname = 'service_role_all') THEN
    CREATE POLICY service_role_all ON private.newsletter_dispatch_runs FOR ALL TO service_role USING (true) WITH CHECK (true);
  END IF;

  IF NOT EXISTS (SELECT 1 FROM pg_policies WHERE schemaname = 'private' AND tablename = 'stripe_customers' AND policyname = 'service_role_all') THEN
    CREATE POLICY service_role_all ON private.stripe_customers FOR ALL TO service_role USING (true) WITH CHECK (true);
  END IF;

  IF NOT EXISTS (SELECT 1 FROM pg_policies WHERE schemaname = 'private' AND tablename = 'rate_limits' AND policyname = 'service_role_all') THEN
    CREATE POLICY service_role_all ON private.rate_limits FOR ALL TO service_role USING (true) WITH CHECK (true);
  END IF;

  IF NOT EXISTS (SELECT 1 FROM pg_policies WHERE schemaname = 'private' AND tablename = 'book_reading_requests' AND policyname = 'service_role_all') THEN
    CREATE POLICY service_role_all ON private.book_reading_requests FOR ALL TO service_role USING (true) WITH CHECK (true);
  END IF;

  IF NOT EXISTS (SELECT 1 FROM pg_policies WHERE schemaname = 'private' AND tablename = 'meeting_operation_requests' AND policyname = 'service_role_all') THEN
    CREATE POLICY service_role_all ON private.meeting_operation_requests FOR ALL TO service_role USING (true) WITH CHECK (true);
  END IF;

  IF NOT EXISTS (SELECT 1 FROM pg_policies WHERE schemaname = 'private' AND tablename = 'community_notification_outbox' AND policyname = 'service_role_all') THEN
    CREATE POLICY service_role_all ON private.community_notification_outbox FOR ALL TO service_role USING (true) WITH CHECK (true);
  END IF;

  IF NOT EXISTS (SELECT 1 FROM pg_policies WHERE schemaname = 'private' AND tablename = 'book_share_invitations' AND policyname = 'service_role_all') THEN
    CREATE POLICY service_role_all ON private.book_share_invitations FOR ALL TO service_role USING (true) WITH CHECK (true);
  END IF;
END $$;

-- 3. Conceder permissao de execucao ao role anon para leitura publica do diretorio de canais
GRANT EXECUTE ON FUNCTION public.get_club_channel_directory(uuid) TO anon;
```

- [ ] **Passo 2: Atualizar testes de segurança em `db/scripts/security-smoke/community-discussions.mjs`**

Adicionar verificação de que todas as 9 tabelas possuem RLS habilitado e que consultas diretas de `authenticated` ou `anon` falham / retornam 0 linhas.

- [ ] **Passo 3: Executar a suite de testes de smoke do banco**

Comando: `cd db/scripts/security-smoke && npm test`
Resultado esperado: Todas as migrações aplicadas com sucesso e todos os testes passando.

---

### Tarefa 2: Ajuste no Rodapé (Footer) e Preferências de Privacidade

**Arquivos:**
- Modificar: `app/src/components/analytics/analytics-consent-banner.tsx`
- Modificar: `app/src/components/site-shell.tsx`
- Modificar: `app/src/app/privacidade/page.tsx`
- Modificar: `app/src/styles/global.css`

**Interfaces:**
- Consome: Variáveis de design tokens e funções de consentimento de analytics.
- Produz: Rodapé visualmente limpo, com ícones animados nas redes e ícones SVG nos links institucionais; remoção do botão de privacidade abaixo do footer.

- [ ] **Passo 1: Remover o botão flutuante em `app/src/components/analytics/analytics-consent-banner.tsx`**

Remover a renderização condicional do botão `<button className="privacy-preferences-trigger">` quando `!open`, mantendo apenas o diálogo `<aside className="analytics-consent">` quando `open === true`.

- [ ] **Passo 2: Adicionar componente interativo na página de privacidade `app/src/app/privacidade/page.tsx`**

Adicionar um botão estilizado na Seção 3 ("Telemetria e Analytics opcional") que permite ao usuário abrir o banner de preferências de consentimento sob demanda diretamente na página de privacidade.

- [ ] **Passo 3: Criar ícones SVG com animações CSS em `app/src/components/site-shell.tsx`**

Atualizar `SiteFooter` para incluir:
- **Redes Sociais (com animações CSS):**
  - **Instagram:** Ícone gradiente com efeito hover de leve escala, rotação e iluminação (`instagram-pulse`).
  - **TikTok:** Ícone nota musical com efeito hover de deslocamento cromático (duplo drop-shadow ciano/magenta).
  - **YouTube:** Ícone retangular com play button que expande suavemente no hover (`youtube-glow`).
- **Links Institucionais:**
  - **Privacidade:** Ícone Shield / Escudo com checkmark.
  - **Termos:** Ícone Document / Prancheta de regras.
  - **Suporte:** Ícone Help / Bóia salva-vidas ou envelope de contato.

- [ ] **Passo 4: Adicionar estilos e keyframes em `app/src/styles/global.css`**

Adicionar classes `.footer-nav-link`, `.footer-social-link`, `.footer-social-icons` com suporte a `:hover`, `:focus-visible` e media query `@media (prefers-reduced-motion: reduce)` desativando transições bruscas.

- [ ] **Passo 5: Verificar build e tipagem**

Comando: `npm run typecheck && npm run test` na pasta `app`.

---

### Tarefa 3: Tela de Login - Substituir Magic Link por "Esqueci minha senha"

**Arquivos:**
- Modificar: `app/src/components/auth-form.tsx`
- Modificar: `app/tests/e2e/auth.spec.ts`

**Interfaces:**
- Consome: `/recuperar-senha` rota e parâmetros de redirecionamento.
- Produz: Formulário de login limpo, com foco em senha e botão secundário direto para recuperação de senha.

- [ ] **Passo 1: Remover lógica de `magicLink` em `app/src/components/auth-form.tsx`**

1. Remover a função `async function magicLink()`.
2. No bloco `{!signup && (...)}`, remover o `<button className="button button-quiet" onClick={magicLink}>Receber um link de acesso</button>` e o antigo `<Link className="text-link auth-recovery-link">Esqueci minha senha</Link>`.
3. No lugar do botão do magic link, inserir:
```tsx
<Link
  className="button button-quiet"
  href={{ pathname: '/recuperar-senha', query: { redirect: destination } }}
>
  Esqueci minha senha
</Link>
```

- [ ] **Passo 2: Atualizar testes E2E em `app/tests/e2e/auth.spec.ts`**

Adicionar teste Playwright garantindo que:
- O botão "Esqueci minha senha" está visível na tela de login.
- Clicar nele navega para `/recuperar-senha?redirect=...`.
- O texto "Receber um link de acesso" não está mais presente na tela.

- [ ] **Passo 3: Rodar testes unitários e de integração**

Comando: `npm run test` na pasta `app`.

---

### Tarefa 4: Header - Remover "Clubes" da Navegação

**Arquivos:**
- Modificar: `app/src/components/desktop-nav.tsx`
- Modificar: `app/src/components/mobile-nav.tsx`

**Interfaces:**
- Consome: Rotas do Next.js.
- Produz: Header despoluído, onde o usuário acessa livros diretamente ou entra em comunidades para acessar clubes e chats.

- [ ] **Passo 1: Atualizar `app/src/components/desktop-nav.tsx`**

1. Substituir o dropdown "Explorar" (que só continha Livros e Clubes) por um link direto `<Link href="/livros">Livros</Link>` com a classe `active` quando `pathname.startsWith('/livros') || pathname.startsWith('/temporadas') || pathname.startsWith('/capitulos')`.
2. No menu dropdown "Comunidade", atualizar o primeiro item:
   - Label: `Clubes e Conversas`
   - Descrição: `Grupos oficiais de leitura, chats e debates`
   - Destino: `/comunidade`

- [ ] **Passo 2: Atualizar `app/src/components/mobile-nav.tsx`**

1. No array `defaultSections`:
   - Em `Explorar`, manter apenas `{ href: '/livros', label: 'Explorar livros' }`, removendo `{ href: '/clubes', label: 'Clubes de leitura' }`.
   - Em `Comunidade`, atualizar para `{ href: '/comunidade', label: 'Clubes e Conversas' }`.

- [ ] **Passo 3: Rodar testes de navegação**

Comando: `npm run test tests/unit/member-navigation.test.ts` e `npm run typecheck`.

---

### Tarefa 5: Página `/comunidade` - Clubes com Lista de Chats Integrados

**Arquivos:**
- Modificar: `app/src/app/comunidade/page.tsx`
- Modificar: `app/src/styles/global.css`

**Interfaces:**
- Consome: RPC `get_club_channel_directory` e tabela `user_clubs`.
- Produz: Interface consolidada de comunidades onde cada clube apresenta seus canais de discussão (chats gerais e por capítulo) com badges de spoiler e link direto para entrar na conversa.

- [ ] **Passo 1: Atualizar a busca de dados em `app/src/app/comunidade/page.tsx`**

1. Buscar os clubes oficiais e seus respectivos livros.
2. Para cada clube, invocar `getClubChannelDirectory(club.id)` para carregar a lista de chats/canais.
3. Montar estrutura tipada:
```typescript
interface ClubWithChannels {
  id: string;
  slug: string;
  name: string;
  description: string | null;
  book: {
    id: string;
    slug: string;
    title: string;
    cover_url: string | null;
    synopsis: string | null;
  } | null;
  channels: ClubChannelDirectoryEntry[];
}
```

- [ ] **Passo 2: Renderizar os chats dentro de cada card de clube**

1. Exibir cabeçalho do clube com capa do livro, título e sinopse.
2. Renderizar bloco de canais com subtítulos "Canais gerais" e "Discussão por capítulo".
3. Cada canal mostra seu nome (ex: `#geral-sem-spoilers`, `#capitulo-1`) e seu pill (`Sem spoiler`, `Cap. liberado`, `Spoiler total`).
4. Clicar no canal leva diretamente a `/clubes/${club.slug}/canais/${channel.slug}`.
5. Botão de ação "Acessar clube completo" direcionando para `/clubes/${club.slug}`.

- [ ] **Passo 3: Adicionar estilização no `app/src/styles/global.css`**

Adicionar estilos para `.community-club-card`, `.community-channels-preview`, `.community-channel-tag` com suporte a tema claro e escuro.

- [ ] **Passo 4: Testar a renderização com e sem login**

Verificar via testes unitários ou teste de componente que usuários autenticados e anônimos visualizam a lista de clubes e canais sem erros de permissão.

---

### Tarefa 6: Correção de Layout e Responsividade em `/livros/[slug]` (ex: `/livros/verity`)

**Arquivos:**
- Modificar: `app/src/app/livros/[slug]/page.tsx`
- Modificar: `app/src/styles/global.css`

**Interfaces:**
- Consome: Dados do livro (`getBook(slug)`), dados da edição e controle de leitura.
- Produz: Layout de página de livro perfeitamente alinhado no desktop (capa à esquerda, textos e ações à direita com espaçamento harmonioso) e 100% responsivo no mobile (sem quebra horizontal, capa proporcional, tipografia legível).

- [ ] **Passo 1: Estruturar as classes de layout em `app/src/app/livros/[slug]/page.tsx`**

1. Garantir que `<header className="book-hero">` envolva o grid principal.
2. Substituir estilos inline do `.book-purchasing-panel` por classes utilitárias no CSS (`book-purchasing-panel`, `book-purchasing-actions`).
3. Adicionar classes nos elementos de tipografia e meta tags para garantir hierarquia visual consistente.

- [ ] **Passo 2: Adicionar as regras CSS responsivas em `app/src/styles/global.css`**

Implementar no CSS:
```css
/* =====================================================================
   Book Hero & Detail Responsive Layout
   ===================================================================== */
.book-hero {
  margin-block: var(--space-6) var(--space-10);
}

.book-hero-grid {
  display: grid;
  grid-template-columns: 1fr;
  gap: var(--space-6);
  align-items: start;
}

@media (min-width: 768px) {
  .book-hero-grid {
    grid-template-columns: 280px 1fr;
    gap: var(--space-8);
  }
}

@media (min-width: 1024px) {
  .book-hero-grid {
    grid-template-columns: 320px 1fr;
    gap: var(--space-10);
  }
}

.book-cover-wrap {
  width: 100%;
  max-width: 260px;
  margin-inline: auto;
}

@media (min-width: 768px) {
  .book-cover-wrap {
    max-width: 100%;
    position: sticky;
    top: calc(var(--header-height, 64px) + 24px);
  }
}

.book-cover-image {
  width: 100%;
  height: auto;
  aspect-ratio: 2 / 3;
  object-fit: cover;
  border-radius: var(--radius-md);
  box-shadow: 0 16px 36px -8px rgba(0, 0, 0, 0.4);
  border: 1px solid var(--border);
  display: block;
}

.book-cover-placeholder {
  width: 100%;
  aspect-ratio: 2 / 3;
  border-radius: var(--radius-md);
  background: var(--surface-2);
  border: 1px solid var(--border);
  display: flex;
  align-items: center;
  justify-content: center;
  padding: var(--space-4);
  text-align: center;
  color: var(--text-tertiary);
}

.book-hero-content {
  display: flex;
  flex-direction: column;
  gap: var(--space-3);
  min-width: 0;
}

.book-hero-content h1 {
  font-size: clamp(26px, 4vw, 40px);
  line-height: 1.15;
  margin: 0;
  color: var(--text-primary);
}

.book-author {
  font-size: var(--text-body);
  color: var(--text-secondary);
  margin: 0;
}

.book-author strong {
  color: var(--text-primary);
}

.book-synopsis {
  font-size: var(--text-body);
  line-height: 1.65;
  color: var(--text-secondary);
  margin-top: var(--space-2);
}

.book-meta-tags {
  display: flex;
  flex-wrap: wrap;
  gap: var(--space-2);
  margin-top: var(--space-2);
}

.book-purchasing-panel {
  margin-top: var(--space-6);
  padding: var(--space-5);
  background: var(--surface-2);
  border-radius: var(--radius-md);
  border: 1px solid var(--border);
}

.book-purchasing-actions {
  display: flex;
  gap: 12px;
  flex-wrap: wrap;
  align-items: center;
}

@media (max-width: 640px) {
  .book-purchasing-actions {
    flex-direction: column;
    align-items: stretch;
  }
  .book-purchasing-actions > * {
    width: 100%;
    text-align: center;
    justify-content: center;
  }
}
```

- [ ] **Passo 3: Verificar testes e responsividade em múltiplos viewports**

Rodar testes e checar integridade de compilação: `npm run build` ou `npm run test` e validação com Playwright se configurado.

---

### Tarefa 7: Auditoria Profunda App & DB (Supabase vs App, Bugs, Gaps e Melhores Práticas)

**Arquivos:**
- Atualizar: `doc/01 - Arquitetura/01.1 - Modelo de Dados e Seguranca RLS.md` (adicionar seção M17 e status de RLS)
- Documentar no relatório final os apontamentos de arquitetura, consistência e performance.

- [ ] **Passo 1: Auditoria de Integridade de Schemas e Tipos**
  - Conferir se os tipos gerados no frontend cobrem 100% das tabelas públicas.
  - Verificar se existem colunas nulas não tratadas no código Next.js.
- [ ] **Passo 2: Auditoria de Triggers e Políticas RLS**
  - Validar se todas as 82 tabelas públicas e 9 tabelas privadas possuem RLS habilitado.
  - Testar RPCs de moderação, avaliações, metas e progresso.
- [ ] **Passo 3: Auditoria de Performance e Web Vitals**
  - Revisar Lazy Loading de imagens (`next/image`), `priority` na capa do livro e scripts inline do tema.

---

## Validação e Critérios de Conclusão

1. **Testes do Banco:** `cd db/scripts/security-smoke && npm test` passa 100%.
2. **Testes do Frontend:** `cd app && npm run typecheck && npm run test && npm run lint` sem erros nem warnings.
3. **Layout `/livros/verity`:** Capa alinhada ao lado das informações no desktop e empilhada responsivamente no mobile.
4. **Header e Footer:** Sem botão de preferências solto abaixo do rodapé, ícones animados para redes sociais, ícones para outros links, e remoção de "Clubes" do header com redirecionamento de fluxo para `/comunidade`.
5. **Login:** Opção de link de acesso removida e substituída pelo botão "Esqueci minha senha" funcional.
