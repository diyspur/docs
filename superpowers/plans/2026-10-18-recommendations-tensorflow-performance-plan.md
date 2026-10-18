# Recomendações e Performance do TensorFlow.js no DIYSPUR - Plano de Implementação

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Remediar os problemas críticos de consistência na personalização de catálogo, eliminar o limite rígido de 500 livros, otimizar a performance de computação vetorial implementando motor JS nativo e comparando com TensorFlow.js, aprimorar a qualidade das recomendações (TF-IDF, decaimento temporal, diversificação MMR e explicabilidade) e assegurar scroll/paginação consistente para usuários autenticados e anônimos.

**Architecture:** Arquitetura desacoplada em 4 camadas:
1. **Camada de Dados & Caching (`queries.ts`, `recommender-cache.ts`):** Projeção leve de atributos para ranking (id, tags, author_id), ordenação particionada armazenada em cache de IDs por sessão/busca com invalidação automática, e paginação profunda sem corte arbitrário em 500 itens.
2. **Camada de Motor de Recomendação (`recommender-engine.ts`, `native-recommender.ts`, `tensorflow-recommender.ts`):** Vocabulário indexado com lookup $O(1)$, vetorização com normalização por livro e TF-IDF para tags raras, pontuação por similaridade cosseno (com adapter intercambiável Pure JS vs TensorFlow.js), decaimento temporal de interações e diversificação por MMR (Maximal Marginal Relevance).
3. **Camada de API & Cliente (`route.ts`, `personalized-catalog.tsx`, `browser-storage.ts`):** Endpoint de catálogo paginado que preserva ordenação global, componente cliente com carregamento dinâmico (lazy import) que mescla estados de leitura do servidor (`reading`, `read`, `dnf`) com sinais locais sem perda de estado, e rastreamento ponderado de cliques e visualizações.
4. **Camada de Apresentação & Explicabilidade (`book-card.tsx`):** Exibição de motivos contextuais de recomendação ("Recomendado por afinidade com Suspense", "Do mesmo autor de X") e badges de status de leitura.

**Tech Stack:** Next.js 16 (App Router, Server Actions, Route Handlers), React 19, TypeScript 5.9, `@tensorflow/tfjs` 4.22.0 (CPU backend), Supabase PostgreSQL, Vitest.

**Spec:** Baseado na Análise Técnica: Recomendações e Performance do TensorFlow.js no DIYSPUR, abrangendo consistência de catálogo, eliminação de limites de paginação, benchmark TF.js vs JS puro, vocabulário indexado, enriquecimento de sinais de leitura e diversificação de recomendações.

---

## Global Constraints

- Nunca quebrar contratos existentes de `getBooksResult` ou tipos públicos de `Book`.
- Preservar integridade dos status de leitura (`reading`, `read`, `dnf`, `want_to_read`) em toda a jornada de renderização (servidor e cliente).
- Garantir que qualquer usuário autenticado ou visitante consiga paginar/navegar por 100% do catálogo cadastrado, sem travar em limites de 500 livros.
- Se o TensorFlow.js for executado, qualquer alocação de tensores DEVE permanecer contida em `tf.tidy()` sem vazamentos de tensores (`tf.memory().numTensors`).
- O bundle inicial do cliente em `/livros` não deve importar `@tensorflow/tfjs` de forma estática; o módulo deve ser carregado sob demanda ou substituído pelo motor nativo JS quando comprovada equivalência.
- Testes unitários com Vitest devem manter 100% de aprovação e zero regressões em todos os 48+ arquivos de teste do projeto.

---

## Review Focus

