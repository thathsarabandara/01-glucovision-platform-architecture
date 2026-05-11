# 🥗 Repo 15 — `recommendation-engine`

> **Status:** Planned  
> **Flag:** —  
> **Difficulty:** ⭐⭐⭐⭐  
> **Build Priority:** 16 (Phase 4 — Intelligence)

---

## Purpose

The **personalized diet and activity recommendation brain** — the service that delivers GlucoVision's core research value proposition. It synthesizes a patient's food intake, glucose levels, physical activity, cultural preferences, and health history into a personalized daily meal plan and exercise routine. Consolidated into one service because diet, nutrition, energy, context, and personalization are a **sequential pipeline** — splitting them creates fake microservices that can't function independently.

**Research basis:** Glycemic Index (GI)-based diet planning, BMR/TDEE calculation, Sri Lankan food nutritional database, Reinforcement Learning personalization, LLaMA-3 dietary advisory [systematic review, multiple citations].

---

## Core Functionality

| Module | What It Does |
|---|---|
| `nutrition_engine/` | Calculates macros, GI load, Sri Lankan food-specific rules |
| `energy_engine/` | BMR calculation (Harris-Benedict) + activity factor → TDEE |
| `diet_recommendation/` | Generates personalized Sri Lankan meal plans (breakfast / lunch / dinner / snacks) |
| `activity_recommendation/` | Home workout plans calibrated to patient energy expenditure |
| `context_engine/` | Holistic signal fusion: food + glucose + activity + time-of-day |
| `personalization/` | RL-based adaptation — learns from patient feedback over time |
| `llm_advisory/` | LLaMA-3 powered dietary Q&A chat interface |
| `meal_schedule/` | Generates timed meal reminders sent to `07` notification-service |

---

## Inputs

| Input | Source | Format |
|---|---|---|
| Food recognition result | `09` food-recognition | JSON `{ food_id, confidence }` |
| Portion weight | `10` portion-estimation | JSON `{ food_id, weight_g }` |
| Glucose forecast | `12` glucose-prediction | JSON `{ horizon_min, predicted_mmol }` |
| Risk alert context | `13` risk-alert-engine | JSON `{ alert_type, severity }` |
| Activity data | `17` edge-iot (wearable: steps, HR) | JSON |
| Patient profile | `06` user-service | JSON `{ diabetes_type, hba1c_target, bmi, food_culture, medications }` |
| Cooking method | `11` cooking-ai | JSON `{ cooking_state, nutrition_adjustment }` |
| Patient feedback (RL signal) | Mobile app (03) (thumbs up/down on recommendations) | JSON |

---

## Outputs

| Output | Consumer | Format |
|---|---|---|
| Daily meal plan | Mobile app (03), Web dashboard (04) | JSON (structured meal plan with Sri Lankan dishes) |
| Nutrition summary | Mobile app (03) | JSON `{ total_carbs, total_calories, gi_load }` |
| Activity recommendations | Mobile app (03) | JSON (workout plan) |
| TDEE calculation | Mobile app (03), Digital twin (18) | JSON `{ bmr, tdee, deficit_surplus }` |
| Meal reminder schedule | `07` notification-service | JSON (scheduled time + meal) |
| LLM dietary chat response | Mobile app (03), `16` assistant | JSON `{ answer, sources }` |
| Recommendation context | `18` digital-twin | JSON |

---

## Dependencies

| Service / Tool | Role |
|---|---|
| `09` food-recognition | Food identified from photo |
| `10` portion-estimation | Portion weight for nutrition calculation |
| `12` glucose-prediction | Future glucose impact of recommended meals |
| `13` risk-alert-engine | Emergency dietary adjustment |
| `06` user-service | Cultural preferences, diabetes type, HbA1c target |
| `11` cooking-ai | Cooking method nutrition adjustment |
| `07` notification-service | Meal reminder scheduling |
| LLaMA-3 (local/API) | LLM dietary advisory |
| LangChain | LLM prompt pipeline + RAG for nutrition knowledge |
| PostgreSQL | Meal plan history, RL feedback store |

---

## Communication

| Protocol | Used For |
|---|---|
| REST / HTTPS | Meal plan API, nutrition summary, activity plan |
| Internal HTTP | Called by `18` digital twin, feeds into `07` for reminders |
| LangChain | LLM invocation pipeline |

---

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | FastAPI (Python) |
| LLM | LLaMA-3 8B / 70B (local via Ollama or API) |
| LLM Framework | LangChain + LangGraph |
| RAG | LangChain + ChromaDB / pgvector for nutrition knowledge base |
| RL | Stable-Baselines3 / custom contextual bandit |
| Nutrition calculation | NumPy, pandas |
| Database | PostgreSQL (SQLAlchemy) |
| Containerization | Docker |

