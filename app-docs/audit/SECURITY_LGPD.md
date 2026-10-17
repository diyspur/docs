# Auditoria de Segurança e Conformidade LGPD — DIYSPUR

**Data:** 11 de outubro de 2026  
**Marco Regulatório:** Lei Geral de Proteção de Dados (Lei nº 13.709/2018)

---

## 1. Inventário de Dados por Categoria

| Categoria de Dado | Finalidade | Base Legal | Retenção | Exportável | Excluível |
|---|---|---|---|---|---|
| **Credenciais / Auth** | Autenticação e controle de sessão | Execução de contrato | Enquanto a conta estiver ativa | ID / Perfil | Sim |
| **Perfil (`profiles`)** | Exibição de nome e bio na comunidade | Consentimento / Contrato | Enquanto a conta estiver ativa | Sim | Sim (anonimização) |
| **Diário (`journal_entries`)** | Notas reflexivas privadas de leitura | Consentimento | Enquanto o leitor mantiver os registros | Sim | Sim |
| **Progresso (`user_progress`)** | Porcentagem lida em cada capítulo | Execução de contrato | Enquanto a conta estiver ativa | Sim | Sim |
| **RSVP (`meeting_rsvps`)** | Confirmação de presença em debates | Legítimo interesse / Contrato | Até encerramento do ciclo do encontro | Sim | Sim |
| **Listas (`reader_lists`)** | Organização pessoal de livros | Consentimento | Enquanto a conta estiver ativa | Sim | Sim |
| **Logs de Auditoria** | Segurança defensiva contra invasões | Cumprimento de obrigação legal | 6 meses (Marco Civil da Internet) | Não (segurança) | Após prazo legal |

---

## 2. Direitos do Titular (Art. 18 da LGPD)

### 2.1 Direito à Portabilidade e Acesso (Exportação de Dados)
Implementado no endpoint seguro `GET /api/account/export`:
- **Autenticação Rigorosa**: Utiliza `getClaims()` da sessão Supabase no servidor;
- **Sem Parâmetros Injetáveis**: O identificador do usuário é extraído diretamente do token JWT verificado (`sub`), impossibilitando enumeração IDOR;
- **Filtragem Estrita**: Consultas filtradas por `user_id = userId`;
- **Cabeçalhos de Download**: `Content-Disposition: attachment; filename="diyspur-data-export.json"` e `Cache-Control: private, no-store, max-age=0` impedem cache intermediário.

### 2.2 Direito à Exclusão e Anonimização
- Ao solicitar o encerramento da conta, os dados pessoais (nome, biografia, notas privadas do diário) devem ser excluídos ou dissociados.
- Comentários públicos em debates podem ser anonimizados (atribuídos a "Leitor Excluído") para preservar a coerência das discussões da comunidade sem reter dados identificáveis.

---

## 3. Arquitetura de Defesa e RLS (Row Level Security)

1. **Row Level Security Habilitado**: Todas as tabelas no schema `public` possuem RLS ativado.
2. **Políticas de Leitura / Escrita**:
   - `journal_entries`: Apenas o titular (`user_id = auth.uid()`) pode ler, inserir, atualizar ou excluir;
   - `profiles`: PII (e-mail, timestamps de consentimento) é protegida contra leitura pública por visitantes anônimos;
   - `user_clubs`: Clubes arquivados são filtrados em consultas públicas;
   - `chapters`: Capítulos não publicados ou em rascunho são ocultados até o limiar temporal configurado.
3. **Funções RPC em Modo `SECURITY DEFINER`**: Funções privilegiadas sempre fixam `SET search_path = ''` para evitar ataques de hijacking de schema.