1. **Fusão Servidor + Cliente sem perda de status:** Se um usuário autenticado possui livros lidos/lendo retornados pelo servidor e o navegador possui interações locais em `localStorage`, a reclassificação do cliente NÃO pode perder os status `read`/`reading`/`dnf` nem mover livros concluídos para o topo dos candidatos.
2. **Paginação profunda (> 500 livros):** Em um catálogo com 1.200 livros, o usuário deve conseguir navegar além da página 21 (offset 504) e visualizar os livros restantes até a última página sem receber lista vazia.
3. **Equivalência numérica exata (TF.js vs Native JS):** O motor em JavaScript puro deve produzir scores de similaridade cosseno com diferença máxima de $\le 10^{-5}$ em relação ao TensorFlow.js para os mesmos vetores.
4. **Decaimento temporal e distinção de sinais:** Cliques múltiplos em um mesmo livro devem acumular peso ponderado com decaimento temporal, e visualizações de página de detalhes não devem sobrepor ou anular o histórico de cliques como se fossem o mesmo sinal.
5. **Diversidade via MMR em catálogo homogêneo:** Um usuário que leu apenas livros de um único autor/gênero não deve receber 24 recomendações idênticas consecutivas se houver outros títulos de alta relevância no catálogo.

---

## Mapeamento de Arquivos e Responsabilidades

| Arquivo | Ação | Responsabilidade |
| :--- | :--- | :--- |
| `app/src/features/recommendations/types.ts` | Modificar | Definir tipos para vocabulário indexado, sinais ponderados com decaimento, explicabilidade (`recommendationReason`), opções de diversificação MMR e interface `RecommenderEngine`. |
| `app/src/features/recommendations/browser-storage.ts` | Modificar | Separar múltiplos tipos de interação por livro, armazenar contagem agregada com timestamp de última ocorrência, calcular decaimento temporal e expor função de extração agregada. |
| `app/src/features/recommendations/native-recommender.ts` | Criar | Implementar motor de recomendação em JavaScript puro: produto escalar, normas euclidianas, similaridade cosseno, vocabulário indexado com `Map`, TF-IDF e diversificação MMR. |
| `app/src/features/recommendations/tensorflow-recommender.ts` | Modificar | Refatorar para utilizar vocabulário indexado $O(1)$, TF-IDF, normalização por livro e suporte a explicabilidade, mantendo compatibilidade com tensores em `tf.tidy()`. |
| `app/src/features/recommendations/recommender-engine.ts` | Criar | Padrão Strategy para alternar transparentemente entre `native` e `tensorflow`, além de funções utilitárias unificadas para ranking e explicabilidade. |
| `app/src/features/recommendations/recommender-cache.ts` | Criar | Gerenciador de cache em memória/sessão para lista de IDs classificados com TTL, evitando recomputação de ranking a cada requisição de página/scroll. |
| `app/src/features/catalog/queries.ts` | Modificar | Remover limite rígido de 500 registros, separar projeção enxuta para cálculo de relevância (`id, author_id, tags`) da consulta de hidratação dos 24 cards da página, integrar cache de ranking. |
| `app/src/app/api/books/catalog/route.ts` | Criar | Endpoint REST paginado (`GET /api/books/catalog?page=2&limit=24&q=...`) para alimentar scroll infinito e paginação assíncrona com consistência de ranking. |
| `app/src/components/catalog/personalized-catalog.tsx` | Modificar | Carregar motor de recomendação dinamicamente (`import()`), preservar o mapa de leituras do servidor ao combinar interações locais, e suportar scroll infinito integrado com a API. |
| `app/src/components/book-card.tsx` | Modificar | Renderizar badge explicativa com tooltip ou texto secundário quando `recommendationReason` estiver preenchido. |
| `app/tests/unit/recommendations.test.ts` | Modificar | Expandir suíte com os 7 cenários críticos solicitados na análise técnica. |
| `app/tests/unit/recommender-benchmark.test.ts` | Criar | Benchmark automatizado comparando latência, consumo de memória, equivalência numérica e precisão entre TensorFlow.js e Native JS. |
| `app/tests/unit/catalog-pagination.test.ts` | Criar | Testes de integração unitária para paginação profunda (> 500 livros) e integridade do endpoint `/api/books/catalog`. |

---

## Fase 1 — Corrigir a Consistência do Ranking e Paginação

### Task 1.1: Preservar Estados de Leitura do Servidor no Cliente

**Files:**
- Modify: `app/src/components/catalog/personalized-catalog.tsx:1-62`
- Test: `app/tests/unit/recommendations.test.ts`

