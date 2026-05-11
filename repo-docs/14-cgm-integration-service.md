# ⚡ Repo 14 — `cgm-integration-service`

> **Status:** Planned  
> **Flag:** ⚡ SEPARATE — Real-Time Medical Device API  
> **Difficulty:** ⭐⭐⭐  
> **Build Priority:** 10 (Phase 3 — Health Monitoring)

---

## Purpose

The **medical device data bridge** that connects GlucoVision to continuous glucose monitors (CGMs). CGM devices like Dexcom G6/G7 and Abbott FreeStyle Libre have proprietary vendor APIs, their own OAuth flows, rate limits, and failure modes entirely unlike any other service in the platform. This service isolates that complexity, normalizes all CGM formats to a standard `mmol/L` time-series, and streams it to the glucose prediction service (`12`).

---

## Core Functionality

| Module | What It Does |
|---|---|
| `dexcom/` | Dexcom G6/G7 OAuth2 authorization + real-time glucose data polling |
| `libre/` | Abbott FreeStyle Libre NFC read integration + LibreLinkUp API |
| `manual_entry/` | Manual glucose log fallback for patients without CGM devices |
| `normalizer/` | Converts all CGM formats (mg/dL → mmol/L, different time formats) to unified schema |
| `influx_writer/` | Writes normalized glucose stream to InfluxDB |
| `polling_scheduler/` | Celery Beat scheduled polling (every 5 min for Dexcom, every 15 min for Libre) |
| `sync_status/` | Per-patient CGM sync status API for mobile display |

---

## Inputs

| Input | Source | Format |
|---|---|---|
| Dexcom OAuth code | Patient via mobile (03) | OAuth2 authorization code |
| Dexcom glucose API response | Dexcom cloud API | JSON `{ egvs: [{ value_mgdl, displayTime }] }` |
| Libre NFC scan | Mobile NFC reader (03) | LibreLinkUp JSON |
| Manual glucose entry | Patient via mobile (03) | JSON `{ value_mmol, timestamp }` |
| Patient CGM type preference | `06` user-service | String (dexcom / libre / manual) |

---

## Outputs

| Output | Consumer | Format |
|---|---|---|
| Normalized glucose time-series | InfluxDB → `12` glucose-prediction | InfluxDB line protocol |
| Real-time CGM reading | `13` risk-alert-engine | JSON WebSocket / REST |
| Sync status | Mobile app (03) | JSON `{ last_sync, device_type, status }` |
| Historical glucose export | `04` web dashboard | JSON REST (paginated) |

---

## Dependencies

| Service / Tool | Role |
|---|---|
| Dexcom Clarity API | External — Dexcom G6/G7 glucose data |
| LibreLinkUp API | External — FreeStyle Libre glucose data |
| InfluxDB | Glucose time-series storage |
| Celery + Redis | Scheduled polling every 5 min |
| `06` user-service | Patient CGM device registration |
| `12` glucose-prediction | Consumes InfluxDB glucose stream |
| `13` risk-alert-engine | Receives real-time glucose readings |
| `08` api-gateway | Request routing + auth |

---

## Communication

| Protocol | Used For |
|---|---|
| OAuth2 / HTTPS | Dexcom + Libre authentication |
| HTTPS (httpx) | Polling Dexcom / Libre APIs |
| REST / HTTPS | Manual entry API, sync status API |
| InfluxDB client | Writing normalized glucose to time-series DB |
| Internal HTTP | Streaming to `13` risk alert engine |

---

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | FastAPI (Python) |
| HTTP Client | HTTPX (async) |
| OAuth2 | authlib |
| Task Queue | Celery |
| Broker | Redis |
| Scheduler | Celery Beat |
| Time-Series DB | InfluxDB |
| Database | PostgreSQL (OAuth tokens, device registrations) |
| Containerization | Docker |

---

## AI/ML Knowledge

> No ML inference in this service — it is a data ingestion and normalization pipeline.

| Relevant context | Detail |
|---|---|
| Data quality for `12` | CGM data gaps (device disconnects) must be handled — interpolation strategy |
| Sensor noise | Raw CGM values can spike ±5% — `12` preprocessing handles smoothing |
| Calibration | Dexcom factory-calibrated (no manual calibration); Libre requires 1 scan = 1 reading |

---

## Data Knowledge

| Storage | Purpose | Format |
|---|---|---|
| InfluxDB | All normalized glucose readings | `measurement=glucose_readings, user_id=tag, value_mmol=field` |
| PostgreSQL | OAuth tokens (encrypted), device registrations | Relational |

**Unified glucose schema (InfluxDB):**
```
measurement: glucose_readings
tags:
  user_id: "patient-uuid"
  source: "dexcom_g7 | libre3 | manual"
  device_serial: "optional"
fields:
  value_mmol: 5.8          # float
  raw_mgdl: 104            # original value (optional)
  trend: "stable"          # rising / falling / stable / rapid_rise
timestamp: 1715000000000000000  # nanoseconds
```

---

## Infrastructure Knowledge

| Concern | Detail |
|---|---|
| Celery Beat | One polling job per active patient per device type |
| Rate limits | Dexcom API: 60 req/min per app token — batch patient polls |
| Token refresh | OAuth2 access tokens refresh automatically before expiry |
| Dexcom sandbox | Use Dexcom Developer Sandbox for testing |
| Libre access | LibreLinkUp API requires patient account linking |

---

## Security Knowledge

| Concern | Detail |
|---|---|
| OAuth tokens | Stored encrypted in PostgreSQL (AES-256) — never in env |
| Patient data | Glucose readings = PHI — InfluxDB encryption at rest |
| Scope | Dexcom OAuth scopes: `offline_access egv` only |
| Token isolation | Per-patient OAuth token — one patient's revocation doesn't affect others |

---

## Performance Knowledge

| Concern | Detail |
|---|---|
| Polling latency | Dexcom provides readings with ~5 min delay; Libre every 1 min |
| Celery worker scaling | Scale workers with patient count (1 worker per ~100 patients) |
| InfluxDB write throughput | Batch writes (100 points per request) for efficiency |
| Manual entry | Instant write — no polling delay |

---

## Testing Knowledge

| Test Type | Approach |
|---|---|
| Unit Tests | Normalizer: mg/dL → mmol/L conversion, trend detection |
| Integration Tests | Dexcom sandbox → poll → InfluxDB write → verify |
| Polling Tests | Verify Celery Beat triggers at 5-min interval |
| Token Tests | OAuth refresh flow on expired token |
| Error Tests | Dexcom API timeout → graceful retry → log |

---

## Observability

| Concern | Detail |
|---|---|
| Metrics | Readings per patient per hour, API error rate, polling latency |
| Logging | Each poll: patient_id, device_type, readings_fetched, timestamp |
| Alerts | Dexcom API error rate > 5% → ops alert; patient missed 2 polls in a row → sync warning |

---

## Research Complexity

| Level | Justification |
|---|---|
| 🟡 Intermediate | Vendor API integration complexity; OAuth2 token lifecycle management; time-series data normalization; real-time medical data pipeline |

---

## Portfolio Value

| Skill Demonstrated |
|---|
| Medical device API integration (Dexcom + FreeStyle Libre) |
| OAuth2 vendor flow implementation |
| Async Celery polling scheduler |
| InfluxDB time-series data ingestion pipeline |
| Multi-device CGM normalization |
| Medical data PHI security patterns |
