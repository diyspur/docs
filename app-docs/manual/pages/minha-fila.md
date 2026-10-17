# Fila de próximos livros / `/minha-fila`

## 1. Overview & Purpose

- **URL Path:** `/minha-fila`
- **Access Level:** Authenticated
- **Core Purpose:** Priorizar até cinco títulos para ler a seguir.

## 2. Prerequisites & Access

Sessão ativa. A adição ocorre na ficha do livro.

## 3. UI Components & Layout

Contagem; títulos priorizados; acesso ao catálogo; remoção; estado vazio.

![Fila de próximos livros — captura da auditoria de 12/10/2026](../images/minha-fila.jpg)

*Captura desktop sem sessão. A imagem mostra o redirecionamento para login; a interface autenticada foi observada antes do logout, mas sua captura não ficou disponível.*

## 4. Step-by-Step User Flow

1. **Step 1:** Abra uma ficha no catálogo e use o botão de adicionar à fila.
2. **Step 2:** Aguarde confirmação e abra **Minha fila** para consultar os títulos.
3. **Step 3:** Clique no livro para iniciar o caminho para uma temporada.
4. **Step 4:** Quando a fila atingir cinco títulos, remova um antes de adicionar outro.

### Interação com o backend

add_to_up_next_atomic insere de forma atômica sob as regras do backend; user_up_next é consultada para o titular. Remoção aplica user_id e book_id.

Os passos de salvamento acima descrevem o contrato implementado. Confirme a mensagem de sucesso antes de considerar uma operação persistida; sua execução nesta rodada está indicada abaixo.

## 5. Error Handling & Edge Cases

Fila vazia oferece Descobrir livros. queue_full retorna Sua fila já tem cinco livros. A mensagem de erro não garante que um título entrou: revise após retorno ao módulo. Não há controle de arrastar/reordenar comprovado nesta sessão.

### Evidência e limite desta rodada

Estado vazio com contagem zero e links de catálogo; proteção anônima por redirect. Adição/remoção não executadas.

Falhas de rede forçadas, comparação entre duas contas e matriz mobile/tablet não executadas nesta sessão. Layout responsivo foi revisado no CSS, sem certificação visual desses tamanhos. Consulte [cobertura QA](../03-qa-coverage.md).

### Referência de implementação

- [src/app/minha-fila/page.tsx](../../../src/app/minha-fila/page.tsx)
- [src/features/up-next/actions.ts](../../../src/features/up-next/actions.ts)
- [src/features/up-next/queries.ts](../../../src/features/up-next/queries.ts)

[Índice do manual](../00-overview.md) · [Primeiros passos](../01-getting-started.md) · [Referência técnica](../02-technical-reference.md)
