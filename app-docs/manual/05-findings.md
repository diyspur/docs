# Achados de QA/UX e revisão antes de publicar

Achados abaixo distinguem evidência de navegador, leitura de código e sugestões. Não foram corrigidos nesta entrega documental. Não usamos severidade crítica para cenários que não foram reproduzidos.

| ID | Natureza / prioridade | Evidência | Efeito e orientação | Fonte |
|---|---|---|---|---|
| F-01 | UX/conteúdo, média | Home autenticada 3/3; após logout 1/1; timeline anônima três etapas | Contador usa capítulos visíveis como denominador e pode sugerir ciclo completo. Calcular total autorizado de metadados de forma consistente, separar “acessíveis para você” de “etapas do ciclo”. | src/features/catalog/queries.ts getActiveSeason; src/app/page.tsx |
| F-02 | Editorial, média | Comunidade e clubes com texto QA RETESTE/QA AUDITORIA de 10/10 | Conteúdo sintético polui experiência pública. Revisar com responsáveis e remover/arquivar pelos fluxos autorizados; produzir fixtures em staging para novos testes. Não removido nesta rodada. | /comunidade; /clubes; images/comunidade.jpg |
| F-03 | Operação, média | Clube Verity com 2 participantes e painel sem ciclo | Nome do clube não vincula automaticamente temporada. Responsável deve definir a temporada na gestão; texto atual explica a ausência. | /clubes/clube-ae595cd137d748619dfcabf7e7848fa0 |
| F-04 | Operação, média | Encontro 14/10 às 20h sem plataforma/link | Publicar instruções de acesso antes do encontro; não documentar um CTA de reunião que ainda não existe. | /calendario |
| F-05 | UX, baixa | Perfil com atividade e mensagem “Quando sua leitura começar…” | Ajustar mensagem de boas-vindas segundo histórico real para não sugerir ausência de atividade. | src/app/perfil/page.tsx; /perfil |
| F-06 | Integridade/contrato, alta; código | Perfil promete cópia integral; export tem limites e domínios ausentes | Paginar export, incluir itens/quiz/votos ou descrever escopo explícito, verificar errors individuais e evitar HTTP 200 com vazio em falha parcial. Não foi baixado/reproduzido caso de falha. | src/app/api/account/export/route.ts; src/app/perfil/page.tsx |
| F-07 | Tratamento de erro, média; código | getBooks/getMeetings retornam [] em erro | Separar falha técnica de zero resultados e oferecer retry. Sem injeção de falha de rede nesta rodada. | src/features/catalog/queries.ts |
| F-08 | Contrato, média; código | Metas HTML 365/50.000/500.000 versus action 1.000/1.000.000/1.000.000 | Centralizar limites e explicar campos opcionais. Não modificar contrato sem revisar intenção do produto. | src/components/goals/goal-form.tsx; src/features/goals/actions.ts |
| F-09 | UX/data, média; código | Diário inicializa dia por toISOString().slice(0,10) | UTC pode adiantar dia no horário noturno de Brasília. Usar data local acordada; testar entre 21h e 23h59 Brasília. Não criado registro nessa janela. | src/components/journal/journal-entry-form.tsx |
| F-10 | Consistência, baixa; código | Boundary usa suporte@diyspur.com; shell/políticas usam oi@diyspur.com | Usar contato canônico no Error Boundary e revisar atendimento do endereço. Falha não forçada no navegador. | src/app/error.tsx; src/lib/site-contact.ts |
| F-11 | Conteúdo técnico, média | README classifica módulos privados; /desafios público e /listas shell público observados | Manter documentação de acesso alinhada ao proxy/página e ao resultado live; este manual registra distinção. | src/lib/supabase/middleware.ts; pages/desafios.md; pages/listas.md |
| F-12 | Capacidade de produto, média | Leituras compartilhadas somente consulta no código | Não ensinar criação/conclusão de checkpoints que não existem nessa UI. Implementação de CRUD exige escopo próprio; preservar consulta atual. | src/app/leituras-compartilhadas/page.tsx |
| F-13 | Auth/contrato, média; código | Magic link afirma “Se cadastrado”, signInWithOtp não passa shouldCreateUser:false | Conferir comportamento do Auth para endereço novo em staging e harmonizar intenção, opção da API e mensagem. Sem disparo de e-mail nesta rodada. | src/components/auth-form.tsx |
| F-14 | Operação editorial, média; código | Painel não lista temporadas; editor de capítulo não edita perguntas | Documentar acesso operacional ao UUID e preparação do quiz; considerar navegação/CRUD editorial futuro. Não supor telas inexistentes. | src/app/admin/page.tsx; src/app/admin/seasons/[id]/page.tsx |

## Sugestões de evolução com validação

Separar “catálogo sem resultado” de “serviço indisponível”; padronizar feedback e retorno pós-login; centralizar limites e contato; disponibilizar contexto completo de denúncias para decisão; associar ciclos de clubes antes de promover a participação; definir rotina editorial para agenda/link e conteúdo público. Cada mudança deve ser validada em staging e em mobile/desktop antes de atualizar as imagens do manual.

## Limites que permanecem

Nenhum achado por código é apresentado como falha de rede reproduzida. Não houve teste de compra, débito, envio de e-mail, admin autorizado, persistência nova ou comparação A/B. Nenhuma declaração de conformidade jurídica, WCAG integral ou aprovação de RLS ponta a ponta resulta desta auditoria.
