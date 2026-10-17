# Cobertura QA, execução e pendências

## Método e limites

Auditoria manual browser-based em produção em 12/10/2026 (Brasília), com uma sessão de leitor já existente e, após logout, navegação anônima. Foram feitas navegações sistemáticas por rotas do App Router, inspeção de DOM/acessibilidade e screenshots. Código app/db e catálogos RLS foram consultados. Não se atribuem a esta rodada testes relatados no README ou em conversas anteriores.

As instruções locais restringem testes de escrita em produção e orientam usar staging/fixtures. A evidência disponível permitiu navegação e inspeção de conteúdo existente; fluxos de escrita foram documentados pelo código e ficaram pendentes de execução em staging. Não há conta admin autorizada na sessão. O navegador não expôs ajuste de viewport. Não usamos esses limites para afirmar que as funcionalidades falham: classificamos como não exercitadas.

## Casos realmente executados

| ID | Ação | Resultado observado |
|---|---|---|
| QA-01 | Abrir home autenticada | Meu espaço/Sair e temporada visíveis |
| QA-02 | Recusar analytics | Banner fechou; demais páginas continuaram acessíveis |
| QA-03 | Abrir catálogo | Skeleton transitório seguido de dois títulos |
| QA-04 | Buscar zzqa-inexistente e enviar | URL q, estado vazio, Limpar busca |
| QA-05 | Abrir duas fichas | Verity com temporada; Hope sem temporada |
| QA-06 | Abrir timeline autenticada | Três etapas com links |
| QA-07 | Abrir três capítulos autenticados | Primeiro concluído; seguintes com formulário/progresso zero; quiz sem resposta desabilitado |
| QA-08 | Abrir calendário/votação | RSVP cancelado existente; agenda sem link; nenhuma enquete aberta; resultado encerrado |
| QA-09 | Abrir feed/clubes | Conteúdo público e avisos; registros QA antigos visíveis |
| QA-10 | Abrir dez áreas pessoais | Histórico/diário/lista existentes; fila/grupos/notificações vazios; metas sem objetivo; ranking preenchido |
| QA-11 | Abrir gestão de lista | Campos de edição e Fechar gestão; nada salvo |
| QA-12 | Abrir diário Arquivados | Estado vazio; nenhum arquivamento novo |
| QA-13 | Abrir quatro rotas admin como leitor | Acesso negado em painel/livros/temporada/moderação |
| QA-14 | Abrir menu Explorar | Menu expandido com Livros e Clubes |
| QA-15 | Sair | Home anônima e Entrar/Fazer parte; sessão anterior encerrada |
| QA-16 | Acessar áreas pessoais sem sessão | Redirect ou shell de login conforme módulo; nenhum dado pessoal renderizado nas observações |
| QA-17 | Abrir capítulos 2/3 e timeline anônimos | Gates de quiz e convite ao login |
| QA-18 | Enviar login vazio | Informe um email válido |
| QA-19 | Enviar recuperação vazia | Informe um email válido; nenhum e-mail disparado |
| QA-20 | Inspecionar home desktop | Um main, largura 1348 sem overflow horizontal |
| QA-22 | Baixar Arquivo .ics | Arquivo VCALENDAR/VEVENT baixado; horário 23h UTC corresponde a 20h Brasília; duração 60 minutos; CRLF reais |
| QA-21 | Consultar catálogo PostgreSQL | 82 tabelas public com RLS enabled e policies selecionadas lidas |

## Registro por módulo

