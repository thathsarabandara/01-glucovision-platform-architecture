# Supervisor Approval Document
## Unified Data Collection, Labeling, and Mobile Capture System for Sri Lankan Food AI

---

## 1. Purpose of This Document

This document combines all required facts from:
- `13_IMAGE_VARIATION_AND_LABELING_PROTOCOL.md`
- `14_MOBILE_APP_COLLECTION_SYSTEM.md`

It is prepared as a single approval-ready plan for the **data collection period**, covering:
- Dataset strategy and labeling design
- Image variation protocol
- Storage architecture and metadata standards
- Guided mobile/web collection workflow
- Google Drive + Google Sheets integration
- Validation, QA, and duplicate prevention
- Multi-dish boundary annotation support
- Implementation timeline and readiness checklist

---

## 2. Project Objective

Build a practical AI-ready Sri Lankan meal dataset and collection system that works in real-world conditions.

The system must support:
- Single-dish images for strong class learning
- Multi-dish plate images for real meal behavior
- Portion-aware metadata from measured weights
- Structured guided capture flow to reduce human error
- Offline-capable field collection with later synchronization

---

## 3. Core Architecture Decision (Approved Strategy)

### 3.1 One Unified Master Dataset

Use **one logical master dataset** for all captures, with explicit scenario labeling:
- `single_dish`
- `multi_dish`
- `single_with_side`

Reason:
- Prevents duplicate management across multiple datasets
- Keeps one `food_id` catalog and one nutrition mapping
- Makes auditing and versioning easier
- Supports future multi-task learning

### 3.2 Multiple Derived Training Views

From the master dataset, export task-specific subsets:
- `single_classification_subset`
- `multi_detection_subset`
- `multi_portion_subset` (where reliable per-item weight exists)

### 3.3 Physical Folder Separation (Operational Only)

Operational raw storage can remain separated:
- `raw/single_dish/`
- `raw/multi_dish/`

But these are still part of one logical dataset connected by shared IDs and metadata tables.

---

## 4. Scenario Definitions and Label Rules

| Scenario | Definition | Typical Labels |
|---|---|---|
| `single_dish` | One dominant food item in image | single class label |
| `multi_dish` | Two or more substantial food items | instance labels (bbox/mask) |
| `single_with_side` | One primary dish + minimal side/garnish | primary class + side tags |

Rules:
- If two foods are both substantial -> label as `multi_dish`
- If secondary item is minimal garnish only -> keep as `single_dish` with garnish metadata

---

## 5. Data Collection Design During Approval Period

## 5.1 Variation Matrix (Mandatory)

Each captured image must follow planned combinations across these dimensions:

| Dimension | Examples | Why |
|---|---|---|
| Plate type | white ceramic, clay, banana leaf, steel | boundary/context robustness |
| Spoon/cutlery | stainless, wooden, brass, none | contextual variation |
| Background | wood table, cloth, granite, woven mat | generalization |
| Portion weight | 80g, 120g, 160g, 220g | portion-learning support |
| Camera angle | `A90`, `A45`, `ASIDE` | appearance and volume cues |
| Garnish style | curry leaf, sambol side, plain | realistic presentation |
| Container type | bowl, plate, banana leaf wrap | local authenticity |

Recommended Sri Lankan props:
- Plates: white hotel plate, clay plate, enamel plate, banana leaf
- Spoons: stainless, wooden, brass
- Serving context: rice and curry side placement, sambol side cup, clay pot bowl

## 5.2 Fixed-Light Policy

Lighting is controlled and not varied:
- `lighting_code = L_FIXED`
- ring light position: front
- intensity: `70%`
- color temperature: `5000K`

Record lighting for traceability only.

## 5.3 Target Volumes

Single-dish target per class:
- `120-180` unique kept images (minimum 120)

Variation expected per class:
- 4 weight bands
- 3 angles (`A90`, `A45`, `ASIDE`)
- 2+ plate/container types
- 2+ spoon styles (+ none)
- 2+ backgrounds
- 2+ garnish styles

Multi-dish target per composition family:
- `80-120` kept images

Example composition families:
- Rice + 1 curry + sambol
- Rice + 2 curries + sambol
- Rice + 3 curries + side
- Hopper/string hopper + 2-3 sides

