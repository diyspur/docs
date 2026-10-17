# Painel administrativo / `/admin`

## 1. Overview & Purpose

- **URL Path:** `/admin`
- **Access Level:** Admin
- **Core Purpose:** Concentrar os caminhos de operação editorial e moderação.

## 2. Prerequisites & Access

Sessão com autorização is_admin confirmada no servidor. Conta comum não é suficiente.

## 3. UI Components & Layout

Quando autorizado: Livros e Moderação, orientação de editor de temporada. Quando negado: Área restrita.

![Painel administrativo — captura da auditoria de 12/10/2026](../images/admin.jpg)

*Captura real do estado disponível nesta rodada. Em telas administrativas, a imagem mostra o bloqueio de uma conta comum; não representa o editor autorizado.*

## 4. Step-by-Step User Flow

1. **Step 1:** Entre com uma conta que já recebeu autorização administrativa pela operação do projeto.
2. **Step 2:** Abra /admin. O servidor consulta is_admin antes de carregar controles.
3. **Step 3:** Use **Livros** para cadastro/edição ou **Moderação** para denúncias.
4. **Step 4:** Abra o editor de uma temporada existente pelo caminho operacional definido para a equipe; o painel atual não oferece uma listagem completa de temporadas.

### Interação com o backend

auth.getClaims e RPC is_admin. Mesma checagem reaparece nas actions administrativas para evitar confiar apenas no gate da página.

Os passos de salvamento acima descrevem o contrato implementado. Confirme a mensagem de sucesso antes de considerar uma operação persistida; sua execução nesta rodada está indicada abaixo.

## 5. Error Handling & Edge Cases

Conta comum mostra Acesso somente para a equipe. Sem sessão, login. Não alterar papéis da conta para contornar o bloqueio. Fluxo de administrador autorizado não foi exercitado.

### Evidência e limite desta rodada

Acesso negado à conta de leitor e redirect anônimo confirmados.

Falhas de rede forçadas, comparação entre duas contas e matriz mobile/tablet não executadas nesta sessão. Layout responsivo foi revisado no CSS, sem certificação visual desses tamanhos. Consulte [cobertura QA](../03-qa-coverage.md).

### Referência de implementação

- [src/app/admin/page.tsx](../../../src/app/admin/page.tsx)
- [src/app/admin/actions.ts](../../../src/app/admin/actions.ts)

[Índice do manual](../00-overview.md) · [Primeiros passos](../01-getting-started.md) · [Referência técnica](../02-technical-reference.md)
