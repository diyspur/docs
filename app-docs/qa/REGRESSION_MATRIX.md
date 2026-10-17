# Matriz de Regressão End-to-End — DIYSPUR

**Data:** 11 de outubro de 2026  
**Suíte de automação:** Playwright (`tests/e2e/*.spec.ts`)

---

## 1. Mapeamento dos Cenários de Regressão (REG-01 a REG-18)

| ID | Cenário | Arquivo de Teste | Ambiente | Pré-condição | Resultado Esperado | Status |
|---|---|---|---|---|---|---|
| **REG-01** | Home e proposta de valor | `tests/e2e/homepage.spec.ts` | Local / CI | Nenhuma | Hero renderiza proposta, h1 visível e CTAs funcionais | PASS |
| **REG-02** | Navegação global | `tests/e2e/homepage.spec.ts` | Local / CI | Nenhuma | Menu principal e links do rodapé respondem sem quebra | PASS |
| **REG-03** | Alternância de tema | `tests/e2e/homepage.spec.ts` | Local / CI | Nenhuma | Atributo `data-theme` alterna entre light e dark sem FOUC | PASS |
| **REG-04** | Catálogo e busca | `tests/e2e/books.spec.ts` | Local / CI | Nenhuma | Busca com debounce atualiza query `q`, busca vazia exibe empty state | PASS |
| **REG-05** | Ficha do livro | `tests/e2e/books.spec.ts` | Local / CI | Livro publicado | Detalhes, metadados, canonical próprio e Schema `Book` presentes | PASS |
| **REG-06** | Higiene de clubes | `tests/e2e/clubs.spec.ts` | Staging / Local | Nenhuma | Zero menções a fixtures residuais (QA RETESTE); arquivados ocultos | PASS |
| **REG-07** | Acesso protegido | `tests/e2e/auth.spec.ts` | Local / CI | Anônimo | `/diario` redireciona para `/entrar?redirect=%2Fdiario&reason=login-required` com aviso contextual | PASS |
| **REG-08** | Login e cadastro | `tests/e2e/auth.spec.ts` | Local / CI | Nenhuma | Validação semântica de email e sanitização de redirects | PASS |
| **REG-09** | Requisitos de senha | `tests/e2e/auth.spec.ts` | Local / CI | Nenhuma | Campo de cadastro exige no mínimo 10 caracteres (`minlength=10`) | PASS |
| **REG-10** | Visibilidade de senha | `tests/e2e/auth.spec.ts` | Local / CI | Nenhuma | Botão alternador altera input entre `password` e `text` | PASS |
| **REG-11** | Agenda de encontros | `tests/e2e/calendar.spec.ts` | Local / CI | Nenhuma | Fuso horário `America/Sao_Paulo` explícito; online sem link com aviso claro | PASS |
| **REG-12** | Spoiler disclosure | `tests/e2e/community.spec.ts` | Local / CI | Comentário spoiler | Botão acessível de revelar spoiler manipulável via teclado | PASS |
| **REG-13** | Votação comunitária | `tests/e2e/voting.spec.ts` | Local / CI | Nenhuma | Lista de enquetes renderiza coerente sem dados corrompidos | PASS |
| **REG-14** | Responsividade mobile | `tests/e2e/accessibility-visual.spec.ts` | Local / CI | Viewports 375-1440 | Ausência de rolagem horizontal em todas as larguras testadas | PASS |
| **REG-15** | Acessibilidade por teclado | `tests/e2e/accessibility-visual.spec.ts` | Local / CI | Nenhuma | Skip-link recebe foco no primeiro Tab e move para `#conteudo` | PASS |
| **REG-16** | Headers defensivos | `tests/e2e/security-headers.spec.ts` | Local / CI | Nenhuma | Resposta HTTP traz `nosniff`, `DENY`, `strict-origin` e CSP report-only | PASS |
| **REG-17** | Canonical e sitemap | `tests/e2e/security-headers.spec.ts` | Local / CI | Nenhuma | Sitemap não expõe `/ranking` nem rotas de admin; canonical exclusivo | PASS |
| **REG-18** | Exportação de evento ICS | `tests/e2e/calendar.spec.ts` | Local / CI | Encontro cadastrado | Link `/api/ics/[meetingId]` presente para adicionar à agenda | PASS |