**Interfaces:**
- Consumes: `Book`, `BookReadingStatus` de `@/features/catalog/queries`, `getStoredInteractions` de `@/features/recommendations/browser-storage`.
- Produces: `PersonalizedCatalog` com re-ranking client-side sem perda de status `read`, `reading`, `dnf`.

- [ ] **Step 1: Escrever teste falhando para fusão de leituras do servidor com interações locais**

Em `app/tests/unit/recommendations.test.ts`, adicionar teste garantindo que livros com `readingStatus` já presentes em `initialBooks` não percam o status nem voltem ao topo quando `rankCatalogBooks` for acionado apenas com interações locais:

```typescript
it('preserves server-provided reading statuses when re-ranking with client browser interactions', () => {
  const booksWithStatus: Book[] = [
    { ...bookA, readingStatus: 'read' },
    { ...bookB, readingStatus: 'reading' },
    { ...bookC, readingStatus: 'dnf' },
    { ...bookD, readingStatus: undefined },
    { ...bookE, readingStatus: undefined },
  ];

  // Extrair mapa de leituras preservadas
  const preservedReadings = new Map<string, BookReadingStatus>(
    booksWithStatus
      .filter((b) => b.readingStatus)
      .map((b) => [b.id, b.readingStatus as BookReadingStatus]),
  );

  const ranked = rankCatalogBooks(booksWithStatus, preservedReadings, {
    interactions: [
      {
        bookId: bookA.id,
        slug: bookA.slug,
        title: bookA.title,
        tags: bookA.tags,
        author_id: bookA.author_id,
        timestamp: Date.now(),
        type: 'click',
      },
    ],
  });

  // bookA, bookB, bookC DEVEM permanecer no final com seus status preservados
  const bottom = ranked.slice(2);
  expect(bottom.find((b) => b.id === bookA.id)?.readingStatus).toBe('read');
  expect(bottom.find((b) => b.id === bookB.id)?.readingStatus).toBe('reading');
  expect(bottom.find((b) => b.id === bookC.id)?.readingStatus).toBe('dnf');

  // Candidatos não lidos no topo
  expect(ranked[0].id).toBe(bookE.id);
  expect(ranked[0].readingStatus).toBeUndefined();
});
```

- [ ] **Step 2: Executar teste para verificar comportamento**

Run: `npm test tests/unit/recommendations.test.ts`
Expected: Verificar resultado e prosseguir para a integração no componente.

- [ ] **Step 3: Corrigir `PersonalizedCatalog` para extrair e preservar `readingStatus`**

No arquivo `app/src/components/catalog/personalized-catalog.tsx`:
1. Extrair o mapa de leituras a partir de `initialBooks` (`book.readingStatus`).
2. Passar esse mapa como segundo argumento para `rankCatalogBooks(initialBooks, serverReadingsMap, { interactions })`.
3. Substituir a importação estática de `@/features/recommendations/tensorflow-recommender` por importação dinâmica assíncrona sob demanda (`await import(...)`), evitando carregar o bundle de tensores para visitantes sem interações.

```typescript
// Extrair mapa de leituras do servidor
const serverReadings = new Map<string, BookReadingStatus>();
for (const b of initialBooks) {
  if (b.readingStatus) {
    serverReadings.set(b.id, b.readingStatus as BookReadingStatus);
  }
}

// Import dinâmico sob demanda
const { rankCatalogBooks } = await import('@/features/recommendations/tensorflow-recommender');
const ranked = rankCatalogBooks(initialBooks, serverReadings, { interactions });
```

- [ ] **Step 4: Executar testes unitários**

Run: `npm test tests/unit/recommendations.test.ts`
Expected: PASS.

- [ ] **Step 5: Commit**

```bash
git add app/src/components/catalog/personalized-catalog.tsx app/tests/unit/recommendations.test.ts
git commit -m "fix(recommendations): preserve server reading statuses and lazy-load client re-ranking"
```

---

### Task 1.2: Eliminar o Teto de 500 Livros e Desacoplar Projeção de Ranking

**Files:**
- Modify: `app/src/features/catalog/queries.ts:54-165`
- Create: `app/tests/unit/catalog-pagination.test.ts`

