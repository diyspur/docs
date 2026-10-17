# Rankings Comunitários: Temporada e Livros / `/ranking` e `/ranking/livros`

## 1. Overview & Purpose

- **URL Paths:** `/ranking` (XP da Temporada) e `/ranking/livros` (Livros da Comunidade)
- **Access Level:** Public (com visão de visitante) / Authenticated (com destaque do leitor)
- **Core Purpose:** Consultar pontuações de participação por XP na temporada ativa e explorar os rankings agregados de leitura de livros na comunidade (Mais Lidos, Lendo Agora, Concluídos e Desistências).

## 2. Prerequisites & Access

- Visualização aberta a todos os leitores e visitantes.
- Sessão ativa destaca a posição do usuário logado na lista de XP da temporada.

## 3. UI Components & Layout

- **Barra de Abas de Navegação:** Alternador rápido entre **🏆 Ranking da Temporada (XP)** (`/ranking`) e **📚 Ranking de Livros da Comunidade** (`/ranking/livros`).
- **Cards de Regras de Pontuação (B18):**
  - 📖 **Capítulo lido:** +20 XP
  - 💬 **Comentário em discussão:** +30 XP
  - 🧠 **Quiz completado:** +50 XP
  - 🔥 **Sequência diária mantida:** +25 XP
  - 📅 **Encontro literário confirmado:** +30 XP
- **Tabela de XP da Temporada:** Posição, avatar, nome de usuário, XP acumulado e destaque da conta atual.
- **Painel de Ranking de Livros (`/ranking/livros`):**
  - **Aba 1 (Mais Lidos):** Soma de leitores com status `reading` + `read`.
  - **Aba 2 (Lendo Agora):** Leitores com status `reading`.
  - **Aba 3 (Concluídos):** Leitores com status `read`.
  - **Aba 4 (Desistências):** Leitores com status `dnf` / `abandoned` e cálculo de `dnf_rate` (taxa de abandono percentual).
  - Capa do livro com link direto para a ficha editorial (`/livros/[slug]`), total de leitores e paginação de 20 itens.

## 4. Step-by-Step User Flow

1. **Step 1:** Acesse **Ranking** pelo menu principal ou pelo cabeçalho superior.
2. **Step 2:** Em `/ranking`, consulte seu posicionamento na temporada ativa e veja as 5 regras de pontuação para aumentar seu XP.
3. **Step 3:** Alterne para a aba **Ranking de Livros da Comunidade** (`/ranking/livros`) para explorar o comportamento coletivo de leitura.
4. **Step 4:** Navegue entre as abas *Mais lidos*, *Lendo agora*, *Concluídos* e *Desistências* para descobrir quais títulos engajam mais ou têm maior taxa de abandono.
5. **Step 5:** Clique na capa ou título de qualquer livro do ranking para abrir a ficha editorial e entrar no clube de discussão.

### Interação com o backend

- `/ranking` consulta `seasons` ativa e `v_season_ranking`, ordenada por position.
- `/ranking/livros` consulta a view `public.v_book_reading_stats` (alimentada pela view materializada `private.mv_book_reading_stats_v2`), suportando ordenação por `total_readers`, `currently_reading`, `finished_reading` e `dnf_readers` com `dnf_rate`.
- A função de banco `public.refresh_book_reading_stats()` é acionada periodicamente via cron ou sob demanda editorial para manter as métricas sincronizadas.

## 5. Error Handling & Edge Cases

- Sem temporada ativa, exibe card informativo convidando a aguardar o próximo ciclo.
- Lista vazia de leituras exibe empty state com convite para registrar livros na estante.
- Visitantes não autenticados recebem banner informativo convidando ao cadastro sem bloqueio da leitura dos rankings públicos.

### Referência de implementação

- [src/app/ranking/page.tsx](../../../src/app/ranking/page.tsx)
- [src/app/ranking/livros/page.tsx](../../../src/app/ranking/livros/page.tsx)
- [src/features/rankings/queries.ts](../../../src/features/rankings/queries.ts)

[Índice do manual](../00-overview.md) · [Primeiros passos](../01-getting-started.md) · [Referência técnica](../02-technical-reference.md)
