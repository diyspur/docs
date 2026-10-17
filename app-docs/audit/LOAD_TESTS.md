# Plano e Relatório de Testes de Carga com k6 — DIYSPUR

**Data:** 11 de outubro de 2026  
**Ambiente Autorizado:** Staging / Homologação (`https://staging.diyspur.app`)  
**Regra Estrita de Segurança:** Testes de carga são **bloqueados em produção** pelo módulo `app/tests/load/guard.js`.

---

## 1. Perfis de Teste de Carga Definidos

### 1.1 Smoke Test (`tests/load/smoke.js`)
- **Objetivo**: Verificar se a infraestrutura responde com integridade sob tráfego leve contínuo.
- **Configuração**: 5 Usuários Virtuais (VUs) durante 1 minuto.
- **Rotas testadas**: `/`, `/livros`, `/clubes`, `/calendario`.
- **Thresholds (SLO)**:
  - `p(95) < 800ms`
  - Taxa de falha de requisições < 1%.

### 1.2 Spike Test (`tests/load/spike.js`)
- **Objetivo**: Simular pico repentino de tráfego (ex.: disparo de e-mail de debate ou abertura de nova temporada).
- **Configuração**: Rampa rápida de 10 para 150 VUs em 30s, mantendo 150 VUs por 20s e descida para 0 em 20s.
- **Thresholds (SLO)**:
  - `p(95) < 1200ms`
  - Taxa de falha de requisições < 2%.

### 1.3 Stress Test (`tests/load/stress.js`)
- **Objetivo**: Avaliar os limites de saturação do pool de conexões do Supabase e do cache de borda da Vercel.
- **Configuração**: Rampa gradual: 25 VUs (1m) ──▶ 75 VUs (2m) ──▶ 150 VUs (2m) ──▶ 300 VUs (2m) ──▶ 0 VUs (1m).
- **Thresholds (SLO)**:
  - `p(95) < 1500ms`
  - Taxa de falha de requisições < 2%.

---

## 2. Guard de Segurança e Bloqueio de Produção

O arquivo `tests/load/guard.js` implementa a função `assertApprovedStaging()` que valida:
1. `BASE_URL` definido e apontando para ambiente com protocolo HTTPS;
2. Bloqueio expresso contra hostnames de produção: `diyspur.vercel.app`, `diyspur.com`, `www.diyspur.com`;
3. Exigência da flag de confirmação humana: `LOAD_TEST_APPROVED=yes`.

### Exemplo de Execução Segura em Staging
```bash
LOAD_TEST_APPROVED=yes BASE_URL=https://staging.diyspur.app k6 run tests/load/smoke.js
```
