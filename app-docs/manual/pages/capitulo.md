# Capítulo: Vídeo, Progresso, Discussão Unificada e Quiz / `/capitulos/[id]`

## 1. Overview & Purpose

- **URL Path:** `/capitulos/[id]`
- **Access Level:** Public / Authenticated; gate de sequência pedagógica
- **Core Purpose:** Acompanhar a contextualização em vídeo do trecho, registrar o avanço percentual e conclusão do capítulo, responder quiz/enquete e ingressar diretamente no canal oficial de discussão do capítulo no clube do livro correspondente.

## 2. Prerequisites & Access

- Capítulo publicado e autorizado pela sequência pedagógica.
- Sessão ativa para registrar progresso, responder quiz, votar na pergunta do anfitrião e participar do canal de discussão do clube.

## 3. UI Components & Layout

- **Navegação & Breadcrumbs:** Trilha de retorno para a temporada e indicação do número do capítulo e intervalo de páginas.
- **Fachada de Vídeo (YouTube):** Player carregado sob demanda ao clicar em "Assistir vídeo".
- **Painel de Progresso do Capítulo (`ChapterProgress`):**
  - Slider numérico de 0 a 100% para salvar estimativa de leitura.
  - Botão de ação rápida **"Concluir capítulo"** (registra 100% e concede +20 XP).
- **Entrada Unificada de Discussão (`DiscussionEntryButton`):**
  - Botão **"💬 Entrar no canal de discussão deste capítulo"**.
  - Encaminha o leitor autenticado diretamente para o canal canônico `#cap-X` dentro do clube oficial do livro (`/clubes/[id]?canal=cap-X`), com suporte a barreiras anti-spoiler.
- **Pergunta do Anfitrião:** Enquete interativa do capítulo com resultados agregados.
- **Quiz de Compreensão:** Perguntas de múltipla escolha validadas no backend com concessão de +50 XP.
- **Navegação Sequencial:** Botões táteis para capítulo anterior e próximo capítulo.

## 4. Step-by-Step User Flow

1. **Step 1:** Abra o capítulo pelo cronograma da temporada ativa.
2. **Step 2:** Acompanhe a contextualização em vídeo clicando em **Assistir vídeo**.
3. **Step 3:** Registre sua leitura ajustando o slider de progresso ou clicando em **Concluir capítulo**.
4. **Step 4:** Para debater o enredo com outros leitores, clique em **💬 Entrar no canal de discussão deste capítulo**. O sistema abre o canal específico do capítulo no clube do livro com proteção ativa contra spoilers.
5. **Step 5:** Responda a **Pergunta do anfitrião** e complete o **Quiz do capítulo** para validar a compreensão e acumular XP.
6. **Step 6:** Prossiga para a próxima etapa pelo botão **Próximo capítulo**.

### Interação com o backend

- `getChapters` consulta `chapters`, `seasons`, `books`, `user_progress`, `v_quiz_questions_public`, `host_prompts`, `v_host_prompt_results` e `host_prompt_votes`.
- `saveChapterProgress` faz upsert em `user_progress(user_id, chapter_id)` e dispara `award_xp`.
- `openChapterDiscussionAction` resolve o canal oficial `#cap-X` em `official_book_groups` e `club_channels`.
- O quiz é validado no servidor via Edge Function com header `Idempotency-Key`.

## 5. Error Handling & Edge Cases

- Capítulo bloqueado: exibe aviso de pré-requisito e convite para concluir o quiz anterior.
- Não autenticado: o botão de discussão redireciona para login preservando o destino; o progresso exibe convite de acesso.
- Progresso 100%: uma vez concluído, o status de conclusão é preservado para fins de gamificação.

### Referência de implementação

- [src/app/capitulos/[id]/page.tsx](../../../src/app/capitulos/[id]/page.tsx)
- [src/components/chapters/chapter-progress.tsx](../../../src/components/chapters/chapter-progress.tsx)
- [src/components/discussions/discussion-entry-button.tsx](../../../src/components/discussions/discussion-entry-button.tsx)
- [src/features/chapters/actions.ts](../../../src/features/chapters/actions.ts)
- [src/features/chapters/schemas.ts](../../../src/features/chapters/schemas.ts)
- [src/features/catalog/queries.ts](../../../src/features/catalog/queries.ts)
- [src/features/host-prompts/actions.ts](../../../src/features/host-prompts/actions.ts)

[Índice do manual](../00-overview.md) · [Primeiros passos](../01-getting-started.md) · [Referência técnica](../02-technical-reference.md)