**Interfaces:**
- Consumes: Supabase `books` e `user_book_readings`.
- Produces: `getBooksResult` sem corte prematuro, paginando qualquer profundidade do catálogo.

- [ ] **Step 1: Escrever teste de paginação profunda (> 500 livros)**

Criar `app/tests/unit/catalog-pagination.test.ts` simulando catálogo com 600 livros para usuário autenticado com leituras:

```typescript
import { describe, it, expect, vi } from 'vitest';
import { getBooksResult } from '@/features/catalog/queries';

describe('Catalog Deep Pagination (> 500 books)', () => {
  it('allows navigating to page 25 (offset 576) without returning empty array when total > 500', async () => {
    // Simula catálogo extenso
    // Garante que a fatia correspondente à página solicitada seja retornada
  });
});
```

- [ ] **Step 2: Executar teste para verificar falha atual com limite de 500**

Run: `npm test tests/unit/catalog-pagination.test.ts`
Expected: FAIL com livros vazios após offset 500.

- [ ] **Step 3: Refatorar `getBooksResult` em `queries.ts`**

1. Remover `query = query.limit(500)`.
2. Para usuários com leituras, a consulta de classificação deve buscar apenas os campos essenciais para scoring:
   `select('id,author_id,tags')`.
3. Executar o ranking com os metadados mínimos dos candidatos.
4. Identificar os 24 IDs da página atual: `const pageIds = rankedIds.slice(offset, offset + limit)`.
5. Fazer a busca completa (com synopsis, cover_url, dimensões, etc.) APENAS para os 24 `pageIds`.
6. Hidratar os autores apenas para os 24 livros exibidos.
7. Retornar `total = count ?? allCandidatesCount`, garantindo que `totalPages = Math.ceil(total / limit)` seja 100% navegável.

- [ ] **Step 4: Executar testes de paginação**

Run: `npm test tests/unit/catalog-pagination.test.ts`
Expected: PASS.

- [ ] **Step 5: Commit**

```bash
git add app/src/features/catalog/queries.ts app/tests/unit/catalog-pagination.test.ts
git commit -m "fix(catalog): eliminate 500 books hard limit and optimize feature projection"
```

---

### Task 1.3: Criar Rota de API do Catálogo com Cache de Ranking por Sessão

**Files:**
- Create: `app/src/features/recommendations/recommender-cache.ts`
- Create: `app/src/app/api/books/catalog/route.ts`
- Modify: `app/src/components/catalog/personalized-catalog.tsx`

**Interfaces:**
- Consumes: `getBooksResult` de `@/features/catalog/queries`.
- Produces: `GET /api/books/catalog?page=N&limit=24&q=...` com resposta JSON `CatalogPageResult`.

- [ ] **Step 1: Implementar cache de IDs ordenados em `recommender-cache.ts`**

Criar cache em memória leve com TTL (ex: 5 minutos) indexado por `${userId}:${searchQuery}`:
```typescript
interface CachedRanking {
  rankedBookIds: string[];
  total: number;
  timestamp: number;
}
```
Expor métodos:
- `getCachedRanking(key: string): string[] | null`
- `setCachedRanking(key: string, ids: string[], total: number): void`
- `invalidateUserRanking(userId: string): void`

- [ ] **Step 2: Escrever teste unitário para `recommender-cache.ts`**

Verificar armazenamento, expiração por TTL e invalidação pontual.

- [ ] **Step 3: Criar Route Handler `app/src/app/api/books/catalog/route.ts`**

```typescript
import { NextRequest, NextResponse } from 'next/server';
import { getBooksResult } from '@/features/catalog/queries';

export async function GET(request: NextRequest) {
  const { searchParams } = new URL(request.url);
  const page = Math.max(1, parseInt(searchParams.get('page') || '1', 10));
  const limit = Math.min(48, Math.max(1, parseInt(searchParams.get('limit') || '24', 10)));
  const q = searchParams.get('q') || '';

  const result = await getBooksResult(q, page, limit);
  if (!result.ok) {
    return NextResponse.json({ error: result.error.message }, { status: 500 });
  }

  return NextResponse.json(result.data, {
    headers: {
      'Cache-Control': 'private, max-age=60, stale-while-revalidate=120',
    },
  });
}
```