Project-scale requirement:
- Multi-dish share of training pool: `25-35%`

---

## 6. Non-Duplication and QA Control

## 6.1 Hard Duplicate Rejection

Reject one image if all match:
- Same `food_id` (or same exact composition set)
- Same weight band(s)
- Same angle
- Same plate/container
- Same background
- Near-identical framing from same burst

## 6.2 Uniqueness Keys

Single-dish key:
- `food_id + weight_band + angle_code + plate_type + background_type + garnish_style + container_type`

Multi-dish key:
- `sorted(food_ids) + item_count + layout_type + angle_code + plate_type + background_type`

## 6.3 QA Duplicate Detection Methods

Use all three:
- Visual review
- Exact hash duplicate check
- Perceptual similarity threshold for near duplicates

---

## 7. Canonical Data Model (Master Tables)

Use structured CSV/SQL tables as source of truth. Image folders are file storage only.

## 7.1 `foods_master`

Fields:
- `food_id` (PK)
- `food_name_en`
- `food_name_local`
- `category`
- `nutrition_per_100g_kcal`
- `nutrition_per_100g_carb_g`
- `nutrition_per_100g_protein_g`
- `nutrition_per_100g_fat_g`
- `nutrition_per_100g_fiber_g` (optional)
- `gi_value` (optional)
- `source_ref`

## 7.2 `capture_sessions`

Fields:
- `session_id` (PK)
- `date`
- `location_type`
- `location_name`
- `device_model`
- `photographer_id` / `collector_id`
- `remarks` / `notes`

## 7.3 `images_metadata` (Core)

Fields include:
- IDs and references (`image_id`, `session_id`, file names)
- Scenario (`scenario_type`)
- Guided template references (`template_id`, `template_step_no`)
- Capture parameters (`angle_code`, `plate_type`, `spoon_type`, `background_type`, `container_type`, `garnish_style`)
- Weight and derived band (`weight_g`, `weight_band`)
- Lighting (`lighting_code`, intensity, kelvin)
- Geodata/time (`capture_time`, optional GPS)
- Quality flags (`quality_score`, `duplicate_flag`, `keep_for_training`)
- Sync metadata (`sync_status`, `last_updated_at`)
- Drive metadata (`drive_file_id`, `drive_file_link`)

## 7.4 `single_dish_labels`

Fields:
- `image_id` (FK)
- `food_id` (FK)
- `weight_g`
- `weight_band`

## 7.5 `plate_compositions`

Fields:
- `composition_id` (PK)
- `image_id` (FK)
- `item_count`
- `meal_type`
- `plate_layout_type`
- `total_plate_weight_g` (optional)

## 7.6 `composition_items`

Fields:
- `composition_item_id` (PK)
- `composition_id` (FK)
- `image_id` (FK)
- `food_id` (FK)
- `instance_index`
- `instance_weight_g` (if measured)
- Bounding box labels (`bbox_x`, `bbox_y`, `bbox_w`, `bbox_h`)
- `occlusion_level`

## 7.7 Additional Operational Tables for Guided System

`capture_templates`:
- template-driven required steps per food/scenario
- mandatory capture parameters and instruction text

`ui_vocabularies`:
- database/sheet-driven dropdown values (plate/spoon/background/container/garnish etc.)

`sync_log`:
- sync attempts, status, errors, timestamps

`food_catalog_admin`:
- admin-friendly source for searchable food list and aliases

---

## 8. Operational Folder Structure

```text
/data/sri_lankan_food_dataset_v2/
  raw/
    single_dish/
    multi_dish/
  processed/
    train/
    val/
    test/
  annotations/
    foods_master.csv
    capture_sessions.csv
    images_metadata.csv
    single_dish_labels.csv
    plate_compositions.csv
    composition_items.csv
    label_map.csv
  exports/
    single_classification/
    multi_detection/
    multi_portion/
```

Google Drive target structure:

```text
sri_lankan_food_dataset/
  raw/
    single_dish/
      YYYY_MM_DD_session_id/
    multi_dish/
      YYYY_MM_DD_session_id/
  processed/
  exports/
```

