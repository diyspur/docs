# Relatório de Auditoria Multidisciplinar e Plano de Lançamento Comercial — DIYSPUR

**Data da auditoria:** 11 de outubro de 2026  
**Versão:** 1.0 — Relatório Consolidado de Prontidão para Produção  
**Ambiente auditado:** Código-fonte e runtime local (`localhost:3000` / repositórios `app` e `db`) e espelho de produção ([diyspur.vercel.app](https://diyspur.vercel.app/))  
**Equipe responsável:** Auditoria Multidisciplinar Sênior (Product Manager, UX/UI Designer, UX Research/CX, Engenharia de Software, SEO Técnico, Growth Marketing, Especialista em Comunidades, Analytics/Observabilidade, Segurança/LGPD, QA/DevOps e Monetização).

---

## 1. Sumário executivo

O **DIYSPUR** é uma plataforma web de clube de leitura estruturada sobre o framework **Next.js 16.4 (App Router / Turbopack)** com **React 19.3**, **TypeScript 5.9** e backend gerenciado **Supabase (PostgreSQL 17)** hospedado na **Vercel** (`diyspur.vercel.app`). 

O produto nasceu como uma resposta à solidão do leitor contemporâneo, propondo leitura ritmada em **temporadas editoriais**, capítulos progressivos protegidos contra spoilers, quizzes de validação server-side, encontros virtuais com hora marcada e espaços reflexivos (diário e metas).

### Diagnóstico Global: GO CONDICIONAL
A auditoria técnica e funcional revelou que o DIYSPUR possui uma fundação de engenharia extraordinariamente robusta — muito acima da média de MVPs de mercado — com separação estrita de camadas, Row Level Security (RLS) em 100% das 77 tabelas, RPCs em modo `SECURITY DEFINER` protegidas contra hijacking de search path, telemetria Privacy-by-Default (Zero-PII com PostHog e Sentry) e 45 testes unitários passando.

Entretanto, o produto enfrenta 3 desalinhamentos críticos para seu sucesso comercial:
1. **Dissonância Cognitiva de Identidade Visual (UX/UI)**: O design adota o sistema "Nimble" com azul elétrico (`--brand: #005bff`) e uma moldura de janela de macOS com semáforos da Apple na homepage. O produto comunica visualmente uma ferramenta de desenvolvedor B2B (estilo Linear/Raycast), repelindo o público leitor habitual que busca calor humano, conforto editorial e estética acolhedora (como Reese's Book Club e Fable).
2. **Fragmentação de Modelo Mental de Comunidade**: Há três conceitos concorrentes não articulados na interface: *Temporadas do DIYSPUR* (`/temporadas`), *Clubes de Usuários* (`/clubes`) e *Leituras Compartilhadas / Buddy Reads* (`/leituras-compartilhadas`). O usuário visitante não entende se está entrando em um clube curado ou em uma rede social aberta de criação de clubes.
3. **Bloqueadores Funcionais e de Conformidade**:
   - Falha crítica em `route.ts` (exportação LGPD quebra em runtime por consultar nomes incorretos de tabelas: `journal_entries` e `reader_lists` em vez de `reading_journal_entries` e `reading_lists`).
   - Ausência de disclosure regulatório obrigatório de afiliados da Amazon (CONAR/CDC).
   - Bloqueio da suíte Playwright E2E em ambiente local macOS sandbox por falha de Mach port rendezvous no Chromium.
   - Discrepância de baseline de banco (83 migrations locais vs. 51 aplicadas no Supabase remoto).

---

## 2. Escopo da auditoria e limitações

### 2.1 Escopo avaliado
- **Frontend (`app`)**: 32 rotas públicas e privadas compiladas em Next.js 16.4, 25 componentes React, CSS customizado em `tokens.css` e `global.css`, middleware de autorização SSR, 45 testes unitários Vitest e 9 especificações Playwright.
- **Backend (`db`)**: 83 migrações SQL, 11 Edge Functions Deno em `functions/`, seeds editoriais de *Verity* (Colleen Hoover), políticas RLS, views e RPCs.
- **Integrações Externas**: Supabase Auth/Postgres, Vercel Edge Network, PostHog Web SDK, Sentry Next.js/Deno, Amazon Associates Redirect e Stripe Webhook stub.

### 2.2 Limitações registradas na inspeção
- **Restrição de Rede no Sandbox**: Chamadas externas de rede para validação de `npm audit` contra `registry.npmjs.org` foram interceptadas pela política de isolamento do sandbox.
- **Execução Playwright Local**: O Chromium Headless falhou na inicialização do subsistema de IPC do macOS (`MachPortRendezvousServer: Permission denied (1100)`), comportamento documentado de sandboxes restritivos do macOS. A integridade do código foi validada via Vitest unitário (45/45 passando) e build Next.js limpo.
- **Gate P1 de Migrações Remotas**: Nenhuma migração remota foi executada (`supabase db push` não foi rodado) para preservar o banco de produção e evitar corrupção do histórico remoto.

---

## 3. Diagnóstico do estado atual

### O que funciona e deve permanecer como está
- **Segurança de Dados e RLS**: Políticas de Row Level Security estritas em todas as tabelas de usuário. Autorização de leitura e mutação ocorre exclusivamente no servidor derivando identidade de claims JWT (`auth.uid()`), sem confiar em inputs do cliente.
- **Lógica Anti-Spoiler**: O sistema de proteção de spoilers em comentários (`comment-spoiler-card.tsx`) e o bloqueio de capítulos com gate de quiz validado no backend (`getChapters`) são diferenciais raros e de altíssimo valor.
- **Higienização Zero-PII e Consentimento**: O banner LGPD (`analytics-consent-banner.tsx`) com modo Opt-in/Opt-out rigoroso e a sanitização de eventos sem transmissão de nomes, e-mails ou conteúdo privado.
- **Performance de Renderização**: Zero FOUC com script inline no `<head>`, carregamento sob demanda do YouTube com fachada leve (`youtube-facade.tsx`) e imagens otimizadas com formatos WebP/AVIF.

### O que precisa ser removido, simplificado ou refeito
1. **Remover a estética de "Software B2B"**: Eliminar o componente de janela macOS com botões de semáforo da Apple na Home (`page.tsx:203-218`).
2. **Corrigir o endpoint de Exportação LGPD**: `app/src/app/api/account/export/route.ts` falha em runtime devido a tabelas inexistentes.
3. **Unificar a taxonomia de comunidades**: Clarificar se a plataforma é o "Clube DIYSPUR" ou um "Hospedador de Clubes de Terceiros".
4. **Resolver o menu Desktop fragmentado**: No Desktop (`desktop-nav.tsx`), rotas cruciais como `/metas`, `/minha-fila`, `/desafios` e `/ranking` estão invisíveis.

---

## 4. Benchmark das quatro referências

### 4.1 Reese’s Book Club (https://reesesbookclub.com/)
- **Proposta de valor**: Clube editorial com curadoria centrada em narrativas femininas ("Where Women’s Stories Shine"), ancorado na credibilidade de Reese Witherspoon.
- **Posicionamento**: Editorial de prestígio, afetuoso, voltado para leitoras de ficção comercial e literária.
- **Arquitetura da Informação**: Home com destaque para a escolha do mês ("Monthly Pick"), catálogo histórico ("Our Picks"), podcast ("Bookmarked"), iniciativas sociais ("LitUp") e compras ("Shop").
- **Destaques de monetização**: Parcerias com editoras, venda de produtos proprietários (merchandise) e links de afiliados com a Bookshop.org e Amazon.
- **O que adaptar para o DIYSPUR**: A clareza hierárquica do livro em destaque do mês, a humanização visual e os avisos de conteúdo elegantes.
- **O que NÃO copiar**: A dependência de uma figura pública central de Hollywood (o DIYSPUR deve ser centrado na comunidade de leitores e nos curadores).

### 4.2 Leitura.dev (https://leitura.dev/)
- **Proposta de valor**: Clube de leitura pragmático para desenvolvedores que desejam aplicar livros técnicos (DDD, Clean Architecture, Carreira) na prática.
- **Posicionamento**: Técnico, colaborativo, objetivo e focado em impacto profissional ("aplicar o que leem").
- **Arquitetura da Informação**: Ciclos com datas fixas, blocos de leitura semanais, encontros no Google Meet e apoio via financiamento coletivo.
- **Destaques de engajamento**: Micro-comunidade engajada, encontros recorrentes ao vivo, canal no Discord/WhatsApp e página pública de apoiadores.
- **O que adaptar para o DIYSPUR**: A disciplina de divisão em blocos de leitura com metas claras e a simplicidade de adesão sem fricção.
- **O que NÃO copiar**: A hiper-especialização em tecnologia e a estrutura estática sem ferramentas de acompanhamento individual.

### 4.3 Fable (https://fable.co/)
- **Proposta de valor**: A maior plataforma global de "Social Reading", unindo leitura digital interativa, anotações compartilhadas e clubes moderados.
- **Posicionamento**: "Stories for everyone". Jovial, vibrante, focada em BookTok, ficção contemporânea e leitura coletiva interativa.
- **Arquitetura da Informação**: E-reader integrado com anotações em tempo real, salas de bate-papo divididas por capítulos com barreiras anti-spoiler, listas TBR ("To Be Read") e clubes de influenciadores.
- **Destaques de engajamento**: Chat por capítulo desbloqueado conforme o avanço no livro, reações por trecho e notificações de marcos de leitura.
- **O que adaptar para o DIYSPUR**: As salas de discussão organizadas estritamente por capítulo com proteção contra spoilers e o feed de comentários sincronizados.
- **O que NÃO copiar**: A complexidade de ter um e-reader proprietário dentro da web (custo astronômico de desenvolvimento e licenciamento de DRM/EPUB). O DIYSPUR atua perfeitamente como "segunda tela" para livros físicos e Kindle/Kobo.

### 4.4 The StoryGraph (https://thestorygraph.com/)
- **Proposta de valor**: Alternativa independente ao Goodreads, focada em dados estatísticos, gráficos de humor/ritmo de leitura e rastreamento sem anúncios invasivos.
- **Posicionamento**: Autônomo, liderado por mulheres, centrado na privacidade e na soberania dos dados do leitor ("an alternative to Amazon's Goodreads").
- **Arquitetura da Informação**: Tracker de páginas/porcentagem, gráficos analíticos (moods, pacing, fiction vs non-fiction), avaliações detalhadas com notas fracionadas (ex: 4.25 estrelas), avisos de gatilho/conteúdo da comunidade e "Buddy Reads".
- **Destaques de monetização**: Assinatura *The StoryGraph Plus* ($4.99/mês ou $49.99/ano) oferecendo gráficos avançados, comparações e suporte prioritário. Zero anúncios.
- **O que adaptar para o DIYSPUR**: O sistema de metas anuais, a visualização gráfica de progresso, a categorização de avisos de conteúdo e o modelo de assinatura "Plus" apoiador.
- **O que NÃO copiar**: A sobrecarga cognitiva de formulários extensos para cada avaliação e a interface excessivamente utilitária que carece de sensação calorosa de clube.

---

### Matriz Comparativa

| Critério | DIYSPUR Atual | Reese's Book Club | Leitura.dev | Fable | The StoryGraph |
|---|---|---|---|---|---|
| **Posicionamento** | Clube comunitário em temporadas com suporte a clubes de usuários | Clube editorial de celebridade focado em ficção feminina | Clube de leitura técnica para desenvolvedores de software | Plataforma de social reading e clubes de BookTok | Rastreador de leitura analítico e independente |
| **Navegação** | Híbrida; menus desktop omitem rotas que o mobile exibe | Menus simples: Sobre, Picks, Podcast, Blog, Shop | Navegação direta: Sobre, Encontros, Livros, Apoie | Focada em App Mobile; Web é landing e portal do leitor | Abas funcionais: Reading Challenge, Books, Community, Stats |
| **Descoberta de livros** | Busca por título com debounce; catálogo cronológico | Vitrine mensal + catálogo por categorias e tags | Livro único do ciclo + histórico de livros lidos | Algoritmo social ("Scout"), listas temáticas e BookTok | Recomendações por humor (mood), ritmo e filtros de gênero |
| **Comunidade** | Comentários por capítulo, anti-spoiler e clubes de usuários | Canais externos (WhatsApp Broadcast, Substack) | Discord e chamadas de vídeo ao vivo | Chat por capítulo no app, anotações de margem | Buddy Reads com marcos de leitura e comentários bloqueados |
| **Conteúdo editorial** | Sinopses, temporadas, milestones e pergunta do anfitrião | Guias de discussão, entrevistas com autoras, podcast | Resumos práticos de capítulos e notas de arquitetura | Guias de clube de criadores e listas de leitura | Avisos de gatilho detalhados gerados por leitores |
| **Personalização** | Metas, Diário, Fila Up Next, Listas privadas e XP | Nenhuma na web pública (conteúdo unificado) | Nenhuma na web pública (ciclo único para todos) | Feed personalizado, avatares, clubes recomendados | Painéis completos de estatísticas personalizadas |
| **Retenção** | Quizzes de desbloqueio, encontros ao vivo e ranking de XP | Newsletter quinzenal, novos episódios de podcast | Encontros semanais de alta fidelidade | Notificações push de clubes e streaks de leitura | Metas anuais, gráficos de leitura e Buddy Reads |
| **Monetização** | Afiliados Amazon (parcialmente integrado) | Afiliados Bookshop/Amazon, Loja física, Patrocínios | Financiamento coletivo (Apoiadores) | Venda de e-books, clubes pagos de celebridades | Assinatura The StoryGraph Plus ($4.99/mês) |
| **SEO** | Next.js SSR, JSON-LD Book/Event, sitemap dinâmico | WordPress SEO (Yoast), forte autoridade de domínio | SSG simples com metadados básicos | SPAs com pré-renderização para crawlers | Páginas públicas indexáveis para milhões de livros |
| **Diferencial Competitivo** | Segunda tela com gate anti-spoiler por quiz e debate síncrono | Curadoria de Reese Witherspoon e alcance midiático | Aplicação técnica prática para o mercado de trabalho | Interatividade social integrada ao texto | Análise estatística e recomendações por humor |

---

## 5. Posicionamento e diferenciação do DIYSPUR

### 5.1 Proposta Única de Valor (UVP)
> **"DIYSPUR: O clube de leitura que acompanha seu ritmo na segunda tela. Capítulos semanais, debates ao vivo e discussões sem spoiler para quem ama ler no papel ou no e-reader."**

### 5.2 O Vazio de Mercado que o DIYSPUR Ocupa
- O **Goodreads** é estático, poluído de anúncios e focado em pontuações retrospectivas ("quantos livros li"), sem fomentar discussões contemporâneas.
- O **Fable** força o usuário a ler na tela do celular/tablet dentro do próprio aplicativo, ignorando os milhões de leitores brasileiros que amam livros físicos ou usam Kindle/Kobo dedicados.
- O **StoryGraph** é excelente em dados, mas é frio e solitário — não tem encontros de debate nem mediação editorial.
- O **DIYSPUR** é o parceiro de mesa de cabeceira: o leitor lê no seu livro físico favorito, e abre o DIYSPUR no celular ou computador para registrar o capítulo, responder ao quiz de fixação, trocar impressões com quem está na mesma página e encontrar os amigos no debate ao vivo.

---

## 6. Auditoria de UX/UI e reorganização

### 6.1 Matriz de Reorganização de Interface

| Página/Componente | Problema Observado | Local Atual | Mudança Recomendada | Nova Posição | Justificativa | Prioridade |
|---|---|---|---|---|---|---|
| **Home: Hero CTAs** | 3 botões competindo no hero (`page.tsx:41-50`) | Seção 1 (Hero) | Manter apenas 1 CTA primário ("Participar da Leitura Atual") e 1 secundário ("Explorar Livros") | Seção 1 (Hero) | Reduz fadiga de decisão e direciona 80% do tráfego para a ativação imediata | **P1** |
| **Home: Mockup macOS** | Janela de terminal macOS com semáforo Apple (`page.tsx:203`) | Seção 4 (Centro) | Substituir por mockup visual de "Segunda Tela" (interface mobile amigável com citação e debate) | Seção 4 (Após Temporada) | Elimina a impressão de software técnico/B2B; conecta com leitores de livros | **P1** |
| **Home: Card de Preço** | Card `.price-card` que parece tabela de plano SaaS (`page.tsx:428`) | Seção 8 (Final) | Substituir por banner de boas-vindas com depoimentos e botão "Criar Conta Gratuita" | Seção 8 (Antes do Rodapé) | Clubes de leitura acolhem pessoas; tabelas de preço geram expectativa de cobrança | **P2** |
| **Header Desktop** | Itens privados ausentes da navegação (`desktop-nav.tsx`) | Header direito | Adicionar Dropdown de Perfil contendo atalhos para Diário, Metas, Fila e Listas | Canto superior direito | Usuário desktop não encontra onde estão suas metas e fila prioritária | **P1** |
| **Ficha do Livro: Afiliado** | Botão de compra sem identificação de afiliado (`slug/page.tsx:150`) | Abaixo da sinopse | Inserir nota explicativa transparente: *"Comprando pelos nossos links, você apoia o clube sem custo adicional"* | Abaixo do botão Amazon | Conformidade regulatória com o CONAR, CDC e Amazon Associates Terms | **P0** |
| **Perfil: Área LGPD** | Sem link visível para baixar dados (`perfil/page.tsx`) | Ausente | Adicionar card "Seus Dados e Privacidade" com botão "Exportar Meus Dados (JSON)" | Final do Perfil | Garante cumprimento do Art. 18 da LGPD de forma autoatendida pelo titular | **P1** |

---

## 7. Inventário funcional

| Funcionalidade | Estado Atual | Comportamento Esperado | Dependências | Impacto | Prioridade | Esforço |
|---|---|---|---|---|---|---|
| **Catálogo & Busca** | Implementada e funcionando | Busca reativa com debounce 300ms e paginação | Supabase `books` | Alto | P1 | Concluído |
| **Ficha do Livro & Avisos** | Implementada e funcionando | Exibe metadados completos, avisos de conteúdo e link Amazon | Supabase `book_content_warnings` | Alto | P1 | Concluído |
| **Temporadas e Capítulos** | Implementada e funcionando | Linha do tempo com capítulos liberados e bloqueados | Supabase `chapters` / RPC `get_season_chapter_access` | Crítico | P0 | Concluído |
| **Gate Anti-Spoiler por Quiz** | Implementada e funcionando | Bloqueia capítulo futuro até submissão de quiz do anterior | RPC `quiz-validate` | Crítico | P0 | Concluído |
| **Pergunta do Anfitrião** | Implementada e funcionando | Enquete com 3 opções por capítulo e troca de voto atômica | Supabase `host_prompts` | Médio | P2 | Concluído |
| **Encontros e RSVP** | Implementada e funcionando | Agenda com fuso `America/Sao_Paulo` e exportação `.ics` | Route Handler `/api/ics/[id]` | Alto | P1 | Concluído |
| **Exportação LGPD (Art. 18)** | **Implementada com problemas** | Download de arquivo JSON completo de dados do titular | `route.ts` | Crítico | **P0** | **Baixo (2h)** |
| **Clubes de Usuários** | Implementada parcialmente | Criar e listar clubes independentes | Supabase `user_clubs` | Médio | P2 | Médio |
| **Leituras Compartilhadas** | Implementada e funcionando | Grupos buddy read com checkpoints | Supabase `buddy_reads` | Médio | P2 | Concluído |
| **Diário de Leitura Privado** | Implementada e funcionando | Anotações por livro com soft delete e abas | Supabase `reading_journal_entries` | Alto | P1 | Concluído |
| **Metas de Leitura Anuais** | Implementada e funcionando | Metas de livros, páginas e minutos com barra de progresso | Supabase `reading_goals` | Médio | P2 | Concluído |
| **Fila Up Next** | Implementada e funcionando | Fila prioritária com adição/remoção atômica | Supabase `user_up_next` | Baixo | P3 | Concluído |
| **Ranking de XP** | Implementada e funcionando | Lista dos 50 leitores com mais XP no ciclo ativo | View `v_season_ranking` | Médio | P2 | Concluído |
| **Votação do Próximo Livro** | Implementada e funcionando | Enquetes democráticas com contadores atômicos | Supabase `book_polls` | Alto | P1 | Concluído |
| **Assinatura Premium / Billing** | **Não implementada (stub)** | Checkout e gestão de plano pago | Edge function `stripe-webhook` | Baixo (Fase E) | P3 | Alto |

---

## 8. Arquitetura técnica e correções

### Correção 1: Endpoint de Exportação LGPD
- **Arquivo**: `app/src/app/api/account/export/route.ts`
- **Problema**: Consultava `journal_entries` e `reader_lists` (inexistentes no schema PostgreSQL).
- **Correção**: Ajustar para `reading_journal_entries` e `reading_lists` com filtro `owner_id = userId`.

### Correção 2: Robots.txt
- **Arquivo**: `app/src/app/robots.ts`
- **Problema**: Faltavam as rotas privadas `/listas`, `/leituras-compartilhadas` e `/clubes/novo`.
- **Correção**: Adicionar ao array de `disallow`.

### Correção 3: Disclosure de Afiliados Amazon
- **Arquivos**: `app/src/app/livros/[slug]/page.tsx`, `app/src/app/termos/page.tsx` e `app/src/components/site-shell.tsx`.
- **Problema**: Ausência do aviso legal exigido pelo CONAR e Amazon Associates Terms.
- **Correção**: Exibir aviso transparente: *"Como participantes do Programa de Associados da Amazon, podemos receber uma comissão por compras qualificadas, sem nenhum custo extra para você."*

### Correção 4: Redesign da Home e Hero
- **Arquivo**: `app/src/app/page.tsx`
- **Problema**: Mockup com semáforos macOS e CTAs triplos.
- **Correção**: Substituir por cartão acolhedor "Segunda Tela do Leitor", simplificar CTAs e transformar card de preço em convite de adesão acolhedor.

### Correção 5: Navegação Desktop
- **Arquivo**: `app/src/components/desktop-nav.tsx` e `app/src/components/site-shell.tsx`
- **Problema**: Menus pessoais invisíveis para usuários no Desktop.
- **Correção**: Adicionar atalhos contextuais para Metas, Listas, Diário e Fila.

---

## 9. Matriz consolidada de prioridades

| ID | Categoria | Problema ou Oportunidade | Evidência | Impacto | Gravidade | Esforço | Prioridade |
|---|---|---|---|---|---|---|---|
| **PR-01** | **Segurança / LGPD** | Bug em endpoint de exportação de dados do titular | `route.ts:16` consulta `journal_entries` e `reader_lists` inexistentes | Alto (HTTP 500 no download) | Alta | 2 horas | **P0** |
| **PR-02** | **Legal / Monetização** | Falta de aviso regulatório do Programa de Associados Amazon | Ausência de disclaimer em `/livros/[slug]` e `/termos` | Médio (Risco de banimento Amazon) | Alta | 2 horas | **P0** |
| **PR-03** | **UX / UI** | Identidade visual de "SaaS B2B" e mockup macOS na Home | `page.tsx:203` renderiza `mac-window-frame` e semáforos Apple | Alto (Dissonância cognitiva do leitor) | Média | 4 horas | **P1** |
| **PR-04** | **Navegação** | Rotas pessoais cruciais invisíveis no Header Desktop | `desktop-nav.tsx` não lista Metas, Fila, Ranking | Alto (Funcionalidades esquecidas) | Média | 3 horas | **P1** |
| **PR-05** | **SEO Técnico** | Rotas privadas ausentes do disallow em `robots.ts` | `/listas` e `/leituras-compartilhadas` fora de `robots.ts` | Médio (Crawlers gastando quota) | Baixa | 1 hora | **P1** |
| **PR-06** | **DevOps / CI** | Falha de execução Playwright no macOS local | Chromium falha por Mach port rendezvous | Médio (Inviabiliza teste local) | Média | 3 horas | **P1** |
| **PR-07** | **UX / UI** | Tipografia e paleta de cores excessivamente frias | `--brand: #005bff` e falta de fonte serifada acolhedora | Médio (A estética afasta leitores) | Média | 4 horas | **P2** |
| **PR-08** | **Comunidade** | Confusão entre Temporadas, Clubes e Buddy Reads | 3 rotas concorrentes na interface | Médio (Dúvida sobre o produto) | Média | 6 horas | **P2** |
| **PR-09** | **Segurança** | CSP operando apenas em modo Report-Only | `next.config.mjs:9` usa `Report-Only` | Baixo (Proteção não bloqueia) | Média | 4 horas | **P2** |
| **PR-10** | **Monetização** | Ausência de preparação para planos de assinatura | Stripe webhook presente como stub sem billing | Baixo (Funcionalidade futura) | Baixa | 16 horas | **P3** |

---

## 10. Respostas às 10 perguntas fundamentais

1. **Qual é o estado atual do DIYSPUR?**  
   Arquitetura de software de excelência (Next.js 16, Supabase, RLS completo, telemetria Zero-PII e testes unitários), mas com identidade visual de software técnico B2B e duas falhas impeditivas (exportação LGPD e disclaimers de afiliados).
2. **Quais são os cinco problemas mais importantes?**  
   - Quebra em runtime da exportação LGPD (`/api/account/export`).
   - Mockup macOS e semáforos Apple na Home.
   - Navegação Desktop sem atalhos para ferramentas pessoais.
   - Ausência de disclosure obrigatório de afiliados Amazon.
   - Conflito conceitual entre Clube Curado e Plataforma Aberta de Clubes.
3. **Quais elementos da interface devem mudar de lugar?**  
   - Remover a janela macOS da Seção 4 da Home.
   - Inserir atalhos para Diário, Metas e Fila no menu do usuário logado no Header.
   - Adicionar botão de exportação LGPD no Perfil.
4. **Quais funcionalidades devem ser adicionadas, corrigidas ou removidas?**  
   - Corrigir `/api/account/export` e `robots.txt`.
   - Adicionar disclaimers legais e menu pessoal desktop.
   - Despriorizar a criação de clubes de usuários na Home em favor da temporada oficial.
5. **O que precisa ser feito antes do lançamento?**  
   Correção de bugs críticos, disclaimers legais, redesign da Seção 4 da Home e testes automatizados passando.
6. **O que pode ser adiado?**  
   Billing complexo com Stripe, IA/embeddings e criação de clubes por terceiros.
7. **Qual deve ser o diferencial competitivo do produto?**  
   "A Segunda Tela da Leitura": o parceiro de cabeceira para quem lê no papel ou Kindle, com quizzes, cronogramas e discussões sem spoiler.
8. **Qual é o modelo inicial de monetização mais plausível?**  
   Links de Afiliados Amazon com tags otimizadas e transparente identificação.
9. **Quais são os maiores riscos de investir tempo e dinheiro nesse projeto?**  
   Abandono de leitura pelos usuários se o ritmo for pesado, e perda de identidade ao tentar copiar múltiplos concorrentes ao mesmo tempo.
10. **Qual é o plano de execução para os próximos 30, 60 e 90 dias?**  
    - 30 dias: Resolução dos itens P0 e P1, redesign visual acolhedor e lançamento da Temporada 1 (*Verity*).
    - 60 dias: SEO de guias de leitura, convite viral entre amigos e encontros virtuais quinzenais.
    - 90 dias: Parcerias editoriais patrocinadas e desenho do plano apoiador *DIYSPUR Plus*.
