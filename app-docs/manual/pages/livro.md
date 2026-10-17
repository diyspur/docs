# Ficha do Livro / `/livros/[slug]`

## 1. Overview & Purpose

- **URL Path:** `/livros/[slug]`
- **Access Level:** Public
- **Core Purpose:** Apresentar a obra literária, dados da edição canônica (ISBN, total de páginas, editora), avisos de conteúdo, temporadas ativas e fornecer ponto único de entrada para o clube oficial de discussão do livro (`DiscussionEntryButton`).

## 2. Prerequisites & Access

- Livro visível e publicado no catálogo.
- Sessão ativa para entrar na discussão do clube com identidade ou enviar convites de leitura.

## 3. UI Components & Layout

- **Breadcrumbs & Identificação:** Navegação de volta ao catálogo e título principal.
- **Capa & Metadados Editoriais:** Imagem da capa, autor, biografia, editora, idioma e ISBN.
- **Avisos de Conteúdo (*Content Warnings*):** Badges informativos sobre temas sensíveis.
- **Sinopse Oficial & Tags:** Resumo da obra e classificação por temas.
- **Entrada Unificada de Discussão (`DiscussionEntryButton`):**
  - Botão de destaque **"💬 Entrar na discussão do livro"**.
  - Resolve o clube oficial da obra e encaminha o leitor para `/clubes/[id]` com canais `#sem-spoilers` e `#com-spoilers`.
- **Ações de Engajamento Literário:**
  - Link de convite: **"Convidar amigos para ler juntos"** (redireciona para `/leituras-compartilhadas?livro=[slug]`).
  - Link de desafios: **"Ver desafios literários relacionados"** (`/desafios`).
  - Redirecionamento de afiliado oficial Amazon para compra do exemplar.
- **Temporadas e Capítulos:** Lista de ciclos de leitura coletiva e semanas ativas com acesso direto aos capítulos.

## 4. Step-by-Step User Flow

1. **Step 1:** Acesse a ficha do livro a partir do catálogo (`/livros`) ou da página inicial.
2. **Step 2:** Confira os dados da edição e os avisos de conteúdo antes de iniciar a leitura.
3. **Step 3:** Clique em **💬 Entrar na discussão do livro** para participar do clube oficial da obra na comunidade.
4. **Step 4:** Para ler em conjunto com amigos, utilize o botão de leitura compartilhada.
5. **Step 5:** Selecione a temporada em destaque para acompanhar o cronograma semanal de leitura e assistir às contextualizações em vídeo.

### Interação com o backend

- `getBook` lê `books`, `authors`, `seasons` e `book_content_warnings`.
- `openOfficialBookGroupAction` resolve o registro em `official_book_groups` e retorna a rota do clube oficial.
- `/api/affiliate/[slug]` registra clique anonimizado e redireciona com tag de associado oficial.

## 5. Error Handling & Edge Cases

- Slug inexistente: aciona `notFound()` do Next.js exibindo tela 404 padrão.
- Usuário sem sessão ao clicar na discussão: preserva o redirecionamento para o clube após o login seguro.
- Obra sem temporadas: exibe aviso de que a curadoria ainda não agendou um ciclo coletivo para o título.

### Referência de implementação

- [src/app/livros/[slug]/page.tsx](../../../src/app/livros/[slug]/page.tsx)
- [src/components/discussions/discussion-entry-button.tsx](../../../src/components/discussions/discussion-entry-button.tsx)
- [src/features/discussions/actions.ts](../../../src/features/discussions/actions.ts)
- [src/features/catalog/queries.ts](../../../src/features/catalog/queries.ts)

[Índice do manual](../00-overview.md) · [Primeiros passos](../01-getting-started.md) · [Referência técnica](../02-technical-reference.md)