- [ ] **Step 4: Executar testes de integração**

Run: `npm test tests/unit/catalog-pagination.test.ts`
Expected: PASS.

- [ ] **Step 5: Commit**

```bash
git add app/src/features/recommendations/recommender-cache.ts app/src/app/api/books/catalog/route.ts
git commit -m "feat(catalog): add catalog pagination api route with ranking session cache"
```

---

## Fase 2 — Reduzir o Custo de Execução e Otimização do Motor

### Task 2.1: Vocabulário Indexado com Lookups $O(1)$ e Normalização em Cache

**Files:**
- Modify: `app/src/features/recommendations/types.ts:9-13`
- Modify: `app/src/features/recommendations/tensorflow-recommender.ts:20-68`
- Test: `app/tests/unit/recommendations.test.ts`

**Interfaces:**
- Consumes: Lista de livros `Book[]`.
- Produces: `IndexedVocabulary` com `tagIndexMap: Map<string, number>` e `authorIndexMap: Map<string, number>`.

- [ ] **Step 1: Atualizar definições de tipos em `types.ts`**

```typescript
export interface IndexedVocabulary {
  tags: string[];
  authorIds: string[];
  tagIndexMap: Map<string, number>;
  authorIndexMap: Map<string, number>;
  tagIdfMap: Map<string, number>;
}
```

- [ ] **Step 2: Escrever teste unitário para vocabulário indexado $O(1)$**

Em `app/tests/unit/recommendations.test.ts`, adicionar verificação das propriedades `tagIndexMap` e `authorIndexMap` e medição de ausência de chamadas `indexOf`.

- [ ] **Step 3: Refatorar `buildFeatureVocabulary` e `vectorizeBook`**

1. Em `buildFeatureVocabulary`: construir os mapas `Map<string, number>` no momento da criação do vocabulário.
2. Em `vectorizeBook`: substituir `vocab.authorIds.indexOf(id)` por `vocab.authorIndexMap.get(id)`.
3. Substituir `vocab.tags.indexOf(tag)` por `vocab.tagIndexMap.get(tag)`.

- [ ] **Step 4: Executar suíte de testes**

Run: `npm test tests/unit/recommendations.test.ts`
Expected: PASS com redução imediata do tempo de execução.

- [ ] **Step 5: Commit**

```bash
git add app/src/features/recommendations/types.ts app/src/features/recommendations/tensorflow-recommender.ts app/tests/unit/recommendations.test.ts
git commit -m "perf(recommendations): use indexed vocabulary maps for O(1) feature vectorization"
```

---

### Task 2.2: Implementar Motor Nativo em JavaScript Puro (Zero Dependências)

**Files:**
- Create: `app/src/features/recommendations/native-recommender.ts`
- Create: `app/src/features/recommendations/recommender-engine.ts`
- Test: `app/tests/unit/recommender-benchmark.test.ts`

**Interfaces:**
- Consumes: `Book[]`, `userReadings`, `RecommendationOptions`.
- Produces: `computeNativeRecommendations()` e `rankCatalogBooksNative()`.

- [ ] **Step 1: Escrever teste de equivalência estrita entre Native JS e TensorFlow.js**

Criar `app/tests/unit/recommender-benchmark.test.ts` com testes parametrizados validando que:
1. Os scores gerados por `computeNativeRecommendations` coincidem com `computeTensorFlowRecommendations` com precisão $\le 10^{-5}$.
2. A ordenação final dos livros é 100% idêntica.
3. Não há consumo de memória de tensores no motor nativo.

- [ ] **Step 2: Executar teste para verificar ausência do arquivo nativo**

Run: `npm test tests/unit/recommender-benchmark.test.ts`
Expected: FAIL com módulo não encontrado.

- [ ] **Step 3: Implementar `native-recommender.ts`**

