# Especificação Técnica de SEO e Indexabilidade — DIYSPUR

## 1. Visão Geral
Este documento define as regras de indexabilidade, geração de sitemap, controle de robôs (`robots.txt`), metadados e dados estruturados da plataforma DIYSPUR.

## 2. Mapa do Site (`sitemap.xml`)
Implementado em `src/app/sitemap.ts`:
- **Rotas fixas**: Exclusivamente páginas públicas: `/`, `/livros`, `/calendario`, `/comunidade`, `/privacidade`, `/termos`.
- **Exclusão de rotas privadas**: Rotas de autenticação, admin, perfil, metas, diário e especialmente `/ranking` são excluídas do sitemap.
- **Filtragem de Conteúdo Dinâmico**:
  - `books`: Inclui livros publicados no catálogo, com `lastModified` derivado de `updated_at`.
  - `seasons`: Somente temporadas com `status in ('active', 'finished')`.
  - `chapters`: Exige explicitamente `is_draft = false`, `published_at IS NOT NULL` e `published_at <= NOW()`. Capítulos futuros ou não publicados são estritamente barrados.
- **Datas reais**: Rotas fixas sem data de modificação verificável omitem `lastModified` em vez de gerar carimbo falso (`new Date()`).
- **Resiliência técnica**: Erros na consulta Supabase são capturados pelo Sentry com a tag `service: 'sitemap'`, retornando as rotas fixas como fallback seguro sem interromper a resposta para crawlers.

## 3. Diretivas para Rastreadores (`robots.txt`)
Implementado em `src/app/robots.ts`:
- **Permitido**: `/` para rastreamento de páginas públicas.
- **Bloqueado**:
  - `/admin/`
  - `/api/`
  - `/auth/`
  - `/perfil`
  - `/notificacoes`
  - `/ranking`
  - `/minha-fila`
  - `/metas`
  - `/diario`
  - `/historico`
  - `/atualizar-senha`
  - `/recuperar-senha`
- **Sitemap**: Aponta para `${siteUrl}/sitemap.xml`.

## 4. Dados Estruturados (`application/ld+json`)
- Implementado em `src/components/json-ld.tsx`.
- **Segurança**: Serialização com escape de caracteres `<` (`.replace(/</g, '\\u003c')`) para prevenir quebras do contexto `<script>`.
- **Schemas Globais e Específicos:**
  - `Organization`: Nome, URL, logotipo oficial e redes sociais (`sameAs`: Instagram, TikTok, YouTube).
  - `WebSite`: Nome, URL e SearchAction apontando para `/livros?q={search_term_string}`.
  - `Book`: Nome, autor (`Person`), sinopse (`description`), idioma (`pt-BR`), ISBN/capa quando presentes, e nó semântico `subjectOf` vinculando a obra ao seu clube oficial de leitura comunitária.
  - `LearningResource` (Capítulos e Guias): Tipado como `DiscussionGuide` para capítulos públicos com `educationalLevel: 'AdultEducation'`, `isPartOf` vinculado ao livro pai e `hasPart` para questões de reflexão e debate comunitário.

## 5. Metadados e Canonical
- `metadataBase` configurada no layout raiz com base em `NEXT_PUBLIC_SITE_URL`.
- Canonical absoluto gerado para evitar duplicação por parâmetros de query string ou rastreamento de campanhas.
- Open Graph e Twitter Cards configurados com imagens em proporção 1200x630.

## 6. Otimização para Motores de IA (GEO & AIO)
- **Descoberta por LLMs:** Otimização para robôs de busca generativa (GPTBot, PerplexityBot, ClaudeBot, Google-Extended).
- **Semântica HTML5 Rigorosa:** Conteúdo envolto em `<article>`, `<main>`, `<header>` e títulos hierárquicos `h1-h6` com dados contextuais explícitos.
- **Prevenção de FOUC:** Script inline no cabeçalho executado antes da hidratação (`theme-preload.js`) para evitar quebras visuais e garantir First Contentful Paint (FCP) de altíssimo escore.

