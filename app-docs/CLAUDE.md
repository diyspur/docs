# CLAUDE.md — DIYSPUR frontend

Guia de implementação de produto, UI e código. Antes de propor alteração, confira os contratos no código atual; em caso de divergência, o comportamento implementado e os tipos/schema versionados prevalecem sobre esta explicação.

## Identidade de marca e estilo visual

DIYSPUR é um clube de leitura com estética **moderna, acolhedora, contemplativa e focada na leitura**, seguindo o design system do **Nimble (gitnimble.com)** com suporte nativo aos temas **Light (padrão)** e **Dark**, construído estritamente com abordagem **Mobile First**.

### Paleta mapeada no código

Valores definidos em `src/styles/tokens.css` organizados em **primitivo → semântico → componente**:

#### Tema Light (padrão)
| Papel | Variável Nimble | Variável Legada | Valor |
|---|---|---|---|
| Marca Primária | `--brand` | `--color-primary` | `#005bff` |
| Hover Marca | `--brand-hover` | `--color-primary-hover` | `#0049cc` |
| Ativo Marca | `--brand-active` | — | `#003dad` |
| Texto sobre Marca | `--brand-text` | `--color-on-primary` | `#ffffff` |
| Fundo Geral | `--background` | `--color-bg` | `#ffffff` |
| Fundo Leve | `--background-light` | — | `#fafafa` |
| Fundo Escuro / Seções | `--background-dark` | — | `#f5f6f8` |
| Superfície 1 (Cards) | `--surface-1` | `--color-surface` | `#ffffff` |
| Superfície 2 (Elevada) | `--surface-2` | `--color-elevated` | `#fafafa` |
| Superfície 3 (Overlay) | `--surface-3` | `--color-overlay` | `#f2f3f5` |
| Texto Principal | `--text-primary` | `--color-fg` | `#282a2f` |
| Texto Secundário | `--text-secondary` | `--color-muted` | `#5a5e66` |
| Texto Terciário / Muted | `--text-tertiary` | `--color-subtle` | `#8a8d94` |
| Divisores | `--separator` | — | `rgba(0, 0, 0, 0.08)` |
| Bordas | `--border` | `--color-border` | `rgba(0, 0, 0, 0.12)` |
| Bordas Fortes | `--border-strong` | `--color-border-strong` | `rgba(0, 0, 0, 0.20)` |
| Sucesso / Positivo | `--positive` / `--positive-text` | `--color-success` | `#009e66` / `#087a52` |
| Erro / Negativo | `--negative` / `--negative-text` | `--color-danger` | `#d84848` / `#c13f3f` |
| Modificado / Acento | `--modified` | `--color-accent` | `#005bff` |
| Não Rastreado / Terciário | `--untracked` | — | `#8e5ae8` |

#### Tema Dark
| Papel | Variável Nimble | Variável Legada | Valor |
|---|---|---|---|
| Marca Primária | `--brand` | `--color-primary` | `#005bff` |
| Hover Marca | `--brand-hover` | `--color-primary-hover` | `#1f6fff` |
| Ativo Marca | `--brand-active` | — | `#3884ff` |
| Fundo Geral | `--background` | `--color-bg` | `#0b0d12` |
| Superfície 1 (Cards) | `--surface-1` | `--color-surface` | `#141820` |
| Superfície 2 (Elevada) | `--surface-2` | `--color-elevated` | `#191d26` |
| Superfície 3 (Overlay) | `--surface-3` | `--color-overlay` | `#202530` |
| Texto Principal | `--text-primary` | `--color-fg` | `#f7f8fb` |
| Texto Secundário | `--text-secondary` | `--color-muted` | `#b8bdc8` |
| Texto Terciário / Muted | `--text-tertiary` | `--color-subtle` | `#858c99` |
| Divisores | `--separator` | — | `rgba(255, 255, 255, 0.10)` |
| Bordas | `--border` | `--color-border` | `rgba(255, 255, 255, 0.14)` |
| Bordas Fortes | `--border-strong` | `--color-border-strong` | `rgba(255, 255, 255, 0.24)` |
| Sucesso / Positivo | `--positive` / `--positive-text` | `--color-success` | `#009e66` / `#5fe0ab` |
| Erro / Negativo | `--negative` / `--negative-text` | `--color-danger` | `#d84848` / `#ff8d8d` |

Use a ordem de tokens **primitivo → semântico → componente**. Prefira os tokens do Nimble (`--brand`, `--background`, `--surface-1..3`, `--text-primary..tertiary`, `--border`, `--separator`) ou a ponte semântica (`--color-*`). Nunca repita valores hexadecimais soltos nos componentes.

### Tipografia e escala

