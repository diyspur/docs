# Diretório da Comunidade e Clubes / `/comunidade`

## 1. Overview & Purpose

- **URL Path:** `/comunidade`
- **Access Level:** Public / Authenticated
- **Core Purpose:** Diretório central com a lista de todos os clubes de leitura oficiais da plataforma (Excalidraw B23), permitindo descobrir debates organizados por livro e acessar canais com ou sem spoilers.

## 2. Prerequisites & Access

- Consulta pública de todos os clubes oficiais do catálogo.
- Sessão ativa para participar ativamente das mensagens e registrar consentimento anti-spoiler.

## 3. UI Components & Layout

- **Cabeçalho Editorial:** Título "Comunidade de Leitura", contextualização da arquitetura *1 livro = 1 clube oficial* e link para explorar o catálogo de livros.
- **Grade de Clubes de Livros:**
  - Card de cada clube com nome do livro, capa e descrição editorial.
  - Indicadores de atividade: contagem de membros e quantidade de canais temáticos.
  - **Atalhos Rápidos de Canais:**
    - Botão direto para `#sem-spoilers` (discussões seguras para quem está iniciando).
    - Botão direto para `#com-spoilers` (discussões profundas e teorias).
- **Banner de Criação/Busca:** Seção inferior convidando o leitor a encontrar seu livro favorito no catálogo para abrir o espaço oficial de debate.

## 4. Step-by-Step User Flow

1. **Step 1:** Acesse **Comunidade** pelo menu superior de navegação.
2. **Step 2:** Explore a lista de todos os clubes de leitura disponíveis.
3. **Step 3:** Escolha o livro que você está lendo ou deseja debater.
4. **Step 4:** Selecione o canal de entrada desejado (`#sem-spoilers` ou `#com-spoilers`). O sistema valida seu consentimento antes de carregar o fluxo de mensagens.
5. **Step 5:** Dentro do clube, utilize os atalhos de capítulos ou o chat em tempo real para interagir com a comunidade.

### Interação com o backend

- Consulta `user_clubs` vinculados à tabela `official_book_groups` e aos metadados de `books`.
- Consulta contagem de membros via `user_club_members` e canais cadastrados em `club_channels`.

## 5. Error Handling & Edge Cases

- Catálogo de clubes vazio: exibe estado com orientação para consultar os livros cadastrados.
- Tentativa de acesso a canal com spoiler: apresenta modal de consentimento explícito antes de exibir as mensagens.

### Referência de implementação

- [src/app/comunidade/page.tsx](../../../src/app/comunidade/page.tsx)
- [src/features/clubs/queries.ts](../../../src/features/clubs/queries.ts)
- [src/lib/discussion-contract.ts](../../../src/lib/discussion-contract.ts)

[Índice do manual](../00-overview.md) · [Primeiros passos](../01-getting-started.md) · [Referência técnica](../02-technical-reference.md)
