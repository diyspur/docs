# DIYSPUR — Design System & Especificação de Padrões (Nimble Next-Gen)

Este documento documenta todos os padrões visuais, tokens e componentes do design system do **Nimble (gitnimble.com)** estendidos com a arquitetura **Next-Gen Futurista** implementados no **DIYSPUR (diyspur-cloud/app)** para servir como guia definitivo para desenvolvimento.

---

## 1. Princípios de Design

1. **Mobile First Rigoroso & Container Queries**: Todo componente e tela é desenvolvido pensando primeiramente no viewport de **375px**, adaptando-se progressivamente com `@media (min-width: ...)` e `@container` para tablet, desktop e monitores ultrawide.
2. **Dual Depth & Glassmorphism Nativo**: Profundidade assimétrica:
   - **Light Mode**: Superfícies translúcidas em camadas límpidas, micropigmentação de bordas e sombras ambientais difusas (`--shadow-ambient`).
   - **Dark Mode**: Luminância ótica, bordas especulares reativas (`--border-specular`), inner glows e acrílicos escurecidos com desfoque de alta saturação (`--glass-blur`).
3. **Compensação Ótica para Telas OLED**: Ajuste fino de pesos nas fontes variáveis no modo escuro (`380` regular / `480` medium / `580` semibold / `680` bold) para eliminar a irradiação ótica (*light bleed*) e garantir leitura confortável sem esforço visual.
4. **Matriz de Densidade Dinâmica**: Alternância em tempo de execução via atributo `data-density="compact"` (para discussões dinâmicas e feeds) ou `data-density="spacious"` (para leitura imersiva e segunda tela).
5. **Zero FOUC (Flash of Unstyled Content)**: Script síncrono inline no `<head>` que injeta `data-theme`, `data-density` e `data-contrast` antes da primeira renderização no cliente.
6. **Acessibilidade Universal (WCAG AAA)**:
   - Modo Alto Contraste (`data-contrast="high"`) com taxa de contraste $\ge 7:1$ e desativação de desfoques.
   - Paletas adaptativas para daltonismo: Protanopia, Deuteranopia e Tritanopia.
   - Hit targets táteis mínimos de **48px** e respeito estrito a `prefers-reduced-motion: reduce`.
7. **Animações Aceleradas por GPU (60 FPS)**: Movimentos compostos exclusivamente via `transform` e `opacity` com curvas bezier fluidas (`--ease-spring`).

---

## 2. Tokens de Design (`src/styles/tokens.css`)

### 2.1 Paleta de Cores & Iluminação Especular

#### Cores de Marca e Acento
| Token | Light | Dark | Descrição |
|---|---|---|---|
| `--brand` | `#005bff` | `#4589ff` | Azul elétrico institucional |
| `--brand-hover` | `#0049cc` | `#5e9bff` | Estado de hover da marca |
| `--brand-active` | `#003dad` | `#78aeff` | Estado ativo/pressionado |
| `--brand-text` | `#ffffff` | `#ffffff` | Texto com alto contraste sobre a marca |
| `--positive` | `#009e66` | `#009e66` | Sucesso e confirmações |
| `--positive-text` | `#087a52` | `#5fe0ab` | Texto de sucesso adaptado ao contraste |
| `--negative` | `#d84848` | `#d84848` | Erros e ações destrutivas |
| `--negative-text` | `#c13f3f` | `#ff8d8d` | Texto de erro adaptado ao contraste |
| `--modified` | `#005bff` | `#3b82f6` | Azul secundário / itens em andamento |
| `--untracked` | `#8e5ae8` | `#8e5ae8` | Roxo de acento e tags especiais |

#### Superfícies, Glassmorphism & Iluminação Ambiente
| Token | Light | Dark | Uso |
|---|---|---|---|
| `--background` | `#ffffff` | `#0b0d12` | Fundo principal da página |
| `--surface-1` | `#ffffff` | `#141820` | Superfície primária de cards e modais |
| `--surface-2` | `#fafafa` | `#191d26` | Superfície secundária e inputs |
| `--surface-3` | `#f2f3f5` | `#202530` | Superfície terciária e overlays |
| `--border-glass` | `rgba(0,0,0,.08)` | `rgba(255,255,255,.09)` | Borda sutil de acrílico |
| `--border-specular` | `rgba(255,255,255,.85)` | `rgba(255,255,255,.18)` | Borda de reflexo especular |
| `--glass-blur` | `blur(20px) saturate(180%)` | `blur(24px) saturate(190%)` | Desfoque de vidro acrílico |
| `--shadow-ambient` | `0 20px 45px -15px rgba(15,23,42,.08)` | `0 25px 50px -12px rgba(0,0,0,.65)` | Sombra difusa de profundidade |
| `--glow-primary` | `0 0 24px color-mix(in srgb, var(--brand) 22%, transparent)` | `0 0 28px rgba(0,91,255,0.32)` | Brilho neon de marca |
| `--glow-cyan` | `0 0 25px rgba(0,188,212,0.20)` | `0 0 35px rgba(6,182,212,0.30)` | Brilho ciano secundário |