- **Interface & Títulos**: `"Inter", -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif` (`--font-body`, `--font-display`, `--font-serif`).
- **Mono**: `ui-monospace, "SF Mono", SFMono-Regular, Menlo, Monaco, Consolas, monospace` (`--font-mono`).
- **Escala Modular (Fluid / Mobile First)**:
  - Display XL: `clamp(2.25rem, 1.6rem + 3.2vw, 3.75rem)` (`--text-display-xl`)
  - Display LG / H2: `clamp(1.75rem, 1.25rem + 2.2vw, 2.75rem)` (`--text-display-lg`)
  - Títulos: `1.25rem` (`--text-title`), `1.1rem` (`--text-title-sm`)
  - Corpo: `1rem` (`--text-body`), `0.875rem` (`--text-body-sm`), Legenda: `0.75rem` (`--text-caption`)
- **Pesos**: Regular `400`, Medium `500`, Semibold `600`, Display `700`.
- **Espaços**: base `4px` (`--space-1`), até `128px` (`--space-32`).
- **Raios**: botão `999px` (`--radius-button`), card `16px` (`--radius-card`), large `24px` (`--radius-large`), pill `999px` (`--radius-pill`).
- **Sombras**: progressivas `--shadow-xs` a `--shadow-xl`, e `--shadow-brand` com `color-mix`.
- **Movimento**: transições de `150ms` (`--duration-fast`) e `220ms` (`--duration-base`); sempre respeitar `prefers-reduced-motion`.

### Componentes e comportamento de UI (Mobile First)

- **Mobile First por Padrão**: Todos os blocos começam projetados para telas móveis (`375px+`), usando `@media (min-width: ...)` para progressão:
  - Mobile (< 640px): coluna única, CTAs em largura total (`.hero-cta { flex-direction: column; width: 100% }`), touch targets $\ge 48\text{px}$, padding `16px`.
  - Tablet (≥ 640px / 768px): grids em 2 ou 3 colunas, botões em linha.
  - Desktop (≥ 860px / 1024px): grids de 3 ou 4 colunas, barra de navegação completa.
- **Botões Nimble**: `.btn`, `.btn-primary`, `.btn-secondary`, `.btn-ghost`, `.btn-lg`, `.btn-sm`, `.btn-block` (estilo pill, touch target $\ge 48\text{px}$).
- **Navegação & ThemeSwitch**:
  - Header fixo com `backdrop-filter: blur(16px)` e altura `64px` (`--nav-height`).
  - Menu mobile: Drawer suspenso (`MobileNav` / `.nav-mobile-panel`) com backdrop e links rápidos.
  - ThemeSwitch: Alternador pill com polegar deslizante suave, ícones SVG e zero FOUC via script no `<head>`.
- **Janela de Mockup macOS**: `.mac-window-frame` com semáforo Apple, toolbar e preview da leitura.
- **Feature Grid**: `.feature-grid` com gaps de 1px usando `--separator` e células `.cell`.
- **Preços / Adesão**: `.price-card` com visual Nimble, badge de gratuidade e garantias.
- **Catálogo de Livros**: `.book-card` com proporção de capa `2 / 2.9`, elevação suave e pílulas de tags (`.tag`).
- **Enquetes e Votações**: `.poll-card`, `.poll-options`, `.poll-bar` e metadados de encerramento em `--surface-1`.
- **Grids de Métricas**: `.metric-grid` auto-fit com células `.metric-card`, valores em `--text-display-lg` e rótulos discretos.
- **Notificações**: `.notification-list` com itens interativos e dots de status (`.notification-dot`).
- **Vídeo Facade**: `.video-facade` 16:9 sob demanda, poster em alta resolução, botão de play com blur e zero tracking inicial.
- **Pergunta do Anfitrião**: `.host-prompt-card` com 3 alternativas e contadores de votos em tempo real.
- **Timeline de Temporada & Anti-Duplicação**:
  - Regra obrigatória em `/temporadas/[slug]`: milestones e capítulos com mesmo número compartilham a mesma linha; suprimir cards de milestones redundantes.
  - Títulos são higienizados removendo repetições de `"Capítulo X — "`.
  - Breadcrumbs não repetem o nome da temporada.
- **Componentes e Utilitários Auxiliares**:
  - `cleanChapterTitle` e `chapterLabel` (`src/lib/chapter-label.ts`): remove prefixos redundantes de capítulos e formata labels limpos.
  - `memberLinks` (`src/lib/member-navigation.ts`): centraliza os 10 links da área pessoal em desktop, mobile, perfil e histórico.
  - `supportEmail` (`src/lib/site-contact.ts`): define o e-mail canônico de contato e suporte (`oi@diyspur.com`).
  - `toLocalDateTime` (`src/components/admin/admin-controls.tsx`): converte timestamps ISO UTC para horário local do dispositivo sem drift em campos `<input type="datetime-local">`.
  - `EmptyState` (`src/components/empty-state.tsx`) e `SubmitButton` (`src/components/submit-button.tsx`) com pending state nativo do React.