| Rota | Observações | Estado/limite | Documento |
|---|---|---|---|
| `/` | 2 observações registradas | Home renderizada com sessão e sem sessão. Desktop sem rolagem horizontal: clientWidth=scrollWidth=1348; um único main. A home mostrou 3/3 com sessão e 1/1 após logout, embora a timeline anônima mostrasse três capítulos. | [Guia](pages/home.md) |
| `/livros` | 3 observações registradas | Dois títulos no catálogo. Busca zzqa-inexistente executada no navegador: q na URL, estado vazio e Limpar busca visíveis. | [Guia](pages/livros.md) |
| `/livros/[slug]` | 3 observações registradas | Verity renderizada com dados editoriais, dois avisos e temporada. O massacre da família Hope renderizado sem temporada. Escritas de fila/lista e redirecionamento Amazon não executados nesta rodada. | [Guia](pages/livro.md) |
| `/temporadas/[slug]` | 2 observações registradas | Timeline Verity com três etapas abertas na sessão existente; sem sessão, capítulo 1 aberto e capítulos 2/3 bloqueados. | [Guia](pages/temporada.md) |
| `/capitulos/[id]` | 1 observações registradas | Três capítulos visitados autenticados. Capítulo 1 já concluído, respostas do anfitrião existentes e quiz com cinco perguntas. Capítulos 2/3 sem progresso nem comentários. Botão de quiz desabilitado sem respostas. Persistência, player e novo envio não executados nesta rodada. | [Guia](pages/capitulo.md) |
| `/calendario` | 1 observações registradas | Encontro exibido para 14/10/2026 às 20h Brasília, duração 60 min, sem acesso publicado. A sessão já tinha presença cancelada; não foi alterada. Exportação/importação não executada. | [Guia](pages/calendario.md) |
| `/votacao` | 1 observações registradas | Nenhuma votação aberta; uma votação encerrada com um voto e resultado agregado. Novo voto não executado porque não havia enquete aberta. | [Guia](pages/votacao.md) |
| `/comunidade` | 1 observações registradas | Feed renderizado com comentários e spoiler oculto. Registros QA de 10/10 visíveis; não criados nem removidos nesta rodada. | [Guia](pages/comunidade.md) |
| `/clubes` | 1 observações registradas | Dois clubes públicos visíveis, incluindo um clube QA antigo. Clube Verity sem ciclo associado. | [Guia](pages/clubes.md) |
| `/clubes/novo` | 2 observações registradas | Formulário renderizado autenticado; acesso sem sessão redirecionou para login preservando /clubes/novo. Criação não enviada. | [Guia](pages/clube-novo.md) |
| `/clubes/[slug]` | 1 observações registradas | Clube Verity: dois participantes, botão Entrar no clube e painel sem ciclo. Não houve ingresso, saída ou convite nesta rodada; controles do titular descritos pelo código. | [Guia](pages/clube.md) |
| `/perfil` | 2 observações registradas | Perfil carregado com atividade existente, 305 XP e um capítulo concluído. Também exibiu texto de boas-vindas orientado a quem ainda não começou; ponto de UX registrado. Exportação não baixada. | [Guia](pages/perfil.md) |
| `/historico` | 2 observações registradas | Capítulo 1 concluído, 100% lido e quiz 5/5 observados; sem sessão houve redirect. | [Guia](pages/historico.md) |
| `/diario` | 2 observações registradas | Formulário e registro privado QA existente observados; Arquivados mostrou vazio. Sem sessão, redirect contextual de login. Nenhuma criação/arquivamento executado. | [Guia](pages/diario.md) |
| `/listas` | 2 observações registradas | Lista QA privada existente com Verity; gestão abriu nome, descrição, Salvar lista e Fechar gestão. Sem sessão, Entre para ver suas listas. Nenhuma alteração salva. | [Guia](pages/listas.md) |
| `/metas` | 2 observações registradas | Ano 2026 e três dimensões sem meta observados; sem sessão redirecionou. Salvamento não executado. | [Guia](pages/metas.md) |
| `/minha-fila` | 2 observações registradas | Estado vazio com contagem zero e links de catálogo; proteção anônima por redirect. Adição/remoção não executadas. | [Guia](pages/minha-fila.md) |
| `/leituras-compartilhadas` | 2 observações registradas | Estado vazio autenticado e convite de login sem sessão observados. Cenário com checkpoints não exercitado por ausência de grupo visível. | [Guia](pages/leituras-compartilhadas.md) |
| `/notificacoes` | 2 observações registradas | Nenhuma notificação/convite visível na sessão. Após logout, proxy redirecionou ao login. Marcar lida/aceitar não executados por ausência de registros. | [Guia](pages/notificacoes.md) |
| `/desafios` | 2 observações registradas | Desafio 12 livros em 12 meses com progresso 0% na sessão existente; sem sessão listou proposta e link de login. Inscrição não executada. | [Guia](pages/desafios.md) |
| `/ranking` | 2 observações registradas | Quatro leitores listados na temporada Verity com XP, conta atual destacada pelo componente. Sem sessão redirecionou. | [Guia](pages/ranking.md) |
| `/entrar` | 1 observações registradas | Formulário público e erro de envio vazio observados. Login novo/magic link não realizados; sessão autenticada já existia no início da rodada. | [Guia](pages/entrar.md) |
| `/cadastro` | 1 observações registradas | Formulário e regras exibidas inspecionados. Nenhuma nova conta criada; confirmação por e-mail descrita pelo código e não como teste E2E concluído. | [Guia](pages/cadastro.md) |
| `/recuperar-senha` | 1 observações registradas | Envio vazio validado, sem disparo de e-mail. Entrega e uso de link não exercitados. | [Guia](pages/recuperar-senha.md) |
| `/atualizar-senha` | 1 observações registradas | Shell sem sessão observado. Nenhum campo de senha preenchido ou credencial alterada. | [Guia](pages/atualizar-senha.md) |
| `/termos` | 1 observações registradas | Conteúdo e contato renderizados. Manual descreve a tela, sem avaliar validade jurídica. | [Guia](pages/termos.md) |
| `/privacidade` | 1 observações registradas | Política renderizada; recusa de analytics executada no início sem impedir navegação. Captura de tráfego e auditoria jurídica não realizadas. | [Guia](pages/privacidade.md) |
| `/admin` | 2 observações registradas | Acesso negado à conta de leitor e redirect anônimo confirmados. | [Guia](pages/admin.md) |
| `/admin/books` | 1 observações registradas | Somente bloqueio para leitor observado. Formulários e salvamento descritos por leitura do código, sem validação visual com admin. | [Guia](pages/admin-books.md) |
| `/admin/seasons/[id]` | 1 observações registradas | Editor da temporada Verity negado à conta de leitor. Contratos de publicação lidos no código; nenhum conteúdo alterado. | [Guia](pages/admin-season.md) |
| `/admin/moderation` | 1 observações registradas | Somente bloqueio de acesso ao leitor observado; nenhuma denúncia criada ou revisada. | [Guia](pages/admin-moderation.md) |

