# Unified Food Dataset Strategy
## Single-Dish + Multi-Dish Collection, Labeling, Storage, and Training

---

## 1. Goal

Build a practical Sri Lankan meal AI system that works in real life:
- Single-dish images (clean class learning)
- Multi-dish plate images (real meal behavior)
- Portion-aware metadata (weight from scale)

Core question answered in this file:
- Should single-dish and multi-dish data be separate datasets or one unified dataset?

Short answer:
- Keep **one unified master dataset** with clear `scenario_type` labels.
- Create **derived training subsets** for different model stages.

This gives easier maintenance, less duplication, and better model evolution.

---

## 2. Best Architecture Decision

Do not shoot randomly. Use a pre-planned matrix so every image has a unique combination.

### Variation Dimensions

| Dimension | Examples | Why it helps |
|---|---|---|
| Plate Type | White ceramic, clay plate, banana leaf, steel plate | Changes context and edge boundaries |
| Spoon/Cutlery Type | Stainless spoon, wooden spoon, brass spoon, no spoon | Changes nearby object cues |
| Background | Wood table, plain cloth, granite, woven mat | Improves generalization |
| Portion Weight | 80g, 120g, 160g, 220g | Supports portion estimation |
| Camera Angle | 90 deg top, 45 deg, side/profile | Supports recognition + volume cues |
| Garnish/Serving Style | With curry leaf, with sambol side, plain | Real serving variation |
| Container/Serving Vessel | Bowl vs plate vs banana leaf wrap | Local authenticity |

### Recommended Local Authentic Props (Sri Lanka)

- Plates: white hotel plate, clay plate, enamel plate, banana leaf.
- Spoons: stainless steel spoon, wooden spoon, brass spoon.
- Serving context: rice and curry style side placement, sambol small side cup, clay pot serving bowl.

---

## 2.1 Recommended approach: One master dataset, multiple training views

Use one source of truth for all images and annotations:
- Same metadata system
- Same `food_id` definitions
- Same QA rules
- Same nutrition master table

Then export filtered subsets:
- `single_classification_subset`
- `multi_detection_subset`
- `multi_portion_subset` (if per-item weight available)

Why this is best:
- No duplicate management between separate repositories.
- Nutrition mapping is consistent.
- Easier versioning and audits.
- Supports future multi-task learning.

## 2.2 When to keep physically separate folders

Physical separation is still useful for operations:
- `raw/single_dish/`
- `raw/multi_dish/`

But logically they remain one dataset via shared IDs and shared metadata tables.

---

## 3. Scenario Definitions

| Scenario | Meaning | Typical Label Type |
|---|---|---|
| `single_dish` | One dominant food item per image | single class label |
| `multi_dish` | Two or more food items on one plate/tray | instance labels (bbox/mask) |
| `single_with_side` | One primary dish + tiny side/garnish | primary class + optional side tags |

Rule:
- If two foods are both substantial, label as `multi_dish`.
- If side is minimal garnish only, keep `single_dish` with garnish metadata.

---

## 4. Collection Strategy

## 4.1 Single-dish collection target

Per food class target:
- `120-180` kept unique images (minimum)

Variation dimensions (no variable light policy):
- Weight bands: 4
- Angles: 3 (`A90`, `A45`, `ASIDE`)
- Plate/container types: 2+
- Spoon/cutlery styles: 2+ (plus none)
- Backgrounds: 2+
- Garnish styles: 2+

## 4.2 Multi-dish collection target

Per composition family target:
- `80-120` kept images

Composition families example:
- Rice + 1 curry + sambol
- Rice + 2 curries + sambol
- Rice + 3 curries + side
- Hopper/string hopper + 2-3 sides

Project-scale recommendation:
- Multi-dish images should be at least `25-35%` of total training pool for production-like behavior.

---

## 5. Non-Duplication Rules

## 5.1 Hard duplicate rule

Reject one image if all are same:
- same `food_id` or same exact composition set
- same weight band(s)
- same angle
- same plate/container
- same background
- near-identical framing from same burst

## 5.2 Uniqueness keys

Single dish:
- `dish_key = food_id + weight_band + angle_code + plate_type + background_type + garnish_style + container_type`

Multi dish:
- `composition_key = sorted(food_ids) + item_count + layout_type + angle_code + plate_type + background_type`

## 5.3 QA duplicate checks

Use all three:
- Visual review
- Hash check (exact duplicate)
- Perceptual similarity threshold (near duplicates)

---

## 6. Fixed-Light Policy

Use one standardized ring-light setup for all captures:

| Field | Value |
|---|---|
| lighting_code | `L_FIXED` |
| position | front |
| intensity_pct | 70 |
| kelvin | 5000 |

Lighting is recorded for traceability, not used as a variation dimension.

---

## 7. Data Saving Approach (Master Tables)

Use CSV/SQL tables as canonical source. Image folders are storage only.

## 7.1 `foods_master`

One row per food class.

| Column | Type | Notes |
|---|---|---|
| food_id | string | primary key |
| food_name_en | string | standard model class name |
| food_name_local | string | Sinhala/Tamil/local |
| category | string | carb/curry/side/etc |
| nutrition_per_100g_kcal | float | required |
| nutrition_per_100g_carb_g | float | required |
| nutrition_per_100g_protein_g | float | required |
| nutrition_per_100g_fat_g | float | required |
| nutrition_per_100g_fiber_g | float | optional |
| gi_value | float | optional |
| source_ref | string | source citation |

