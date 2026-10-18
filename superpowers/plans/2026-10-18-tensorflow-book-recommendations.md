# TensorFlow.js Book Recommendations Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Integrar o TensorFlow.js no DIYSpur Reading Club para ranquear o catálogo em `/livros`: recomendar livros no topo com base nas leituras ativas e concluídas (`reading` e `read`), mantendo livros já lidos, em leitura ou abandonados (`read`, `reading`, `dnf`) no final da lista.

**Architecture:** Módulo de recomendação autônomo baseado em `@tensorflow/tfjs` com vetorização de features (tags, autor, metadados), projeção de perfil de preferências do usuário ponderado pelo histórico de leitura, cálculo vetorial de similaridade por cosseno em `tf.tidy()` (sem vazamento de memória de tensores), e ordenação particionada (Candidatos ranqueados por afinidade no topo -> Livros lidos/lendo/dnf no rodapé). Integrado de forma segura no Server Component e queries de `/livros` com fallback transparente para visitantes e usuários sem histórico.

**Tech Stack:** Next.js 16 (App Router, Server Components), TypeScript 5.9, React 19, `@tensorflow/tfjs` 4.22.0 (CPU backend), Supabase Auth & PostgreSQL (`user_book_readings`), Vitest.

**Spec:** Baseado na especificação solicitada pelo usuário e nos contratos canônicos de leitura (`user_book_readings`, `Book`, `getBooksResult`).

## Global Constraints

- Nunca quebrar contratos existentes de `getBooksResult` ou rotas de API.
- Fallback gracioso imediato para usuários deslogados (visitantes) ou usuários sem histórico de leitura.
- Gestão estrita de memória do TensorFlow.js: qualquer operação de tensores DEVE executar dentro de `tf.tidy()` para garantir que `tf.memory().numTensors` não cresça.
- Sem dependências de canvas/WebGL nativas em tempo de execução no servidor (uso compatível e leve de CPU).
- Preservar integridade de estilo, acessibilidade e conformidade com CSP.

## Review Focus

1. **Cold-start (Visitante ou usuário sem leitura):** Deve exibir o catálogo na ordem padrão sem nenhum erro ou atraso perceptível.
2. **Recomendação com afinidade:** Se o usuário leu ou está lendo um livro de "Suspense psicológico", livros com tags e autores similares devem ter maior score e aparecer no início.
3. **Exclusão/Rebaixamento para o final:** Livros com status `read`, `reading` e `dnf` DEVEM ficar obrigatoriamente no final da lista.
4. **Vazamento de Tensores:** Ausência de memory leaks — `tf.memory().numTensors` deve retornar ao valor inicial após cada execução.
5. **Busca e Paginação:** Personalização deve respeitar filtros de busca (`q`) e fatiamento de páginas.

---

### Task 1: Instalação e Tipagens de Recomendação

**Files:**
- Create: `app/src/features/recommendations/types.ts`
- Modify: `app/src/features/catalog/queries.ts:5-17`

- [ ] **Step 1: Definir tipos do módulo de recomendação**
Criar `app/src/features/recommendations/types.ts` contendo:
- `BookFeatureVector`: representação numérica do livro.
- `UserReadingSignal`: status (`read`, `reading`, `dnf`, `want_to_read`) e peso atribuído.
- `RankedBookResult`: livro com score, flags (`isRecommended`, `readingStatus`).

- [ ] **Step 2: Estender interface `Book` em `features/catalog/queries.ts`**
Adicionar campos opcionais compatíveis:
- `readingStatus?: 'want_to_read' | 'reading' | 'read' | 'dnf' | null;`
- `isRecommended?: boolean;`
- `recommendationScore?: number;`

---

### Task 2: Motor de Recomendação TensorFlow.js (TDD)

**Files:**
- Create: `app/src/features/recommendations/tensorflow-recommender.ts`
- Create: `app/tests/unit/recommendations.test.ts`

- [ ] **Step 1: Escrever testes unitários em `app/tests/unit/recommendations.test.ts`**
Cobrir:
1. Vetorização de tags e autores.
2. Cálculo de similaridade por cosseno com `tf.tidy()`.
3. Priorização de candidatos similares no topo para usuário com livros lidos/lendo.
4. Posicionamento de livros `read`, `reading` e `dnf` no final da lista.
5. Estabilidade de memória do TensorFlow (`tf.memory().numTensors`).
6. Fallback resiliente em caso de dados inválidos ou tensores zerados.

- [ ] **Step 2: Implementar `tensorflow-recommender.ts`**
Funções essenciais:
- `buildFeatureVocabulary(books: Book[])`
- `vectorizeBook(book: Book, vocab: FeatureVocab): number[]`
- `computeTensorFlowRecommendations(candidates: Book[], historyBooks: { book: Book; status: 'read' | 'reading' }[]): Map<string, number>`
- `rankCatalogBooks(books: Book[], userReadings: Map<string, BookReadingStatus>): Book[]`

- [ ] **Step 3: Executar vitest e garantir 100% de aprovação**
Rodar `npm test app/tests/unit/recommendations.test.ts`.

---

### Task 3: Integração no Catálogo (`queries.ts` e `/livros/page.tsx`)

**Files:**
- Modify: `app/src/features/catalog/queries.ts:49-102`
- Modify: `app/src/app/livros/page.tsx:30-154`
- Modify: `app/src/components/book-card.tsx:1-33`
- Modify: `app/src/styles/global.css`

- [ ] **Step 1: Atualizar `getBooksResult` para carregar `user_book_readings` se autenticado**
Quando `readerId` estiver presente, consultar `user_book_readings` do usuário e aplicar `rankCatalogBooks()` antes de paginar. Se deslogado, manter o fluxo padrão.

- [ ] **Step 2: Atualizar `BookCard` e estilos para exibir badges**
- Badge `★ Recomendado` para livros com alta afinidade calculada.
- Badges de leitura para livros no rodapé (`Lendo`, `Lido`, `Desistiu`).
- Adicionar estilos correspondentes em `global.css`.

- [ ] **Step 3: Atualizar `/livros/page.tsx` para apresentar contexto de personalização**
Adicionar indicador sutil quando a listagem for personalizada com base nas leituras do usuário.

---

### Task 4: QA Completo, Performance e Segurança

**Files:**
- Test: `app/tests/unit/recommendations.test.ts`
- Run: `npm test`
- Run: `npm run typecheck`
- Run: `npm run lint`
- Run: `npm run build`

- [ ] **Step 1: Testes automatizados completos**
Rodar suíte inteira de testes unitários do app (47+ arquivos) garantindo zero regressões.

- [ ] **Step 2: Checagem de tipagem e lint**
Rodar `npm run typecheck` e `npm run lint`.

- [ ] **Step 3: Build de produção Next.js**
Rodar `npm run build` para garantir que o empacotamento do TensorFlow.js não cause problemas de compilação ou bundle.

- [ ] **Step 4: Auditoria de Performance e Segurança**
- Medir tempo de computação dos tensores (< 5ms).
- Verificar que dados de outros usuários não vazam (RLS em `user_book_readings`).
- Checar `npm run audit`.
