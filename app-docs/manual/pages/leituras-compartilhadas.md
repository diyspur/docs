# Leituras compartilhadas / `/leituras-compartilhadas`

## 1. Overview & Purpose

- **URL Path:** `/leituras-compartilhadas`
- **Access Level:** Authenticated; shell público
- **Core Purpose:** Consultar grupos de leitura e seus checkpoints autorizados.

## 2. Prerequisites & Access

Sessão e vínculo de dono/membro/autorização para leituras privadas.

## 3. UI Components & Layout

- **Card de Convite Contextual (`?livro=[slug]`, Excalidraw B17):** Exibição em destaque da obra com opção de convidar leitores parceiros via e-mail ou link direto.
- **Painel de Convites Pendentes:** Link e atalho para a Central de Notificações (`/notificacoes`, F25/F26) para aceitar ou recusar convites de amigos.
- **Lista de Leituras Compartilhadas:** Grupos ativos; relação Você organiza / Você participa; livro associado; período de leitura; checkpoints e metas por páginas ou percentual; estado vazio com convite para ler em conjunto.

![Leituras compartilhadas — captura da auditoria de 12/10/2026](../images/leituras-compartilhadas.jpg)

*Captura desktop sem sessão. A imagem mostra o estado público ou o convite de acesso. As funções pessoais descritas exigem sessão.*

## 4. Step-by-Step User Flow

1. **Step 1:** Abra **Leituras compartilhadas** na área pessoal ou pelo botão "Convidar amigos" na ficha de um livro (`?livro=[slug]`).
2. **Step 2:** Se houver um livro selecionado, utilize o formulário de convite para convidar amigos para a leitura da obra.
3. **Step 3:** Verifique convites recebidos acessando o atalho para a **Central de Notificações** (`/notificacoes`).
4. **Step 4:** Acompanhe o avanço dos checkpoints e limites de leitura acordados com seus parceiros de leitura.

### Interação com o backend

Leituras em buddy_reads, vínculos em buddy_read_members e metas em buddy_read_checkpoints. Busca próprias, vínculos pessoais e privadas visíveis por RLS; associa books.

Os passos de salvamento acima descrevem o contrato implementado. Confirme a mensagem de sucesso antes de considerar uma operação persistida; sua execução nesta rodada está indicada abaixo.

## 5. Error Handling & Edge Cases

Sem sessão mostra login no próprio shell. Sem grupo mostra Nenhuma leitura compartilhada ainda. Erros de sessão, grupos, checkpoints ou livros têm mensagens distintas e opção de tentar novamente. Schema existente não comprova CRUD disponível na UI.

### Evidência e limite desta rodada

Estado vazio autenticado e convite de login sem sessão observados. Cenário com checkpoints não exercitado por ausência de grupo visível.

Falhas de rede forçadas, comparação entre duas contas e matriz mobile/tablet não executadas nesta sessão. Layout responsivo foi revisado no CSS, sem certificação visual desses tamanhos. Consulte [cobertura QA](../03-qa-coverage.md).

### Referência de implementação

- [src/app/leituras-compartilhadas/page.tsx](../../../src/app/leituras-compartilhadas/page.tsx)

[Índice do manual](../00-overview.md) · [Primeiros passos](../01-getting-started.md) · [Referência técnica](../02-technical-reference.md)