---

### 2.2 Eixos Tipográficos e Compensação OLED

```css
/* Light Mode */
--weight-regular: 400;
--weight-medium: 500;
--weight-semibold: 600;
--weight-display: 700;

/* Dark Mode (Compensação de irradiação em telas OLED) */
:root[data-theme="dark"] {
  --weight-regular: 380;
  --weight-medium: 480;
  --weight-semibold: 580;
  --weight-display: 680;
}
```

---

### 2.3 Modos de Densidade e Acessibilidade

```css
/* Densidade Compacta (Feeds Rápidos) */
:root[data-density="compact"] {
  --space-card-p: 0.875rem;
  --space-item-gap: 0.5rem;
  --input-height: 2.25rem;
  --btn-height: 2.25rem;
  --table-cell-py: 0.45rem;
}

/* Densidade Espaçosa (Segunda Tela / Conforto) */
:root[data-density="spacious"] {
  --space-card-p: 1.75rem;
  --space-item-gap: 1.25rem;
  --input-height: 3.125rem;
  --btn-height: 3.125rem;
  --table-cell-py: 1rem;
}

/* Alto Contraste (WCAG AAA) */
:root[data-contrast="high"] {
  --border: #ffffff !important;
  --border-glass: #ffffff !important;
  --border-specular: #ffffff !important;
  --text-primary: #ffffff !important;
  --background: #000000 !important;
  --surface-1: #000000 !important;
  --glass-blur: none !important;
  --shadow-ambient: none !important;
}

/* Paletas Adaptativas para Daltonismo */
:root[data-colorblind="protanopia"] {
  --positive: #0077bb;
  --negative: #ee7733;
}
:root[data-colorblind="deuteranopia"] {
  --positive: #33bbee;
  --negative: #cc3311;
}
:root[data-colorblind="tritanopia"] {
  --positive: #009988;
  --brand: #ee3377;
}
```

---

## 3. Catálogo de Componentes Core Futuristas (`src/components/ui/`)

### 3.1 `GlassCard` (`src/components/ui/glass-card.tsx`)
Card interativo com reflexo especular radial rastreando o cursor do mouse:
- **Efeito Radial Dinâmico:** Captura coordenadas `(x, y)` em `onMouseMove` e projeta um gradiente radial suave (`450px`) com a cor especificada (`blue`, `cyan`, `purple`).
- **Aceleração por Hardware:** O reflexo é projetado em uma camada absoluta com `pointer-events: none` sem causar recalculo de layout.
- **Variantes:** `interactive` (elevação e borda especular no hover), `static` e `accent`.

### 3.2 `CyberButton` (`src/components/ui/cyber-button.tsx`)
Botão tátil com micro-haptic e feedback de estado:
- **Tamanhos Ergonômicos:** `sm` (40px), `md` (48px - touch standard), `lg` (54px).
- **Variantes Estilísticas:** `primary` (gradiente com sombra neon), `secondary`, `glass`, `ghost` e `danger`.
- **Estado de Carregamento Acessível:** Spinner com anúncio para tecnologias assistivas via `aria-live="polite"`.

### 3.3 `CyberSkeleton` (`src/components/ui/cyber-skeleton.tsx`)
Loader com varredura contínua de laser especular:
- **Animação Shimmer:** Utiliza `@keyframes shimmer` para varredura de luz contínua em gradiente linear.
- **Latência Reduzida:** Transição instantânea sem layout shift (CLS = 0).

---

## 4. Otimização para Buscadores e Agentes de IA (SEO/GEO/AIO)

1. **Schema Organization & WebSite (`src/app/layout.tsx`):**
   - Dados de organização contendo links oficiais de redes sociais (`sameAs`), logotipo canônico e busca interna parametrizada via `potentialAction: SearchAction`.
2. **Schema Book Conectado à Comunidade (`src/app/livros/[slug]/page.tsx`):**
   - Metadados editoriais conectados ao canal de debate comunitário via `subjectOf: DiscussionForumPosting`.
3. **Schema LearningResource para Capítulos (`src/app/capitulos/[id]/page.tsx`):**
   - Marcação semântica estruturada categorizando o roteiro de segunda tela como recurso educacional interativo (*DiscussionGuide*).

---

## 5. Checklist para Novos Componentes e Telas

Ao implementar uma nova tela ou componente no DIYSPUR:
- [ ] O componente funciona e fica legível em viewport de **375px**?
- [ ] Os botões de call-to-action ocupam pelo menos **48px** de altura para conforto tátil?
- [ ] O componente foi testado e validado nos temas **Light** e **Dark**?
- [ ] Foi validado em modo **Alto Contraste** e nos modos de **Densidade**?
- [ ] As animações respeitam `prefers-reduced-motion: reduce`?
- [ ] As consultas SSR e schemas estruturados JSON-LD estão presentes para rotas públicas?
- [ ] `npm run typecheck`, `npm run test` e `npm run build` rodam com 100% de sucesso?
