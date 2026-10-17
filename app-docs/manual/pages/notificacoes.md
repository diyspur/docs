# Notificações e convites / `/notificacoes`

## 1. Overview & Purpose

- **URL Path:** `/notificacoes`
- **Access Level:** Authenticated
- **Core Purpose:** Consultar atualizações pessoais e responder convites de clubes.

## 2. Prerequisites & Access

Sessão ativa; notificação ou convite válido para uma lista preenchida.

## 3. UI Components & Layout

Lista; título/corpo/data; marca de leitura; links seguros; convites e aceitação; estado vazio.

![Notificações e convites — captura da auditoria de 12/10/2026](../images/notificacoes.jpg)

*Captura desktop sem sessão. A imagem mostra o redirecionamento para login; a interface autenticada foi observada antes do logout, mas sua captura não ficou disponível.*

## 4. Step-by-Step User Flow

1. **Step 1:** Abra **Notificações** na área pessoal.
2. **Step 2:** Leia as atualizações e abra o destino quando houver link permitido.
3. **Step 3:** Use a ação de marcar como lida para atualizar o estado da notificação.
4. **Step 4:** Para convite de clube, confira o grupo e use **Aceitar** quando quiser participar. Aguarde confirmação antes de abrir a ficha como membro.
5. **Step 5:** Se não houver novidades, escolha uma leitura pelo CTA disponível.

### Interação com o backend

notifications filtrada por user_id; marcar lida altera read_at apenas em registro ainda não lido e do titular. Convites em user_club_invitations; accept_club_invitation valida aceitação no backend.

Os passos de salvamento acima descrevem o contrato implementado. Confirme a mensagem de sucesso antes de considerar uma operação persistida; sua execução nesta rodada está indicada abaixo.

## 5. Error Handling & Edge Cases

Sessão expirada pede login. Sem registros mostra Por enquanto, tudo tranquilo. Convite expirado/revogado não deve ser aceito. O proxy protege /notificacoes sem sessão apesar do shell de login definido na página.

### Evidência e limite desta rodada

Nenhuma notificação/convite visível na sessão. Após logout, proxy redirecionou ao login. Marcar lida/aceitar não executados por ausência de registros.

Falhas de rede forçadas, comparação entre duas contas e matriz mobile/tablet não executadas nesta sessão. Layout responsivo foi revisado no CSS, sem certificação visual desses tamanhos. Consulte [cobertura QA](../03-qa-coverage.md).

### Referência de implementação

- [src/app/notificacoes/page.tsx](../../../src/app/notificacoes/page.tsx)
- [src/app/notificacoes/queries.ts](../../../src/app/notificacoes/queries.ts)
- [src/app/notificacoes/actions.ts](../../../src/app/notificacoes/actions.ts)
- [src/components/clubs/club-invitations.tsx](../../../src/components/clubs/club-invitations.tsx)

[Índice do manual](../00-overview.md) · [Primeiros passos](../01-getting-started.md) · [Referência técnica](../02-technical-reference.md)