---

## AI/ML Knowledge

| Concern | Detail |
|---|---|
| **GI-based planning** | Recommends low-GI foods for stable glucose; respects patient HbA1c target |
| **BMR calculation** | Harris-Benedict equation: BMR = 88.36 + (13.4 × weight_kg) + (4.8 × height_cm) − (5.7 × age) |
| **TDEE** | BMR × activity factor (1.2–1.9 based on wearable step + HR data) |
| **Sri Lankan rules** | Rice and curry: carb-heavy but can be GI-adjusted by rice variety (red rice GI=55 vs white rice GI=72) |
| **RL personalization** | Multi-armed bandit: tries different meal plan variants, learns from patient acceptance rate |
| **LLaMA-3 advisory** | RAG over Sri Lankan Nutrition Database + WHO diabetes guidelines → grounded dietary Q&A |
| **Context fusion** | Combine: time-of-day + last glucose + activity + previous meal → next meal recommendation |
| **Postmeal feedback** | If `13` fires post-meal spike → recommendation engine penalizes that meal type |

---

## Data Knowledge

| Storage | Purpose | Format |
|---|---|---|
| PostgreSQL | Meal plan history, RL feedback, patient nutrition targets | Relational |
| ChromaDB / pgvector | Nutrition knowledge base for RAG (LLaMA-3) | Vector embeddings |
| Sri Lankan Nutrition DB | Culturally specific food-nutrition mapping | PostgreSQL / CSV |
| WHO Guidelines (RAG) | Diabetes diet guidelines embedded for LLM | Vector store |

**Meal plan schema:**
```json
{
  "user_id": "patient-uuid",
  "date": "2026-05-10",
  "meals": {
    "breakfast": [
      { "food_id": "lk_string_hoppers_001", "weight_g": 150, "carbs_g": 45, "gi": 51 }
    ],
    "lunch": [...],
    "dinner": [...],
    "snacks": [...]
  },
  "totals": { "carbs_g": 180, "protein_g": 65, "fat_g": 40, "calories": 1650 },
  "gi_load_total": 72,
  "generated_by": "rl_v2.1 + llama3-8b"
}
```

---

## Infrastructure Knowledge

| Concern | Detail |
|---|---|
| LLaMA-3 | Run locally via Ollama (8B) for low latency; or Groq API for speed |
| GPU | LLaMA-3 70B needs GPU; 8B runs on CPU |
| PostgreSQL | Heavy read/write for meal history and RL feedback |
| Vector store | ChromaDB (local) or pgvector (PostgreSQL extension) |

---

## Security Knowledge

| Concern | Detail |
|---|---|
| PHI | Meal plans reference patient dietary history — encrypted |
| LLM safety | System prompt guardrails: LLaMA-3 must not give medication advice |
| Clinician override | Clinician can override or flag recommendations via `04` dashboard |

---

## Performance Knowledge

| Concern | Detail |
|---|---|
| Meal plan generation | Target < 2s (RL lookup fast; LLM for Q&A can be 3–5s) |
| Caching | Cache daily meal plan per patient (regenerate on new glucose event) |
| RAG retrieval | Pre-index nutrition knowledge base; vector search < 50ms |

---

## Testing Knowledge

| Test Type | Approach |
|---|---|
| Nutrition Tests | TDEE calculation accuracy vs Harris-Benedict reference values |
| GI Tests | Low-GI meal plan generated for T2D patient |
| Sri Lankan Tests | Meal plan contains only Sri Lankan cuisine for `lk` food_culture patients |
| RL Tests | Patient rejection of a meal type → next plan avoids it |
| LLM Tests | LLaMA-3 dietary Q&A stays within safe dietary advice scope |

---

## Observability

| Concern | Detail |
|---|---|
| Metrics | Recommendation acceptance rate, meal plan generation latency, LLM response time |
| Logging | Each plan: user_id, generation_method, gi_load, calories |
| Alerts | Acceptance rate < 50% → recommendation quality issue |

---

## Research Complexity

| Level | Justification |
|---|---|
| 🔴 Advanced | Multi-signal context fusion, RL personalization, LLM RAG advisory, Sri Lankan nutritional domain expertise, GI-based dietary planning |

---

## Portfolio Value

| Skill Demonstrated |
|---|
| LLaMA-3 RAG-based dietary advisory |
| Reinforcement learning recommendation personalization |
| GI-based diabetic meal planning |
| Sri Lankan cuisine nutritional modeling |
| Multi-signal context fusion (food + glucose + activity) |
| TDEE / BMR energy balance calculation |
| LangChain pipeline for dietary Q&A |
