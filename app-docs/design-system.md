# DIYSPUR — Design System & Especificação de Padrões (Nimble)

Este documento documenta todos os padrões visuais, tokens e componentes extraídos do design system do **Nimble (gitnimble.com)** implementados no **DIYSPUR (diyspur-cloud/app)** para servir como guia definitivo para desenvolvimento futuro.

---

## 1. Princípios de Design

1. **Mobile First Rigoroso**: Todo componente e tela deve ser desenvolvido pensando primeiramente no viewport de **375px**, expandindo progressivamente com `@media (min-width: ...)` para tablet e desktop.
2. **Bitemático (Light & Dark Nativo)**: Nenhum elemento possui cores estáticas hardcoded. Todos os fundos, textos, bordas e destaques usam variáveis CSS que se adaptam instantaneamente ao tema ativo (`data-theme="light"` ou `data-theme="dark"`).
3. **Zero FOUC (Flash of Unstyled Content)**: A preferência do usuário é verificada e aplicada sincronicamente no elemento `<html>` antes do primeiro paint via script inline no `<head>`.
4. **Tipografia Geométrica e Moderna**: A fonte **Inter** é o padrão universal para títulos, corpo e componentes, com tracking negativo consistente (`-0.01em` a `-0.04em`).
5. **Acessibilidade e Hit Targets**: Botões e áreas interativas móveis têm altura mínima de **48px** (ou 38px para variantes compactas secundárias). Foco de teclado visível (`:focus-visible`), contraste AAA e respeito a `prefers-reduced-motion`.

---

## 2. Tokens de Design (`src/styles/tokens.css`)

### 2.1 Paleta de Cores

#### Cores de Marca e Acento
| Token | Light | Dark | Descrição |
|---|---|---|---|
| `--brand` | `#005bff` | `#005bff` | Azul elétrico institucional |
| `--brand-hover` | `#0049cc` | `#1f6fff` | Estado de hover da marca |
| `--brand-active` | `#003dad` | `#3884ff` | Estado ativo/pressionado |
| `--brand-text` | `#ffffff` | `#ffffff` | Texto com alto contraste sobre a marca |
| `--positive` | `#009e66` | `#009e66` | Sucesso e confirmações |
| `--positive-text` | `#087a52` | `#5fe0ab` | Texto de sucesso adaptado ao contraste |
| `--negative` | `#d84848` | `#d84848` | Erros e ações destrutivas |
| `--negative-text` | `#c13f3f` | `#ff8d8d` | Texto de erro adaptado ao contraste |
| `--modified` | `#005bff` | `#3b82f6` | Azul secundário / itens em andamento |
| `--untracked` | `#8e5ae8` | `#8e5ae8` | Roxo de acento e tags especiais |

#### Superfícies e Fundos
| Token | Light | Dark | Uso |
|---|---|---|---|
| `--background` | `#ffffff` | `#0b0d12` | Fundo principal da página |
| `--background-light` | `#fafafa` | `#10131a` | Fundo de seções claras/destaque |
| `--background-dark` | `#f5f6f8` | `#151820` | Fundo de seções secundárias |
| `--surface-1` | `#ffffff` | `#141820` | Superfície primária de cards e modais |
| `--surface-2` | `#fafafa` | `#191d26` | Superfície secundária e inputs |
| `--surface-3` | `#f2f3f5` | `#202530` | Superfície terciária e overlays |

#### Tipografia e Estrutura
| Token | Light | Dark | Uso |
|---|---|---|---|
| `--text-primary` | `#282a2f` | `#f7f8fb` | Títulos e texto de alta ênfase |
| `--text-secondary` | `#5a5e66` | `#b8bdc8` | Descrições e parágrafos |
| `--text-tertiary` | `#8a8d94` | `#858c99` | Metadados, datas e legendas |
| `--separator` | `rgba(0,0,0,.08)` | `rgba(255,255,255,.10)` | Linhas de divisão de listas e grids |
| `--border` | `rgba(0,0,0,.12)` | `rgba(255,255,255,.14)` | Bordas de cards e botões |
| `--border-strong` | `rgba(0,0,0,.20)` | `rgba(255,255,255,.24)` | Bordas ativas ou em foco |
| `--hover-overlay` | `rgba(0,0,0,.05)` | `rgba(255,255,255,.08)` | Hover em botões ghost e links |

