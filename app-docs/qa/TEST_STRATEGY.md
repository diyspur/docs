# Estratégia e Governança de Testes — DIYSPUR

**Data:** 11 de outubro de 2026  
**Alvo:** `diyspur-cloud/app` e `diyspur-cloud/db`

---

## 1. Princípio Fundamental de Segurança em Produção

> **REGRA DE OURO OPERACIONAL:**  
> **É TERMINANTEMENTE PROIBIDO EXECUTAR TESTES DE ESCRITA CONTRA PRODUÇÃO.**  
> Cadastro de usuários, comentários, avaliações, RSVP, votos, progresso de leitura, exclusões ou seeds NUNCA devem ser disparados apontando para `diyspur.vercel.app` ou para o projeto Supabase de produção.

Qualquer teste automatizado que execute em produção deve ser estritamente **GET / Read-Only Smoke**, validando apenas códigos de resposta HTTP, presença de headers e integridade estrutural.

---

## 2. Pirâmide e Níveis de Teste

```
          ▲
         / \     E2E / Playwright (Jornadas completas e regressão visual)
        /   \    k6 Load Tests (Staging aprovado com janela formal)
       /-----\   Integration Smoke (PGlite WASM / Edge Functions)
      /       \  Unit Tests (Vitest: schemas, parsing, redirecionamento, hooks)
     /─────────\ Static Analysis (TypeScript 5.9, ESLint 9, pglast parser)
```

### 2.1 Análise Estática
- **TypeScript**: `tsc --noEmit` em strict mode, zero tolerância a `any` implícito.
- **ESLint**: Linting com flat config cobrindo regras de acessibilidade e boas práticas Next.js.
- **SQL Parser**: `pglast` analisa as 85 migrations em busca de sintaxe inválida antes de qualquer execução.

### 2.2 Testes Unitários (Vitest)
- Validação de regras de negócio, schemas Zod de formulários, sanitização de redirecionamento (`safeLocalRedirect`), links de calendário e lógica de consentimento.
- Rápida execução (< 2s) sem dependência de rede ou banco de dados externo.

### 2.3 Smoke de Segurança Backend (PGlite WASM)
- Inicializa uma instância efêmera do PostgreSQL 17 em WebAssembly em memória.
- Aplica as migrations incrementais e valida políticas de RLS e funções RPC com múltiplos papéis.

### 2.4 End-to-End (Playwright)
- Executado via Chromium em headless mode com resolução parametrizada.
- Validação das rotas públicas, contraste, navegação por teclado e responsividade nos viewports (375px a 1440px).

---

## 3. Matriz de Usuários e Identidades para Testes

| Persona | Permissões / Papel | Cenários Típicos | Ambiente Permitido |
|---|---|---|---|
| **Anônimo (Visitante)** | `anon` | Navegação na home, catálogo, busca de livros, visualização de encontros, redirecionamento com contexto ao tentar acessar diário | Local, CI, Staging, Produção (Read-only) |
| **Leitor Comum (Usuário A)** | `authenticated` (`reader`) | Criação de entradas no diário, participação em clube, registro de progresso, voto em enquete | Local, Staging |
| **Leitor Concorrente (Usuário B)** | `authenticated` (`reader`) | Teste de isolamento de diário (A não lê B), leitura colaborativa, ausência de IDOR | Local, Staging |
| **Administrador Editorial** | `authenticated` (`admin`) | Painel editorial, moderação de comentários denunciados, aprovação de capítulos | Local, Staging |

---

## 4. Regras Anti-Flakiness

1. **Seletores Semânticos**: Priorizar sempre `getByRole`, `getByLabel` e `getByText` em detrimento de seletores CSS frágeis.
2. **Esperas Determinísticas**: Usar `expect.poll()` ou asserções automáticas do Playwright em vez de `sleep()` ou timeouts arbitrários.
3. **Isolamento de Estado**: Cada teste deve iniciar com cookies limpos e `sessionStorage` isolado.
4. **Retry Controlado**: Máximo de 1 retry no ambiente de CI para mitigar oscilações transitórias de I/O.