- **Landmark `<main>` Único**: O elemento `<main id="conteudo">` em `src/app/layout.tsx` é o único landmark `<main>` do documento. Páginas internas devem utilizar `<section>`, `<article>` ou `<div>`, nunca um segundo `<main>`.

Breakpoints de teste: **375, 640, 768, 860, 1024 e 1440px**. Mantenha HTML semântico, `<html lang="pt-BR">`, labels, alt text, `aria-label`, skip link, foco de teclado visível (`:focus-visible`) e ausência de rolagem horizontal.

## Padrões de código e boas práticas

### Nomeação e imports

- Componentes React, tipos, interfaces, enums: `PascalCase`.
- Variáveis, funções, handlers: `camelCase`.
- Arquivos de componentes/helpers/features: `kebab-case`; manter convenções App Router para `page.tsx`, `layout.tsx`, `route.ts`, `loading.tsx`.
- Rotas/classes CSS: `kebab-case`; colunas seguem naming do schema backend.
- Use `@/*` para módulos de `src/*`; agrupe dependências externas antes das internas e use `import type` para tipos.

### TypeScript e React

- `tsconfig.json` tem `strict: true`, `noEmit: true`, alias `@/*`, `moduleResolution: bundler` e runtime `react-jsx`.
- **Não use `any`, `@ts-ignore`, cast duplo para apagar erro ou desative regra sem justificativa e permissão.** Valide entradas incertas com Zod e modele a resposta explicitamente.
- Prefira Server Components e dados de servidor; transforme em Client Component apenas quando interação/browser for necessária.
- Não introduza estado client-side para dados que já possam ser obtidos com Server Component.
- Mantenha `src/features/catalog/queries.ts` como ponto comum para reads de domínio; evite SQL/PostgREST duplicado por rota.
- `src/types/database.ts` é schema/tipagem sincronizada do repositório `diyspur-cloud/db`; sincronize a fonte geradora em vez de editar o snapshot à mão.
- Não suponha que campo existe porque parece útil: confira `Database` e migrations/contracts em `db`.

### Segurança de dados

- Segredos server-only nunca usam `NEXT_PUBLIC_`. Frontend usa só URL Supabase e publishable key pública.
- Para SSR, use `src/lib/supabase/clients.ts`; para browser, `src/lib/supabase/browser.ts`. Não misture imports desses módulos.
- Autorize com `auth.getClaims()` validado e derive usuário de `claims.sub`; nunca confie em `user_metadata`, role do client ou `user_id` recebido do form.
- RLS do backend é parte do modelo de autorização, não uma alternativa ao guard de rota. Escritas novas precisam validar sessão, schema/input e policies/RPC no backend.
- Valide URL de retorno, mantenha links externos `https` e `noopener noreferrer`, sanitize valores exportados e limite consultas.
- Não faça mutações, DDL ou signup real em produção para “testar”. Use staging e fixtures/usuários de teste.

### Pastas e commits

- `src/app/`: rotas, layouts, metadados, API Route Handlers e telas de erro/loading.
- `src/components/`: componentes UI reutilizáveis.
- `src/features/<domain>/`: queries e lógica de domínio.
- `src/lib/`: clientes, funções utilitárias e segurança.
- `src/styles/`: tokens CSS e estilos globais.
- `src/types/`: tipos do schema backend.
- `tests/unit/`, `tests/e2e/`: Vitest e Playwright.

Siga commits **Conventional Commits** observados no histórico: `feat: ...`, `fix: ...`, `chore: ...`, `docs: ...`, `test: ...`. Mensagem curta, descreve a intenção, sem combinar refactor alheio ao pedido.

## Comandos principais

```bash
npm ci
npm run dev
npm run build
npm start
npm run lint
npm run typecheck
npm test
npm run test:e2e
npm run audit
npm audit
```

`npm test` executa `tests/unit/**/*.test.ts`. `npm run test:e2e` executa Playwright; instala Chromium com `npx playwright install chromium` se necessário. `npm run audit` audita somente dependências de produção; `npm audit` audita todas.

Antes de encerrar mudança, rode ao menos `npm run lint`, `npm run typecheck` e os testes afetados. Alterações em SSR, roteamento ou CSS global também exigem `npm run build`; alterações visuais exigem E2E/inspeção nos breakpoints pertinentes.

## Auth callbacks e produção

Mantenha Site URL/Redirect URLs do Supabase sincronizados com o host atual. Google/GitHub dependem de credenciais externas e configuração administrativa — não habilitar nem simular apenas adicionando um botão.

`NEXT_PUBLIC_SITE_URL` alimenta metadata, sitemap e robots (robots é prerenderizado). No deploy, defina-o **antes do build** para a URL canônica do ambiente; faça rebuild quando o domínio muda. Configure as três variáveis do `.env.example` no ambiente de deploy, sem commitar `.env.local`.
