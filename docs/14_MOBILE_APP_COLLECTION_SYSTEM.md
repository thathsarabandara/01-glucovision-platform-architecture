# Guided Web App Collection System (PWA)
## Structured Capture Flow with Google Drive + Google Sheets

---

## 1. Objective

Build a web app (Progressive Web App) that lets your team:
- Capture food images in a strict guided sequence.
- See the exact next photo to take after selecting a food item.
- Validate photo quality before accepting.
- Auto-fix metadata fields based on the step template.
- Auto-upload images to Google Drive in background.
- Auto-write/update metadata in Google Sheets.
- Work offline and sync later.

This document is optimized for your requirement: structured guidance without building a native Android app.

---

## 2. Best Practical Architecture (Web-First)

Use a 3-layer setup:

1. Web App (PWA on phone browser)
- Camera capture via browser (`getUserMedia`).
- Guided step engine decides the next required shot.
- Save locally first (IndexedDB).
- Background sync when internet is available.

2. Backend API (small service)
- Validation + sync handling.
- Upload to Google Drive via service account.
- Write/update rows in Google Sheets.
- Return sync status to app.

3. Google Workspace Storage
- Google Drive: image files.
- Google Sheets: metadata table.

Why this is best:
- Collectors use one URL (no Play Store release needed).
- Google Drive and Sheets stay clean and structured.
- You keep a reliable API layer for validation and future upgrades.
- PWA can behave like an app icon on Android home screen.

---

## 3. Technology Recommendation

## 3.1 Web Frontend

Preferred: React + PWA
- Fast form UX and camera integration.
- Service worker for offline caching and sync.
- Installable on Android as home-screen app.

Alternative: Vue + PWA

## 3.2 Backend

Preferred: Node.js (Express/Fastify)
- Easy Google APIs integration.
- Easy JSON handling and validation.

Alternative: Python FastAPI

## 3.3 Data and queue

- Local browser DB: IndexedDB.
- Server DB (optional but recommended): PostgreSQL/SQLite for sync logs.
- Queue (optional later): simple retry worker.

---

## 4. User Roles

| Role | Permissions |
|---|---|
| Collector | Capture photos, fill metadata, submit/sync |
| Reviewer | Edit metadata, approve/reject records |
| Admin Mode | Manage food list, dropdown vocab, exports |

No authentication mode:
- Roles are UI modes, not user accounts.
- Suitable for trusted same-network usage during data collection.

---

## 5. Field Workflow (Guided Mode)

1. Open web app -> select session.
2. Select scenario type:
- `single_dish`
- `multi_dish`
3. Select food item (example: `white_rice`).
4. App loads capture checklist template for that food.
5. App shows `Step 1` required shot (example: `A90`, weight band `140_170`, plate type `white_ceramic`).
6. Capture photo from camera.
7. Run quick quality check (blur/exposure/frame completeness).
8. If pass: auto-fill metadata fields from template + user inputs.
9. Save locally instantly.
10. App starts background upload.
11. App shows `Step 2` required shot, then `Step 3`, etc.
7. On success:
- Image stored in Drive.
- Metadata row inserted/updated in Sheet.
- Record marked `synced`.

Offline case:
- Keep local pending records.
- Auto-retry every N minutes when internet returns.

---

## 5A. Guided Capture Engine (Core Requirement)

This is the key feature you requested.

For each food, define a required step template in a table (`capture_templates`).

Example template for `white_rice`:
- Step 1: `A90`, weight `140_170`, plate `white_ceramic`, background `plain_cloth`
- Step 2: `A45`, weight `140_170`, plate `white_ceramic`, background `plain_cloth`
- Step 3: `ASIDE`, weight `140_170`, plate `white_ceramic`, background `plain_cloth`
- Step 4: `A90`, weight `180_230`, plate `clay_plate`, background `wood_table`
- Step 5: `A45`, weight `100_130`, plate `banana_leaf`, background `wood_table`

After each accepted photo:
- Mark current step `completed`.
- Unlock next required step.
- Auto-generate guidance text: "Next: change plate to clay plate and capture A90".

