# State of the System

**Last Updated:** 2026-01-22 (Docs Updated)

## 🚦 System Status: ONLINE (Bypassed)

The **Agent Hub (V2)** is migrated to **Google Cloud** infrastructure.

- **Access:** Via Cloudflare Tunnel (Bypass Firewall).
- **Core:** Stable.
- **Integrations:**
  - ⚠️ **oInbox:** Evolution API Deploying on Google Cloud.
  - ✅ **TecnoPubli:** Integrated.
  - ✅ **SavePlate:** Integrated.
- **Infrastructure:**
  - **VPS Google (Principal):** 130.211.119.131 (ONLINE).
  - **Evolution API:** v2.3.7 + Postgres + Redis (Low RAM tuning).
  - **Webhook:** `https://webhook.obot.oconnector.tech/webhook` (Configurado ✅ - DNS Propagando ⏳).

## ⚠️ Critical Risks

- **Evolution API (Fresh DB):** All previous WhatsApp sessions were lost. User MUST scan QR Code again.
- **Free Tier Limits:** Google `e2-micro` has 1GB RAM. SWAP Usage is normal.
- **SPOF (Audit):** Single VPS instance for WhatsApp is a critical point of failure (ARCH-02).
- **Quality (Audit):** Zero automated tests and missing runtime validation (CODE-01/03).

## Next Steps

1. **Urgent:** Scan QR Code for `oConnector` to restore service.
2. **Strategy:** Plan "Event-Driven" architecture migration (Cloudflare Queues + Durable Objects).
3. **Stability:** Implement Smoke Tests for critical paths.