#### Ponte de Compatibilidade com o Código Legado
Para garantir que nenhum componente antigo quebre, os seguintes aliases semânticos estão ativos:
- `--color-bg` $\to$ `var(--background)`
- `--color-surface` $\to$ `var(--surface-1)`
- `--color-elevated` $\to$ `var(--surface-2)`
- `--color-overlay` $\to$ `var(--surface-3)`
- `--color-fg` $\to$ `var(--text-primary)`
- `--color-primary` $\to$ `var(--brand)`
- `--color-primary-hover` $\to$ `var(--brand-hover)`
- `--color-border` $\to$ `var(--border)`
- `--color-border-strong` $\to$ `var(--border-strong)`
- `--color-muted` $\to$ `var(--text-secondary)`
- `--color-subtle` $\to$ `var(--text-tertiary)`
- `--color-on-primary` $\to$ `var(--brand-text)`
- `--color-danger` $\to$ `var(--negative-text)`
- `--color-success` $\to$ `var(--positive-text)`

---

### 2.2 Tipografia

- **Famílias**:
  - Interface e Títulos: `"Inter", -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif` (`--font-body`, `--font-display`, `--font-serif`)
  - Código e Monospaced: `ui-monospace, "SF Mono", SFMono-Regular, Menlo, Monaco, Consolas, monospace` (`--font-mono`)
- **Pesos**:
  - `--weight-regular`: `400`
  - `--weight-medium`: `500`
  - `--weight-semibold`: `600`
  - `--weight-display`: `700`
- **Escala Modular Fluida**:
  - `--text-display-xl`: `clamp(2.25rem, 1.6rem + 3.2vw, 3.75rem)` (Títulos hero)
  - `--text-display-lg`: `clamp(1.75rem, 1.25rem + 2.2vw, 2.75rem)` (Cabeçalhos H2)
  - `--text-title`: `1.25rem` (H3 e subtítulos)
  - `--text-title-sm`: `1.1rem` (Títulos de cards)
  - `--text-body`: `1rem` (16px base)
  - `--text-body-sm`: `0.875rem` (14px descrições)
  - `--text-caption`: `0.75rem` (12px metadados)

---

### 2.3 Espaçamentos, Raios e Sombras

- **Espaçamento**:
  - `--space-1` (4px), `--space-2` (8px), `--space-3` (12px), `--space-4` (16px), `--space-5` (20px), `--space-6` (24px)
  - `--space-8` (32px), `--space-10` (40px), `--space-12` (48px), `--space-16` (64px), `--space-20` (80px), `--space-24` (96px), `--space-32` (128px)
- **Raios de Borda**:
  - `--radius-sm`: `8px`
  - `--radius-md`: `12px`
  - `--radius-card`: `16px`
  - `--radius-large`: `24px`
  - `--radius-button`: `999px` (estilo pill)
  - `--radius-pill`: `999px`
- **Sombras**:
  - `--shadow-xs`: `0 1px 2px rgba(0, 0, 0, 0.05)`
  - `--shadow-sm`: `0 2px 6px rgba(0, 0, 0, 0.06), 0 1px 2px rgba(0, 0, 0, 0.04)`
  - `--shadow-md`: `0 8px 24px rgba(0, 0, 0, 0.08), 0 2px 6px rgba(0, 0, 0, 0.05)`
  - `--shadow-lg`: `0 16px 40px rgba(0, 0, 0, 0.10), 0 4px 12px rgba(0, 0, 0, 0.06)`
  - `--shadow-xl`: `0 24px 60px rgba(0, 0, 0, 0.14)`
  - `--shadow-brand`: `0 12px 32px color-mix(in srgb, var(--brand) 28%, transparent)`

---

## 3. Padrões de Layout (Mobile First)

