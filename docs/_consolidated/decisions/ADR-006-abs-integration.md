# ADR-006: Integração ABS Kernel e Memória Persistente

**Data:** 2026-01-22
**Status:** Aceito

## Contexto
A agente Manú (`agent-hub`) operava com memória volátil (Durable Object RAM) e janela de contexto limitada a 20 mensagens. Além disso, as regras de negócio estavam hardcoded no código ("regras implícitas"), dificultando a governança e auditoria centralizada.

## Decisão
1. **Memória de Longo Prazo (LTM):** Implementar persistência em **D1 (SQLite)** com tabelas `conversation_memory` e `user_facts`, TTL de 90 dias e busca semântica via **Vectorize**.
2. **Governança ABS (ABS Governance):** Integrar com **ABS Kernel (`abscore.app`)** via HTTP Connector.
   - Todo tool call é interceptado por `ABSConnector.evaluate()`.
   - Políticas definias no Kernel (ou fallback local).
   - Auditoria centralizada no Kernel (WAL).
3. **Infrastructure:** ABS Kernel deployed como serviço independente (Worker).

## Consequências
- **Positivas:**
  - Manú agora lembra de fatos e conversas passadas (melhor UX).
  - Vendas e ações críticas são auditadas externamente.
  - Separação clara entre "Cérebro" (LLM/Hub) e "Lei" (ABS Kernel).
- **Negativas/Riscos:**
  - Latência extra (>100ms) por tool call para consulta HTTP ao Kernel.
  - Dependência do serviço `api.abscore.app` (fallback local implementado).

## Status de Implementação
- [x] Schema D1 e LTM Service
- [x] ABS Connector
- [x] Deploy ABS Kernel
- [x] Configuração de Secrets (`ABS_API_KEY`)
