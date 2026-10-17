# Início / `/`

## 1. Overview & Purpose

- **URL Path:** `/`
- **Access Level:** Public
- **Core Purpose:** Apresentar o clube, a leitura coletiva em destaque e os caminhos para começar a participar.

## 2. Prerequisites & Access

Nenhuma conta é necessária para explorar. Para guardar progresso ou interagir, entre na sua conta.

## 3. UI Components & Layout

Cabeçalho com Explorar e Comunidade; apresentação; Como funciona; temporada em destaque; demonstração de segunda tela; agenda; votação; depoimentos; catálogo resumido; convite para participar; rodapé e preferências de analytics.

![Início — captura da auditoria de 12/10/2026](../images/home-anon.jpg)

*Captura real do estado disponível nesta rodada. Em telas administrativas, a imagem mostra o bloqueio de uma conta comum; não representa o editor autorizado.*

## 4. Step-by-Step User Flow

1. **Step 1:** Abra a página inicial e leia a apresentação do clube. O site acompanha sua leitura de um exemplar físico ou digital; não é uma biblioteca de downloads dos livros.
2. **Step 2:** Clique em **Começar a ler junto** para chegar à seção da temporada em destaque. A ação movimenta a página até a âncora, sem criar inscrição.
3. **Step 3:** Clique em **Acompanhar temporada** para ver o cronograma ou em **Ver ficha do livro** para consultar a edição e os avisos de conteúdo.
4. **Step 4:** Use **Ver todos →** na agenda para abrir o calendário. Consulte a data e o fuso antes de decidir participar.
5. **Step 5:** Se ainda não tem conta, use **Fazer parte**. Com sessão ativa, **Meu espaço** reúne seus registros e conquistas.

### Interação com o backend

Consultas SSR em seasons, books, chapters, meetings e enquetes por src/features/catalog/queries.ts. A quantidade de capítulos devolvida depende da visibilidade da sessão e das policies; o contador não deve ser interpretado como o número de capítulos do exemplar.

Os passos de salvamento acima descrevem o contrato implementado. Confirme a mensagem de sucesso antes de considerar uma operação persistida; sua execução nesta rodada está indicada abaixo.

## 5. Error Handling & Edge Cases

Sem temporada ou agenda, a experiência depende de conteúdo editorial disponível. Demonstração de segunda tela é ilustrativa, não um player real nem uma lista pessoal. Falhas de algumas consultas são convertidas em ausência de conteúdo; ver relatório QA.

### Evidência e limite desta rodada

Home renderizada com sessão e sem sessão. Desktop sem rolagem horizontal: clientWidth=scrollWidth=1348; um único main. A home mostrou 3/3 com sessão e 1/1 após logout, embora a timeline anônima mostrasse três capítulos.

Falhas de rede forçadas, comparação entre duas contas e matriz mobile/tablet não executadas nesta sessão. Layout responsivo foi revisado no CSS, sem certificação visual desses tamanhos. Consulte [cobertura QA](../03-qa-coverage.md).

### Referência de implementação

- [src/app/page.tsx](../../../src/app/page.tsx)
- [src/features/catalog/queries.ts](../../../src/features/catalog/queries.ts)

[Índice do manual](../00-overview.md) · [Primeiros passos](../01-getting-started.md) · [Referência técnica](../02-technical-reference.md)
