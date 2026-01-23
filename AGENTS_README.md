# 🤖 OBot & OBrain Agents Documentation

Bem-vindo ao catálogo oficial dos Agentes Inteligentes do ecossistema OBot/OBrain.
Aqui você encontra a definição, propósito e status de cada bot.

## 📚 Catálogo de Agentes (10 Bots)

| ID | Nome | Função Principal | Stack | Status |
| :--- | :--- | :--- | :--- | :--- |
| **hub-bot** | **Manú (Orquestrador)** | Atendimento N1, triagem e roteamento de intenções. | Cloudflare Workers (Hono) | ✅ Prod |
| **copy-master** | **CopyMaster** | Gera descrições persuasivas para produtos. | Cloudflare Workers | ✅ Prod |
| **trend-spotter** | **TrendSpotter** | Analisa tendências de mercado (Google Trends/Twitter). | Cloudflare Workers | ✅ Prod |
| **image-validator** | **ImageValidator** | Valida qualidade e compliance de imagens de produtos. | Cloudflare Workers | ✅ Prod |
| **jarbs** | **Jarbs (Guru)** | Consultor estratégico de e-commerce e negócios. | Cloudflare Workers | ✅ Prod |
| **quality-gate** | **QualityGate** | Validação final de anúncios antes de publicar (antigo Sherlock). | Cloudflare Workers | ✅ Prod |
| **supplier-sync** | **SupplierSync** | Sincroniza estoque e preços com fornecedores (CJ, Ali). | Cloudflare Workers | ✅ Prod |
| **stock-alert** | **StockAlert** | Monitora níveis críticos de estoque e avisa admins. | Cloudflare Workers | ✅ Prod |
| **price-watch** | **PriceWatch** | Monitora preços da concorrência (Mercado Livre/Shopee). | Cloudflare Workers | ✅ Prod |
| **order-bot** | **OrderBot** | Automação de status de pedidos e pós-venda. | Cloudflare Workers | ✅ Prod |

---

## 🛠 Como Integrar

Os agentes são expostos como microsserviços via **Agent Hub Gateway**.

### Endpoint Base
\`https://api.o-bot.app\` (ou localhost:8787)

### Exemplo de Chamada (Universal)
Todos os agentes seguem a interface `v1/agent/execute`:

```bash
POST /v1/agent/execute
Content-Type: application/json
Authorization: Bearer <SEU_TOKEN>

{
  "agentId": "copy-master",
  "task": "generate-description",
  "payload": {
    "productName": "Tênis Nike Air",
    "features": ["Confortável", "Esportivo"]
  }
}
```

---

## 📂 Estrutura do Código

*   **`src/agents/`**: Implementação da lógica de cada bot.
*   **`src/routes/`**: Definição dos endpoints REST.
*   **`src/services/`**: Integrações externas (LLM Gateway, Evolution API, Bancos).
*   **`docs/`**: Documentação técnica detalhada.

## 🚀 Deploy e Logs

Para monitorar os logs em tempo real:
```bash
npx wrangler tail
```

Para deployar alterações:
```bash
npm run deploy
```