Collector UI should always show:
- Current step number
- Required parameters
- Progress (for example `3/12 completed`)
- Missing mandatory shots

---

## 6. Core Features for the Web App

## 6.1 Capture and metadata

- Camera capture (with grid).
- Auto timestamp.
- Optional GPS capture.
- Food ID dropdown.
- Portion weight input from scale (grams).
- Angle code dropdown (`A90`, `A45`, `ASIDE`).
- Plate/background/spoon dropdowns.
- Scenario type switch (`single_dish` or `multi_dish`).
- For multi-dish: add multiple items in one record.
- Guided "next shot" panel with parameter checklist.

## 6.2 Auto upload and sync

- Save-first local design.
- Background upload worker.
- Retry with exponential backoff.
- Sync status chip:
- `pending`
- `uploading`
- `synced`
- `failed`

## 6.3 Metadata editing

- In-app edit screen for each record.
- Sheet-based editing by reviewer.
- App can refresh updated rows from sheet/backend.

## 6.4 Quality support

- Blur warning (basic image quality check).
- Required-field validation before sync.
- Duplicate warning by unique key check.
- Step validation (cannot skip required step unless reviewer override).

## 6.5 Multi-dish boundary auto-detection (ML assist)

For `multi_dish` capture, add an assisted boundary tool:

1. User captures plate photo.
2. App sends image to backend ML endpoint.
3. Model returns proposed dish boundaries (boxes or masks).
4. User reviews and adjusts boundaries quickly.
5. Confirmed boundaries are saved into `composition_items`.

Recommended methods (priority):
- Method A (best): lightweight segmentation model (YOLOv8-seg/Mask R-CNN) fine-tuned on your dish data.
- Method B (starter): detection model (YOLO bbox) then manual refine.
- Method C (baseline): color+texture clustering with k-nearest neighbor classifier for region labeling.

Note on k-nearest neighbor:
- KNN can support a baseline for classifying candidate regions.
- For accurate boundaries in mixed meals, segmentation models will perform better than pure KNN.

User experience goal:
- Auto-propose boundaries in under 2 seconds on local network.
- One-tap accept for clear cases; drag handles for correction when needed.

---

## 7. Google Drive Design

## 7.1 Folder structure

```text
Drive Root: sri_lankan_food_dataset/
  raw/
    single_dish/
      YYYY_MM_DD_session_id/
    multi_dish/
      YYYY_MM_DD_session_id/
  processed/
  exports/
```

## 7.2 File naming format

Single:
- `IMG_{foodId}_{sessionId}_{weightBand}_{angle}_{timestamp}.jpg`

Multi:
- `IMG_MIX_{sessionId}_{itemCount}_{angle}_{timestamp}.jpg`

## 7.3 Drive permissions

- Use one service account to upload.
- Share only target dataset folder with service account email.
- Restrict public sharing.

---

## 8. Google Sheets Design

Use separate tabs in one spreadsheet.

## 8.0 New Tab: `capture_templates` (Guided Steps)

| Column |
|---|
| template_id |
| food_id |
| scenario_type |
| step_no |
| required_angle_code |
| required_weight_band |
| required_plate_type |
| required_background_type |
| required_spoon_type |
| required_container_type |
| required_garnish_style |
| is_mandatory |
| instruction_text |

## 8.0A New Tab: `ui_vocabularies` (Dropdown Sources)

| Column |
|---|
| vocab_type |
| vocab_key |
| display_text |
| is_active |
| sort_order |

Examples of `vocab_type`:
- `plate_type`
- `spoon_type`
- `background_type`
- `container_type`
- `garnish_style`

This allows all dropdown lists to be controlled from the database/sheet instead of hard-coded app values.

## 8.1 Tab: `foods_master`

| Column |
|---|
| food_id |
| food_name_en |
| food_name_local |
| category |
| nutrition_per_100g_kcal |
| nutrition_per_100g_carb_g |
| nutrition_per_100g_protein_g |
| nutrition_per_100g_fat_g |
| nutrition_per_100g_fiber_g |
| gi_value |
| source_ref |

