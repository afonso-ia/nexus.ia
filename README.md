# NEXUS AI — Deploy Guide

## Estrutura do projeto

```
nexus-ai/
├── index.html        ← Frontend completo (Binance WebSocket + UI)
├── api/
│   └── chat.js       ← Vercel Function (proxy seguro para Anthropic)
├── vercel.json       ← Configuração Vercel
├── .env.example      ← Template de variáveis de ambiente
├── .gitignore        ← Protege sua API key
└── README.md
```

---

## Deploy em 5 minutos

### 1. Crie um repositório no GitHub
- Acesse github.com → New repository → nome: `nexus-ai`
- Faça upload de todos os arquivos desta pasta

### 2. Conecte ao Vercel
- Acesse vercel.com → Add New Project
- Importe o repositório `nexus-ai` do GitHub
- Framework Preset: **Other**
- Clique em Deploy (vai falhar, é normal — ainda falta a API key)

### 3. Configure a variável de ambiente
- No painel Vercel → seu projeto → Settings → Environment Variables
- Adicione:
  - **Name:** `ANTHROPIC_API_KEY`
  - **Value:** `sk-ant-sua-chave-aqui`
- Clique em Save

### 4. Redeploy
- Vercel → seu projeto → Deployments → clique nos 3 pontinhos → Redeploy
- Aguarde ~30 segundos

### 5. Acesse sua URL
- Vercel fornece uma URL tipo: `https://nexus-ai-seu-nome.vercel.app`
- O sistema conectará automaticamente à Binance WebSocket

---

## Como funciona a segurança

```
Browser → index.html (frontend, sem API key)
    ↓
/api/chat (Vercel Function serverless)
    ↓
Anthropic API (com API key segura no servidor)
```

A API key NUNCA vai para o browser. Fica apenas nas variáveis de ambiente do Vercel.

---

## Dados em tempo real (Binance)

O frontend conecta diretamente ao WebSocket da Binance:
- `btcusdt@kline_1m` — candles 1 minuto
- `btcusdt@trade` — trades para cálculo de fluxo
- `btcusdt@depth20@100ms` — order book ao vivo
- `btcusdt@miniTicker` + outros pares — ticker strip

**Zero custo, sem API key, latência ~50-100ms.**

---

## Para adicionar autenticação/monetização no futuro

### Opção simples (token por compra):
1. Gere um token único por venda (ex: UUID)
2. Salve os tokens válidos no banco (ex: Vercel KV, PlanetScale)
3. No `api/chat.js`, valide o token antes de chamar a Anthropic API

### Opção SaaS (Stripe):
1. Integre Stripe Checkout para assinaturas
2. Use Stripe webhooks para ativar/desativar usuários
3. Adicione login com NextAuth.js ou Clerk

---

## Custos estimados

| Serviço | Plano | Custo |
|---------|-------|-------|
| Vercel | Hobby | Grátis |
| Binance WebSocket | — | Grátis |
| Anthropic API | Pay-per-use | ~$0.003/mensagem |

Para 100 usuários com 20 perguntas/dia: ~$6/mês na Anthropic.
