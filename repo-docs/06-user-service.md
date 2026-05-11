# 👤 Repo 06 — `user-service`

> **Status:** Planned  
> **Flag:** —  
> **Difficulty:** ⭐⭐  
> **Build Priority:** 4 (after auth-service)

---

## Purpose

The **patient and clinician profile store** of GlucoVision. Separated from the auth service because authentication (who you are) and user data (what we know about you) change at different rates and for different reasons. A password reset touches `05-auth`. A patient updating their HbA1c target touches `06-user`. These are independent domains with independent business logic.

---

## Core Functionality

| Module | What It Does |
|---|---|
| `profiles/` | Patient and clinician core profile CRUD (name, DOB, diabetes type) |
| `preferences/` | Cultural food preferences, language setting (Sinhala / Tamil / English) |
| `health_metadata/` | BMI, weight history, HbA1c target, medication list |
| `clinician_link/` | Associate patient accounts with their assigned clinician |
| `consent/` | Patient data sharing consent records (GDPR/HIPAA) |

---

## Inputs

| Input | Source | Format |
|---|---|---|
| New user registration data | `05` auth-service (post-registration hook) | JSON |
| Profile update | Mobile app (03), Web dashboard (04) | JSON (PATCH) |
| Health metadata update | Patient via mobile (03) | JSON |
| Clinician assignment | Admin via web dashboard (04) | JSON |
| Language preference | App settings | String (en / si / ta) |

---

## Outputs

| Output | Consumer | Format |
|---|---|---|
| Full patient profile | Mobile app (03), Web dashboard (04) | JSON REST |
| Cultural food preferences | `15` recommendation engine | JSON |
| Health metadata (BMI, HbA1c target) | `12` glucose prediction, `15` recommendation | JSON |
| Language preference | `07` notification service (localized copy) | String |
| Patient-clinician mapping | `04` web dashboard | JSON |

---

## Dependencies

| Service / Tool | Role |
|---|---|
| `05` auth-service | Auth token validation; calls user-service to resolve user_id on login |
| PostgreSQL | Persistent profile storage |
| `08` api-gateway | All external traffic routed through gateway |

---

## Communication

| Protocol | Used For |
|---|---|
| REST / HTTPS | CRUD on profiles, preferences, health metadata |
| Internal HTTP | Called by `15`, `12` to fetch patient context |

---

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | FastAPI (Python) |
| Database ORM | SQLAlchemy + Alembic (migrations) |
| Database | PostgreSQL |
| Validation | Pydantic v2 |
| Containerization | Docker |

---

## AI/ML Knowledge

> Not applicable — pure domain CRUD service.

| Relevant data for AI | Detail |
|---|---|
| Diabetes type (T1D / T2D / GDM) | Consumed by `15` to select meal planning rules |
| HbA1c target | Consumed by `12` to calibrate glucose prediction thresholds |
| Cultural food preferences | Consumed by `15` to filter Sri Lankan meal recommendations |
| BMI + weight history | Consumed by `15` for energy (TDEE) calculation |

---

## Data Knowledge

| Table | Key Fields | Purpose |
|---|---|---|
| `users` | id, auth_user_id (FK→auth), name, dob, diabetes_type | Core identity |
| `preferences` | user_id, food_culture, language, dietary_restrictions | Personalization |
| `health_metadata` | user_id, bmi, weight_kg, hba1c_target, medications (JSONB) | Clinical context |
| `clinician_links` | patient_id, clinician_id, assigned_at | Care team mapping |
| `consent_records` | user_id, scope, granted_at, revoked_at | GDPR compliance |

---

## Infrastructure Knowledge

| Concern | Detail |
|---|---|
| Deployment | Docker container behind gateway |
| Migrations | Alembic — versioned DB schema changes |
| Scaling | Stateless service → horizontal scale with connection pool |
| Data residency | Patient data must remain in-region (regulatory) |

---

## Security Knowledge

| Concern | Detail |
|---|---|
| Auth | Every endpoint validates JWT from `05` |
| Authorization | Users can only access their own data; clinicians access assigned patients |
| PII handling | Name, DOB — encrypted at rest in PostgreSQL |
| Consent records | Immutable append-only consent log |
| Data retention | Support account deletion (GDPR right to erasure) |

---

## Performance Knowledge

| Concern | Detail |
|---|---|
| Read performance | Index on `user_id`, `auth_user_id` for fast lookups |
| JSONB fields | Medications stored as JSONB — flexible but indexed |
| Connection pooling | PgBouncer or SQLAlchemy pool for high concurrency |

---

## Testing Knowledge

| Test Type | Approach |
|---|---|
| Unit Tests | Profile update logic, consent record validation |
| Integration Tests | Full CRUD flow: create → update → fetch → delete |
| Auth Tests | Verify unauthorized access returns 403 |
| Data Tests | GDPR erasure leaves no PII traces |

---

## Observability

| Concern | Detail |
|---|---|
| Logging | Profile update events with user_id (no PII in log body) |
| Metrics | Profile fetch rate, update frequency |
| Alerts | Unusual bulk update activity (possible data breach indicator) |

---

## Research Complexity

| Level | Justification |
|---|---|
| 🟢 Beginner–Intermediate | Standard REST CRUD; complexity is in data modeling for medical profiles and GDPR compliance |

---

## Portfolio Value

| Skill Demonstrated |
|---|
| Domain-driven microservice design (user ≠ auth) |
| Multi-culture localization data modeling (Sinhala / Tamil) |
| GDPR-compliant patient data handling |
| Medical profile schema design (diabetes type, HbA1c, medications) |
| SQLAlchemy + Alembic production patterns |