### 3.1 Breakpoints de Responsividade
Sempre escreva o CSS base para **Mobile** e adicione progressões usando **`min-width`**:

```css
/* Base: Mobile (375px a 639px) */
.minha-secao {
  display: flex;
  flex-direction: column;
  padding: 16px;
}

/* Tablet (>= 640px) */
@media (min-width: 640px) {
  .minha-secao {
    flex-direction: row;
    padding: 24px;
  }
}

/* Desktop (>= 860px ou 1024px) */
@media (min-width: 860px) {
  .minha-secao {
    max-width: var(--container);
    margin: 0 auto;
  }
}
```

### 3.2 Larguras de Container
- `.container`: `max-width: 1120px; margin: auto; padding-inline: 16px;` (24px em tablet, 32px em desktop)
- `.container-wide`: `max-width: 1240px; margin: auto; padding-inline: 16px;`

---

## 4. Biblioteca de Componentes

### 4.1 Botões (`.btn`)

```html
<!-- Primário (azul, preenchido, hover com glow) -->
<button class="btn btn-primary btn-lg">Ação Principal ↗</button>

<!-- Secundário (superfície elevada com borda sutil) -->
<button class="btn btn-secondary btn-lg">Ação Secundária</button>

<!-- Ghost (transparente, texto secundário) -->
<button class="btn btn-ghost btn-sm">Ação Discreta</button>

<!-- Mobile Full Width (ocupa 100% da largura em smartphones) -->
<button class="btn btn-primary btn-block">Ação em Tela Cheia</button>
```

### 4.2 Alternador de Tema (`ThemeSwitch` / `.theme-switch`)
Localizado em `src/components/theme-toggle.tsx`:
- Renderiza trilho tipo pill de `54px x 28px` com polegar circular deslizante animado.
- Contém ícones SVG de Sol e Lua para clareza visual.
- Acessível com `role="switch"`, `aria-checked` e label descritivo.
- Persiste a escolha em `localStorage.getItem('theme')` e altera `data-theme` no `<html>`.

### 4.3 Cabeçalho e Menu Mobile Drawer
- **Desktop**: Wordmark à esquerda, links centrais (`.nav-links`), ações à direita (`.theme-switch` + `.btn.btn-primary.btn-sm`).
- **Mobile**: Wordmark à esquerda, `.theme-switch` + botão de hambúrguer (`.mobile-nav-toggle`) à direita.
- **Drawer**: Ao clicar no hambúrguer, o painel `.nav-mobile-panel` desce suavemente com backdrop escurecido, links grandes com seta `→` e botões de ação em bloco (`.btn-block`). Fecha automaticamente com tecla `Escape` ou mudança de rota.

### 4.4 Seção Hero Centrada
Estrutura recomendada para páginas institucionais e landing pages:
```html
<section class="hero">
  <div class="container hero-copy">
    <div class="hero-badge">
      <span class="badge-dot" aria-hidden="true"></span>
      Badge Informativo
    </div>
    <h1>Título Principal com <em>Destaque.</em></h1>
    <p class="hero-lede">Texto introdutório claro e legível em tamanho confortável.</p>
    <div class="hero-cta">
      <a class="btn btn-primary btn-lg" href="...">Ação Principal ↗</a>
      <a class="btn btn-secondary btn-lg" href="...">Ação Secundária</a>
    </div>
    <p class="hero-micro">Garantia ou metadado sutil.</p>
  </div>
</section>
```

### 4.5 Mockup de Janela macOS (`.mac-window-frame`)
Simulação nativa de aplicativo para exibir prévias de leitura, código ou atividades:
- `.mac-window-frame`: Moldura com cantos `var(--radius-large)` e borda `--mock-border`.
- `.mac-window-toolbar`: Barra superior de 44px com semáforo Apple (`.light.red`, `.light.amber`, `.light.green`).
- `.mac-live-pill`: Badge verde pulsante de status (*"14 leitores online"*).
- `.mac-window-body`: Conteúdo em 2 colunas adaptável para 1 coluna no mobile.

