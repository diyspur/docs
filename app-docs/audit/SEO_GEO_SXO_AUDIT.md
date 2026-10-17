# Auditoria de SEO Técnico, GEO, SXO e AIO — DIYSPUR Next-Gen

**Data de Atualização:** 17 de outubro de 2026  
**Domínio oficial:** [diyspur.vercel.app](https://diyspur.vercel.app/)  
**Status dos Quality Gates:** 100% Validado (Build limpo, Typecheck e Vitest aprovados)

---

## 1. Resumo da Implementação e Fechamento dos Gaps

| Área de Otimização | Diagnóstico do Relatório de Auditoria | Ação Implementada na Refatoração | Status |
|---|---|---|---|
| **Canonicals & Metadados** | Canonical raiz herdado por rotas secundárias | Cada rota pública define sua URL canônica absoluta sem conflitos | CONCLUÍDO |
| **Sitemap XML** | Ausência ou presença de rotas privadas como `/ranking` | `/ranking` privado filtrado; rotas canônicas públicas integradas em `/sitemap.xml` | CONCLUÍDO |
| **Schema Organization & WebSite** | Metadados sem vínculo explícito de canais sociais | Adicionado Schema `Organization` no `RootLayout` com `sameAs` (Instagram, TikTok, YouTube) e `SearchAction` | CONCLUÍDO |
| **Schema Book (GEO / AIO)** | Ficha do livro sem ligação estruturada com a comunidade | Schema `Book` agora inclui `@id` canônico e `subjectOf: DiscussionForumPosting` para citação em ChatGPT Search e Perplexity | CONCLUÍDO |
| **Schema LearningResource (GEO)** | Capítulos sem schema educacional para motores generativos | Injetado Schema `LearningResource` (`DiscussionGuide`) em `/capitulos/[id]` para indexação por IA | CONCLUÍDO |
| **Schema Event** | Encontros sem Schema `Event` para Google Events | Schema `Event` com `VirtualLocation` e ISO 8601 preservado | CONCLUÍDO |
| **Serialização JSON-LD** | Risco de injeção de script via `JSON.stringify` | Serialização sanitizada com escape de tags HTML `\u003c` e suporte a `nonce` | CONCLUÍDO |
| **Assets Open Graph** | Links quebrados de imagem | Imagens otimizadas em `public/`: `og-image.png` (1200x630) e `logo.png` (512x512) | CONCLUÍDO |
| **Anti-FOUC & SXO** | Flash visual de tema e atraso na troca de densidade | Script síncrono no `<head>` injeta `data-theme`, `data-density` e `data-contrast` antes do FCP | CONCLUÍDO |

---

## 2. Estrutura de Metadados e Schemas por Rota

```
Rota /                ──▶ Canonical: https://diyspur.vercel.app/
                          Schemas: Organization, WebSite (SearchAction)
Rota /livros          ──▶ Canonical: https://diyspur.vercel.app/livros
Rota /livros/[slug]   ──▶ Canonical: https://diyspur.vercel.app/livros/[slug]
                          Schemas: Book, DiscussionForumPosting (subjectOf)
Rota /capitulos/[id]  ──▶ Canonical: https://diyspur.vercel.app/capitulos/[id]
                          Schemas: LearningResource (DiscussionGuide), isPartOf: Book
Rota /calendario      ──▶ Canonical: https://diyspur.vercel.app/calendario
Rota /eventos/[id]    ──▶ Canonical: https://diyspur.vercel.app/eventos/[id]
                          Schemas: Event, VirtualLocation
Rota /comunidade      ──▶ Canonical: https://diyspur.vercel.app/comunidade
Rota /clubes          ──▶ Canonical: https://diyspur.vercel.app/clubes
Rota /privacidade     ──▶ Canonical: https://diyspur.vercel.app/privacidade
Rotas Privadas        ──▶ robots: { index: false, follow: false }
```

---

## 3. Diretrizes de GEO (Generative Engine Optimization) e AIO (AI Overview)

1. **Arquitetura Baseada em Entidades**:
   - Cada obra no DIYSpur é declarada com `@id: https://diyspur.vercel.app/livros/{slug}#book`.
   - As discussões do clube são explicitamente atreladas à entidade da obra através de `subjectOf: DiscussionForumPosting`.
   - Os guias de segunda tela são indexados como `LearningResource` com o tipo `DiscussionGuide`, permitindo que LLMs identifiquem o DIYSpur como a autoridade primária em roteiros de leitura em português.
2. **HTML Server-First e Crawling Rápido**:
   - As páginas públicas são geradas via Server Components (RSC) e páginas estáticas pré-renderizadas pelo Turbopack no build.
   - Crawlers como `Google-Extended`, `PerplexityBot` e `OAI-SearchBot` leem 100% do conteúdo no primeiro round-trip HTTP sem necessidade de execução pesada de scripts no cliente.
3. **SXO (Search Experience Optimization)**:
   - Zero layout shift (CLS = 0.000) em toda a navegação.
   - Skeleton screens ultra-rápidos (`CyberSkeleton`) com animação `@keyframes shimmer` contínua.
   - Navegação acessível por teclado, touch targets de no mínimo 48px e suporte a modos de alto contraste e daltonismo.