## 8.2 Tab: `capture_sessions`

| Column |
|---|
| session_id |
| date |
| location_type |
| location_name |
| collector_id |
| device_model |
| notes |

## 8.3 Tab: `images_metadata`

| Column |
|---|
| image_id |
| session_id |
| scenario_type |
| template_id |
| template_step_no |
| local_file_name |
| drive_file_id |
| drive_file_link |
| capture_time |
| gps_lat |
| gps_lng |
| angle_code |
| plate_type |
| spoon_type |
| background_type |
| container_type |
| garnish_style |
| weight_g |
| weight_band |
| lighting_code |
| quality_score |
| duplicate_flag |
| keep_for_training |
| sync_status |
| last_updated_at |

## 8.4 Tab: `composition_items` (for multi-dish)

| Column |
|---|
| composition_item_id |
| image_id |
| item_index |
| food_id |
| instance_weight_g |
| bbox_x |
| bbox_y |
| bbox_w |
| bbox_h |
| occlusion_level |

## 8.5 Tab: `sync_log`

| Column |
|---|
| sync_id |
| image_id |
| attempt_no |
| status |
| error_message |
| created_at |

## 8.6 New Tab: `food_catalog_admin` (Admin Entry Form Source)

| Column |
|---|
| food_id |
| food_name_en |
| food_name_local |
| aliases |
| category |
| default_portion_band |
| is_active |
| created_by |
| created_at |

Use this tab as the source for searchable food selection in UI.

---

## 9. API Endpoints (Minimal Set)

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
- `POST /ml/multidish-boundaries` (image -> proposed boxes/masks)
- `POST /records/{image_id}/composition-items` (save confirmed boundaries)

Recommended request flow:
1. `POST /records` (metadata first).
2. `POST /records/{image_id}/upload` (file to Drive).
3. `POST /records/{image_id}/sync` (write final row to Sheets).

---

## 10. Auto Upload Logic

Pseudo flow:

1. User saves record -> local DB status `pending`.
2. Sync worker picks pending records.
3. Upload file to Drive.
4. If upload success: save `drive_file_id` and link.
5. Write/update metadata row in Sheets.
6. If success: mark `synced`.
7. If fail: mark `failed`, store message, schedule retry.

Retry policy:
- 1st retry: 30 sec
- 2nd retry: 2 min
- 3rd retry: 10 min
- then every 30 min (max configurable)

Also store guided progress:
- `step_status = pending/completed/skipped`
- `next_step_no` calculated server-side for consistency.

For multi-dish flow:
- boundary proposal status: `not_requested/requested/proposed/confirmed`
- save both `model_proposal` and `user_corrected` versions for audit.

---

## 11. Metadata Editing Strategy (Easy + Safe)

## 11.1 In app

- Collector can edit unsynced and synced records.
- Every edit creates `last_updated_at` change.
- App pushes patch update to backend.

## 11.2 In Google Sheets

- Reviewer edits columns directly.
- Backend periodic pull (or webhook-like trigger via Apps Script) updates app-facing database.

Safe rule:
- Use `row_version` or `updated_at` to avoid overwrite conflicts.

---

## 12. Validation Rules

Mandatory fields before sync:
- `image_id`
- `session_id`
- `scenario_type`
- `template_id`
- `template_step_no`
- `angle_code`
- `weight_g` (for single-dish)
- `at least one composition item` (for multi-dish)
- `food_id` values must exist in `foods_master`

For `multi_dish` additional requirement:
- confirmed boundaries must exist (bbox or mask) for each declared item.

Quality gates:
- reject if image too blurry (threshold)
- reject if duplicate key already exists (or mark duplicate)
- warn if captured parameters do not match required template step

---

## 13. Security and Access

Simple mode (no authentication):
- No login screen required.
- App runs in trusted local network only.
- Google service account credentials stored only on backend.
- Never keep Drive service key in browser app.
- Keep admin actions in a separate UI route (for example `/admin`) and use network isolation for protection.

