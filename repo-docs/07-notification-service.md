# 🔔 Repo 07 — `notification-service`

> **Status:** Planned  
> **Flag:** —  
> **Difficulty:** ⭐⭐  
> **Build Priority:** 8 (after risk-alert-engine 13 is designed)

---

## Purpose

The **async notification delivery engine** for GlucoVision. All patient-facing alerts — glucose emergencies, medication reminders, meal reminders — flow through this service. It is decoupled from the request path so that a slow FCM delivery never blocks a glucose prediction call. It owns its own queue, its own delivery retry logic, and its own localized content templates for Sinhala and Tamil patients.

---

## Core Functionality

| Module | What It Does |
|---|---|
| `push/` | FCM (Android) and APNs (iOS) push notification delivery |
| `alerts/` | Receive glucose threshold alerts from `13` risk-alert-engine, format and deliver |
| `reminders/` | Scheduled meal reminders, medication reminders, appointment reminders |
| `templates/` | Localized notification copy: English / Sinhala / Tamil |
| `delivery_log/` | Track notification delivery status, read receipts |
| `preferences/` | Per-user notification opt-in/out settings |

---

## Inputs

| Input | Source | Format |
|---|---|---|
| Glucose alert events | `13` risk-alert-engine | JSON via message queue |
| Reminder schedule | `15` recommendation engine (meal plan) | JSON (scheduled time + user_id) |
| User language preference | `06` user service | String (en / si / ta) |
| FCM device tokens | Mobile app (03) on registration | String |
| Notification config | Web dashboard admin (04) | JSON |

---

## Outputs

| Output | Consumer | Format |
|---|---|---|
| Push notification | Patient mobile device (03) | FCM / APNs payload |
| Delivery receipt | Internal delivery log | JSON |
| Notification history | Web dashboard (04) | JSON REST |
| Failed delivery report | Ops team | Log / metric |

---

## Dependencies

| Service / Tool | Role |
|---|---|
| `13` risk-alert-engine | Triggers real-time glucose alerts |
| `06` user-service | Fetch language preference for localization |
| `15` recommendation-engine | Sends meal reminder schedule |
| Redis | Celery broker + result backend |
| Celery | Async task queue for background delivery |
| FCM (Firebase Cloud Messaging) | Android push delivery |
| APNs (Apple Push Notification Service) | iOS push delivery |
| PostgreSQL | Notification logs, user device tokens, reminder schedules |

---

## Communication

| Protocol | Used For |
|---|---|
| REST / HTTPS | Admin config API, notification history API |
| Message Queue (Celery + Redis) | Async task dispatch for notification delivery |
| FCM / APNs (HTTPS) | Push delivery to devices |
| Internal HTTP | Called by `13`, `15` to trigger notifications |

---

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | FastAPI (Python) |
| Task Queue | Celery |
| Broker | Redis |
| Push Android | firebase-admin (FCM) |
| Push iOS | apns2 or httpx (APNs HTTP/2) |
| Database | PostgreSQL (SQLAlchemy) |
| Containerization | Docker |
| Scheduler | Celery Beat (for reminder cron jobs) |

---

## AI/ML Knowledge

> Not applicable — no ML inference.

| Relevant context | Detail |
|---|---|
| Alert intelligence | Receives already-computed risk level from `13` — formats and delivers |
| Reminder optimization | Future: ML-based optimal reminder timing (not in MVP) |

---

## Data Knowledge

| Table | Key Fields | Purpose |
|---|---|---|
| `device_tokens` | user_id, platform (android/ios), token, updated_at | FCM/APNs delivery |
| `notification_logs` | id, user_id, type, payload, status, sent_at, read_at | Delivery audit trail |
| `reminder_schedules` | user_id, reminder_type, scheduled_time, recurrence | Meal/medication reminders |
| `templates` | locale, notification_type, title, body | Localized copy |

**Localization keys:**
```
glucose_alert.hypo.title: "ගිනිකොළ අනතුරු ඇඟවීම" (Sinhala)
glucose_alert.hypo.body: "ඔබේ රුධිර ගිනිකොළ මට්ටම {value} mmol/L — ඉක්මනින් ග්ලූකෝස් ගන්න"
```

---

## Infrastructure Knowledge

| Concern | Detail |
|---|---|
| Deployment | Docker container; Celery worker as separate container |
| Scaling | Scale Celery workers independently from FastAPI |
| Retry policy | Celery retry with exponential backoff for FCM failures |
| Celery Beat | Separate container for cron-based reminder scheduling |

---

## Security Knowledge

| Concern | Detail |
|---|---|
| FCM server key | Stored in HashiCorp Vault — never in env file |
| APNs certificate | p8 key file in Vault |
| Patient PII | Notification body must not include full glucose values in logs |
| Notification auth | Only `13` and `15` can trigger alerts (internal auth header) |

---

## Performance Knowledge

| Concern | Detail |
|---|---|
| Glucose alert latency | Target < 2 seconds from alert trigger to device push |
| Queue throughput | Celery workers scale to handle burst (post-meal period) |
| FCM rate limits | Google FCM: 600 messages/sec per project — batch if needed |
| Deduplication | Deduplicate rapid duplicate alerts (glucose alert storm protection) |

---

## Testing Knowledge

| Test Type | Approach |
|---|---|
| Unit Tests | Template rendering with localization, alert formatting |
| Integration Tests | Alert event → Celery task → FCM mock delivery |
| Localization Tests | All 3 locales produce valid push content |
| Delivery Tests | Retry logic on FCM 500 errors |

---

## Observability

| Concern | Detail |
|---|---|
| Logging | Every notification attempt: user_id, type, delivery status |
| Metrics | Delivery success rate, average delivery latency, queue depth |
| Alerts | Delivery failure rate > 5% → ops alert |
| Tracing | OpenTelemetry trace from `13` alert event → FCM delivery |

---

## Research Complexity

| Level | Justification |
|---|---|
| 🟢 Beginner–Intermediate | Async queue + FCM integration is well-documented; complexity is in localization and delivery guarantees |

---

## Portfolio Value

| Skill Demonstrated |
|---|
| Async task queue architecture (Celery + Redis) |
| Multi-platform push notification delivery (FCM + APNs) |
| Multi-language localization system (Sinhala / Tamil / English) |
| Medical alert delivery pipeline |
| Decoupled event-driven notification design |
| Celery Beat scheduled reminders |
