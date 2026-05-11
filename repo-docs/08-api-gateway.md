# 🌐 Repo 08 — `api-gateway`

> **Status:** Planned  
> **Flag:** —  
> **Difficulty:** ⭐⭐  
> **Build Priority:** 5 (Foundation — clients can't reach backend without it)

---

## Purpose

The **single entry point and observability hub** for the entire GlucoVision platform. Every request from mobile (03), web (04), and IoT devices (17) passes through here before reaching any backend service. The gateway handles routing, authentication forwarding, rate limiting, and provides the central observability stack (metrics, tracing, logs). Merged with observability because monitoring is inherently a gateway concern — you can't separate Prometheus scraping from what routes Traefik manages.

---

## Core Functionality

| Module | What It Does |
|---|---|
| `gateway/` | Traefik reverse proxy — routes requests to the correct service |
| `auth_forward/` | Forwards JWT to `05` auth-service for validation before routing |
| `rate_limiting/` | Per-IP and per-user rate limits on all endpoints |
| `monitoring/` | Prometheus scraping + Grafana dashboards for all 20 services |
| `tracing/` | OpenTelemetry distributed trace collection (Jaeger/Tempo) |
| `compose/` | Master `docker-compose.yml` for full local dev environment |
| `circuit_breaker/` | Traefik circuit breaker for downstream service failures |

---

## Inputs

| Input | Source | Format |
|---|---|---|
| HTTP requests | Mobile (03), Web (04), Edge IoT (17) | REST / WebSocket |
| JWT tokens | All clients | Bearer header |
| Health check probes | Prometheus scraper | HTTP GET /health |
| Trace spans | All backend services (OTLP push) | OpenTelemetry protocol |
| Metrics | All backend services | Prometheus exposition format |

---

## Outputs

| Output | Consumer | Format |
|---|---|---|
| Routed request | Target backend service | Proxied HTTP |
| Auth rejection (401/403) | Client | JSON error |
| Rate limit rejection (429) | Client | JSON error |
| Metrics dashboard | Ops team / researchers | Grafana UI |
| Distributed traces | Debugging engineers | Jaeger / Tempo UI |
| Aggregated logs | Ops team | Loki + Grafana |

---

## Dependencies

| Service / Tool | Role |
|---|---|
| `05` auth-service | Token validation (forward auth) |
| All 20 services | Downstream routing targets |
| Prometheus | Metrics scraping from all services |
| Grafana | Dashboard visualization |
| Loki | Log aggregation |
| Jaeger / Tempo | Distributed trace storage |
| OpenTelemetry Collector | Trace + metric aggregation pipeline |
| Redis | Rate limit counters |

---

## Communication

| Protocol | Used For |
|---|---|
| HTTP/HTTPS | Reverse proxy for REST APIs |
| WebSocket | Proxied for real-time endpoints (`13` alerts, `11` cooking guidance) |
| OTLP (gRPC / HTTP) | OpenTelemetry trace/metric export from services |
| Prometheus pull | Metrics scraping from `/metrics` endpoints |

---

## Tech Stack

| Layer | Technology |
|---|---|
| Reverse Proxy | Traefik v3 |
| Auth Middleware | Traefik ForwardAuth → `05` auth-service |
| Metrics | Prometheus |
| Dashboards | Grafana |
| Tracing | OpenTelemetry Collector + Jaeger / Tempo |
| Log Aggregation | Promtail + Loki |
| Rate Limiting | Traefik middleware + Redis |
| Local Orchestration | Docker Compose |
| CI Config | GitHub Actions per-service pipeline config |

---

## AI/ML Knowledge

> Not applicable — infrastructure routing layer.

| Relevant context | Detail |
|---|---|
| ML service routing | Routes to GPU-heavy services (`09`, `10`, `12`) — important for timeout config |
| Model drift alerts | Grafana alert rule on MAPE metric from `12` |

---

## Data Knowledge

| Storage | Purpose | Format |
|---|---|---|
| Prometheus TSDB | All service metrics time-series | Binary (Prometheus format) |
| Loki | Log streams from all containers | Log lines + labels |
| Jaeger / Tempo | Distributed trace spans | OTLP |
| Redis | Rate limit counters | Key-Value with TTL |

---

## Infrastructure Knowledge

| Concern | Detail |
|---|---|
| Deployment | Traefik runs as Docker container with socket access |
| Service discovery | Docker labels → Traefik auto-discovers services in Compose |
| TLS termination | Traefik Let's Encrypt ACME for production |
| Dev environment | `docker-compose.yml` brings up all 20 services locally |
| High availability | Traefik HA mode for production (multiple instances) |
| Kubernetes | Traefik Ingress Controller for K8s deployment (via `20`) |

---

## Security Knowledge

| Concern | Detail |
|---|---|
| ForwardAuth | Every request validated against `05` before reaching backend |
| HTTPS everywhere | TLS termination at Traefik; internal traffic can be HTTP in trusted network |
| Rate limiting | Protect `05` login endpoint (5 req/min), AI endpoints (10 req/sec) |
| CORS | Traefik CORS middleware — only allow known origins |
| Dashboard security | Traefik dashboard behind auth — never public |
| Secret management | All API keys via Vault or Docker secrets |

---

## Performance Knowledge

| Concern | Detail |
|---|---|
| Request overhead | Traefik adds ~1ms per request (acceptable) |
| Auth forward latency | `05` auth validation: target < 5ms (cached JWT verify) |
| Circuit breaker | Auto-open on 50% error rate → prevents cascade failures |
| AI endpoint timeouts | 30s timeout for `09`, `10` (image inference is slow) |
| WebSocket | Long-lived connections for `13` risk alerts — configure keep-alive |

---

## Testing Knowledge

| Test Type | Approach |
|---|---|
| Routing Tests | Each service reachable via gateway path |
| Auth Tests | Unauthenticated requests rejected with 401 |
| Rate Limit Tests | Verify 429 after threshold |
| Circuit Breaker Tests | Downstream failure → gateway returns 503 |
| Observability Tests | Verify metrics appear in Prometheus after service request |

---

## Observability

> This repo IS the observability hub.

| Concern | Detail |
|---|---|
| Metrics | Per-service request rate, error rate, latency (RED metrics) |
| Dashboards | Grafana: per-service RED dashboard + cross-service flow |
| Alerts | PagerDuty/email on: error rate > 1%, p99 > 500ms, service down |
| Traces | Full distributed trace from mobile request → multiple services |
| Logs | All container logs aggregated to Loki, searchable in Grafana |

---

## Research Complexity

| Level | Justification |
|---|---|
| 🟡 Intermediate | Traefik config is declarative but distributed tracing integration requires depth; Grafana dashboards require metric design knowledge |

---

## Portfolio Value

| Skill Demonstrated |
|---|
| Production API gateway design (Traefik) |
| Full observability stack: Prometheus + Grafana + Loki + Jaeger |
| OpenTelemetry distributed tracing implementation |
| Rate limiting and circuit breaker configuration |
| Docker Compose master environment orchestration |
| Security-first gateway (ForwardAuth + CORS + TLS) |
