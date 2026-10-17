# Auditoria de Performance e Otimização Web (WPO) — DIYSPUR

**Data:** 11 de outubro de 2026  
**Stack de Otimização:** Next.js 16.4 (Turbopack), React 19.3, Vercel Edge Network

---

## 1. Métricas de Performance e Core Web Vitals (CWV)

As métricas referenciadas são as validadas no Relatório de Auditoria Técnica:

| Métrica | Objetivo (Google CWV) | Resultado Desktop | Resultado Mobile | Diagnóstico |
|---|---|---|---|---|
| **LCP (Largest Contentful Paint)** | ≤ 2.5s | 1.8s | 2.4s | BOM (Capas com otimização e priority onde aplicável) |
| **INP (Interaction to Next Paint)** | ≤ 200ms | 45ms | 85ms | EXCELENTE (React 19 Server Components reduzem JS do cliente) |
| **CLS (Cumulative Layout Shift)** | ≤ 0.1 | 0.02 | 0.04 | EXCELENTE (Dimensões explícitas `width`/`height` em imagens) |
| **FCP (First Contentful Paint)** | ≤ 1.8s | 1.1s | 1.6s | BOM (HTML inicial pré-renderizado no servidor) |
| **TTFB (Time to First Byte)** | ≤ 800ms | 320ms | 480ms | BOM (Vercel Edge Network e streaming de dados) |

---

## 2. Otimização de Imagens (Pipeline Next.js)

1. **Remoção de `unoptimized` na Ficha**: O atributo `unoptimized` na imagem de capa de `/livros/[slug]` foi removido. A imagem agora é processada pelo Image Optimization da Vercel, entregando formatos AVIF/WebP redimensionados para a tela do dispositivo.
2. **Dimensões e Proporção Fixas**: `BookCard` e a página de detalhes declaram `width` e `height` nas tags `Image`, garantindo reserva de espaço no layout antes do carregamento do arquivo e prevenindo CLS.
3. **Allowlist Segura de Hostnames**: O wildcard inseguro `**` foi eliminado de `next.config.mjs`, permitindo exclusivamente domínios de mídia oficiais (Supabase Storage, Editora Record e Amazon).

---

## 3. Prevenção de FOUC (Flash of Unstyled Content)

- O script de inicialização de tema (`themeInitScript`) é inserido diretamente no `<head>` do documento em `src/app/layout.tsx`.
- Esse script lê o `localStorage` e a preferência do sistema operacional (`prefers-color-scheme`) e define o atributo `data-theme` no `<html>` antes da primeira pintura da página, eliminando qualquer cintilação ou troca súbita de cor.

---

## 4. Plano de Coleta de Field Data (RUM)

- **PostHog Performance Plugin**: Monitoramento da distribuição real de latência de rede e tempo de carregamento entre os usuários com consentimento.
- **Sentry Performance Monitoring**: Rastreamento de transações de Server Components e tempo de execução de consultas de catálogo no Supabase.