File naming:
- Single: `IMG_{foodId}_{sessionId}_{weightBand}_{angle}_{timestamp}.jpg`
- Multi: `IMG_MIX_{sessionId}_{itemCount}_{angle}_{timestamp}.jpg`

---

## 9. Guided Web App Collection System (PWA)

## 9.1 Collection Platform

Web-first 3-layer architecture:

1. PWA Frontend (phone browser)
- Camera via `getUserMedia`
- Guided next-step capture engine
- Local save in IndexedDB
- Background sync

2. Backend API service
- Validation and orchestration
- Google Drive upload via service account
- Google Sheets row insert/update
- Sync state responses to app

3. Google Workspace storage
- Drive for images
- Sheets for metadata operations

Why selected:
- No Play Store release needed
- One URL for all collectors
- Offline-first behavior possible
- Strong future extensibility

## 9.2 Roles and Access Model

Roles:
- Collector
- Reviewer
- Admin mode

Authentication mode for pilot:
- No formal login
- Trusted local network operation

Minimum safeguards:
- Backend restricted to same LAN IP range
- Strict CORS for app origin
- Service key stored only server-side
- Admin route isolated (for example `/admin`)

## 9.3 Guided Capture Engine (Key Requirement)

Per selected food + scenario:
- Load `capture_templates`
- Show mandatory current step
- Lock next step until current step is completed
- Auto-generate instruction text for next action

UI must always show:
- Step number
- Required capture parameters
- Progress (`completed/total`)
- Missing mandatory shots

Step completion behavior:
- Save capture
- Run quality check
- Mark step complete
- Unlock next step

---

## 10. Required App Features

## 10.1 Capture + Metadata

- Camera capture with guide grid
- Auto timestamp
- Optional GPS
- Searchable food selector
- Weight input from scale
- Angle code input (`A90`, `A45`, `ASIDE`)
- Dropdowns for plate/spoon/background/container/garnish
- Scenario switch (`single_dish`, `multi_dish`)
- Multi-item entry for `multi_dish`
- Guided next-shot panel

## 10.2 Auto Upload + Sync

- Save-first local design
- Background upload worker
- Exponential retry policy
- Status states: `pending`, `uploading`, `synced`, `failed`

Retry schedule:
- 30 sec
- 2 min
- 10 min
- every 30 min (configurable)

## 10.3 Metadata Editing

- In-app editing for unsynced/synced records
- Reviewer edits in Sheets
- Backend reconciliation with version protection (`updated_at`/`row_version`)

## 10.4 Quality and Rule Enforcement

- Blur warning
- Required field validation
- Duplicate warning via uniqueness key
- Step compliance validation (skip only with reviewer override)

## 10.5 Multi-Dish Boundary Assist

Flow:
1. Capture mixed plate image
2. Send to backend ML endpoint
3. Receive proposed bboxes/masks
4. User adjusts if needed
5. Save confirmed boundaries into `composition_items`

Method priority:
- A: Segmentation model (YOLOv8-seg / Mask R-CNN)
- B: Detection model + manual refine
- C: Color+texture clustering + KNN baseline

UX target:
- Proposal response under ~2 sec on local network
- One-tap accept for clear cases

---

## 11. API Scope (Minimum Required)

- `POST /sessions`
- `GET /foods`
- `GET /foods/search?q=...`
- `POST /foods` (admin)
- `PUT /foods/{food_id}` (admin)
- `GET /capture-templates?food_id=...&scenario_type=...`
- `GET /vocabularies?vocab_type=...`
- `POST /records`
- `PUT /records/{image_id}`
- `POST /records/{image_id}/upload`
- `POST /records/{image_id}/sync`
- `GET /records/{image_id}/status`
- `POST /sessions/{session_id}/steps/{step_no}/complete`
- `GET /sessions/{session_id}/next-step`
- `POST /ml/multidish-boundaries`
- `POST /records/{image_id}/composition-items`

Recommended transaction sequence:
1. Create metadata record
2. Upload image to Drive
3. Final sync to Sheets/backend

---

## 12. Validation Rules Before Accepting Synced Records

Mandatory core fields:
- `image_id`
- `session_id`
- `scenario_type`
- `template_id`
- `template_step_no`
- `angle_code`

Conditional mandatory fields:
- For `single_dish`: `weight_g`
- For `multi_dish`: at least one composition item
- All `food_id` values must exist in `foods_master`