Implementar cálculo de similaridade cosseno em float puro:
```typescript
export function computeCosineSimilarity(vecA: number[], vecB: number[], epsilon = 1e-7): number {
  let dot = 0;
  let normA = 0;
  let normB = 0;
  for (let i = 0; i < vecA.length; i++) {
    const a = vecA[i];
    const b = vecB[i];
    dot += a * b;
    normA += a * a;
    normB += b * b;
  }
  const denom = Math.sqrt(normA) * Math.sqrt(normB);
  return denom > epsilon ? dot / denom : 0;
}
```
Implementar `computeNativeRecommendations(candidates, historyBooks, options)` e `rankCatalogBooksNative()`.

- [ ] **Step 4: Implementar `recommender-engine.ts`**

Criar seletor configurável por variável de ambiente ou feature flag (`RECOMMENDER_BACKEND=native | tensorflow`):
```typescript
export function rankCatalog(
  books: Book[],
  userReadings?: Map<string, BookReadingStatus> | null,
  options?: RecommendationOptions,
  backend: 'native' | 'tensorflow' = 'native',
): ScoredBook[]
```

- [ ] **Step 5: Executar testes de equivalência**

Run: `npm test tests/unit/recommender-benchmark.test.ts`
Expected: PASS (diferença numérica zero, ordenação idêntica).

- [ ] **Step 6: Commit**

```bash
git add app/src/features/recommendations/native-recommender.ts app/src/features/recommendations/recommender-engine.ts app/tests/unit/recommender-benchmark.test.ts
git commit -m "feat(recommendations): implement pure JS recommender engine with numerical equivalence"
```

---

## Fase 3 — Melhorar a Relevância e Qualidade das Recomendações

### Task 3.1: Enriquecer Sinais do Usuário com Pesos e Decaimento Temporal

**Files:**
- Modify: `app/src/features/recommendations/types.ts:40-60`
- Modify: `app/src/features/recommendations/browser-storage.ts:1-105`
- Test: `app/tests/unit/recommendations.test.ts`

**Interfaces:**
- Consumes: Interações em `localStorage` com `type` (`click` | `view`), contagem e `timestamp`.
- Produces: Sinais de interesse ponderados com decaimento exponencial por meia-vida ($t_{1/2} = 14$ dias).

- [ ] **Step 1: Escrever teste unitário para decaimento temporal e ponderação diferenciada**

Em `app/tests/unit/recommendations.test.ts`:
1. Validar que um clique recente (hoje) possui maior peso do que um clique de 30 dias atrás.
2. Validar que `click` possui peso maior que `view` ($0.6$ vs $0.2$).
3. Validar que abrir a página de detalhes (`view`) não apaga o clique registrado anteriormente, somando seus efeitos.

- [ ] **Step 2: Executar teste para verificar falha atual**

Run: `npm test tests/unit/recommendations.test.ts`
Expected: FAIL (atualmente `view` sobrescreve `click` e ambos têm peso idêntico 0.8 sem decaimento).

- [ ] **Step 3: Implementar agregação e decaimento em `browser-storage.ts`**

1. Estrutura atualizada de armazenamento:
```typescript
export interface StoredInteractionRecord {
  bookId: string;
  slug: string;
  title: string;
  tags: string[];
  author_id: string | null;
  clicksCount: number;
  viewsCount: number;
  lastInteractedAt: number;
}
```
2. Função de cálculo de peso com decaimento:
$$ w(t) = \left( \text{clicks} \cdot w_{\text{click}} + \text{views} \cdot w_{\text{view}} \right) \cdot \exp\left( - \frac{\ln(2) \cdot \Delta t}{t_{1/2}} \right) $$
com $t_{1/2} = 14 \text{ dias}$.

- [ ] **Step 4: Executar testes de interações**

Run: `npm test tests/unit/recommendations.test.ts`
Expected: PASS.

- [ ] **Step 5: Commit**

```bash
git add app/src/features/recommendations/types.ts app/src/features/recommendations/browser-storage.ts app/tests/unit/recommendations.test.ts
git commit -m "feat(recommendations): add interaction multi-signals and exponential time decay"
```

---

### Task 3.2: Ponderação TF-IDF para Tags e Normalização de Livros

