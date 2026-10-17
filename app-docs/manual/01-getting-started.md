# Primeiros passos: conta e primeira leitura

## Conhecer o clube antes de cadastrar

Explore a home e o catálogo sem login. Abra uma ficha e consulte edição, avisos de conteúdo e temporadas. Prepare seu exemplar físico ou digital. Comprar pelo link do clube é opcional. Para continuar uma sequência com quiz, comentar ou organizar leituras, use sua conta.

## Criar conta

1. Clique em Fazer parte ou abra /cadastro.
2. Preencha nome de exibição, usuário com 3–24 letras/números/_ e e-mail válido.
3. Escolha senha com 10–128 caracteres; revise termos e privacidade.
4. Clique em Criar minha conta e aguarde. Não repita enquanto houver envio pendente.
5. Se a tela pedir confirmação, confira a caixa de entrada e spam e abra a mensagem recebida. O fluxo depende da configuração de confirmação de e-mail do Supabase; não foi verificado pela entrega de e-mail nesta rodada.
6. O callback /auth/callback troca o código por sessão e retorna ao perfil ou destino local solicitado. Um link expirado ou já usado leva ao aviso de novo acesso.

O formulário envia display_name/username como metadados de cadastro. Isso não concede acesso administrativo. Não existe wizard obrigatório de preferências após o cadastro no código inspecionado.

![Cadastro](images/anon-cadastro.jpg)

## Entrar e manter o destino

1. Abra Entrar; quando uma página pessoal redirecionar, o parâmetro redirect guarda o destino.
2. Preencha seu e-mail e senha no próprio site e use Entrar na comunidade.
3. Aguarde retorno à página solicitada ou ao perfil. Um clique/envio sozinho não comprova sessão: confirme que a área pessoal carregou.
4. Como alternativa, informe o e-mail e use Receber um link de acesso. Confira a mensagem e siga o link, quando entregue.

Somente destinos locais passam por safeLocalRedirect. Os botões Google/GitHub não aparecem no formulário auditado; não documentamos login social como recurso disponível. O magic link usa signInWithOtp sem shouldCreateUser:false no código; portanto a intenção “apenas endereço cadastrado” da mensagem deve ser conferida com a configuração real do Auth antes de prometer essa restrição.

| Situação | Orientação exibida/implementada |
|---|---|
| E-mail vazio ou inválido | Informe um email válido. — observado ao enviar formulário vazio |
| Credenciais rejeitadas | Email ou senha não conferem. — contrato do componente |
| E-mail não confirmado | Confirme seu email antes de entrar. — contrato |
| Link inválido/expirado | Solicite um novo acesso. — callback/componente |
| Falha de conexão | Confira sua conexão e tente novamente. — contrato |

![Login](images/anon-entrar.jpg)

## Recuperar senha

1. Em Entrar, use Esqueci minha senha.
2. Informe e-mail e use Enviar instruções. A confirmação genérica evita revelar se a conta existe.
3. Confira a caixa de e-mail e abra o link válido; callback encaminha à tela de atualização.
4. Digite a nova senha com pelo menos dez caracteres e confirme igual.
5. Use Atualizar senha e confirme o sucesso antes de considerar a credencial alterada.

A tela de atualização também abre sem sessão, mas auth.updateUser exige sessão válida. Nesta auditoria nenhum e-mail foi disparado e nenhuma senha foi alterada. [Guia de recuperação](pages/recuperar-senha.md) e [atualização](pages/atualizar-senha.md).

## Primeira leitura e organização

1. Abra Livros, escolha a obra e leia os avisos de conteúdo.
2. Abra a temporada e o primeiro capítulo disponível.
3. Leia o trecho no exemplar e acompanhe o vídeo, se desejar.
4. Registre o progresso ou conclua a etapa; participe do quiz para seguir a sequência quando exigido.
5. Ao comentar, marque spoiler. Leia avisos antes de revelar mensagens protegidas.
6. Consulte Calendário para o próximo encontro e confirme presença se quiser participar.
7. Use Diário para anotações privadas, Listas para coleções, Minha fila para até cinco prioridades e Metas para objetivos anuais.

Se a ficha mostrar Esta leitura ainda não tem temporada aberta, não há ciclo para iniciar naquele título. Se o clube mostrar painel sem temporada, aguarde o responsável associar o ciclo. Leituras compartilhadas é atualmente uma tela de consulta de grupos autorizados, sem formulário de criação na própria rota.

## Sair e privacidade

Use Sair no cabeçalho. O fluxo faz POST /auth/signout e retorna à home; nesta rodada, o cabeçalho passou a Entrar/Fazer parte e páginas pessoais bloquearam os dados. A cópia de uma sessão existente não foi usada para entrar em segunda conta.

Analytics é opcional: Preferências de privacidade permite revisar a escolha. Recusar não bloqueou o uso durante a auditoria. Exportação JSON fica no perfil e contém dados próprios: mantenha o arquivo privado. Não confunda o recurso atual, limitado, com exportação integral garantida.