## 7.2 `capture_sessions`

One row per shoot session.

| Column | Type | Notes |
|---|---|---|
| session_id | string | primary key |
| date | date | shoot date |
| location_type | string | home/restaurant/studio |
| location_name | string | optional |
| device_model | string | camera model |
| photographer_id | string | owner |
| remarks | string | optional |

## 7.3 `images_metadata` (core table)

One row per image.

| Column | Type | Notes |
|---|---|---|
| image_id | string | primary key |
| file_path | string | relative image path |
| session_id | string | FK capture_sessions |
| scenario_type | string | `single_dish`/`multi_dish`/`single_with_side` |
| split | string | train/val/test |
| angle_code | string | A90/A45/ASIDE |
| plate_type | string | controlled vocab |
| spoon_type | string | controlled vocab |
| background_type | string | controlled vocab |
| container_type | string | controlled vocab |
| garnish_style | string | controlled vocab |
| lighting_code | string | `L_FIXED` |
| ring_light_intensity_pct | int | fixed 70 |
| ring_light_kelvin | int | fixed 5000 |
| reference_object | string | credit_card/ruler/coin |
| quality_score | float | QA score |
| duplicate_flag | int | 1 reject |
| keep_for_training | int | 1 include |
| reject_reason | string | optional |

## 7.4 `single_dish_labels`

One row for single-dish target info.

| Column | Type | Notes |
|---|---|---|
| image_id | string | FK images_metadata |
| food_id | string | FK foods_master |
| weight_g | float | measured value |
| weight_band | string | derived band |

## 7.5 `plate_compositions`

One row per mixed-plate image.

| Column | Type | Notes |
|---|---|---|
| composition_id | string | primary key |
| image_id | string | FK images_metadata |
| item_count | int | 2,3,4,5... |
| meal_type | string | rice_and_curry/etc |
| plate_layout_type | string | clock_wise/center/etc |
| total_plate_weight_g | float | optional but useful |

## 7.6 `composition_items`

One row per item instance inside mixed plate image.

| Column | Type | Notes |
|---|---|---|
| composition_item_id | string | primary key |
| composition_id | string | FK plate_compositions |
| image_id | string | FK images_metadata |
| food_id | string | FK foods_master |
| instance_index | int | ordering |
| instance_weight_g | float | if measured |
| bbox_x | int | detection label |
| bbox_y | int | detection label |
| bbox_w | int | detection label |
| bbox_h | int | detection label |
| occlusion_level | string | none/low/medium/high |

---

## 8. Folder Structure (Operational)

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

This keeps operations simple while maintaining one logical dataset.

---

## 9. Label Export Formats Before Training

## 9.1 For single-dish classifier

Export from tables:
- `single_train.csv`: `image_path,class_id`
- `single_val.csv`: `image_path,class_id`
- `single_test.csv`: `image_path,class_id`

Optional:
- `image_path,class_id,weight_g`

## 9.2 For multi-dish detector

Export either:
- YOLO format (`txt` per image with class + bbox), or
- COCO JSON with instances.

Source always:
- `images_metadata` + `composition_items`.

## 9.3 For portion estimation (optional head)

Export:
- `image_id,food_id,bbox,instance_weight_g,reference_object`

Use only rows with reliable `instance_weight_g`.

---

## 10. Training Plan (Best Practical Approach)

## Phase A: Single-dish foundation

Train classifier on `single_dish` subset first.
Purpose:
- Learn core visual identity of each food.

## Phase B: Mixed-plate detection

Train detector/segmenter using `multi_dish` subset.
Purpose:
- Find all foods in one plate.

## Phase C: Optional portion head

Train per-item gram estimation using measured rows.
Purpose:
- Move from recognition to nutritional estimation.

## Phase D: Joint evaluation

Run end-to-end on realistic mixed-plate test set.

---

## 11. Split and Leakage Control

Split at session/composition level, not random image level.

Rules:
- Burst-neighbor images must stay in same split.
- Very similar composition layouts must not cross train/test.
- Keep scenario balance in each split:
  - single-dish proportion
  - multi-dish proportion
  - item-count distribution in multi-dish

Recommended:
- train 70%
- val 15%
- test 15%

---

## 12. Metrics You Should Report

Single-dish:
- Top-1 accuracy
- Macro F1

Multi-dish:
- mAP (detection)
- per-class precision/recall in mixed scenes
- set-level plate F1 (predicted food set vs ground truth set)

Portion (if enabled):
- MAE in grams per item
- Relative error percentage

---

## 13. What To Do Today (Practical Start)

1. Keep collecting single-dish as planned.
2. Start a parallel mixed-plate collection sheet from day one.
3. Use the same `food_id` catalog for both scenarios.
4. Save everything into unified master tables.
5. Export separate training files only at training time.

This is the most scalable and research-safe approach for your final production goal.

---

## 14. Minimum Completion Checklist

| Check | Target |
|---|---|
| Single-dish unique images per class | >=120 |
| Mixed-plate composition families | >=8 |
| Multi-dish share of total train pool | 25-35% |
| Duplicate rate after QA | <5% |
| Fixed-light compliance | 100% (`L_FIXED`) |
| Images with complete metadata | 100% |
| Multi-dish images with instance labels | 100% |

If all rows above are satisfied, the dataset is ready for real-world model training, not just lab-style single-dish classification.
