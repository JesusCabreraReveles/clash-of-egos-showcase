# Security Hardening & Operational Audit

This document outlines the defensive engineering measures, cryptographic validations, and infrastructure security policies implemented in **Clash of Egos**.

---

## 1. Cryptographic Webhook Integrity (Anti-Tampering)

All financial state mutations originating from payment gateways are guarded by constant-time cryptographic verification:

* **Stripe Webhook Signatures:** Incoming payloads must bear a valid `Stripe-Signature` timestamped header. The signature is computed via HMAC-SHA256 using an isolated signing secret (`STRIPE_WEBHOOK_SECRET`).
* **Replay Attack Mitigation:** Payloads older than 300 seconds are rejected automatically to prevent replay of historic transaction events.
* **Idempotency Store:** Every handled webhook event registers its unique provider event ID (`evt_...`) in an indexed table wrapped in the same database transaction as the business mutation. Duplicate event deliveries from network retries are detected and safely acknowledged without re-executing transactions.

---

## 2. Injection Prevention & Strict Typing

* **SQL Injection Immunity:** The application relies on parameterized queries via PostgreSQL prepared statements (`pgx` in Go and strict parameter binding). String interpolation in SQL queries is banned by static linting rules.
* **XSS Sanitization:** User-submitted profile titles, quotes, URLs, and display names are sanitized on ingestion using strict HTML entity escaping and validated against regex patterns for allowable characters and protocol schemas (`https://` required for outbound URLs).
* **Strict Type Safety:** All contracts shared between backend and frontend are strictly typed across the entire pipeline using TypeScript and Go structs.

---

## 3. Network & Transport Security

* **Strict HTTPS Enforcement:** Nginx terminates TLS 1.3 with automated Let's Encrypt certificates, enforcing HTTP Strict Transport Security (`HSTS`) headers:
  ```nginx
  add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload" always;
  add_header X-Content-Type-Options "nosniff" always;
  add_header X-Frame-Options "DENY" always;
  add_header X-XSS-Protection "1; mode=block" always;
  add_header Referrer-Policy "strict-origin-when-cross-origin" always;
  ```
* **Reverse Proxy Isolation:** Internal services (SvelteKit SSR node and PostgreSQL database) bind exclusively to `127.0.0.1` or internal Docker bridge networks, completely unexposed to the public internet.

---

## 4. Rate Limiting & Denial-of-Service Defense

* **Edge Rate Limiting:** Cloudflare DDoS shields front the domain.
* **Application-Level Token Bucket:** Sensitive endpoints (such as `POST /api/v1/bids/intent` and profile lookups) apply per-IP and per-account rate limits to prevent brute-force usurpation spamming or card testing attempts.
