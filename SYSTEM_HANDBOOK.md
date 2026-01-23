# 📘 System Handbook V2: Agent Harness

> **Link:** [obot.oconnector.tech](https://obot.oconnector.tech)  
> **Contact:** [dev@oconnector.tech](mailto:dev@oconnector.tech)

This document consolidates all knowledge, architecture, and operational processes of Agent-Hub V2. Use this guide to maintain, extend, or recover the project.

## 🏗️ Architecture (The Big Picture)

**Agent-Hub V2** is an operating system for autonomous agents based on Cloudflare Workers and Durable Objects.

### Logical Diagram
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

## 🌐 Ecosystem Topology (Manú Executive Gateway)

Manú acts as the single entry point for the entire **OConnector Technology** group.

### Gateway Functions
1.  **Recruitment (HR):**
    *   **Trigger:** Job/vacancy related messages (Intent Detection via Prompt).
    *   **Action:** Collects name/phone via chat, saves to `candidates` table (D1) and notifies via WhatsApp (+17813195478).
    *   **Anti-Impersonation:** For recruiters offering jobs to Rodrigo, Manú enters "Reception" mode: Thanks them and forwards the message without accepting/declining.
2.  **Multi-Tenant Routing:**
    *   Identifies which business the client is talking about (or lead source) and dispatches to the correct CRM.

| Source Domain / Intent | Lead Destination (CRM) | Manú Profile |
| :--- | :--- | :--- |
| **General WhatsApp** | Default: `oconnector.tech` or AI decision | General Executive Assistant |
| `opos.oconnector.tech` | CRM oPOS | POS Specialist |
| `oinbox.oconnector.tech` | CRM oInbox | Omnichannel Specialist |
| `obot.oconnector.tech` | CRM oBot | Bot Engineer |
| `sell.oconnector.tech` | CRM Sell | Sales Consultant |
| `obrain.oconnector.tech` | CRM oBrain | Data Scientist |
| **Recruiters/Business** | Head of Tech (Rodrigo) | Receptionist Mode (Forward Only) |

### Consolidated Tools (v2.1)
*   **generate_copy:** Creates sales copy (Llama 3).
*   **analyze_image:** Sees and validates products via Gemini Vision.

---

## 🗺️ Route Map (Endpoints)

### Production
*   **POST `/v1/hub/orchestrate`**: Main entry point. Receives `{ request: string, userId: string }`. Returns `{ meta: ..., result: { response: ... } }`.
    *   *Note:* Migrated to use V2 on 01/16/2026.

### Audit Tools
*   **POST `/v2/test/kernel/:id/run`**: Executes prompt directly on Kernel (bypass router).
*   **GET `/state/dump`** (on DO): Returns entire current memory.

### Client Onboarding (Automation)
*   **POST `/v1/admin/provision`**: Creates User (KV) + WhatsApp Instance (Evolution) + Returns QR Code.
*   **CLI Script**: `scripts/onboard-client.ts` wraps the provision endpoint for valid tokens and easier usage.
    ```bash
    npx ts-node scripts/onboard-client.ts --name="Client" --email="email@domain.com"
    ```


---

## 🛠️ Operational Procedures (How-To)

### 1. Regression Testing (Post-Deploy)
Whenever a change is made, run the audit script to ensure memory, identity, and tools continue to function.

```bash
# Runs 6-turn battery against PROD
node scripts/audit_conversation.js
```

**Success Criteria:** Score 5/6 or 6/6 (Regex failures on prices can be tolerated if the value is correct).

### 2. RAG Debugging (Knowledge)
If the agent says "I don't know":
1.  Check logs with `npx wrangler tail`. Look for `[RAG] Search for "query" found X matches`.
2.  If X=0: Check if content was indexed.
3.  If Score is low: Adjust `minScore` in `src/services/knowledge.ts` (Current: 0.4).

### 3. Rate Limiting and Costs
*   System blocks IPs exceeding 100 reqs/10s.
*   Controlled via KV (`env.AGENTS_KV`).
*   TTL set to 60s (Cloudflare minimum limitation).

---

## 🧠 Lessons Learned (Do Not Repeat)
1.  **DO NOT use `eval()`**: Tried using a calculator with `eval`, it was a security risk. Use native functions.
2.  **Node Tokens:** Workers do not have `node-fetch`. Use global native `fetch`.
3.  **RAG Filtering:** Careful when filtering by `agentId` in Vectorize if docs don't have strict metadata.
4.  **Infinite Memory:** Kernel has "Rolling Context" (20 msgs). Without it, token costs explode and agent crashes.

---

## 🔮 Future Roadmap (V3)
*   **WebSocket Frontend:** Replace HTTP polling with persistent connection.
*   **Multi-Model:** Allow choosing between Gemini/Llama/DeepSeek per task.
*   **Visual Dashboard:** See the "brain" in real-time.
