# Relatório de Auditoria de UX, CX e Acessibilidade (SXO) — DIYSPUR

**Data da análise:** 11 de outubro de 2026  
**Escopo:** Avaliação da experiência do leitor, fricções de jornada, clareza editorial e conformidade de acessibilidade (WCAG 2.2 AA).

---

## 1. Mapeamento da Jornada do Leitor

```
Descoberta (Home / Livros) ──▶ Ficha do Livro ──▶ Agenda de Debates ──▶ Espaço Privado (Diário / Clube)
       │                               │                  │                      │
       ▼                               ▼                  ▼                      ▼
  Proposta clara              Capítulos e temporada   Fuso e acesso      Redirecionamento com
  e busca fluida              com avisos de conteúdo  esclarecidos       motivo e retorno seguro
```

---

## 2. Diagnóstico de Fricções: Intencionais vs. Acidentais

| Ponto de Contato | Comportamento Anterior | Classificação | Solução Implementada |
|---|---|---|---|
| **Acesso ao Diário sem Sessão** | Redirecionamento silencioso para `/entrar?redirect=/diario` | Fricção Acidental | Adicionado parâmetro `reason=login-required` e aviso contextual: *"Entre na sua conta para acessar este espaço. Depois do login, você voltará para a página solicitada."* |
| **Encontros Virtuais sem Link** | Exibia apenas *"Online"* sem instruções ou plataforma | Fricção Acidental | Adicionado aviso da equipe informando que a plataforma e instruções serão divulgadas, eliminando suposições. |
| **Pesquisa de Livros** | Exigia pressionar Enter ou clicar no botão "Buscar" | Fricção Acidental | Implementado `BookSearch` com debounce de 300ms, preservando formulário acessível e fallback. |
| **Revelação de Spoilers** | Marcado com blur ou ocultamento | Fricção Intencional | Preservado intencionalmente para resguardar leitores; aprimorado com controle acessível via teclado. |
| **Senha no Cadastro** | Requisito de 10 caracteres mínimos | Fricção Intencional | Mantido por segurança defensiva da conta do usuário. |

---

## 3. Estados Vazios e Tratamento de Erros

- **Catálogo de Livros**: Quando não há livros para a busca `q`, renderiza ícone semântico, mensagem amigável e link para limpar busca.
- **Calendário**: Quando não há encontros futuros, exibe aviso acolhedor de que as datas estão sendo combinadas e link para conhecer o catálogo.
- **Diário Privado**: Aba de registros vazia orienta o leitor sobre como registrar impressões logo após terminar o capítulo.
- **Avisos de Erro de Autenticação**: Mensagens claras em português simples evitando jargões técnicos de banco ou SDKs.

---

## 4. Acessibilidade Assistiva e WCAG 2.2 AA

### 4.1 Verificações Automatizadas Concluídas
- **Contraste de Cores**: Relação de contraste acima de 4.5:1 para texto normal e 3:1 para elementos de UI em ambos os temas (light e dark).
- **Semântica de Títulos**: Hierarquia estrita de headings (`h1` único por página, sem saltos de nível).
- **Landmarks ARIA**: `<header>`, `<main id="conteudo">`, `<nav>` e `<footer>` devidamente identificados.
- **Skip Link**: Primeiro foco por teclado salta navegação e entrega o foco ao conteúdo principal.
- **Target Sizes**: Botões e campos interativos respeitam dimensões mínimas recomendadas (mínimo de 44x44px ou 48x48px).

### 4.2 Verificações Manuais Recomendadas para Homologação
- [ ] Teste de navegação com leitor de tela NVDA no Windows e VoiceOver no macOS/iOS.
- [ ] Navegação completa por teclado sem auxílio de mouse em todas as páginas privadas.
- [ ] Validação do comportamento com ampliação de texto de até 200% no navegador.
