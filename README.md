<div align="center">

# 🏗️ GlucoVision — Platform Architecture

**The canonical design hub for the GlucoVision 21-service AI platform.**  
*System architecture · OpenAPI contracts · ADRs · Service maps · Build roadmap*

[![Architecture](https://img.shields.io/badge/Architecture-21%20Repos-6366f1?style=for-the-badge)](#)
[![Research](https://img.shields.io/badge/Research-AI%20Diabetic%20Management-10b981?style=for-the-badge)](#)
[![Docs](https://img.shields.io/badge/Docs-OpenAPI%203.1-f59e0b?style=for-the-badge)](#)

</div>

---

## 📌 What This Repo Is

This is **not a service** — it runs no code. It is the **single source of truth** for the entire GlucoVision platform: every architectural decision, every service boundary, every API contract, and every inter-service data flow is documented here before being implemented elsewhere.

> Based on: *Systematic Review of AI-Based Personalized Diet and Activity Recommendation System for Diabetic Individuals*

---

## 🗂️ Repository Structure

```
01-glucovision-platform-architecture/
├── glucovision_reanalysis.md     # Master architecture — 21-repo system design
├── repo-docs/                    # Per-repo knowledge documents (all 21 services)
│   ├── README.md                 # Index of all 21 repo docs
│   ├── 01-platform-architecture.md
│   ├── 02-data-collect-app.md
│   ├── ...
│   └── 21-devops-infra.md
└── docs/                         # OpenAPI specs, ADRs, diagrams (in progress)
```

---

## 📦 The 21-Repo System

```
Phase 1 — Foundation          Phase 2 — Food AI         Phase 3 — Health AI
──────────────────────        ─────────────────         ────────────────────
01 platform-architecture      09 food-recognition 🖥️    12 glucose-pred 🔴
02 data-collect-app           10 portion-estimation     13 risk-alerts  🔴
03 glucovision-mobile         11 cooking-ai             14 cgm-integration ⚡
04 web-dashboard
05 auth-service 🔐            Phase 4 — Intelligence    Phase 5 — Research
06 user-service               ──────────────────────    ──────────────────
07 notification-service       15 recommendation-engine  19 digital-twin 🔬
08 api-gateway                16 assistant-service      20 federated-learning 🔬
17 esp32-cam-streaming        21 devops-infra
18 wearable-sync
```

---

## 📖 Key Documents

| Document | Purpose |
|---|---|
| [`glucovision_reanalysis.md`](./glucovision_reanalysis.md) | Full 21-repo architecture plan with Mermaid diagrams, build order, decision table |
| [`repo-docs/README.md`](./repo-docs/README.md) | Quick-reference index for all 21 repo knowledge documents |
| [`repo-docs/`](./repo-docs/) | Per-service knowledge docs covering tech stack, AI/ML, security, testing |

---

## 🔑 Core Design Principles

| Principle | Applied To |
|---|---|
| 🔐 **Security isolation** | `05` auth — independent security CVE lifecycle |
| 🔴 **Medical independence** | `12` + `13` never share deployment — independent rollback |
| ⚡ **Real-time separation** | `13`, `14` — different SLA from batch services |
| 🔬 **Research separation** | `19`, `20` — experimental lifecycle |
| 🎥 **IoT domain split** | `17` camera vs `18` wearable — different hardware domains |
| 🧠 **Pipeline consolidation** | `15` recommendation — sequential pipeline, not fake microservices |

---

## 🛠️ How to Use This Repo

1. **Before building any service** → read its knowledge doc in `repo-docs/`
2. **Before defining an API** → add an OpenAPI spec to `docs/openapi/`
3. **Before making an architectural decision** → create an ADR in `docs/adr/`
4. **Service communication** → check the Mermaid dependency graph in `glucovision_reanalysis.md`

---

## 🔗 Related Repos

| Repo | Description |
|---|---|
| `02-data-collect-app` | Sri Lankan food image dataset collection platform |
| `03-glucovision-mobile` | Patient-facing Flutter mobile application |
| `04-glucovision-web-dashboard` | Clinician and researcher React web portal |

---

<div align="center">

*GlucoVision — AI-Powered Personalized Diabetes Management for Sri Lankan Patients*

</div>
