# 📐 Repo 01 — `platform-architecture`

> **Status:** Existing · Active  
> **Flag:** —  
> **Difficulty:** ⭐  
> **Build Priority:** 1 (Foundation — build first)

---

## Purpose

This repository is the **single source of truth** for the entire GlucoVision platform. It contains no running code — only design artifacts, specifications, and governance documents that every other repository depends on. Without this repo, inter-service contracts, API boundaries, and architectural decisions have no canonical reference.

---

## Core Functionality

| Module | What It Does |
|---|---|
| OpenAPI Specs | Machine-readable REST API contracts for all 20 services |
| Mermaid Diagrams | System topology, data flow, service dependency maps |
| ADRs (Architecture Decision Records) | Why each architectural decision was made and alternatives considered |
| Service Communication Maps | Who calls whom, over which protocol, with what payload |
| Data Model Schemas | Canonical definitions for Patient, Glucose Reading, Meal Record, etc. |
| Build Roadmap | Phase-by-phase development timeline and priority ordering |
| Repo Ownership Docs | Per-repo knowledge docs (this set of files) |

---

## Inputs

| Input | Source |
|---|---|
| Research paper insights | *Systematic Review of AI-Based PDAR System for Diabetic Individuals* |
| Team design decisions | Architecture Decision Records (ADRs) |
| API surface agreements | Cross-team service contract negotiations |

---

## Outputs

| Output | Consumer |
|---|---|
| OpenAPI YAML/JSON specs | All 20 services, API gateway (08), web dashboard (04) |
| Architecture diagrams | Developers, stakeholders, presentations |
| ADR decisions | Every repo that implements a referenced decision |
| Service dependency graph | DevOps infra (20), API gateway (08) |

---

## Dependencies

| Dependency | Type |
|---|---|
| None (runtime) | This repo generates no artifacts at runtime |
| Git/GitHub | Version control and ADR history |
| Mermaid CLI / Docusaurus | Optional diagram rendering |

---

## Communication

| Protocol | Use |
|---|---|
| None (static docs) | No runtime communication |
| Git commit hooks | Lint OpenAPI specs on commit |

---

## Tech Stack

| Layer | Technology |
|---|---|
| Documentation | Markdown, Mermaid |
| API Specification | OpenAPI 3.1 (YAML) |
| Diagram Rendering | Mermaid CLI / GitHub native rendering |
| ADR Format | MADR (Markdown Architectural Decision Records) |
| Optional Portal | Docusaurus / Swagger UI |

---

## AI/ML Knowledge

> Not applicable — this repo contains no ML code.

---

## Data Knowledge

| Artifact | Format |
|---|---|
| OpenAPI Specs | YAML / JSON |
| Diagrams | Mermaid DSL embedded in Markdown |
| ADRs | Markdown |
| Schema Definitions | JSON Schema or Protobuf stubs (reference only) |

---

## Infrastructure Knowledge

| Concern | Detail |
|---|---|
| Deployment | Static — no deployment needed |
| Hosting | GitHub repository / optional GitHub Pages |
| CI | OpenAPI linting (spectral), Mermaid render checks |

---

## Security Knowledge

| Concern | Detail |
|---|---|
| Sensitive data | Must never contain real patient data or secrets |
| Secret management | `.env.example` files only — never real credentials |
| ADR for security | Records auth strategy, encryption decisions |

---

## Performance Knowledge

> Not applicable — no runtime workload.

---

## Testing Knowledge

| Test Type | Approach |
|---|---|
| OpenAPI Linting | `spectral lint openapi.yaml` in CI |
| Link checking | Validate all internal doc links |
| Schema validation | JSON Schema validation of data model files |

---

## Observability

> Not applicable — no runtime service.

---

## Research Complexity

| Level | Justification |
|---|---|
| 🟢 Beginner | Writing docs and specs requires domain knowledge, not algorithmic complexity |

---

## Portfolio Value

| Skill Demonstrated |
|---|
| System architecture thinking across 20 microservices |
| OpenAPI design and cross-service contract definition |
| ADR-driven decision documentation |
| Mermaid-based system visualization |
| Research-to-engineering translation (paper → architecture) |
