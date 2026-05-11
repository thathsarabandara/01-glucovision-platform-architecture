# 🚨 Repo 13 — `risk-alert-engine`

> **Status:** Planned  
> **Flag:** 🔴 SEPARATE — Real-Time Safety Critical  
> **Difficulty:** ⭐⭐⭐⭐  
> **Build Priority:** 14 (after glucose-prediction 12)

---

## Purpose

The **real-time clinical safety layer** that fires hypoglycemia and hyperglycemia alerts before a dangerous glucose event fully materializes. This service must have sub-second latency, zero downtime, and independent uptime SLA because a delayed alert can cause a patient to miss treatment for hypoglycemia — a medical emergency. It must never share a deployment unit with `12` glucose prediction because both must independently roll back during incidents.

---

## Core Functionality

| Module | What It Does |
|---|---|
| `threshold_monitor/` | Continuously evaluates real-time glucose values against patient-specific thresholds |
| `hypo_detector/` | Detects hypoglycemia trend (glucose < 3.9 mmol/L or predicted to drop below) |
| `hyper_detector/` | Detects hyperglycemia trend (glucose > 10 mmol/L persistent or rising rapidly) |
| `postmeal_validator/` | Post-meal prediction vs actual glucose loop — catches unexpected post-meal spikes |
| `alert_publisher/` | Publishes alerts to Kafka topic → `07` notification-service for delivery |
| `alert_config/` | Per-patient threshold configuration (clinician-set) |
| `alert_history/` | Queryable log of all past alerts |

---

## Inputs

| Input | Source | Format |
|---|---|---|
| Real-time glucose readings | `14` cgm-integration (streaming) | Kafka message / WebSocket |
| Glucose forecast curve | `12` glucose-prediction | JSON `{ horizon_min, predicted_mmol }` |
| Patient threshold configuration | `04` web dashboard (clinician) | JSON `{ hypo_threshold, hyper_threshold }` |
| Post-meal expected glucose | `15` recommendation engine | JSON |

---

## Outputs

| Output | Consumer | Format |
|---|---|---|
| Hypoglycemia alert | `07` notification-service | Kafka message `{ alert_type: "HYPO", severity, user_id, value }` |
| Hyperglycemia alert | `07` notification-service | Kafka message `{ alert_type: "HYPER", severity, user_id, trend }` |
| Post-meal spike alert | `07` notification-service, `15` recommendation | Kafka message |
| Alert event | Mobile app (03) via WebSocket | JSON (real-time display) |
| Alert feed | `15` recommendation engine | JSON (adjust next meal plan) |
| Alert history | `04` web dashboard | JSON REST |

---

## Dependencies

| Service / Tool | Role |
|---|---|
| `12` glucose-prediction | Forecast data to predict future hypo/hyper |
| `14` cgm-integration | Real-time CGM glucose stream |
| `07` notification-service | Delivers push alerts to patient |
| `15` recommendation-engine | Receives alert to adapt next meal recommendation |
| `06` user-service | Patient threshold settings |
| Kafka | Event streaming for alert publishing |
| Redis | In-memory alert state (debounce / deduplication) |
| `08` api-gateway | WebSocket proxying to mobile clients |

---

## Communication

| Protocol | Used For |
|---|---|
| Kafka (pub/sub) | Alert event publishing to `07` notification-service |
| WebSocket | Real-time alert push to mobile app (03) |
| REST / HTTPS | Alert history API, threshold config API |
| Internal HTTP | Receives forecast from `12`, sends alert context to `15` |

---

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | FastAPI (Python) |
| Event Streaming | Apache Kafka (confluent-kafka-python) |
| Real-Time State | Redis (debounce + deduplication cache) |
| WebSocket | FastAPI WebSocket endpoint |
| Database | PostgreSQL (alert history, threshold config) |
| Containerization | Docker |

---

## AI/ML Knowledge

