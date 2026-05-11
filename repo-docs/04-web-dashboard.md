# 🖥️ Repo 04 — `web-dashboard`

> **Status:** Existing · Expand  
> **Flag:** —  
> **Difficulty:** ⭐⭐  
> **Build Priority:** 7 (parallel with mobile, after gateway)

---

## Purpose

The **clinician and researcher-facing web portal** for GlucoVision. While the mobile app serves patients, this dashboard serves doctors, nutritionists, and researchers who need to monitor patient populations, review AI model performance, manage the federated learning system, and explore digital twin scenarios. It is the **analytics and governance layer** of the platform.

---

## Core Functionality

| Module | What It Does |
|---|---|
| Patient Analytics | Aggregate + per-patient glucose trend visualization |
| Glucose / Activity Charts | Interactive time-series charts (InfluxDB data via `12`) |
| Food Log Review | Review patient food logs and AI-recognized meals |
| Model Monitoring | ML model drift metrics, accuracy dashboards (from MLflow via `09`, `12`) |
| FL Status Dashboard | Federated learning round progress, convergence metrics from `19` |
| Digital Twin Studio | Interactive what-if simulation UI connected to `18` |
| Recommendation Review | Clinician review and override of AI diet recommendations |
| Alert Management | Review and configure risk alert thresholds for `13` |
| User Management | Patient and clinician account management (via `06`) |
| Notification Controls | Configure push notification triggers (via `07`) |

---

## Inputs

| Input | Source | Format |
|---|---|---|
| Patient glucose data | `12` glucose prediction + `14` CGM | JSON REST / time-series |
| Food logs | `09` food recognition, `10` portion estimation | JSON REST |
| Activity data | `15` recommendation engine | JSON REST |
| Model metrics | MLflow registry | JSON REST |
| FL round metrics | `19` federated learning | JSON REST |
| Digital twin projections | `18` | JSON REST |
| Risk alert history | `13` | JSON REST |
| User list | `06` user service | JSON REST |

---

## Outputs

| Output | Destination | Format |
|---|---|---|
| Clinician override decisions | `15` recommendation engine | JSON REST (PATCH) |
| Alert threshold configuration | `13` risk alert engine | JSON REST (PUT) |
| Notification template edits | `07` notification service | JSON REST |
| User role changes | `05` auth service | JSON REST |

---

## Dependencies

| Service | Role |
|---|---|
| `05` auth-service | Clinician / admin login, RBAC enforcement |
| `06` user-service | Patient and clinician profiles |
| `07` notification-service | Notification management UI |
| `08` api-gateway | Single entry point for all API calls |
| `09` food-recognition | Food log data display |
| `12` glucose-prediction | Glucose forecast charts |
| `13` risk-alert-engine | Alert history and threshold config |
| `15` recommendation-engine | Recommendation review and override |
| `18` digital-twin | What-if simulation studio |
| `19` federated-learning | FL round monitoring |
| MLflow | Model registry and metrics |

---

## Communication

| Protocol | Used For |
|---|---|
| REST / HTTPS | All API calls (via gateway `08`) |
| WebSocket | Real-time glucose alert feed for clinician dashboard |
| Server-Sent Events (SSE) | (Optional) streaming model training progress |

---

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | React + Vite (TypeScript) |
| State Management | Redux Toolkit / Zustand |
| Routing | React Router v6 |
| Charts | Recharts / Chart.js / D3.js |
| UI Components | Shadcn/UI or custom component library |
| HTTP Client | Axios / React Query |
| WebSocket | native WebSocket API |
| CSS | Vanilla CSS / CSS Modules |
| Auth | JWT storage in httpOnly cookie |
| Build | Vite |
| Linting | ESLint + Prettier |
| Containerization | Docker |

---

## AI/ML Knowledge

| Concern | Detail |
|---|---|
| Model drift visualization | MAPE trend charts for glucose prediction model |
| Federated learning monitoring | Round convergence, client participation, loss curves |
| Digital twin controls | UI for running Neural ODE what-if scenarios |
| Recommendation explainability | Display why a meal plan was generated (LLM rationale from `15`) |

---

## Data Knowledge

| Data Source | Access Method | Format |
|---|---|---|
| Patient glucose time-series | REST via `12` (backed by InfluxDB) | JSON |
| Food and nutrition records | REST via `09`, `15` (backed by PostgreSQL) | JSON |
| Model registry | REST via MLflow | JSON |
| FL metrics | REST via `19` | JSON |
| Digital twin projections | REST via `18` | JSON |

---

## Infrastructure Knowledge

| Concern | Detail |
|---|---|
| Deployment | Docker container, served behind Traefik gateway (`08`) |
| Static hosting | Can be deployed to Nginx or CDN for static assets |
| Auth guard | Route-level RBAC enforcement (patient vs clinician vs admin vs researcher) |
| Environment config | `.env` for API base URL, Firebase config |

---

## Security Knowledge

| Concern | Detail |
|---|---|
| Auth | JWT via httpOnly cookie (prevents XSS token theft) |
| RBAC | Route and component-level role checks (patient data hidden from wrong roles) |
| HTTPS | All API calls over TLS |
| CORS | Backend allows only dashboard origin |
| Sensitive data | Patient names / glucose values — mask/blur until clinician confirms access |

---

## Performance Knowledge

| Concern | Detail |
|---|---|
| Chart rendering | Virtualize large glucose datasets; paginate or downsample for display |
| API caching | React Query stale-while-revalidate for non-real-time data |
| Bundle size | Code-split by route (lazy load ML monitoring, FL studio) |
| Real-time | WebSocket for alert feed; avoid polling for low-latency data |

---

## Testing Knowledge

| Test Type | Approach |
|---|---|
| Unit Tests | Redux reducers, data transformation utilities |
| Component Tests | Vitest + React Testing Library for chart components |
| E2E Tests | Playwright: login → view patient glucose chart → override recommendation |
| Accessibility | axe-core accessibility audit on key screens |

---

## Observability

| Concern | Detail |
|---|---|
| Error logging | Sentry for frontend error capture |
| Analytics | Page-level usage analytics for feature prioritization |
| Performance | Lighthouse CI on pull requests |
| API monitoring | Track API error rates from dashboard to gateway |

---

## Research Complexity

| Level | Justification |
|---|---|
| 🟡 Intermediate | Complex data visualization, RBAC, real-time integration, ML monitoring UI |

---

## Portfolio Value

| Skill Demonstrated |
|---|
| Clinical analytics dashboard design |
| ML model monitoring and drift visualization |
| Federated learning progress UI |
| Digital twin interactive simulation studio |
| Multi-role RBAC frontend implementation |
| Real-time WebSocket data in React |
| Complex time-series chart rendering (D3 / Recharts) |