### 4.6 Feature Grid de 3 Colunas (`.feature-grid`)
Estrutura com divisórias de 1px usando `--separator`:
```html
<div class="feature-grid">
  <div class="cell">
    <span class="cell-icon">📖</span>
    <h3 class="cell-title">Título do Recurso</h3>
    <p class="cell-body">Descrição detalhada do recurso ou funcionalidade.</p>
  </div>
  <!-- Mais células -->
</div>
```
- **Mobile**: 1 coluna.
- **Tablet (≥ 640px)**: 2 colunas.
- **Desktop (≥ 900px)**: 3 colunas.

### 4.7 Card de Preço / Adesão (`.price-card`)
Card centralizado de alta conversão:
- `.price-head`: Valor em `--text-display` e termo secundário.
- `.price-note`: Descrição do modelo de gratuidade ou benefícios.
- `.price-actions`: Botão de adesão primário e botão de navegação secundário.
- `.price-guarantee`: Linha de garantia com ícone de checkmark.

### 4.8 Catálogo e Cards de Livro (`.book-grid`, `.book-card`)
- `.book-grid`: Grid responsivo (`2 colunas no mobile` $\to$ `3 em tablet` $\to$ `4 em desktop`).
- `.book-card`: Card vertical com capa em proporção `2 / 2.9`, cantos arredondados, efeito hover com leve elevação e tags em pílulas (`.tag`).

### 4.9 Formulários e Campos (`.field`)
- `.field`: Altura mínima de 48px, borda sutil, foco acessível com anel azul `--brand` e sem rolagem indesejada em smartphones.
- `.auth-card`: Card centralizado para telas de login, cadastro e recuperação de senha.

### 4.10 Enquetes e Votações (`.poll-card`)
Card interativo para enquetes de temporada e escolhas do clube:
- `.poll-card`: Container em `--surface-1` com borda `--border` e raio `--radius-card`.
- `.poll-options`: Lista flexível de opções com espaçamento `--space-3`.
- `.poll-bar`: Barra de porcentagem preenchida com gradiente ou `--brand` translúcido.
- `.poll-meta`: Metadados de encerramento, total de votos e status (aberta/encerrada).

### 4.11 Grids de Métricas e Estatísticas (`.metric-grid`)
Exibição de progresso, XP, badges e médias de leitura:
- `.metric-grid`: Grid auto-fit (`repeat(auto-fit, minmax(140px, 1fr))`) em mobile expandindo para colunas proporcionais em desktop.
- `.metric-card`: Célula com padding `--space-4`, fundo `--surface-1` ou `--surface-2`, e borda `--border`.
- `.metric-value`: Número em destaque com `--text-display-lg` e peso `--weight-display`.
- `.metric-label`: Rótulo descritivo com `--text-caption`, cor `--text-secondary` e caixa alta com tracking sutil.

### 4.12 Notificações (`.notification-list`, `.notification-item`)
Lista de avisos e atividades da comunidade:
- `.notification-item`: Item interativo com hover, distinção clara de lido/não-lido via `.notification-dot`.
- Cores de status mapeadas diretamente para `--positive`, `--brand` e `--negative`.

### 4.13 Player e Fachada de Vídeo YouTube (`.video-facade`)
Elemento otimizado para reprodução de encontros e discussões sem carregar iframes pesados antes do clique:
- `.video-facade`: Proporção `16 / 9`, raio `--radius-card`, imagem poster em alta resolução com lazy load.
- `.video-facade-play`: Botão de play centralizado com backdrop blur, borda e ícone SVG.
- `.video-facade-hint`: Dica de privacidade e economia de dados informando que a conexão ao YouTube só ocorre após interação.

### 4.14 Prompt do Anfitrião (`.host-prompt-card`)
Destaque interativo para perguntas semanais com 3 alternativas e contadores em tempo real:
- Permite votação e troca de resposta instantânea com mutação otimista e sincronização server-side.
- Feedback de seleção com anel `--brand` e porcentagem animada.