## O que ainda precisa de execução

| Grupo | Cenários e critérios para concluir |
|---|---|
| Auth em staging | Cadastro, confirmação, login novo, logout, magic link, senha incorreta, duplicidade de usuário/e-mail, recuperação, link expirado/reutilizado |
| Leitura | Salvar 1/50/99%, concluir, não regressão, navegar antes/depois de quiz, datas futuras, rascunho, XP pendente |
| Quiz | Incompleto, completo, acerto/erro, tentativa seguinte, Idempotency-Key repetida, rate limit, token expirado |
| Comentários | Criar/editar/remover próprios, negar alteração de outro leitor, spoiler, comentário temporizado, denúncia legítima e moderação |
| Organização | Diário válido/inválido, intervalo invertido, arquivar/restaurar, listagem/cursor; lista criar/editar/adicionar/duplicar/remover/restaurar |
| Metas e fila | Persistência/derivação; limites HTML/action; fila cinco/sexto, duplicidade e concorrência |
| Comunidade | Criar clube público/privado; duas contas; convite válido/expirado/revogado; proprietário vs membro; troca de versão |
| Calendário/enquete | RSVP aceitar/cancelar persistido; ICS importar em calendário externo; revisar outros fusos; encontro cancelado; voto em enquete aberta/fechada e duplicado |
| Admin | Conta autorizada; criar/editar livro; publicar/agendar com cinco questões/replay; status do ciclo; resolver/dispensar/ocultar denúncia |
| Dados pessoais | Exportação autorizada/401; verificar limites, arrays, falhas parciais e domínios ausentes; isolamento com contas A/B |
| Falhas | Offline/timeout, indisponibilidade do banco, Error Boundary, reset, duplicidade após resposta ambígua |

## Matriz responsiva pendente

| Largura | Estado desta rodada | O que conferir |
|---|---|---|
| 375 px | Não executado | Menu mobile, formulário, CTA, alvo de toque, overflow |
| 640 px | Não executado | Transição de cards/grids e botões |
| 768 px | Não executado | Tablet, edição, player e listas |
| 860 px | Não executado | Troca do menu e colunas |
| 1024 px | Não executado | Conteúdo/aside, cards, tabelas |
| 1440 px | Não executado | Largura de texto, alinhamento e escalabilidade |
| Desktop observado 1348 px | Home verificada | Um main, scrollWidth=clientWidth; screenshots dos módulos |

Em cada viewport, testar tema claro/escuro, teclado, foco, Escape/fechamento do menu, zoom de texto e conteúdo longo. Os screenshots deste manual são desktop, não evidências mobile/tablet.

## Critério de aceite

Uma página visitada não significa que cada botão foi testado. A conclusão integral do QA depende de executar as mutações, cenários de falha, papéis admin/A/B e viewports acima, anexar evidências e atualizar status. Esta entrega fornece o manual e a trilha de auditoria parcial comprovada; não um selo de aprovação E2E completa.