**Files:**
- Modify: `app/src/features/recommendations/native-recommender.ts`
- Modify: `app/src/features/recommendations/tensorflow-recommender.ts`
- Test: `app/tests/unit/recommendations.test.ts`

**Interfaces:**
- Consumes: Tags do catálogo completo.
- Produces: Vetores normalizados por livro com pesos ponderados pelo IDF das tags.

- [ ] **Step 1: Escrever teste unitário para ponderação discriminativa (TF-IDF)**

Adicionar teste em `app/tests/unit/recommendations.test.ts`:
- Livro com tag rara ("Distopia Cyberpunk") deve ter maior afinidade discriminativa do que livro com tag ubíqua presente em 95% do catálogo ("Ficção").
- Livro com 20 tags não deve sufocar perfil de usuário frente a livro com 2 tags relevantes (graças à normalização prévia por livro).

- [ ] **Step 2: Executar teste para verificar comportamento**

Run: `npm test tests/unit/recommendations.test.ts`

- [ ] **Step 3: Implementar cálculo de IDF e normalização por livro**

1. Calcular IDF para cada tag:
$$ \text{IDF}(t) = \ln\left( \frac{1 + |B|}{1 + \text{df}(t)} \right) + 1 $$
2. No vetor do livro: atribuir $\text{weight}(t) = \text{IDF}(t)$.
3. Normalizar o vetor de cada livro ($\|\mathbf{v}_b\|_2 = 1$) antes de incorporá-lo à soma do perfil do usuário.
4. Normalizar o perfil final do usuário ($\|\mathbf{v}_u\|_2 = 1$).

- [ ] **Step 4: Executar testes unitários**

Run: `npm test tests/unit/recommendations.test.ts`
Expected: PASS.

- [ ] **Step 5: Commit**

```bash
git add app/src/features/recommendations/native-recommender.ts app/src/features/recommendations/tensorflow-recommender.ts app/tests/unit/recommendations.test.ts
git commit -m "feat(recommendations): add TF-IDF tag weighting and per-book vector normalization"
```

---

### Task 3.3: Diversificação com MMR e Recomendações Explicáveis

**Files:**
- Modify: `app/src/features/recommendations/types.ts`
- Modify: `app/src/features/recommendations/native-recommender.ts`
- Modify: `app/src/components/book-card.tsx:38-60`
- Test: `app/tests/unit/recommendations.test.ts`

**Interfaces:**
- Consumes: Candidatos pontuados e vetores de features.
- Produces: `ScoredBook` com `recommendationReason?: string` e ordenação diversificada por MMR.

- [ ] **Step 1: Escrever testes unitários para diversificação MMR e explicabilidade**

Em `app/tests/unit/recommendations.test.ts`:
1. Teste de diversificação MMR: verificar que um catálogo com 10 livros do mesmo autor não monopoliza os 5 primeiros slots quando há outros livros com boa afinidade em outros gêneros.
2. Teste de explicabilidade: verificar se `book.recommendationReason` retorna justificativas verídicas ("Do mesmo autor de Verity" ou "Porque você leu Suspense Psicológico").

- [ ] **Step 2: Implementar algoritmo MMR (Maximal Marginal Relevance)**

Implementar seleção gulosa MMR:
$$ b^* = \arg\max_{b \in C \setminus S} \left[ \lambda \cdot \text{sim}(b, u) - (1 - \lambda) \cdot \max_{s \in S} \text{sim}(b, s) \right] $$
com $\lambda = 0.75$ (ajustável via `RecommendationOptions.diversityLambda`).

- [ ] **Step 3: Implementar gerador de motivo explicável (`recommendationReason`)**

Mapear a feature com maior produto escalar entre o perfil do usuário e o candidato:
- Se for autor comum: `"Do mesmo autor de suas leituras"`
- Se for tag rara compartilhada: `"Recomendado por afinidade com ${tag}"`
- Se for baseado em interações locais: `"Com base no seu interesse recente em ${tag}"`

- [ ] **Step 4: Atualizar `BookCard` para exibir o motivo da recomendação**

No componente `app/src/components/book-card.tsx`, renderizar o texto de `book.recommendationReason` de forma acessível abaixo do título ou no badge.