Recommended minimum safeguards even without auth:
- Allow only same-LAN IP range to call backend.
- Enable CORS only for your web app origin.
- Keep periodic local backups of Sheets/metadata exports.

---

## 14. Recommended UI Screens

- Dashboard (today count, pending sync, failed sync)
- New Capture
- Guided Capture Wizard (required next shot panel)
- Single-dish form
- Multi-dish form (add multiple items)
- Multi-dish Boundary Review (auto-detect + manual correction)
- Record list
- Record detail/edit
- Sync center (pending/failed/retry)
- Settings (offline mode, sync interval)
- Admin Food Catalog (create/edit foods)
- Admin Vocabulary Manager (dropdown values)

---

## 14A. Searchable DB-driven Input UX

All major input fields should be database-backed with search:

- Food selector:
  - type-to-search (`GET /foods/search?q=rice`)
  - show top matches with category and alias
- Dropdown fields:
  - load from `ui_vocabularies`
  - only active values shown
  - cached locally for offline use

Benefits:
- Faster entry
- Fewer spelling errors
- Centralized control without app redeploy

---

## 15. Implementation Plan

## Phase 1 (MVP: 2-3 weeks)

- PWA capture + local save (IndexedDB).
- Backend upload to Drive.
- Backend write to Sheets.
- Basic sync status.
- Guided step template fetch + next-step screen.

## Phase 2 (1-2 weeks)

- Multi-dish item-entry form.
- Edit metadata flow.
- Retry logic + failure dashboard.
- Step compliance validation + optional reviewer override.
- Searchable food DB inputs and vocabulary-managed dropdowns.

## Phase 3 (1-2 weeks)

- Reviewer tools.
- Duplicate/quality checks.
- Export endpoints for training format.
- Analytics on missing step patterns.
- Multi-dish boundary auto-detection service + correction UI.

## Phase 4 (1 week)

- Local network deployment hardening.
- Role-based admin screens for food data and vocabulary editing.
- Performance tuning for on-laptop inference.

---

## 15A. Run From Laptop on Same Network (Your Plan)

Your plan is valid and practical for field pilots.

Setup:
1. Run backend + web app on laptop.
2. Connect laptop and phones to same Wi-Fi/hotspot.
3. Access app from phone using laptop LAN IP.

Example:
- Laptop IP: `192.168.8.10`
- Web app URL on phone: `http://192.168.8.10:5173`
- API base URL: `http://192.168.8.10:8080`

Requirements:
- Backend listen on `0.0.0.0`.
- Frontend dev/prod server listen on `0.0.0.0`.
- Open firewall for app/API ports.
- Use fixed local IP or DHCP reservation.

Recommended local services on laptop:
- PWA frontend
- API backend
- Optional local ML inference service (for boundary detection)

Production note:
- Start with HTTP on local trusted network.
- Move to HTTPS + domain + reverse proxy when scaling beyond pilot.

---

## 16. What Goes to Training Pipeline

Keep sheet as collection/edit layer, then export to training files:

Single-dish export:
- `image_path,class_id[,weight_g]`

Multi-dish export:
- COCO or YOLO labels from `composition_items`

Final rule:
- App + Sheets are operational tools.
- Canonical training exports are generated from validated sheet/backend records.

---

## 17. Final Recommendation

Best approach for your project:
- Build one guided web app (PWA) with offline-first design.
- Use backend service account integration for Drive and Sheets.
- Keep one unified metadata model with `scenario_type`.
- Use template-driven next-step guidance after each capture.
- Add multi-dish ML boundary proposal with human correction.
- Use searchable database-driven inputs and admin-managed dropdown vocabularies.
- Host on laptop and run over same network for easy pilot deployment.
- Keep the app authentication-free for fast field operation in trusted environments.
- Use auto-upload + retry + sync dashboard to reduce manual work.

This will give you fast field collection, low annotation friction, and clean training-ready data.