| Concern | Detail |
|---|---|
| **Threshold rules** | Deterministic rules: hypo < 3.9 mmol/L, hyper > 10 mmol/L (clinician-configurable) |
| **Predictive alerting** | Use `12` forecast — alert before threshold is crossed (e.g., predicted to reach 3.9 in 30 min) |
| **Rate of change** | Alert on rapid glucose drop (> 0.1 mmol/L/min) even if not yet below threshold |
| **Post-meal validation** | Compare `12` predicted post-meal curve against actual CGM readings — spike = alert |
| **Alert severity** | Low (trending) / Medium (at threshold) / High (crossing threshold fast) / Critical (below 3.0) |
| **Deduplication** | Redis TTL: don't re-alert same patient for same event within 5 minutes |

---

## Data Knowledge

| Storage | Purpose | Format |
|---|---|---|
| Redis | Alert deduplication state, patient current alert status | Key-Value with TTL |
| PostgreSQL | Alert history, threshold configuration | Relational |
| Kafka | Alert event stream (7-day retention) | Binary (Avro/JSON) |

**Alert event schema:**
```json
{
  "alert_id": "uuid",
  "user_id": "patient_uuid",
  "alert_type": "HYPO_PREDICTED",
  "severity": "HIGH",
  "current_mmol": 4.1,
  "predicted_mmol_30min": 3.6,
  "threshold_mmol": 3.9,
  "timestamp": "ISO8601",
  "model_version": "v2.3.1"
}
```

---

## Infrastructure Knowledge

| Concern | Detail |
|---|---|
| Uptime SLA | 99.99% — alerts must fire even during other service deployments |
| Independent deployment | Must never share deployment unit with `12` glucose prediction |
| Kafka | At-least-once delivery guarantee for alerts |
| Redis | In-memory state with AOF persistence for restart safety |
| WebSocket | Sticky sessions for persistent patient connections |

---

## Security Knowledge

| Concern | Detail |
|---|---|
| Alert auth | Only `12` and `14` can push glucose data to this service (internal auth header) |
| Patient data | Glucose values in alerts — encrypted in transit (TLS) |
| Audit trail | Every alert logged immutably — cannot be deleted |
| Threshold config | Only clinician role can modify patient thresholds |

---

## Performance Knowledge

| Concern | Detail |
|---|---|
| Alert latency | < 500ms from CGM reading received to notification enqueued |
| Kafka throughput | Supports thousands of simultaneous patients |
| Redis dedup | Sub-millisecond deduplication check |
| WebSocket | Keep-alive + reconnect for mobile clients |

---

## Testing Knowledge

| Test Type | Approach |
|---|---|
| Alert Trigger Tests | Inject hypo glucose value → verify Kafka message published |
| Predictive Alert Tests | Inject declining forecast → verify pre-emptive alert |
| Deduplication Tests | Same event twice → verify single alert |
| Latency Tests | Measure end-to-end from CGM → Kafka → FCM |
| Uptime Tests | Kill `12` service → verify `13` continues alerting on rules |

---

## Observability

| Concern | Detail |
|---|---|
| Metrics | Alert rate per type, latency from trigger to Kafka publish, false positive rate |
| Logging | Every alert with severity, glucose value, threshold, patient_id |
| Critical alerts | If alert pipeline fails (Kafka down), trigger fallback SMS |
| Tracing | OpenTelemetry: CGM reading → threshold check → Kafka publish → FCM delivery |

---

## Research Complexity

| Level | Justification |
|---|---|
| 🟠 Advanced | Real-time event-driven medical alert pipeline; Kafka integration; predictive alerting using time-series forecasts; sub-second latency requirements |

---

## Portfolio Value

| Skill Demonstrated |
|---|
| Real-time clinical alert pipeline design |
| Kafka event streaming for medical alerts |
| Predictive hypoglycemia/hyperglycemia detection |
| Redis-backed alert deduplication |
| WebSocket real-time alert push to mobile |
| Medical-grade independent deployment architecture |
| Post-meal glucose spike validation loop |
