# Cadastro e edição editorial de livros / `/admin/books`

## 1. Overview & Purpose

- **URL Path:** `/admin/books`
- **Access Level:** Admin
- **Core Purpose:** Manter os títulos e a ficha editorial do catálogo.

## 2. Prerequisites & Access

Administrador e autor existente no catálogo de autores para cadastro.

## 3. UI Components & Layout

Conforme código: Cadastrar novo livro; título, slug, autor, sinopse, capa HTTPS e páginas; lista de editores de livros existentes.

![Cadastro e edição editorial de livros — captura da auditoria de 12/10/2026](../images/admin-books.jpg)

*Captura real do estado disponível nesta rodada. Em telas administrativas, a imagem mostra o bloqueio de uma conta comum; não representa o editor autorizado.*

## 4. Step-by-Step User Flow

1. **Step 1:** Abra **Livros** no painel administrativo.
2. **Step 2:** Em novo livro, informe título, slug com letras minúsculas/números/hífens e selecione autor existente.
3. **Step 3:** Complete sinopse, capa HTTPS e total de páginas da edição; pressione **Cadastrar livro**.
4. **Step 4:** Para livro existente, revise título, sinopse, capa e páginas e use **Salvar livro**.
5. **Step 5:** Após confirmação, abra a ficha pública e confira conteúdo e capa. Alterar dados bibliográficos requer revisão editorial da edição.

### Interação com o backend

create_editorial_book cadastra por RPC; updateBook altera books e grava editorial_audit_log. Consultas limitam livros a 100; dados são revalidados no catálogo/ficha.

Os passos de salvamento acima descrevem o contrato implementado. Confirme a mensagem de sucesso antes de considerar uma operação persistida; sua execução nesta rodada está indicada abaixo.

## 5. Error Handling & Edge Cases

Sem permissão mostra Acesso restrito à equipe editorial. Slug duplicado, autor inválido e payload inválido devem ser tratados pela action. Criação exige capa HTTPS quando fornecida; editor existente tem validação de URL menos restrita, ponto técnico a revisar.

### Evidência e limite desta rodada

Somente bloqueio para leitor observado. Formulários e salvamento descritos por leitura do código, sem validação visual com admin.

Falhas de rede forçadas, comparação entre duas contas e matriz mobile/tablet não executadas nesta sessão. Layout responsivo foi revisado no CSS, sem certificação visual desses tamanhos. Consulte [cobertura QA](../03-qa-coverage.md).

### Referência de implementação

- [src/app/admin/books/page.tsx](../../../src/app/admin/books/page.tsx)
- [src/components/admin/admin-controls.tsx](../../../src/components/admin/admin-controls.tsx)
- [src/app/admin/actions.ts](../../../src/app/admin/actions.ts)

[Índice do manual](../00-overview.md) · [Primeiros passos](../01-getting-started.md) · [Referência técnica](../02-technical-reference.md)
