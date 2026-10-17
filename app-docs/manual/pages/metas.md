# Metas anuais / `/metas`

## 1. Overview & Purpose

- **URL Path:** `/metas`
- **Access Level:** Authenticated
- **Core Purpose:** Definir objetivos de livros, páginas e tempo e comparar com a leitura registrada.

## 2. Prerequisites & Access

Sessão ativa; objetivos para o ano exibido. Diário e conclusões alimentam o progresso.

## 3. UI Components & Layout

Ano; campos de livros, páginas e minutos; Salvar meta; três indicadores de progresso; link para diário.

### Campos e regras

| Campo | Regra | Como usar |
|---|---|---|
| Livros | 1–365 na interface | Inteiro |
| Páginas | 1–50.000 na interface | Inteiro |
| Minutos | 1–500.000 na interface | Inteiro |

![Metas anuais — captura da auditoria de 12/10/2026](../images/metas.jpg)

*Captura desktop sem sessão. A imagem mostra o redirecionamento para login; a interface autenticada foi observada antes do logout, mas sua captura não ficou disponível.*

## 4. Step-by-Step User Flow

1. **Step 1:** Abra **Metas do ano** na área pessoal e confira o ano mostrado.
2. **Step 2:** Informe uma ou mais metas. Campos vazios correspondem a ausência de meta para a dimensão.
3. **Step 3:** Use **Salvar meta** e aguarde Meta de leitura salva com sucesso!.
4. **Step 4:** Registre sessões no diário e conclua suas leituras; retorne para acompanhar progresso.
5. **Step 5:** Para ajustar a meta, altere o número e salve. A operação atualiza a meta do mesmo usuário e ano.

### Interação com o backend

setReadingGoal faz upsert em reading_goals com conflito user_id,year. getUserReadingGoal consulta reading_goal_progress por goal_id. Dados são calculados no backend; formulário não permite escrever progresso diretamente.

Os passos de salvamento acima descrevem o contrato implementado. Confirme a mensagem de sucesso antes de considerar uma operação persistida; sua execução nesta rodada está indicada abaixo.

## 5. Error Handling & Edge Cases

No HTML, limites são 365 livros, 50.000 páginas e 500.000 minutos. A action aceita até 1.000 livros e 1.000.000 páginas/minutos: divergência de contrato registrada. Números negativos/zero não são metas válidas. Sem meta mostra convite a definir objetivo.

### Evidência e limite desta rodada

Ano 2026 e três dimensões sem meta observados; sem sessão redirecionou. Salvamento não executado.

Falhas de rede forçadas, comparação entre duas contas e matriz mobile/tablet não executadas nesta sessão. Layout responsivo foi revisado no CSS, sem certificação visual desses tamanhos. Consulte [cobertura QA](../03-qa-coverage.md).

### Referência de implementação

- [src/app/metas/page.tsx](../../../src/app/metas/page.tsx)
- [src/components/goals/goal-form.tsx](../../../src/components/goals/goal-form.tsx)
- [src/features/goals/actions.ts](../../../src/features/goals/actions.ts)
- [src/features/goals/queries.ts](../../../src/features/goals/queries.ts)

[Índice do manual](../00-overview.md) · [Primeiros passos](../01-getting-started.md) · [Referência técnica](../02-technical-reference.md)
