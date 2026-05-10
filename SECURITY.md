# Security Policy

## Supported Versions

| Component | Version | Status |
| :--- | :--- | :--- |
| Intent Bus Server | v7.5+ | ✅ Supported |
| Python SDK (`intent-bus`) | v1.2.0+ | ✅ Supported |

---

## Security Model Overview

Intent Bus uses a **Dual-Auth Model** to balance simplicity and security.

### 1. Standard Authentication

Requires:

```http
X-API-KEY: <key>
```

Used over HTTPS, this protects against passive network interception.

#### Limitation

Standard authentication does **not** provide replay protection. Captured requests may be replayed by an active attacker.

---

### 2. Strict Authentication (HMAC)

Each request includes:

- Timestamp
- Nonce
- HMAC-SHA256 signature

Provides:

- Replay protection
- Payload integrity
- Request authenticity

#### Recommendation

Use Strict Auth in all production environments.

Enable globally with:

```bash
BUS_REQUIRE_SIGNATURES=true
```

---

## Server Operations

### Admin & Dashboard Access

Admin endpoints (`/admin/*`) and the dashboard use a separate privileged authentication layer.

Supported methods:

1. Header authentication:

```http
X-Admin-Token: <BUS_ADMIN_SECRET>
```

If `BUS_ADMIN_SECRET` is unset, the server falls back to accepting `BUS_SECRET`.

2. HTTP Basic authentication:

```text
Username: admin
Password: <DASHBOARD_PASSWORD>
```

---

### Reverse Proxy & HTTPS

The server can strictly enforce HTTPS in production:

```bash
BUS_ENFORCE_HTTPS=true
```

If deploying behind Nginx, Apache, Traefik, or another reverse proxy:

- Ensure `X-Forwarded-Proto` is forwarded correctly
- Set:

```bash
BUS_TRUST_PROXY=true
```

This enables Werkzeug `ProxyFix` support so the server correctly detects secure requests and client IP addresses.

---

## Threat Model (High-Level)

### Mitigated

- Replay attacks (Strict Auth only)
- Concurrent claim race conditions (SQLite transactional locking)
- Infinite retry loops (configurable max attempts)
- Cross-tenant access (API key + namespace isolation)
- Basic denial-of-service abuse (rate limits and payload caps)

### Not Mitigated

- Malicious or unsafe worker execution
- Compromised API keys
- Host/VPS compromise
- Side-channel attacks

---

## Reporting a Vulnerability

**Do not open public GitHub issues for security vulnerabilities.**

Report privately via:

- **Email:** dsecurity49@gmail.com

---

### Include in Your Report

- Clear description of the issue
- Step-by-step reproduction
- Proof of concept (if available)
- Impact assessment

---

### Response Policy

- **Acknowledgement:** within 48 hours
- **Initial triage:** within 3–5 days
- **Fix timeline:** depends on severity

Valid reports may receive:

- Credit in release notes (optional)
- Fast-tracked fixes

---

## Security Best Practices

When using Intent Bus:

- NEVER expose API keys in client-side code
- ALWAYS use HTTPS
- USE Strict Auth in production
- ROTATE API keys periodically
- AVOID storing sensitive data in payloads

---

## Known Limitations

### 1. Payload Exposure

Payloads are **not encrypted at rest** within the SQLite database.

Public intents (`visibility="public"`) may be claimed by any authenticated worker in the same namespace.

#### Do NOT include:

- API keys
- Passwords
- PII
- Secrets of any kind

---

### 2. Data Retention

- Intents are ephemeral
- Open intents expire via TTL (default: 24 hours)
- Fulfilled intents and dead letters are deleted after 7 days during cleanup
- KV store values expire at their configured TTL

---

### 3. Hardcoded Limits (Anti-DoS)

The following limits are enforced to protect the single-file SQLite architecture:

| Limit | Value |
| --- | --- |
| Payload Size | 8KB maximum |
| Rate Limit | 60 requests / minute |
| Open Intent Cap | 100 open intents per publisher key |

Admin keys bypass rate limits and open-intent caps.

---

### 4. Concurrency Constraints

SQLite operates using a **single-writer WAL model**.

Under high load:

- Increased latency may occur
- Requests may temporarily fail with:

```http
503 Service Unavailable
```

Clients SHOULD implement retries with backoff.

---

### 5. Replay Protection Scope

Replay protection is enforced **only** when using Strict Auth.

Standard authentication is replayable by design.

---

## Out of Scope

The following are NOT considered vulnerabilities:

- Denial-of-service using valid requests
- Worker-side execution bugs
- Unsafe user code (e.g. `eval`)
- API key misuse by authorized users
- Expected retry behavior
- SQLite `database is locked` errors under heavy concurrency

---

## Disclosure Policy

- Fixes are released before public disclosure
- Critical patches may be shipped without advance notice
- Changelogs include relevant security notes when applicable

---

## Contact

For security concerns:

- **Email:** dsecurity49@gmail.com
- **Discord:** https://discord.gg/bzAneAQzGX

For non-sensitive communication, DM `dsecurity`.

---

## License

MIT
