# Iniciar Clube do Livro / `/clubes/novo`

## 1. Overview & Purpose

- **URL Path:** `/clubes/novo` (suporta query string contextual `?livro=[slug]`)
- **Access Level:** Authenticated
- **Core Purpose:** Atuar como **resolvedor contextual de clube de livro** (Excalidraw B25: *"O botão 'Iniciar clube' abre o clube do livro"*). Redireciona o leitor ao clube oficial da obra especificada ou permite localizar títulos do catálogo para iniciar discussões canônicas, eliminando clubes duplicados ou órfãos.

## 2. Prerequisites & Access

- Sessão ativa com autenticação.
- Parâmetro `?livro=[slug]` para resolução automática do clube oficial correspondente.

## 3. UI Components & Layout

- **Modo Contextual (`?livro=[slug]`):**
  - Card de identificação do livro com título, sinopse curta e autor.
  - Alerta informativo sobre o vínculo do clube canônico oficial.
  - Botão de ação primária: **"Acessar Clube Oficial deste Livro"** (redireciona para `/clubes/[id]` com canais `#sem-spoilers` e `#com-spoilers`).
- **Modo Descoberta (sem parâmetro de livro):**
  - Painel explicativo da arquitetura *1 Livro = 1 Clube Oficial*.
  - Lista de livros recentes do catálogo com botões diretos para iniciar ou abrir a discussão de cada obra.
  - Link de retorno para a biblioteca de livros (`/livros`).

## 4. Step-by-Step User Flow

1. **Step 1:** O leitor clica em "Iniciar clube" na ficha de um livro ou acessa `/clubes/novo?livro=o-massacre-da-familia-hope`.
2. **Step 2:** A rota `/clubes/novo` carrega os dados editoriais do livro especificado no banco de dados.
3. **Step 3:** Se o clube oficial já existir, o leitor é guiado para entrar diretamente nas discussões da comunidade.
4. **Step 4:** Caso acesse sem parâmetro, o leitor escolhe um livro do catálogo e é redirecionado ao clube correspondente.

### Interação com o backend

- A página consulta a tabela `books` e verifica a existência do vínculo em `official_book_groups`.
- A criação de clubes independentes sem livro foi descontinuada (`createClub` deprecated) e a trigger de segurança `require_canonical_new_group` impede a inserção de clubes sem vínculo canônico.

## 5. Error Handling & Edge Cases

- Parâmetro `?livro=slug` inexistente: exibe mensagem amigável informando que o título não foi localizado no catálogo e apresenta opções de livros válidos.
- Usuário não autenticado: o middleware redireciona para `/entrar?returnUrl=/clubes/novo...`.

### Referência de implementação

- [src/app/clubes/novo/page.tsx](../../../src/app/clubes/novo/page.tsx)
- [src/features/clubs/actions.ts](../../../src/features/clubs/actions.ts)
- [src/lib/group-entry-context.ts](../../../src/lib/group-entry-context.ts)

[Índice do manual](../00-overview.md) · [Primeiros passos](../01-getting-started.md) · [Referência técnica](../02-technical-reference.md)
