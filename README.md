<div align="center">

# 👑 CLASH OF EGOS
### The Sovereign Executive Arena · Commercial Architecture Showcase

[![Production Status](https://img.shields.io/badge/Production-Live%20at%20clashofegos.lol-f5c026?style=for-the-badge&logo=google-chrome&logoColor=black)](https://clashofegos.lol)
[![Frontend](https://img.shields.io/badge/Frontend-SvelteKit%205%20%7C%20TypeScript-ff3e00?style=for-the-badge&logo=svelte&logoColor=white)](https://svelte.dev)
[![Database](https://img.shields.io/badge/Database-PostgreSQL%2016-336791?style=for-the-badge&logo=postgresql&logoColor=white)](https://www.postgresql.org)
[![Cloud](https://img.shields.io/badge/Infra-AWS%20EC2%20%7C%20Docker-FF9900?style=for-the-badge&logo=amazon-aws&logoColor=white)](https://aws.amazon.com)
[![Payments](https://img.shields.io/badge/Payments-Stripe%20HMAC-635bff?style=for-the-badge&logo=stripe&logoColor=white)](https://stripe.com)
[![License](https://img.shields.io/badge/License-Proprietary%20%2F%20Closed%20Source-red?style=for-the-badge)](./LICENSE)

<br />

**[🌐 Launch Live Arena](https://clashofegos.lol)** · **[👔 View Founder Profile](https://clashofegos.lol/c/jesus-cabrerareveles)** · **[📐 System Architecture](./docs/ARCHITECTURE.md)** · **[🛡️ Security Specifications](./docs/SECURITY_AUDIT.md)**

<br />

<p align="center">
  <img src="./assets/screenshots/01-hero-throne-arena.png" alt="Clash of Egos Sovereign Arena" width="950" style="border-radius: 8px; box-shadow: 0 10px 30px rgba(0,0,0,0.5);" />
</p>

</div>

---

## 📌 Executive Summary

**Clash of Egos** is a high-stakes, real-time social capital platform where startup founders, C-level executives, and venture investors compete for the sovereign digital spotlight through financial liquidity and prestige.

Users place bids via Stripe to usurp the active Monarch, claim the #1 global spotlight, and drive verified high-intent traffic to their LinkedIn profiles and venture websites. The system features dynamic pricing (+10% minimum outbid increments), regional sovereignty arenas (Mexico & United States), real-time financial marquees, and automated verifiable executive credentials.

> **Note on Proprietary Codebase:**  
> This showcase repository provides an architectural, visual, and operational review of the system. In accordance with commercial trade secret protection, the private business logic, database migrations, and production secrets remain strictly hosted in private repositories. All system designs, invariants, and architectural solutions documented here represent the production system operating live at [https://clashofegos.lol](https://clashofegos.lol).

---

## 📸 Platform Showcase & Visual Experience

### 1. The Sovereign Monarch Throne & Real-Time Marquee
The primary arena displays the active monarch, dynamic outbid cost, real-time financial news marquee, and zero-friction challenge interface.

<p align="center">
  <img src="./assets/screenshots/01-hero-throne-arena.png" alt="Hero Throne" width="850" />
</p>

---

### 2. Praetorian Podium & Contender Leaderboard
Ranks #2 and #3 flank the throne with real-time liquidity deficits, while the comprehensive directory organizes corporate challengers across Software & AI, Startups & VCs, Capital & Finance, and Media.

<p align="center">
  <img src="./assets/screenshots/02-podium-and-leaderboard.png" alt="Podium and Leaderboard" width="850" />
</p>

---

### 3. Executive Identity & Social Capital Profile (`/c/{slug}`)
Dynamic executive portfolio pages featuring social verification, total throne holding duration, total capital deployed, and customized badges.

<p align="center">
  <img src="./assets/screenshots/04-executive-profile.png" alt="Executive Profile" width="850" />
</p>

---

### 4. Zero-Friction Usurpation & Stripe Checkout
Instant bid creation with multi-currency support (USD/MXN), automatic tax handling, and instant cryptographic settlement.

<p align="center">
  <img src="./assets/screenshots/05-claim-modal.png" alt="Claim Modal" width="850" />
</p>

---

### 5. Dual-Palette Luxury Styling (Dark / Wall Street White)
Users can toggle between the signature *High-Net-Worth Stealth Dark* theme and the *Wall Street Luxury White* theme with zero Cumulative Layout Shift (CLS).

<p align="center">
  <img src="./assets/screenshots/03-arena-luxury-theme.png" alt="Luxury White Theme" width="850" />
</p>

---

## 🛠️ Technology Stack Breakdown

| Layer | Technologies | Architectural Rationale |
| :--- | :--- | :--- |
| **Frontend Framework** | **SvelteKit 5 (Runes: `$state`, `$derived`, `$effect`)** | Near-zero runtime overhead, fine-grained reactivity, lightning-fast SSR, and superior Core Web Vitals. |
| **Language** | **TypeScript 5.x & Go 1.22+** | End-to-end static typing, zero runtime type errors, and shared data contracts across microservices. |
| **Styling & UI** | **Vanilla Scoped CSS / Design Tokens** | Handcrafted luxury aesthetic with glassmorphism, gold accents, CSS variables, and zero bundle bloat. |
| **Database** | **PostgreSQL 16 with `btree_gist`** | ACID compliance, temporal exclusion constraints to prevent overlapping reigns, and integer minor units for financial precision. |
| **Payments** | **Stripe API + Webhook Engine** | Provider-agnostic payment abstraction with cryptographic HMAC-SHA256 signature verification and idempotent reconciliation. |
| **Reverse Proxy** | **Nginx + Let's Encrypt TLS 1.3** | High-concurrency connection pooling, SSL termination, HTTP/2 streaming, and security headers (HSTS, CSP). |
| **Infrastructure** | **AWS EC2 (Ubuntu 24.04 LTS) + Docker Compose** | Containerized multi-stage builds, isolated network bridges, healthchecks, and zero-downtime rolling deploys. |
| **Internationalization** | **Bilingual Engine (ES / EN)** | Multi-locale dictionary stores with country detection (Mexico & US arenas). |

---

## 🧠 Key Engineering Challenges & Solutions

### 1. Temporal Concurrency: Zero-Collision Throne Usurpation
* **Problem:** In high-velocity bidding windows, multiple contenders can trigger payment checkouts simultaneously. A naive system risks granting duplicate active reigns or overwriting bids.
* **Solution:** Database-level temporal exclusion constraint using PostgreSQL's `btree_gist` extension, coupled with `SERIALIZABLE` transaction isolation:
  ```sql
  ALTER TABLE reigns ADD CONSTRAINT no_overlapping_active_reigns 
  EXCLUDE USING gist (
      season_id WITH =,
      tstzrange(started_at, COALESCE(ended_at, 'infinity'::timestamptz)) WITH &&
  ) WHERE (status = 'valid');
  ```
* Read the complete [Concurrency Specification in ARCHITECTURE.md](./docs/ARCHITECTURE.md).

### 2. Idempotent Financial Webhook Pipeline
* **Problem:** Network retries from Stripe or malicious replay attacks could cause double-bidding or fraudulent throne acquisitions.
* **Solution:** An asynchronous webhook verification pipeline enforcing timestamp expiration (rejecting events > 300 seconds), constant-time HMAC-SHA256 signature matching, and a persistent `processed_events` idempotency table executed in the same database transaction.
* Read the complete [Security Hardening in SECURITY_AUDIT.md](./docs/SECURITY_AUDIT.md).

### 3. Dynamic Viral OpenGraph Card Generation
* **Problem:** High-net-worth users share their victories on LinkedIn and X. Generic link previews hurt viral engagement.
* **Solution:** SvelteKit server hooks dynamically render custom OpenGraph `<meta>` tags for every `/c/{slug}` route, including the user's executive title, avatar, throne duration, and current status.

---

## 🏛️ System Architecture Diagram

```mermaid
graph TD
    User["🌐 User Browser (Desktop / Mobile)"] -->|HTTPS / WSS| CF["☁️ Cloudflare Edge (WAF & DDoS)"]
    CF -->|Reverse Proxy| NGINX["🛡️ Nginx Reverse Proxy (SSL TLS 1.3)"]

    subgraph AWS EC2 Node ["AWS EC2 Production Server"]
        NGINX -->|HTTP :3000| Web["⚡ SvelteKit 5 SSR Engine"]
        NGINX -->|HTTP :8080| API["⚙️ Core API Service"]
        
        API -->|pgx pool / SERIALIZABLE| DB[("🗄️ PostgreSQL 16 DB (Temporal Exclusions)")]
        API -->|Fast State & Rate Limit| Cache[("💾 In-Memory Cache")]
    end

    API -->|HMAC-SHA256 Webhooks| Stripe["💳 Stripe Payment Gateway"]
```

---

## 👨‍💻 Founder & Principal Engineer

**Jesús Cabrera Reveles**  
*CEO & CTO — Clash of Egos*

* 🌐 **Live Platform:** [https://clashofegos.lol](https://clashofegos.lol)
* 👔 **Executive Profile:** [clashofegos.lol/c/jesus-cabrerareveles](https://clashofegos.lol/c/jesus-cabrerareveles)
* 💼 **LinkedIn:** [linkedin.com/in/jesus-cabrerareveles](https://www.linkedin.com/in/jesus-cabrerareveles)
* 🐙 **GitHub:** [@JesusCabreraReveles](https://github.com/JesusCabreraReveles)

---

## 📄 License & Commercial Notice

All rights reserved © 2026 Jesús Cabrera Reveles.  
This repository is published exclusively as an engineering architecture showcase. Unauthorized copying, distribution, or reverse engineering of the production systems and proprietary codebase of **Clash of Egos** is strictly prohibited. See [LICENSE](./LICENSE) for full legal terms.