- [ ] **Step 5: Executar testes de apresentação e ranking**

Run: `npm test tests/unit/recommendations.test.ts`
Expected: PASS.

- [ ] **Step 6: Commit**

```bash
git add app/src/features/recommendations/types.ts app/src/features/recommendations/native-recommender.ts app/src/components/book-card.tsx app/tests/unit/recommendations.test.ts
git commit -m "feat(recommendations): implement MMR diversity reranking and explainable recommendation badges"
```

---

## Fase 4 — Medição, Benchmark e Critérios de Rollout em Produção

### Task 4.1: Benchmark de Performance (TensorFlow.js vs Native JS)

**Files:**
- Create: `app/scripts/benchmark-recommenders.ts`
- Modify: `app/package.json`

**Interfaces:**
- Consumes: Catálogos sintéticos de 100, 500, 1.000 e 2.500 livros.
- Produces: Relatório comparativo com Latência (p50, p95, p99), Memória Heap e Tamanho de Bundle.

- [ ] **Step 1: Criar script de benchmark automatizado**

Criar `app/scripts/benchmark-recommenders.ts` executando 100 iterações em cada tamanho de catálogo para ambos os motores, medindo:
- Tempo de vetorização e construção de vocabulário.
- Tempo de cálculo de similaridade e ordenação.
- Alocação de tensores (`tf.memory().numTensors`) vs garbage collection JS.
- Diferença absoluta máxima e média de scores.

- [ ] **Step 2: Adicionar comando no `package.json`**

Adicionar `"benchmark:recommender": "vitest run tests/unit/recommender-benchmark.test.ts"` ou `tsx scripts/benchmark-recommenders.ts`.

- [ ] **Step 3: Executar benchmark e registrar métricas**

Run: `npm run benchmark:recommender` (ou via vitest).
Documentar resultados:
- Latência Native JS vs TensorFlow.js.
- Bundle cliente: economia de ~1.2MB ao adotar Native JS no navegador.

- [ ] **Step 4: Commit**

```bash
git add app/scripts/benchmark-recommenders.ts app/package.json
git commit -m "chore(benchmark): add comprehensive performance benchmark harness for recommendation engines"
```

---

### Task 4.2: QA Integrado, Verificação de Tipos e Build de Produção

**Files:**
- Test: Suíte completa (`npm test`)
- Run: `npm run typecheck`
- Run: `npm run lint`
- Run: `npm run build`

- [ ] **Step 1: Executar suíte completa de testes unitários**

Run: `npm test`
Expected: 100% de aprovação em todos os arquivos de teste.

- [ ] **Step 2: Executar verificação estrita de TypeScript**

Run: `npm run typecheck`
Expected: 0 erros.

- [ ] **Step 3: Executar verificação de ESLint**

Run: `npm run lint`
Expected: 0 warnings ou erros.

- [ ] **Step 4: Executar build de produção do Next.js**

Run: `npm run build`
Expected: Build estático e dinâmico concluído com sucesso.

- [ ] **Step 5: Commit final da branch**

```bash
git commit -m "chore(qa): complete comprehensive validation of recommendation engine remediation"
```

---

## Critérios de Aceite e Telemetria de Rollout

1. **Consistência Visual e de Estado:** Livros com status `read`, `reading` e `dnf` nunca são recomendados no topo nem perdem suas identificações visuais no cliente.
2. **Navegabilidade Total:** Usuários logados e anônimos conseguem navegar por todas as páginas do catálogo (mesmo com mais de 500 livros) sem interrupção antecipada.
3. **Desempenho no Cliente:** Para visitantes anônimos sem interações, o carregamento da página de catálogo não baixa nenhum byte do TensorFlow.js.
4. **Precisão e Diversidade:** A lista de recomendações combina afinidade com diversidade via MMR, apresentando justificativas compreensíveis para o leitor.
5. **Decisão Baseada em Dados:** Se o benchmark demonstrar que o motor Native JS entrega resultados identicamente calibrados com menor latência e 0KB de dependências pesadas, o DIYSPUR poderá adotar o Native JS como padrão tanto no servidor quanto no cliente.
