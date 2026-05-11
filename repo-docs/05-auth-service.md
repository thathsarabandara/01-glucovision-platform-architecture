# 🔐 Repo 05 — `auth-service`

> **Status:** Planned · Build First  
> **Flag:** 🔐 Security Critical — SEPARATE  
> **Difficulty:** ⭐⭐  
> **Build Priority:** 3 (Foundation — every service depends on it)

---

## Purpose

The **security perimeter** of the entire GlucoVision platform. Every other service delegates authentication and authorization to this one. It exists as a separate repo because security vulnerabilities have their own lifecycle — a JWT library CVE must be patchable and deployable without touching patient glucose data, food recognition models, or any other service. Independent deployability is a patient safety requirement.

---

## Core Functionality

| Module | What It Does |
|---|---|
| `auth/` | JWT issuance, refresh, and revocation |
| `oauth2/` | OAuth2 authorization code flow (Google, Apple) |
| `rbac/` | Role-Based Access Control: patient / clinician / admin / researcher |
| `mfa/` | Multi-Factor Authentication: TOTP (Google Authenticator) + biometric token |
| `token_blacklist/` | Redis-backed revoked token store for instant logout |
| `session/` | Session management for web dashboard |

---

## Inputs

| Input | Source | Format |
|---|---|---|
| Login credentials | Mobile app (03), Web dashboard (04) | JSON (email + password) |
| OAuth2 authorization code | Google / Apple IdP | OAuth2 redirect |
| Refresh tokens | Client apps | JSON |
| TOTP codes | Authenticator app | 6-digit string |
| Token validation requests | All backend services (via gateway 08) | JWT Bearer header |

---

## Outputs

| Output | Consumer | Format |
|---|---|---|
| Access JWT (short-lived) | All clients + services | JWT string |
| Refresh token (long-lived, httpOnly) | Client browser / app | Secure cookie / JSON |
| User ID + role claims | API gateway (08), user service (06) | JWT payload |
| Token validation response | All backend services | JSON boolean + claims |
| RBAC decision | API gateway (08) | JSON |

---

## Dependencies

| Service / Tool | Role |
|---|---|
| PostgreSQL | User credential store (hashed passwords, OAuth links) |
| Redis | Token blacklist (revoked JWTs), session state |
| `06` user-service | Fetch user profile on login (role assignment) |
| Google / Apple OAuth | External IdP for social login |
| SMTP / SendGrid | Email verification, password reset |

---

## Communication

| Protocol | Used For |
|---|---|
| REST / HTTPS | Login, refresh, logout, validate endpoints |
| Redis pub/sub | Token revocation broadcast to gateway |
| Internal HTTP (service mesh) | Token validation calls from other services |

---

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | FastAPI (Python) |
| JWT | python-jose / PyJWT |
| Password hashing | bcrypt (passlib) |
| OAuth2 | authlib |
| TOTP | pyotp |
| Token store | Redis (redis-py) |
| Database | PostgreSQL (SQLAlchemy + Alembic) |
| Containerization | Docker |

---

## AI/ML Knowledge

> Not applicable — pure security infrastructure.

---

## Data Knowledge

| Storage | Purpose | Format |
|---|---|---|
| PostgreSQL | Users table: email, hashed_password, role, oauth_provider, mfa_secret | Relational |
| Redis | JWT blacklist (key: token_jti, TTL: token expiry) | Key-Value with TTL |

**Critical schema note:**
- Passwords: bcrypt hash only — never store plaintext
- MFA secret: AES-encrypted at rest
- JWT payload: `{ sub: user_id, role: "patient|clinician|admin|researcher", exp, jti }`

---

## Infrastructure Knowledge

| Concern | Detail |
|---|---|
| Deployment | Docker container, exposed only through gateway (08) — never directly |
| Scaling | Stateless (Redis holds state) → horizontal scale ready |
| Uptime SLA | Must be 99.9% — auth down = platform down |
| Independent rollback | Can rollback to previous image without touching any other service |

---

## Security Knowledge

| Concern | Detail |
|---|---|
| JWT signing | RS256 (asymmetric) — private key signs, all services validate with public key |
| Token expiry | Access token: 15 min; Refresh token: 7 days |
| Token revocation | jti (JWT ID) stored in Redis blacklist on logout |
| RBAC enforcement | Role claim in JWT; gateway enforces route-level rules |
| Password policy | Minimum 12 chars, bcrypt cost factor 12 |
| Rate limiting | Login: 5 attempts / 15 min / IP (gateway rate limit) |
| MFA | TOTP required for clinician + admin roles |
| Secrets | Private key in HashiCorp Vault; never in env file |

---

## Performance Knowledge

| Concern | Detail |
|---|---|
| Token validation | Stateless JWT validation (no DB hit on every request) |
| Redis lookups | Sub-millisecond blacklist check |
| OAuth2 redirect latency | Cached IdP public keys (JWKS) |
| Horizontal scaling | Stateless — run multiple instances behind gateway |

---

## Testing Knowledge

| Test Type | Approach |
|---|---|
| Unit Tests | Token generation, RBAC logic, password hashing |
| Integration Tests | Full login → token → refresh → logout → blacklist flow |
| Security Tests | Expired token rejection, tampered JWT rejection, brute-force lockout |
| Penetration Test | OWASP API Security Top 10 checks |

---

## Observability

| Concern | Detail |
|---|---|
| Logging | Every auth event: login success/fail, token refresh, logout (with user_id, IP, timestamp) |
| Metrics | Login rate, failed attempts, token issuance rate |
| Alerts | Spike in failed logins → possible brute-force attack |
| Tracing | OpenTelemetry spans for auth request lifecycle |

---

## Research Complexity

| Level | Justification |
|---|---|
| 🟡 Intermediate | Standard auth patterns but requires rigorous security discipline; RBAC + MFA design has depth |

---

## Portfolio Value

| Skill Demonstrated |
|---|
| Production-grade JWT auth service with RS256 |
| OAuth2 social login integration |
| TOTP MFA implementation |
| Redis-backed stateless token revocation |
| RBAC design for multi-role medical platform |
| Security-first engineering discipline |
| Independent deployability for security patching |