Additional `multi_dish` rule:
- Confirmed boundaries must exist for each declared item (bbox or mask)

Quality gates:
- Reject or flag blur below threshold
- Reject or flag duplicate key conflict
- Warn on mismatch with required template parameters

---

## 13. Data Split and Leakage Prevention Policy

Split method:
- Session/composition-level split, not random image-level split

Leakage controls:
- Keep burst-neighbor images in same split
- Keep very similar compositions in same split
- Preserve scenario and item-count distribution across splits

Recommended split ratio:
- Train: 70%
- Validation: 15%
- Test: 15%

---

## 14. Training Export and Modeling Plan

## 14.1 Export Formats

Single-dish classifier:
- `single_train.csv`, `single_val.csv`, `single_test.csv`
- Format: `image_path,class_id[,weight_g]`

Multi-dish detector/segmenter:
- YOLO txt per image or COCO JSON
- Source: `images_metadata + composition_items`

Portion estimation head:
- `image_id,food_id,bbox,instance_weight_g,reference_object`
- Use only records with reliable instance weight

## 14.2 Training Phases

Phase A:
- Train single-dish classifier

Phase B:
- Train mixed-plate detector/segmenter

Phase C:
- Train optional per-item gram estimator

Phase D:
- End-to-end evaluation on realistic mixed-plate test set

---

## 15. Evaluation Metrics

Single-dish:
- Top-1 accuracy
- Macro F1

Multi-dish:
- Detection mAP
- Per-class precision/recall in mixed scenes
- Set-level plate F1 (predicted item set vs ground truth)

Portion (if enabled):
- MAE in grams per item
- Relative error percentage

---

## 16. Deployment Plan for Data Collection Period

Pilot mode:
- Host frontend + backend on laptop
- Connect laptop and phones to same Wi-Fi/hotspot
- Use LAN IP for access from phones

Example:
- Web app: `http://192.168.8.10:5173`
- API: `http://192.168.8.10:8080`

Technical requirements:
- Frontend bind to `0.0.0.0`
- Backend bind to `0.0.0.0`
- Firewall open for required ports
- Prefer fixed local IP / DHCP reservation

Optional local ML service:
- Multi-dish boundary inference service on laptop

---

## 17. Implementation Timeline for Approval

## Phase 1 (2-3 weeks)
- PWA capture and local IndexedDB save
- Drive upload integration
- Sheets metadata write integration
- Basic sync status
- Guided template fetch + next-step UI

## Phase 2 (1-2 weeks)
- Multi-dish item entry
- Record edit flow
- Retry/failure dashboard
- Step compliance validation + reviewer override
- Searchable food input + vocabulary-driven dropdowns

## Phase 3 (1-2 weeks)
- Reviewer tooling
- Duplicate and quality checks
- Export endpoints for training
- Missing-step analytics
- Multi-dish boundary assist + correction UI

## Phase 4 (1 week)
- Local network hardening
- Admin screens for food and vocabulary management
- Performance optimization for on-laptop inference

---

## 18. Daily Practical Collection Instructions

1. Continue planned single-dish collection.
2. Start parallel multi-dish collection from day one.
3. Use the same `food_id` catalog for all scenarios.
4. Save everything to the unified master tables.
5. Export task-specific training files only during training stage.

---

## 19. Approval Success Criteria (Minimum Completion Checklist)

| Check | Target |
|---|---|
| Single-dish unique images per class | `>=120` |
| Mixed-plate composition families | `>=8` |
| Multi-dish share of training pool | `25-35%` |
| Duplicate rate after QA | `<5%` |
| Fixed-light compliance | `100%` (`L_FIXED`) |
| Images with complete metadata | `100%` |
| Multi-dish images with instance labels | `100%` |

When all criteria are met, the dataset is considered production-ready for real-world model training (not only lab-style single-dish classification).

---

## 20. Final Approval Request

This unified plan is submitted for supervisor approval for the data collection period.

Approval confirms agreement on:
- Unified dataset strategy and schema
- Guided PWA-based collection workflow
- Google Drive + Sheets operations
- QA and validation standards
- Multi-phase implementation timeline
- Completion criteria for training readiness
