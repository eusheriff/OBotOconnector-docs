# ADR 001: Migration to Google Cloud Free Tier with Cloudflare Tunnel

## Status

Accepted

## Context

The previous infrastructure on Oracle Cloud became unavailable or was marked for decommissioning. A cost-effective replacement was required to host the Evolution API (WhatsApp Integration) for the Agent Hub.
The target environment is Google Cloud's Free Tier (`e2-micro`), which has significant resource constraints (2 vCPUs, 1GB RAM) and strict VPC Firewall rules.

## Decision

1. **Infrastructure:** Migrate Evolution API to Google Cloud `e2-micro` instance (`130.211.119.131`).
2. **Access Layer:** Use **Cloudflare Tunnel** (`cloudflared`) to expose the local service (port 8080) to the public internet (`trycloudflare.com` or custom domain).
3. **Resource Management:**
   - Enable **6GB Swap** to compensate for the 1GB physical RAM limit.
   - Tune Postgres and Redis for low memory consumption.
   - Run **only** the Evolution API stack on this VPS (Single Responsibility).

## Consequences

### Positive

- **Cost:** Zero (within Free Tier limits).
- **Security:** No need to open inbound ports (80/443/8080) on the VPC Firewall; Tunnel handles secure egress.
- **HTTPS:** Managed automatically by Cloudflare.

### Negative

- **Performance:** 1GB RAM is borderline for Puppeteer (Chrome), causing slow or failed QR Code generation.
- **Complexity:** Requires `cloudflared` daemon maintenance.
- **Latency:** Extra hop via Cloudflare Tunnel.

## Compliance

- **Backup:** Previous data (Oracle) was lost; system reset to fresh state.
- **Security:** API Key rotated and `.env` updated.
