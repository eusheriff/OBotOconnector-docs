# 📘 System Handbook V2: Agent Harness

Este documento consolida todo o conhecimento, arquitetura e processos operacionais do Agent-Hub V2. Use este guia para manter, estender ou recuperar o projeto.

## 🏗️ Arquitetura (The Big Picture)

O **Agent-Hub V2** é um sistema operacional para agentes autônomos baseado em Cloudflare Workers e Durable Objects.

### Diagrama Lógico
```mermaid
graph TD
    Client[WhatsApp/Frontend] -->|HTTP POST| Gateway[Worker Gateway]
    Gateway -->|/v1/hub/orchestrate| Router{Prod Router}
    
    subgraph "Agent Harness V2 (Durable Objects)"
        Router -->|Session ID| Kernel[Agent Kernel (DO)]
        Kernel --> Memory[(Rolling Memory)]
        Kernel -->|ReAct Loop| Tools[Tool Registry]
    end
    
    subgraph "Capabilities"
        Tools -->|search_knowledge| Vectorize[(Knowledge Base)]
        Tools -->|AI Embeddings| CloudflareAI
        Tools -->|list_products| D1[(SQL Database)]
        Tools -->|check_competitors| ML_API[Mercado Livre]
        Tools -->|analyze_image| Vision[Gemini Vision]
    end
```

## 🌐 Topologia do Ecossistema (Manú Executive Gateway)

Manú atua como o ponto de entrada único para todo o grupo **OConnector Technology**.

### Funções de Gateway
1.  **Recrutamento (HR):**
    *   **Trigger:** Mensagens sobre vagas/emprego (Intent Detection via Prompt).
    *   **Ação:** Coleta nome/telefone via chat, salva em `candidates` table (D1) e notifica via WhatsApp (+17813195478).
    *   **Anti-Impersonation:** Para recrutadores oferencendo vagas ao Rodrigo, a Manú entra em modo "Recepção": Agradece e encaminha a mensagem sem recusar ou aceitar.
2.  **Roteamento Multi-Tenant:**
    *   Ela identifica de qual negócio o cliente está falando (ou de onde veio o lead) e despacha para o CRM correto.

| Domínio de Origem / Intenção | Destino do Lead (CRM) | Perfil da Manú |
| :--- | :--- | :--- |
| **WhatsApp Geral** | Padrão: `oconnector.tech` ou Decisão via AI | Secretária Executiva Geral |
| `opos.oconnector.tech` | CRM oPOS | Especialista em PDV |
| `oinbox.oconnector.tech` | CRM oInbox | Especialista Omnichannel |
| `obot.oconnector.tech` | CRM oBot | Engenheira de Bots |
| `sell.oconnector.tech` | CRM Sell | Consultora de Vendas |
| `obrain.oconnector.tech` | CRM oBrain | Data Scientist |
| `salveplate.oconnector.tech` | CRM SalvePlate | Sustainability Specialist |
| `oinbox.oconnector.tech` | CRM oInbox | Omnichannel Specialist |
| `tecnopubli.pt` | CRM TecnoPubli | Marketing Specialist |
| **Recruiters/Business** | Head of Tech (Rodrigo) | Receptionist Mode (Forward Only) |

### Novas Ferramentas Consolidadas (v2.1)
*   **generate_copy:** Cria textos de venda (Llama 3).
*   **check_competitors:** Verifica preços reais no Mercado Livre.
*   **analyze_image:** Vê e valida produtos via Gemini Vision.

---

## 🗺️ Mapa de Rotas (Endpoints)

### Produção
*   **POST `/v1/hub/orchestrate`**: Principal ponto de entrada. Recebe `{ request: string, userId: string }`. Retorna `{ meta: ..., result: { response: ... } }`.
    *   *Nota:* Migrado para usar V2 em 16/01/2026.

### Ferramentas de Auditoria
*   **POST `/v2/test/kernel/:id/run`**: Executa prompt direto no Kernel (bypass de router).
*   **GET `/state/dump`** (no DO): Retorna toda a memória atual.

---

## 🛠️ Procedimentos Operacionais (How-To)

### 1. Teste de Regressão (Pós-Deploy)
Sempre que fizer uma mudança, execute o script de auditoria para garantir que a memória, identidade e tools continuam funcionando.

```bash
# Executa bateria de 6 turnos contra PROD
node scripts/audit_conversation.js
```

**Critério de Sucesso:** Score 5/6 ou 6/6 (Falhas de regex em preços podem ser toleradas se o valor estiver correto).

### 2. Debugging de RAG (Conhecimento)
Se o agente disser "Não sei":
1.  Verifique os logs com `npx wrangler tail`. Procure por `[RAG] Search for "query" found X matches`.
2.  Se X=0: Verifique se o conteúdo foi indexado.
3.  Se Score baixo: Ajuste `minScore` em `src/services/knowledge.ts` (Atual: 0.4).

### 3. Rate Limiting e Custos
*   O sistema bloqueia IPs que excedem 100 reqs/10s.
*   Controlado via KV (`env.AGENTS_KV`).
*   TTL configurado para 60s (Limitação mínima do Cloudflare).

---

## 🧠 Lições Aprendidas (Do Not Repeat)
1.  **NÃO use `eval()`**: Tentamos usar uma calculadora com `eval` e foi um risco de segurança. Use funções nativas.
2.  **Tokens Node:** Workers não têm `node-fetch`. Use o `fetch` nativo global.
3.  **Filtragem RAG:** Cuidado ao filtrar por `agentId` no Vectorize se os documentos não tiverem esse metadado estrito.
4.  **Memória Infinita:** O Kernel tem "Rolling Context" (20 msgs). Sem isso, o custo de tokens explode e o agente trava.

---

## 🔮 Roadmap Futuro (V3)
*   **Frontend WebSocket:** Substituir polling HTTP por conexão persistente.
*   **Multi-Model:** Permitir escolher entre Gemini/Llama/DeepSeek por task.
*   **Dashboard Visual:** Ver o "cérebro" em tempo real.