### 4.15 Timeline de Temporada e Regras Anti-Duplicação
Estrutura da página de temporada (`/temporadas/[slug]`):
- **Deduplicação de Milestones e Capítulos**: Quando um milestone e um capítulo compartilham o mesmo número (`milestone.chapter_number === chapter.chapter_number`), o componente suprime o card redundante de milestone e integra os metadados (data de encontro e status) diretamente na linha do capítulo.
- **Higienização de Títulos**: Títulos vindos do banco que já contêm o prefixo `"Capítulo X — "` têm o prefixo limpo via regex para evitar títulos repetidos como `"Capítulo 1 — Capítulo 1: O Início"`.
- **Breadcrumb Limpo**: Previne duplicidade de breadcrumb (`/temporadas/t1-verity` mostra `Início / Temporadas / Verity`, sem repetição de nome).

### 4.16 Estados Vazios (`EmptyState`) e Ações com Loading (`SubmitButton`)
- `EmptyState`: Componente em `src/components/empty-state.tsx` com ícone ilustrativo, título, descrição e botão de ação para estados sem dados (ex: sem notificações, sem listas).
- `SubmitButton`: Componente em `src/components/submit-button.tsx` utilizando `useFormStatus` do React para alternar automaticamente o texto e exibir spinner ou opacidade desabilitada durante requisições.

---

## 5. Histórico da Auditoria UI/UX e Padrões Extraídos

### 5.1 O que foi identificado na auditoria inicial:
- Ausência de sistema de cores simétrico para Light e Dark mode (cores antigas eram fixas em tons escuros).
- Classes CSS ausentes ou quebradas em `global.css` (mais de 60 classes referenciadas em páginas não existiam).
- Falta de navegação móvel adaptada (ausência de drawer/menu hamburger acessível).
- Elementos com estouro horizontal em resoluções móveis inferiores a 400px.
- Duplicação de cards de capítulos e metas na página `/temporadas/t1-verity`.
- Inline styles espalhados por diversas páginas e componentes.

### 5.2 O que foi implementado:
1. **Adoção dos tokens exatos de gitnimble.com**:
   - Paleta de cores Light & Dark completas em `src/styles/tokens.css`.
   - Compatibilidade retroativa com variáveis legadas `--color-*`.
   - Script de inicialização zero FOUC em `src/app/layout.tsx`.
   - Alternador de tema em pílula `ThemeSwitch` (`src/components/theme-toggle.tsx`).
2. **Menu Mobile Drawer**:
   - Componente `MobileNav` (`src/components/mobile-nav.tsx`) com animação, transições suaves, foco gerenciado e fechamento com tecla `Escape` ou clique fora.
   - Hit targets mínimos de 48px para facilitar o toque com polegar.
3. **Refatoração Global de CSS**:
   - `src/styles/global.css` expandido com todas as classes ausentes: `.poll-*`, `.metric-*`, `.notification-*`, `.video-facade-*`, `.host-prompt-*`, etc.
   - Remoção de todos os inline styles não essenciais e padronização com classes utilitárias.
4. **Resolução de Duplicação em `/temporadas/[slug]`**:
   - Deduplicação inteligente de milestones e capítulos por número.
   - Regex para remover prefixos repetidos de títulos de capítulos.
   - Normalização de breadcrumbs.

---

## 6. Checklist para Novos Componentes e Telas

Ao implementar uma nova tela ou componente no DIYSPUR:
- [ ] O componente funciona e fica legível em viewport de **375px**?
- [ ] Os botões de call-to-action ocupam largura total em mobile ou são facilmente clicáveis com o polegar?
- [ ] O componente foi testado nos temas **Light** e **Dark**?
- [ ] Todas as cores usam variáveis CSS (`var(--...)`) ou `color-mix()` em vez de valores hexadecimais literais?
- [ ] A tipografia utiliza os tokens `--font-body` ou `--font-display` com tracking e line-height adequados?
- [ ] Os touch targets possuem pelo menos 48px de altura (ou 38px para botões pequenos)?
- [ ] Não há estouro ou rolagem horizontal da página (`scrollWidth <= clientWidth`)?
- [ ] `npm test`, `npm run typecheck`, `npm run lint` e `npm run build` continuam passando com código 0?
